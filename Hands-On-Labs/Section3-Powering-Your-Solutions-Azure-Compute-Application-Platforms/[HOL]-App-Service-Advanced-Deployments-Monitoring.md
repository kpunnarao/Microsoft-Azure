# Lab: App Service - Advanced Deployments & Monitoring 🚀

## Lab Scenario 🧪

You are a DevOps engineer responsible for a critical web application. A new feature has been developed and needs to be deployed with **zero downtime**. Before pushing it live, you need to test it in a dedicated environment. You will use Azure App Service deployment slots to achieve this, then monitor the application's performance and troubleshoot a simulated issue.

**Your specific objectives for this lab are:**

* Create a **staging** deployment slot.

* Deploy a new version of your application to the staging slot.

* Perform a **zero-downtime swap** to move the new version to production.

* Monitor key application performance metrics.

* Review live application logs.

* Troubleshoot a simulated common app issue.

* Clean up all created resources to avoid costs.

## Part 1: Prerequisites & Architecture

* An active Azure subscription.

* **Azure Cloud Shell:** This lab uses Azure CLI and Azure PowerShell. Access it from the Azure portal by clicking the `>_` icon.

* **A running App Service:** You will need a functioning Azure App Service to begin. You can either use the one from the previous lab or follow these quick steps to create a new one:
    ```bash
    # Set your unique names and location
    RESOURCE_GROUP="rg-appservice-adv"
    LOCATION="eastus"
    APP_SERVICE_PLAN_NAME="appplan-adv-lab"
    WEB_APP_NAME="webapp-adv-lab-youruniqueid"

    # Create the resource group
    az group create --name $RESOURCE_GROUP --location $LOCATION

    # Create the App Service plan (F1 Free tier)
    az appservice plan create --name $APP_SERVICE_PLAN_NAME --resource-group $RESOURCE_GROUP --location $LOCATION --sku F1 --is-linux

    # Create the Web App and deploy sample code
    az webapp create --name $WEB_APP_NAME --resource-group $RESOURCE_GROUP --plan $APP_SERVICE_PLAN_NAME --runtime "NODE:18-lts"
    az webapp deployment source config --name $WEB_APP_NAME --resource-group $RESOURCE_GROUP --repo-url "https://github.com/Azure-Samples/nodejs-docs-hello-world" --branch main --git-token ""
    ```
    You now have an App Service with a sample app deployed to your production slot.

-----

## Part 2: Creating a Staging Deployment Slot

Deployment slots are live apps with their own hostnames. You can deploy to a slot and swap it into production, allowing you to test a new version before releasing it to your users.

### Method 1: Azure Portal 🌐

1.  Navigate to your Web App in the Azure Portal.

2.  In the left-hand menu, under **Deployment**, select **Deployment slots**.

3.  Click **+ Add Slot**.

4.  **Name**: `staging`.

5.  **Clone settings from**: `production` (this copies app settings, connection strings, etc.).

6.  Click **Add**. The new slot will take a few moments to create. You can access it via its unique URL, e.g., `webapp-adv-lab-youruniqueid-staging.azurewebsites.net`.

### Method 2: Azure CLI 🖥️

```bash
az webapp deployment slot create \
  --name $WEB_APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --slot staging \
  --configuration-source production
```

### Method 3: Azure PowerShell 💻

```powershell
New-AzWebAppSlot -Name $WEB_APP_NAME `
    -ResourceGroupName $RESOURCE_GROUP `
    -Slot "staging" `
    -AppServicePlan $APP_SERVICE_PLAN_NAME `
    -SourceWebApp $WEB_APP_NAME
```

-----

## Part 3: Deploying a New Version to the Staging Slot

Now, let's pretend a new version of the application is ready. We will deploy it directly to the staging slot.

### Method 1: Azure Portal 🌐

1.  Navigate to your **staging** slot (it will be listed under the **Deployment slots** blade of your main web app).

2.  On the `staging` slot's overview page, go to **Deployment Center** in the left menu.

3.  Deploy a new version of code from a different branch or repository. For this lab, simply note that this is where you would configure the deployment for the staging slot. The production app remains untouched and unaffected.

### Method 2: Azure CLI 🖥️

```bash
# This command deploys to the "staging" slot
az webapp deployment source config \
  --name $WEB_APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --slot staging \
  --repo-url "https://github.com/Azure-Samples/nodejs-docs-hello-world" \
  --branch main --git-token "" # Replace with your deployment source
```

-----

## Part 4: Swapping Deployment Slots (Zero-Downtime Deployment)

With the new version deployed and tested in `staging`, it's time to swap it with `production`. This process is fast and seamless.

### Method 1: Azure Portal 🌐

1.  Navigate to your main Web App's **Deployment slots** blade.

2.  Select the `staging` slot and click the **Swap** button at the top.

3.  Review the source and target slots.
      * **Source**: `staging`.
      * **Target**: `production`.

4.  Click **Swap**. This operation redirects traffic from the production slot to the warmed-up staging slot.

### Method 2: Azure CLI 🖥️

```bash
az webapp deployment slot swap \
  --name $WEB_APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --slot staging \
  --target-slot production
```

### Method 3: Azure PowerShell 💻

```powershell
Move-AzWebAppSlot -Name $WEB_APP_NAME `
    -ResourceGroupName $RESOURCE_GROUP `
    -SourceSlotName "staging" `
    -DestinationSlotName "production"
```

-----

## Part 5: Monitoring App Service Performance

Now that your app is live, let's explore its monitoring capabilities.

### 5.1. View Metrics

1.  In the Azure Portal, navigate to your Web App's **Overview** blade.

2.  The overview page provides a high-level view of metrics.

3.  In the left menu, under **Monitoring**, select **Metrics**.

4.  Use this blade to view a chart of various metrics, such as:
      * **CPU Percentage**
      * **Memory Percentage**
      * **HTTP Server Errors** (`Http 5xx`)
      * **Data In/Out**

### 5.2. View Logs

1.  In the Web App's left menu, under **Monitoring**, select **Log stream**.

2.  This blade provides a live log stream of your application's output, helping you see errors and debug in real-time.

-----

## Part 6: Troubleshooting a Common Issue

Let's simulate a simple issue and use the built-in troubleshooting tools.

1.  **Simulate an issue**: In the left menu, under **Development Tools**, select **Advanced Tools** and click **Go -\>**. This opens the Kudu console.

2.  Navigate to the **Debug Console** and select **CMD**.
3.  Go to the `site\wwwroot` directory and delete the `server.js` file (or equivalent startup file for your language). This will cause the app to fail.

4.  Now, navigate back to your Web App's **Overview** page and click the URL. You will see an error.

5.  In the left menu, select **Diagnose and solve problems**. This blade is an intelligent troubleshooting tool that can analyze your application's state and suggest solutions.

6.  The tool will likely suggest that your app is failing due to a missing startup file. This demonstrates how to use the built-in tooling to quickly identify and resolve issues.

-----

## Part 7: Cleanup Resources (Critical\!)

To avoid incurring ongoing costs, it is essential to delete all resources created in this lab. Deleting the resource group will remove the App Service Plan, Web App, and all associated deployment slots.

### Method 1: Azure Portal 🌐

1.  In the Azure Portal, search for `Resource groups` and select it.

2.  Find and click on your resource group: `rg-appservice-adv`.

3.  On the resource group's Overview blade, click **Delete resource group**.

4.  Type the resource group name (`rg-appservice-adv`) to confirm, then click **Delete**.

### Method 2: Azure CLI 🖥️

```bash
az group delete --name rg-appservice-adv --yes --no-wait
```

### Method 3: Azure PowerShell 💻

```powershell
Remove-AzResourceGroup -Name "rg-appservice-adv" -Force
```

Congratulations\! You have successfully implemented a zero-downtime deployment strategy, monitored your application, and practiced troubleshooting common issues.