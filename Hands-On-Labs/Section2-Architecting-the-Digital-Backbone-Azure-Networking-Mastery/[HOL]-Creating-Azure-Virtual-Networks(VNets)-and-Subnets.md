# Lab: Creating Azure Virtual Networks (VNets) and Subnets 🌐

This lab will guide you through the fundamental process of establishing a secure and organized network infrastructure in Azure by creating a Virtual Network (VNet) and segmenting it into multiple subnets. 

This is a crucial foundational step for deploying almost any Azure service securely. 

We will demonstrate the configuration using Azure Portal, Azure CLI, and Azure PowerShell for comprehensive understanding.

## Lab Scenario 🧪

You are tasked with setting up a new network environment in Azure to host a multi-tier application.

This application requires different logical segments for its web servers, application servers, database servers, and a dedicated segment for future private endpoint integrations.

Your goal is to:

* Create a new Azure Resource Group.

* Create an Azure Virtual Network with a specified address space.

* Divide this VNet into several distinct subnets, each with its own IP range.

* Configure one subnet specifically for Azure Private Endpoints by disabling private endpoint network policies.

This lab will demonstrate the core steps of network planning, resource group creation, VNet deployment, and subnet segmentation.

## Part 1: Prerequisites

* An active Azure subscription.

* Appropriate Azure RBAC permissions to create resources (e.g., Contributor role or custom roles for Networking and Resource Group management).

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

-----

## Part 2: Architecture Diagram (Conceptual)

Below is a conceptual diagram of the network architecture we will create in this lab.

```
+------------------------------------------------------------------+
|               Azure Virtual Network (vnet-app-prod)              |
|               Address Space: 10.0.0.0/16                         |
|                                                                  |
|  +-------------------+   +-------------------+   +--------------+
|  |   Subnet: snet-web|   |   Subnet: snet-app|   |  Subnet:     |
|  |   10.0.0.0/24     |   |   10.0.2.0/24     |   |  snet-db     |
|  |                   |   |                   |   |  10.0.3.0/24 |
|  +-------------------+   +-------------------+   +--------------+
|                                                                  |
|  +---------------------------------------------------------------+
|  |   Subnet: snet-privateendpoints                               |
|  |   10.0.1.0/24 (Private Endpoint Policies Disabled)            |
|  +---------------------------------------------------------------+
|                                                                  |
+------------------------------------------------------------------+
```

-----

## Part 3: Lab Setup: Pre-configuration

There is no specific pre-configuration required other than having an active Azure subscription and the necessary permissions. We will start by creating a dedicated Resource Group for this lab.

-----

## Part 4: Step-by-Step Configuration: Creating VNet and Subnets

### 4.1 Create the Resource Group

First, we will create a dedicated Resource Group to hold all resources for this lab.

#### 4.1.1 Using Azure Portal

1.  **Sign in to the [Azure Portal](https://portal.azure.com/)**.
2.  In the search bar at the top, type `Resource groups` and select it from the results.
3.  Click **+ Create**.
4.  **Basics Tab**:
      * **Subscription**: Select your Azure subscription.
      * **Resource group name**: Enter `rg-network-lab`.
      * **Region**: Choose a region near you (e.g., `East US`, `West Europe`, `Southeast Asia`).
5.  Click **Review + create**, then **Create**.

#### 4.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
LOCATION="<YourAzureRegion>" # e.g., eastus

echo "Creating resource group: $RESOURCE_GROUP in $LOCATION"
az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION \
  --output tsv
```

#### 4.1.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$location = "<YourAzureRegion>" # e.g., EastUS

Write-Host "Creating resource group: $resourceGroupName in $location"
New-AzResourceGroup -Name $resourceGroupName -Location $location
```

### 4.2 Create the Virtual Network (VNet)

Now we will create the Virtual Network with its main address space.

#### 4.2.1 Using Azure Portal

1.  In the Azure Portal search bar, type `Virtual networks` and select it.
2.  Click **+ Create**.
3.  **Basics Tab**:
      * **Subscription**: Select your subscription.
      * **Resource group**: Choose `rg-network-lab`.
      * **Virtual network name**: Enter `vnet-app-prod`.
      * **Region**: Select the same region as your resource group (e.g., `East US`).
4.  **IP Addresses Tab**:
      * **IPv4 address space**: Enter `10.0.0.0/16`.
      * **Subnets**: The portal automatically creates a `default` subnet. You can remove this or keep it for now; we'll create our specific subnets.
5.  Click **Review + create**, then **Create**.

#### 4.2.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
LOCATION="<YourAzureRegion>"
VNET_NAME="vnet-app-prod"
VNET_ADDRESS_PREFIX="10.0.0.0/16"

echo "Creating Virtual Network: $VNET_NAME with address space $VNET_ADDRESS_PREFIX"
az network vnet create \
  --resource-group $RESOURCE_GROUP \
  --name $VNET_NAME \
  --address-prefix $VNET_ADDRESS_PREFIX \
  --location $LOCATION \
  --output tsv
```

#### 4.2.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$location = "<YourAzureRegion>"
$vnetName = "vnet-app-prod"
$vnetAddressPrefix = "10.0.0.0/16"

Write-Host "Creating Virtual Network: $vnetName with address space $vnetAddressPrefix"
New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName -Location $location -AddressPrefix $vnetAddressPrefix
```

### 4.3 Create Multiple Subnets within the VNet

Now we will add our specialized subnets. Note the configuration for the private endpoints subnet.

#### 4.3.1 Using Azure Portal

1.  Navigate to your newly created Virtual Network: `vnet-app-prod`.

2.  On the left-hand menu, under "Settings," select **"Subnets"**.

3.  If a `default` subnet exists, you can click on it and **Delete** it (optional, but good for a clean lab).

4.  Click **+ Subnet** for each of the following:

      * **Subnet 1: `snet-web` (for web servers)**

          * **Name**: `snet-web`
          * **Address range (CIDR block)**: `10.0.0.0/24`
          * **Network security group**: `None` (for this lab)
          * **Private endpoint policies**: `Disabled` (leave as default)
          * Click **Save**.

      * **Subnet 2: `snet-privateendpoints` (for Private Endpoints)**

          * **Name**: `snet-privateendpoints`
          * **Address range (CIDR block)**: `10.0.1.0/24`
          * **Network security group**: `None` (for this lab)
          * **Private endpoint policies**: **Disabled** (Crucial\! Select "Disabled" from the dropdown).
          * Click **Save**.

      * **Subnet 3: `snet-app` (for application servers)**

          * **Name**: `snet-app`
          * **Address range (CIDR block)**: `10.0.2.0/24`
          * **Network security group**: `None` (for this lab)
          * **Private endpoint policies**: `Disabled` (leave as default)
          * Click **Save**.

      * **Subnet 4: `snet-db` (for database servers)**

          * **Name**: `snet-db`
          * **Address range (CIDR block)**: `10.0.3.0/24`
          * **Network security group**: `None` (for this lab)
          * **Private endpoint policies**: `Disabled` (leave as default)
          * Click **Save**.

#### 4.3.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
VNET_NAME="vnet-app-prod"

# Subnet 1: snet-web
echo "Creating subnet: snet-web (10.0.0.0/24)"
az network vnet subnet create \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --name "snet-web" \
  --address-prefixes "10.0.0.0/24" \
  --output tsv

# Subnet 2: snet-privateendpoints (with private endpoint policies disabled)
echo "Creating subnet: snet-privateendpoints (10.0.1.0/24) with private endpoint policies disabled"
az network vnet subnet create \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --name "snet-privateendpoints" \
  --address-prefixes "10.0.1.0/24" \
  --disable-private-endpoint-network-policies true \
  --output tsv

# Subnet 3: snet-app
echo "Creating subnet: snet-app (10.0.2.0/24)"
az network vnet subnet create \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --name "snet-app" \
  --address-prefixes "10.0.2.0/24" \
  --output tsv

# Subnet 4: snet-db
echo "Creating subnet: snet-db (10.0.3.0/24)"
az network vnet subnet create \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --name "snet-db" \
  --address-prefixes "10.0.3.0/24" \
  --output tsv
```

#### 4.3.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$vnetName = "vnet-app-prod"

# Get the VNet object
$vnet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName

# Create Subnet Configurations
Write-Host "Creating subnet configuration: snet-web (10.0.0.0/24)"
$snetWeb = Add-AzVirtualNetworkSubnetConfig -Name "snet-web" -AddressPrefix "10.0.0.0/24" -VirtualNetwork $vnet

Write-Host "Creating subnet configuration: snet-privateendpoints (10.0.1.0/24) with private endpoint policies disabled"
$snetPE = Add-AzVirtualNetworkSubnetConfig -Name "snet-privateendpoints" -AddressPrefix "10.0.1.0/24" -PrivateEndpointNetworkPoliciesFlag "Disabled" -VirtualNetwork $vnet

Write-Host "Creating subnet configuration: snet-app (10.0.2.0/24)"
$snetApp = Add-AzVirtualNetworkSubnetConfig -Name "snet-app" -AddressPrefix "10.0.2.0/24" -VirtualNetwork $vnet

Write-Host "Creating subnet configuration: snet-db (10.0.3.0/24)"
$snetDb = Add-AzVirtualNetworkSubnetConfig -Name "snet-db" -AddressPrefix "10.0.3.0/24" -VirtualNetwork $vnet

# Apply subnet configurations to the VNet
Write-Host "Applying subnet configurations to VNet: $vnetName"
$vnet | Set-AzVirtualNetwork
```

-----

## Part 5: Verification

Verify that your Virtual Network and its subnets have been created correctly.

### 5.1 Using Azure Portal

1.  Navigate to your Virtual Network: `vnet-app-prod`.
2.  On the **Overview** blade, you can see the address space and the count of subnets.
3.  On the left-hand menu, select **"Subnets"**.
4.  You should see `snet-web`, `snet-privateendpoints`, `snet-app`, and `snet-db` listed.
5.  Click on `snet-privateendpoints` and verify that **"Private endpoint policies"** shows as **"Disabled"**.

### 5.2 Using Azure CLI

```bash
# Verify VNet details
echo "Verifying VNet details for $VNET_NAME:"
az network vnet show \
  --resource-group $RESOURCE_GROUP \
  --name $VNET_NAME \
  --query '{Name:name, Location:location, AddressSpace:addressSpace.addressPrefixes}' \
  --output jsonc

# Verify Subnet details
echo "Verifying Subnet details for $VNET_NAME:"
az network vnet subnet list \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --query '[].{Name:name, AddressPrefix:addressPrefix, PrivateEndpointNetworkPolicies:privateEndpointNetworkPolicies}' \
  --output table
```

### 5.3 Using Azure PowerShell

```powershell
# Verify VNet details
Write-Host "Verifying VNet details for $vnetName:"
Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName | Select-Object Name, Location, AddressSpace

# Verify Subnet details
Write-Host "Verifying Subnet details for $vnetName:"
Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName |
Select-Object -ExpandProperty Subnets |
Select-Object Name, AddressPrefix, PrivateEndpointNetworkPolicies
```

**Expected Results for Verification:**

  * You should see your VNet `vnet-app-prod` with the `10.0.0.0/16` address space.
  * You should see four subnets: `snet-web` (10.0.0.0/24), `snet-privateendpoints` (10.0.1.0/24), `snet-app` (10.0.2.0/24), and `snet-db` (10.0.3.0/24).
  * For `snet-privateendpoints`, `PrivateEndpointNetworkPolicies` should be `Disabled`. For other subnets, it will likely be `Enabled` (default).

-----

## Part 6: Troubleshooting

  * **"Address space conflicts" or "Invalid CIDR"**:
      * Ensure your VNet address space is large enough to contain all your subnet address spaces.
      * Make sure subnet address spaces are non-overlapping and fall within the VNet's overall address space.
      * Verify CIDR notation is correct (e.g., `/24`, `/16`).
  * **"Subnet creation failed: Private endpoint policies not disabled"**:
      * This usually happens if you try to deploy a private endpoint into a subnet where `privateEndpointNetworkPolicies` are still enabled. Ensure you explicitly disabled them for your `snet-privateendpoints` subnet.
  * **Resource Group/VNet not found**:
      * Double-check the names and regions in your commands/portal inputs.
      * Ensure you are logged into the correct Azure subscription.

-----

## Part 7: Cleanup Resources

To avoid incurring ongoing costs, it's crucial to delete the resources created in this lab.

**(Using Azure CLI - Recommended for comprehensive cleanup)**

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"

# Delete the resource group and all contained resources
echo "Deleting resource group: $RESOURCE_GROUP and all its contents..."
az group delete --name $RESOURCE_GROUP --yes --no-wait
echo "Deletion process initiated. It may take some time to complete."
```

**(Using Azure Portal)**

1.  **Delete the Resource Group**:
      * Search for **"Resource groups"**.
      * Select `rg-network-lab`.
      * Click **"Delete resource group"**, type the resource group name (`rg-network-lab`) to confirm, and click **"Delete"**.
      * **Warning**: This will delete *all* resources within that resource group, including your VNet and all its subnets.

-----

Congratulations\! You have successfully created an Azure Virtual Network and segmented it into multiple subnets, including a dedicated subnet for Private Endpoints. 

You've gained practical experience in fundamental Azure networking configurations using the Azure Portal, Azure CLI, and Azure PowerShell. This VNet is now ready to host your Azure resources securely.