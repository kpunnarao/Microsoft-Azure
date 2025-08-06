# Lab: Creating an Azure Storage Account 💾

## Lab Scenario 🧪

You are a cloud administrator tasked with provisioning a new storage solution for your organization's data. You need a general-purpose, cost-effective storage account that can serve as the foundation for storing various types of data, including files and backups. This lab will walk you through the creation of this core resource.

## Part 1: Prerequisites

* An **active Azure subscription**.

* **Azure Cloud Shell**: This lab uses the Azure CLI and Azure PowerShell, which are pre-installed in the Cloud Shell. Access it from the Azure portal by clicking the `>_` icon.

-----

## Part 2: Create the Storage Account

The **storage account** is a container that groups a set of Azure Storage services. We will create a **General-purpose v2** account, which is the recommended type for most scenarios.

### Method 1: Azure Portal 🌐

1.  Sign in to the **Azure Portal**.

2.  In the global search bar, type `Storage accounts` and select it from the results.

3.  Click **+ Create**.

4.  On the **Basics** tab, provide the following details:
      * **Subscription**: Select your Azure subscription.
      * **Resource group**: Click **Create new** and enter `rg-storage-lab`.
      * **Storage account name**: Enter a **globally unique name** (e.g., `labstorageaccount123`). This name must be 3-24 characters long, with only lowercase letters and numbers.
      * **Region**: Select a region close to you, such as `(US) East US`.
      * **Performance**: Select **Standard** for general-purpose use.
      * **Redundancy**: Select **Locally-redundant storage (LRS)**. This is the lowest-cost option and provides data durability.
5.  Click **Review** and then **Create**.

##### Method 2: Azure CLI 🖥️

1.  Open your **Azure Cloud Shell** and ensure you are in the **Bash** environment.

2.  **Set variables**:
    ```bash
    RESOURCE_GROUP="rg-storage-lab"
    LOCATION="eastus"
    STORAGE_ACCOUNT_NAME="labstorageaccount123" # Replace with your unique name
    ```

3.  **Create the resource group**:
    ```bash
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

4.  **Create the storage account**:
    ```bash
    az storage account create \
      --name $STORAGE_ACCOUNT_NAME \
      --resource-group $RESOURCE_GROUP \
      --location $LOCATION \
      --sku Standard_LRS \
      --kind StorageV2
    ```

##### Method 3: Azure PowerShell 💻

1.  Open your **Azure Cloud Shell** and ensure you are in the **PowerShell** environment.
2.  **Set variables**:
    ```powershell
    $resourceGroup = "rg-storage-lab"
    $location = "eastus"
    $storageAccountName = "labstorageaccount123" # Replace with your unique name
    ```
3.  **Create the resource group**:
    ```powershell
    New-AzResourceGroup -Name $resourceGroup -Location $location
    ```
4.  **Create the storage account**:
    ```powershell
    New-AzStorageAccount -ResourceGroupName $resourceGroup `
      -Name $storageAccountName `
      -Location $location `
      -SkuName "Standard_LRS" `
      -Kind "StorageV2"
    ```

-----

#### Part 3: Verify the Deployment

  * **Azure Portal**: Navigate to **Resource groups**, click on `rg-storage-lab`, and you will see your new storage account listed.
  * **Azure CLI**: Use the command `az storage account show --name $STORAGE_ACCOUNT_NAME --resource-group $RESOURCE_GROUP`.
  * **Azure PowerShell**: Use the command `Get-AzStorageAccount -Name $storageAccountName -ResourceGroupName $resourceGroup`.

-----

#### Part 4: Cleanup Resources (Critical\!)

To avoid incurring ongoing costs, it is essential to delete all resources created in this lab. Deleting the resource group will remove the storage account and all of its contents.

##### Method 1: Azure Portal 🌐

1.  Navigate to **Resource groups**, click on `rg-storage-lab`.
2.  Click **Delete resource group**, type the name to confirm, and click **Delete**.

##### Method 2: Azure CLI 🖥️

```bash
az group delete --name rg-storage-lab --yes --no-wait
```

##### Method 3: Azure PowerShell 💻

```powershell
Remove-AzResourceGroup -Name "rg-storage-lab" -Force
```