# Lab: Azure Functions - Serverless Compute ⚡

## Lab Scenario 🧪

You are a cloud developer exploring serverless computing for a new project. You need to create a simple HTTP endpoint that can be triggered by a web request. Additionally, you want to see how to use Azure Functions to automate a task, such as sending a message to a storage queue without writing complex code.

**Your specific objectives for this lab are:**

* Understand the concept of **serverless computing**.

* Create a **Function App**, which is the hosting environment for your functions.

* Develop a basic **HTTP-triggered function**.

* Test the function to ensure it works correctly.

* Integrate the function with an Azure Storage Queue using an **output binding**.

* Clean up all created resources to avoid costs.

## Part 1: Prerequisites & Architecture

* An active Azure subscription.

* **Azure Cloud Shell**: This lab uses Azure CLI and Azure PowerShell. Access it from the Azure portal by clicking the `>_` icon.

* **Function App Runtime**: We will use **Node.js** as the runtime stack for this lab.

-----

## Part 2: Create the Function App

A Function App hosts the execution of your functions. We will use the **Consumption plan** which is the true serverless model—you only pay when your function is running.

### Method 1: Azure Portal 🌐

1.  Navigate to the Azure Portal and search for `Function App`. Click **+ Create**.

2.  **Basics Tab**:
      * **Subscription**: Select your subscription.
      * **Resource Group**: Click **Create new** and enter `rg-functions-lab`.
      * **Function App name**: Enter a unique name, e.g., `functions-lab-yourname`.
      * **Publish**: `Code`.
      * **Runtime stack**: `Node.js`.
      * **Version**: `18 LTS`.
      * **Region**: `(US) East US`.
      * **Operating System**: `Windows`.
      * **Plan type**: `Consumption (Serverless)`.

3.  Click **Review + create**, then **Create**.

### Method 2: Azure CLI 🖥️

1.  **Set Variables**:
    ```bash
    RESOURCE_GROUP="rg-functions-lab"
    LOCATION="eastus"
    FUNCTION_APP_NAME="functions-lab-youruniqueid"
    STORAGE_ACCOUNT_NAME="storageyouruniqueid" # Must be globally unique
    ```

2.  **Create a Storage Account** (required for the Function App).
    ```bash
    az storage account create --name $STORAGE_ACCOUNT_NAME --location $LOCATION --resource-group $RESOURCE_GROUP --sku Standard_LRS
    ```

3.  **Create the Function App**:
    ```bash
    az functionapp create --resource-group $RESOURCE_GROUP --name $FUNCTION_APP_NAME --storage-account $STORAGE_ACCOUNT_NAME --consumption-plan-location $LOCATION --runtime "node" --runtime-version "18" --functions-version "4"
    ```

### Method 3: Azure PowerShell 💻

1.  **Set Variables**:
    ```powershell
    $resourceGroup = "rg-functions-lab"
    $location = "EastUS"
    $functionAppName = "functions-lab-youruniqueid"
    $storageAccountName = "storageyouruniqueid"
    ```

2.  **Create the Resource Group**:
    ```powershell
    New-AzResourceGroup -Name $resourceGroup -Location $location
    ```

3.  **Create a Storage Account**:
    ```powershell
    New-AzStorageAccount -ResourceGroupName $resourceGroup -Name $storageAccountName -Location $location -SkuName "Standard_LRS" -Kind "StorageV2"
    ```

4.  **Create the Function App**:
    ```powershell
    New-AzFunctionApp -ResourceGroupName $resourceGroup -Name $functionAppName `
        -Location $location -StorageAccountName $storageAccountName `
        -Runtime "node" -RuntimeVersion "18" `
        -FunctionsVersion "4" -OSType "Windows" `
        -ConsumptionPlan
    ```

-----

## Part 3: Creating a Simple HTTP-triggered Function

### Method 1: Azure Portal 🌐

1.  Navigate to your new Function App in the Azure Portal.

2.  In the left-hand menu, select **Functions**, then click **+ Create**.

3.  **Development environment**: Choose **In-portal**.

4.  **Template**: Select **HTTP trigger**.

5.  **New Function**:
      * **New Function Name**: `HttpExample`.
      * **Authorization level**: `Function`.

6.  Click **Create**.

### Method 2: Azure CLI 🖥️

```bash
# Not possible to create a function from a template directly in CLI.
# You must deploy code. However, you can scaffold a project.
# You can use VS Code with the Azure Functions extension for this.
```

*For this lab, we will use the Portal to create the function, then use the CLI to manage it.*

### Method 3: Azure PowerShell 💻

*Like the CLI, PowerShell does not have a cmdlet to create a function from a template. You would need to publish a project from a local environment. We will use the Portal to create the function and then manage it with PowerShell.*

-----

## Part 4: Testing the Function

1.  Navigate to your `HttpExample` function in the Portal.

2.  Click **Code + Test**.

3.  Click the **Get Function URL** button and copy the URL.

4.  Open a new browser tab and paste the URL. You should see a message: `"Please pass a name on the query string or in the request body"`.

5.  Now, add a query parameter to the URL: `&name=Azure`. The full URL will be similar to `https://functions-lab-yourname.azurewebsites.net/api/HttpExample?code=<your_code>&name=Azure`.

6.  You should now see the personalized message: `"Hello, Azure. This HTTP triggered function executed successfully."`.

-----

## Part 5: Integrating with Other Services (Bindings)

We'll add an **output binding** to send a message to a storage queue every time the function is triggered.

1.  **Create a Storage Queue**: In the Azure Portal, navigate to the storage account you created for your Function App. In the left menu, select **Queues** under **Data storage**, and click **+ Queue**. Name it `outqueue`.

2.  **Add the Output Binding**:

      * In your Function App, go to the `HttpExample` function.
      * Select **Integration**.
      * Click **+ Add output**.
      * **Binding Type**: `Azure Queue Storage`.
      * **Storage account connection**: Select your storage account.
      * **Queue name**: `outqueue`.
      * **Message parameter name**: `outputQueueItem`.
      * Click **OK**.

3.  **Modify the Function Code**:

      * Go to **Code + Test** for your function.
      * The `function.json` file now includes the new binding.
      * Modify the `index.js` file to send a message. Replace the existing code with the following:

    <!-- end list -->

    ```javascript
    module.exports = async function (context, req) {
        context.log('JavaScript HTTP trigger function processed a request.');
        const name = (req.query.name || (req.body && req.body.name));
        const responseMessage = name
            ? "Hello, " + name + ". This HTTP triggered function executed successfully."
            : "Please pass a name on the query string or in the request body for a personalized response.";
        context.res = {
            body: responseMessage
        };
        context.bindings.outputQueueItem = "Message sent to queue at " + new Date().toUTCString();
    };
    ```

    *This code now sets a message to the `outputQueueItem` binding, which will send it to the `outqueue`.*

4.  **Test the Integration**:

      * Run the function again with the `&name=Azure` query parameter.
      * In the Azure Portal, navigate back to your storage account and select the `outqueue` queue. You should now see a new message in the queue.

-----

## Part 6: Cleanup Resources (Critical\!)

### Method 1: Azure Portal 🌐

1.  Navigate to `Resource groups`, click on `rg-functions-lab`.
2.  Click **Delete resource group**, type the name to confirm, and click **Delete**.

### Method 2: Azure CLI 🖥️

```bash
az group delete --name rg-functions-lab --yes --no-wait
```

### Method 3: Azure PowerShell 💻

```powershell
Remove-AzResourceGroup -Name "rg-functions-lab" -Force
```

Congratulations\! You have successfully created and tested a serverless function, and integrated it with an Azure Storage Queue using the power of bindings.