# Lab: Creating an Azure Cosmos DB Account ☁️💾:

Let's start with **Lab: Creating an Azure Cosmos DB Account**. We'll focus on the SQL API, and I'll recommend the **Serverless** capacity mode for cost-effectiveness during the lab, as you only pay for operations consumed.

## Lab Scenario 🧪

Before you can store any data in Azure Cosmos DB, you need to create an Azure Cosmos DB account. 

This account serves as the unit of management for your Cosmos DB resources, including databases and containers. 

In this lab, you will create a new Cosmos DB account configured for the **NoSQL API** and use the **Serverless** capacity mode for cost-effectiveness during development and testing.

## Part 1: Prerequisites

  * An active Azure subscription.
  * Azure CLI or Azure PowerShell installed and logged in (if you plan to use those methods).
  * A unique name for your Cosmos DB account.

## Part 2: Create Azure Cosmos DB Account

You will create an Azure Cosmos DB account with the NoSQL API (formerly SQL API) in Serverless capacity mode. This is ideal for learning and development as you only pay for the Request Units (RUs) consumed, rather than provisioned throughput.

**Important Considerations for Account Naming**:

  * Your Cosmos DB account name must be **globally unique** across all of Azure.
  * It can contain only lowercase letters, numbers, and the hyphen (-) character.
  * It must be between 3 and 44 characters long.
  * It will become part of your account's URI (e.g., `youruniqueaccountname.documents.azure.com`).

**(Choose one method: Azure Portal, Azure CLI, or Azure PowerShell)**

### 2.1. Azure Portal Method

1.  Sign in to the [Azure Portal](https://portal.azure.com/).
2.  In the Azure portal search bar, type `Azure Cosmos DB` and select **Azure Cosmos DB** from the results.
3.  On the Azure Cosmos DB page, click **+ Create**.
4.  On the **Select API option** page, under **Azure Cosmos DB for NoSQL**, click **Create**.
5.  On the **Create Azure Cosmos DB Account - Basics** tab, fill in the following details:
      * **Subscription**: Select your Azure subscription.
      * **Resource Group**: Select an existing resource group (`rg-sqldb-lab` if you created it in previous labs, or create a new one, e.g., `rg-cosmosdb-lab`).
      * **Account Name**: Enter a **globally unique** name for your Cosmos DB account (e.g., `cosmosdb-youruniqueid`).
      * **Location**: Select a region close to you (e.g., `East US`, `West Europe`, `Southeast Asia`).
      * **Capacity mode**: Select **Serverless**.
          * *(Note: The "Apply Azure Cosmos DB free tier discount" option might not be available for Serverless accounts, as Serverless by nature bills per consumption, which is often very low for dev/test scenarios).*
      * **Limit total account throughput**: Leave unchecked.
6.  Click **Review + create**.
7.  Review the settings. Once validation passes, click **Create**.
8.  The deployment will start and can take a few minutes to complete. Wait for the portal to display "Your deployment is complete."
9.  Click **Go to resource** to navigate to your new Azure Cosmos DB account.

### 2.2. Azure CLI Method

1.  Open your terminal or Cloud Shell (shell.azure.com).
2.  Set environment variables. Remember to replace `youruniqueid` with a unique string.
    ```bash
    RESOURCE_GROUP="rg-cosmosdb-lab" # Or use your existing rg-sqldb-lab
    LOCATION="eastus" # Choose a region close to you
    COSMOS_ACCOUNT_NAME="cosmosdb-youruniqueid" # Must be globally unique and lowercase
    API_TYPE="NoSql" # For SQL API
    ```
3.  (Optional) Create the resource group if it doesn't exist:
    ```bash
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```
4.  Create the Azure Cosmos DB account in Serverless mode:
    ```bash
    az cosmosdb create \
      --name $COSMOS_ACCOUNT_NAME \
      --resource-group $RESOURCE_GROUP \
      --locations regionName=$LOCATION \
      --kind $API_TYPE \
      --server-version 4.0 \
      --capabilities Serverless \
      --default-consistency-level Session \
      --output table
    ```
      * `--locations`: Specifies the primary region. Serverless accounts currently operate in a single region.
      * `--kind NoSql`: Specifies the API type.
      * `--capabilities Serverless`: This is the crucial flag for enabling Serverless mode.
      * `--default-consistency-level Session`: A common and practical consistency level for many applications.
5.  The command will output the details of the created account. This process may take a few minutes.

### 2.3. Azure PowerShell Method

1.  Open PowerShell.
2.  Set environment variables. Remember to replace `youruniqueid` with a unique string.
    ```powershell
    $resourceGroupName = "rg-cosmosdb-lab" # Or use your existing rg-sqldb-lab
    $location = "eastus" # Choose a region close to you
    $cosmosAccountName = "cosmosdb-youruniqueid" # Must be globally unique and lowercase
    $apiKind = "Sql" # For SQL API
    ```
3.  (Optional) Create the resource group if it doesn't exist:
    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location $location
    ```
4.  Create the Azure Cosmos DB account in Serverless mode.
      * **Note on PowerShell for Serverless**: As of this writing (July 2025), direct creation of a Serverless account using `New-AzCosmosDBAccount` *with a specific parameter for Serverless* is not as straightforward as CLI or Portal, and sometimes requires using the `-Capabilities` parameter with "Serverless" explicitly or setting `ApiProperties.Serverless` in a more advanced way if directly interacting with the ARM API. However, the most common way to create a SQL API serverless account is often implicitly handled if you don't specify RU/s or if a specific `ApiProperties` or `EnableServerless` parameter becomes available. If the command below creates a provisioned throughput account, you might need to adjust (or use CLI/Portal for Serverless accounts more reliably). For simplicity in a lab setting, if the `--capabilities Serverless` for CLI is available, it's generally preferred for a direct serverless creation.
      * Let's try the common `New-AzCosmosDBAccount` with `DefaultConsistencyLevel` which usually implies serverless if no RUs are set for *new* API versions, or explicitly try the `Serverless` capability if available.
    <!-- end list -->
    ```powershell
    # As of current module versions, 'Capabilities' parameter with 'Serverless' is the way to specify this
    New-AzCosmosDBAccount -ResourceGroupName $resourceGroupName `
      -Name $cosmosAccountName `
      -Location $location `
      -ApiKind $apiKind `
      -DefaultConsistencyLevel Session `
      -Kind GlobalDocumentDB # For NoSQL API
      -EnableServerless # This parameter specifically indicates serverless capacity mode.
    ```
      * **Important Check**: After running, verify in the Azure Portal that the "Capacity mode" for your newly created account is indeed "Serverless". If not, use the Azure Portal or Azure CLI for this step. The PowerShell module updates frequently, so the exact parameter for Serverless can sometimes vary.
5.  The command will output the details of the created account. This process may take a few minutes.

-----

## Part 3: Verify Account Creation

Once the deployment is complete:

1.  **Azure Portal**: Navigate to your new Cosmos DB account. On the **Overview** blade, check:
      * **API**: Should be "Azure Cosmos DB for NoSQL".
      * **Location**: Your chosen region.
      * **Capacity Mode**: Should be "Serverless".
2.  **Azure CLI**:
    ```bash
    az cosmosdb show \
      --name $COSMOS_ACCOUNT_NAME \
      --resource-group $RESOURCE_GROUP \
      --query "{name:name, apiKind:kind, location:location, capabilities:capabilities}" \
      --output table
    ```
      * Look for `Serverless` in the `capabilities` output.
3.  **Azure PowerShell**:
    ```powershell
    Get-AzCosmosDBAccount -ResourceGroupName $resourceGroupName -Name $cosmosAccountName | Select-Object Name, ApiKind, Location, @{Name='Capabilities'; Expression={$_.Capabilities.Name}}
    ```
      * Look for `Serverless` in the `Capabilities` output.

-----

## Part 4: Cleanup (Optional, but good practice if you want to stop here)

You can delete the Cosmos DB account if you wish to stop incurring costs immediately. However, for the next lab, you will need this account.

**(Choose one method: Azure Portal, Azure CLI, or Azure PowerShell)**

### 4.1. Azure Portal Method

1.  Navigate to your Azure Cosmos DB account in the Azure Portal.
2.  Click **Delete** from the top toolbar.
3.  Confirm the deletion by typing the account name and clicking **Delete**.

### 4.2. Azure CLI Method

```bash
RESOURCE_GROUP="rg-cosmosdb-lab"
COSMOS_ACCOUNT_NAME="cosmosdb-youruniqueid"
az cosmosdb delete \
  --name $COSMOS_ACCOUNT_NAME \
  --resource-group $RESOURCE_GROUP \
  --yes # Acknowledge prompt
echo "Cosmos DB account '$COSMOS_ACCOUNT_NAME' deletion initiated."
```

### 4.3. Azure PowerShell Method

```powershell
$resourceGroupName = "rg-cosmosdb-lab"
$cosmosAccountName = "cosmosdb-youruniqueid"
Remove-AzCosmosDBAccount -ResourceGroupName $resourceGroupName `
  -Name $cosmosAccountName -Force
Write-Host "Cosmos DB account '$cosmosAccountName' deletion initiated."
```

-----

You have successfully created an Azure Cosmos DB account configured for the NoSQL API in Serverless mode\!