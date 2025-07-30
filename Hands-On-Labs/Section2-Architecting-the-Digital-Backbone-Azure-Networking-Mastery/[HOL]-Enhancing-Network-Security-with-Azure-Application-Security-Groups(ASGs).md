# Lab: Enhancing Network Security with Azure Application Security Groups (ASGs) 🚀

This lab will guide you through the process of utilizing Azure Application Security Groups (ASGs) to simplify and scale your Network Security Group (NSG) rules. 

ASGs allow you to group virtual machines logically by application workload, enabling you to define NSG rules that reference these groups rather than explicit IP addresses or subnets. 

This makes network policy management much more agile and less prone to error in dynamic environments. 

We will demonstrate the configuration using Azure Portal, Azure CLI, and Azure PowerShell for comprehensive understanding.

## Lab Scenario 🧪

Building on our multi-tier application scenario, your `vm-web` resides in `snet-web` and `vm-app` resides in `snet-app`. 

Currently, your NSG rules reference CIDR ranges (e.g., `10.0.0.0/24` for `snet-web`). You want to abstract these rules to refer to logical groups like "web servers" and "application servers" instead of network subnets. 

This will simplify management when adding or removing VMs.

Your goal is to:

* Create two Application Security Groups: one for web servers and one for application servers.

* Associate the Network Interface Cards (NICs) of your existing VMs (`vm-web`, `vm-app`) with their respective ASGs.

* Modify the existing NSG rules (`nsg-web` and `nsg-app`) to use these ASGs as source or destination, replacing the subnet CIDR ranges.

* Verify that traffic flow remains as intended after the changes.

This lab will demonstrate the creation of ASGs, their association with NICs, and the modification of NSG rules for advanced network control.

## Part 1: Prerequisites

* An active Azure subscription.

* **Existing Azure Virtual Network and Subnets**:
    
    * `vnet-app-prod` with `snet-web` (10.0.0.0/24) and `snet-app` (10.0.2.0/24) from previous labs.

* **Existing Azure Virtual Machines (VMs)**:
    
    * `vm-web` (in `snet-web`) and `vm-app` (in `snet-app`) from the NSG lab. These VMs should be running.

* **Existing Network Security Groups (NSGs)**:
    
    * `nsg-web` associated with `snet-web`.
    
    * `nsg-app` associated with `snet-app`.
    
* **Review Existing Rules**: Recall the rules we created in the NSG lab:

    * On `nsg-web`: `Allow-Outbound-to-AppTier` (Destination `10.0.2.0/24` on port 8080)

    * On `nsg-app`: `Allow-Inbound-from-WebTier` (Source `10.0.0.0/24` on port 8080)

    * We will be modifying these specific rules.

* **RDP/SSH client**: To connect to your test VMs.

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

-----

## Part 2: Architecture Diagram (Conceptual)

Below is a conceptual diagram showing how ASGs will abstract the source/destination for NSG rules.

```
+------------------------------------------------------------------+
|               Azure Virtual Network (vnet-app-prod)              |
|                                                                  |
|  +-------------------+              +-------------------+        |
|  |   Subnet: snet-web|              |   Subnet: snet-app|        |
|  |                   |              |                   |        |
|  |  +------------+   |              |  +------------+   |        |
|  |  |   vm-web   |-----> Associated with:                  |        |
|  |  +------------+   |  | asg-webservers                 |        |
|  +---------|---------+  |                                |        |
|             |            |  +------------+   |            |        |
|             |            |  |   vm-app   |<----- Associated with: |
|             |            |  +------------+   |  | asg-appservers |
|             |            |                   |            |        |
|             |            +-------------------+            |        |
|             |                                             |        |
|             v (nsg-web: Allow_Outbound_to_AppTier)         |        |
| +---------------------+                                   |        |
| | Source: Any         |                                   |        |
| | Destination:        |----------------------------------->(nsg-app: Allow_Inbound_from_WebTier)
| |   asg-appservers    |                                   ^        |
| | Port: 8080          |                                   |        |
| +---------------------+                                   |        |
|          ^                                                |        |
|          | (Internet Traffic)                              |        |
|          |                                                |        |
| +--------|---------+                                      |        |
| |       NSG: nsg-web    |                                 |        |
| +-----------------------+                                 |        |
|                                                          |        |
|                                                          |        |
|                                                          |        |
|                                                          |        |
|                                                          |        |
| +-----------------------+                                 |        |
| |       NSG: nsg-app    |                                 |        |
| +-----------------------+                                 |        |
| | Source:             |                                   |        |
| |   asg-webservers    |<-----------------------------------+        |
| | Destination: Any    |                                            |
| | Port: 8080          |                                            |
| +---------------------+                                            |
|                                                                    |
+------------------------------------------------------------------+
```

-----

## Part 3: Step-by-Step Configuration: Creating ASGs and Updating NSG Rules

### 3.1 Create Application Security Groups (ASGs)

We will create two ASGs: `asg-webservers` and `asg-appservers`.

#### 3.1.1 Using Azure Portal

1.  In the Azure Portal search bar, type `Application security groups` and select it.
2.  Click **+ Create**.
3.  **Create ASG 1 (`asg-webservers`)**:
      * **Subscription**: Your subscription.
      * **Resource group**: `rg-network-lab`.
      * **Name**: `asg-webservers`.
      * **Region**: Same as your VNet.
      * Click **Review + create**, then **Create**.
4.  **Create ASG 2 (`asg-appservers`)**:
      * Repeat the steps above, but with **Name**: `asg-appservers`.

#### 3.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
LOCATION="<YourAzureRegion>"

echo "Creating Application Security Group: asg-webservers"
az network asg create \
  --resource-group $RESOURCE_GROUP \
  --name "asg-webservers" \
  --location $LOCATION \
  --output tsv

echo "Creating Application Security Group: asg-appservers"
az network asg create \
  --resource-group $RESOURCE_GROUP \
  --name "asg-appservers" \
  --location $LOCATION \
  --output tsv
```

#### 3.1.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$location = "<YourAzureRegion>"

Write-Host "Creating Application Security Group: asg-webservers"
New-AzApplicationSecurityGroup -Name "asg-webservers" -ResourceGroupName $resourceGroupName -Location $location

Write-Host "Creating Application Security Group: asg-appservers"
New-AzApplicationSecurityGroup -Name "asg-appservers" -ResourceGroupName $resourceGroupName -Location $location
```

### 3.2 Associate Network Interfaces (NICs) with ASGs

We will associate `vm-web`'s NIC with `asg-webservers` and `vm-app`'s NIC with `asg-appservers`.

#### 3.2.1 Using Azure Portal

1.  Navigate to `vm-web`.
2.  On the left-hand menu, under "Settings," select **"Networking"**.
3.  Click on the **"Network Interface"** (e.g., `vm-web-nic`).
4.  On the NIC blade, select **"Application security groups"**.
5.  Click **"Configure application security groups"**.
6.  Check the box next to `asg-webservers`.
7.  Click **Save**.
8.  Repeat these steps for `vm-app`'s NIC, associating it with `asg-appservers`.

#### 3.2.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
VM_WEB_NIC="vm-web-nic" # Replace if your NIC name is different
VM_APP_NIC="vm-app-nic" # Replace if your NIC name is different
ASG_WEB_ID=$(az network asg show -g $RESOURCE_GROUP -n "asg-webservers" --query id -o tsv)
ASG_APP_ID=$(az network asg show -g $RESOURCE_GROUP -n "asg-appservers" --query id -o tsv)

echo "Associating $VM_WEB_NIC with asg-webservers"
az network nic update \
  --resource-group $RESOURCE_GROUP \
  --name $VM_WEB_NIC \
  --application-security-groups "$ASG_WEB_ID" \
  --output tsv

echo "Associating $VM_APP_NIC with asg-appservers"
az network nic update \
  --resource-group $RESOURCE_GROUP \
  --name $VM_APP_NIC \
  --application-security-groups "$ASG_APP_ID" \
  --output tsv
```

#### 3.2.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$vmWebNicName = "vm-web-nic" # Replace if your NIC name is different
$vmAppNicName = "vm-app-nic" # Replace if your NIC name is different

# Get ASG Objects
$asgWeb = Get-AzApplicationSecurityGroup -Name "asg-webservers" -ResourceGroupName $resourceGroupName
$asgApp = Get-AzApplicationSecurityGroup -Name "asg-appservers" -ResourceGroupName $resourceGroupName

# Get NIC Objects
$nicWeb = Get-AzNetworkInterface -Name $vmWebNicName -ResourceGroupName $resourceGroupName
$nicApp = Get-AzNetworkInterface -Name $vmAppNicName -ResourceGroupName $resourceGroupName

Write-Host "Associating $vmWebNicName with asg-webservers"
$nicWeb.ApplicationSecurityGroups.Add($asgWeb)
$nicWeb | Set-AzNetworkInterface

Write-Host "Associating $vmAppNicName with asg-appservers"
$nicApp.ApplicationSecurityGroups.Add($asgApp)
$nicApp | Set-AzNetworkInterface
```

### 3.3 Update NSG Rules to Use ASGs

Now we will modify the NSG rules to reference ASGs instead of CIDR ranges. This usually involves deleting the old rule and creating a new one with the same name and priority.

#### 3.3.1 Using Azure Portal

1.  Navigate to `nsg-web`.

2.  On the left-hand menu, select **"Outbound security rules"**.

3.  Click on the `Allow_Outbound_to_AppTier` rule (the one with Destination `VirtualNetwork` or `10.0.2.0/24`).

4.  Click **Delete**, then confirm.

5.  Click **+ Add** to create the new rule:

      * **Source**: `Any`
      * **Source port ranges**: `*`
      * **Destination**: `Application security group`
      * **Destination application security group**: Select `asg-appservers`
      * **Destination port ranges**: `8080`
      * **Protocol**: `TCP`
      * **Action**: `Allow`
      * **Priority**: `100` (must be the same as the deleted rule)
      * **Name**: `Allow_Outbound_to_AppTier` (must be the same as the deleted rule)
      * Click **Add**.

6.  Navigate to `nsg-app`.

7.  On the left-hand menu, select **"Inbound security rules"**.

8.  Click on the `Allow_Inbound_from_WebTier` rule (the one with Source `10.0.0.0/24`).

9.  Click **Delete**, then confirm.

10. Click **+ Add** to create the new rule:

      * **Source**: `Application security group`
      * **Source application security group**: Select `asg-webservers`
      * **Source port ranges**: `*`
      * **Destination**: `Any`
      * **Destination port ranges**: `8080`
      * **Protocol**: `TCP`
      * **Action**: `Allow`
      * **Priority**: `100` (must be the same as the deleted rule)
      * **Name**: `Allow_Inbound_from_WebTier` (must be the same as the deleted rule)
      * Click **Add**.

#### 3.3.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
NSG_WEB_NAME="nsg-web"
NSG_APP_NAME="nsg-app"
ASG_WEB_ID=$(az network asg show -g $RESOURCE_GROUP -n "asg-webservers" --query id -o tsv)
ASG_APP_ID=$(az network asg show -g $RESOURCE_GROUP -n "asg-appservers" --query id -o tsv)

echo "Updating NSG rule on $NSG_WEB_NAME: Allow-Outbound-to-AppTier"
# Delete the old rule
az network nsg rule delete \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_WEB_NAME \
  --name "Allow-Outbound-to-AppTier" \
  --output tsv

# Create the new rule referencing ASG
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
  --destination-asgs "$ASG_APP_ID" \
  --destination-port-ranges "8080" \
  --description "Allow outbound traffic to appservers ASG on port 8080" \
  --output tsv

echo "Updating NSG rule on $NSG_APP_NAME: Allow-Inbound-from-WebTier"
# Delete the old rule
az network nsg rule delete \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_APP_NAME \
  --name "Allow-Inbound-from-WebTier" \
  --output tsv

# Create the new rule referencing ASG
az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_APP_NAME \
  --name "Allow-Inbound-from-WebTier" \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-asgs "$ASG_WEB_ID" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges "8080" \
  --description "Allow inbound from webservers ASG on port 8080" \
  --output tsv
```

#### 3.3.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$nsgWebName = "nsg-web"
$nsgAppName = "nsg-app"

# Get ASG Objects
$asgWeb = Get-AzApplicationSecurityGroup -Name "asg-webservers" -ResourceGroupName $resourceGroupName
$asgApp = Get-AzApplicationSecurityGroup -Name "asg-appservers" -ResourceGroupName $resourceGroupName

# Get NSG Objects
$nsgWeb = Get-AzNetworkSecurityGroup -Name $nsgWebName -ResourceGroupName $resourceGroupName
$nsgApp = Get-AzNetworkSecurityGroup -Name $nsgAppName -ResourceGroupName $resourceGroupName

Write-Host "Updating NSG rule on $nsgWebName: Allow-Outbound-to-AppTier"
# Remove the old rule
$nsgWeb | Remove-AzNetworkSecurityRuleConfig -Name "Allow-Outbound-to-AppTier" -ErrorAction SilentlyContinue

# Add the new rule referencing ASG
$nsgWeb | Add-AzNetworkSecurityRuleConfig -Name "Allow-Outbound-to-AppTier" `
    -Description "Allow outbound traffic to appservers ASG on port 8080" `
    -Direction Outbound -Priority 100 -Access Allow -Protocol Tcp `
    -SourceAddressPrefix "*" -SourcePortRange "*" `
    -DestinationApplicationSecurityGroup $asgApp -DestinationPortRange "8080" | Out-Null
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsgWeb

Write-Host "Updating NSG rule on $nsgAppName: Allow-Inbound-from-WebTier"
# Remove the old rule
$nsgApp | Remove-AzNetworkSecurityRuleConfig -Name "Allow-Inbound-from-WebTier" -ErrorAction SilentlyContinue

# Add the new rule referencing ASG
$nsgApp | Add-AzNetworkSecurityRuleConfig -Name "Allow-Inbound-from-WebTier" `
    -Description "Allow inbound from webservers ASG on port 8080" `
    -Direction Inbound -Priority 100 -Access Allow -Protocol Tcp `
    -SourceApplicationSecurityGroup $asgWeb -SourcePortRange "*" `
    -DestinationAddressPrefix "*" -DestinationPortRange "8080" | Out-Null
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsgApp
```

-----

## Part 4: Testing and Validation

The beauty of ASGs is that the functional network connectivity should remain *exactly the same*, but the underlying NSG rules are now simpler and more scalable.

### 4.1 Repeat Inter-Subnet Connectivity Test (from `vm-web` to `vm-app`)

1.  **Connect to `vm-web`**.
2.  Try to establish a TCP connection to `vm-app` on port 8080 (private IP of `vm-app`).
      * **Windows (PowerShell)**: `Test-NetConnection -ComputerName <vm-app-private-ip> -Port 8080`
      * **Linux (Netcat)**: `nc -vz <vm-app-private-ip> 8080`
      * **Expected Result**: The connection test to port 8080 should still succeed. This confirms that the ASG-based rules are correctly allowing traffic.
3.  Try to connect to a *different* port (e.g., 80) on `vm-app` from `vm-web`.
      * **Expected Result**: This connection should still **fail**, demonstrating that ASGs enforce the same precise control as CIDRs.

### 4.2 Verify ASG Association and Effective Rules

1.  **Azure Portal**: Navigate to `vm-web`'s Network Interface (NIC).
2.  On the NIC blade, select **"Application security groups"**. You should see `asg-webservers` listed.
3.  On the VM's Networking blade, select **"Effective security rules"**.
4.  Examine the Inbound and Outbound rules. You should now see rules where the **Source** or **Destination** column indicates the ASG name (e.g., `asg-webservers` or `asg-appservers`) instead of a CIDR. This is how you visually confirm the ASG integration.

### 4.3 Demonstrate ASG Scalability (Conceptual)

Imagine you need to add a new web server (`vm-web-02`) to `snet-web`.

1.  You would deploy `vm-web-02` into `snet-web`.
2.  Crucially, you would simply associate `vm-web-02`'s NIC with `asg-webservers`.
3.  **No changes are required to `nsg-app`'s inbound rule\!** It will automatically apply to `vm-web-02` because `vm-web-02` is now part of the `asg-webservers` group. This is the power of ASGs.

-----

## Part 5: Troubleshooting

  * **Connectivity issues after ASG migration**:
      * **Rule Deletion/Creation Error**: Did you correctly delete the old rule and create the new one with the *exact same priority and direction*? Any deviation might cause unintended behavior or implicit deny rules to take precedence.
      * **Incorrect ASG Association**: Verify that the NICs of your VMs are correctly associated with the intended ASGs (check the VM's NIC -\> Application security groups).
      * **ASG Not Found in Rule**: Ensure you selected the correct ASG from the dropdown or used the correct ASG ID in CLI/PowerShell commands when creating the new NSG rule.
      * **Effective Security Rules**: This remains your primary debugging tool. Look for the rule that's blocking/allowing traffic and see if it's ASG-based as expected.

-----

## Part 6: Cleanup Resources

To avoid incurring ongoing costs, it's crucial to delete the resources created in this lab.

**(Using Azure CLI - Recommended for comprehensive cleanup)**

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab" # The resource group containing your VNet, Subnets, VMs, NSGs, and ASGs

# Delete the resource group and all contained resources
echo "Deleting resource group: $RESOURCE_GROUP and all its contents..."
az group delete --name $RESOURCE_GROUP --yes --no-wait
echo "Deletion process initiated. It may take some time to complete."
```

**(Using Azure Portal)**

1.  **Delete the VMs**:
      * Search for `Virtual machines`.
      * Select `vm-web` and `vm-app`, then click **Delete**. Confirm deletion. This will also delete their associated Public IPs and NICs.
2.  **Delete the Network Security Groups**:
      * Search for `Network security groups`.
      * Select `nsg-web` and `nsg-app`, then click **Delete**. Confirm deletion.
3.  **Delete the Application Security Groups**:
      * Search for `Application security groups`.
      * Select `asg-webservers` and `asg-appservers`, then click **Delete**. Confirm deletion.
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

Congratulations\! You have successfully leveraged Azure Application Security Groups to abstract and simplify your NSG rules, improving the maintainability and scalability of your network security policies. 

You've gained practical experience in ASG creation, NIC association, and NSG rule modification using the Azure Portal, Azure CLI, and Azure PowerShell.