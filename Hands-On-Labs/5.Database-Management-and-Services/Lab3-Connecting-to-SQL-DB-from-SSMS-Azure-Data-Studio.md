# Lab: Connecting to Azure SQL DB from SSMS/Azure Data Studio:

Connecting to your Azure SQL Database from a local machine requires a few key pieces of information and one essential security setting.

This guide will assume you have already created your logical SQL server and a single database on it, as outlined in our previous lab.

### Prerequisites:
* An Azure SQL Database is already deployed.
* You have the server's admin username and password.
* Either **SQL Server Management Studio (SSMS)** or **Azure Data Studio** is installed on your local machine.

---

### Step 1: Get the Server Name

Your logical SQL server in Azure has a unique, fully qualified name that you must use to connect.

1.  Sign in to the [Azure Portal](https://portal.azure.com/).
2.  Navigate to your **SQL database** (e.g., `app-db`) or your **SQL server** (e.g., `sqldbserver-youruniqueid`).
3.  On the database's **Overview** page, look for the **Server name**. It will be in the format: `sqldbserver-youruniqueid.database.windows.net`.
4.  **Copy this entire server name** to your clipboard. This is what you'll use in your client tool.

### Step 2: Configure the Firewall Rule

By default, Azure SQL Database firewalls block all public access to the server. You must create a firewall rule to allow connections from your local machine's public IP address.

1.  In the Azure Portal, go to your **SQL server** resource (not the database).
2.  In the left-hand menu, under **Security**, select **Networking**.
3.  On the **Public access** tab, ensure **Public network access** is set to `Selected networks`.
4.  Under the **Firewall rules** section, click **Add your client IPv4 address** on the toolbar. This will automatically detect your current public IP address and create a new rule with a name like "ClientIPRule_2025-07-24".
5.  Click **Save**. It may take a few moments for the firewall rule to take effect.

> **Note**: If your machine's IP address changes (e.g., if you are on a dynamic IP network), you will need to come back to this page and add the new IP address. For static IP addresses or specific IP ranges, you can manually add a new rule.

---

### Step 3: Connect Using Your Client Tool

Now that you have the server name and the firewall is configured, you can connect using your chosen tool.

#### **Connecting from SQL Server Management Studio (SSMS)**

1.  Open SSMS.
2.  The **Connect to Server** dialog box should appear automatically. If not, click the **Connect** button in the Object Explorer pane and select **Database Engine**.
3.  Fill in the details:
    * **Server name**: Paste the full server name you copied earlier (e.g., `sqldbserver-youruniqueid.database.windows.net`).
    * **Authentication**: Select **SQL Server Authentication**.
    * **Login**: Enter your server admin username (e.g., `azureadmin`).
    * **Password**: Enter the password you created for the server admin.
4.  Click **Connect**.

#### **Connecting from Azure Data Studio**

1.  Open Azure Data Studio.
2.  Click the **New Connection** icon in the **SERVERS** sidebar.
3.  Fill in the details:
    * **Server**: Paste the full server name (e.g., `sqldbserver-youruniqueid.database.windows.net`).
    * **Authentication type**: Select **SQL Login**.
    * **User name**: Enter your server admin username (e.g., `azureadmin`).
    * **Password**: Enter the password you created.
4.  Click **Connect**.

### Troubleshooting Common Connection Errors:

* **"Login failed for user..."**: This usually means the username or password you entered is incorrect. Double-check your credentials.
* **"A network-related or instance-specific error occurred..."**: This is a very common error and almost always means your public IP address is not included in the server's firewall rules. Go back to Step 2 and ensure your current IP is added.
* **"Server not found..."**: Ensure the server name is spelled correctly and includes the `.database.windows.net` suffix.
* **"Connection timeout expired..."**: If the connection fails after a long delay, it could be a firewall issue, a local network issue, or a slow connection. Double-check the firewall first.

You have now successfully completed the final section of our planned Azure SQL Database basics. 

This knowledge will be crucial for any work you do with relational databases in Azure.