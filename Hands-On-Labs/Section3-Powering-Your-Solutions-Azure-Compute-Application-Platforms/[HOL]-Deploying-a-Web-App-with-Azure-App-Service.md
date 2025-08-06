# Lab: Deploying a Web App with Azure App Service 🌐

## Lab Scenario 🧪

You are a developer tasked with deploying a new web application. You want to use a fully managed platform to avoid dealing with server maintenance, operating system patching, and scaling infrastructure. 

Azure App Service is the ideal solution. This lab will guide you through creating an App Service Plan and a Web App, and then deploying a simple application to it.

**Your specific objectives for this lab are:**

* Understand the relationship between an App Service Plan and a Web App.

* Create a free-tier App Service Plan.

* Deploy a Web App linked to the plan.

* Deploy a simple application from a public code repository.

* Verify the live application is running.

* Clean up all created resources to avoid costs.

## Part 1: Prerequisites & Architecture

* An active Azure subscription.

* **Azure Cloud Shell:** This lab uses Azure CLI and Azure PowerShell, which are pre-installed in the Azure Cloud Shell. Access it from the Azure portal by clicking the `>_` icon.

* **Basic Git knowledge:** Familiarity with Git is helpful for the deployment section.

## Part 2: Common Setup (Create the Resource Group)

All three deployment methods require a resource group. We will create it using Azure CLI.

1.  **Open Azure Cloud Shell** and select **Bash** environment.

2.  **Set Variables**:

    ```bash
    RESOURCE_GROUP="rg-appservice-lab"
    LOCATION="eastus" # Replace with a region close to you
    APP_SERVICE_PLAN_NAME="appplan-lab"
    WEB_APP_NAME="webapp-lab-youruniqueid" # Must be globally unique
    ```

3.  **Create the Resource Group**:

    ```bash
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

-----

## Part 3: Create the App Service Plan

The App Service Plan defines the underlying compute resources for your web app. We will use the `F1 (Free)` tier.

### Method 1: Azure Portal (GUI) 🌐

1.  Navigate to the Azure Portal and search for `App Service plans`.
2.  Click **+ Create**.
3.  **Basics Tab**:
      * **Subscription**: Select your subscription.
      * **Resource Group**: Select `rg-appservice-lab`.
      * **Name**: Enter `appplan-lab-portal`.
      * **Operating System**: Select `Linux`.
      * **Region**: `(US) East US`.
      * **Pricing tier**: Click **Change size**. Under the **Dev/Test** tab, select **F1 (Free)**. Click **Apply**.
4.  Click **Review + create**, then **Create**.

### Method 2: Azure CLI 🖥️

```bash
az appservice plan create \
  --name $APP_SERVICE_PLAN_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku F1 \
  --is-linux
```

*The `--sku F1` flag specifies the free tier, and `--is-linux` sets the operating system.*

### Method 3: Azure PowerShell 💻

```powershell
# Create the App Service plan
New-AzAppServicePlan -Name $APP_SERVICE_PLAN_NAME `
    -ResourceGroupName $RESOURCE_GROUP `
    -Location $LOCATION `
    -Tier "Free" `
    -IsLinux `
    -Verbose
```

-----

## Part 4: Create the Web App and Deploy Code

Now we'll create the Web App and deploy a simple Node.js application from a GitHub repository.

### Method 1: Azure Portal (GUI) 🌐

1.  Navigate to the Azure Portal and search for `App Services`.
2.  Click **+ Create** -\> **Web App**.
3.  **Basics Tab**:
      * **Subscription**: Select your subscription.
      * **Resource Group**: Select `rg-appservice-lab`.
      * **Name**: Enter a unique name like `webapp-lab-yourname`.
      * **Publish**: `Code`.
      * **Runtime stack**: `Node 18 LTS`.
      * **Operating System**: `Linux`.
      * **Region**: `(US) East US`.
      * **App Service Plan**: Select `appplan-lab-portal`.
4.  Click **Review + create**, then **Create**.
5.  Once the app is deployed, navigate to its overview blade and click on the **Deployment Center** menu item.
6.  Under **Settings**, select **Source: GitHub**.
7.  Follow the prompts to connect to your GitHub account and choose a repository. For a simple app, you can use the sample code from Microsoft's repository: `https://github.com/Azure-Samples/nodejs-docs-hello-world`.

### Method 2: Azure CLI 🖥️

```bash
# Create the Web App and link it to the App Service Plan
az webapp create \
  --name $WEB_APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --plan $APP_SERVICE_PLAN_NAME \
  --runtime "NODE:18-lts"

# Deploy code from a public GitHub repository
az webapp deployment source config \
  --name $WEB_APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --repo-url "https://github.com/Azure-Samples/nodejs-docs-hello-world" \
  --branch main \
  --git-token "github_token" # Omit if using a public repo
```

*The `--git-token` is not required for public repos but is good practice to be aware of.*

### Method 3: Azure PowerShell 💻

*Note: Deploying from a public Git repository is not a single cmdlet in PowerShell. For simplicity in a lab, it's often done via the Portal or CLI. Here's how to create the web app, which is the core part.*

```powershell
# Create the Web App
New-AzWebApp -Name $WEB_APP_NAME `
    -ResourceGroupName $RESOURCE_GROUP `
    -Location $LOCATION `
    -AppServicePlan $APP_SERVICE_PLAN_NAME `
    -RuntimeStack "NODE|18-lts"
```

-----

## Part 5: Verify the Live Application

1.  **Get the URL of your Web App**:

      * **Azure Portal**: Go to your Web App resource, and the URL will be at the top of the **Overview** blade.
      * **Azure CLI**: `az webapp show --name $WEB_APP_NAME --resource-group $RESOURCE_GROUP --query "defaultHostName" -o tsv`
      * **Azure PowerShell**: `(Get-AzWebApp -ResourceGroupName $RESOURCE_GROUP -Name $WEB_APP_NAME).DefaultHostName`

2.  Open a web browser and navigate to the URL. You should see the sample application running.

-----

## Part 6: Cleanup Resources (Critical\!)

To avoid incurring ongoing costs, it is essential to delete all resources created in this lab. Deleting the resource group will remove both the App Service Plan and the Web App.

### Method 1: Azure Portal 🌐

1.  In the Azure Portal, search for `Resource groups` and select it.
2.  Find and click on your resource group: `rg-appservice-lab`.
3.  On the resource group's Overview blade, click **Delete resource group**.
4.  Type the resource group name (`rg-appservice-lab`) to confirm, then click **Delete**.

### Method 2: Azure CLI 🖥️

```bash
az group delete --name rg-appservice-lab --yes --no-wait
```

### Method 3: Azure PowerShell 💻

```powershell
Remove-AzResourceGroup -Name "rg-appservice-lab" -Force
```

Congratulations\! You have successfully created and deployed an application to Azure App Service, a fully managed PaaS platform, using all three primary Azure management tools.