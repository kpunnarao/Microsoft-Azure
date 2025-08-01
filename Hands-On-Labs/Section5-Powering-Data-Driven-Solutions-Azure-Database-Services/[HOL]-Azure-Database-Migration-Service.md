# Lab: Azure Database Migration Service - MySQL Offline Migration ➡️☁️

 This lab will guide you through migrating a small, local MySQL database to your Azure Database for MySQL - Flexible Server using DMS. We'll focus on an **offline migration** for simplicity, which is common for initial learning.

## Lab Scenario 🧪

You have an existing on-premises (simulated locally) MySQL database with some basic data. 

Your goal is to migrate this database to your Azure Database for MySQL - Flexible Server using Azure Database Migration Service (DMS).

This lab will demonstrate the core steps of setting up a migration project, initiating data transfer, and verifying the migration.

## Part 1: Prerequisites

  * An active Azure subscription.
  * **Azure Database for MySQL - Flexible Server**: You should have one created from Day 26's lab (e.g., `mymysqlflexserver-youruniqueid`). Ensure its **Public access** is configured to allow connections from your current public IP address.
  * **Local MySQL Server**: A running MySQL instance on your local machine.
      * **Recommended for simplicity**: Use [Docker Desktop](https://www.docker.com/products/docker-desktop/) with a MySQL container, or a pre-packaged solution like [XAMPP](https://www.apachefriends.org/index.html) (Windows/Linux) / [MAMP](https://www.mamp.info/en/mamp/mac/) (macOS) which bundles MySQL.
  * **MySQL Client Tool**: Such as [MySQL Workbench](https://www.mysql.com/products/workbench/) or the `mysql` command-line client to interact with your local and Azure MySQL servers.
  * **Azure Cloud Shell / Azure CLI**: For managing Azure resources.

## Part 2: Prepare Your Local (Source) MySQL Database

First, let's create a simple database and table on your local MySQL instance.

1.  **Start your Local MySQL Server**: Ensure your local MySQL server (Docker, XAMPP, MAMP) is running.

2.  **Connect to your Local MySQL Server**:

      * **Using `mysql` command-line (if installed and configured for your local MySQL):**
        ```bash
        mysql -u root -p # Or your local MySQL admin user
        ```
        (Enter your local MySQL root password when prompted.)
      * **Using MySQL Workbench:**
          * Create a new connection to `localhost:3306` (or your local MySQL port) using your local admin credentials.

3.  **Create a Sample Database and Table**:
    Execute the following SQL commands in your MySQL client:

    ```sql
    CREATE DATABASE mylocaldb;
    USE mylocaldb;

    CREATE TABLE products (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(100) NOT NULL,
        price DECIMAL(10, 2) NOT NULL,
        description TEXT,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );

    INSERT INTO products (name, price, description) VALUES
    ('Laptop Pro', 1200.00, 'Powerful laptop for professionals'),
    ('Wireless Mouse', 25.50, 'Ergonomic wireless mouse'),
    ('Mechanical Keyboard', 99.99, 'RGB mechanical keyboard with tactile switches'),
    ('USB-C Hub', 49.00, 'Multi-port USB-C adapter'),
    ('External SSD 1TB', 150.00, 'Fast external solid state drive');

    SELECT * FROM products;
    ```

    Verify that the `products` table is created and populated with data.

4.  **Note Local MySQL Connection Details**:

      * **Hostname**: Typically `localhost` or `127.0.0.1`
      * **Port**: `3306` (default)
      * **Username**: `root` (or your chosen admin user)
      * **Password**: Your local MySQL root password
      * **Database Name**: `mylocaldb`

## Part 3: Configure Azure Database Migration Service (DMS)

We will use the Azure Portal for this part as it provides a guided workflow.

1.  **Sign in to the [Azure Portal](https://portal.azure.com/)**.

2.  **Create a DMS Instance**:

      * In the search bar, type `Azure Database Migration Service` and select it.
      * Click **+ Create**.
      * **Basics tab**:
          * **Subscription**: Select your subscription.
          * **Resource group**: Create new: `rg-dms-lab` (or use an existing one).
          * **Migration service name**: Enter a unique name (e.g., `mydmsinstance-youruniqueid`).
          * **Location**: Choose the **same region as your Azure Database for MySQL - Flexible Server**. This is crucial for optimal performance and network connectivity.
          * **Service mode**: Select `Offline migration (Standard SKU)` for this lab. (For zero-downtime migrations, you'd choose online and a Premium SKU).
          * **Pricing tier**: Select `Standard (S1)` - this is the minimum for MySQL migrations.
      * **Networking tab**:
          * **Virtual network**: Click `Create new`. Provide a name (e.g., `vnet-dms-lab`) and a subnet name (e.g., `default`).
          * **Important**: DMS needs to connect to both your source (local MySQL) and target (Azure MySQL). This VNet will contain the DMS service. For your local MySQL, DMS will use your public IP if your local MySQL is accessible over the internet (or you'd use VPN/ExpressRoute). Ensure your Azure MySQL Flexible Server's firewall allows the DMS VNet's outbound IP (or its own Public Access IP range if using public endpoints). For simplicity, ensure your Azure MySQL Flexible Server is set to **Public Access (allowed IP addresses)** and your **current public IP** is allowed.
      * Click **Review + create**, then **Create**. This deployment can take 10-15 minutes.

3.  **Create a Migration Project**:

      * Once the DMS instance deployment is complete, go to its resource.
      * On the DMS instance overview page, click **+ New migration project**.
      * **Project type**:
          * **Project name**: `migrate-mylocaldb`
          * **Source server type**: `MySQL`
          * **Target server type**: `Azure Database for MySQL`
          * **Migration activity type**: `Offline data migration` (chosen in previous step)
      * Click **Create and run activity**.

4.  **Specify Source Details (Step 1 of Migration Activity)**:

      * **Source connection mode**: `IP Address`
      * **Server name**: Enter your **local MySQL Hostname** (e.g., `localhost` or your machine's public IP if accessible from Azure, or its private IP if using VPN). For a local setup without complex networking, you might need to use your machine's **public IP address** and ensure your local router/firewall forwards port 3306.
      * **Server port**: `3306`
      * **Username**: Your local MySQL admin username (e.g., `root`)
      * **Password**: Your local MySQL admin password
      * Click **Test connection**. If successful, click **Save**.

5.  **Specify Target Details (Step 2 of Migration Activity)**:

      * **Server name**: Enter the **Fully Qualified Domain Name (FQDN)** of your Azure Database for MySQL - Flexible Server (e.g., `mymysqlflexserver-youruniqueid.mysql.database.azure.com`).
      * **Server port**: `3306`
      * **Username**: Your Azure MySQL admin username (e.g., `myadmin`)
      * **Password**: Your Azure MySQL admin password
      * Click **Test connection**. If successful, click **Save**.

6.  **Select Databases (Step 3 of Migration Activity)**:

      * From the **Source databases** list, select `mylocaldb`.
      * For **Target database name**, you can either choose an existing database on your Azure server or type `mylocaldb` to have DMS create it.
      * Click **Save**.

7.  **Select Tables (Step 4 of Migration Activity)**:

      * Review the tables identified (you should see `products`).
      * Ensure `Migrate only schema` is **unchecked** (we want schema and data).
      * Click **Save**.

8.  **Configure Migration Settings (Step 5 of Migration Activity)**:

      * **Activity name**: `MyLocalDBMigration` (or a descriptive name).
      * Review other settings, generally defaults are fine for a basic lab.
      * Click **Save**.

9.  **Summary (Step 6 of Migration Activity)**:

      * Review the summary of your migration project.
      * Click **Run migration**.

## Part 4: Monitor and Verify Migration

1.  **Monitor Progress**:

      * On the DMS activity page, you will see the migration status. It will go through phases like "Initializing," "Pending," "Running," and finally "Completed."
      * For offline migration, it will transfer the schema first, then the data.
      * Keep an eye on the **Data migration** status, it should show rows migrated.

2.  **Verify Data on Azure Database for MySQL**:

      * Once the migration activity shows "Completed," connect to your **Azure Database for MySQL - Flexible Server** using MySQL Workbench or your `mysql` command-line client.
      * **Using `mysql` command-line:**
        ```bash
        mysql -h <Azure_MySQL_FQDN> -u <Azure_MySQL_Admin_User> -p
        ```
        (Enter your Azure MySQL admin password.)
        Then:
        ```sql
        USE mylocaldb;
        SELECT * FROM products;
        ```
      * **Using MySQL Workbench:**
          * Connect to your Azure Database for MySQL Flexible Server.
          * In the Navigator pane, refresh the schemas. You should see `mylocaldb`.
          * Expand `mylocaldb`, then `Tables`. Right-click on `products` and select `Select Rows - Limit 1000`.
      * Verify that all 5 rows from your local `products` table have been successfully migrated to the `mylocaldb` database on your Azure server.

-----

## Part 5: Cleanup Resources

To avoid incurring ongoing costs, it's crucial to delete the DMS instance, and if you no longer need them, also the Azure Database for MySQL Flexible Server and your local setup.

**(Using Azure CLI - Recommended for comprehensive cleanup)**

1.  **Delete the DMS resource group**:

    ```bash
    az group delete --name rg-dms-lab --yes --no-wait
    ```

    This will delete the DMS instance and the VNet it created.

2.  **Delete the Azure MySQL Flexible Server resource group** (if you no longer need it, created in Day 26 lab):

    ```bash
    az group delete --name my-mysql-lab-rg --yes --no-wait
    ```

    (Replace `my-mysql-lab-rg` if you used a different name.)

3.  **Clean up Local MySQL**:

      * If you used Docker: `docker stop <mysql_container_name>` and `docker rm <mysql_container_name>` (and `docker rmi <mysql_image_name>` if you want to remove the image).
      * If you used XAMPP/MAMP: Stop the MySQL service. You can optionally uninstall the software.

-----

Congratulations\! You have successfully performed an offline database migration from a local MySQL instance to Azure Database for MySQL - Flexible Server using Azure Database Migration Service. This experience gives you a practical understanding of how data migration works in Azure.