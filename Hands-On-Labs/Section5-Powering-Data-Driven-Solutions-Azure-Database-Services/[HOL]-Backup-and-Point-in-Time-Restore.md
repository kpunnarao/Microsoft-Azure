# Lab: Azure SQL Database - Backup and Point-in-Time Restore ⏰🔄

This is a super important lab for understanding data recovery in the cloud. 

We'll simulate a common scenario: accidental data deletion, and then use Azure's built-in point-in-time restore capabilities to recover.

## Lab Scenario 🧪

Accidental data loss is a common problem. In this lab, you will simulate an accidental data deletion in your `app-db` and then use Azure SQL Database's automated backups to perform a point-in-time restore to a new database, recovering the lost data.

## Part 1: Prerequisites

  * An active Azure subscription.
  * The `rg-sqldb-lab` resource group, `sqldbserver-youruniqueid` logical server, and `app-db` database must exist and contain the `Products` table with the sample data from Day 23 Lab 1.
  * SQL Server Management Studio (SSMS) or Azure Data Studio connected to your `app-db`.
  * Azure CLI or Azure PowerShell installed and logged in.

## Part 2: Simulate Data Loss

First, let's create a scenario where we accidentally delete some data.

1.  **Connect to `app-db`**:

      * Open SSMS or Azure Data Studio and connect to your `sqldbserver-youruniqueid.database.windows.net` server using your `azureadmin` credentials.
      * Open a new query window for the `app-db` database.

2.  **Record Current Time**:

      * Before you delete, note down the **exact current time (including seconds)**. This is crucial for your point-in-time restore. It's best to record it in **UTC** (Coordinated Universal Time) if possible, as Azure typically uses UTC for timestamps.
      * You can run `SELECT GETUTCDATE();` in your query window to get the current UTC time from the database.
      * *Example: 2025-07-24 19:19:30.000*

3.  **Verify Existing Data**:

      * Execute the following query to see all current products:
        ```sql
        SELECT * FROM Products;
        ```
      * You should see your three products: Laptop, Mouse, and Keyboard.

4.  **Simulate Deletion**:

      * Execute the following `DELETE` statement to remove the 'Laptop' product.
        ```sql
        DELETE FROM Products WHERE Name = 'Laptop';
        ```
      * Verify the deletion:
        ```sql
        SELECT * FROM Products;
        ```
      * You should now only see 'Mouse' and 'Keyboard'.

-----

## Part 3: Perform Point-in-Time Restore (PITR)

Now, we will restore the database to a point *before* you deleted the 'Laptop' product. Remember, a restore always creates a **new database**.

**(Choose one method: Azure Portal, Azure CLI, or Azure PowerShell)**

**3.1. Azure Portal Method**

1.  Sign in to the [Azure Portal](https://portal.azure.com/).
2.  Navigate to your original database: `app-db`.
3.  On the **Overview** page, click the **Restore** button on the top toolbar.
4.  **Restore Point**:
      * **Source database**: `app-db` (should be pre-selected).
      * **Restore to**: Select `Point-in-time`.
      * **Point in time**: Select **Custom**.
          * Carefully enter the **UTC date and time** you recorded in **Part 2, Step 2**, ensuring it is *before* your deletion operation.
          * *Example: If you deleted at 2025-07-24 19:19:30 UTC, set your restore point to 2025-07-24 19:19:00 UTC (or any time before the deletion).*
5.  **Restore details**:
      * **New database name**: Enter `app-db-restored`.
      * **Target server**: Select your existing server `sqldbserver-youruniqueid.database.windows.net`.
6.  Click **Review + create**, then **Create**.
7.  The deployment will start. This process can take several minutes to complete, depending on the size and activity of your database. You can monitor the progress in the Azure portal notifications.

**3.2. Azure CLI Method**

1.  Open your terminal or Cloud Shell.

2.  Set variables (replace placeholders):

    ```bash
    RESOURCE_GROUP="rg-sqldb-lab"
    SQL_SERVER="sqldbserver-youruniqueid" # Your unique server name
    ORIGINAL_DB_NAME="app-db"
    RESTORED_DB_NAME="app-db-restored"
    # *** IMPORTANT: Replace with the UTC time BEFORE your deletion ***
    # Example: RESTORE_TIME="2025-07-24T19:19:00.000Z" (Note the 'Z' for UTC)
    RESTORE_TIME="YYYY-MM-DDTHH:MM:SS.000Z"
    ```

      * **Crucial**: Make sure `RESTORE_TIME` is the UTC time *before* you executed the `DELETE` statement.

3.  Perform the point-in-time restore:

    ```bash
    az sql db restore \
      --resource-group $RESOURCE_GROUP \
      --server $SQL_SERVER \
      --name $ORIGINAL_DB_NAME \
      --dest-name $RESTORED_DB_NAME \
      --time $RESTORE_TIME \
      --output none
    echo "Restore of '$ORIGINAL_DB_NAME' to '$RESTORED_DB_NAME' initiated."
    ```

4.  Monitor the restore process in the Azure portal or use `az sql db show --resource-group $RESOURCE_GROUP --server $SQL_SERVER --name $RESTORED_DB_NAME --query status`.

**3.3. Azure PowerShell Method**

1.  Open PowerShell.

2.  Set variables (replace placeholders):

    ```powershell
    $resourceGroupName = "rg-sqldb-lab"
    $sqlServerName = "sqldbserver-youruniqueid" # Your unique server name
    $originalDbName = "app-db"
    $restoredDbName = "app-db-restored"
    # *** IMPORTANT: Replace with the UTC time BEFORE your deletion ***
    # Example: $restoreTime = Get-Date "2025-07-24T19:19:00Z" -AsUTC
    $restoreTime = Get-Date "YYYY-MM-DDTHH:MM:SSZ" -AsUTC
    ```

      * **Crucial**: Make sure `$restoreTime` is the UTC time *before* you executed the `DELETE` statement. Use the `-AsUTC` flag to ensure PowerShell handles the time as UTC.

3.  Perform the point-in-time restore:

    ```powershell
    Restore-AzSqlDatabase -FromPointInTimeBackup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $sqlServerName `
      -DatabaseName $originalDbName `
      -TargetDatabaseName $restoredDbName `
      -PointInTime $restoreTime
    Write-Host "Restore of '$originalDbName' to '$restoredDbName' initiated."
    ```

4.  Monitor the restore process in the Azure portal or using `Get-AzSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $sqlServerName -DatabaseName $restoredDbName`.

-----

## Part 4: Verify Data Recovery

Once the restore deployment is complete and the `app-db-restored` database is online:

1.  **Connect to `app-db-restored`**:
      * In SSMS or Azure Data Studio, establish a **new connection** to your server `sqldbserver-youruniqueid.database.windows.net`.
      * This time, in the **Connect to Server** dialog (or New Connection), ensure you select `app-db-restored` as the **Database name** (if using SSMS). In ADS, you'll connect to the server and then select the database in the object explorer.
2.  **Verify Recovered Data**:
      * Open a new query window for the `app-db-restored` database.
      * Execute the following query:
        ```sql
        SELECT * FROM Products;
        ```
      * **Expected Result**: You should now see all three original products, including the 'Laptop' that was deleted from your `app-db`\! This confirms a successful point-in-time recovery.

-----

## Part 5: Cleanup (Crucial\!)

**You must delete the `app-db-restored` database to avoid incurring additional costs\!** You can leave your original `app-db` for the next labs.

**(Choose one method: Azure Portal, Azure CLI, or Azure PowerShell)**

**5.1. Azure Portal Method**

1.  Navigate to the `app-db-restored` database in the Azure Portal.
2.  Click **Delete** on the top toolbar.
3.  Confirm the deletion by typing the database name and clicking **Delete**.

**5.2. Azure CLI Method**

1.  ```bash
    RESOURCE_GROUP="rg-sqldb-lab"
    SQL_SERVER="sqldbserver-youruniqueid"
    RESTORED_DB_NAME="app-db-restored"
    az sql db delete \
      --resource-group $RESOURCE_GROUP \
      --server $SQL_SERVER \
      --name $RESTORED_DB_NAME \
      --yes # Acknowledge prompt
    echo "Database '$RESTORED_DB_NAME' deletion initiated."
    ```

**5.3. Azure PowerShell Method**

1.  ```powershell
    $resourceGroupName = "rg-sqldb-lab"
    $sqlServerName = "sqldbserver-youruniqueid"
    $restoredDbName = "app-db-restored"
    Remove-AzSqlDatabase -ResourceGroupName $resourceGroupName `
      -ServerName $sqlServerName `
      -DatabaseName $restoredDbName -Force
    Write-Host "Database '$restoredDbName' deletion initiated."
    ```

-----

You have successfully performed a point-in-time restore for your Azure SQL Database\! This demonstrates how Azure's automated backups provide robust recovery capabilities.