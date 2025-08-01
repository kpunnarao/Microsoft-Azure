# Lab Guide: Understanding and Configuring Azure ExpressRoute (Conceptual): 🌐

* **Conceptual vs. Real-world Lab**: a full ExpressRoute setup requires an active circuit from a connectivity provider. Therefore, a lab environment will focus on the Azure-side configuration steps and will be largely conceptual. The lab will guide the user on *what* they would do, rather than having them perform a live connection test.

* **Key Components**: Following are the core components:
    
    * **ExpressRoute Circuit**: The central object that represents the private connection. The lab will show how to create this, choose a `Service Provider`, `Peering Location`, `Bandwidth`, and `SKU` (`Standard` or `Premium`). It also highlights that the circuit is billed from the moment the `Service Key` is issued.
    
    * **Virtual Network Gateway**: The on-ramp into the Azure VNet for the ExpressRoute connection. The lab emphasize that this gateway type is `ExpressRoute`, not `VPN`, and requires a dedicated `GatewaySubnet` of at least `/27`.
    
    * **Connection**: The object that links the ExpressRoute Virtual Network Gateway to the ExpressRoute Circuit. The lab explain how to create this link and mention the use of an `Authorization key` for cross-subscription connections.

* **Command-line and Portal Instructions**: The results provide specific and up-to-date Azure CLI and PowerShell commands for each step: `az network express-route create`, `az network express-route peering create`, and `az network vnet-gateway create` (with `--gateway-type ExpressRoute`).

* **Important Caveats**: Following are few important poitns:
    
    * The long deployment time of the ExpressRoute Gateway (up to 45 minutes).
    
    * The importance of the `GatewaySubnet` name (`GatewaySubnet`) and size (`/27` or larger).
    
    * The fact that the `Service Key` is needed to provision the circuit with the connectivity provider.
    
    * The shared responsibility model with the service provider for the physical circuit.

* **Cleanup Steps**: The lab provide the correct order for deleting resources: delete the connection first, then the gateway, and finally the circuit. It also mentions that the circuit must be de-provisioned by the provider to stop billing.

This lab will guide you through the process of configuring Azure ExpressRoute, a crucial service for private, high-speed connectivity between your on-premises network and Azure. 

Since ExpressRoute relies on a physical circuit from a connectivity provider, this lab will be a conceptual walk-through, focusing on the steps you would take within Azure. 

You will not perform a live connection, but you will learn how to set up the necessary components on the Azure side.

## Lab Scenario 🧪

Your organization has decided to use Azure ExpressRoute for a dedicated and private connection from its on-premises data center to Azure. 

This connection will provide higher bandwidth and lower latency than a site-to-site VPN. 

You have a meeting with your networking team and the connectivity provider to get started, and your task is to understand and prepare the Azure side of the configuration.

**Your specific objectives for this conceptual lab are:**

* Understand the key components of an ExpressRoute deployment.

* "Create" a conceptual ExpressRoute circuit in Azure.

* "Create" an ExpressRoute-specific Virtual Network Gateway.

* "Link" the gateway to the circuit.

* Understand the billing and service key concepts.

* Learn the proper cleanup procedure for ExpressRoute resources.

## Part 1: Prerequisites

* An active Azure subscription.

* **Existing VNet and VM**: You will use a VNet (e.g., `vnet-lab` with address space `10.0.0.0/16`) and a VM (e.g., `vm-app-01`) from previous labs.

* **GatewaySubnet**: Your `vnet-lab` should have a `GatewaySubnet` of at least `/27` from the VPN Gateway lab.

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

* **A note on pricing**: ExpressRoute is a high-cost service. Creating a circuit object will incur charges as soon as the service key is issued. **You should not perform the final creation steps in this lab unless you intend to pay for the service.** This lab is designed to be a conceptual exercise to prevent unexpected charges.

-----

## Part 2: Architecture Diagram (Conceptual)

The following diagram illustrates how the ExpressRoute circuit physically and logically connects your on-premises network to Azure:

```
+------------------------------------------------------------------------------------------------+
|                                        Azure Subscription                                        |
|                                                                                                |
|   +--------------------------------------+      (Private, Dedicated Link)      +-------------+ |
|   |         Azure VNet (`vnet-lab`)      |   <-------------------------------> | Azure Edge  | |
|   |   (Address Space: 10.0.0.0/16)       |                                     | Router      | |
|   |                                      |                                     |             | |
|   |   +--------------------------+       |  +---------------------------+      |   +-------+ | |
|   |   |     `GatewaySubnet`      |       |  |  ExpressRoute Circuit     |  <---|   |   * | | |
|   |   |   (10.0.255.0/27)        |       |  |  (`er-lab-circuit`)       |      |   |   * | | |
|   |   |                          |       |  +---------------------------+      |   |   * | | |
|   |   |   +------------------+   |       |             ^                       |   +-------+ | |
|   |   |   |  VNet Gateway    |---+---+   |             |                       +-------------+
|   |   |   |  (ExpressRoute)  |       |   |             | (Service Key)              ^
|   |   |   +------------------+       |   |             |                            |
|   |   +------------------------------+   |             |                            |
|   |            |                         |             |                            |
|   |  +-----------------+                |   +---------------------+               |
|   |  | `vm-app-01`     |                |   | Connectivity Provider |               |
|   |  | (10.0.1.4)      |                |   | (e.g., Equinix)       |               |
|   |  +-----------------+                |   +---------------------+               |
|   +--------------------------------------+             ^                            |
|                                                      |                            |
|   +--------------------------------------------------+----------------------------+
|   |                                                 |                             |
|   |                  On-Premises Network                                          |
|   |                                                                                |
+------------------------------------------------------------------------------------------------+
```

-----

## Part 3: Step-by-Step Configuration (Conceptual)

### 3.1 Create an ExpressRoute Circuit

This is the first step on the Azure side. You would create a circuit resource that represents your physical connection.

#### 3.1.1 Using Azure Portal

1.  In the Azure Portal, search for `ExpressRoute circuits` and select it.
2.  Click **+ Create** and fill in the details. **Do not click Create yet.**
      * **Subscription**: Your subscription.
      * **Resource Group**: `rg-network-lab`.
      * **Region**: The region of your VNet (e.g., East US).
      * **Name**: `er-lab-circuit`.
      * **Port type**: `Provider`.
      * **Service Provider**: Select a provider (e.g., `Equinix`).
      * **Peering Location**: Select a location (e.g., `Silicon Valley`).
      * **Bandwidth**: Choose a bandwidth (e.g., `50 Mbps`).
      * **SKU**: `Standard`.
      * **Billing Model**: `Metered` or `Unlimited`.
3.  Click **Review + create** to see the summary, but **do not click Create**. This step generates the `Service Key` and starts billing.

#### 3.1.2 Using Azure CLI

```bash
# Define variables (for conceptual understanding only)
RESOURCE_GROUP="rg-network-lab"
CIRCUIT_NAME="er-lab-circuit"
LOCATION="eastus"
PROVIDER="Equinix" # Change to a real provider from `az network express-route list-service-providers`
PEERING_LOCATION="Silicon Valley"
BANDWIDTH=50
SKU="Standard"
SKU_FAMILY="MeteredData"

echo "Command to create the circuit (DO NOT RUN this command in a live environment without a provider agreement!)"
# az network express-route create --resource-group $RESOURCE_GROUP \
#   --name $CIRCUIT_NAME \
#   --location $LOCATION \
#   --provider $PROVIDER \
#   --peering-location "$PEERING_LOCATION" \
#   --bandwidth $BANDWIDTH \
#   --sku-family $SKU_FAMILY \
#   --sku-tier $SKU
```

**Conceptual Takeaway**: Once this circuit is created, you would take its `Service Key` (a unique GUID) and provide it to your connectivity provider. They use this key to provision the physical connection. The circuit status would remain `Not-Provisioned` until they complete their work.

### 3.2 Deploy an ExpressRoute Virtual Network Gateway

This gateway is a dedicated resource that handles the traffic from the ExpressRoute circuit into your VNet.

#### 3.2.1 Using Azure Portal

1.  Search for `Virtual network gateways` and click **+ Create**.
2.  Fill in the details:
      * **Gateway type**: Select **ExpressRoute**.
      * **VPN type**: N/A (this is automatically hidden).
      * **SKU**: `Standard` (The SKU determines throughput).
      * **Generation**: `Generation1`.
      * **Virtual network**: Select your `vnet-lab`.
      * **Public IP address**: Select **Create new** and name it `er-gateway-pip`.
3.  Click **Create**. This deployment will take time to provision.

#### 3.2.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
VNET_NAME="vnet-lab"
GATEWAY_NAME="er-vnet-gateway"
LOCATION="eastus"
PUBLIC_IP_NAME="er-gateway-pip"
GATEWAY_SUBNET_ID=$(az network vnet subnet show -g $RESOURCE_GROUP -n "GatewaySubnet" --vnet-name $VNET_NAME --query id -o tsv)

echo "Creating the ExpressRoute Virtual Network Gateway..."
az network public-ip create -g $RESOURCE_GROUP -n $PUBLIC_IP_NAME --allocation-method Dynamic --sku Basic
az network vnet-gateway create \
  --resource-group $RESOURCE_GROUP \
  --name $GATEWAY_NAME \
  --location $LOCATION \
  --gateway-type ExpressRoute \
  --sku Standard \
  --gateway-ip-configs name="ipconfig1" public-ip-address=$PUBLIC_IP_NAME subnet=$GATEWAY_SUBNET_ID \
  --output tsv
```

### 3.3 Link the VNet Gateway to the Circuit

This is the final step on the Azure side, creating the logical connection between the gateway and the circuit.

#### 3.3.1 Using Azure Portal

1.  Navigate to your `er-lab-circuit` ExpressRoute circuit.
2.  Under **Settings**, click **Connections**.
3.  Click **+ Add**.
4.  **Name**: `vnet-link`.
5.  **Virtual network gateway**: Select your `er-vnet-gateway` from the dropdown.
6.  Click **OK**.

#### 3.3.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
CIRCUIT_NAME="er-lab-circuit"
GATEWAY_NAME="er-vnet-gateway"

echo "Linking the ExpressRoute gateway to the circuit..."
az network express-route connection create \
  --resource-group $RESOURCE_GROUP \
  --name "vnet-link" \
  --express-route-gateway-name $GATEWAY_NAME \
  --circuit-name $CIRCUIT_NAME \
  --authorization-key <authorization-key-if-needed> \
  --output tsv
```

**Conceptual Takeaway**: Once this connection is created and the physical circuit is provisioned, BGP (Border Gateway Protocol) will be established between Azure and your on-premises routers. Your Azure VNet's resources will now have a private, high-bandwidth path to your on-premises network.

-----

## Part 4: Cleanup Resources (Conceptual and Real)

This is a critical section for ExpressRoute. Deleting resources must be done in a specific order to avoid being charged.

1.  **Delete the connection**: First, you must delete the connection object that links the gateway to the circuit.
2.  **Delete the VNet Gateway**: Next, delete the ExpressRoute Virtual Network Gateway.
3.  **De-provision the circuit**: You must work with your connectivity provider to **de-provision** the physical circuit. They will release the resource on their end.
4.  **Delete the ExpressRoute Circuit**: Once the provider has de-provisioned the circuit, you can delete the ExpressRoute circuit resource from your Azure subscription.

**Warning**: If you fail to perform step 3, you will continue to be billed for the circuit, even if it is not linked to any gateways.

-----

Congratulations\! You have successfully completed the conceptual configuration of Azure ExpressRoute, understanding its core components and the critical steps required for a successful hybrid cloud deployment. 

This knowledge is essential for architecting high-performance, private connectivity solutions with Azure.