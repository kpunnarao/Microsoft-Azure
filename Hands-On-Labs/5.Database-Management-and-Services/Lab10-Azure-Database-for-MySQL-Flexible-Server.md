# Lab: Azure Database for MySQL & PostgreSQL Flexible Server:

### Prerequisites:

  * An Azure Subscription. If you don't have one, you can create a [free Azure account](https://azure.microsoft.com/free/).
  * Azure Cloud Shell (recommended for ease of use) or Azure CLI and Azure PowerShell installed locally.
  * A client tool to connect to MySQL (e.g., MySQL Workbench, `mysql` command-line client, Azure Data Studio) and PostgreSQL (e.g., pgAdmin, `psql` command-line client, Azure Data Studio).

-----

### Lab 1: Azure Database for MySQL - Flexible Server

**Goal**: Create a MySQL Flexible Server and connect to it.

#### Step 1: Create the MySQL Flexible Server

**(Choose one method: Azure Portal, Azure CLI, or Azure PowerShell)**

**1.1. Azure Portal Method**

1.  Sign in to the [Azure Portal](https://portal.azure.com/).
2.  In the search bar at the top, type `Azure Database for MySQL` and select **Azure Database for MySQL flexible server**.
3.  Click **+ Create**.
4.  On the **Basics** tab:
      * **Subscription**: Select your Azure subscription.
      * **Resource Group**: Click `Create new` and enter `my-mysql-lab-rg`, then click **OK**. (Or choose an existing one).
      * **Server name**: Enter a **globally unique** name (e.g., `mymysqlflexserver-youruniqueid`).
      * **Region**: Select a region close to you (e.g., `East US`, `West Europe`).
      * **MySQL version**: Select `8.0` (recommended).
      * **Workload type**: Select `Development`. This will pre-select Burstable B1ms SKU.
      * **Admin username**: Enter `myadmin`.
      * **Password**: Enter a strong password (`YourSecurePassword123!` or similar, note it down).
      * **Confirm password**: Re-enter the password.
5.  Click **Networking** tab.
      * **Connectivity method**: Select **Public access (allowed IP addresses)**.
      * **Add current client IP address**: Click **Add current client IP address**. This will automatically add a firewall rule for your current public IP.
      * **Allow public access from any Azure service within Azure to this server**: Leave unchecked for now (or check if you know you'll need it).
6.  Click **Review + create**.
7.  Review the settings, then click **Create**.
8.  The deployment will start. This can take several minutes. Wait for the deployment to complete.

**1.2. Azure CLI Method**

1.  **Open Azure Cloud Shell** (or your local terminal with Azure CLI installed and logged in).
2.  Set environment variables:
    ```bash
    RESOURCE_GROUP="my-mysql-lab-rg"
    LOCATION="eastus" # Choose a region close to you
    MYSQL_SERVER_NAME="mymysqlflexserver-youruniqueid" # Must be globally unique
    ADMIN_USER="myadmin"
    ADMIN_PASSWORD="YourSecurePassword123!" # Replace with a strong password
    CLIENT_IP="$(curl -s ifconfig.me)" # Gets your current public IP
    ```
3.  **Create a Resource Group** (if it doesn't exist):
    ```bash
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```
4.  **Create the MySQL Flexible Server**:
    ```bash
    az mysql flexible-server create \
      --resource-group $RESOURCE_GROUP \
      --name $MYSQL_SERVER_NAME \
      --location $LOCATION \
      --admin-user $ADMIN_USER \
      --admin-password $ADMIN_PASSWORD \
      --sku-name Standard_B1ms \
      --tier Burstable \
      --version 8.0 \
      --storage-size 20 \
      --public-access $CLIENT_IP \
      --output table
    ```
      * `--public-access $CLIENT_IP`: This directly adds a firewall rule for your current IP during creation. If you omit this, you'd need to add it separately using `az mysql flexible-server firewall-rule create`.

**1.3. Azure PowerShell Method**

1.  **Open Azure PowerShell** (or your local terminal with Azure PowerShell module installed and logged in).
2.  Set environment variables:
    ```powershell
    $resourceGroupName = "my-mysql-lab-rg"
    $location = "eastus"
    $mysqlServerName = "mymysqlflexserver-youruniqueid"
    $adminUser = "myadmin"
    $adminPassword = "YourSecurePassword123!" # Replace with a strong password
    $clientIp = (Invoke-RestMethod -Uri "https://api.ipify.org").Ip # Gets your current public IP
    ```
3.  **Create a Resource Group** (if it doesn't exist):
    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location $location
    ```
4.  **Create the MySQL Flexible Server**:
    ```powershell
    New-AzMySqlFlexibleServer -ResourceGroupName $resourceGroupName `
      -Name $mysqlServerName `
      -Location $location `
      -AdministratorLogin $adminUser `
      -AdministratorLoginPassword $adminPassword `
      -SkuName "Standard_B1ms" `
      -Tier "Burstable" `
      -Version "8.0" `
      -StorageSizeGb 20 `
      -PublicAccess ($clientIp + "," + $clientIp) # Start IP, End IP comma-separated
    ```
      * `PublicAccess`: For PowerShell, you provide the `start-ip,end-ip` string.
      * If you wanted to allow all public IPs, you could use `0.0.0.0` for both start and end IP, but it's not recommended for production.

#### Step 2: Get Connection Information

Once the server is deployed (check portal or CLI/PowerShell output):

**2.1. Azure Portal Method:**

1.  Go to your MySQL Flexible Server resource in the Azure Portal.
2.  On the **Overview** blade, look for **Server name** (this is your FQDN) and **Admin username**.

**2.2. Azure CLI Method:**

```bash
az mysql flexible-server show \
  --resource-group $RESOURCE_GROUP \
  --name $MYSQL_SERVER_NAME \
  --query "{fullyQualifiedDomainName:fullyQualifiedDomainName, administratorLogin:administratorLogin}"
```

**2.3. Azure PowerShell Method:**

```powershell
(Get-AzMySqlFlexibleServer -ResourceGroupName $resourceGroupName -Name $mysqlServerName).FullyQualifiedDomainName
(Get-AzMySqlFlexibleServer -ResourceGroupName $resourceGroupName -Name $mysqlServerName).AdministratorLogin
```

#### Step 3: Connect to the MySQL Server

Use your preferred MySQL client with the connection information from Step 2 and the password you set in Step 1.

**Using `mysql` command-line client (from Azure Cloud Shell or local machine):**

```bash
mysql -h <fullyQualifiedDomainName> -u <administratorLogin> -p
```

  * Replace `<fullyQualifiedDomainName>` and `<administratorLogin>` with your values.
  * When prompted, enter your password.

**Using MySQL Workbench:**

1.  Open MySQL Workbench.
2.  Click the `+` icon to add a new connection.
3.  **Hostname**: Enter `<fullyQualifiedDomainName>`.
4.  **Port**: `3306`.
5.  **Username**: `<administratorLogin>`.
6.  **Password**: Click `Store in Keychain` and enter your password.
7.  Click **Test Connection** (you might get a warning about SSL, proceed anyway). Click **OK**.
8.  Click **OK** to save the connection, then click on it to connect.

Once connected, you can create databases, tables, and run queries\!

**Important: Clean Up Resources**

To avoid incurring unnecessary costs, remember to delete the resource groups once you are done with your labs:

```bash
az group delete --name my-mysql-lab-rg --yes --no-wait
```
