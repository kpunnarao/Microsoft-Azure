# Lab: Securing Azure Database for MySQL - Flexible Server with Private Endpoints 🔒

This lab will guide you through enhancing the security of your Azure Database for MySQL - Flexible Server by enabling Azure Private Endpoints. 

This allows private access to your database from within your Azure Virtual Network, effectively isolating it from the public internet. 

We will demonstrate the configuration using Azure Portal, Azure CLI, and Azure PowerShell for comprehensive understanding.

## Lab Scenario 🧪

You have an existing Azure Database for MySQL - Flexible Server (potentially migrated from a previous lab) that is currently accessible via its public endpoint (or has specific public IP firewall rules).

Your goal is to:

* Configure a Private Endpoint for your MySQL Flexible Server within a dedicated subnet of your Azure Virtual Network.
  
  
* Integrate with Azure Private DNS to ensure seamless name resolution.

* Restrict public network access to your MySQL server.

* Verify that you can only connect to the database from within your Virtual Network via the private endpoint.

This lab will demonstrate the core steps of setting up the private endpoint, configuring network access, and verifying secure connectivity.

## Part 1: Prerequisites

* An active Azure subscription.

* **Azure Database for MySQL - Flexible Server**: An existing server (e.g., `mymysqlflexserver-youruniqueid`).
    
    * **Crucial Note for this Lab**: For initial setup and testing, ensure your MySQL server's "Public access" is currently enabled or configured to allow your current public IP. We will disable this later.

* **Azure Virtual Network (VNet)**: An existing VNet where you will deploy the Private Endpoint.
      
    * Example: `vnet-mysql-pe-lab` with address space `10.0.0.0/16`.

* **Dedicated Subnet for Private Endpoint**: A subnet within your VNet specifically for Private Endpoints.
      
    * Example: `snet-privateendpoints` within `vnet-mysql-pe-lab`, with address space `10.0.1.0/24`.
      
    * **Important**: This subnet must have `privateEndpointNetworkPolicies` disabled.

* **Virtual Machine (VM) in the same VNet**: An Azure VM (e.g., Windows or Linux) deployed in *another subnet* within the same VNet as your Private Endpoint (e.g., `snet-vm` with `10.0.0.0/24`). This VM will be used to test connectivity via the private endpoint.
  
* **MySQL Client Tool**: Such as [MySQL Workbench](https://www.mysql.com/products/workbench/) or the `mysql` command-line client installed on your VM to interact with your Azure MySQL server.

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

-----

## Part 2: Lab Setup: Prepare Your Environment

Before configuring the Private Endpoint, let's ensure your network and MySQL server are ready.

### 2.1 Verify or Create Azure Virtual Network and Subnets

You need a VNet and at least two subnets: one for your VM (if it's not already in its own subnet) and a *dedicated* one for the Private Endpoint.

#### 2.1.1 Using Azure Portal

1.  **Sign in to the [Azure Portal](https://portal.azure.com/)**.
2.  Search for and select **"Virtual networks"**.
3.  **Verify/Create VNet**: Check if you have a VNet (e.g., `vnet-mysql-pe-lab`). If not:
      * Click **+ Create**.
      * **Basics**: Select your `Subscription`, `Resource group` (create new: `rg-mysql-pe-lab`), `Virtual network name` (e.g., `vnet-mysql-pe-lab`), `Region`.
      * **IP Addresses**: Leave default `10.0.0.0/16` for now.
      * **Subnets**: Keep the `default` subnet as `10.0.0.0/24`. We'll add another.
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
      * Create a new VM, selecting your `vnet-mysql-pe-lab` and a different subnet (e.g., `default`). Ensure it has inbound RDP/SSH rules for your public IP for management.

#### 2.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-mysql-pe-lab"
LOCATION="<YourAzureRegion>" # e.g., eastus
VNET_NAME="vnet-mysql-pe-lab"
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
$resourceGroupName = "rg-mysql-pe-lab"
$location = "<YourAzureRegion>" # e.g., EastUS
$vnetName = "vnet-mysql-pe-lab"
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

### 2.2 Verify Azure Database for MySQL - Flexible Server

Ensure you have your MySQL Flexible Server ready and note its FQDN and admin credentials.

1.  **Azure Portal**: Search for and select **"Azure Database for MySQL flexible servers"**.
2.  Click on your server (e.g., `mymysqlflexserver-youruniqueid`).
3.  Note down the **Server name** (FQDN, e.g., `mymysqlflexserver-youruniqueid.mysql.database.azure.com`) and your **Admin username**.

### 2.3 Deploy a Test VM (if not already done)

Deploy a Windows or Linux VM into the `snet-vm` subnet of your `vnet-mysql-pe-lab`. This VM will be used for testing private connectivity. Ensure you can RDP/SSH into it.

-----

## Part 3: Configure Azure Private Endpoint for MySQL

Now, let's create the Private Endpoint and configure its DNS integration.

### 3.1 Create the Private Endpoint

#### 3.1.1 Using Azure Portal (Recommended for Visual Workflow)

1.  **Navigate to your Azure Database for MySQL - Flexible Server** in the Azure Portal.
2.  On the left-hand menu, under "Settings," select **"Networking"**.
3.  Go to the **"Private endpoint connections"** tab.
4.  Click **"+ Create a private endpoint"**.
5.  **Basics Tab**:
      * **Subscription**: Select your subscription.
      * **Resource group**: Choose `rg-mysql-pe-lab` (or your MySQL server's resource group if different).
      * **Name**: Enter a descriptive name (e.g., `pe-mymysqlflexserver`).
      * **Region**: Select the **same region as your Virtual Network** (`vnet-mysql-pe-lab`).
      * Click **"Next: Resource \>"**.
6.  **Resource Tab**:
      * **Connection method**: `Connect to an Azure resource in my directory`
      * **Subscription**: Select the subscription where your MySQL server resides.
      * **Resource type**: `Microsoft.DBforMySQL/flexibleServers`
      * **Resource**: Select your `mymysqlflexserver-youruniqueid` from the dropdown.
      * **Target sub-resource**: `mysqlServer` (this is the only option for MySQL Flexible Server).
      * Click **"Next: Network \>"**.
7.  **Network Tab**:
      * **Virtual network**: Select `vnet-mysql-pe-lab`.
      * **Subnet**: Select `snet-privateendpoints`.
      * **Private IP configuration**: Keep default `Dynamically allocate IP address`.
      * **Integrate with private DNS zone**:
          * **YES (Recommended)**: This is crucial for name resolution.
          * **Subscription**: Select your subscription.
          * **Resource group**: Choose `rg-mysql-pe-lab` (or the one you prefer for DNS zones).
          * **Private DNS Zone**:
              * **Create new**: It will automatically suggest `privatelink.mysql.database.azure.com`. Accept this.
      * Click **"Next: Tags \>"** (Optional).
8.  **Tags Tab (Optional)**: Add any relevant tags.
9.  **Review + Create Tab**:
      * Review all settings.
      * Click **"Create"**.
10. **Verify Deployment**: Once created, navigate back to your MySQL server's "Networking" blade. You should see `pe-mymysqlflexserver` listed under "Private endpoint connections" with a `Status` of `Approved`.

#### 3.1.2 Using Azure CLI

```bash
# Define variables (ensure these match your environment)
RESOURCE_GROUP="rg-mysql-pe-lab"
LOCATION="<YourAzureRegion>" # e.g., eastus
VNET_NAME="vnet-mysql-pe-lab"
PE_SUBNET_NAME="snet-privateendpoints"
MYSQL_SERVER_NAME="mymysqlflexserver-youruniqueid" # Replace with your MySQL server name
PRIVATE_ENDPOINT_NAME="pe-${MYSQL_SERVER_NAME}"
PRIVATE_DNS_ZONE_NAME="privatelink.mysql.database.azure.com"

# Get MySQL Server Resource ID
echo "Retrieving MySQL Server ID..."
MYSQL_SERVER_ID=$(az mysql flexible-server show \
    --resource-group $RESOURCE_GROUP \
    --name $MYSQL_SERVER_NAME \
    --query id -o tsv)
if [ -z "$MYSQL_SERVER_ID" ]; then
    echo "Error: MySQL server '$MYSQL_SERVER_NAME' not found in '$RESOURCE_GROUP'."
    exit 1
fi
echo "MySQL Server ID: $MYSQL_SERVER_ID"

# Create Private Endpoint
echo "Creating Private Endpoint: $PRIVATE_ENDPOINT_NAME"
az network private-endpoint create \
  --name $PRIVATE_ENDPOINT_NAME \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --subnet $PE_SUBNET_NAME \
  --connection-name "${MYSQL_SERVER_NAME}-private-link-connection" \
  --private-connection-resource-id "$MYSQL_SERVER_ID" \
  --group-id "mysqlServer" \
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

# Get the FQDN of the MySQL Flexible Server
MYSQL_FQDN="${MYSQL_SERVER_NAME}.mysql.database.azure.com"

echo "Creating A record in Private DNS Zone for $MYSQL_FQDN to $PRIVATE_IP"
az network private-dns record-set a add-record \
  --resource-group $RESOURCE_GROUP \
  --zone-name $PRIVATE_DNS_ZONE_NAME \
  --record-set-name $MYSQL_SERVER_NAME \
  --ipv4-address $PRIVATE_IP \
  --output tsv
```

#### 3.1.3 Using Azure PowerShell

```powershell
# Define variables (ensure these match your environment)
$resourceGroupName = "rg-mysql-pe-lab"
$location = "<YourAzureRegion>" # e.g., EastUS
$vnetName = "vnet-mysql-pe-lab"
$peSubnetName = "snet-privateendpoints"
$mysqlServerName = "mymysqlflexserver-youruniqueid" # Replace with your MySQL server name
$privateEndpointName = "pe-$mysqlServerName"
$privateDnsZoneName = "privatelink.mysql.database.azure.com"

# Get existing VNet and Subnet
Write-Host "Retrieving VNet '$vnetName' and Subnet '$peSubnetName'..."
$vnet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName
$subnet = Get-AzVirtualNetworkSubnetConfig -Name $peSubnetName -VirtualNetwork $vnet
if (-not $subnet) {
    Write-Error "Subnet '$peSubnetName' not found in VNet '$vnetName'. Ensure it exists and network policies are disabled."
    exit 1
}

# Get MySQL Server resource ID
Write-Host "Retrieving MySQL Server ID for '$mysqlServerName'..."
$mysqlServer = Get-AzMySqlFlexibleServer -ResourceGroupName $resourceGroupName -Name $mysqlServerName
if (-not $mysqlServer) {
    Write-Error "MySQL Flexible Server '$mysqlServerName' not found."
    exit 1
}
$mysqlResourceId = $mysqlServer.Id

# Create Private Link Service Connection
Write-Host "Creating Private Link Service Connection..."
$privateLinkServiceConnection = New-AzPrivateLinkServiceConnection -Name "$mysqlServerName-private-link-connection" `
    -PrivateLinkServiceId $mysqlResourceId `
    -GroupId "mysqlServer"

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
Add-AzVirtualNetworkPeering -Name "${vnetName}-link" -VirtualNetwork $vnet -RemoteVirtualNetworkId $privateDnsZone.Id
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

$mysqlFqdnPrefix = $mysqlServerName # The part before .mysql.database.azure.com

Write-Host "Creating A record in Private DNS Zone for '$mysqlFqdnPrefix' to '$privateIp'"
Set-AzPrivateDnsRecordSet -ResourceGroupName $resourceGroupName `
    -ZoneName $privateDnsZoneName `
    -Name $mysqlFqdnPrefix `
    -RecordType A `
    -Ttl 3600 `
    -PrivateDnsRecords (New-AzPrivateDnsRecordConfig -IPv4Address $privateIp) -Overwrite
```

-----

## Part 4: Restrict Public Network Access on Azure MySQL Flexible Server

This is a critical security step. Once the Private Endpoint is active and DNS is configured, you should disable public access to your MySQL server.

### 4.1 Using Azure Portal

1.  **Navigate to your Azure Database for MySQL - Flexible Server**.
2.  On the left-hand menu, under "Settings," select **"Networking"**.
3.  Under **"Public network access"**:
      * Select **"Disabled"**.
      * **Crucial Note**: Ensure that "Allow Azure services on the trusted services list to access this server" is checked if you have other Azure services that need to connect via service endpoints (e.g., Azure App Service, Azure Functions *without* VNet integration). For private endpoints, this is often not needed, but review your architecture.
4.  Click **"Save"**.
5.  Confirm the change. It may take a few minutes for the setting to apply.

### 4.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-mysql-pe-lab" # Or your MySQL server's resource group
MYSQL_SERVER_NAME="mymysqlflexserver-youruniqueid" # Replace with your MySQL server name

echo "Updating public network access for MySQL server to Disabled..."
az mysql flexible-server update \
  --resource-group $RESOURCE_GROUP \
  --name $MYSQL_SERVER_NAME \
  --public-network-access Disabled
echo "Public network access disabled. This may take a few minutes to take effect."
```

### 4.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-mysql-pe-lab" # Or your MySQL server's resource group
$mysqlServerName = "mymysqlflexserver-youruniqueid" # Replace with your MySQL server name

Write-Host "Updating public network access for MySQL server to Disabled..."
Update-AzMySqlFlexibleServer -ResourceGroupName $resourceGroupName `
    -Name $mysqlServerName `
    -PublicNetworkAccess "Disabled"
Write-Host "Public network access disabled. This may take a few minutes to take effect."
```

-----

## Part 5: Testing and Validation

Now, let's confirm that your MySQL server is only accessible via the Private Endpoint from within your VNet.

### 5.1 Test Connectivity from Within the VNet (VM)

1.  **Connect to your Test VM** (e.g., via RDP or SSH).
2.  **Open a command prompt or terminal**.
3.  **Test DNS Resolution**:
      * Run `nslookup mymysqlflexserver-youruniqueid.mysql.database.azure.com` (replace with your server's FQDN).
      * **Expected Result**: The output should show the **private IP address** assigned to your Private Endpoint (e.g., `10.0.1.x`) and *not* a public IP address.
4.  **Test MySQL Connectivity**:
      * Use your MySQL client tool (MySQL Workbench or `mysql` command-line).
      * **Host/Hostname**: `mymysqlflexserver-youruniqueid.mysql.database.azure.com` (the FQDN, *not* the private IP).
      * **Port**: `3306`
      * **Username**: Your MySQL admin username (e.g., `myadmin`).
      * **Password**: Your MySQL admin password.
      * **Expected Result**: You should successfully connect to the database. Try running a simple query (e.g., `SELECT VERSION();` or `USE mylocaldb; SELECT * FROM products;` if you migrated data in the previous lab).

### 5.2 Test Connectivity from Outside the VNet (Public Internet)

1.  **From your local machine** (which is *not* connected to your Azure VNet via VPN/ExpressRoute), try to connect to your Azure Database for MySQL Flexible Server using MySQL Workbench or the `mysql` client.
      * **Host/Hostname**: `mymysqlflexserver-youruniqueid.mysql.database.azure.com` (the FQDN).
      * **Port**: `3306`
      * **Expected Result**: The connection attempt should **fail** (e.g., timeout, connection refused, or host not found depending on DNS propagation and firewall rules). This confirms that public access has been successfully blocked.

-----

## Part 6: Troubleshooting

  * **DNS Resolution Fails or Returns Public IP**:
      * Ensure the Private DNS Zone `privatelink.mysql.database.azure.com` exists in your resource group.
      * Verify the Private DNS Zone is linked to your `vnet-mysql-pe-lab`.
      * Check for an A record in the Private DNS Zone matching your MySQL server's FQDN (e.g., `mymysqlflexserver-youruniqueid` pointing to the private IP of the Private Endpoint).
      * Ensure the VM is configured to use Azure's default DNS servers (or your custom DNS servers if they forward to Azure DNS). Avoid hardcoding public DNS servers on the VM's network adapter.
  * **Connection Fails from VM (even after correct DNS)**:
      * Verify that `privateEndpointNetworkPolicies` are `Disabled` on the `snet-privateendpoints` subnet.
      * Check Network Security Group (NSG) rules on *both* the `snet-vm` (outbound to 10.0.1.x:3306) and `snet-privateendpoints` (inbound from 10.0.0.x:any) subnets. For private endpoints, typically, the private endpoint subnet should have no NSG applied, or an NSG allowing all inbound/outbound on its private IP.
      * Ensure the MySQL server's "Public network access" is set to "Disabled" (after verifying internal connectivity).
  * **Private Endpoint Status is Not "Approved"**:
      * Go to your MySQL server -\> Networking -\> Private endpoint connections. If the status is "Pending" or "Rejected," you might need to approve it manually (though it's usually automatic when created from the same subscription).

-----

## Part 7: Cleanup Resources

To avoid ongoing costs, it's crucial to delete the resources created in this lab.

**(Using Azure CLI - Recommended for comprehensive cleanup)**

```bash
# Define variables
RESOURCE_GROUP="rg-mysql-pe-lab" # The resource group containing your VNet, PE, and DNS Zone

# Delete the resource group and all contained resources
echo "Deleting resource group: $RESOURCE_GROUP and all its contents..."
az group delete --name $RESOURCE_GROUP --yes --no-wait
echo "Deletion process initiated. It may take some time to complete."

# Optional: If your MySQL Flexible Server is in a different resource group and you want to delete it:
# MYSQL_SERVER_RESOURCE_GROUP="<YourMySQLServerResourceGroup>" # e.g., my-mysql-lab-rg
# MYSQL_SERVER_NAME="mymysqlflexserver-youruniqueid"
# echo "Deleting MySQL Flexible Server: $MYSQL_SERVER_NAME in $MYSQL_SERVER_RESOURCE_GROUP..."
# az mysql flexible-server delete --resource-group $MYSQL_SERVER_RESOURCE_GROUP --name $MYSQL_SERVER_NAME --yes --no-wait
```

**(Using Azure Portal)**

1.  **Delete the Private Endpoint**:
      * Navigate to your Azure Database for MySQL - Flexible Server.
      * Go to **"Networking"** -\> **"Private endpoint connections"**.
      * Select `pe-mymysqlflexserver` and click **"Delete"**.
2.  **Delete the Private DNS Zone**:
      * Search for **"Private DNS zones"**.
      * Select `privatelink.mysql.database.azure.com`.
      * Click **"Delete"**.
3.  **Delete the Virtual Network**:
      * Search for **"Virtual networks"**.
      * Select `vnet-mysql-pe-lab`.
      * Click **"Delete"**. This will also delete the subnets and the VM within it if the VM was deployed *after* the VNet.
4.  **Delete the Resource Group (Recommended)**:
      * The easiest way to clean up all associated resources is to delete the resource group.
      * Search for **"Resource groups"**.
      * Select `rg-mysql-pe-lab`.
      * Click **"Delete resource group"**, type the resource group name to confirm, and click **"Delete"**.
      * **Warning**: This will delete *all* resources within that resource group, including your VM, VNet, Private Endpoint, and Private DNS Zone. If your MySQL Flexible Server is in *this* resource group, it will also be deleted.

-----

Congratulations\! You have successfully configured and tested Azure Private Endpoints for your Azure Database for MySQL - Flexible Server, dramatically improving its security posture by allowing private network access while blocking public exposure. 

You also gained experience configuring resources using Azure Portal, Azure CLI, and Azure PowerShell.