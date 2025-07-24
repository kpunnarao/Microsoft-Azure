# Lab: Azure Cache for Redis - Create and Connect 💾⚡

## Lab Scenario 🧪

In this lab, you will provision an Azure Cache for Redis instance, which is a fully managed, in-memory data store. You will then retrieve its connection credentials and use a simple Python application to connect to the cache, store a key-value pair, and retrieve it, demonstrating the basic functionality of a distributed cache.

## Part 1: Prerequisites

  * An active Azure subscription.
  * Azure CLI or Azure PowerShell installed and logged in (if you plan to use those methods).
  * **Python 3.x** installed on your local machine.
  * A text editor or IDE for writing Python code (e.g., VS Code).

## Part 2: Create Azure Cache for Redis Instance

We will create a **Basic C0 or C1** tier cache for this lab, as it's cost-effective for development and testing.

**Important Considerations for Cache Naming**:

  * Your cache name must be **globally unique** across all of Azure.
  * It must be between 1 and 63 characters long.
  * It can contain only lowercase letters, numbers, and hyphens.

**(Choose one method: Azure Portal, Azure CLI, or Azure PowerShell)**

### 2.1. Azure Portal Method

1.  Sign in to the [Azure Portal](https://portal.azure.com/).
2.  In the Azure portal search bar, type `Azure Cache for Redis` and select **Azure Cache for Redis** from the results.
3.  On the Azure Cache for Redis page, click **+ Create**.
4.  On the **New Redis Cache** blade, fill in the following details on the **Basics** tab:
      * **Subscription**: Select your Azure subscription.
      * **Resource Group**: Click `Create new` and enter `rg-redis-lab`, then click **OK**. (Or choose an existing one).
      * **DNS name**: Enter a **globally unique** name for your Redis Cache (e.g., `myredis-youruniqueid`).
      * **Location**: Select a region close to you (e.g., `East US`, `West Europe`, `Southeast Asia`).
      * **Cache type**: Select **Basic**.
      * **Cache size**: Select `C0 (250 MB)` or `C1 (1 GB)` (C0 is the cheapest, C1 offers slightly more capacity for minimal cost difference).
5.  Click **Review + create**.
6.  Review the settings. Once validation passes, click **Create**.
7.  The deployment will start and can take a few minutes to complete. Wait for the portal to display "Your deployment is complete."
8.  Click **Go to resource** to navigate to your new Azure Cache for Redis instance.

### 2.2. Azure CLI Method

1.  Open your terminal or Cloud Shell (shell.azure.com).
2.  Set environment variables. Remember to replace `youruniqueid` with a unique string.
    ```bash
    RESOURCE_GROUP="rg-redis-lab"
    LOCATION="eastus" # Choose a region close to you
    REDIS_CACHE_NAME="myredis-youruniqueid" # Must be globally unique and lowercase
    CACHE_SKU="Basic"
    CACHE_FAMILY="C" # C for Basic/Standard, P for Premium
    CACHE_CAPACITY="0" # 0 for C0 (250MB), 1 for C1 (1GB) etc.
    ```
3.  (Optional) Create the resource group if it doesn't exist:
    ```bash
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```
4.  Create the Azure Cache for Redis instance:
    ```bash
    az redis create \
      --resource-group $RESOURCE_GROUP \
      --name $REDIS_CACHE_NAME \
      --location $LOCATION \
      --sku $CACHE_SKU \
      --vm-family $CACHE_FAMILY \
      --capacity $CACHE_CAPACITY \
      --output table
    ```
      * The command will output the details of the created cache. This process may take a few minutes.

### 2.3. Azure PowerShell Method

1.  Open Azure PowerShell (or your local terminal with Azure PowerShell module installed and logged in).
2.  Set environment variables. Remember to replace `youruniqueid` with a unique string.
    ```powershell
    $resourceGroupName = "rg-redis-lab"
    $location = "eastus"
    $redisCacheName = "myredis-youruniqueid"
    $cacheSku = "Basic"
    $cacheFamily = "C" # C for Basic/Standard, P for Premium
    $cacheCapacity = 0 # 0 for C0 (250MB), 1 for C1 (1GB) etc.
    ```
3.  (Optional) Create the resource group if it doesn't exist:
    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location $location
    ```
4.  Create the Azure Cache for Redis instance:
    ```powershell
    New-AzRedisCache -ResourceGroupName $resourceGroupName `
      -Name $redisCacheName `
      -Location $location `
      -Sku $cacheSku `
      -VmSize $cacheFamily `
      -Capacity $cacheCapacity `
      -EnableNonSslPort $false # Recommended to disable non-SSL port for security
    ```
      * The command will output the details of the created cache. This process may take a few minutes.

## Part 3: Get Connection Information and Access Keys

You'll need the hostname and access keys to connect your application to the Redis Cache.

### 3.1. Azure Portal Method

1.  Navigate to your Azure Cache for Redis instance in the Azure Portal.
2.  In the left-hand menu, under **Settings**, click **Properties**. Note the **Host name**.
3.  In the left-hand menu, under **Settings**, click **Access keys**.
4.  Copy the **Primary connection string** or the **Primary key**. You'll typically use the host name, port, and the key separately in your code.
      * **Host name**: (from Properties) e.g., `myredis-youruniqueid.redis.cache.windows.net`
      * **SSL Port**: `6380` (default for SSL connections)
      * **Non-SSL Port**: `6379` (if enabled, not recommended for production)
      * **Primary/Secondary Key**: The actual access key string.

### 3.2. Azure CLI Method

```bash
# Get Hostname
az redis show \
  --resource-group $RESOURCE_GROUP \
  --name $REDIS_CACHE_NAME \
  --query "hostName" --output tsv

# Get Primary Access Key
az redis list-keys \
  --resource-group $RESOURCE_GROUP \
  --name $REDIS_CACHE_NAME \
  --query "primaryKey" --output tsv
```

### 3.3. Azure PowerShell Method

```powershell
# Get Hostname
(Get-AzRedisCache -ResourceGroupName $resourceGroupName -Name $redisCacheName).HostName

# Get Primary Access Key
(Get-AzRedisCacheKey -ResourceGroupName $resourceGroupName -Name $redisCacheName).PrimaryKey
```

## Part 4: Connect and Interact with Redis using Python

Now, let's write a simple Python script to connect to your Redis Cache and perform basic `SET` and `GET` operations.

1.  **Install the Redis Python Client**:
    Open your terminal or command prompt and run:

    ```bash
    pip install redis
    ```

2.  **Create a Python script**:
    Create a new file named `redis_test.py` (or any other name you prefer) and paste the following code into it:

    ```python
    import os
    import redis
    import ssl

    # --- Configuration ---
    # Get these values from Part 3 (Azure Portal, CLI, or PowerShell)
    REDIS_HOST = os.environ.get("REDIS_HOST", "YOUR_REDIS_HOSTNAME") # e.g., myredis-youruniqueid.redis.cache.windows.net
    REDIS_PORT = int(os.environ.get("REDIS_PORT", 6380)) # Use 6380 for SSL
    REDIS_PASSWORD = os.environ.get("REDIS_PASSWORD", "YOUR_REDIS_PRIMARY_KEY") # Your primary access key

    # --- Connect to Redis ---
    try:
        # For Azure Cache for Redis, always use SSL (port 6380)
        r = redis.StrictRedis(host=REDIS_HOST,
                              port=REDIS_PORT,
                              password=REDIS_PASSWORD,
                              ssl=True,
                              ssl_cert_reqs=ssl.CERT_NONE) # Use CERT_NONE for simplicity in lab, CERT_REQUIRED for production

        print(f"Successfully connected to Redis at {REDIS_HOST}:{REDIS_PORT}")

        # --- Perform Redis Operations ---
        key = "mykey"
        value = "Hello from Azure Cache for Redis!"

        # 1. Set a key-value pair
        r.set(key, value)
        print(f"Set '{key}': '{value}'")

        # 2. Get the value back
        retrieved_value = r.get(key)
        if retrieved_value:
            print(f"Retrieved '{key}': '{retrieved_value.decode('utf-8')}'")
        else:
            print(f"Key '{key}' not found.")

        # 3. Increment a counter
        counter_key = "mycounter"
        r.incr(counter_key)
        r.incr(counter_key)
        print(f"Incremented '{counter_key}', current value: {r.get(counter_key).decode('utf-8')}")

        # 4. Add to a list
        list_key = "mylist"
        r.rpush(list_key, "item1", "item2", "item3")
        print(f"Added items to '{list_key}', current list: {r.lrange(list_key, 0, -1)}")

        # Clean up (optional)
        r.delete(key, counter_key, list_key)
        print("Cleaned up keys.")

    except redis.exceptions.ConnectionError as e:
        print(f"Error connecting to Redis: {e}")
        print("Please ensure your Redis host, port, and password are correct and your IP is allowed by firewall rules.")
    except Exception as e:
        print(f"An unexpected error occurred: {e}")

    ```

3.  **Replace Placeholders**:
    In the `redis_test.py` script, replace `YOUR_REDIS_HOSTNAME` and `YOUR_REDIS_PRIMARY_KEY` with the actual values you obtained in Part 3.

      * Alternatively, you can set these as environment variables before running the script (recommended for production):
        ```bash
        # For Linux/macOS
        export REDIS_HOST="myredis-youruniqueid.redis.cache.windows.net"
        export REDIS_PASSWORD="<your_primary_key>"
        # For Windows (Command Prompt)
        set REDIS_HOST="myredis-youruniqueid.redis.cache.windows.net"
        set REDIS_PASSWORD="<your_primary_key>"
        ```

4.  **Run the Python script**:
    Open your terminal or command prompt, navigate to the directory where you saved `redis_test.py`, and run:

    ```bash
    python redis_test.py
    ```

    You should see output similar to this, indicating successful connection and operations:

    ```
    Successfully connected to Redis at myredis-youruniqueid.redis.cache.windows.net:6380
    Set 'mykey': 'Hello from Azure Cache for Redis!'
    Retrieved 'mykey': 'Hello from Azure Cache for Redis!'
    Incremented 'mycounter', current value: 2
    Added items to 'mylist', current list: [b'item1', b'item2', b'item3']
    Cleaned up keys.
    ```

## Part 5: Cleanup Resources

To avoid incurring ongoing costs, it's crucial to delete the Azure Cache for Redis instance and its associated resource group when you are finished with this lab.

**(Choose one method: Azure Portal, Azure CLI, or Azure PowerShell)**

### 5.1. Azure Portal Method

1.  Navigate to your Azure Cache for Redis instance in the Azure Portal.
2.  Click **Delete** from the top toolbar.
3.  Confirm the deletion by typing the cache name and clicking **Delete**.

### 5.2. Azure CLI Method

```bash
az group delete \
  --name $RESOURCE_GROUP \
  --yes --no-wait
echo "Resource group '$RESOURCE_GROUP' and all its resources (including Redis Cache) deletion initiated."
```

### 5.3. Azure PowerShell Method

```powershell
Remove-AzResourceGroup -Name $resourceGroupName -Force -AsJob
Write-Host "Resource group '$resourceGroupName' and all its resources (including Redis Cache) deletion initiated."
```

-----

Congratulations\! You've successfully created, connected to, and performed basic operations on an Azure Cache for Redis instance. 

This gives you a solid foundation for understanding how to integrate caching into your applications.