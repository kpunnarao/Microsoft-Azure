# Lab: Configuring Azure Virtual Network Peering 🤝

This lab will guide you through the process of setting up and validating a Virtual Network (VNet) peering connection in Azure. 

VNet peering allows you to connect two or more Virtual Networks seamlessly, enabling resources in one VNet to communicate with resources in another VNet as if they were part of the same network.

This is a fundamental building block for creating complex network topologies like hub-and-spoke.

## Lab Scenario 🧪

Your company is adopting a hub-and-spoke network architecture. 

You have your existing application resources in a VNet (`vnet-app-prod`).

You need to create a new, separate VNet (`vnet-hub-prod`) that will house shared services. 

Your goal is to establish a private, high-bandwidth connection between these two VNets using peering, allowing for private IP-based communication between all resources.

**Your specific objectives for this lab are:**

* Create a new VNet (`vnet-hub-prod`) with a non-overlapping address space.

* Deploy a test VM into the new VNet.

* Configure a two-way VNet peering connection between `vnet-app-prod` and `vnet-hub-prod`.

* Validate the connectivity by pinging/testing connections between VMs in the two different VNets.

* Examine the effective route tables to understand how peering works behind the scenes.

## Part 1: Prerequisites

* An active Azure subscription.

* **Existing VNet and VM**: You will use your `vnet-app-prod` VNet and a test VM within it (e.g., `vm-web-01`) from previous labs.

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

-----

## Part 2: Architecture Diagram (Conceptual)

The following diagram illustrates the simple hub-and-spoke topology you will create with VNet peering:

```
+------------------------------------------------------------------------------------------------+
|                                        Azure Cloud                                             |
|                                                                                                |
|     +------------------------------------------+       +------------------------------------------+
|     |     vnet-hub-prod (10.1.0.0/16)          |       |     vnet-app-prod (10.0.0.0/16)          |
|     |                                          |       |                                          |
|     |  +---------------------+                 |       |  +---------------------+                 |
|     |  | Subnet: snet-shared |                 |       |  | Subnet: snet-web    |                 |
|     |  | (10.1.1.0/24)       |                 |       |  | (10.0.0.0/24)       |                 |
|     |  |                     |                 |       |  |                     |                 |
|     |  |  +--------------+   |                 |       |  |  +--------------+   |                 |
|     |  |  | vm-hub-01    |   |                 |       |  |  | vm-web-01    |   |                 |
|     |  |  | (Private IP:  |   |                |       |  |  | (Private IP:  |   |                 |
|     |  |  | 10.1.1.4)    |   |                 |       |  |  | 10.0.0.4)    |   |                 |
|     |  |  +--------------+   |                 |       |  |  +--------------+   |                 |
|     +--------------------------|----------------+       +--------------------------|----------------+
|                                |                                                               |
|                                v                                                               v
|                             +----------------------------------------------------------------+
|                             |                   VNet Peering (Bi-directional)                |
|                             |  - vnet-hub-prod to vnet-app-prod                            |
|                             |  - vnet-app-prod to vnet-hub-prod                            |
|                             |                                                                |
|                             +----------------------------------------------------------------+
|                                                                                                |
+------------------------------------------------------------------------------------------------+
```

-----

## Part 3: Lab Setup: Prepare Your Environment

### 3.1 Create a New VNet and a Test VM

We'll create a new VNet with a non-overlapping CIDR block to represent our "hub" network.

#### 3.1.1 Using Azure Portal

1.  Navigate to `Virtual networks` and click **+ Create**.
2.  **Basics Tab**:
      * **Subscription**: Your subscription.
      * **Resource group**: `rg-network-lab`.
      * **Name**: `vnet-hub-prod`.
      * **Region**: Same as `vnet-app-prod`.
3.  **IP Addresses Tab**:
      * **IPv4 address space**: `10.1.0.0/16` (Ensure this does not overlap with `10.0.0.0/16`).
      * **Subnets**: Click the default subnet and edit it:
          * **Subnet name**: `snet-shared`.
          * **Address range**: `10.1.1.0/24`.
      * Click **Save**.
4.  Review and click **Create**.
5.  Now, deploy a new VM into this VNet. Go to `Virtual machines`, click **+ Create**, and create `vm-hub-01`.
      * Place it in `rg-network-lab`, `vnet-hub-prod`, and the `snet-shared` subnet.
      * Provide a username, password, and public IP (or use Bastion) for initial access.
      * Ensure the NSG rules for `vm-hub-01` allow RDP/SSH from your client IP.

#### 3.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
LOCATION="<YourAzureRegion>"
HUB_VNET_NAME="vnet-hub-prod"
HUB_VNET_PREFIX="10.1.0.0/16"
HUB_SUBNET_NAME="snet-shared"
HUB_SUBNET_PREFIX="10.1.1.0/24"
HUB_VM_NAME="vm-hub-01"
ADMIN_USERNAME="azureuser"
ADMIN_PASSWORD="<YourSecurePassword>" # IMPORTANT: Replace

echo "Creating VNet and Subnet for the Hub..."
az network vnet create \
  --resource-group $RESOURCE_GROUP \
  --name $HUB_VNET_NAME \
  --location $LOCATION \
  --address-prefix $HUB_VNET_PREFIX \
  --subnet-name $HUB_SUBNET_NAME \
  --subnet-prefix $HUB_SUBNET_PREFIX \
  --output tsv

echo "Creating VM in the Hub VNet..."
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $HUB_VM_NAME \
  --image "Ubuntu2204" \
  --admin-username $ADMIN_USERNAME \
  --admin-password $ADMIN_PASSWORD \
  --vnet-name $HUB_VNET_NAME \
  --subnet $HUB_SUBNET_NAME \
  --public-ip-address "vm-hub-01-pip" \
  --output tsv
```

#### 3.1.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$location = "<YourAzureRegion>"
$hubVnetName = "vnet-hub-prod"
$hubVnetPrefix = "10.1.0.0/16"
$hubSubnetName = "snet-shared"
$hubSubnetPrefix = "10.1.1.0/24"
$hubVmName = "vm-hub-01"
$adminUsername = "azureuser"
$adminPassword = ConvertTo-SecureString -String "<YourSecurePassword>" -AsPlainText -Force # IMPORTANT: Replace

Write-Host "Creating VNet and Subnet for the Hub..."
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name $hubSubnetName -AddressPrefix $hubSubnetPrefix
$vnet = New-AzVirtualNetwork -Name $hubVnetName -ResourceGroupName $resourceGroupName -Location $location -AddressPrefix $hubVnetPrefix -Subnet $subnetConfig

Write-Host "Creating VM in the Hub VNet..."
New-AzVm -ResourceGroupName $resourceGroupName -Name $hubVmName -Location $location `
  -Image "Ubuntu2204" -Size "Standard_B2s" `
  -Credential (New-Object System.Management.Automation.PSCredential($adminUsername, $adminPassword)) `
  -VnetName $hubVnetName -SubnetName $hubSubnetName `
  -PublicIpAddressName "vm-hub-01-pip"
```

-----

## Part 4: Configuring VNet Peering

VNet peering is a two-way relationship. You must create peering from VNet A to VNet B, and also from VNet B to VNet A.

### 4.1 Using Azure Portal

1.  **Peer from `vnet-app-prod` to `vnet-hub-prod`**:

      * Navigate to the `vnet-app-prod` Virtual Network.
      * In the left-hand menu, click **Peerings**.
      * Click **+ Add**.
      * **This virtual network**:
          * **Peering link name**: `peering-to-hub`.
      * **Remote virtual network**:
          * **Peering link name**: `peering-to-app`.
          * **Subscription**: Your subscription.
          * **Virtual Network**: Select `vnet-hub-prod`.
      * Leave `Allow forwarded traffic`, `Allow gateway transit`, and `Use remote gateways` unchecked for this simple lab.
      * Click **Add**.

2.  **Verify Peering Status**:

      * After a few moments, the peering status on `vnet-app-prod` should change from **Updating** to **Connected**.
      * Navigate to the `vnet-hub-prod` VNet and check its **Peerings** blade. It should show a peering link named `peering-to-app` with a status of **Connected**.

### 4.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
APP_VNET_NAME="vnet-app-prod"
HUB_VNET_NAME="vnet-hub-prod"

echo "Creating peering from App VNet to Hub VNet..."
az network vnet peering create \
  --resource-group $RESOURCE_GROUP \
  --name "peering-to-hub" \
  --vnet-name $APP_VNET_NAME \
  --remote-vnet $HUB_VNET_NAME \
  --allow-vnet-access \
  --output tsv

echo "Creating peering from Hub VNet to App VNet..."
az network vnet peering create \
  --resource-group $RESOURCE_GROUP \
  --name "peering-to-app" \
  --vnet-name $HUB_VNET_NAME \
  --remote-vnet $APP_VNET_NAME \
  --allow-vnet-access \
  --output tsv

echo "Peering creation commands sent. Status should become 'Connected' shortly."
```

### 4.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$appVnetName = "vnet-app-prod"
$hubVnetName = "vnet-hub-prod"

# Get VNet objects
$vnetApp = Get-AzVirtualNetwork -Name $appVnetName -ResourceGroupName $resourceGroupName
$vnetHub = Get-AzVirtualNetwork -Name $hubVnetName -ResourceGroupName $resourceGroupName

Write-Host "Creating peering from App VNet to Hub VNet..."
Add-AzVirtualNetworkPeering -Name "peering-to-hub" -VirtualNetwork $vnetApp `
  -RemoteVirtualNetworkId $vnetHub.Id -AllowVirtualNetworkAccess

Write-Host "Creating peering from Hub VNet to App VNet..."
Add-AzVirtualNetworkPeering -Name "peering-to-app" -VirtualNetwork $vnetHub `
  -RemoteVirtualNetworkId $vnetApp.Id -AllowVirtualNetworkAccess

Write-Host "Peering creation commands sent. Status should become 'Connected' shortly."
```

-----

## Part 5: Testing and Validation

### 5.1 Test Inter-VNet Connectivity

1.  Connect to `vm-web-01` (your VM in `vnet-app-prod`) via RDP or SSH.
2.  Find the private IP address of `vm-hub-01` (e.g., `10.1.1.4`).
3.  From `vm-web-01`, try to ping or test connectivity to `vm-hub-01`.
      * **Windows (PowerShell)**: `Test-NetConnection -ComputerName 10.1.1.4 -Port 3389` (or other RDP/SSH port).
      * **Linux (Netcat)**: `nc -vz 10.1.1.4 22`.
      * **Ping (ICMP)**: `ping 10.1.1.4`. *Note: You may need to enable an NSG rule and/or Windows/Linux host firewall rule to allow ICMP traffic.*
4.  **Expected Result**: The connection test should succeed. The VM in one VNet can now communicate with the VM in the other VNet using their private IP addresses.
5.  Repeat the test in reverse: Connect to `vm-hub-01` and try to connect to `vm-web-01`'s private IP (`10.0.0.4`). It should also succeed.

### 5.2 Validate with Effective Routes

You can confirm that Azure has automatically configured the necessary routes for VNet peering.

1.  In the Azure Portal, navigate to a VM (e.g., `vm-web-01`).
2.  In the left-hand menu, under "Settings," select **"Networking"**.
3.  Click on the Network Interface (e.g., `vm-web-01-nic`).
4.  Under "Support + troubleshooting," select **"Effective routes"**.
5.  Examine the routes. You should see a route with the following properties:
      * **Address Prefix**: `10.1.0.0/16` (the CIDR of `vnet-hub-prod`).
      * **Next Hop Type**: `VnetPeering`.
      * **Source**: `Virtual network gateway`.

This confirms that traffic destined for the hub VNet is correctly routed through the peering connection.

-----

## Part 6: Troubleshooting

  * **"Connected" Status**: Ensure both peering links show a status of **Connected**. If one says `Connected` and the other says `Initiated`, the configuration is not complete.
  * **NSG Rules**: By default, NSGs allow all inbound and outbound traffic within the VNet. However, if you have modified this, or have specific inbound/outbound rules on your test VMs, they might block traffic from the peered VNet. The NSG rule for RDP/SSH is often scoped to `Internet`, but for inter-VNet traffic, you may need a specific rule with `Source` as `10.1.0.0/16` (or the `VirtualNetwork` service tag, if `snet-shared` is within the same VNet as the destination).
  * **Overlapping Address Spaces**: If your VNets have overlapping CIDR ranges, peering will fail to establish.
  * **Host-based Firewall**: Ensure the VM's OS firewall is not blocking traffic from the peered VNet.

-----

## Part 7: Cleanup Resources

To avoid incurring ongoing costs, it's crucial to delete the resources created in this lab.

**(Using Azure CLI - Recommended for comprehensive cleanup)**

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab" # The resource group containing all your resources

# Delete the resource group and all contained resources
echo "Deleting resource group: $RESOURCE_GROUP and all its contents..."
az group delete --name $RESOURCE_GROUP --yes --no-wait
echo "Deletion process initiated. It may take some time to complete."
```

**(Using Azure Portal)**

1.  **Delete the VNet Peering Links First**:
      * Navigate to `vnet-app-prod`, select **Peerings**, select the peering links, and click **Delete**. Repeat for `vnet-hub-prod`.
2.  **Delete the VMs**:
      * Search for `Virtual machines`.
      * Select `vm-web-01` and `vm-hub-01`, then click **Delete**.
3.  **Delete the Virtual Networks**:
      * Search for `Virtual networks`.
      * Select `vnet-app-prod` and `vnet-hub-prod`, then click **Delete**.
4.  **Delete the Resource Group**:
      * If you're done with all labs in `rg-network-lab`, the most efficient way is to delete the entire resource group as detailed in previous labs.

-----

Congratulations\! You have successfully established and validated a VNet peering connection. 

You've gained hands-on experience with this fundamental networking concept, which is essential for building complex, multi-VNet architectures in Azure.