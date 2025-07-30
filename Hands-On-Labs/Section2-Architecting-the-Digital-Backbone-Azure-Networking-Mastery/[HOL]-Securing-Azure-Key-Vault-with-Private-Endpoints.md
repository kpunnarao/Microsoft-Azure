# Lab: Securing Azure Key Vault with Private Endpoints 🔒

This lab will guide you through enhancing the security of your Azure Key Vault by enabling Azure Private Endpoints. 

This allows private access to your Key Vault from within your Azure Virtual Network, effectively isolating it from the public internet. 

We will demonstrate the configuration using Azure Portal, Azure CLI, and Azure PowerShell for comprehensive understanding.

## Lab Scenario 🧪

You have an existing Azure Key Vault that is currently accessible via its public endpoint (or has specific public IP firewall rules). 

You want to restrict access to this Key Vault so that it can only be accessed from resources within your Azure Virtual Network.

Your goal is to:

* Configure a Private Endpoint for your Key Vault within a dedicated subnet of your Azure Virtual Network.

* Integrate with Azure Private DNS to ensure seamless name resolution.

* Restrict public network access to your Key Vault using network firewall rules.

* Verify that you can only connect to the Key Vault from within your Virtual Network via the private endpoint.

This lab will demonstrate the core steps of setting up the private endpoint, configuring network access, and verifying secure connectivity.

## Part 1: Prerequisites

* An active Azure subscription.

* **Azure Key Vault**: An existing Key Vault (e.g., `mykeyvaultyouruniqueid`).
    
    * **Crucial Note for this Lab**: For initial setup and testing, ensure your Key Vault's "Firewall and virtual networks" setting is currently set to "Allow public access from all networks" or "Allow public access from specific virtual networks and IP addresses" with your current IP allowed. We will restrict this later.
    
    * The Key Vault should contain at least one secret (e.g., `mytestsecret`) with a dummy value to test retrieval.

* **Azure Virtual Network (VNet)**: An existing VNet where you will deploy the Private Endpoint.
    
    * Example: `vnet-kv-pe-lab` with address space `10.0.0.0/16`.

* **Dedicated Subnet for Private Endpoint**: A subnet within your VNet specifically for Private Endpoints.
    
    * Example: `snet-privateendpoints` within `vnet-kv-pe-lab`, with address space `10.0.1.0/24`.
    
    * **Important**: This subnet must have `privateEndpointNetworkPolicies` disabled.

* **Virtual Machine (VM) in the same VNet**: An Azure VM (e.g., Windows or Linux) deployed in *another subnet* within the same VNet as your Private Endpoint (e.g., `snet-vm` with `10.0.0.0/24`). This VM will be used to test connectivity via the private endpoint.

* **Azure CLI / Azure PowerShell with Key Vault Module**: Installed on your test VM to interact with the Key Vault. Ensure the VM has Managed Identity enabled and granted appropriate permissions to access Key Vault secrets (or you can use client credentials for simplicity in testing).

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

-----

## Part 2: Lab Setup: Prepare Your Environment

Before configuring the Private Endpoint, let's ensure your network and Key Vault are ready.

### 2.1 Verify or Create Azure Virtual Network and Subnets

You need a VNet and at least two subnets: one for your VM and a *dedicated* one for the Private Endpoint.

#### 2.1.1 Using Azure Portal

1.  **Sign in to the [Azure Portal](https://portal.azure.com/)**.
2.  Search for and select **"Virtual networks"**.
3.  **Verify/Create VNet**: Check if you have a VNet (e.g., `vnet-kv-pe-lab`). If not:
      * Click **+ Create**.
      * **Basics**: Select your `Subscription`, `Resource group` (create new: `rg-kv-pe-lab`), `Virtual network name` (e.g., `vnet-kv-pe-lab`), `Region`.
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
      * Create a new VM, selecting your `vnet-kv-pe-lab` and a different subnet (e.g., `default`). Ensure it has inbound RDP/SSH rules for your public IP for management.

#### 2.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-kv-pe-lab"
LOCATION="<YourAzureRegion>" # e.g., eastus
VNET_NAME="vnet-kv-pe-lab"
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
$resourceGroupName = "rg-kv-pe-lab"
$location = "<YourAzureRegion>" # e.g., EastUS
$vnetName = "vnet-kv-pe-lab"
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

### 2.2 Verify Azure Key Vault and Configure Secret

Ensure you have your Key Vault ready, and add a test secret.

1.  **Azure Portal**: Search for and select **"Key vaults"**.
2.  Click on your Key Vault (e.g., `mykeyvaultyouruniqueid`).
3.  On the left-hand menu, select **"Secrets"**.
4.  Click **"+ Generate/Import"**.
5.  **Create a secret**:
      * **Upload options**: `Manual`
      * **Name**: `mytestsecret`
      * **Value**: `ThisIsASecretValue`
      * Click **"Create"**.
6.  Note down the **Vault URI** (e.g., `https://mykeyvaultyouruniqueid.vault.azure.net/`).

### 2.3 Deploy a Test VM (if not already done) and Configure Access

Deploy a Windows or Linux VM into the `snet-vm` subnet of your `vnet-kv-pe-lab`. This VM will be used for testing private connectivity. Ensure you can RDP/SSH into it.

**For VM to access Key Vault (Important):**

  * **Option A (Recommended for Production): Managed Identity + RBAC**:
    1.  On your VM in Azure Portal, go to **"Identity"** and enable **"System assigned"** Managed Identity.
    2.  On your Key Vault in Azure Portal, go to **"Access policies"**.
    3.  Click **"+ Create"**.
    4.  Under **"Secret permissions"**, select `Get` and `List`.
    5.  Under **"Principal"**, search for and select your VM's name.
    6.  Click **"Review + create"**, then **"Create"**.
  * **Option B (Simple for Lab, less secure): Direct CLI/PowerShell Login**: You can just log in to Azure CLI/PowerShell from the VM using your Azure credentials, which will inherit your permissions. This is simpler for a quick lab but not best practice for applications.

-----

## Part 3: Configure Azure Private Endpoint for Key Vault

Now, let's create the Private Endpoint and configure its DNS integration.

### 3.1 Create the Private Endpoint

#### 3.1.1 Using Azure Portal (Recommended for Visual Workflow)

1.  **Navigate to your Azure Key Vault** in the Azure Portal.
2.  On the left-hand menu, under "Settings," select **"Networking"**.
3.  Go to the **"Private endpoint connections"** tab.
4.  Click **"+ Private endpoint"**.
5.  **Basics Tab**:
      * **Subscription**: Select your subscription.
      * **Resource group**: Choose `rg-kv-pe-lab` (or your Key Vault's resource group if different).
      * **Name**: Enter a descriptive name (e.g., `pe-mykeyvault`).
      * **Region**: Select the **same region as your Virtual Network** (`vnet-kv-pe-lab`).
      * Click **"Next: Resource \>"**.
6.  **Resource Tab**:
      * **Connection method**: `Connect to an Azure resource in my directory`
      * **Subscription**: Select the subscription where your Key Vault resides.
      * **Resource type**: `Microsoft.KeyVault/vaults`
      * **Resource**: Select your `mykeyvaultyouruniqueid` from the dropdown.
      * **Target sub-resource**: Select `vault`.
      * Click **"Next: Network \>"**.
7.  **Network Tab**:
      * **Virtual network**: Select `vnet-kv-pe-lab`.
      * **Subnet**: Select `snet-privateendpoints`.
      * **Private IP configuration**: Keep default `Dynamically allocate IP address`.
      * **Integrate with private DNS zone**:
          * **YES (Recommended)**: This is crucial for name resolution.
          * **Subscription**: Select your subscription.
          * **Resource group**: Choose `rg-kv-pe-lab` (or the one you prefer for DNS zones).
          * **Private DNS Zone**:
              * **Create new**: It will automatically suggest `privatelink.vaultcore.azure.net`. Accept this.
      * Click **"Next: Tags \>"** (Optional).
8.  **Tags Tab (Optional)**: Add any relevant tags.
9.  **Review + Create Tab**:
      * Review all settings.
      * Click **"Create"**.
10. **Verify Deployment**: Once created, navigate back to your Key Vault's "Networking" blade. You should see `pe-mykeyvault` listed under "Private endpoint connections" with a `Status` of `Approved`.

#### 3.1.2 Using Azure CLI

```bash
# Define variables (ensure these match your environment)
RESOURCE_GROUP="rg-kv-pe-lab"
LOCATION="<YourAzureRegion>" # e.g., eastus
VNET_NAME="vnet-kv-pe-lab"
PE_SUBNET_NAME="snet-privateendpoints"
KEY_VAULT_NAME="mykeyvaultyouruniqueid" # Replace with your Key Vault name
PRIVATE_ENDPOINT_NAME="pe-${KEY_VAULT_NAME}"
PRIVATE_DNS_ZONE_NAME="privatelink.vaultcore.azure.net"

# Get Key Vault Resource ID
echo "Retrieving Key Vault ID..."
KEY_VAULT_ID=$(az keyvault show \
    --resource-group $RESOURCE_GROUP \
    --name $KEY_VAULT_NAME \
    --query id -o tsv)
if [ -z "$KEY_VAULT_ID" ]; then
    echo "Error: Key Vault '$KEY_VAULT_NAME' not found in '$RESOURCE_GROUP'."
    exit 1
fi
echo "Key Vault ID: $KEY_VAULT_ID"

# Create Private Endpoint for Key Vault
echo "Creating Private Endpoint: $PRIVATE_ENDPOINT_NAME for Key Vault"
az network private-endpoint create \
  --name $PRIVATE_ENDPOINT_NAME \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --subnet $PE_SUBNET_NAME \
  --connection-name "${KEY_VAULT_NAME}-private-link-connection" \
  --private-connection-resource-id "$KEY_VAULT_ID" \
  --group-ids "vault" \
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

# Get the FQDN prefix of the Key Vault
KEY_VAULT_FQDN_PREFIX="$KEY_VAULT_NAME.vaultcore" # The part before .azure.net

echo "Creating A record in Private DNS Zone for $KEY_VAULT_FQDN_PREFIX to $PRIVATE_IP"
az network private-dns record-set a add-record \
  --resource-group $RESOURCE_GROUP \
  --zone-name $PRIVATE_DNS_ZONE_NAME \
  --record-set-name $KEY_VAULT_FQDN_PREFIX \
  --ipv4-address $PRIVATE_IP \
  --output tsv
```

#### 3.1.3 Using Azure PowerShell

```powershell
# Define variables (ensure these match your environment)
$resourceGroupName = "rg-kv-pe-lab"
$location = "<YourAzureRegion>" # e.g., EastUS
$vnetName = "vnet-kv-pe-lab"
$peSubnetName = "snet-privateendpoints"
$keyVaultName = "mykeyvaultyouruniqueid" # Replace with your Key Vault name
$privateEndpointName = "pe-$keyVaultName"
$privateDnsZoneName = "privatelink.vaultcore.azure.net"

# Get existing VNet and Subnet
Write-Host "Retrieving VNet '$vnetName' and Subnet '$peSubnetName'..."
$vnet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName
$subnet = Get-AzVirtualNetworkSubnetConfig -Name $peSubnetName -VirtualNetwork $vnet
if (-not $subnet) {
    Write-Error "Subnet '$peSubnetName' not found in VNet '$vnetName'. Ensure it exists and network policies are disabled."
    exit 1
}

# Get Key Vault resource ID
Write-Host "Retrieving Key Vault ID for '$keyVaultName'..."
$keyVault = Get-AzKeyVault -ResourceGroupName $resourceGroupName -VaultName $keyVaultName
if (-not $keyVault) {
    Write-Error "Key Vault '$keyVaultName' not found."
    exit 1
}
$keyVaultId = $keyVault.ResourceId

# Create Private Link Service Connection
Write-Host "Creating Private Link Service Connection..."
$privateLinkServiceConnection = New-AzPrivateLinkServiceConnection -Name "$keyVaultName-private-link-connection" `
    -PrivateLinkServiceId $keyVaultId `
    -GroupId "vault" # Target sub-resource for Key Vault

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

$keyVaultFqdnPrefix = "$keyVaultName.vaultcore" # The part before .azure.net

Write-Host "Creating A record in Private DNS Zone for '$keyVaultFqdnPrefix' to '$privateIp'"
Set-AzPrivateDnsRecordSet -ResourceGroupName $resourceGroupName `
    -ZoneName $privateDnsZoneName `
    -Name $keyVaultFqdnPrefix `
    -RecordType A `
    -Ttl 3600 `
    -PrivateDnsRecords (New-AzPrivateDnsRecordConfig -IPv4Address $privateIp) -Overwrite
```

-----

## Part 4: Restrict Public Network Access on Azure Key Vault

This is a critical security step. Once the Private Endpoint is active and DNS is configured, you should restrict public access to your Key Vault.

### 4.1 Using Azure Portal

1.  **Navigate to your Azure Key Vault**.
2.  On the left-hand menu, under "Settings," select **"Networking"**.
3.  Under **"Firewalls and virtual networks"**:
      * Select **"Private endpoint and selected networks"**.
      * Ensure the checkbox **"Allow trusted Microsoft services to bypass this firewall."** is **checked**. (This is important for services like Azure Backup, Azure Monitor, etc., to still access your Key Vault if needed.)
      * You will see your newly created Private Endpoint listed under "Private endpoint connections."
4.  Click **"Save"**.
5.  Confirm the change. It may take a few minutes for the setting to apply.

### 4.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-kv-pe-lab" # Or your Key Vault's resource group
KEY_VAULT_NAME="mykeyvaultyouruniqueid" # Replace with your Key Vault name

echo "Updating Key Vault network access to 'Private endpoint and selected networks'..."
az keyvault update \
  --resource-group $RESOURCE_GROUP \
  --name $KEY_VAULT_NAME \
  --default-action Deny \
  --bypass AzureServices # Allows trusted Microsoft services to bypass the firewall
echo "Key Vault public network access restricted. This may take a few minutes to take effect."
```

### 4.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-kv-pe-lab" # Or your Key Vault's resource group
$keyVaultName = "mykeyvaultyouruniqueid" # Replace with your Key Vault name

Write-Host "Updating Key Vault network access to 'Private endpoint and selected networks'..."
Update-AzKeyVault -ResourceGroupName $resourceGroupName `
    -VaultName $keyVaultName `
    -DefaultAction Deny `
    -Bypass "AzureServices" # Allows trusted Microsoft services to bypass the firewall
Write-Host "Key Vault public network access restricted. This may take a few minutes to take effect."
```

-----

## Part 5: Testing and Validation

Now, let's confirm that your Key Vault is only accessible via the Private Endpoint from within your VNet.

### 5.1 Test Connectivity from Within the VNet (VM)

1.  **Connect to your Test VM** (e.g., via RDP or SSH).
2.  **Open a command prompt or terminal**.
3.  **Test DNS Resolution**:
      * Run `nslookup mykeyvaultyouruniqueid.vault.azure.net` (replace with your Key Vault's FQDN).
      * **Expected Result**: The output should show the **private IP address** assigned to your Private Endpoint (e.g., `10.0.1.x`) and *not* a public IP address.
4.  **Test Key Vault Connectivity**:
      * **Ensure Azure CLI or PowerShell is installed on the VM.**
      * **If using Managed Identity (Option A from Part 2.3):**
          * First, log in to Azure from the VM (e.g., `az login --identity` for CLI).
          * Try to retrieve the secret:
            ```bash
            # Azure CLI
            az keyvault secret show --vault-name mykeyvaultyouruniqueid --name mytestsecret --query value -o tsv

            # Azure PowerShell
            Get-AzKeyVaultSecret -VaultName mykeyvaultyouruniqueid -Name mytestsecret | Select-Object -ExpandProperty SecretValueText
            ```
          * **Expected Result**: The value `ThisIsASecretValue` should be returned successfully.
      * **If using Direct Login (Option B from Part 2.3):**
          * Log in to Azure from the VM (e.g., `az login` or `Connect-AzAccount`).
          * Then run the commands above to retrieve the secret.
          * **Expected Result**: The value `ThisIsASecretValue` should be returned successfully.

### 5.2 Test Connectivity from Outside the VNet (Public Internet)

1.  **From your local machine** (which is *not* connected to your Azure VNet via VPN/ExpressRoute), try to connect to your Azure Key Vault using Azure CLI/PowerShell or a browser.
      * For example, try to run `az keyvault secret show --vault-name mykeyvaultyouruniqueid --name mytestsecret` after logging in on your local machine.
      * **Expected Result**: The connection attempt should **fail** with an error indicating unauthorized access, network restriction, or a timeout. This confirms that public access has been successfully blocked.

-----

## Part 6: Troubleshooting

  * **DNS Resolution Fails or Returns Public IP**:
      * Ensure the Private DNS Zone `privatelink.vaultcore.azure.net` exists in your resource group.
      * Verify the Private DNS Zone is linked to your `vnet-kv-pe-lab`.
      * Check for an A record in the Private DNS Zone matching your Key Vault's FQDN (e.g., `mykeyvaultyouruniqueid.vaultcore` pointing to the private IP of the Private Endpoint).
      * Ensure the VM is configured to use Azure's default DNS servers (or your custom DNS servers if they forward to Azure DNS). Avoid hardcoding public DNS servers on the VM's network adapter.
  * **Connection Fails from VM (even after correct DNS)**:
      * Verify that `privateEndpointNetworkPolicies` are `Disabled` on the `snet-privateendpoints` subnet.
      * Check Network Security Group (NSG) rules on *both* the `snet-vm` (outbound to 10.0.1.x:443) and `snet-privateendpoints` (inbound from 10.0.0.x:any) subnets. For private endpoints, typically, the private endpoint subnet should have no NSG applied, or an NSG allowing all inbound/outbound on its private IP.
      * Ensure the Key Vault's "Firewalls and virtual networks" is set to "Private endpoint and selected networks" and that the "Allow trusted Microsoft services" is enabled if you rely on it.
      * Verify that your VM's identity (or your user account) has appropriate "Access policies" granted on the Key Vault (e.g., Get/List Secret permissions).
  * **Private Endpoint Status is Not "Approved"**:
      * Go to your Key Vault -\> Networking -\> Private endpoint connections. If the status is "Pending" or "Rejected," you might need to approve it manually (though it's usually automatic when created from the same subscription).

-----

## Part 7: Cleanup Resources

To avoid ongoing costs, it's crucial to delete the resources created in this lab.

**(Using Azure CLI - Recommended for comprehensive cleanup)**

```bash
# Define variables
RESOURCE_GROUP="rg-kv-pe-lab" # The resource group containing your VNet, PE, and DNS Zone

# Delete the resource group and all contained resources
echo "Deleting resource group: $RESOURCE_GROUP and all its contents..."
az group delete --name $RESOURCE_GROUP --yes --no-wait
echo "Deletion process initiated. It may take some time to complete."

# Optional: If your Key Vault is in a different resource group and you want to delete it:
# KEY_VAULT_RESOURCE_GROUP="<YourKeyVaultResourceGroup>"
# KEY_VAULT_NAME="mykeyvaultyouruniqueid"
# echo "Deleting Key Vault: $KEY_VAULT_NAME in $KEY_VAULT_RESOURCE_GROUP..."
# az keyvault delete --resource-group $KEY_VAULT_RESOURCE_GROUP --name $KEY_VAULT_NAME --yes --no-wait
```

**(Using Azure Portal)**

1.  **Delete the Private Endpoint**:
      * Navigate to your Azure Key Vault.
      * Go to **"Networking"** -\> **"Private endpoint connections"**.
      * Select `pe-mykeyvault` and click **"Delete"**.
2.  **Delete the Private DNS Zone**:
      * Search for **"Private DNS zones"**.
      * Select `privatelink.vaultcore.azure.net`.
      * Click **"Delete"**.
3.  **Delete the Virtual Network**:
      * Search for **"Virtual networks"**.
      * Select `vnet-kv-pe-lab`.
      * Click **"Delete"**. This will also delete the subnets and the VM within it if the VM was deployed *after* the VNet.
4.  **Delete the Resource Group (Recommended)**:
      * The easiest way to clean up all associated resources is to delete the resource group.
      * Search for **"Resource groups"**.
      * Select `rg-kv-pe-lab`.
      * Click **"Delete resource group"**, type the resource group name to confirm, and click **"Delete"**.
      * **Warning**: This will delete *all* resources within that resource group, including your VM, VNet, Private Endpoint, and Private DNS Zone. If your Key Vault is in *this* resource group, it will also be deleted.

-----

Congratulations\! You have successfully configured and tested Azure Private Endpoints for your Azure Key Vault, significantly enhancing its security posture by allowing private network access while preventing public exposure. 

You also gained valuable experience configuring resources using Azure Portal, Azure CLI, and Azure PowerShell.