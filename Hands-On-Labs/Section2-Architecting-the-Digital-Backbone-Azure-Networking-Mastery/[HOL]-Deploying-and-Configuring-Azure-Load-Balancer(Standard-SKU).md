# Lab Guide: Deploying and Configuring Azure Load Balancer (Standard SKU) ⚖️

This lab will guide you through the process of deploying and configuring an Azure Standard Load Balancer to distribute incoming internet traffic across multiple backend Virtual Machines (VMs). 

By the end of this lab, you will have a highly available and scalable web application tier. We will cover the creation of the Load Balancer, its associated components (Public IP, Backend Pool, Health Probe, Load Balancing Rule), and essential Network Security Group (NSG) configurations. 

The configuration steps will be provided for Azure Portal, Azure CLI, and Azure PowerShell.

## Lab Scenario 🧪

Your existing multi-tier application needs a highly available and scalable web front-end. To achieve this, you will set up two identical web server VMs and place them behind an Azure Standard Load Balancer. 

The Load Balancer will act as a single point of contact for external users, distributing their requests to the healthy web servers in the backend pool. 

This ensures continuous service even if one web server becomes unavailable.

Your specific objectives for this lab are:

* Deploy two new web server VMs (`vm-web-01`, `vm-web-02`) in your existing `snet-web`.

* Install and configure a simple web server (IIS for Windows, Nginx/Apache for Linux) on each of these VMs.

* Create an Azure Standard Public Load Balancer.

* Configure the Load Balancer's Frontend IP, Backend Pool, Health Probe, and Load Balancing Rule.

* Adjust NSG rules to allow traffic from the Load Balancer and the internet to your web servers.

* Test and validate that traffic is being distributed and that the solution is resilient to a single VM failure.

## Part 1: Prerequisites

* An active Azure subscription.

* **Existing Azure Virtual Network and Subnets**:

    * `vnet-app-prod` with `snet-web` (e.g., `10.0.0.0/24`) from previous labs.

* **Existing Network Security Group (NSG)**:

    * `nsg-web` associated with `snet-web` (or directly with the NICs of VMs in `snet-web`).

* **RDP/SSH client**: To connect to your test VMs for web server installation.

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

-----

## Part 2: Architecture Diagram (Conceptual)

The following diagram illustrates the components you will deploy and configure in this lab:

```
+------------------------------------------------------------------------------------------------+
|                                  Internet                                                      |
|                                                                                                |
|                                        |                                                       |
|                                        v                                                       |
|    +--------------------------------------------------------------------------+                |
|    |                      Azure Public Load Balancer                          |                |
|    |   (Standard SKU)                                                         |                |
|    |                                                                          |                |
|    |   +-------------------+         +-------------------+                    |                |
|    |   | Frontend IP       |         | Health Probe      |<-------------------+                |
|    |   | (Public IP for LB)|<------->| (TCP/HTTP on 80)  |  (Monitors VM health)              |
|    |   +-------------------+         +-------------------+                    |                |
|    |             |                                                            |                |
|    |             |                                                            |                |
|    |             v (Load Balancing Rule: Port 80 -> Port 80)                  |                |
|    |   +-------------------+                                                  |                |
|    |   | Backend Pool      |                                                  |                |
|    |   | (Distributes traffic)                                                |                |
|    |   +---------|---------+                                                  |                |
|    |             |                                                            |                |
|    +--------------------------------------------------------------------------+                |
|                          |                                                                     |
|                          |                                                                     |
|                          v                                                                     |
|    +-------------------------------------------------------------------------------------+     |
|    |                  Azure Virtual Network (vnet-app-prod)                            |     |
|    |                                                                                     |     |
|    |      +-------------------------------------------------------------------------+    |     |
|    |      |                         Subnet: snet-web (10.0.0.0/24)                  |    |     |
|    |      |                                                                         |    |     |
|    |      |  +----------------+                                 +----------------+  |    |     |
|    |      |  |   vm-web-01    |<-------------------------------->|   vm-web-02    | |    |     |
|    |      |  | (Web Server)   |                                 | (Web Server)   |  |    |     |
|    |      |  +----------------+                                 +----------------+  |    |     |
|    |      |       ^                                                ^                |    |     |
|    |      |       | (nsg-web: Allow HTTP/HTTPS from AzureLoadBalancer & Internet)   |    |     |
|    |      +-------------------------------------------------------------------------+    |     |
|    +-------------------------------------------------------------------------------------+     |
```

-----

## Part 3: Lab Setup: Prepare Your Environment

We will leverage your existing `vnet-app-prod` and `snet-web`.

### 3.1 Deploy Two Identical Web Server VMs

Deploy two new Virtual Machines, `vm-web-01` and `vm-web-02`, into the `snet-web` subnet. Ensure they are running the same operating system (e.g., Windows Server 2022 Datacenter or Ubuntu Server 22.04 LTS).

**Important**: Do *not* assign Public IP addresses to these VMs if you want traffic to flow only through the Load Balancer. For this lab, we will configure them *without* Public IPs. Ensure you have a method to connect (e.g., Azure Bastion or a jump box with private IP access) for initial setup.

#### 3.1.1 Using Azure Portal

1.  Navigate to `Virtual machines` and click **+ Create**.
2.  **Create `vm-web-01`**:
      * **Basics Tab**:
          * **Subscription**: Your subscription.
          * **Resource group**: `rg-network-lab`.
          * **Virtual machine name**: `vm-web-01`.
          * **Region**: Same as your VNet.
          * **Image**: Choose your preferred OS (e.g., "Windows Server 2022 Datacenter - Gen2" or "Ubuntu Server 22.04 LTS").
          * **Size**: A small size like `Standard_B2s` is sufficient.
          * **Administrator account**: Create a username and password.
      * **Disks Tab**: Default settings are fine.
      * **Networking Tab**:
          * **Virtual network**: Select `vnet-app-prod`.
          * **Subnet**: Select `snet-web`.
          * **Public IP**: Select **"None"**.
          * **NIC network security group**: Select "Advanced" and choose `nsg-web`.
          * **Load balancing**: Keep as "None".
      * **Management Tab**:
          * **Login credentials**: For Windows, consider enabling `Just-in-Time VM access` or using Azure Bastion for secure RDP. For Linux, ensure SSH is allowed via `nsg-web` from your source IP.
      * Review and Create.
3.  **Create `vm-web-02`**: Repeat the exact steps above, but use `vm-web-02` as the name.

#### 3.1.2 Using Azure CLI (Example for Ubuntu)

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
LOCATION="<YourAzureRegion>"
VNET_NAME="vnet-app-prod"
SUBNET_NAME="snet-web"
NSG_NAME="nsg-web"
ADMIN_USERNAME="azureuser"
ADMIN_PASSWORD="<YourSecurePassword>" # IMPORTANT: Replace with a strong password

# Get Subnet ID
SUBNET_ID=$(az network vnet subnet show -g $RESOURCE_GROUP --vnet-name $VNET_NAME -n $SUBNET_NAME --query id -o tsv)

# Get NSG ID
NSG_ID=$(az network nsg show -g $RESOURCE_GROUP -n $NSG_NAME --query id -o tsv)

echo "Creating vm-web-01..."
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name "vm-web-01" \
  --location $LOCATION \
  --image "Ubuntu2204" \
  --size "Standard_B2s" \
  --admin-username $ADMIN_USERNAME \
  --admin-password $ADMIN_PASSWORD \
  --vnet-name $VNET_NAME \
  --subnet $SUBNET_ID \
  --nsg $NSG_ID \
  --public-ip-address "" \
  --output tsv

echo "Creating vm-web-02..."
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name "vm-web-02" \
  --location $LOCATION \
  --image "Ubuntu2204" \
  --size "Standard_B2s" \
  --admin-username $ADMIN_USERNAME \
  --admin-password $ADMIN_PASSWORD \
  --vnet-name $VNET_NAME \
  --subnet $SUBNET_ID \
  --nsg $NSG_ID \
  --public-ip-address "" \
  --output tsv
```

#### 3.1.3 Using Azure PowerShell (Example for Windows)

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$location = "<YourAzureRegion>"
$vnetName = "vnet-app-prod"
$subnetName = "snet-web"
$nsgName = "nsg-web"
$adminUsername = "azureuser"
$adminPassword = ConvertTo-SecureString -String "<YourSecurePassword>" -AsPlainText -Force # IMPORTANT: Replace

# Get existing VNet and Subnet
$vnet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName
$subnet = Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vnet

# Get existing NSG
$nsg = Get-AzNetworkSecurityGroup -Name $nsgName -ResourceGroupName $resourceGroupName

Write-Host "Creating vm-web-01..."
New-AzVM -ResourceGroupName $resourceGroupName -Name "vm-web-01" -Location $location `
    -Image "Win2022Datacenter" -Size "Standard_B2s" `
    -AuthenticationType Password -Credential (New-Object System.Management.Automation.PSCredential($adminUsername, $adminPassword)) `
    -SubnetId $subnet.Id -NetworkSecurityGroupId $nsg.Id `
    -DisablePublicIpAddress

Write-Host "Creating vm-web-02..."
New-AzVM -ResourceGroupName $resourceGroupName -Name "vm-web-02" -Location $location `
    -Image "Win2022Datacenter" -Size "Standard_B2s" `
    -AuthenticationType Password -Credential (New-Object System.Management.Automation.PSCredential($adminUsername, $adminPassword)) `
    -SubnetId $subnet.Id -NetworkSecurityGroupId $nsg.Id `
    -DisablePublicIpAddress
```

### 3.2 Install and Configure Web Servers on VMs

Connect to `vm-web-01` and `vm-web-02` using Azure Bastion (recommended) or your jump box, and install a simple web server.

#### For Windows VMs (IIS):

1.  Connect to `vm-web-01` (and then `vm-web-02`).
2.  Open **Server Manager**.
3.  Click **Add roles and features**.
4.  Select **Role-based or feature-based installation**.
5.  Select your server.
6.  Under Server Roles, check **Web Server (IIS)**. Add required features.
7.  Proceed with the installation.
8.  After installation, navigate to `C:\inetpub\wwwroot`.
9.  Open `iisstart.htm` or create a new `index.html`.
10. **For `vm-web-01`'s `index.html` (or `iisstart.htm`):** Edit the file to display a message like `<h1>Hello from vm-web-01!</h1>`.
11. **For `vm-web-02`'s `index.html` (or `iisstart.htm`):** Edit the file to display a message like `<h1>Hello from vm-web-02!</h1>`.
12. Test locally by opening `http://localhost/` in the VM's browser.

#### For Linux VMs (Nginx):

1.  Connect to `vm-web-01` (and then `vm-web-02`) via SSH.
2.  **Install Nginx**:
    ```bash
    sudo apt update
    sudo apt install -y nginx
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```
3.  **Configure Nginx default page**:
      * **For `vm-web-01`**:
        ```bash
        echo "<h1>Hello from vm-web-01!</h1>" | sudo tee /var/www/html/index.nginx-debian.html
        ```
      * **For `vm-web-02`**:
        ```bash
        echo "<h1>Hello from vm-web-02!</h1>" | sudo tee /var/www/html/index.nginx-debian.html
        ```
4.  Test locally by using `curl http://localhost/` in the VM's terminal.

-----

## Part 4: Deploying and Configuring Azure Load Balancer

### 4.1 Create the Azure Load Balancer

We'll create a Public Standard SKU Load Balancer.

#### 4.1.1 Using Azure Portal

1.  In the Azure Portal search bar, type `Load balancers` and select it.
2.  Click **+ Create**.
3.  **Basics Tab**:
      * **Subscription**: Your subscription.
      * **Resource group**: `rg-network-lab`.
      * **Name**: `lb-web-prod`.
      * **Region**: Same as your VNet and VMs.
      * **SKU**: Select **Standard**.
      * **Type**: Select **Public**.
      * **Tier**: Select **Regional**.
      * Click **Next: Frontend IP configuration**.

#### 4.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
LOCATION="<YourAzureRegion>"
LB_NAME="lb-web-prod"
LB_PUBLIC_IP_NAME="lb-web-prod-pip"

echo "Creating Public IP for Load Balancer..."
az network public-ip create \
  --resource-group $RESOURCE_GROUP \
  --name $LB_PUBLIC_IP_NAME \
  --location $LOCATION \
  --sku Standard \
  --allocation-method Static \
  --output tsv

echo "Creating Standard Load Balancer..."
az network lb create \
  --resource-group $RESOURCE_GROUP \
  --name $LB_NAME \
  --location $LOCATION \
  --sku Standard \
  --public-ip-address $LB_PUBLIC_IP_NAME \
  --frontend-ip-name "lb-frontend-pip" \
  --output tsv
```

#### 4.1.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$location = "<YourAzureRegion>"
$lbName = "lb-web-prod"
$lbPublicIpName = "lb-web-prod-pip"

Write-Host "Creating Public IP for Load Balancer..."
$publicIp = New-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name $lbPublicIpName `
    -Location $location -AllocationMethod Static -Sku Standard

Write-Host "Creating Frontend IP configuration..."
$frontendIp = New-AzLoadBalancerFrontendIpConfig -Name "lb-frontend-pip" -PublicIpAddress $publicIp

Write-Host "Creating Standard Load Balancer..."
New-AzLoadBalancer -ResourceGroupName $resourceGroupName -Name $lbName `
    -Location $location -Sku Standard -FrontendIpConfiguration $frontendIp
```

### 4.2 Configure Backend Pool

Add the NICs of your web server VMs to the backend pool.

#### 4.2.1 Using Azure Portal

1.  On the **Frontend IP configuration** tab, confirm `lb-frontend-pip` is created and click **Next: Backend pools**.
2.  Click **+ Add a backend pool**.
      * **Name**: `lb-backend-pool`.
      * **Virtual network**: Select `vnet-app-prod`.
      * Under **Virtual machines**, click **+ Add**.
      * Select `vm-web-01` and `vm-web-02`. Click **Add**.
      * Click **Add** to save the backend pool.
3.  Click **Next: Health probes**.

#### 4.2.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
LB_NAME="lb-web-prod"
VM_WEB_01_NIC_ID=$(az vm nic list -g $RESOURCE_GROUP --vm-name "vm-web-01" --query "[0].id" -o tsv)
VM_WEB_02_NIC_ID=$(az vm nic list -g $RESOURCE_GROUP --vm-name "vm-web-02" --query "[0].id" -o tsv)

echo "Creating Backend Pool and adding VMs..."
az network lb address-pool create \
  --resource-group $RESOURCE_GROUP \
  --lb-name $LB_NAME \
  --name "lb-backend-pool" \
  --output tsv

az network nic ip-config update \
  --resource-group $RESOURCE_GROUP \
  --nic-name $(basename $VM_WEB_01_NIC_ID) \
  --name "ipconfig1" \
  --lb-address-pool $LB_NAME/backendAddressPools/lb-backend-pool \
  --output tsv

az network nic ip-config update \
  --resource-group $RESOURCE_GROUP \
  --nic-name $(basename $VM_WEB_02_NIC_ID) \
  --name "ipconfig1" \
  --lb-address-pool $LB_NAME/backendAddressPools/lb-backend-pool \
  --output tsv
```

*Note: `ipconfig1` is the default name for the primary IP configuration on a VM's NIC.*

#### 4.2.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$lbName = "lb-web-prod"
$backendPoolName = "lb-backend-pool"

# Get Load Balancer object
$lb = Get-AzLoadBalancer -Name $lbName -ResourceGroupName $resourceGroupName

Write-Host "Adding Backend Pool to Load Balancer..."
$backendPool = New-AzLoadBalancerBackendAddressPoolConfig -Name $backendPoolName
Set-AzLoadBalancer -LoadBalancer $lb -BackendAddressPool $backendPool

# Add NICs to Backend Pool
$nic1 = Get-AzNetworkInterface -ResourceGroupName $resourceGroupName -Name (Get-AzVM -ResourceGroupName $resourceGroupName -Name "vm-web-01").NetworkProfile.NetworkInterfaces.Id.Split('/') | Select-Object -Last 1
$nic2 = Get-AzNetworkInterface -ResourceGroupName $resourceGroupName -Name (Get-AzVM -ResourceGroupName $resourceGroupName -Name "vm-web-02").NetworkProfile.NetworkInterfaces.Id.Split('/') | Select-Object -Last 1

$nic1.IpConfigurations[0].LoadBalancerBackendAddressPools = $lb.BackendAddressPools[0]
Set-AzNetworkInterface -NetworkInterface $nic1

$nic2.IpConfigurations[0].LoadBalancerBackendAddressPools = $lb.BackendAddressPools[0]
Set-AzNetworkInterface -NetworkInterface $nic2
```

### 4.3 Configure Health Probe

The health probe determines if a backend VM is healthy and can receive traffic.

#### 4.3.1 Using Azure Portal

1.  On the **Health probes** tab, click **+ Add a health probe**.
      * **Name**: `web-probe`.
      * **Protocol**: `HTTP`. (Choose `TCP` if your web server doesn't respond to HTTP GET on port 80, or if it's a non-HTTP service).
      * **Port**: `80`.
      * **Path**: `/`. (For HTTP/HTTPS probes, this is the path the LB will request).
      * **Interval**: `5` (seconds between probe attempts).
      * **Unhealthy threshold**: `2` (number of consecutive failures before marking as unhealthy).
      * Click **Add**.
2.  Click **Next: Load balancing rules**.

#### 4.3.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
LB_NAME="lb-web-prod"

echo "Creating Health Probe..."
az network lb probe create \
  --resource-group $RESOURCE_GROUP \
  --lb-name $LB_NAME \
  --name "web-probe" \
  --protocol Http \
  --port 80 \
  --path "/" \
  --interval 5 \
  --threshold 2 \
  --output tsv
```

#### 4.3.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$lbName = "lb-web-prod"
$probeName = "web-probe"

# Get Load Balancer object
$lb = Get-AzLoadBalancer -Name $lbName -ResourceGroupName $resourceGroupName

Write-Host "Adding Health Probe to Load Balancer..."
$probe = New-AzLoadBalancerProbeConfig -Name $probeName -Protocol Http -Port 80 -RequestPath "/" -IntervalInSeconds 5 -ProbeCount 2
$lb.Probes.Add($probe)
Set-AzLoadBalancer -LoadBalancer $lb
```

### 4.4 Configure Load Balancing Rule

This rule defines how incoming traffic is distributed.

#### 4.4.1 Using Azure Portal

1.  On the **Load balancing rules** tab, click **+ Add a load balancing rule**.
      * **Name**: `http-rule`.
      * **IP Version**: `IPv4`.
      * **Frontend IP address**: Select `lb-frontend-pip`.
      * **Backend pool**: Select `lb-backend-pool`.
      * **Protocol**: `TCP`.
      * **Port**: `80`.
      * **Backend port**: `80`.
      * **Health probe**: Select `web-probe`.
      * **Session persistence**: `None` (default, round-robin distribution). You can choose Client IP if you need sticky sessions.
      * **Idle timeout (minutes)**: `4` (default).
      * **TCP reset**: `Enabled` (recommended).
      * **Floating IP**: `Disabled` (default).
      * Click **Add**.
2.  Click **Review + create**, then **Create**.

#### 4.4.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
LB_NAME="lb-web-prod"
FRONTEND_IP_NAME="lb-frontend-pip"
BACKEND_POOL_NAME="lb-backend-pool"
PROBE_NAME="web-probe"

echo "Creating Load Balancing Rule..."
az network lb rule create \
  --resource-group $RESOURCE_GROUP \
  --lb-name $LB_NAME \
  --name "http-rule" \
  --protocol Tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name $FRONTEND_IP_NAME \
  --backend-pool-name $BACKEND_POOL_NAME \
  --probe-name $PROBE_NAME \
  --enable-tcp-reset true \
  --output tsv
```

#### 4.4.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$lbName = "lb-web-prod"
$frontendIpName = "lb-frontend-pip"
$backendPoolName = "lb-backend-pool"
$probeName = "web-probe"
$ruleName = "http-rule"

# Get Load Balancer object (with updated frontend, backend pool, and probe)
$lb = Get-AzLoadBalancer -Name $lbName -ResourceGroupName $resourceGroupName

# Get references to the configured components
$frontendIp = $lb.FrontendIpConfigurations | Where-Object {$_.Name -eq $frontendIpName}
$backendPool = $lb.BackendAddressPools | Where-Object {$_.Name -eq $backendPoolName}
$probe = $lb.Probes | Where-Object {$_.Name -eq $probeName}

Write-Host "Adding Load Balancing Rule..."
Add-AzLoadBalancerRuleConfig -Name $ruleName `
    -LoadBalancer $lb `
    -FrontendIpConfiguration $frontendIp `
    -BackendAddressPool $backendPool `
    -Probe $probe `
    -Protocol Tcp `
    -FrontendPort 80 `
    -BackendPort 80 `
    -EnableTcpReset $true

Set-AzLoadBalancer -LoadBalancer $lb
```

### 4.5 Crucial NSG Rule Update for Load Balancer Traffic

Since Standard Load Balancers are **secure by default**, you *must* explicitly allow inbound traffic in the NSG associated with your backend VMs (or subnet). We need to allow HTTP traffic from the internet and, crucially, from the Azure Load Balancer health probes.

#### 4.5.1 Using Azure Portal (Update `nsg-web`)

1.  Navigate to `nsg-web` (the NSG associated with `snet-web`).

2.  Select **Inbound security rules**.

3.  Click **+ Add**.

      * **Priority**: A low number, e.g., `105` (higher priority than default deny).
      * **Source**: Select **"Service Tag"**.
      * **Source service tag**: Select **"AzureLoadBalancer"**.
      * **Source port ranges**: `*`.
      * **Destination**: `Any`.
      * **Destination port ranges**: `80`.
      * **Protocol**: `TCP`.
      * **Action**: `Allow`.
      * **Name**: `Allow-LB-Probe-HTTP`.
      * Click **Add**.

4.  Click **+ Add** again for client traffic.

      * **Priority**: A low number, e.g., `110`.
      * **Source**: Select **"Service Tag"**.
      * **Source service tag**: Select **"Internet"**.
      * **Source port ranges**: `*`.
      * **Destination**: `Any`.
      * **Destination port ranges**: `80`.
      * **Protocol**: `TCP`.
      * **Action**: `Allow`.
      * **Name**: `Allow-Internet-HTTP`.
      * Click **Add**.

#### 4.5.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
NSG_NAME="nsg-web"

echo "Adding NSG rule for AzureLoadBalancer probes..."
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_NAME \
  --name "Allow-LB-Probe-HTTP" \
  --priority 105 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "AzureLoadBalancer" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges "80" \
  --description "Allow HTTP traffic from Azure Load Balancer health probes" \
  --output tsv

echo "Adding NSG rule for Internet HTTP traffic..."
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_NAME \
  --name "Allow-Internet-HTTP" \
  --priority 110 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "Internet" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges "80" \
  --description "Allow inbound HTTP traffic from the Internet" \
  --output tsv
```

#### 4.5.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$nsgName = "nsg-web"

# Get NSG object
$nsg = Get-AzNetworkSecurityGroup -Name $nsgName -ResourceGroupName $resourceGroupName

Write-Host "Adding NSG rule for AzureLoadBalancer probes..."
Add-AzNetworkSecurityRuleConfig -Name "Allow-LB-Probe-HTTP" `
    -NetworkSecurityGroup $nsg `
    -Description "Allow HTTP traffic from Azure Load Balancer health probes" `
    -Direction Inbound -Priority 105 -Access Allow -Protocol Tcp `
    -SourceAddressPrefix "AzureLoadBalancer" -SourcePortRange "*" `
    -DestinationAddressPrefix "*" -DestinationPortRange "80" | Out-Null
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg

Write-Host "Adding NSG rule for Internet HTTP traffic..."
Add-AzNetworkSecurityRuleConfig -Name "Allow-Internet-HTTP" `
    -NetworkSecurityGroup $nsg `
    -Description "Allow inbound HTTP traffic from the Internet" `
    -Direction Inbound -Priority 110 -Access Allow -Protocol Tcp `
    -SourceAddressPrefix "Internet" -SourcePortRange "*" `
    -DestinationAddressPrefix "*" -DestinationPortRange "80" | Out-Null
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg
```

-----

## Part 5: Testing and Validation

### 5.1 Access Web Application via Load Balancer Public IP

1.  **Get Load Balancer Public IP**:
      * In Azure Portal, navigate to `lb-web-prod`. The **Frontend IP configuration** section will show its Public IP address.
      * Alternatively, using CLI: `az network public-ip show -g rg-network-lab -n lb-web-prod-pip --query ipAddress -o tsv`
      * Alternatively, using PowerShell: `(Get-AzPublicIpAddress -ResourceGroupName "rg-network-lab" -Name "lb-web-prod-pip").IpAddress`
2.  Open a web browser on your local machine and navigate to `http://<LoadBalancerPublicIP>`.
3.  **Expected Result**: You should see either "Hello from vm-web-01\!" or "Hello from vm-web-02\!".
4.  **Observe Load Balancing**: Refresh the page multiple times. You should see the message alternate between "Hello from vm-web-01\!" and "Hello from vm-web-02\!", demonstrating that the Load Balancer is distributing traffic in a round-robin fashion.

### 5.2 Health Probe Test (Simulate VM Failure)

1.  **Stop Web Service on `vm-web-01`**:
      * **Windows**: Connect to `vm-web-01` via RDP. Open PowerShell as Administrator and run `Stop-Service W3SVC`.
      * **Linux**: Connect to `vm-web-01` via SSH. Run `sudo systemctl stop nginx` (or `apache2`).
2.  Return to your local machine and refresh the web browser repeatedly at `http://<LoadBalancerPublicIP>`.
3.  **Expected Result**: After a few seconds (based on your probe interval and unhealthy threshold), all traffic should now be directed exclusively to "Hello from vm-web-02\!". This shows the Load Balancer has detected `vm-web-01` as unhealthy and removed it from the backend pool.
4.  **Restart Web Service on `vm-web-01`**:
      * **Windows**: `Start-Service W3SVC`.
      * **Linux**: `sudo systemctl start nginx` (or `apache2`).
5.  Refresh the web page again after a short delay.
6.  **Expected Result**: Traffic should once again be distributed between both "Hello from vm-web-01\!" and "Hello from vm-web-02\!".

### 5.3 Verify Effective Security Rules on VM NICs

1.  In the Azure Portal, navigate to `vm-web-01`.
2.  On the left-hand menu, under "Settings," select **"Networking"**.
3.  Click on the Network Interface (e.g., `vm-web-01-nic`).
4.  Under "Settings," select **"Effective security rules"**.
5.  Review the **Inbound rules**. You should now see the `Allow-LB-Probe-HTTP` and `Allow-Internet-HTTP` rules listed, with their sources as `AzureLoadBalancer` and `Internet` service tags respectively, indicating that the NSG is correctly configured to receive traffic from the Load Balancer.

-----

## Part 6: Troubleshooting

  * **VMs not showing healthy in Backend Pool**:
      * **Web Server Not Running**: Ensure IIS/Nginx/Apache is actually running on the VMs and listening on port 80.
      * **NSG Blocking Probe**: Verify the `Allow-LB-Probe-HTTP` rule on `nsg-web` (or directly on the VM NIC's NSG) is correctly configured to allow inbound TCP 80 from `AzureLoadBalancer`.
      * **Firewall on VM**: If you have a host-based firewall (e.g., Windows Firewall, `ufw` on Linux), ensure it's allowing inbound TCP 80.
      * **Probe Path Issue**: For HTTP/HTTPS probes, ensure the path (`/` in this case) exists and returns a 200 OK response.
  * **Traffic not reaching VMs (browser shows timeout/error)**:
      * **Public IP/DNS Resolution**: Double-check the Load Balancer's Public IP address.
      * **Load Balancing Rule Mismatch**: Ensure the frontend port, backend port, and protocol in the load balancing rule match your web server configuration.
      * **NSG Blocking Internet Traffic**: Verify the `Allow-Internet-HTTP` rule on `nsg-web` is correctly configured to allow inbound TCP 80 from `Internet`.
      * **Backend Pool Empty/Unhealthy**: Check the "Backend pools" section of your Load Balancer in the portal to see if the VMs are listed as "Healthy".
  * **No Load Balancing (always hits the same VM)**:
      * **Session Persistence**: Check the "Session persistence" setting on your load balancing rule. If it's set to "Client IP" or "Client IP and Protocol", it will try to send subsequent requests from the same client to the same VM. Set it to "None" for round-robin.
      * **Browser Caching**: Try using a different browser, incognito/private mode, or clear your browser cache to ensure it's not simply serving a cached page.

-----

## Part 7: Cleanup Resources

To avoid incurring ongoing costs, it's crucial to delete the resources created in this lab.

**(Using Azure CLI - Recommended for comprehensive cleanup)**

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab" # The resource group containing your VNet, Subnets, VMs, NSGs, ASGs, and Load Balancer

# Delete the resource group and all contained resources
echo "Deleting resource group: $RESOURCE_GROUP and all its contents..."
az group delete --name $RESOURCE_GROUP --yes --no-wait
echo "Deletion process initiated. It may take some time to complete."
```

**(Using Azure Portal)**

1.  **Delete the Load Balancer**:
      * Search for `Load balancers`.
      * Select `lb-web-prod`, then click **Delete**. Confirm deletion. This will also delete its associated Public IP.
2.  **Delete the VMs**:
      * Search for `Virtual machines`.
      * Select `vm-web-01` and `vm-web-02`, then click **Delete**. Confirm deletion. This will also delete their associated NICs (as they had no Public IPs).
3.  **Delete other resources if you're not planning future labs in this RG**:
      * If you're done with all labs in `rg-network-lab`, the most efficient way is to delete the entire resource group as detailed in previous labs.

-----

Congratulations\! You have successfully deployed and configured an Azure Standard Load Balancer, demonstrating how to achieve high availability and load distribution for your web application. 

You've gained hands-on experience with key Load Balancer concepts and crucial NSG integrations for secure and efficient traffic flow.