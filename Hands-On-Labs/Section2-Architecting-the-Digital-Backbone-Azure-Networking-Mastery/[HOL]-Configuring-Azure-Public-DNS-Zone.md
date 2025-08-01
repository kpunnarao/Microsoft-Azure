# Lab: Configuring Azure Public DNS Zone 🌐

This lab will walk you through deploying and configuring an Azure Public DNS zone to host a custom domain. 

You will learn how to manage DNS records (A and CNAME records) within Azure and understand the process of delegating a domain to Azure's nameservers.

## Lab Scenario 🧪

You have a web application running on an Azure Virtual Machine with a public IP. 

Instead of having users access the application via a long and cryptic Azure-generated FQDN (e.g., `vm-web-01-pip.eastus.cloudapp.azure.com`), you want to use a more user-friendly custom domain name, such as `www.yourlabdomain.com`.

For this lab, we will use a hypothetical domain name and a practical demonstration method that doesn't require you to purchase a new domain. 

You will create a DNS zone and configure records to point to your existing web server's public IP address.

**Your specific objectives for this lab are:**

* Create a Public DNS zone in your resource group.

* Add an `A` record to map a subdomain (e.g., `www`) to a public IP address.

* Add a `CNAME` record to map another subdomain (e.g., `blog`) to the first subdomain.

* Understand the role of Azure's nameservers in making the domain resolvable.

* Validate that DNS resolution works as expected.

## Part 1: Prerequisites

* An active Azure subscription.

* **Existing VM with Public IP**: You will use a VM with a public IP address from one of the previous labs (e.g., `vm-web-01`). The VM should be running and have a web server (like IIS or Apache) configured on port 80.

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

-----

## Part 2: Architecture Diagram (Conceptual)

The following diagram illustrates how the public DNS zone will resolve a custom domain name to your web server:

```
+------------------------------------------------------------------------------------------------+
|                                        Global Internet                                         |
|                                                                                                |
|   +----------------------------------------------------------------------------------------+   |
|   |                       User requests `www.yourlabdomain.com`                              |   |
|   |                                                                                        |   |
|   +--------------------------->  DNS Query for `www.yourlabdomain.com`  <-------------------+   |
|                                                                                                |
|   +----------------------------------------------------------------------------------------+   |
|   |                                  DNS Resolution Process                                  |   |
|   |                                                                                        |   |
|   |   1. Query hits the domain's nameservers                                               |   |
|   |      (delegated to Azure DNS)                                                          |   |
|   |                                                                                        |   |
|   |   2. Azure Public DNS Zone (`yourlabdomain.com`) responds with the A record:           |   |
|   |      `www.yourlabdomain.com` -> `20.10.20.10`                                          |   |
|   |                                                                                        |   |
|   +----------------------------------------------------------------------------------------+   |
|                                         |                                                    |
|                                         v                                                    |
|                                                                                                |
|   +----------------------------+    +--------------------------------+                       |
|   | Public IP Address          |    |     Azure Web Server VM        |                       |
|   | `20.10.20.10`              +--> |     `vm-web-01`                |                       |
|   +----------------------------+    +--------------------------------+                       |
|                                                                                                |
+------------------------------------------------------------------------------------------------+
```

-----

## Part 3: Step-by-Step Configuration: Deploying and Configuring a Public DNS Zone

### 3.1 Create a Public DNS Zone

#### 3.1.1 Using Azure Portal

1.  In the Azure Portal search bar, type `DNS zones` and select it.
2.  Click **+ Create** and fill in the details:
      * **Subscription**: Your subscription.
      * **Resource group**: `rg-network-lab`.
      * **Name**: `yourlabdomain.com` (use a placeholder name).
3.  Click **Review + create**, then **Create**.
4.  Once created, navigate to the zone. You will see two record sets already created for you: an `SOA` (Start of Authority) record and an `NS` (Nameserver) record. These are essential for DNS delegation. Note the four nameservers listed in the `NS` record. In a real-world scenario, you would update these with your domain registrar.

#### 3.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
DNS_ZONE_NAME="yourlabdomain.com" # Replace with your placeholder domain

echo "Creating Azure Public DNS zone: $DNS_ZONE_NAME..."
az network dns zone create \
  --resource-group $RESOURCE_GROUP \
  --name $DNS_ZONE_NAME \
  --output tsv
```

#### 3.1.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$dnsZoneName = "yourlabdomain.com" # Replace with your placeholder domain

Write-Host "Creating Azure Public DNS zone: $dnsZoneName..."
New-AzDnsZone -Name $dnsZoneName -ResourceGroupName $resourceGroupName
```

### 3.2 Add an 'A' Record

This record will map a hostname (`www`) to the public IP address of your web server.

#### 3.2.1 Using Azure Portal

1.  In your `yourlabdomain.com` DNS zone, click **+ Record set**.
2.  **Name**: `www` (this will create `www.yourlabdomain.com`).
3.  **Type**: `A - Address record`.
4.  **TTL**: `3600` (1 hour, a common value).
5.  **IP address**: Enter the public IP address of your `vm-web-01`.
6.  Click **OK**.

#### 3.2.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
DNS_ZONE_NAME="yourlabdomain.com"
VM_PUBLIC_IP=$(az network public-ip show -g $RESOURCE_GROUP -n "vm-web-01-pip" --query ipAddress -o tsv)

echo "Adding A record for www to point to VM public IP: $VM_PUBLIC_IP..."
az network dns record-set a add-record \
  --resource-group $RESOURCE_GROUP \
  --zone-name $DNS_ZONE_NAME \
  --record-set-name "www" \
  --ipv4-address $VM_PUBLIC_IP \
  --output tsv
```

#### 3.2.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$dnsZoneName = "yourlabdomain.com"
$vmPublicIp = (Get-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name "vm-web-01-pip").IpAddress

Write-Host "Adding A record for www to point to VM public IP: $vmPublicIp..."
$record = New-AzDnsRecordConfig -IPv4Address $vmPublicIp
New-AzDnsRecordSet -Name "www" -RecordType "A" -ZoneName $dnsZoneName `
  -ResourceGroupName $resourceGroupName -Ttl 3600 -DnsRecords $record
```

### 3.3 Add a 'CNAME' Record

This record will map another hostname (`blog`) to the `www` hostname.

#### 3.3.1 Using Azure Portal

1.  In your `yourlabdomain.com` DNS zone, click **+ Record set**.
2.  **Name**: `blog`.
3.  **Type**: `CNAME - Canonical name record`.
4.  **TTL**: `3600`.
5.  **Alias name**: `www.yourlabdomain.com.` (note the trailing dot, which indicates a fully qualified domain name).
6.  Click **OK**.

#### 3.3.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
DNS_ZONE_NAME="yourlabdomain.com"

echo "Adding CNAME record for blog to point to www..."
az network dns record-set cname set-record \
  --resource-group $RESOURCE_GROUP \
  --zone-name $DNS_ZONE_NAME \
  --record-set-name "blog" \
  --cname "www.$DNS_ZONE_NAME." \
  --output tsv
```

#### 3.3.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$dnsZoneName = "yourlabdomain.com"

Write-Host "Adding CNAME record for blog to point to www..."
$record = New-AzDnsRecordConfig -Cname "www.$dnsZoneName."
New-AzDnsRecordSet -Name "blog" -RecordType "CNAME" -ZoneName $dnsZoneName `
  -ResourceGroupName $resourceGroupName -Ttl 3600 -DnsRecords $record
```

-----

## Part 4: Testing and Validation

Since you likely don't own the placeholder domain `yourlabdomain.com`, your local DNS resolver won't know to look at Azure's nameservers. To test the resolution, we will bypass the global DNS system and directly query the Azure nameservers.

1.  **Get Azure Nameservers**:

      * In the Azure Portal, navigate to your `yourlabdomain.com` DNS zone.
      * Look for the **Name servers** section at the top. Copy one of the four nameserver addresses (e.g., `ns1-07.azure-dns.com`).

2.  **Test DNS Resolution with `nslookup` or `dig`**:

      * Open a command prompt or terminal.
      * Use `nslookup` to query your record, specifying the Azure nameserver directly.

    <!-- end list -->

    ```bash
    # Syntax: nslookup <hostname> <azure_nameserver>
    nslookup www.yourlabdomain.com ns1-07.azure-dns.com
    ```

      * **Expected Result**: The output should return the public IP address of your VM.
      * Now, test the CNAME record in the same way.

    <!-- end list -->

    ```bash
    nslookup blog.yourlabdomain.com ns1-07.azure-dns.com
    ```

      * **Expected Result**: This query should also return the public IP address of your VM, as the CNAME record is a pointer to the `www` record.

3.  **Real-world Application**: In a live scenario, after updating your domain registrar's nameservers, you would simply open a browser and navigate to `http://www.yourlabdomain.com` to see your web page.

-----

## Part 5: Troubleshooting

  * **Resolution Fails**:
      * Double-check that the A record's IP address is correct.
      * Ensure the CNAME record's alias name is a fully qualified domain name ending in a dot (`.`).
      * Verify that the VM's Public IP is correct and that the web server is running.
  * **Domain not resolving globally**: This is expected behavior because you have not delegated the domain to Azure DNS. The `nslookup` command from Part 4 is the correct way to test without delegation.
  * **DNS propagation**: If you were to delegate a real domain, DNS changes can take up to 48 hours to propagate globally, though it often happens much faster.

-----

## Part 6: Cleanup Resources

To avoid incurring ongoing costs, delete the DNS zone and any resources you no longer need.

**(Using Azure CLI)**

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
DNS_ZONE_NAME="yourlabdomain.com"

echo "Deleting DNS zone: $DNS_ZONE_NAME..."
az network dns zone delete \
  --resource-group $RESOURCE_GROUP \
  --name $DNS_ZONE_NAME \
  --yes \
  --output tsv

echo "DNS zone deletion process initiated."
```

**(Using Azure Portal)**

1.  Navigate to the `DNS zones` service in the portal.
2.  Select `yourlabdomain.com` and click **Delete**.
3.  You can then proceed to delete any other resources you are done with, or the entire `rg-network-lab` resource group.

-----

Congratulations\! You have successfully configured an Azure Public DNS zone and its records. 

This is a fundamental skill for making your cloud applications accessible and user-friendly.