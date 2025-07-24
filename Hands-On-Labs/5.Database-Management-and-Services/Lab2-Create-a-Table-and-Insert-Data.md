## Lab: Create a Table and Insert Data:

Now that you have a database, the next step is to create a table and add some data. 

This will give you a complete, functional database to work with.


In this section, you will use Transact-SQL (T-SQL) commands to create a simple table and populate it with some sample data.

1.  **Open a New Query Window**:

      * In SSMS or Azure Data Studio, navigate to your database in the **Object Explorer** pane.
      * Right-click on your database (`app-db`), and select **New Query**. A new query window will open.

2.  **Create a Table**:

      * Copy and paste the following T-SQL code into the query window. This script creates a table named `Products` to store information about products.
      * Click **Execute** to run the command.

    <!-- end list -->

    ```sql
    CREATE TABLE Products (
        ProductID INT PRIMARY KEY IDENTITY(1,1),
        Name NVARCHAR(50) NOT NULL,
        Category NVARCHAR(50),
        Price DECIMAL(18, 2)
    );
    ```

3.  **Insert Data**:

      * Now, copy and paste the following T-SQL code to insert three sample products into your new table.
      * Click **Execute**.

    <!-- end list -->

    ```sql
    INSERT INTO Products (Name, Category, Price) VALUES
    ('Laptop', 'Electronics', 1200.00),
    ('Mouse', 'Electronics', 25.50),
    ('Keyboard', 'Electronics', 75.00);
    ```

4.  **Verify the Data**:

      * To confirm that the data was inserted correctly, run the following `SELECT` statement.
      * Click **Execute**. You should see the three rows you just inserted in the results pane.

    <!-- end list -->

    ```sql
    SELECT * FROM Products;
    ```

-----

## Part 4: Cleaning Up Resources (Reminder)

Remember to delete all the resources you created to avoid any unexpected costs. You can refer back to the cleanup instructions in the previous response.

  * **Resource Group**: `rg-sqldb-lab`

-----

You have now successfully created a server, a database, a table, and inserted data. 

This completes the core lab. 