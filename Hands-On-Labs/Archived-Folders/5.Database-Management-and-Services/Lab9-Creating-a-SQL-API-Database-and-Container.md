# Lab 2: Creating a SQL API Database and Container 🗃️📦

## Lab Scenario 🧪

With your Azure Cosmos DB account ready, the next step is to create a logical database and then a container within it. 

The container is where your actual data (JSON documents) will live. We will also define a **partition key**, which is crucial for scalability and performance in Cosmos DB.

## Part 1: Prerequisites

  * An active Azure subscription.
  * The Azure Cosmos DB account created in Lab 1 (e.g., `cosmosdb-youruniqueid`), configured with the **NoSQL API** and **Serverless** capacity mode.
  * Azure CLI or Azure PowerShell installed and logged in (if you plan to use those methods).

## Part 2: Create a Database and Container

For the NoSQL API, data is organized hierarchically: **Account \> Database \> Container (Collection) \> Items (Documents)**.

**(Choose one method: Azure Portal, Azure CLI, or Azure PowerShell)**

### 2.1. Azure Portal Method

1.  Sign in to the [Azure Portal](https://portal.azure.com/).
2.  Navigate to your Azure Cosmos DB account (e.g., `cosmosdb-youruniqueid`).
3.  In the left-hand menu, under **Resources**, click **Data Explorer**.
4.  On the Data Explorer blade, click **New Container** from the top menu.
5.  In the **New Container** dialog:
      * **Database id**:
          * Select **Create new** and enter `appdb` (This will create the database).
      * **Container id**: Enter `products`.
      * **Partition key**: Enter `/category`.
          * *Explanation*: For a simple `Products` collection, `/category` is a reasonable choice for a partition key. It implies that items with the same `category` value (e.g., "Electronics", "Clothing") will be stored together on the same logical partition. Choosing an effective partition key is crucial for performance and scalability in Cosmos DB, ensuring even data distribution and minimizing cross-partition operations for common queries.
      * **Throughput**: Since your account is **Serverless**, the throughput options (Manual or Autoscale) are greyed out, as throughput scales automatically based on consumption.
      * **Unique keys**: Leave blank.
      * **Analytical store**: Leave as `Off`.
6.  Click **OK**.
7.  The portal will show a notification that the container is being created. This usually takes a few seconds.
8.  Once created, you will see `appdb` and `products` under the **Data Explorer** tree. You can expand `products` to see **Items**, **Scale & Settings**, etc.

### 2.2. Azure CLI Method

1.  Open your terminal or Cloud Shell.

2.  Set environment variables (ensure `COSMOS_ACCOUNT_NAME` matches your account):

    ```bash
    RESOURCE_GROUP="rg-cosmosdb-lab" # Or your existing resource group name
    COSMOS_ACCOUNT_NAME="cosmosdb-youruniqueid" # Your unique Cosmos DB account name
    DATABASE_NAME="appdb"
    CONTAINER_NAME="products"
    PARTITION_KEY_PATH="/category"
    ```

3.  Create the database (if it doesn't already exist):

    ```bash
    az cosmosdb sql database create \
      --account-name $COSMOS_ACCOUNT_NAME \
      --resource-group $RESOURCE_GROUP \
      --name $DATABASE_NAME \
      --output table
    ```

      * *Note*: For Serverless accounts, you don't specify RUs for the database level directly.

4.  Create the container within the database:

    ```bash
    az cosmosdb sql container create \
      --account-name $COSMOS_ACCOUNT_NAME \
      --resource-group $RESOURCE_GROUP \
      --database-name $DATABASE_NAME \
      --name $CONTAINER_NAME \
      --partition-key-path $PARTITION_KEY_PATH \
      --idx-policy "Automatic" \
      --output table
    ```

      * `--partition-key-path`: Specifies the property in your JSON documents that will serve as the partition key.
      * `--idx-policy "Automatic"`: Confirms automatic indexing for all properties, which is the default for NoSQL API.
      * *Note*: Again, for Serverless accounts, you don't specify `--throughput` (RUs) here.

5.  The commands will output the details of the created database and container.

### 2.3. Azure PowerShell Method

1.  Open PowerShell.

2.  Set environment variables (ensure `$cosmosAccountName` matches your account):

    ```powershell
    $resourceGroupName = "rg-cosmosdb-lab"
    $cosmosAccountName = "cosmosdb-youruniqueid" # Your unique Cosmos DB account name
    $databaseName = "appdb"
    $containerName = "products"
    $partitionKeyPath = "/category"
    ```

3.  Create the database (if it doesn't already exist):

    ```powershell
    New-AzCosmosDBSqlDatabase -ResourceGroupName $resourceGroupName `
      -AccountName $cosmosAccountName `
      -Name $databaseName
    ```

4.  Create the container within the database:

    ```powershell
    $containerParameters = @{
        Name = $containerName
        PartitionKeyPath = $partitionKeyPath
        DefaultTtl = -1 # -1 means no default TTL
    }

    New-AzCosmosDBSqlContainer -ResourceGroupName $resourceGroupName `
      -AccountName $cosmosAccountName `
      -DatabaseName $databaseName `
      -Name $containerName `
      -PartitionKeyPath $partitionKeyPath `
      -DefaultTtl -1 # No default TTL
    ```

      * *Note*: Similar to CLI, for Serverless accounts, you do not specify `-Throughput` or `-AutoscaleThroughput`.

5.  The commands will output the details of the created database and container.

-----

## Part 3: Verify Database and Container Creation

1.  **Azure Portal**: Go to your Cosmos DB account, then **Data Explorer**. You should see `appdb` expanded, with `products` nested under it.
2.  You can click on **products** then **Items** to see that the container is empty, awaiting your data.

-----

## Part 4: Cleanup (Optional)

You have now successfully created a Cosmos DB database and container. You will need these resources for future labs where we will interact with the data.

When you are completely finished with all Azure Cosmos DB labs, remember to delete the entire `rg-cosmosdb-lab` resource group (or whichever resource group you used for Cosmos DB) to avoid ongoing costs.

-----

Excellent work\! You've successfully provisioned your Cosmos DB account, database, and a container. 

This sets the stage for adding and querying data.