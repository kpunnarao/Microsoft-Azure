# Lab: Configuring an Azure VPN Gateway for Site-to-Site Connectivity 🌉

This lab will guide you through the process of configuring an Azure VPN Gateway to create a secure, site-to-site connection. 

This type of connection is a foundational element of hybrid cloud networking, allowing your on-premises network to securely connect to your Azure Virtual Networks over the public internet.

## Lab Scenario 🧪

Your company has an on-premises network and an Azure Virtual Network (`vnet-lab`). You need to establish a secure, encrypted connection between the two to allow resources in both environments to communicate privately. 

You will simulate this setup by creating a second virtual network to act as your "on-premises" network and connect it to your Azure VNet using a VPN Gateway.

**Your specific objectives for this lab are:**

* Create a dedicated `GatewaySubnet` in your main VNet.

* Deploy an Azure VPN Gateway and a Public IP address for it.

* Create a "simulated on-premises" VNet to represent your local network.

* Configure a Local Network Gateway to represent the on-premises VPN device.

* Create a VPN connection between the Azure VPN Gateway and the Local Network Gateway.

* Validate connectivity by pinging a VM in the simulated on-premises VNet from a VM in the Azure VNet.

## Part 1: Prerequisites

* An active Azure subscription.

* **Existing VNet and VM**: You will use a VNet (e.g., `vnet-lab` with address space `10.0.0.0/16`) and a 
VM (e.g., `vm-app-01`) from previous labs.

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

* **A note on pricing**: VPN Gateways are a paid service and can take up to 45 minutes or more to deploy. Be mindful of costs and delete the resources as soon as you are done with the lab.

-----

## Part 2: Architecture Diagram (Conceptual)

The following diagram illustrates the components you will configure to establish the site-to-site connection:

```
+------------------------------------------------------------------------------------------------+
|                                    Azure Subscription                                            |
|                                                                                                |
|   +--------------------------------------+     (Secure IPsec Tunnel)    +---------------------+ |
|   |         Azure VNet (`vnet-lab`)      |   <------------------------> | On-Premises VNet    | |
|   |   (Address Space: 10.0.0.0/16)       |                              | (`vnet-onprem`)     | |
|   |                                      |                              | (Address Space:     | |
|   |   +--------------------------+       |                              |  192.168.1.0/24)    | |
|   |   |     `GatewaySubnet`      |       |                              |                     | |
|   |   |   (10.0.255.0/27)        |       |                              |  +----------------+ | |
|   |   |                          |       |                              |  | `vm-onprem-01` | | |
|   |   |                          |       |                              |  |  (192.168.1.4) | | |
|   |   |   +------------------+   |       |                              |  +----------------+ | |
|   |   |   |  VPN Gateway     |---+---+   |                              |                     | |
|   |   |   |  (`vpngw-lab`)   |       |   |   +--------------------+     |                     | |
|   |   |   +------------------+       |   |   | Local Network      |     |                     | |
|   |   +------------------------------+   |   | Gateway            |     |                     | |
|   |            |                         |   | (`lng-onprem`)     |     |                     | |
|   |  +-----------------+                |   | (represents your   |     |                     | |
|   |  | `vm-app-01`     |                |   | on-prem VPN device)|     |                     | |
|   |  | (10.0.1.4)      |                |   +--------------------+     |                     | |
|   |  +-----------------+                +------------------------------+                     |
|   +--------------------------------------+                                                     |
|                                                                                                |
+------------------------------------------------------------------------------------------------+
```

-----

## Part 3: Step-by-Step Configuration: Deploying and Configuring VPN Gateway

### 3.1 Prepare the Azure VNet

The VPN Gateway requires a dedicated subnet named `GatewaySubnet`. This subnet must not have any other resources deployed to it.

#### 3.1.1 Using Azure Portal

1.  Navigate to your `vnet-lab` virtual network.
2.  Under **Settings**, select **Subnets**.
3.  Click **+ Gateway subnet** and fill in the details:
      * **Name**: `GatewaySubnet` (This name is mandatory).
      * **Subnet address range**: `10.0.255.0/27` (A `/27` or larger is required for most SKUs. It's a good practice to use this range to avoid conflicts with standard subnets).
4.  Click **Save**.

#### 3.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
VNET_NAME="vnet-lab"

echo "Adding GatewaySubnet to the Azure VNet..."
az network vnet subnet create \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --name "GatewaySubnet" \
  --address-prefixes "10.0.255.0/27" \
  --output tsv
```

### 3.2 Deploy the VPN Gateway

This step will take a significant amount of time to complete (30-45 minutes).

#### 3.2.1 Using Azure Portal

1.  Search for `Virtual network gateways` and select it.
2.  Click **+ Create** and fill in the details:
      * **Subscription**: Your subscription.
      * **Resource Group**: `rg-network-lab`.
      * **Name**: `vpngw-lab`.
      * **Region**: The same region as your `vnet-lab`.
      * **Gateway type**: **VPN**.
      * **VPN type**: **Route-based**.
      * **SKU**: **VpnGw1** (A solid choice for lab/production. `Basic` is cheaper but has limitations and is being retired).
      * **Generation**: **Generation1**.
      * **Virtual network**: Select your `vnet-lab`.
      * **Public IP address**: Select **Create new**.
          * **Public IP address name**: `vpngw-lab-pip`.
      * **Enable active-active mode**: Leave as `Disabled`.
      * **Configure BGP**: Leave as `Disabled`.
3.  Click **Review + create**, then **Create**. This deployment will take time.

### 3.3 Simulate On-Premises Network and Create Local Network Gateway

While the VPN Gateway is deploying, we will set up the "on-premises" side of the connection.

#### 3.3.1 Using Azure Portal

1.  **Create a new VNet**: Create a new VNet named `vnet-onprem` with an address space of `192.168.1.0/24` and a subnet of `192.168.1.0/24`. Deploy a simple VM (e.g., `vm-onprem-01`) into this VNet.
2.  **Create a Public IP for the Local Network Gateway**: Search for `Public IP addresses` and create a new public IP resource.
3.  **Create a Local Network Gateway**: Search for `Local network gateways` and select it.
      * Click **+ Create**.
      * **Subscription**: Your subscription.
      * **Resource Group**: `rg-network-lab`.
      * **Name**: `lng-onprem`.
      * **Region**: The same region as your `vnet-lab`.
      * **IP address**: Enter the public IP address of the gateway you just created for the `vnet-onprem`.
      * **Address space**: `192.168.1.0/24` (the CIDR of your simulated on-premises network).
4.  Click **Review + create**, then **Create**.

### 3.4 Create the VPN Connection

This step links the Azure VPN Gateway to the Local Network Gateway.

#### 3.4.1 Using Azure Portal

1.  Navigate to your `vpngw-lab` virtual network gateway.
2.  Under **Settings**, select **Connections**.
3.  Click **+ Add**.
      * **Name**: `vnet-to-onprem`.
      * **Connection type**: **Site-to-site (IPsec)**.
      * **Virtual network gateway**: Your `vpngw-lab` should be pre-selected.
      * **Local network gateway**: Select `lng-onprem`.
      * **Shared key (PSK)**: Enter a simple, but secure string (e.g., `Azurelab123!`). This key must match on both ends of the connection (in our case, the VPN Gateway and the Local Network Gateway).
4.  Click **OK**.

-----

## Part 4: Testing and Validation

1.  **Verify Connection Status**: Navigate back to the `Connections` blade for your `vpngw-lab` gateway. After a few minutes, the **Status** of the `vnet-to-onprem` connection should change from `Unknown` or `Connecting` to **Connected**.
2.  **Ping a VM across the connection**:
      * Log in to `vm-app-01` in `vnet-lab`.
      * Open a command prompt or terminal.
      * Use `ping` to reach the private IP of `vm-onprem-01` (e.g., `ping 192.168.1.4`).
      * **Expected Result**: You should receive a successful reply, confirming that traffic is securely flowing between your Azure VNet and the simulated on-premises network.

-----

## Part 5: Troubleshooting

  * **Long Deployment Time**: This is normal for VPN Gateways. It's a resource-intensive process.
  * **Connection Status is not "Connected"**:
      * Check if the shared key in the connection object matches the one you would have configured on your on-premises device (in our case, the Local Network Gateway).
      * Verify the public IP and address space of the Local Network Gateway are correct.
      * Ensure that there are no overlapping CIDR ranges between the two VNets.
      * Check for NSG rules that may be blocking traffic.

-----

## Part 6: Cleanup Resources

VPN Gateways incur costs as long as they are provisioned. To avoid unexpected charges, delete the resources as soon as you are done with the lab.

1.  **Delete the Connection**: Navigate to the `vpngw-lab` gateway, go to **Connections**, select `vnet-to-onprem`, and click **Delete**.
2.  **Delete the Gateways**: Delete the `vpngw-lab` Virtual Network Gateway and the `lng-onprem` Local Network Gateway.
3.  **Delete other resources**: Delete the `vnet-onprem` VNet and its associated VM.
4.  Finally, delete the `rg-network-lab` resource group to remove all remaining resources.

Congratulations\! You have successfully configured an Azure VPN Gateway for a site-to-site connection, a crucial component for hybrid cloud architectures.