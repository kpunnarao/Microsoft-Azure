# Day 23 Lab 1: Creating an Azure SQL Database 🌐

Let's create our first Azure SQL Database using a practical, cost-effective approach.

Here is the lab document with instructions for the Azure Portal, Azure CLI, and Azure PowerShell. 

We'll use the **Serverless** compute model to keep costs as low as possible, as it automatically scales down to zero vCores during periods of inactivity.

## Lab Scenario 🧪

You are a developer tasked with provisioning a new relational database for a simple web application. 

The application will have intermittent usage, so a cost-effective solution is needed.

You'll create an Azure SQL Database using the Serverless compute tier and then connect to it using a client tool.

In this lab, you will:

1.  Create a logical Azure SQL Server.
2.  Create a single Azure SQL Database on that server, configured in the cost-effective **Serverless** tier.
3.  Configure a server-level firewall rule to allow client connections from your local machine.
4.  Connect to the new database using SQL Server Management Studio (SSMS) or Azure Data Studio.
5.  Clean up all created resources.

**Resources you'll be creating in Azure:**

  * **New Resource Group**: `rg-sqldb-lab`
  * **New Logical SQL Server**: `sqldbserver-youruniqueid`
  * **New Azure SQL Database**: `app-db`

-----

## Part 1: Prerequisites

1.  **Azure Account**: An active Azure subscription.
2.  **Azure CLI or Azure PowerShell**: Installed and logged in.
3.  **Client Tool (required for Part 3)**:
      * **SQL Server Management Studio (SSMS)**: A free, comprehensive tool for managing SQL Server instances. You can [download it here](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms).
      * **Azure Data Studio**: A cross-platform tool for data professionals. You can [download it here](https://www.google.com/search?q=https://docs.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio).
      * Choose one of these tools and have it installed on your machine.

-----

## Part 2: Create the Logical Server and Single Database

In Azure, a logical server acts as a management container for your databases. You create the server first, then place databases on it.

**(Choose one method: Azure Portal, Azure CLI, or Azure PowerShell)**

**2.1. Azure Portal Method**

1.  Sign in to the [Azure Portal](https://portal.azure.com/).
2.  In the search bar at the top, type `SQL database` and select it from the results.
3.  Click **+ Create**.
4.  **Basics Tab**:
      * **Subscription**: Select your Azure subscription.
      * **Resource group**: Click **Create new** and enter `rg-sqldb-lab`.
      * **Database details**:
          * **Database name**: `app-db`
      * **Server**: Click **Create new**.
          * **Server name**: Enter a unique name, like `sqldbserver-youruniqueid`. Note that this name must be globally unique across all of Azure.
          * **Server admin login**: Enter a username, like `azureadmin`.
          * **Password**: Create a strong password and remember it.
          * **Location**: Choose a region close to you.
      * Click **OK**.
5.  **Compute + storage**:
      * Click the **Configure database** link.
      * **Service tier**: Select **General Purpose**.
      * **Compute tier**: Select **Serverless**.
      * **Maximum vCores**: Leave at `4` (the default).
      * **Min vCores**: Leave at `0.5` (the default).
      * **Auto-pause delay**: Leave at `1 hour`.
      * **Database size**: Leave at `32 GiB`.
      * Click **Apply**.
6.  Click **Next: Networking \>**.
      * **Connectivity method**: Leave as `Public endpoint`.
      * **Firewall rules**: Select `Yes` for **Add current client IP address**. This is a crucial step that will allow you to connect from your machine.
7.  Click **Review + create**, then **Create**.
8.  Wait for the deployment to finish. This may take a few minutes.

**2.2. Azure CLI Method**

1.  Open your terminal or Cloud Shell.
2.  Set variables (replace placeholders with your unique values):
    ```bash
    RESOURCE_GROUP="rg-sqldb-lab"
    LOCATION="eastus" # Choose a region close to you
    SQL_SERVER="sqldbserver-youruniqueid" # Must be globally unique
    ADMIN_USER="azureadmin"
    ADMIN_PASSWORD="YourStrongPassword123!" # Replace with a strong, secure password
    DB_NAME="app-db"
    ```
3.  Get your public IP address to set the firewall rule:
    ```bash
    MY_IP=$(curl -s https://ifconfig.me)
    echo "Your public IP address is: $MY_IP"
    ```
4.  Create the resource group:
    ```bash
    az group create --name $RESOURCE_GROUP --location $LOCATION --output none
    ```
5.  Create the logical SQL server:
    ```bash
    az sql server create \
      --name $SQL_SERVER \
      --resource-group $RESOURCE_GROUP \
      --location $LOCATION \
      --admin-user $ADMIN_USER \
      --admin-password "$ADMIN_PASSWORD" \
      --output none
    ```
6.  Create a firewall rule to allow your IP:
    ```bash
    az sql server firewall-rule create \
      --resource-group $RESOURCE_GROUP \
      --server $SQL_SERVER \
      --name "AllowMyIP" \
      --start-ip-address $MY_IP \
      --end-ip-address $MY_IP \
      --output none
    ```
7.  Create the single database in the Serverless tier:
    ```bash
    az sql db create \
      --resource-group $RESOURCE_GROUP \
      --server $SQL_SERVER \
      --name $DB_NAME \
      --edition GeneralPurpose \
      --compute-model Serverless \
      --family Gen5 \
      --capacity 2 \
      --output none
    ```
8.  The database will be created. This command will exit before the creation is fully complete, but the process has started.

**2.3. Azure PowerShell Method**

1.  Open PowerShell.
2.  Set variables (replace placeholders with your unique values):
    ```powershell
    $resourceGroupName = "rg-sqldb-lab"
    $location = "EastUS" # Choose a region close to you
    $sqlServerName = "sqldbserver-youruniqueid" # Must be globally unique
    $adminUser = "azureadmin"
    $adminPassword = "YourStrongPassword123!" # Replace with a strong, secure password!
    $dbName = "app-db"
    ```
3.  Create the resource group:
    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location $location
    ```
4.  Create the logical SQL server:
    ```powershell
    New-AzSqlServer -ResourceGroupName $resourceGroupName `
      -ServerName $sqlServerName `
      -Location $location `
      -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminUser, (ConvertTo-SecureString -String $adminPassword -AsPlainText -Force))
    ```
5.  Get your public IP address to set the firewall rule:
    ```powershell
    $myIp = (Invoke-RestMethod -Uri "https://api.ipify.org").Ip
    Write-Host "Your public IP address is: $myIp"
    ```
6.  Create a firewall rule to allow your IP:
    ```powershell
    New-AzSqlServerFirewallRule -ResourceGroupName $resourceGroupName `
      -ServerName $sqlServerName `
      -FirewallRuleName "AllowMyIP" `
      -StartIpAddress $myIp `
      -EndIpAddress $myIp
    ```
7.  Create the single database in the Serverless tier:
    ```powershell
    New-AzSqlDatabase -ResourceGroupName $resourceGroupName `
      -ServerName $sqlServerName `
      -DatabaseName $dbName `
      -Edition "GeneralPurpose" `
      -ComputeModel "Serverless" `
      -ComputeGeneration "Gen5" `
      -VCore 2
    ```

-----

## Part 3: Connect to the Database

With the database created and a firewall rule in place, you can now connect from your local client tool.

1.  **Get the Server Name**:
      * In the Azure Portal, navigate to your new database (`app-db`).
      * On the **Overview** page, find the **Server name**. It will be in the format: `sqldbserver-youruniqueid.database.windows.net`. **Copy this full name.**
2.  **Open SSMS or Azure Data Studio**:
3.  **Establish a New Connection**:
      * **Server Name**: Paste the full server name you just copied.
      * **Authentication**: Select `SQL Server Authentication`.
      * **Login**: Enter `azureadmin` (or the username you chose).
      * **Password**: Enter the strong password you created.
4.  Click **Connect**.
5.  **Success\!**: If the connection is successful, you'll see your server and database (`app-db`) in the **Object Explorer** pane. You can expand the database, right-click on it, and select **New Query** to start writing SQL commands.

-----

## Part 4: Cleaning Up Resources (Critical\!)

To avoid incurring ongoing costs, it is essential to delete all resources created in this lab.

**(Choose one method: Azure Portal, Azure CLI, or Azure PowerShell)**

**4.1. Azure Portal Method**

1.  In the Azure Portal, search for `Resource groups` and select it.
2.  Find and click on your resource group: `rg-sqldb-lab`.
3.  On the resource group's Overview blade, click **Delete resource group**.
4.  Type the resource group name (`rg-sqldb-lab`) to confirm, then click **Delete**.
5.  This will delete your logical SQL server and the database within it.

**4.2. Azure CLI Method**

1.  Open your terminal or Cloud Shell.
2.  ```bash
    RESOURCE_GROUP="rg-sqldb-lab"
    az group delete --name $RESOURCE_GROUP --no-wait --yes
    echo "Resource group '$RESOURCE_GROUP' deletion initiated."
    ```

**4.3. Azure PowerShell Method**

1.  Open PowerShell.
2.  ```powershell
    $resourceGroupName = "rg-sqldb-lab"
    Remove-AzResourceGroup -Name $resourceGroupName -Force -AsJob
    Write-Host "Resource group '$resourceGroupName' deletion initiated."
    ```

-----

Congratulations\! You have successfully created and connected to a fully managed Azure SQL Database. 

You now know how to provision a cost-effective, scalable relational database in the cloud. We've covered the first three items on our list.