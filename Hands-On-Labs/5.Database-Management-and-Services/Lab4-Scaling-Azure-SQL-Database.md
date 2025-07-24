# Lab: Scaling Azure SQL Database (vCores & Auto-Pause) 📈

Excellent\! Let's get hands-on with scaling. This lab will demonstrate how to adjust the compute resources for your Azure SQL Database and explore the cost-saving auto-pause feature of the Serverless tier.

**Important Note before starting:** This lab assumes you still have the `rg-sqldb-lab` resource group, `sqldbserver-youruniqueid` logical server, and `app-db` database from our previous labs. If you deleted them, please re-create them quickly using the Azure Portal, CLI, or PowerShell methods from **3.[HOL]-Creating-an-Azure-SQL-Database**.

## Lab Scenario 🧪

You need to adjust the performance of your `app-db` to handle changing workloads. 

You'll practice scaling it to a different vCore configuration and, if you're using the Serverless tier, observe its auto-pausing behavior for cost optimization.

## Part 1: Prerequisites

  * An active Azure subscription.
  * The `rg-sqldb-lab` resource group, `sqldbserver-youruniqueid` logical server, and `app-db` database (preferably in the **General Purpose, Serverless** tier) must exist.
  * Azure CLI or Azure PowerShell installed and logged in.
  * SQL Server Management Studio (SSMS) or Azure Data Studio to connect and monitor.

## Part 2: Scaling the Database

We'll focus on the vCore model since it's the recommended modern approach. We'll specifically look at scaling a Serverless database, which is often chosen for its scaling flexibility.

**(Choose one method: Azure Portal, Azure CLI, or Azure PowerShell)**

**2.1. Azure Portal Method**

1.  Sign in to the [Azure Portal](https://portal.azure.com/).

2.  In the search bar, type `SQL databases` and select it.

3.  Click on your database: `app-db`.

4.  In the left-hand menu, under **Settings**, click on **Compute + storage**.

5.  You will see your current **Service tier** (e.g., General Purpose) and **Compute tier** (e.g., Serverless).

6.  **To Scale Up/Down (vCores):**

      * Click on the **Configure database** link.
      * You can adjust the **Max vCores** slider. For example, increase it from 4 vCores to 6 vCores, or decrease it if desired (e.g., to 2 vCores). Note the corresponding change in price.
      * If you wanted to switch to a **Provisioned** compute tier, you could select that option here.
      * Click **Apply**.
      * The portal will show a notification that the database is being updated. This process typically takes **less than 5 minutes** for single databases. During this time, you might experience a brief connection drop if you are connected via SSMS/ADS.
      * Once complete, refresh the overview page to see the updated vCore count.

7.  **To Configure Auto-Pause Delay (for Serverless only):**

      * If your database is in the **Serverless** tier, on the **Compute + storage** blade, you'll see "Serverless" details.
      * Locate **Auto-pause delay**. The default is 60 minutes.
      * Click on it and change the value to something shorter for testing, e.g., **15 minutes** (the minimum).
      * Click **Apply**.

**2.2. Azure CLI Method**

1.  Open your terminal or Cloud Shell.

2.  Set variables:

    ```bash
    RESOURCE_GROUP="rg-sqldb-lab"
    SQL_SERVER="sqldbserver-youruniqueid" # Your unique server name
    DB_NAME="app-db"
    ```

3.  **To Scale Up/Down (vCores):**

      * To scale to 6 max vCores (Serverless, General Purpose, Gen5):
        ```bash
        az sql db update \
          --resource-group $RESOURCE_GROUP \
          --server $SQL_SERVER \
          --name $DB_NAME \
          --max-vcores 6 \
          --service-objective GP_S_Gen5_6 # This is the Service Level Objective for 6 max vCores, Serverless, Gen5
        ```
      * To scale to 2 max vCores (Serverless, General Purpose, Gen5):
        ```bash
        az sql db update \
          --resource-group $RESOURCE_GROUP \
          --server $SQL_SERVER \
          --name $DB_NAME \
          --max-vcores 2 \
          --service-objective GP_S_Gen5_2
        ```
      * Observe the output. The command will return, and the scaling operation will continue in the background. You can check its status in the Azure portal or use `az sql db show --resource-group $RESOURCE_GROUP --server $SQL_SERVER --name $DB_NAME --query serviceLevelObjective`

4.  **To Configure Auto-Pause Delay (for Serverless only):**

      * To set auto-pause delay to 15 minutes (default is 60 minutes):
        ```bash
        az sql db update \
          --resource-group $RESOURCE_GROUP \
          --server $SQL_SERVER \
          --name $DB_NAME \
          --auto-pause-delay 15
        ```
      * To disable auto-pause (database always stays online):
        ```bash
        az sql db update \
          --resource-group $RESOURCE_GROUP \
          --server $SQL_SERVER \
          --name $DB_NAME \
          --auto-pause-delay -1
        ```

**2.3. Azure PowerShell Method**

1.  Open PowerShell.

2.  Set variables:

    ```powershell
    $resourceGroupName = "rg-sqldb-lab"
    $sqlServerName = "sqldbserver-youruniqueid" # Your unique server name
    $dbName = "app-db"
    ```

3.  **To Scale Up/Down (vCores):**

      * To scale to 6 max vCores (Serverless, General Purpose, Gen5):
        ```powershell
        Set-AzSqlDatabase -ResourceGroupName $resourceGroupName `
          -ServerName $sqlServerName `
          -DatabaseName $dbName `
          -Edition "GeneralPurpose" `
          -ComputeModel "Serverless" `
          -ComputeGeneration "Gen5" `
          -VCore 6
        ```
      * To scale to 2 max vCores (Serverless, General Purpose, Gen5):
        ```powershell
        Set-AzSqlDatabase -ResourceGroupName $resourceGroupName `
          -ServerName $sqlServerName `
          -DatabaseName $dbName `
          -Edition "GeneralPurpose" `
          -ComputeModel "Serverless" `
          -ComputeGeneration "Gen5" `
          -VCore 2
        ```
      * The command will execute, and the scaling will happen in the background.

4.  **To Configure Auto-Pause Delay (for Serverless only):**

      * To set auto-pause delay to 15 minutes:
        ```powershell
        Set-AzSqlDatabase -ResourceGroupName $resourceGroupName `
          -ServerName $sqlServerName `
          -DatabaseName $dbName `
          -AutoPauseDelayInMinutes 15
        ```
      * To disable auto-pause:
        ```powershell
        Set-AzSqlDatabase -ResourceGroupName $resourceGroupName `
          -ServerName $sqlServerName `
          -DatabaseName $dbName `
          -AutoPauseDelayInMinutes -1
        ```

## Part 3: Observe Auto-Pause/Resume Behavior (for Serverless)

This part is crucial for understanding the Serverless tier's cost-saving features.

1.  **Ensure Auto-Pause is Enabled and Set to a Short Delay**: Make sure you have set the `Auto-pause delay` to **15 minutes** in Part 2.
2.  **Disconnect All Sessions**: Close all connections to `app-db` from SSMS, Azure Data Studio, or any applications. Ensure there are no active queries or background processes connected.
3.  **Wait for Auto-Pause**: Wait for at least the `Auto-pause delay` duration (e.g., 15 minutes) after disconnecting all sessions.
      * You can periodically check the database status in the Azure Portal. On the `app-db` **Overview** page, its status will eventually change to **Paused**.
4.  **Observe Auto-Resume**: Once the database is paused, try to connect to it again using SSMS or Azure Data Studio.
      * You will notice a **delay** (typically 10-30 seconds, sometimes longer for the first resume) while the database resumes. During this time, your client tool might show "Connecting..." or similar messages.
      * Once resumed, the database status in the portal will change back to **Online**.

## Part 4: Cleanup

Keep your `app-db` for the next labs. No specific cleanup for this lab, other than ensuring your auto-pause delay is set as desired.

-----

You have now successfully experienced scaling your Azure SQL Database\! You've seen how easy it is to adjust resources and understand how the Serverless tier can save costs by auto-pausing.