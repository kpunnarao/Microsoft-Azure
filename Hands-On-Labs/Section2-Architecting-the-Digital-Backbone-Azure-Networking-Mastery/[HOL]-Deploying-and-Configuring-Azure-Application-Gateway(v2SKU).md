# Lab: Deploying and Configuring Azure Application Gateway (v2 SKU) 🌐🛡️

This lab will guide you through the process of deploying and configuring an Azure Application Gateway (v2 SKU) to provide secure, scalable, and intelligent routing for your web application. 

You will learn how Application Gateway functions as a web traffic load balancer, handling tasks like SSL termination and intelligent content-based routing, differentiating it from the simpler Layer 4 Azure Load Balancer.

## Lab Scenario 🧪

Your multi-tier web application needs more advanced traffic management capabilities than a basic Load Balancer provides. Specifically, you want to:

* Perform SSL termination at the perimeter, offloading encryption/decryption from your backend web servers.

* Intelligently route HTTP/HTTPS traffic to your backend web servers.

* Potentially protect your web application from common web vulnerabilities using a Web Application Firewall (WAF).

You will configure an Azure Application Gateway to achieve these goals, using your existing web server VMs as backend targets.

**Your specific objectives for this lab are:**


* Create a new, dedicated subnet for the Azure Application Gateway.

* Deploy an Azure Application Gateway (v2 SKU) with a Public IP address into this new subnet.

* Configure the Application Gateway's Frontend IP, Backend Pool (containing your existing web servers), Health Probe, HTTP Settings, and Listener/Routing Rule.

* Crucially, update Network Security Group (NSG) rules on your backend web server subnet to allow traffic from the Application Gateway.

* Test and validate traffic flow and observe load balancing.

## Part 1: Prerequisites

* An active Azure subscription.

* **Existing Azure Virtual Network**: `vnet-app-prod` from previous labs.

* **Existing Web Server VMs**: `vm-web-01` and `vm-web-02` in `snet-web` from the Azure Load Balancer lab. Ensure their web servers (IIS/Nginx/Apache) are running and responding on **HTTP port 80**.

* **Existing Network Security Group (NSG)**: `nsg-web` associated with `snet-web`.

* **RDP/SSH client**: To connect to your test VMs.

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

-----

## Part 2: Architecture Diagram (Conceptual)

The following diagram illustrates the components you will deploy and configure in this lab, highlighting the Application Gateway's role in Layer 7 traffic management:

```
+------------------------------------------------------------------------------------------------+
|                                  Internet                                                      |
|                                                                                                |
|                                        |                                                       |
|                                        v                                                       |
|    +--------------------------------------------------------------------------+                |
|    |                      Azure Application Gateway                           |                |
|    |   (Standard_v2 SKU)                                                      |                |
|    |                                                                          |                |
|    |   +-------------------+                                                  |                |
|    |   | Frontend IP       |                                                  |                |
|    |   | (Public IP for AppGW)                                                |                |
|    |   +-------------------+                                                  |                |
|    |             |                                                            |                |
|    |             v (Listener: HTTP Port 80, [Optional: HTTPS Port 443 w/SSL]) |                |
|    |   +-------------------+                                                  |                |
|    |   | Routing Rule      |                                                  |                |
|    |   |                   |                                                  |                |
|    |   +---------|---------+                                                  |                |
|    |             | (HTTP Settings: Backend Protocol HTTP, Port 80)            |                |
|    |             v                                                            |                |
|    |   +-------------------+         +-------------------+                    |                |
|    |   | Backend Pool      |<------->| Health Probe      |<-------------------+                |
|    |   | (vm-web-01, vm-web-02)      | (HTTP on 80)      |  (Monitors VM health)              |
|    |   +---------|---------+         +-------------------+                    |                |
|    |             |                                                            |                |
|    +--------------------------------------------------------------------------+                |
|                          |                                                                     |
|                          | (Requires dedicated subnet for AppGW)                               |
|                          v                                                                     |
|    +-------------------------------------------------------------------------------------+     |
|    |                  Azure Virtual Network (vnet-app-prod)                              |     |
|    |                                                                                     |     |
|    |      +-------------------------------------------------------------------------+    |     |
|    |      |                         Subnet: snet-appgw (10.0.1.0/27)                |    |     |
|    |      |  (Dedicated subnet for Application Gateway)                             |    |     |
|    |      +-------------------------------------------------------------------------+    |     |
|    |                                                                                     |     |
|    |      +-------------------------------------------------------------------------+    |     |
|    |      |                         Subnet: snet-web (10.0.0.0/24)                  |    |     |
|    |      |                                                                         |    |     |
|    |      |  +----------------+                                 +----------------+  |    |     |
|    |      |  |   vm-web-01    |<-------------------------------->|   vm-web-02    | |    |     |
|    |      |  | (Web Server)   |                                 | (Web Server)   |  |    |     |
|    |      |  +----------------+                                 +----------------+  |    |     |
|    |      |       ^                                                ^                |    |     |
|    |      |       | (nsg-web: Allow HTTP/HTTPS from AzureApplicationGateway ServiceTag)  |    |     |
|    |      +-------------------------------------------------------------------------+    |     |
|    +-------------------------------------------------------------------------------------+     |
```

-----

## Part 3: Lab Setup: Prepare Your Environment

### 3.1 Create a Dedicated Subnet for Application Gateway

Application Gateway requires its own dedicated subnet, meaning no other resources (like VMs) can reside in it. 

For v2 SKU, a minimum size of /27 is acceptable for basic deployments, but `/26` is often recommended for better scaling and maintenance. We'll use `/27`.

#### 3.1.1 Using Azure Portal

1.  Navigate to your `vnet-app-prod` Virtual Network.
2.  In the left-hand menu, click **Subnets**.
3.  Click **+ Subnet**.
      * **Name**: `snet-appgw`.
      * **Address range (CIDR block)**: `10.0.1.0/27` (ensure this range doesn't overlap with existing subnets like `snet-web`).
      * **Network security group**: None (Application Gateway manages its own security, though you can associate an NSG for advanced outbound control if needed).
      * **Service endpoints**: Leave as default or None.
      * **Subnet delegation**: Leave as default or None.
      * Click **Save**.

#### 3.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
VNET_NAME="vnet-app-prod"
APPGW_SUBNET_NAME="snet-appgw"
APPGW_SUBNET_PREFIX="10.0.1.0/27" # Ensure this is unique in your VNet

echo "Adding dedicated subnet for Application Gateway..."
az network vnet subnet create \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --name $APPGW_SUBNET_NAME \
  --address-prefixes $APPGW_SUBNET_PREFIX \
  --output tsv
```

#### 3.1.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$vnetName = "vnet-app-prod"
$appgwSubnetName = "snet-appgw"
$appgwSubnetPrefix = "10.0.1.0/27" # Ensure this is unique in your VNet

Write-Host "Getting VNet object..."
$vnet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName

Write-Host "Adding dedicated subnet for Application Gateway..."
Add-AzVirtualNetworkSubnetConfig -Name $appgwSubnetName `
    -AddressPrefix $appgwSubnetPrefix `
    -VirtualNetwork $vnet | Out-Null # Pipe to null to avoid verbose output

Set-AzVirtualNetwork -VirtualNetwork $vnet
```

### 3.2 Update NSG Rules for Backend VMs

Your `nsg-web` (associated with `snet-web`) needs to allow inbound HTTP (and potentially HTTPS) traffic from the Application Gateway's service tag. This is crucial because Application Gateway probes and forwards traffic from its own internal IP range.

#### 3.2.1 Using Azure Portal (Update `nsg-web`)

1.  Navigate to `nsg-web` (the NSG associated with `snet-web`).

2.  Select **Inbound security rules**.

3.  Click **+ Add**.

      * **Priority**: A low number, e.g., `100` (higher priority than `Allow-LB-Probe-HTTP` from previous lab, or adjust accordingly).
      * **Source**: Select **"Service Tag"**.
      * **Source service tag**: Select **"AzureApplicationGateway"**.
      * **Source port ranges**: `*`.
      * **Destination**: `Any`.
      * **Destination port ranges**: `80`.
      * **Protocol**: `TCP`.
      * **Action**: `Allow`.
      * **Name**: `Allow-AppGW-HTTP`.
      * Click **Add**.

    *Note: The `Allow-Internet-HTTP` rule you created for the Azure Load Balancer lab is still necessary for external client access to the App Gateway's public IP.*
    *Optional: If you plan to configure HTTPS on App Gateway later, add another rule for port 443 with Source Service Tag `AzureApplicationGateway`.*

#### 3.2.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
NSG_NAME="nsg-web"

echo "Adding NSG rule for AzureApplicationGateway to allow HTTP traffic..."
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_NAME \
  --name "Allow-AppGW-HTTP" \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "AzureApplicationGateway" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges "80" \
  --description "Allow HTTP traffic from Azure Application Gateway" \
  --output tsv
```

#### 3.2.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$nsgName = "nsg-web"

# Get NSG object
$nsg = Get-AzNetworkSecurityGroup -Name $nsgName -ResourceGroupName $resourceGroupName

Write-Host "Adding NSG rule for AzureApplicationGateway to allow HTTP traffic..."
Add-AzNetworkSecurityRuleConfig -Name "Allow-AppGW-HTTP" `
    -NetworkSecurityGroup $nsg `
    -Description "Allow HTTP traffic from Azure Application Gateway" `
    -Direction Inbound -Priority 100 -Access Allow -Protocol Tcp `
    -SourceAddressPrefix "AzureApplicationGateway" -SourcePortRange "*" `
    -DestinationAddressPrefix "*" -DestinationPortRange "80" | Out-Null
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg
```

-----

## Part 4: Deploying and Configuring Azure Application Gateway

### 4.1 Create the Azure Application Gateway

We will deploy a Standard\_v2 SKU Application Gateway with a Public IP.

#### 4.1.1 Using Azure Portal

1.  In the Azure Portal search bar, type `Application Gateways` and select it.
2.  Click **+ Create**.
3.  **Basics Tab**:
      * **Subscription**: Your subscription.
      * **Resource group**: `rg-network-lab`.
      * **Application gateway name**: `appgw-web-prod`.
      * **Region**: Same as your VNet and VMs.
      * **Tier**: Select **Standard V2**.
      * **Enable autoscaling**: Yes (recommended for v2).
          * **Minimum instance count**: `0` (or `1` for immediate availability).
          * **Maximum instance count**: `10`.
      * **Availability zone**: Select at least `Zone 1` (or `None` if your region doesn't support zones). For redundancy, select all available zones.
      * **HTTP2**: `Enabled`.
      * **Virtual network**: Select `vnet-app-prod`.
      * **Subnet**: Select `snet-appgw`.
      * **Public IP address**: Select **Add new**.
          * **Public IP address name**: `appgw-web-prod-pip`.
          * **SKU**: `Standard`.
          * **Assignment**: `Static`.
      * **Web Application Firewall**: `Disabled` for now to keep it simple, but note that this is where you'd enable WAF\_v2.
      * Click **Next: Frontends**.

#### 4.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
LOCATION="<YourAzureRegion>"
VNET_NAME="vnet-app-prod"
APPGW_SUBNET_NAME="snet-appgw"
APPGW_NAME="appgw-web-prod"
APPGW_PIP_NAME="appgw-web-prod-pip"

# Get Application Gateway Subnet ID
APPGW_SUBNET_ID=$(az network vnet subnet show -g $RESOURCE_GROUP --vnet-name $VNET_NAME -n $APPGW_SUBNET_NAME --query id -o tsv)

echo "Creating Public IP for Application Gateway..."
az network public-ip create \
  --resource-group $RESOURCE_GROUP \
  --name $APPGW_PIP_NAME \
  --location $LOCATION \
  --sku Standard \
  --allocation-method Static \
  --output tsv

echo "Creating Application Gateway (this can take 10-15 minutes)..."
az network application-gateway create \
  --resource-group $RESOURCE_GROUP \
  --name $APPGW_NAME \
  --location $LOCATION \
  --sku Standard_v2 \
  --min-capacity 0 \
  --max-capacity 10 \
  --public-ip-address $APPGW_PIP_NAME \
  --vnet-name $VNET_NAME \
  --subnet $APPGW_SUBNET_ID \
  --http2-enabled true \
  --output tsv
```

#### 4.1.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$location = "<YourAzureRegion>"
$vnetName = "vnet-app-prod"
$appgwSubnetName = "snet-appgw"
$appgwName = "appgw-web-prod"
$appgwPublicIpName = "appgw-web-prod-pip"

Write-Host "Getting VNet and Application Gateway Subnet..."
$vnet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName
$appgwSubnet = Get-AzVirtualNetworkSubnetConfig -Name $appgwSubnetName -VirtualNetwork $vnet

Write-Host "Creating Public IP for Application Gateway..."
$publicIp = New-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name $appgwPublicIpName `
    -Location $location -AllocationMethod Static -Sku Standard

Write-Host "Creating Application Gateway (this can take 10-15 minutes)..."
$appgw = New-AzApplicationGateway -Name $appgwName -ResourceGroupName $resourceGroupName -Location $location `
    -Sku Standard_v2 -MinCapacity 0 -MaxCapacity 10 `
    -Zone 1 ` # Or '1', '2', '3' for specific zones, or multiple zones for redundancy
    -PublicIpAddress $publicIp -Subnet $appgwSubnet -Http2Enabled $true
```

*Note: PowerShell creation combines some steps compared to Portal/CLI initial setup.*

### 4.2 Configure Backend Pool

Add your web server VMs (`vm-web-01` and `vm-web-02`) to the backend pool using their private IP addresses.

#### 4.2.1 Using Azure Portal

1.  On the **Frontends** tab, confirm `appgw-web-prod-pip` is listed and click **Next: Backend pools**.
2.  Click **+ Add a backend pool**.
      * **Name**: `backend-pool-web`.
      * **Target type**: `Virtual machine`.
      * Under **Target**, select `vm-web-01`.
      * Click **Add**.
      * Under **Target**, select `vm-web-02`.
      * Click **Add**.
      * Click **Add** to save the backend pool.
3.  Click **Next: Configuration**.

#### 4.2.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
APPGW_NAME="appgw-web-prod"
BACKEND_POOL_NAME="backend-pool-web"
VM_WEB_01_PRIVATE_IP=$(az vm show -g $RESOURCE_GROUP -n "vm-web-01" --query "privateIps" -o tsv)
VM_WEB_02_PRIVATE_IP=$(az vm show -g $RESOURCE_GROUP -n "vm-web-02" --query "privateIps" -o tsv)

echo "Creating Backend Pool and adding VMs..."
az network application-gateway address-pool create \
  --resource-group $RESOURCE_GROUP \
  --gateway-name $APPGW_NAME \
  --name $BACKEND_POOL_NAME \
  --output tsv

az network application-gateway address-pool update \
  --resource-group $RESOURCE_GROUP \
  --gateway-name $APPGW_NAME \
  --name $BACKEND_POOL_NAME \
  --servers $VM_WEB_01_PRIVATE_IP $VM_WEB_02_PRIVATE_IP \
  --output tsv
```

#### 4.2.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$appgwName = "appgw-web-prod"
$backendPoolName = "backend-pool-web"

# Get IP addresses of backend VMs
$vmWeb01Ip = (Get-AzVM -ResourceGroupName $resourceGroupName -Name "vm-web-01").PrivateIpAddress
$vmWeb02Ip = (Get-AzVM -ResourceGroupName $resourceGroupName -Name "vm-web-02").PrivateIpAddress

# Get Application Gateway object
$appgw = Get-AzApplicationGateway -Name $appgwName -ResourceGroupName $resourceGroupName

Write-Host "Adding Backend Pool to Application Gateway..."
$backendPool = Add-AzApplicationGatewayBackendAddressPool -Name $backendPoolName -ApplicationGateway $appgw `
    -BackendIPAddress $vmWeb01Ip, $vmWeb02Ip

Set-AzApplicationGateway -ApplicationGateway $backendPool # Update the App Gateway with the new pool
```

### 4.3 Configure Health Probe

Create a custom health probe to monitor the health of your web servers.

#### 4.3.1 Using Azure Portal

1.  On the **Configuration** tab, click **+ Add a routing rule**.
2.  This opens the "Add a routing rule" blade. We'll define the probe here.
3.  Under the **Health probe** section, select **Create new**.
      * **Name**: `web-probe-http`.
      * **Protocol**: `HTTP`.
      * **Port**: `80`.
      * **Host**: `127.0.0.1` (or leave empty to send to backend IP). `127.0.0.1` is good if your web server only responds on localhost, or if you don't use host headers on your backend.
      * **Path**: `/`.
      * **Use probe matching**: `No` (default, checks for 200-399 status code).
      * **Interval (seconds)**: `5`.
      * **Timeout (seconds)**: `10`.
      * **Unhealthy threshold**: `2`.
      * Click **Add**.

#### 4.3.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
APPGW_NAME="appgw-web-prod"
PROBE_NAME="web-probe-http"

echo "Creating Health Probe..."
az network application-gateway probe create \
  --resource-group $RESOURCE_GROUP \
  --gateway-name $APPGW_NAME \
  --name $PROBE_NAME \
  --protocol Http \
  --host 127.0.0.1 \
  --path "/" \
  --interval 5 \
  --timeout 10 \
  --threshold 2 \
  --output tsv
```

#### 4.3.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$appgwName = "appgw-web-prod"
$probeName = "web-probe-http"

# Get Application Gateway object
$appgw = Get-AzApplicationGateway -Name $appgwName -ResourceGroupName $resourceGroupName

Write-Host "Adding Health Probe to Application Gateway..."
$probe = Add-AzApplicationGatewayProbeConfig -Name $probeName -ApplicationGateway $appgw `
    -Protocol Http -Host 127.0.0.1 -Path "/" -Interval 5 -Timeout 10 -UnhealthyThreshold 2

Set-AzApplicationGateway -ApplicationGateway $probe # Update the App Gateway with the new probe
```

### 4.4 Configure HTTP Settings

HTTP settings define how Application Gateway communicates with the backend servers.

#### 4.4.1 Using Azure Portal

1.  On the "Add a routing rule" blade, under the **Backend settings** section, select **Create new**.
      * **Name**: `http-settings-80`.
      * **Backend protocol**: `HTTP`.
      * **Backend port**: `80`.
      * **Use custom probe**: Select `Yes`, and choose `web-probe-http`.
      * **Override backend path**: `No`.
      * **Cookie-based affinity**: `Disabled` (default, or `Enabled` if you need sticky sessions).
      * **Connection draining**: `Disabled`.
      * Click **Add**.

#### 4.4.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
APPGW_NAME="appgw-web-prod"
HTTP_SETTING_NAME="http-settings-80"
PROBE_NAME="web-probe-http"

echo "Creating HTTP Settings..."
az network application-gateway http-settings create \
  --resource-group $RESOURCE_GROUP \
  --gateway-name $APPGW_NAME \
  --name $HTTP_SETTING_NAME \
  --port 80 \
  --protocol Http \
  --probe $PROBE_NAME \
  --output tsv
```

#### 4.4.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$appgwName = "appgw-web-prod"
$httpSettingName = "http-settings-80"
$probeName = "web-probe-http"

# Get Application Gateway object (with updated probe)
$appgw = Get-AzApplicationGateway -Name $appgwName -ResourceGroupName $resourceGroupName
$probe = $appgw.Probes | Where-Object {$_.Name -eq $probeName}

Write-Host "Adding HTTP Settings to Application Gateway..."
$httpSetting = Add-AzApplicationGatewayHttpSettings -Name $httpSettingName -ApplicationGateway $appgw `
    -Port 80 -Protocol Http -Probe $probe

Set-AzApplicationGateway -ApplicationGateway $httpSetting # Update the App Gateway
```

### 4.5 Configure Listener

The listener defines the frontend IP, port, and protocol that Application Gateway listens on for incoming traffic.

#### 4.5.1 Using Azure Portal

1.  On the "Add a routing rule" blade, under the **Listener** section, select **Create new**.
      * **Listener name**: `http-listener`.
      * **Frontend IP**: `Public` (This refers to the public IP created during AppGW deployment).
      * **Protocol**: `HTTP`.
      * **Port**: `80`.
      * **Listener type**: `Basic`.
      * **Error page URL**: `No`.
      * Click **Add**.

#### 4.5.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
APPGW_NAME="appgw-web-prod"
FRONTEND_IP_NAME="appgw-web-prod-pip" # This is the name of the public IP resource
LISTENER_NAME="http-listener"

echo "Creating Listener..."
az network application-gateway http-listener create \
  --resource-group $RESOURCE_GROUP \
  --gateway-name $APPGW_NAME \
  --name $LISTENER_NAME \
  --frontend-ip $FRONTEND_IP_NAME \
  --frontend-port 80 \
  --protocol Http \
  --output tsv
```

#### 4.5.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$appgwName = "appgw-web-prod"
$listenerName = "http-listener"

# Get Application Gateway object (with updated HTTP settings)
$appgw = Get-AzApplicationGateway -Name $appgwName -ResourceGroupName $resourceGroupName
$frontendIp = $appgw.FrontendIPConfigurations | Where-Object {$_.Name -eq "appgw-web-prod-pip"} # Name of frontend IP created during gateway setup

Write-Host "Adding Listener to Application Gateway..."
$listener = Add-AzApplicationGatewayHttpListener -Name $listenerName -ApplicationGateway $appgw `
    -FrontendIpConfiguration $frontendIp -FrontendPort 80 -Protocol Http

Set-AzApplicationGateway -ApplicationGateway $listener # Update the App Gateway
```

### 4.6 Configure Routing Rule

The routing rule binds the listener to the backend pool and HTTP settings.

#### 4.6.1 Using Azure Portal

1.  On the "Add a routing rule" blade (which should still be open), confirm the sections you just created:
      * **Rule name**: `rule-http`.
      * **Listener**: `http-listener`.
      * **Backend target**: `backend-pool-web`.
      * **HTTP settings**: `http-settings-80`.
      * Click **Add**.
2.  Click **Review + create**, then **Create**. (This step will take several minutes as Application Gateway initializes).

#### 4.6.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
APPGW_NAME="appgw-web-prod"
RULE_NAME="rule-http"
LISTENER_NAME="http-listener"
BACKEND_POOL_NAME="backend-pool-web"
HTTP_SETTING_NAME="http-settings-80"

echo "Creating Routing Rule..."
az network application-gateway rule create \
  --resource-group $RESOURCE_GROUP \
  --gateway-name $APPGW_NAME \
  --name $RULE_NAME \
  --http-listener $LISTENER_NAME \
  --rule-type Basic \
  --backend-address-pool $BACKEND_POOL_NAME \
  --http-settings $HTTP_SETTING_NAME \
  --output tsv
```

#### 4.6.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$appgwName = "appgw-web-prod"
$ruleName = "rule-http"

# Get Application Gateway object (with all components updated)
$appgw = Get-AzApplicationGateway -Name $appgwName -ResourceGroupName $resourceGroupName

# Get references to the configured components
$listener = $appgw.HttpListeners | Where-Object {$_.Name -eq "http-listener"}
$backendPool = $appgw.BackendAddressPools | Where-Object {$_.Name -eq "backend-pool-web"}
$httpSetting = $appgw.BackendHttpSettingsCollection | Where-Object {$_.Name -eq "http-settings-80"}

Write-Host "Adding Routing Rule to Application Gateway..."
Add-AzApplicationGatewayRequestRoutingRule -Name $ruleName -ApplicationGateway $appgw `
    -RuleType Basic -HttpListener $listener -BackendAddressPool $backendPool -BackendHttpSettings $httpSetting

Set-AzApplicationGateway -ApplicationGateway $appgw # Final update for the App Gateway
```

-----

## Part 5: Testing and Validation

### 5.1 Access Web Application via Application Gateway Public IP

1.  **Get Application Gateway Public IP**:
      * In Azure Portal, navigate to `appgw-web-prod`. The **Overview** page will show its Public IP address.
      * Alternatively, using CLI: `az network public-ip show -g rg-network-lab -n appgw-web-prod-pip --query ipAddress -o tsv`
      * Alternatively, using PowerShell: `(Get-AzPublicIpAddress -ResourceGroupName "rg-network-lab" -Name "appgw-web-prod-pip").IpAddress`
2.  Open a web browser on your local machine and navigate to `http://<ApplicationGatewayPublicIP>`.
3.  **Expected Result**: You should see either "Hello from vm-web-01\!" or "Hello from vm-web-02\!".
4.  **Observe Load Balancing**: Refresh the page multiple times. You should see the message alternate between "Hello from vm-web-01\!" and "Hello from vm-web-02\!", confirming Application Gateway is distributing traffic.

### 5.2 Health Probe Test (Simulate VM Failure)

1.  **Stop Web Service on `vm-web-01`**:
      * **Windows**: Connect to `vm-web-01` via RDP. Open PowerShell as Administrator and run `Stop-Service W3SVC`.
      * **Linux**: Connect to `vm-web-01` via SSH. Run `sudo systemctl stop nginx` (or `apache2`).
2.  Return to your local machine and refresh the web browser repeatedly at `http://<ApplicationGatewayPublicIP>`.
3.  **Expected Result**: After a few seconds (based on your probe interval and unhealthy threshold), all traffic should now be directed exclusively to "Hello from vm-web-02\!". This shows the Application Gateway has detected `vm-web-01` as unhealthy and removed it from the backend pool.
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
5.  Review the **Inbound rules**. You should now see the `Allow-AppGW-HTTP` rule listed, with its source as `AzureApplicationGateway` service tag, indicating that the NSG is correctly configured to receive traffic from the Application Gateway.

-----

## Part 6: Troubleshooting

  * **Application Gateway stuck in "Starting" or "Stopped" state**: This can take some time during creation. Ensure the dedicated subnet has enough available IP addresses (min `/27` for v2, but `/26` or `/24` recommended).
  * **VMs not showing healthy in Backend Pool**:
      * **Web Server Not Running**: Ensure IIS/Nginx/Apache is actually running on the VMs and listening on port 80.
      * **NSG Blocking Probe**: Verify the `Allow-AppGW-HTTP` rule on `nsg-web` is correctly configured to allow inbound TCP 80 from `AzureApplicationGateway` service tag.
      * **Firewall on VM**: If you have a host-based firewall (e.g., Windows Firewall, `ufw` on Linux), ensure it's allowing inbound TCP 80.
      * **Probe Path Issue**: For HTTP/HTTPS probes, ensure the path (`/` in this case) exists and returns a 200-399 OK response.
      * **Backend IP/Hostname Resolution**: If using FQDN for backend targets, ensure DNS resolution works from the Application Gateway subnet.
  * **"502 Bad Gateway" errors from Application Gateway**:
      * This usually indicates Application Gateway cannot reach the backend servers or the health probe is failing. Check the health of your backend pool in the Application Gateway overview or backend health blade.
      * Revisit NSG rules and VM host firewalls.
  * **Application Gateway not routing traffic**:
      * **Public IP/DNS Resolution**: Double-check the Application Gateway's Public IP address.
      * **Routing Rule Mismatch**: Ensure the listener's protocol/port matches incoming requests, and the HTTP settings/backend pool are correctly linked.
      * **HTTP Settings/Backend Pool/Probe Configuration**: Ensure these components are correctly defined and linked.
      * **Client NSG (if applicable)**: If you're accessing from a restricted network, ensure your client's egress NSG allows outbound traffic to the Application Gateway's public IP on the correct port.

-----

## Part 7: Cleanup Resources

To avoid incurring ongoing costs, it's crucial to delete the resources created in this lab.

**(Using Azure CLI - Recommended for comprehensive cleanup)**

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab" # The resource group containing your VNet, Subnets, VMs, NSGs, and Application Gateway

# Delete the resource group and all contained resources
echo "Deleting resource group: $RESOURCE_GROUP and all its contents..."
az group delete --name $RESOURCE_GROUP --yes --no-wait
echo "Deletion process initiated. It may take some time to complete."
```

**(Using Azure Portal)**

1.  **Delete the Application Gateway**:
      * Search for `Application Gateways`.
      * Select `appgw-web-prod`, then click **Delete**. Confirm deletion. This will also delete its associated Public IP.
2.  **Delete the VMs**:
      * Search for `Virtual machines`.
      * Select `vm-web-01` and `vm-web-02`, then click **Delete**. Confirm deletion. This will also delete their associated NICs.
3.  **Delete the Subnet**:
      * Navigate to your `vnet-app-prod` Virtual Network.
      * Go to **Subnets**, select `snet-appgw`, and click **Delete**. Confirm deletion.
4.  **Delete other resources if you're not planning future labs in this RG**:
      * If you're done with all labs in `rg-network-lab`, the most efficient way is to delete the entire resource group as detailed in previous labs.

-----

Congratulations\! You have successfully deployed and configured an Azure Application Gateway, demonstrating its capabilities as a Layer 7 load balancer for your web application. 

You've gained hands-on experience with its core components and crucial NSG integrations for secure and intelligent web traffic management.