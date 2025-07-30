# Lab: Securing Azure Storage Account with Private Endpoints: 🔒

This lab will guide you through enhancing the security of your Azure Storage Account by enabling Azure Private Endpoints. 

This allows private access to your storage account (specifically the Blob service) from within your Azure Virtual Network, effectively isolating it from the public internet. 

We will demonstrate the configuration using Azure Portal, Azure CLI, and Azure PowerShell for comprehensive understanding.

## Lab Scenario 🧪

You have an existing Azure Storage Account that is currently accessible via its public endpoint (or has specific public IP firewall rules).

Your goal is to:

* Configure a Private Endpoint for your Storage Account within a dedicated subnet of your Azure Virtual Network.

* Integrate with Azure Private DNS to ensure seamless name resolution for the Blob service.

* Restrict public network access to your Storage Account.

* Verify that you can only connect to the storage account from within your Virtual Network via the private endpoint.

This lab will demonstrate the core steps of setting up the private endpoint, configuring network access, and verifying secure connectivity.

## Part 1: Prerequisites

* An active Azure subscription.

* **Azure Storage Account**: An existing General-purpose v2 storage account (e.g., `mystorageacctyouruniqueid`).
    * **Crucial Note for this Lab**: For initial setup and testing, ensure your Storage Account's "Public network access" is currently set to "Enabled from all networks" or "Enabled from selected virtual networks and IP addresses" with your current IP allowed. We will disable this later.
    
    * It should have at least one Blob container (e.g., `mytestcontainer`) with a sample file inside to test connectivity (e.g., `testfile.txt`).

* **Azure Virtual Network (VNet)**: An existing VNet where you will deploy the Private Endpoint.
      * Example: `vnet-storage-pe-lab` with address space `10.0.0.0/16`.

* **Dedicated Subnet for Private Endpoint**: A subnet within your VNet specifically for Private Endpoints.
    
    * Example: `snet-privateendpoints` within `vnet-storage-pe-lab`, with address space `10.0.1.0/24`.
    
    * **Important**: This subnet must have `privateEndpointNetworkPolicies` disabled.

* **Virtual Machine (VM) in the same VNet**: An Azure VM (e.g., Windows or Linux) deployed in *another subnet* within the same VNet as your Private Endpoint (e.g., `snet-vm` with `10.0.0.0/24`). This VM will be used to test connectivity via the private endpoint.

* **Azure Storage Explorer / AzCopy / Azure CLI**: Installed on your test VM to interact with the storage account.

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

-----

## Part 2: Lab Setup: Prepare Your Environment

Before configuring the Private Endpoint, let's ensure your network and Storage Account are ready.

### 2.1 Verify or Create Azure Virtual Network and Subnets

You need a VNet and at least two subnets: one for your VM and a *dedicated* one for the Private Endpoint.

#### 2.1.1 Using Azure Portal

1.  **Sign in to the [Azure Portal](https://portal.azure.com/)**.
2.  Search for and select **"Virtual networks"**.
3.  **Verify/Create VNet**: Check if you have a VNet (e.g., `vnet-storage-pe-lab`). If not:
      * Click **+ Create**.
      * **Basics**: Select your `Subscription`, `Resource group` (create new: `rg-storage-pe-lab`), `Virtual network name` (e.g., `vnet-storage-pe-lab`), `Region`.
      * **IP Addresses**: Leave default `10.0.0.0/16`.
      * **Subnets**: Keep the `default` subnet as `10.0.0.0/24`.
      * Click **Review + create**, then **Create**.
4.  **Add Dedicated Subnet for Private Endpoints**:
      * Once the VNet is created, go to your VNet resource.
      * On the left-hand menu, select **"Subnets"**.
      * Click **+ Subnet**.
      * **Name**: `snet-privateendpoints`
      * **Address range (CIDR block)**: `10.0.1.0/24`
      * **Network security group**: None (for simplicity in lab, typically associate one)
      * **Private endpoint policies**: **Disable** (This is crucial for Private Endpoints)
      * Leave other settings as default. Click **Save**.
5.  **Verify VM Subnet**: Ensure you have another subnet (e.g., `default` or `snet-vm`) where your test VM resides. If you need to create a VM:
      * Create a new VM, selecting your `vnet-storage-pe-lab` and a different subnet (e.g., `default`). Ensure it has inbound RDP/SSH rules for your public IP for management.

#### 2.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-storage-pe-lab"
LOCATION="<YourAzureRegion>" # e.g., eastus
VNET_NAME="vnet-storage-pe-lab"
VM_SUBNET_NAME="snet-vm"
PE_SUBNET_NAME="snet-privateendpoints"
VNET_ADDRESS_PREFIX="10.0.0.0/16"
VM_SUBNET_PREFIX="10.0.0.0/24"
PE_SUBNET_PREFIX="10.0.1.0/24"

# Create Resource Group
echo "Creating resource group: $RESOURCE_GROUP"
az group create --name $RESOURCE_GROUP --location $LOCATION

# Create Virtual Network and VM Subnet
echo "Creating VNet: $VNET_NAME with subnet: $VM_SUBNET_NAME"
az network vnet create \
  --resource-group $RESOURCE_GROUP \
  --name $VNET_NAME \
  --address-prefix $VNET_ADDRESS_PREFIX \
  --location $LOCATION \
  --subnet-name $VM_SUBNET_NAME \
  --subnet-prefix $VM_SUBNET_PREFIX

# Add Private Endpoint Subnet with disabled network policies
echo "Adding Private Endpoint subnet: $PE_SUBNET_NAME to $VNET_NAME"
az network vnet subnet create \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --name $PE_SUBNET_NAME \
  --address-prefixes $PE_SUBNET_PREFIX \
  --disable-private-endpoint-network-policies true
```

#### 2.1.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-storage-pe-lab"
$location = "<YourAzureRegion>" # e.g., EastUS
$vnetName = "vnet-storage-pe-lab"
$vmSubnetName = "snet-vm"
$peSubnetName = "snet-privateendpoints"
$vnetAddressPrefix = "10.0.0.0/16"
$vmSubnetPrefix = "10.0.0.0/24"
$peSubnetPrefix = "10.0.1.0/24"

# Create Resource Group
Write-Host "Creating resource group: $resourceGroupName"
New-AzResourceGroup -Name $resourceGroupName -Location $location

# Create VM Subnet Configuration
$vmSubnet = New-AzVirtualNetworkSubnetConfig -Name $vmSubnetName -AddressPrefix $vmSubnetPrefix

# Create Private Endpoint Subnet Configuration with disabled policies
$peSubnet = New-AzVirtualNetworkSubnetConfig -Name $peSubnetName -AddressPrefix $peSubnetPrefix -PrivateEndpointNetworkPoliciesFlag "Disabled"

# Create Virtual Network with both subnets
Write-Host "Creating VNet: $vnetName with subnets: $vmSubnetName and $peSubnetName"
New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName -Location $location -AddressPrefix $vnetAddressPrefix -Subnet $vmSubnet,$peSubnet
```

### 2.2 Verify Azure Storage Account

Ensure you have your Storage Account ready and note its name.

1.  **Azure Portal**: Search for and select **"Storage accounts"**.
2.  Click on your storage account (e.g., `mystorageacctyouruniqueid`).
3.  Note down its **name** and create a sample Blob container (e.g., `mytestcontainer`) with a simple `testfile.txt` inside.

-----

## Part 3: Configure Azure Private Endpoint for Storage Account

Now, let's create the Private Endpoint and configure its DNS integration for the Blob service.

### 3.1 Create the Private Endpoint

#### 3.1.1 Using Azure Portal (Recommended for Visual Workflow)

1.  **Navigate to your Azure Storage Account** in the Azure Portal.
2.  On the left-hand menu, under "Security + networking," select **"Networking"**.
3.  Go to the **"Private endpoint connections"** tab.
4.  Click **"+ Private endpoint"**.
5.  **Basics Tab**:
      * **Subscription**: Select your subscription.
      * **Resource group**: Choose `rg-storage-pe-lab` (or your storage account's resource group if different).
      * **Name**: Enter a descriptive name (e.g., `pe-mystorageacct`).
      * **Region**: Select the **same region as your Virtual Network** (`vnet-storage-pe-lab`).
      * Click **"Next: Resource \>"**.
6.  **Resource Tab**:
      * **Connection method**: `Connect to an Azure resource in my directory`
      * **Subscription**: Select the subscription where your Storage Account resides.
      * **Resource type**: `Microsoft.Storage/storageAccounts`
      * **Resource**: Select your `mystorageacctyouruniqueid` from the dropdown.
      * **Target sub-resource**: Select `blob`. (You can add private endpoints for other sub-resources like `file`, `queue`, `table` separately if needed).
      * Click **"Next: Network \>"**.
7.  **Network Tab**:
      * **Virtual network**: Select `vnet-storage-pe-lab`.
      * **Subnet**: Select `snet-privateendpoints`.
      * **Private IP configuration**: Keep default `Dynamically allocate IP address`.
      * **Integrate with private DNS zone**:
          * **YES (Recommended)**: This is crucial for name resolution.
          * **Subscription**: Select your subscription.
          * **Resource group**: Choose `rg-storage-pe-lab` (or the one you prefer for DNS zones).
          * **Private DNS Zone**:
              * **Create new**: It will automatically suggest `privatelink.blob.core.windows.net`. Accept this.
      * Click **"Next: Tags \>"** (Optional).
8.  **Tags Tab (Optional)**: Add any relevant tags.
9.  **Review + Create Tab**:
      * Review all settings.
      * Click **"Create"**.
10. **Verify Deployment**: Once created, navigate back to your Storage Account's "Networking" blade. You should see `pe-mystorageacct` listed under "Private endpoint connections" with a `Status` of `Approved`.

#### 3.1.2 Using Azure CLI

```bash
# Define variables (ensure these match your environment)
RESOURCE_GROUP="rg-storage-pe-lab"
LOCATION="<YourAzureRegion>" # e.g., eastus
VNET_NAME="vnet-storage-pe-lab"
PE_SUBNET_NAME="snet-privateendpoints"
STORAGE_ACCOUNT_NAME="mystorageacctyouruniqueid" # Replace with your storage account name
PRIVATE_ENDPOINT_NAME="pe-${STORAGE_ACCOUNT_NAME}"
PRIVATE_DNS_ZONE_NAME="privatelink.blob.core.windows.net"

# Get Storage Account Resource ID
echo "Retrieving Storage Account ID..."
STORAGE_ACCOUNT_ID=$(az storage account show \
    --resource-group $RESOURCE_GROUP \
    --name $STORAGE_ACCOUNT_NAME \
    --query id -o tsv)
if [ -z "$STORAGE_ACCOUNT_ID" ]; then
    echo "Error: Storage account '$STORAGE_ACCOUNT_NAME' not found in '$RESOURCE_GROUP'."
    exit 1
fi
echo "Storage Account ID: $STORAGE_ACCOUNT_ID"

# Create Private Endpoint for Blob service
echo "Creating Private Endpoint: $PRIVATE_ENDPOINT_NAME for Blob service"
az network private-endpoint create \
  --name $PRIVATE_ENDPOINT_NAME \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --subnet $PE_SUBNET_NAME \
  --connection-name "${STORAGE_ACCOUNT_NAME}-blob-private-link-connection" \
  --private-connection-resource-id "$STORAGE_ACCOUNT_ID" \
  --group-ids "blob" \
  --location $LOCATION \
  --output tsv

# Create Private DNS Zone (if it doesn't exist)
echo "Creating Private DNS Zone: $PRIVATE_DNS_ZONE_NAME (if it doesn't exist)"
az network private-dns zone create \
  --resource-group $RESOURCE_GROUP \
  --name $PRIVATE_DNS_ZONE_NAME \
  --output tsv

# Link Private DNS Zone to Virtual Network
echo "Linking Private DNS Zone to Virtual Network: $VNET_NAME"
az network private-dns link vnet create \
  --resource-group $RESOURCE_GROUP \
  --zone-name $PRIVATE_DNS_ZONE_NAME \
  --name "${VNET_NAME}-link" \
  --virtual-network $VNET_NAME \
  --registration-enabled false \
  --output tsv

# Create DNS record for the Private Endpoint (automatic if integrated in portal, but good to know for CLI)
# Get the IP address of the Private Endpoint (this might take a moment to be assigned)
echo "Waiting for Private Endpoint IP address..."
sleep 30 # Give Azure some time to assign the IP
PRIVATE_IP=$(az network private-endpoint show \
    --name $PRIVATE_ENDPOINT_NAME \
    --resource-group $RESOURCE_GROUP \
    --query 'networkInterfaces[0].ipConfigurations[0].privateIpAddress' -o tsv)

if [ -z "$PRIVATE_IP" ]; then
    echo "Error: Could not retrieve Private Endpoint IP address. Manual check may be needed."
    exit 1
fi
echo "Private Endpoint IP: $PRIVATE_IP"

# Get the FQDN prefix of the Storage Account for Blob
STORAGE_FQDN_PREFIX="$STORAGE_ACCOUNT_NAME.blob" # The part before .core.windows.net

echo "Creating A record in Private DNS Zone for $STORAGE_FQDN_PREFIX to $PRIVATE_IP"
az network private-dns record-set a add-record \
  --resource-group $RESOURCE_GROUP \
  --zone-name $PRIVATE_DNS_ZONE_NAME \
  --record-set-name $STORAGE_FQDN_PREFIX \
  --ipv4-address $PRIVATE_IP \
  --output tsv
```

#### 3.1.3 Using Azure PowerShell

```powershell
# Define variables (ensure these match your environment)
$resourceGroupName = "rg-storage-pe-lab"
$location = "<YourAzureRegion>" # e.g., EastUS
$vnetName = "vnet-storage-pe-lab"
$peSubnetName = "snet-privateendpoints"
$storageAccountName = "mystorageacctyouruniqueid" # Replace with your storage account name
$privateEndpointName = "pe-$storageAccountName"
$privateDnsZoneName = "privatelink.blob.core.windows.net"

# Get existing VNet and Subnet
Write-Host "Retrieving VNet '$vnetName' and Subnet '$peSubnetName'..."
$vnet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName
$subnet = Get-AzVirtualNetworkSubnetConfig -Name $peSubnetName -VirtualNetwork $vnet
if (-not $subnet) {
    Write-Error "Subnet '$peSubnetName' not found in VNet '$vnetName'. Ensure it exists and network policies are disabled."
    exit 1
}

# Get Storage Account resource ID
Write-Host "Retrieving Storage Account ID for '$storageAccountName'..."
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
if (-not $storageAccount) {
    Write-Error "Storage Account '$storageAccountName' not found."
    exit 1
}
$storageAccountId = $storageAccount.Id

# Create Private Link Service Connection
Write-Host "Creating Private Link Service Connection..."
$privateLinkServiceConnection = New-AzPrivateLinkServiceConnection -Name "$storageAccountName-blob-private-link-connection" `
    -PrivateLinkServiceId $storageAccountId `
    -GroupId "blob" # Target sub-resource for blobs

# Create Private Endpoint
Write-Host "Creating Private Endpoint '$privateEndpointName'..."
$privateEndpoint = New-AzPrivateEndpoint -Name $privateEndpointName `
    -ResourceGroupName $resourceGroupName `
    -Location $location `
    -Subnet $subnet `
    -PrivateLinkServiceConnection $privateLinkServiceConnection

# Create Private DNS Zone (if it doesn't exist)
Write-Host "Creating Private DNS Zone '$privateDnsZoneName' (if it doesn't exist)..."
$privateDnsZone = Get-AzPrivateDnsZone -Name $privateDnsZoneName -ResourceGroupName $resourceGroupName -ErrorAction SilentlyContinue
if (-not $privateDnsZone) {
    $privateDnsZone = New-AzPrivateDnsZone -Name $privateDnsZoneName -ResourceGroupName $resourceGroupName
}

# Link Private DNS Zone to Virtual Network
Write-Host "Linking Private DNS Zone to Virtual Network '$vnetName'..."
New-AzPrivateDnsVirtualNetworkLink -ResourceGroupName $resourceGroupName `
    -ZoneName $privateDnsZoneName `
    -Name "${vnetName}-link" `
    -VirtualNetworkId $vnet.Id `
    -RegistrationEnabled $false

# Create DNS record for the Private Endpoint
Write-Host "Waiting for Private Endpoint IP address..."
Start-Sleep -Seconds 30 # Give Azure some time to assign the IP
$privateIp = (Get-AzNetworkInterface -ResourceGroupName $resourceGroupName -Name "$privateEndpointName.nic.$location").IpConfigurations.PrivateIpAddress
if ([string]::IsNullOrWhiteSpace($privateIp)) {
    Write-Error "Could not retrieve Private Endpoint IP address. Manual check may be needed."
    exit 1
}
Write-Host "Private Endpoint IP: $privateIp"

$storageFqdnPrefix = "$storageAccountName.blob" # The part before .core.windows.net

Write-Host "Creating A record in Private DNS Zone for '$storageFqdnPrefix' to '$privateIp'"
Set-AzPrivateDnsRecordSet -ResourceGroupName $resourceGroupName `
    -ZoneName $privateDnsZoneName `
    -Name $storageFqdnPrefix `
    -RecordType A `
    -Ttl 3600 `
    -PrivateDnsRecords (New-AzPrivateDnsRecordConfig -IPv4Address $privateIp) -Overwrite
```

-----

## Part 4: Restrict Public Network Access on Azure Storage Account

This is a critical security step. Once the Private Endpoint is active and DNS is configured, you should disable public access to your Storage Account.

### 4.1 Using Azure Portal

1.  **Navigate to your Azure Storage Account**.
2.  On the left-hand menu, under "Security + networking," select **"Networking"**.
3.  Under **"Public network access"**:
      * Select **"Disabled"**.
      * **Crucial Note**: Ensure that "Allow Azure services on the trusted services list to access this storage account" is checked if you have other Azure services that need to connect via service endpoints (e.g., Azure Backup, Azure Monitor).
4.  Click **"Save"**.
5.  Confirm the change. It may take a few minutes for the setting to apply.

### 4.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-storage-pe-lab" # Or your storage account's resource group
STORAGE_ACCOUNT_NAME="mystorageacctyouruniqueid" # Replace with your storage account name

echo "Updating public network access for Storage Account to Disabled..."
az storage account update \
  --resource-group $RESOURCE_GROUP \
  --name $STORAGE_ACCOUNT_NAME \
  --public-network-access Disabled
echo "Public network access disabled. This may take a few minutes to take effect."
```

### 4.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-storage-pe-lab" # Or your storage account's resource group
$storageAccountName = "mystorageacctyouruniqueid" # Replace with your storage account name

Write-Host "Updating public network access for Storage Account to Disabled..."
Set-AzStorageAccount -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -PublicNetworkAccess Disabled
Write-Host "Public network access disabled. This may take a few minutes to take effect."
```

-----

## Part 5: Testing and Validation

Now, let's confirm that your Storage Account is only accessible via the Private Endpoint from within your VNet.

### 5.1 Test Connectivity from Within the VNet (VM)

1.  **Connect to your Test VM** (e.g., via RDP or SSH).
2.  **Open a command prompt or terminal**.
3.  **Test DNS Resolution**:
      * Run `nslookup mystorageacctyouruniqueid.blob.core.windows.net` (replace with your server's FQDN).
      * **Expected Result**: The output should show the **private IP address** assigned to your Private Endpoint (e.0.1.x\`) and *not* a public IP address.
4.  **Test Storage Connectivity**:
      * **Using AzCopy (recommended for CLI)**:
          * First, retrieve a SAS token or Storage Account Key from the portal for `mytestcontainer`.
          * Example command to list blobs:
            ```bash
            azcopy ls "https://mystorageacctyouruniqueid.blob.core.windows.net/mytestcontainer?<YourSAS Token>"
            ```
            Or if using a storage account key:
            ```bash
            export AZURE_STORAGE_ACCOUNT=mystorageacctyouruniqueid
            export AZURE_STORAGE_KEY="<YourStorageAccountKey>"
            azcopy ls "https://mystorageacctyouruniqueid.blob.core.windows.net/mytestcontainer"
            ```
          * **Expected Result**: You should successfully list the contents of `mytestcontainer` (e.g., `testfile.txt`).
      * **Using Azure Storage Explorer (GUI)**:
          * Install Azure Storage Explorer on the VM.
          * Connect to your storage account using its FQDN.
          * Navigate to `Blob Containers` -\> `mytestcontainer`.
          * Try to download `testfile.txt`.
          * **Expected Result**: Successful connection and file download.

### 5.2 Test Connectivity from Outside the VNet (Public Internet)

1.  **From your local machine** (which is *not* connected to your Azure VNet via VPN/ExpressRoute), try to connect to your Azure Storage Account using Azure Storage Explorer or AzCopy.
      * **Expected Result**: The connection attempt should **fail** (e.g., network timeout, "Host not found" if DNS cached, or "Access Denied" if the public network access is set to "Disabled"). This confirms that public access has been successfully blocked.

-----

## Part 6: Troubleshooting

  * **DNS Resolution Fails or Returns Public IP**:
      * Ensure the Private DNS Zone `privatelink.blob.core.windows.net` exists in your resource group.
      * Verify the Private DNS Zone is linked to your `vnet-storage-pe-lab`.
      * Check for an A record in the Private DNS Zone matching your Storage Account's Blob FQDN (e.g., `mystorageacctyouruniqueid.blob` pointing to the private IP of the Private Endpoint).
      * Ensure the VM is configured to use Azure's default DNS servers (or your custom DNS servers if they forward to Azure DNS). Avoid hardcoding public DNS servers on the VM's network adapter.
  * **Connection Fails from VM (even after correct DNS)**:
      * Verify that `privateEndpointNetworkPolicies` are `Disabled` on the `snet-privateendpoints` subnet.
      * Check Network Security Group (NSG) rules on *both* the `snet-vm` (outbound to 10.0.1.x:443) and `snet-privateendpoints` (inbound from 10.0.0.x:any) subnets. For private endpoints, typically, the private endpoint subnet should have no NSG applied, or an NSG allowing all inbound/outbound on its private IP.
      * Ensure the Storage Account's "Public network access" is set to "Disabled" (after verifying internal connectivity).
  * **Private Endpoint Status is Not "Approved"**:
      * Go to your Storage Account -\> Networking -\> Private endpoint connections. If the status is "Pending" or "Rejected," you might need to approve it manually (though it's usually automatic when created from the same subscription).

-----

## Part 7: Cleanup Resources

To avoid ongoing costs, it's crucial to delete the resources created in this lab.

**(Using Azure CLI - Recommended for comprehensive cleanup)**

```bash
# Define variables
RESOURCE_GROUP="rg-storage-pe-lab" # The resource group containing your VNet, PE, and DNS Zone

# Delete the resource group and all contained resources
echo "Deleting resource group: $RESOURCE_GROUP and all its contents..."
az group delete --name $RESOURCE_GROUP --yes --no-wait
echo "Deletion process initiated. It may take some time to complete."

# Optional: If your Storage Account is in a different resource group and you want to delete it:
# STORAGE_ACCOUNT_RESOURCE_GROUP="<YourStorageAccountResourceGroup>"
# STORAGE_ACCOUNT_NAME="mystorageacctyouruniqueid"
# echo "Deleting Storage Account: $STORAGE_ACCOUNT_NAME in $STORAGE_ACCOUNT_RESOURCE_GROUP..."
# az storage account delete --resource-group $STORAGE_ACCOUNT_RESOURCE_GROUP --name $STORAGE_ACCOUNT_NAME --yes --no-wait
```

**(Using Azure Portal)**

1.  **Delete the Private Endpoint**:
      * Navigate to your Azure Storage Account.
      * Go to **"Networking"** -\> **"Private endpoint connections"**.
      * Select `pe-mystorageacct` and click **"Delete"**.
2.  **Delete the Private DNS Zone**:
      * Search for **"Private DNS zones"**.
      * Select `privatelink.blob.core.windows.net`.
      * Click **"Delete"**.
3.  **Delete the Virtual Network**:
      * Search for **"Virtual networks"**.
      * Select `vnet-storage-pe-lab`.
      * Click **"Delete"**. This will also delete the subnets and the VM within it if the VM was deployed *after* the VNet.
4.  **Delete the Resource Group (Recommended)**:
      * The easiest way to clean up all associated resources is to delete the resource group.
      * Search for **"Resource groups"**.
      * Select `rg-storage-pe-lab`.
      * Click **"Delete resource group"**, type the resource group name to confirm, and click **"Delete"**.
      * **Warning**: This will delete *all* resources within that resource group, including your VM, VNet, Private Endpoint, and Private DNS Zone. If your Storage Account is in *this* resource group, it will also be deleted.

-----

Congratulations\! You have successfully configured and tested Azure Private Endpoints for your Azure Storage Account, dramatically improving its security posture by allowing private network access while blocking public exposure. 

You also gained experience configuring resources using Azure Portal, Azure CLI, and Azure PowerShell.

-----