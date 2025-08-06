# Lab: Working with Azure Blob Storage 📦

## Lab Scenario 🧪

You are a data engineer responsible for managing a large collection of unstructured data, such as images and log files. You need to use Azure Blob Storage to securely store this data and perform basic upload and download operations. This lab will guide you through the process of creating containers, uploading blobs, and retrieving them.

## Part 1: Prerequisites

* An **active Azure subscription**.

* **Azure Cloud Shell**: This lab uses the Azure CLI and Azure PowerShell.

* **An Azure Storage Account**: You can use the storage account created in the previous lab. If you do not have one, run the following commands to create a resource group and a new storage account.
    ```bash
    # Run these commands if you don't have a storage account
    RESOURCE_GROUP="rg-blob-lab"
    LOCATION="eastus"
    STORAGE_ACCOUNT_NAME="labblobstorage123" # Replace with your unique name
    az group create --name $RESOURCE_GROUP --location $LOCATION
    az storage account create --name $STORAGE_ACCOUNT_NAME --resource-group $RESOURCE_GROUP --location $LOCATION --sku Standard_LRS --kind StorageV2
    ```

-----

## Part 2: Create a Blob Container

A **container** in Blob Storage acts as a directory to group a set of blobs. We'll create one to hold our sample files.

### Method 1: Azure Portal 🌐

1.  In the Azure Portal, navigate to your storage account.

2.  In the left-hand menu, under **Data storage**, select **Containers**.

3.  Click **+ Container**.

4.  **Name**: Enter `my-first-blob-container`.

5.  **Public access level**: Select **Private**. This is the most secure setting, as it restricts access to authorized users.

6.  Click **Create**.

### Method 2: Azure CLI 🖥️

1.  Open your **Azure Cloud Shell** and ensure you are in the **Bash** environment.
2.  **Set variables**:
    ```bash
    RESOURCE_GROUP="rg-blob-lab"
    STORAGE_ACCOUNT_NAME="labblobstorage123" # Your unique name
    CONTAINER_NAME="my-first-blob-container"
    ```
3.  **Create the container**:
    ```bash
    az storage container create --account-name $STORAGE_ACCOUNT_NAME --name $CONTAINER_NAME
    ```

### Method 3: Azure PowerShell 💻

1.  Open your **Azure Cloud Shell** and ensure you are in the **PowerShell** environment.
2.  **Set variables**:
    ```powershell
    $resourceGroup = "rg-blob-lab"
    $storageAccountName = "labblobstorage123" # Your unique name
    $containerName = "my-first-blob-container"
    $ctx = (Get-AzStorageAccount -ResourceGroupName $resourceGroup -Name $storageAccountName).Context
    ```
3.  **Create the container**:
    ```powershell
    New-AzStorageContainer -Context $ctx -Name $containerName -Permission Off
    ```

-----

## Part 3: Uploading a Blob (File)

Now let's upload a file into our new container. First, we need to create a sample file.

### 1\. Create a Sample File

Run the following command in your Cloud Shell (Bash or PowerShell) to create a simple text file:

```bash
echo "This is my first blob file." > samplefile.txt
```

### 2\. Upload the Blob

#### Method 1: Azure Portal 🌐

1.  In your storage account, navigate to **Containers** and click on `my-first-blob-container`.
2.  Click **Upload**.
3.  Select the `samplefile.txt` file from your local machine.
4.  Click **Upload**. The file will appear in the container list.

#### Method 2: Azure CLI 🖥️

```bash
az storage blob upload \
  --account-name $STORAGE_ACCOUNT_NAME \
  --container-name $CONTAINER_NAME \
  --name "samplefile.txt" \
  --file "samplefile.txt"
```

#### Method 3: Azure PowerShell 💻

```powershell
Set-AzStorageBlobContent -Context $ctx `
  -Container $containerName `
  -File "samplefile.txt" `
  -Blob "samplefile.txt"
```

-----

## Part 4: Downloading a Blob

You can download a blob to your local machine or view it directly.

### Method 1: Azure Portal 🌐

1.  In your storage account, go to **Containers** and click on `my-first-blob-container`.
2.  Click on `samplefile.txt`.
3.  Click **Download**. The file will be downloaded to your local machine.

### Method 2: Azure CLI 🖥️

```bash
az storage blob download \
  --account-name $STORAGE_ACCOUNT_NAME \
  --container-name $CONTAINER_NAME \
  --name "samplefile.txt" \
  --file "downloaded_samplefile.txt"
```

*This command saves the blob as a new file named `downloaded_samplefile.txt` in your current directory.*

### Method 3: Azure PowerShell 💻

```powershell
Get-AzStorageBlobContent -Context $ctx `
  -Container $containerName `
  -Blob "samplefile.txt" `
  -Destination "downloaded_samplefile.txt"
```

*This command saves the blob to a new file named `downloaded_samplefile.txt` in your current directory.*

-----

## Part 5: Cleanup Resources (Critical\!)

To avoid costs, delete the resource group, which will remove the storage account and everything inside it.

### Method 1: Azure Portal 🌐

1.  Navigate to **Resource groups**, click on `rg-blob-lab`.
2.  Click **Delete resource group**, type the name to confirm, and click **Delete**.

### Method 2: Azure CLI 🖥️

```bash
az group delete --name rg-blob-lab --yes --no-wait
```

### Method 3: Azure PowerShell 💻

```powershell
Remove-AzResourceGroup -Name "rg-blob-lab" -Force
```