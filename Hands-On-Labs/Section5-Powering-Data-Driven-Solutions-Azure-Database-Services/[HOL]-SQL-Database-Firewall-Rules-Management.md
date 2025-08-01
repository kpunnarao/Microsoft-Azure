# Lab: SQL Database Firewall Rules Management 🛡️🚪

Understanding and managing firewall rules is fundamental to securing your Azure SQL Database. 

This lab will give you direct experience with controlling network access.

## Lab Scenario 🧪

Controlling network access to your Azure SQL Database is a critical security measure.

In this lab, you will actively manage server-level firewall rules, testing the impact of allowing and blocking your client IP address. 

You'll also explore the "Allow Azure services" option.

## Part 1: Prerequisites

  * An active Azure subscription.
  * The `rg-sqldb-lab` resource group and `sqldbserver-youruniqueid` logical server from previous labs must exist.
  * SQL Server Management Studio (SSMS) or Azure Data Studio connected to your `app-db` (or able to connect).
  * Your current public IP address (you can find this by searching "what is my ip" on Google or using `curl https://ifconfig.me` in your terminal).

## Part 2: Test Current Connectivity

Before making changes, let's ensure you can currently connect.

1.  **Open SSMS or Azure Data Studio**:
      * Attempt to connect to your SQL server (`sqldbserver-youruniqueid.database.windows.net`) using your `azureadmin` credentials.
      * You should be able to connect successfully, as your IP was added in Lab 1.
2.  **Keep this connection window open (or be ready to reconnect quickly) for testing.**

-----

## Part 3: Remove Your IP from Firewall Rules (Simulate Block)

We will now remove the existing firewall rule that allows your IP address. This will simulate what happens if an unauthorized IP tries to connect or if your IP changes and is no longer allowed.

**(Choose one method: Azure Portal, Azure CLI, or Azure PowerShell)**

**3.1. Azure Portal Method**

1.  Sign in to the [Azure Portal](https://portal.azure.com/).
2.  Navigate to your **SQL server** (`sqldbserver-youruniqueid`).
3.  In the left-hand menu, under **Security**, select **Networking**.
4.  Under the **Firewall rules** section, find the rule that corresponds to your IP address (it might be named "AllowMyIP" or "ClientIPRule\_...") and note its name.
5.  Click the **"..." (More options)** next to that rule and select **Delete**.
6.  Confirm the deletion by clicking **Yes**.
7.  Click **Save** at the top of the networking blade.

**3.2. Azure CLI Method**

1.  Open your terminal or Cloud Shell.
2.  Set variables:
    ```bash
    RESOURCE_GROUP="rg-sqldb-lab"
    SQL_SERVER="sqldbserver-youruniqueid" # Your unique server name
    FIREWALL_RULE_NAME="AllowMyIP" # Or the name of your specific IP rule (e.g., ClientIPRule_YYYY-MM-DD)
    ```
      * **Note**: To find the exact rule name if you didn't name it "AllowMyIP":
        ```bash
        az sql server firewall-rule list --resource-group $RESOURCE_GROUP --server $SQL_SERVER --query "[].name" -o tsv
        ```
3.  Delete the firewall rule:
    ```bash
    az sql server firewall-rule delete \
      --resource-group $RESOURCE_GROUP \
      --server $SQL_SERVER \
      --name $FIREWALL_RULE_NAME \
      --output none
    echo "Firewall rule '$FIREWALL_RULE_NAME' deleted."
    ```

**3.3. Azure PowerShell Method**

1.  Open PowerShell.
2.  Set variables:
    ```powershell
    $resourceGroupName = "rg-sqldb-lab"
    $sqlServerName = "sqldbserver-youruniqueid" # Your unique server name
    $firewallRuleName = "AllowMyIP" # Or the name of your specific IP rule
    ```
      * **Note**: To find the exact rule name:
        ```powershell
        Get-AzSqlServerFirewallRule -ResourceGroupName $resourceGroupName -ServerName $sqlServerName | Select-Object Name
        ```
3.  Remove the firewall rule:
    ```powershell
    Remove-AzSqlServerFirewallRule -ResourceGroupName $resourceGroupName `
      -ServerName $sqlServerName `
      -FirewallRuleName $firewallRuleName -Force
    Write-Host "Firewall rule '$firewallRuleName' deleted."
    ```

-----

## Part 4: Test Blocked Connection

Now that the rule allowing your IP is removed, try to connect again.

1.  **From SSMS/Azure Data Studio**: Attempt to reconnect to your SQL server (`sqldbserver-youruniqueid.database.windows.net`).
2.  **Expected Result**: The connection should **fail**. You will typically see an error message similar to:
      * `"Cannot open server 'sqldbserver-youruniqueid' requested by the login. Client with IP address 'YourPublicIP' is not allowed to access the server."`
      * This confirms the firewall rule is working as intended.

-----

## Part 5: Re-add Your IP to Firewall Rules (Allow Access)

Let's re-enable access for your machine.

**(Choose one method: Azure Portal, Azure CLI, or Azure PowerShell)**

**5.1. Azure Portal Method**

1.  In the Azure Portal, navigate back to your **SQL server** (`sqldbserver-youruniqueid`) -\> **Networking** blade.
2.  Under **Firewall rules**, click **+ Add your client IPv4 address**.
3.  Click **Save** at the top.

**5.2. Azure CLI Method**

1.  Ensure `MY_IP` variable is set to your current public IP address (from Part 3.2, or run `MY_IP=$(curl -s https://ifconfig.me)` again if unsure).
2.  Create the firewall rule (you can use the same name or a new one):
    ```bash
    RESOURCE_GROUP="rg-sqldb-lab"
    SQL_SERVER="sqldbserver-youruniqueid"
    MY_IP=$(curl -s https://ifconfig.me) # Get current IP again
    FIREWALL_RULE_NAME="AllowMyIPAgain" # Can be same as before if it was deleted
    az sql server firewall-rule create \
      --resource-group $RESOURCE_GROUP \
      --server $SQL_SERVER \
      --name $FIREWALL_RULE_NAME \
      --start-ip-address $MY_IP \
      --end-ip-address $MY_IP \
      --output none
    echo "Firewall rule '$FIREWALL_RULE_NAME' for IP '$MY_IP' re-added."
    ```

**5.3. Azure PowerShell Method**

1.  Ensure `$myIp` variable is set to your current public IP address (from Part 3.3, or run `$myIp = (Invoke-RestMethod -Uri "https://api.ipify.org").Ip` again).
2.  Create the firewall rule:
    ```powershell
    $resourceGroupName = "rg-sqldb-lab"
    $sqlServerName = "sqldbserver-youruniqueid"
    $myIp = (Invoke-RestMethod -Uri "https://api.ipify.org").Ip # Get current IP again
    $firewallRuleName = "AllowMyIPAgain" # Can be same as before if it was deleted
    New-AzSqlServerFirewallRule -ResourceGroupName $resourceGroupName `
      -ServerName $sqlServerName `
      -FirewallRuleName $firewallRuleName `
      -StartIpAddress $myIp `
      -EndIpAddress $myIp
    Write-Host "Firewall rule '$firewallRuleName' for IP '$myIp' re-added."
    ```

-----

## Part 6: Verify Re-enabled Connection

1.  **From SSMS/Azure Data Studio**: Attempt to reconnect to your SQL server (`sqldbserver-youruniqueid.database.windows.net`).
2.  **Expected Result**: The connection should now be **successful**.

-----

## Part 7: Explore "Allow Azure services and resources to access this server" (Optional, Highly Recommended to Understand)

This setting can be convenient but also a security risk if not understood.

1.  **In the Azure Portal**, navigate to your **SQL server** (`sqldbserver-youruniqueid`) -\> **Networking** blade.
2.  Under **Firewall rules**, find the setting **"Allow Azure services and resources to access this server"**.
3.  Toggle it to **Yes** and click **Save**.
4.  **Understand the Implication**: When this is `Yes`, any Azure service (e.g., an App Service, Function App, or even a SQL database in another subscription with different ownership) can connect to your SQL server *without needing an IP firewall rule*. This means traffic originates from internal Azure IP ranges that are not public.
5.  **Security Best Practice**: For production, this setting is generally **discouraged** due to its broad nature. Instead, use **Virtual Network Service Endpoints** or **Azure Private Link** for secure, private connectivity from specific Azure resources or VNets.
6.  **Toggle it back to No** immediately after observing/understanding, and click **Save**, unless you specifically intend to use it for a non-production scenario.

-----

## Part 8: Cleanup

Keep your `app-db` and `sqldbserver-youruniqueid` for the next lab. Ensure that your client IP is allowed via a firewall rule if you plan to continue using SSMS/ADS.

-----

You have successfully managed SQL Database Firewall Rules\! You now have a practical understanding of how to control network access to your Azure SQL Database, which is a fundamental security skill.