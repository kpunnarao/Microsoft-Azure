# Lab: Transparent Data Encryption (TDE) Status Check 🔒

## Lab Scenario 🧪

You want to verify that your Azure SQL Database is protected by Transparent Data Encryption (TDE) and understand how this critical security feature is managed. 

You'll confirm its default enabled state and explore the options for customer-managed keys.

## Part 1: Prerequisites

  * An active Azure subscription.
  * The `rg-sqldb-lab` resource group, `sqldbserver-youruniqueid` logical server, and `app-db` database from previous labs must exist.
  * SQL Server Management Studio (SSMS) or Azure Data Studio connected to your `app-db` to run T-SQL queries.

-----

## Part 2: Verify TDE Status

TDE is enabled by default for all new Azure SQL Databases. Let's confirm this through different methods.

**(Choose one method for general status check + T-SQL method)**

**2.1. Azure Portal Method (Visual Check)**

1.  Sign in to the [Azure Portal](https://portal.azure.com/).
2.  Navigate to your database: `app-db`.
3.  In the left-hand menu, under **Security**, select **Transparent data encryption**.
4.  On this blade, you should see **Data encryption** status as **"On"**.
5.  Under **Key management**, it should show **"Service-managed key"**, indicating that Azure is automatically managing the encryption keys.

**2.2. Azure CLI Method (Command-line Check)**

1.  Open your terminal or Cloud Shell.
2.  Set variables:
    ```bash
    RESOURCE_GROUP="rg-sqldb-lab"
    SQL_SERVER="sqldbserver-youruniqueid" # Your unique server name
    DB_NAME="app-db"
    ```
3.  Check the TDE status of your database:
    ```bash
    az sql db tde show \
      --resource-group $RESOURCE_GROUP \
      --server $SQL_SERVER \
      --database $DB_NAME
    ```
4.  **Expected Output**: Look for `"state": "Enabled"`.

**2.3. Azure PowerShell Method (Command-line Check)**

1.  Open PowerShell.
2.  Set variables:
    ```powershell
    $resourceGroupName = "rg-sqldb-lab"
    $sqlServerName = "sqldbserver-youruniqueid" # Your unique server name
    $dbName = "app-db"
    ```
3.  Check the TDE status of your database:
    ```powershell
    Get-AzSqlDatabaseTransparentDataEncryption -ResourceGroupName $resourceGroupName `
      -ServerName $sqlServerName `
      -DatabaseName $dbName
    ```
4.  **Expected Output**: Look for `TransparentDataEncryptionStatus : Enabled`.

**2.4. T-SQL Method (In-Database Check)**

1.  Open SSMS or Azure Data Studio and connect to your `sqldbserver-youruniqueid.database.windows.net` server.
2.  Open a new query window for the `app-db` database.
3.  Execute the following query to check the encryption status from within the database itself:
    ```sql
    SELECT name, is_encrypted FROM sys.databases;
    ```
4.  **Expected Result**: In the results pane, locate your `app-db` (or `app-db-restored` if it's still present). The `is_encrypted` column for `app-db` should show **`1`**, indicating that the database is encrypted.

-----

## Part 3: Explore Customer-Managed Key (CMK) Options (Optional Exploration)

While we won't configure CMK in this basic lab, it's good to know where the option exists and its significance.

1.  **In the Azure Portal**, navigate to your **SQL server** (`sqldbserver-youruniqueid`).
2.  In the left-hand menu, under **Security**, select **Transparent data encryption**.
3.  On this blade, you'll see a section for **Key management** and the option to switch from "Service-managed key" to **"Customer-managed key"**.
4.  **Clicking on "Customer-managed key"** would prompt you to select an Azure Key Vault and a key from within it.
      * **Note**: Configuring CMK requires setting up Azure Key Vault with specific permissions and properties (like soft-delete and purge protection) beforehand. This is typically done for higher security and compliance requirements. For this lab, simply observe the interface, **do not proceed with configuration**.
5.  Navigate away from this section without making any changes.

-----

## Part 4: Cleanup

Your database and server are ready for potential future labs. No specific cleanup is required for this lab, but remember to delete the entire `rg-sqldb-lab` resource group when you are completely finished with all Azure SQL Database labs to avoid ongoing costs.

-----

You have now successfully verified that Transparent Data Encryption is enabled by default for your Azure SQL Database and understand the key management options\! This concludes our exploration of Azure SQL Database advanced features and security.