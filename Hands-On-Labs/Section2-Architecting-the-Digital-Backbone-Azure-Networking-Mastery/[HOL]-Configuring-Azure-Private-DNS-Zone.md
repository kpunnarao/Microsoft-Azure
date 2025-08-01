# Lab: Configuring Azure Private DNS Zone 🔐

This lab will guide you through deploying and configuring an Azure Private DNS zone. 

You will learn how to enable DNS name resolution for virtual machines within a specific virtual network and understand how it differs from a public DNS zone.

## Lab Scenario 🧪

You have a virtual network (`vnet-lab`) with two virtual machines (`vm-app-01` and `vm-db-01`) that need to communicate with each other using a custom internal domain name (e.g., `vm-db-01.yourprivatelab.com`) instead of their private IP addresses. 

You will create a private DNS zone to host these records and link it to the virtual network. This will allow the VMs to resolve names within the private network.

**Your specific objectives for this lab are:**

* Create a Private DNS zone within a resource group.

* Link the private DNS zone to your virtual network.

* Configure `A` records for your virtual machines.

* Understand the concept of auto-registration.

* Validate name resolution by pinging one VM from another using its internal FQDN.

## Part 1: Prerequisites

* An active Azure subscription.

* **Existing VNet and VMs**: You will need an existing virtual network (e.g., `vnet-lab`) and at least two virtual machines (e.g., `vm-app-01` and `vm-db-01`) deployed within that VNet. These VMs will be used to test the private DNS resolution.

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

-----

## Part 2: Architecture Diagram (Conceptual)

The following diagram illustrates how the private DNS zone will resolve a custom internal domain name for a VM within the same VNet:

```
+------------------------------------------------------------------------------------------------+
|                                    Azure Virtual Network: `vnet-lab`                             |
|                                                                                                |
|   +--------------------------+                                                               |
|   |                          |                                                               |
|   |  `vm-app-01`             |       +--------------------------------------------+          |
|   |  Private IP: 10.0.0.4    |  <--> | Azure Private DNS Zone: `yourprivatelab.com` |          |
|   |                          |       |                                            |          |
|   |   +--------------------+ |       |   Records:                                 |          |
|   |   | Ping vm-db-01...   | |       |     A Record: `vm-db-01` -> 10.0.0.5       |          |
|   |   +--------------------+ |       |     A Record: `vm-app-01` -> 10.0.0.4      |          |
|   |                          |       +--------------------------------------------+          |
|   +--------------------------+                                                               |
|                                   |                                                          |
|   +--------------------------+    v                                                          |
|   |                          |                                                               |
|   |  `vm-db-01`              |                                                               |
|   |  Private IP: 10.0.0.5    |                                                               |
|   |                          |                                                               |
|   +--------------------------+                                                               |
|                                                                                                |
+------------------------------------------------------------------------------------------------+
```

-----

## Part 3: Step-by-Step Configuration: Deploying and Configuring a Private DNS Zone

### 3.1 Create a Private DNS Zone

#### 3.1.1 Using Azure Portal

1.  In the Azure Portal search bar, type `Private DNS zones` and select it.
2.  Click **+ Create** and fill in the details:
      * **Subscription**: Your subscription.
      * **Resource group**: `rg-network-lab`.
      * **Name**: `yourprivatelab.com` (use a placeholder name).
3.  Click **Review + create**, then **Create**.

#### 3.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
PRIVATE_DNS_ZONE_NAME="yourprivatelab.com" # Replace with your placeholder domain

echo "Creating Azure Private DNS zone: $PRIVATE_DNS_ZONE_NAME..."
az network private-dns zone create \
  --resource-group $RESOURCE_GROUP \
  --name $PRIVATE_DNS_ZONE_NAME \
  --output tsv
```

#### 3.1.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$privateDnsZoneName = "yourprivatelab.com" # Replace with your placeholder domain

Write-Host "Creating Azure Private DNS zone: $privateDnsZoneName..."
New-AzPrivateDnsZone -Name $privateDnsZoneName -ResourceGroupName $resourceGroupName
```

### 3.2 Link the Private DNS Zone to the Virtual Network

To enable name resolution, the private DNS zone must be linked to the virtual network where your VMs are located. You can also enable auto-registration, which automatically creates `A` records for VMs as they are deployed to the VNet.

#### 3.2.1 Using Azure Portal

1.  Navigate to your `yourprivatelab.com` private DNS zone.
2.  Under **Settings**, select **Virtual network links**.
3.  Click **+ Add**.
4.  **Link name**: `vnet-lab-link`.
5.  **Virtual network**: Select your `vnet-lab` from the dropdown.
6.  Check the box for **Enable auto registration**.
7.  Click **OK**.

#### 3.2.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
PRIVATE_DNS_ZONE_NAME="yourprivatelab.com"
VNET_NAME="vnet-lab"

echo "Linking the private DNS zone to the VNet with auto-registration enabled..."
az network private-dns link vnet create \
  --resource-group $RESOURCE_GROUP \
  --zone-name $PRIVATE_DNS_ZONE_NAME \
  --name "vnet-lab-link" \
  --virtual-network $VNET_NAME \
  --registration-enabled true \
  --output tsv
```

#### 3.2.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$privateDnsZoneName = "yourprivatelab.com"
$vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name "vnet-lab"

Write-Host "Linking the private DNS zone to the VNet with auto-registration enabled..."
New-AzPrivateDnsVirtualNetworkLink -ResourceGroupName $resourceGroupName `
  -ZoneName $privateDnsZoneName -Name "vnet-lab-link" `
  -VirtualNetworkId $vnet.Id -EnableRegistration
```

### 3.3 Add a Custom 'A' Record (Optional)

In some cases, you may have a resource (like a database service) that doesn't auto-register. Here's how to manually add an `A` record.

#### 3.3.1 Using Azure Portal

1.  In your `yourprivatelab.com` private DNS zone, click **+ Record set**.
2.  **Name**: `db` (this will create `db.yourprivatelab.com`).
3.  **Type**: `A - Address record`.
4.  **TTL**: `3600`.
5.  **IP address**: Enter the private IP address of your `vm-db-01`.
6.  Click **OK**.

#### 3.3.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
PRIVATE_DNS_ZONE_NAME="yourprivatelab.com"
VM_DB_PRIVATE_IP=$(az vm show -g $RESOURCE_GROUP -n "vm-db-01" --query "privateIps" -o tsv)

echo "Adding manual A record for a custom db hostname..."
az network private-dns record-set a add-record \
  --resource-group $RESOURCE_GROUP \
  --zone-name $PRIVATE_DNS_ZONE_NAME \
  --record-set-name "db" \
  --ipv4-address $VM_DB_PRIVATE_IP \
  --output tsv
```

-----

## Part 4: Testing and Validation

1.  **Access a VM**: Log in to one of your virtual machines (e.g., `vm-app-01`) using Remote Desktop or SSH.
2.  **Ping the other VM**: From the command line or terminal of `vm-app-01`, try to ping `vm-db-01` using its FQDN.
    ```bash
    # From vm-app-01
    ping vm-db-01.yourprivatelab.com
    ```
      * **Expected Result**: The ping command should successfully resolve the name to the private IP address of `vm-db-01` and you should receive a reply. This confirms that the private DNS zone is working correctly.
3.  **Test the manual record**:
    ```bash
    # From vm-app-01
    ping db.yourprivatelab.com
    ```
      * **Expected Result**: The ping should also resolve to the private IP of `vm-db-01`.

-----

## Part 5: Cleanup Resources

To avoid ongoing costs, delete the private DNS zone and any other resources you no longer need.

**(Using Azure CLI)**

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
PRIVATE_DNS_ZONE_NAME="yourprivatelab.com"

echo "Deleting private DNS zone: $PRIVATE_DNS_ZONE_NAME..."
az network private-dns zone delete \
  --resource-group $RESOURCE_GROUP \
  --name $PRIVATE_DNS_ZONE_NAME \
  --yes \
  --output tsv

echo "Private DNS zone deletion process initiated."
```

**(Using Azure Portal)**

1.  Navigate to the `Private DNS zones` service in the portal.
2.  Select `yourprivatelab.com` and click **Delete**.
3.  You can then proceed to delete any other resources you are done with, or the entire `rg-network-lab` resource group.

-----

Congratulations\! You have successfully configured an Azure Private DNS zone, linked it to a virtual network, and validated that internal name resolution is working.