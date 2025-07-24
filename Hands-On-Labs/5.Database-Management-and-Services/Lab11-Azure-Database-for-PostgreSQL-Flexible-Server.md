# Lab: Azure Database for PostgreSQL - Flexible Server

**Goal**: Create a PostgreSQL Flexible Server and connect to it.

#### Step 1: Create the PostgreSQL Flexible Server

**(Choose one method: Azure Portal, Azure CLI, or Azure PowerShell)**

**1.1. Azure Portal Method**

1.  Sign in to the [Azure Portal](https://portal.azure.com/).
2.  In the search bar at the top, type `Azure Database for PostgreSQL` and select **Azure Database for PostgreSQL flexible server**.
3.  Click **+ Create**.
4.  On the **Basics** tab:
      * **Subscription**: Select your Azure subscription.
      * **Resource Group**: Click `Create new` and enter `my-pg-lab-rg`, then click **OK**. (Or choose an existing one).
      * **Server name**: Enter a **globally unique** name (e.g., `mypgflexserver-youruniqueid`).
      * **Region**: Select a region close to you (e.g., `East US`, `West Europe`).
      * **PostgreSQL version**: Select `16` (recommended).
      * **Workload type**: Select `Development`. This will pre-select Burstable B1ms SKU.
      * **Admin username**: Enter `myadmin`.
      * **Password**: Enter a strong password (`YourSecurePassword123!` or similar, note it down).
      * **Confirm password**: Re-enter the password.
5.  Click **Networking** tab.
      * **Connectivity method**: Select **Public access (allowed IP addresses)**.
      * **Add current client IP address**: Click **Add current client IP address**.
      * **Allow public access from any Azure service within Azure to this server**: Leave unchecked for now.
6.  Click **Review + create**.
7.  Review the settings, then click **Create**.
8.  The deployment will start. This can take several minutes. Wait for the deployment to complete.

**1.2. Azure CLI Method**

1.  **Open Azure Cloud Shell** (or your local terminal with Azure CLI installed and logged in).
2.  Set environment variables:
    ```bash
    RESOURCE_GROUP="my-pg-lab-rg"
    LOCATION="eastus"
    PG_SERVER_NAME="mypgflexserver-youruniqueid" # Must be globally unique
    ADMIN_USER="myadmin"
    ADMIN_PASSWORD="YourSecurePassword123!" # Replace with a strong password
    CLIENT_IP="$(curl -s ifconfig.me)" # Gets your current public IP
    ```
3.  **Create a Resource Group** (if it doesn't exist):
    ```bash
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```
4.  **Create the PostgreSQL Flexible Server**:
    ```bash
    az postgres flexible-server create \
      --resource-group $RESOURCE_GROUP \
      --name $PG_SERVER_NAME \
      --location $LOCATION \
      --admin-user $ADMIN_USER \
      --admin-password $ADMIN_PASSWORD \
      --sku-name Standard_B1ms \
      --tier Burstable \
      --version 16 \
      --storage-size 20 \
      --public-access $CLIENT_IP \
      --output table
    ```
      * `--public-access $CLIENT_IP`: This directly adds a firewall rule for your current IP during creation.

**1.3. Azure PowerShell Method**

1.  **Open Azure PowerShell** (or your local terminal with Azure PowerShell module installed and logged in).
2.  Set environment variables:
    ```powershell
    $resourceGroupName = "my-pg-lab-rg"
    $location = "eastus"
    $pgServerName = "mypgflexserver-youruniqueid"
    $adminUser = "myadmin"
    $adminPassword = "YourSecurePassword123!" # Replace with a strong password
    $clientIp = (Invoke-RestMethod -Uri "https://api.ipify.org").Ip # Gets your current public IP
    ```
3.  **Create a Resource Group** (if it doesn't exist):
    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location $location
    ```
4.  **Create the PostgreSQL Flexible Server**:
    ```powershell
    New-AzPostgreSqlFlexibleServer -ResourceGroupName $resourceGroupName `
      -Name $pgServerName `
      -Location $location `
      -AdministratorLogin $adminUser `
      -AdministratorLoginPassword $adminPassword `
      -SkuName "Standard_B1ms" `
      -Tier "Burstable" `
      -Version "16" `
      -StorageSizeGb 20 `
      -PublicAccess ($clientIp + "," + $clientIp)
    ```

#### Step 2: Get Connection Information

Once the server is deployed:

**2.1. Azure Portal Method:**

1.  Go to your PostgreSQL Flexible Server resource in the Azure Portal.
2.  On the **Overview** blade, look for **Server name** (this is your FQDN) and **Admin username**.

**2.2. Azure CLI Method:**

```bash
az postgres flexible-server show \
  --resource-group $RESOURCE_GROUP \
  --name $PG_SERVER_NAME \
  --query "{fullyQualifiedDomainName:fullyQualifiedDomainName, administratorLogin:administratorLogin}"
```

**2.3. Azure PowerShell Method:**

```powershell
(Get-AzPostgreSqlFlexibleServer -ResourceGroupName $resourceGroupName -Name $pgServerName).FullyQualifiedDomainName
(Get-AzPostgreSqlFlexibleServer -ResourceGroupName $resourceGroupName -Name $pgServerName).AdministratorLogin
```

#### Step 3: Connect to the PostgreSQL Server

Use your preferred PostgreSQL client with the connection information from Step 2 and the password you set in Step 1.

**Using `psql` command-line client (from Azure Cloud Shell or local machine):**

```bash
psql -h <fullyQualifiedDomainName> -U <administratorLogin> -d postgres
```

  * Replace `<fullyQualifiedDomainName>` and `<administratorLogin>` with your values.
  * The default database is `postgres`.
  * When prompted, enter your password.

**Using pgAdmin:**

1.  Open pgAdmin.
2.  Right-click on **Servers** and select **Create \> Server...**.
3.  On the **General** tab, enter a name (e.g., `MyAzurePGServer`).
4.  On the **Connection** tab:
      * **Host name/address**: Enter `<fullyQualifiedDomainName>`.
      * **Port**: `5432`.
      * **Maintenance database**: `postgres`.
      * **Username**: `<administratorLogin>`.
      * **Password**: Enter your password, and optionally check `Save password`.
5.  Click **Save**. If the connection is successful, your server will appear under the Servers list.

Once connected, you can create databases, tables, and run queries\!

-----

**Important: Clean Up Resources**

To avoid incurring unnecessary costs, remember to delete the resource groups once you are done with your labs:

```bash
az group delete --name my-mysql-lab-rg --yes --no-wait
```
