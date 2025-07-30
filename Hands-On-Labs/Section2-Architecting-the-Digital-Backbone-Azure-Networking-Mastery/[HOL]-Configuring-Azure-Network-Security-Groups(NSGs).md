# Lab: Configuring Azure Network Security Groups (NSGs) 🛡️

This lab will guide you through creating and configuring Azure Network Security Groups (NSGs) to control network traffic flow within your Azure Virtual Network (VNet) and to/from the internet. 

NSGs are a fundamental component of network security in Azure, allowing you to filter traffic at a granular level. 

We will demonstrate the configuration using Azure Portal, Azure CLI, and Azure PowerShell for comprehensive understanding.

## Lab Scenario 🧪

You have a multi-tier application deployed across different subnets within an Azure VNet. To enhance security, you need to implement network access control using NSGs to ensure that only authorized traffic flows between application tiers and to/from the internet. Specifically:

* **Web Tier (snet-web)**: Should allow inbound HTTP/HTTPS from the internet and outbound traffic to the application tier.

* **Application Tier (snet-app)**: Should only allow inbound traffic from the web tier on a specific port and outbound traffic to the database tier. It should block direct inbound traffic from the internet.

Your goal is to:

* Create two Network Security Groups.

* Associate these NSGs with their respective subnets.

* Configure inbound and outbound security rules to enforce the desired traffic flow.

* Verify the rules are working as expected using test VMs.

This lab will demonstrate the core steps of NSG creation, rule definition, and association.

## Part 1: Prerequisites

* An active Azure subscription.

* **Existing Azure Virtual Network and Subnets**:
    
    * Ideally, the `vnet-app-prod` VNet with `snet-web` (10.0.0.0/24) and `snet-app` (10.0.2.0/24) subnets created from the "Creating Azure Virtual Networks (VNets) and Subnets" lab.

* **Two Azure Virtual Machines (VMs)**:
    
    * `vm-web`: Deployed into `snet-web`. (e.g., a simple Windows Server or Linux VM).
    
    * `vm-app`: Deployed into `snet-app`. (e.g., a simple Windows Server or Linux VM).
    
    * **Note**: Ensure these VMs have public IPs for initial RDP/SSH access during setup. We will restrict this later.

* **RDP/SSH client**: To connect to your test VMs.

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

-----

## Part 2: Architecture Diagram (Conceptual)

Below is a conceptual diagram of the network architecture we will secure in this lab.

```
+------------------------------------------------------------------+
|               Azure Virtual Network (vnet-app-prod)              |
|               Address Space: 10.0.0.0/16                         |
|                                                                  |
|  +-------------------+              +-------------------+        |
|  |   Subnet: snet-web|              |   Subnet: snet-app|        |
|  |   10.0.0.0/24     |              |   10.0.2.0/24     |        |
|  |                   |              |                   |        |
|  |  +------------+   |              |  +------------+   |        |
|  |  |   vm-web   |<------------------->|   vm-app   |   |        |
|  |  +------------+   |              |  +------------+   |        |
|  +--------^----------+              +---------^---------+        |
|           |                                    |                   |
|           |                                    |                   |
| +---------|---------+                    +----|----+              |
| |  Internet Traffic |                    |   NSG   |              |
| | (HTTP/HTTPS)      |                    | nsG-app |              |
| +-------------------+                    +---------+              |
|           |   ^                                ^                   |
|           |   |                                |                   |
|           v   | Allow 80,443 Inbound         | Allow 8080 Inbound |
| +---------|---|---------+                  +--|--+                |
| |       NSG: nsg-web    | <------------------| (from snet-web)    |
| +-----------------------+                  |   |                  |
|                                            |   |                  |
+------------------------------------------------------------------+
```

-----

## Part 3: Lab Setup: Prepare Your Environment

### 3.1 Verify Existing VNet and Subnets

Ensure you have the VNet (`vnet-app-prod`) and the following subnets from the previous lab:

  * `snet-web` (10.0.0.0/24)
  * `snet-app` (10.0.2.0/24)

If not, please refer to the "Creating Azure Virtual Networks (VNets) and Subnets" lab document to create them first.

### 3.2 Deploy Test Virtual Machines

Deploy two simple Windows or Linux VMs, ensuring they have public IPs for initial management.

#### 3.2.1 Using Azure Portal

1.  **Deploy `vm-web`**:
      * In the Azure Portal, search for `Virtual machines` and click **+ Create**.
      * **Basics**:
          * **Subscription**: Your subscription.
          * **Resource group**: `rg-network-lab` (or your existing VNet's RG).
          * **Virtual machine name**: `vm-web`.
          * **Region**: Same as your VNet.
          * **Image**: Choose a simple image (e.g., Windows Server 2019 Datacenter or Ubuntu Server).
          * **Size**: A small size (e.g., `Standard_B2s`).
          * **Administrator account**: Create a username and password.
          * **Public inbound ports**: `Allow selected ports`.
          * **Select inbound ports**: Check `RDP` (3389) or `SSH` (22) for initial management.
      * **Disks**: Leave defaults.
      * **Networking**:
          * **Virtual network**: Select `vnet-app-prod`.
          * **Subnet**: Select `snet-web`.
          * **Public IP**: `(new)` - accept default name.
          * **NIC network security group**: `Basic` (we will replace this with our custom NSG later).
      * Click **Review + create**, then **Create**.
2.  **Deploy `vm-app`**:
      * Repeat the steps above, but with:
          * **Virtual machine name**: `vm-app`.
          * **Subnet**: Select `snet-app`.
          * Ensure `RDP` (3389) or `SSH` (22) is allowed for initial inbound access.

#### 3.2.2 Using Azure CLI

```bash
# Define variables (ensure these match your environment)
RESOURCE_GROUP="rg-network-lab"
LOCATION="<YourAzureRegion>"
VNET_NAME="vnet-app-prod"
WEB_SUBNET_NAME="snet-web"
APP_SUBNET_NAME="snet-app"

# Deploy vm-web
echo "Deploying vm-web in $WEB_SUBNET_NAME..."
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name "vm-web" \
  --image "Win2019Datacenter" \
  --vnet-name $VNET_NAME \
  --subnet $WEB_SUBNET_NAME \
  --admin-username "azureuser" \
  --admin-password "StrongP@ssw0rd!" \
  --public-ip-sku Standard \
  --nsg-rule "RDP" \
  --location $LOCATION \
  --no-wait

# Deploy vm-app
echo "Deploying vm-app in $APP_SUBNET_NAME..."
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name "vm-app" \
  --image "Ubuntu2204" \
  --vnet-name $VNET_NAME \
  --subnet $APP_SUBNET_NAME \
  --admin-username "azureuser" \
  --admin-password "StrongP@ssw0rd!" \
  --public-ip-sku Standard \
  --nsg-rule "SSH" \
  --location $LOCATION \
  --no-wait

echo "VM deployments started. This may take a few minutes. Check status with: az vm list -d -g $RESOURCE_GROUP -o table"
```

#### 3.2.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$location = "<YourAzureRegion>"
$vnetName = "vnet-app-prod"
$webSubnetName = "snet-web"
$appSubnetName = "snet-app"
$vmAdminUsername = "azureuser"
$vmAdminPassword = "StrongP@ssw0rd!"

# Get VNet and Subnets
$vnet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName
$webSubnet = (Get-AzVirtualNetworkSubnetConfig -Name $webSubnetName -VirtualNetwork $vnet).Id
$appSubnet = (Get-AzVirtualNetworkSubnetConfig -Name $appSubnetName -VirtualNetwork $vnet).Id

# Create Public IP for vm-web
$publicIpWeb = New-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name "vm-web-ip" -Location $location -AllocationMethod Static -Sku Standard

# Create Network Interface for vm-web
$nicWeb = New-AzNetworkInterface -ResourceGroupName $resourceGroupName -Name "vm-web-nic" -Location $location -SubnetId $webSubnet -PublicIpAddress $publicIpWeb

# Create vm-web
Write-Host "Creating vm-web in $webSubnetName..."
New-AzVM -ResourceGroupName $resourceGroupName -Name "vm-web" -Location $location -VMSize "Standard_B2s" `
    -AuthenticationType "Password" -AdminUsername $vmAdminUsername -AdminPassword $vmAdminPassword `
    -Image "Win2019Datacenter" -NetworkInterface $nicWeb `
    -OpenPorts 3389 # Allow RDP initially

# Create Public IP for vm-app
$publicIpApp = New-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name "vm-app-ip" -Location $location -AllocationMethod Static -Sku Standard

# Create Network Interface for vm-app
$nicApp = New-AzNetworkInterface -ResourceGroupName $resourceGroupName -Name "vm-app-nic" -Location $location -SubnetId $appSubnet -PublicIpAddress $publicIpApp

# Create vm-app
Write-Host "Creating vm-app in $appSubnetName..."
New-AzVM -ResourceGroupName $resourceGroupName -Name "vm-app" -Location $location -VMSize "Standard_B2s" `
    -AuthenticationType "Password" -AdminUsername $vmAdminUsername -AdminPassword $vmAdminPassword `
    -Image "UbuntuServer" -NetworkInterface $nicApp `
    -OpenPorts 22 # Allow SSH initially

Write-Host "VM deployments started. This may take a few minutes."
```

-----

## Part 4: Step-by-Step Configuration: Creating and Configuring NSGs

### 4.1 Create Network Security Groups (NSGs)

We will create two NSGs: `nsg-web` and `nsg-app`.

#### 4.1.1 Using Azure Portal

1.  In the Azure Portal search bar, type `Network security groups` and select it.
2.  Click **+ Create**.
3.  **Create Network security group 1 (`nsg-web`)**:
      * **Subscription**: Your subscription.
      * **Resource group**: `rg-network-lab`.
      * **Name**: `nsg-web`.
      * **Region**: Same as your VNet.
      * Click **Review + create**, then **Create**.
4.  **Create Network security group 2 (`nsg-app`)**:
      * Repeat the steps above, but with **Name**: `nsg-app`.

#### 4.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
LOCATION="<YourAzureRegion>"

echo "Creating Network Security Group: nsg-web"
az network nsg create \
  --resource-group $RESOURCE_GROUP \
  --name "nsg-web" \
  --location $LOCATION \
  --output tsv

echo "Creating Network Security Group: nsg-app"
az network nsg create \
  --resource-group $RESOURCE_GROUP \
  --name "nsg-app" \
  --location $LOCATION \
  --output tsv
```

#### 4.1.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$location = "<YourAzureRegion>"

Write-Host "Creating Network Security Group: nsg-web"
New-AzNetworkSecurityGroup -Name "nsg-web" -ResourceGroupName $resourceGroupName -Location $location

Write-Host "Creating Network Security Group: nsg-app"
New-AzNetworkSecurityGroup -Name "nsg-app" -ResourceGroupName $resourceGroupName -Location $location
```

### 4.2 Associate NSGs with Subnets

Now, we will associate the created NSGs with their respective subnets.

#### 4.2.1 Using Azure Portal

1.  Navigate to your Virtual Network: `vnet-app-prod`.
2.  On the left-hand menu, select **"Subnets"**.
3.  Click on the `snet-web` subnet.
      * Under **"Network security group"**, select `nsg-web` from the dropdown.
      * Click **Save**.
4.  Click on the `snet-app` subnet.
      * Under **"Network security group"**, select `nsg-app` from the dropdown.
      * Click **Save**.

#### 4.2.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
VNET_NAME="vnet-app-prod"
WEB_SUBNET_NAME="snet-web"
APP_SUBNET_NAME="snet-app"
NSG_WEB_NAME="nsg-web"
NSG_APP_NAME="nsg-app"

echo "Associating $NSG_WEB_NAME with $WEB_SUBNET_NAME"
az network vnet subnet update \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --name $WEB_SUBNET_NAME \
  --network-security-group $NSG_WEB_NAME \
  --output tsv

echo "Associating $NSG_APP_NAME with $APP_SUBNET_NAME"
az network vnet subnet update \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --name $APP_SUBNET_NAME \
  --network-security-group $NSG_APP_NAME \
  --output tsv
```

#### 4.2.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$vnetName = "vnet-app-prod"
$webSubnetName = "snet-web"
$appSubnetName = "snet-app"
$nsgWebName = "nsg-web"
$nsgAppName = "nsg-app"

# Get VNet and NSGs
$vnet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName
$nsgWeb = Get-AzNetworkSecurityGroup -Name $nsgWebName -ResourceGroupName $resourceGroupName
$nsgApp = Get-AzNetworkSecurityGroup -Name $nsgAppName -ResourceGroupName $resourceGroupName

# Associate nsg-web with snet-web
Write-Host "Associating $nsgWebName with $webSubnetName"
$webSubnet = Get-AzVirtualNetworkSubnetConfig -Name $webSubnetName -VirtualNetwork $vnet
Set-AzVirtualNetworkSubnetConfig -Name $webSubnet.Name -VirtualNetwork $vnet -AddressPrefix $webSubnet.AddressPrefix -NetworkSecurityGroup $nsgWeb | Out-Null
Set-AzVirtualNetwork -VirtualNetwork $vnet

# Associate nsg-app with snet-app
Write-Host "Associating $nsgAppName with $appSubnetName"
$appSubnet = Get-AzVirtualNetworkSubnetConfig -Name $appSubnetName -VirtualNetwork $vnet
Set-AzVirtualNetworkSubnetConfig -Name $appSubnet.Name -VirtualNetwork $vnet -AddressPrefix $appSubnet.AddressPrefix -NetworkSecurityGroup $nsgApp | Out-Null
Set-AzVirtualNetwork -VirtualNetwork $vnet
```

### 4.3 Configure Inbound and Outbound Security Rules for `nsg-web`

We will configure rules for the web tier. Default rules exist but often need to be overridden or supplemented. NSG rules are processed by priority (lower number = higher priority).

#### 4.3.1 Using Azure Portal

1.  Navigate to `nsg-web`.

2.  On the left-hand menu, select **"Inbound security rules"**.

3.  **Delete** the default `AllowRDP` or `AllowSSH` rule if it exists (or just ensure its priority is higher than our explicit denies later).

4.  Click **+ Add** for each rule:

      * **Allow HTTP (Port 80) from Internet**:

          * **Source**: `Any`
          * **Source port ranges**: `*`
          * **Destination**: `Any`
          * **Destination port ranges**: `80`
          * **Protocol**: `TCP`
          * **Action**: `Allow`
          * **Priority**: `100`
          * **Name**: `Allow_HTTP_Inbound`
          * Click **Add**.

      * **Allow HTTPS (Port 443) from Internet**:

          * **Source**: `Any`
          * **Source port ranges**: `*`
          * **Destination**: `Any`
          * **Destination port ranges**: `443`
          * **Protocol**: `TCP`
          * **Action**: `Allow`
          * **Priority**: `110`
          * **Name**: `Allow_HTTPS_Inbound`
          * Click **Add**.

      * **Allow RDP/SSH (Management)** (Temporarily from your Public IP):

          * **Source**: `IP Addresses` or `My IP address`
          * **Source IP addresses/CIDR ranges**: `<YourPublicIPAddress>`
          * **Destination**: `Any`
          * **Destination port ranges**: `3389` (RDP) or `22` (SSH)
          * **Protocol**: `TCP`
          * **Action**: `Allow`
          * **Priority**: `90` (Higher priority than HTTP/HTTPS if needed)
          * **Name**: `Allow_Management_IP`
          * Click **Add**.

5.  On the left-hand menu, select **"Outbound security rules"**.

6.  Click **+ Add** for each rule:

      * **Allow Outbound to Application Tier (Port 8080 - Custom App Port)**:

          * **Source**: `Any`
          * **Source port ranges**: `*`
          * **Destination**: `Virtual Network` (This refers to any IP within your VNet)
          * **Destination port ranges**: `8080`
          * **Protocol**: `TCP`
          * **Action**: `Allow`
          * **Priority**: `100`
          * **Name**: `Allow_Outbound_to_AppTier`
          * Click **Add**.

      * **Allow Outbound to Internet**:

          * **Source**: `Any`
          * **Source port ranges**: `*`
          * **Destination**: `Internet`
          * **Destination port ranges**: `*`
          * **Protocol**: `Any`
          * **Action**: `Allow`
          * **Priority**: `120` (Lower priority than specific app tier rule)
          * **Name**: `Allow_Outbound_Internet`
          * Click **Add**.

#### 4.3.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
NSG_WEB_NAME="nsg-web"

echo "Configuring Inbound rules for $NSG_WEB_NAME"
# Allow RDP/SSH from your public IP (replace <YOUR_PUBLIC_IP>)
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_WEB_NAME \
  --name "Allow-Management-IP" \
  --priority 90 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "<YOUR_PUBLIC_IP_ADDRESS>/32" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges "3389" "22" \
  --description "Allow RDP/SSH from my public IP" \
  --output tsv

# Allow HTTP from Internet
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_WEB_NAME \
  --name "Allow-HTTP-Inbound" \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges "80" \
  --description "Allow HTTP from Internet" \
  --output tsv

# Allow HTTPS from Internet
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_WEB_NAME \
  --name "Allow-HTTPS-Inbound" \
  --priority 110 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges "443" \
  --description "Allow HTTPS from Internet" \
  --output tsv

echo "Configuring Outbound rules for $NSG_WEB_NAME"
# Allow Outbound to Application Tier (port 8080) - assuming snet-app is 10.0.2.0/24
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_WEB_NAME \
  --name "Allow-Outbound-to-AppTier" \
  --priority 100 \
  --direction Outbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes "10.0.2.0/24" \
  --destination-port-ranges "8080" \
  --description "Allow outbound traffic to app subnet on port 8080" \
  --output tsv

# Allow Outbound to Internet
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_WEB_NAME \
  --name "Allow-Outbound-Internet" \
  --priority 120 \
  --direction Outbound \
  --access Allow \
  --protocol "*" \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes "Internet" \
  --destination-port-ranges "*" \
  --description "Allow all outbound to Internet" \
  --output tsv
```

#### 4.3.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$nsgWebName = "nsg-web"
$yourPublicIp = "<YOUR_PUBLIC_IP_ADDRESS>/32" # Replace with your current public IP
$appSubnetCIDR = "10.0.2.0/24"

# Get the NSG object
$nsgWeb = Get-AzNetworkSecurityGroup -Name $nsgWebName -ResourceGroupName $resourceGroupName

Write-Host "Configuring Inbound rules for $nsgWebName"
# Allow RDP/SSH from your public IP
$nsgWeb | Add-AzNetworkSecurityRuleConfig -Name "Allow-Management-IP" `
    -Description "Allow RDP/SSH from my public IP" `
    -Direction Inbound -Priority 90 -Access Allow -Protocol Tcp `
    -SourceAddressPrefix $yourPublicIp -SourcePortRange "*" `
    -DestinationAddressPrefix "*" -DestinationPortRange "3389", "22" | Out-Null

# Allow HTTP from Internet
$nsgWeb | Add-AzNetworkSecurityRuleConfig -Name "Allow-HTTP-Inbound" `
    -Description "Allow HTTP from Internet" `
    -Direction Inbound -Priority 100 -Access Allow -Protocol Tcp `
    -SourceAddressPrefix "*" -SourcePortRange "*" `
    -DestinationAddressPrefix "*" -DestinationPortRange "80" | Out-Null

# Allow HTTPS from Internet
$nsgWeb | Add-AzNetworkSecurityRuleConfig -Name "Allow-HTTPS-Inbound" `
    -Description "Allow HTTPS from Internet" `
    -Direction Inbound -Priority 110 -Access Allow -Protocol Tcp `
    -SourceAddressPrefix "*" -SourcePortRange "*" `
    -DestinationAddressPrefix "*" -DestinationPortRange "443" | Out-Null

Write-Host "Configuring Outbound rules for $nsgWebName"
# Allow Outbound to Application Tier (port 8080)
$nsgWeb | Add-AzNetworkSecurityRuleConfig -Name "Allow-Outbound-to-AppTier" `
    -Description "Allow outbound traffic to app subnet on port 8080" `
    -Direction Outbound -Priority 100 -Access Allow -Protocol Tcp `
    -SourceAddressPrefix "*" -SourcePortRange "*" `
    -DestinationAddressPrefix $appSubnetCIDR -DestinationPortRange "8080" | Out-Null

# Allow Outbound to Internet
$nsgWeb | Add-AzNetworkSecurityRuleConfig -Name "Allow-Outbound-Internet" `
    -Description "Allow all outbound to Internet" `
    -Direction Outbound -Priority 120 -Access Allow -Protocol "*" `
    -SourceAddressPrefix "*" -SourcePortRange "*" `
    -DestinationAddressPrefix "Internet" -DestinationPortRange "*" | Out-Null

# Save the NSG configuration
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsgWeb
```

### 4.4 Configure Inbound and Outbound Security Rules for `nsg-app`

We will configure rules for the application tier.

#### 4.4.1 Using Azure Portal

1.  Navigate to `nsg-app`.

2.  On the left-hand menu, select **"Inbound security rules"**.

3.  **Delete** the default `AllowRDP` or `AllowSSH` rule if it exists.

4.  Click **+ Add** for each rule:

      * **Allow RDP/SSH (Management)** (Temporarily from your Public IP):

          * **Source**: `IP Addresses` or `My IP address`
          * **Source IP addresses/CIDR ranges**: `<YourPublicIPAddress>/32`
          * **Destination**: `Any`
          * **Destination port ranges**: `3389` (RDP) or `22` (SSH)
          * **Protocol**: `TCP`
          * **Action**: `Allow`
          * **Priority**: `90`
          * **Name**: `Allow_Management_IP`
          * Click **Add**.

      * **Allow Inbound from Web Tier (Port 8080 - Custom App Port)**:

          * **Source**: `IP Addresses`
          * **Source IP addresses/CIDR ranges**: `10.0.0.0/24` (This is the `snet-web` range)
          * **Destination**: `Any`
          * **Destination port ranges**: `8080`
          * **Protocol**: `TCP`
          * **Action**: `Allow`
          * **Priority**: `100`
          * **Name**: `Allow_Inbound_from_WebTier`
          * Click **Add**.

5.  On the left-hand menu, select **"Outbound security rules"**.

6.  Click **+ Add** for each rule:

      * **Allow Outbound to Database Tier (Port 3306 - MySQL)**:

          * **Source**: `Any`
          * **Source port ranges**: `*`
          * **Destination**: `Virtual Network` (assuming `snet-db` is also in this VNet, 10.0.3.0/24)
          * **Destination port ranges**: `3306`
          * **Protocol**: `TCP`
          * **Action**: `Allow`
          * **Priority**: `100`
          * **Name**: `Allow_Outbound_to_DBTier`
          * Click **Add**.

      * **Deny Outbound to Internet**: (This rule ensures no direct outbound calls to the internet from the app tier)

          * **Source**: `Any`
          * **Source port ranges**: `*`
          * **Destination**: `Internet`
          * **Destination port ranges**: `*`
          * **Protocol**: `Any`
          * **Action**: `Deny`
          * **Priority**: `110` (Lower priority than the allow rule, but higher than default implicit allow)
          * **Name**: `Deny_Outbound_Internet`
          * Click **Add**.

#### 4.4.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
NSG_APP_NAME="nsg-app"
WEB_SUBNET_CIDR="10.0.0.0/24" # snet-web range
DB_SUBNET_CIDR="10.0.3.0/24" # snet-db range (assuming you created it earlier)

echo "Configuring Inbound rules for $NSG_APP_NAME"
# Allow RDP/SSH from your public IP (replace <YOUR_PUBLIC_IP>)
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_APP_NAME \
  --name "Allow-Management-IP" \
  --priority 90 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "<YOUR_PUBLIC_IP_ADDRESS>/32" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges "3389" "22" \
  --description "Allow RDP/SSH from my public IP" \
  --output tsv

# Allow Inbound from Web Tier (port 8080)
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_APP_NAME \
  --name "Allow-Inbound-from-WebTier" \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes $WEB_SUBNET_CIDR \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges "8080" \
  --description "Allow inbound from web subnet on port 8080" \
  --output tsv

echo "Configuring Outbound rules for $NSG_APP_NAME"
# Allow Outbound to Database Tier (port 3306 - MySQL)
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_APP_NAME \
  --name "Allow-Outbound-to-DBTier" \
  --priority 100 \
  --direction Outbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes $DB_SUBNET_CIDR \
  --destination-port-ranges "3306" \
  --description "Allow outbound traffic to DB subnet on port 3306" \
  --output tsv

# Deny Outbound to Internet
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_APP_NAME \
  --name "Deny-Outbound-Internet" \
  --priority 110 \
  --direction Outbound \
  --access Deny \
  --protocol "*" \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes "Internet" \
  --destination-port-ranges "*" \
  --description "Deny all outbound to Internet" \
  --output tsv
```

#### 4.4.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$nsgAppName = "nsg-app"
$yourPublicIp = "<YOUR_PUBLIC_IP_ADDRESS>/32" # Replace with your current public IP
$webSubnetCIDR = "10.0.0.0/24"
$dbSubnetCIDR = "10.0.3.0/24"

# Get the NSG object
$nsgApp = Get-AzNetworkSecurityGroup -Name $nsgAppName -ResourceGroupName $resourceGroupName

Write-Host "Configuring Inbound rules for $nsgAppName"
# Allow RDP/SSH from your public IP
$nsgApp | Add-AzNetworkSecurityRuleConfig -Name "Allow-Management-IP" `
    -Description "Allow RDP/SSH from my public IP" `
    -Direction Inbound -Priority 90 -Access Allow -Protocol Tcp `
    -SourceAddressPrefix $yourPublicIp -SourcePortRange "*" `
    -DestinationAddressPrefix "*" -DestinationPortRange "3389", "22" | Out-Null

# Allow Inbound from Web Tier (port 8080)
$nsgApp | Add-AzNetworkSecurityRuleConfig -Name "Allow-Inbound-from-WebTier" `
    -Description "Allow inbound from web subnet on port 8080" `
    -Direction Inbound -Priority 100 -Access Allow -Protocol Tcp `
    -SourceAddressPrefix $webSubnetCIDR -SourcePortRange "*" `
    -DestinationAddressPrefix "*" -DestinationPortRange "8080" | Out-Null

Write-Host "Configuring Outbound rules for $nsgAppName"
# Allow Outbound to Database Tier (port 3306 - MySQL)
$nsgApp | Add-AzNetworkSecurityRuleConfig -Name "Allow-Outbound-to-DBTier" `
    -Description "Allow outbound traffic to DB subnet on port 3306" `
    -Direction Outbound -Priority 100 -Access Allow -Protocol Tcp `
    -SourceAddressPrefix "*" -SourcePortRange "*" `
    -DestinationAddressPrefix $dbSubnetCIDR -DestinationPortRange "3306" | Out-Null

# Deny Outbound to Internet
$nsgApp | Add-AzNetworkSecurityRuleConfig -Name "Deny-Outbound-Internet" `
    -Description "Deny all outbound to Internet" `
    -Direction Outbound -Priority 110 -Access Deny -Protocol "*" `
    -SourceAddressPrefix "*" -SourcePortRange "*" `
    -DestinationAddressPrefix "Internet" -DestinationPortRange "*" | Out-Null

# Save the NSG configuration
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsgApp
```

-----

## Part 5: Testing and Validation

Now, let's test the NSG rules to ensure they are functioning as expected.

### 5.1 Initial Connectivity Test (from your local machine)

1.  **RDP/SSH to `vm-web`**: You should be able to connect via RDP (Windows) or SSH (Linux) from your local machine, as your specific public IP was allowed.
2.  **RDP/SSH to `vm-app`**: Similarly, you should be able to connect to `vm-app` from your local machine.

### 5.2 Test Web Server Access (from local machine to `vm-web`)

1.  **On `vm-web`**:
      * **Windows**: Install IIS. Open PowerShell as Administrator and run:
        ```powershell
        Install-WindowsFeature -Name Web-Server -IncludeManagementTools
        # Optional: create a simple index.html in C:\inetpub\wwwroot
        echo "<h1>Hello from vm-web!</h1>" | Out-File C:\inetpub\wwwroot\index.html
        ```
      * **Linux**: Install Nginx.
        ```bash
        sudo apt update
        sudo apt install nginx -y
        sudo systemctl start nginx
        sudo systemctl enable nginx
        ```
2.  **From your local machine**: Open a web browser and navigate to the Public IP address of `vm-web`.
      * **Expected Result**: You should see the default IIS page or Nginx welcome page / your custom "Hello" message. This confirms HTTP/HTTPS inbound rules on `nsg-web` are working.

### 5.3 Test Inter-Subnet Connectivity (from `vm-web` to `vm-app`)

1.  **On `vm-app`**:
      * **Windows**: Disable Windows Firewall temporarily for testing, or open port 8080 inbound.
      * **Linux**: Open port 8080 with `ufw`.
        ```bash
        sudo ufw allow 8080/tcp
        # Optional: Run a simple Python web server to listen on 8080
        # python3 -m http.server 8080
        ```
2.  **From `vm-web`**:
      * Try to ping `vm-app`'s Private IP (e.g., `ping 10.0.2.4`). This might fail if ICMP is blocked by default NSG rules, but TCP connectivity should work.
      * Try to establish a TCP connection to `vm-app` on port 8080.
          * **Windows (PowerShell)**: `Test-NetConnection -ComputerName 10.0.2.4 -Port 8080` (replace `10.0.2.4` with `vm-app`'s private IP)
          * **Linux (Netcat)**: `nc -vz 10.0.2.4 8080`
      * **Expected Result**: The connection test to port 8080 should succeed.
      * **Now, try to connect to a *different* port** (e.g., 80) on `vm-app` from `vm-web`.
      * **Expected Result**: This connection should **fail**, as `nsg-app` only allows 8080 from `snet-web`.

### 5.4 Test Internet Outbound from `vm-app`

1.  **Connect to `vm-app`**.
2.  Try to browse to a public website (e.g., `google.com`) or run a command like `curl google.com`.
      * **Expected Result**: The connection should **fail** or time out, as `nsg-app` has a `Deny_Outbound_Internet` rule.

### 5.5 Check Effective Security Rules (Troubleshooting tool)

If anything doesn't work as expected, you can check the effective security rules.

1.  **Azure Portal**: Navigate to your VM (`vm-web` or `vm-app`).
2.  On the left-hand menu, under "Networking," select **"Effective security rules"**.
3.  Review the inbound and outbound rules for the VM's NIC and the associated subnet. This view shows the combined effect of all NSGs applied (NIC + Subnet).

-----

## Part 6: Troubleshooting

  * **Connectivity fails even with "Allow" rule**:
      * **Rule Priority**: Remember, NSG rules are processed by priority (lower number = higher priority). Ensure your Allow rule has a lower priority number than any Deny rule that might be blocking the traffic.
      * **Rule Direction**: Is it an Inbound or Outbound rule? Traffic must be allowed in *both* directions across both NSGs if relevant. (e.g., `nsg-web` Outbound to `snet-app`, AND `nsg-app` Inbound from `snet-web`).
      * **Source/Destination**: Are the IP addresses/CIDR ranges correct? `VirtualNetwork` covers all IPs within the VNet. `Internet` covers public IPs outside the VNet.
      * **Protocol/Port**: Are the correct protocol (TCP, UDP, Any, ICMP) and ports specified?
      * **Firewall on VM**: Ensure the OS firewall (Windows Firewall, Linux UFW/firewalld) on the VM itself is not blocking the traffic. Temporarily disable it for testing, then configure it correctly.
      * **Effective Security Rules**: Use the "Effective security rules" tool in the Azure Portal (under the VM's "Networking" blade) to see the actual rules applied to a VM's network interface. This is invaluable for debugging.
  * **NSG not associated**:
      * Confirm the NSG is correctly associated with the subnet or the Network Interface Card (NIC) of the VM. Subnet association is more common for multi-VM environments.

-----

## Part 7: Cleanup Resources

To avoid incurring ongoing costs, it's crucial to delete the resources created in this lab.

**(Using Azure CLI - Recommended for comprehensive cleanup)**

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab" # The resource group containing your VNet, Subnets, VMs, and NSGs

# Delete the resource group and all contained resources
echo "Deleting resource group: $RESOURCE_GROUP and all its contents..."
az group delete --name $RESOURCE_GROUP --yes --no-wait
echo "Deletion process initiated. It may take some time to complete."
```

**(Using Azure Portal)**

1.  **Delete the VMs**:
      * Search for `Virtual machines`.
      * Select `vm-web` and `vm-app`, then click **Delete**. Confirm deletion. This will also delete their associated Public IPs and NICs.
2.  **Disassociate NSGs from Subnets**: (Only if you are NOT deleting the VNet)
      * Navigate to your Virtual Network: `vnet-app-prod`.
      * On the left-hand menu, select **"Subnets"**.
      * Click on `snet-web`, set "Network security group" to `None`, and **Save**.
      * Repeat for `snet-app`.
3.  **Delete the Network Security Groups**:
      * Search for `Network security groups`.
      * Select `nsg-web` and `nsg-app`, then click **Delete**. Confirm deletion.
4.  **Delete the Virtual Network**: (Only if you are NOT keeping it for future labs)
      * Search for `Virtual networks`.
      * Select `vnet-app-prod`.
      * Click **Delete**. Confirm deletion.
5.  **Delete the Resource Group (Recommended)**:
      * The easiest way to clean up all associated resources is to delete the resource group.
      * Search for **"Resource groups"**.
      * Select `rg-network-lab`.
      * Click **"Delete resource group"**, type the resource group name (`rg-network-lab`) to confirm, and click **"Delete"**.
      * **Warning**: This will delete *all* resources within that resource group.

-----

Congratulations\! You have successfully created and configured Azure Network Security Groups to control traffic flow between application tiers and to/from the internet. 

You've gained practical experience in implementing fundamental network security using the Azure Portal, Azure CLI, and Azure PowerShell, and verified your setup with test VMs.