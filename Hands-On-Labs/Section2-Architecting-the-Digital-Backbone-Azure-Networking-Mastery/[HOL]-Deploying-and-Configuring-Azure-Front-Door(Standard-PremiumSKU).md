# Lab: Deploying and Configuring Azure Front Door (Standard/Premium SKU): 🌐

Azure Front Door is a powerful global service that naturally follows our discussion of regional load balancing with Azure Load Balancer and Application Gateway. 

It's a key component for building truly global and highly performant applications.

This lab will guide you through the process of deploying and configuring Azure Front Door to provide a globally distributed, highly available, and accelerated entry point for your web application. 

You will learn how Front Door acts as a global load balancer and content delivery network (CDN), routing traffic from the edge of the network to your backend services.

## Lab Scenario 🧪

Your company is growing, and you need to provide a unified, performant, and secure endpoint for your web application to users worldwide. 

You have already deployed an Application Gateway (`appgw-web-prod`) to handle regional traffic and distribute it to your web servers. 

Now, you will place Azure Front Door in front of the Application Gateway. Front Door will act as a global entry point, routing all incoming user requests to your regional Application Gateway.

**Your specific objectives for this lab are:**

* Create an Azure Front Door (Standard or Premium SKU) profile.

* Configure a Front Door endpoint that will be your global public URL.

* Define an **Origin Group** that points to the public IP of your existing Application Gateway.

* Configure a **Route** to link the endpoint to the origin group.

* Validate that traffic is successfully flowing from the Front Door endpoint, through the Application Gateway, and to your backend web servers.

## Part 1: Prerequisites

* An active Azure subscription.

* **Existing Application Gateway**: You will use `appgw-web-prod` from the previous lab. It must be running and have a Public IP address.

* **Existing Backend Web Servers**: The `vm-web-01` and `vm-web-02` VMs behind the Application Gateway must be running and responding to HTTP traffic on port 80.

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

-----

## Part 2: Architecture Diagram (Conceptual)

The following diagram illustrates the layered architecture you will build, with Front Door at the global edge and Application Gateway handling regional traffic:

```
+------------------------------------------------------------------------------------------------+
|                                        Azure Cloud (Global)                                    |
|                                                                                                |
|                                        +-------------------+                                   |
|   +----------------------------------> | Azure Front Door  | <----------------------------------+
|   | Global User Traffic                | (Standard/Premium)|                                  |
|   |                                    | (Endpoint:        |                                  |
|   |                                    |   <name>.azurefd.net) |                               |
|   |                                    +---------|---------+                                   |
|   |                                              | (Routing Rule)                              |
|   |                                              v                                             |
|   |                                    +-------------------+                                   |
|   |                                    | Origin Group      |                                   |
|   |                                    |                   |                                   |
|   |                                    |  +-------------+  |                                   |
|   |                                    |  |   Origin    |  |                                   |
|   |                                    |  |  (appgw-web-prod's |                               |
|   |                                    |  |  Public IP)   |  |                                 |
|   |                                    |  +-------------+  |                                   |
|   |                                    +---------|---------+                                   |
|   |                                              |                                             |
|   |                                              v                                             |
|   |                     +----------------------------------------------------------------+     |
|   |                     |                Azure Region (e.g., East US)                  |     |
|   +-------------------> |   +------------------------------------------+               |     |
|                         |   |    Azure Application Gateway           |               |     |
|                         |   |    (appgw-web-prod)                      |               |     |
|                         |   +-------------------|----------------------+               |     |
|                         |                       | (Backend Pool)                       |     |
|                         |                       v                                      |     |
|                         |    +------------------+------------------+                    |     |
|                         |    |                  |                  |                    |     |
|                         |    v                  v                  v                    |     |
|                         |  +--------------+  +--------------+  +--------------+         |     |
|                         |  |  vm-web-01   |  |  vm-web-02   |  |  vm-web-03   |  (VMs in a Backend Subnet) |
|                         |  +--------------+  +--------------+  +--------------+         |     |
|                         |                                                                |     |
+------------------------------------------------------------------------------------------------+
```

-----

## Part 3: Step-by-Step Configuration: Deploying and Configuring Azure Front Door

### 3.1 Create a Front Door Profile

You will create a new Front Door profile and select the Standard SKU, which includes both load balancing and CDN features.

#### 3.1.1 Using Azure Portal

1.  In the Azure Portal search bar, type `Front Doors` and select it.

2.  Click **+ Create** and then click **Create a profile** under **Azure Front Door (Standard/Premium)**.

3.  **Basics Tab**:

      * **Subscription**: Your subscription.
      * **Resource group**: `rg-network-lab`.
      * **Region**: Leave this as the default or select a region. Azure Front Door is a global service, so the region here is only for metadata storage.
      * **Tier**: Select **Standard**.
      * Click **Review + create**, then click **Create**. This will create the basic profile.

4.  Once the profile is created, navigate to it and click on the `Front Door manager` under the `Settings` section.

#### 3.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
LOCATION="<YourAzureRegion>"
FD_PROFILE_NAME="fd-web-prod"
FD_SKU="Standard_AzureFrontDoor"

echo "Creating Azure Front Door profile (Standard SKU)..."
az afd profile create \
  --resource-group $RESOURCE_GROUP \
  --profile-name $FD_PROFILE_NAME \
  --location $LOCATION \
  --sku $FD_SKU \
  --output tsv
```

#### 3.1.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$location = "<YourAzureRegion>"
$fdProfileName = "fd-web-prod"

Write-Host "Creating Azure Front Door profile (Standard SKU)..."
New-AzFrontDoorCdnProfile -ResourceGroupName $resourceGroupName -ProfileName $fdProfileName `
    -SkuName Standard_AzureFrontDoor -Location $location
```

### 3.2 Configure Endpoint

The endpoint is the publicly accessible URL for your Front Door.

#### 3.2.1 Using Azure Portal

1.  In the **Front Door manager** of your `fd-web-prod` profile, click **+ Add an endpoint**.
2.  **Name**: `fd-endpoint-prod` (or a unique name).
3.  Click **Add**.

#### 3.2.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
FD_PROFILE_NAME="fd-web-prod"
FD_ENDPOINT_NAME="fd-endpoint-prod"

echo "Creating Front Door endpoint..."
az afd endpoint create \
  --resource-group $RESOURCE_GROUP \
  --profile-name $FD_PROFILE_NAME \
  --endpoint-name $FD_ENDPOINT_NAME \
  --enabled-state Enabled \
  --output tsv
```

#### 3.2.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$fdProfileName = "fd-web-prod"
$fdEndpointName = "fd-endpoint-prod"

Write-Host "Creating Front Door endpoint..."
New-AzFrontDoorCdnEndpoint -ResourceGroupName $resourceGroupName -ProfileName $fdProfileName `
    -EndpointName $fdEndpointName -IsCompressionEnabled:$true
```

### 3.3 Configure Origin Group

This is where you define the backend service (your Application Gateway) that Front Door will route traffic to.

#### 3.3.1 Using Azure Portal

1.  In the **Front Door manager**, click **+ Add an Origin group**.
2.  **Name**: `origin-group-appgw`.
3.  **Health Probe path**: `/` (to check the root of the web app).
4.  **Health Probe protocol**: `HTTP`.
5.  **Add Origin**: Click **+ Add**.
      * **Name**: `appgw-origin`.
      * **Origin type**: Select **Custom**.
      * **Host name**: Find the public IP of `appgw-web-prod`. You can find this by navigating to the Public IP resource in the portal. Enter it here (e.g., `20.10.20.30`).
      * **HTTP port**: `80`.
      * **HTTPS port**: `443` (leave as is).
      * **Priority**: `1`.
      * **Weight**: `50`.
      * Click **Add**.
6.  Click **Add** to save the origin group.

#### 3.3.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
FD_PROFILE_NAME="fd-web-prod"
ORIGIN_GROUP_NAME="origin-group-appgw"
APPGW_PUBLIC_IP=$(az network public-ip show -g $RESOURCE_GROUP -n "appgw-web-prod-pip" --query ipAddress -o tsv)

echo "Creating Origin Group..."
az afd origin-group create \
  --resource-group $RESOURCE_GROUP \
  --profile-name $FD_PROFILE_NAME \
  --origin-group-name $ORIGIN_GROUP_NAME \
  --probe-path "/" \
  --probe-protocol Http \
  --probe-interval-in-seconds 100 \
  --output tsv

echo "Adding Application Gateway as an Origin to the Origin Group..."
az afd origin create \
  --resource-group $RESOURCE_GROUP \
  --profile-name $FD_PROFILE_NAME \
  --origin-group-name $ORIGIN_GROUP_NAME \
  --origin-name "appgw-origin" \
  --host-name $APPGW_PUBLIC_IP \
  --http-port 80 \
  --https-port 443 \
  --enabled-state Enabled \
  --output tsv
```

### 3.4 Configure Route

The route links the Front Door endpoint to the origin group.

#### 3.4.1 Using Azure Portal

1.  In the **Front Door manager**, click **+ Add a route**.
2.  **Name**: `route-to-appgw`.
3.  **Endpoint**: Select `fd-endpoint-prod`.
4.  **Domains**: Leave the default Front Door domain.
5.  **Patterns to match**: `/*`.
6.  **Origin group**: Select `origin-group-appgw`.
7.  **Accepted protocols**: `HTTP` (and `HTTPS` if you configured it).
8.  **Redirect**: `No`.
9.  **Forwarding protocol**: `HTTP only`.
10. **Enable caching**: `No` (default, but you can enable this for CDN functionality).
11. Click **Add**.
12. Click **Save** on the Front Door manager page to apply all changes. This can take several minutes to deploy globally.

#### 3.4.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
FD_PROFILE_NAME="fd-web-prod"
FD_ENDPOINT_NAME="fd-endpoint-prod"
ORIGIN_GROUP_NAME="origin-group-appgw"
ROUTE_NAME="route-to-appgw"

echo "Creating Route..."
az afd route create \
  --resource-group $RESOURCE_GROUP \
  --profile-name $FD_PROFILE_NAME \
  --endpoint-name $FD_ENDPOINT_NAME \
  --route-name $ROUTE_NAME \
  --origin-group $ORIGIN_GROUP_NAME \
  --forwarding-protocol HttpOnly \
  --supported-protocols Http \
  --patterns-to-match "/*" \
  --link-to-default-domain Enabled \
  --output tsv
```

*Note: You may need to run `az afd endpoint show` to get the default domain after endpoint creation to properly set up the route.*

### 3.5 Configure Final NSG Rule on Application Gateway

The Application Gateway's NSG needs to allow inbound traffic from the Front Door service. While our App Gateway is public, it's a best practice to lock it down to only accept traffic from Front Door's backend service.

1.  Navigate to the Public IP of your `appgw-web-prod` and note its IP address.
2.  Navigate to your `appgw-web-prod` and select **Networking**.
3.  Click the NSG associated with the Application Gateway's subnet.
4.  Create an Inbound security rule with:
      * **Priority**: A low number, e.g., `100`.
      * **Source**: `Service Tag`.
      * **Source service tag**: `AzureFrontDoor.Backend`.
      * **Source port ranges**: `*`.
      * **Destination**: `Any`.
      * **Destination port ranges**: `80`.
      * **Protocol**: `TCP`.
      * **Action**: `Allow`.
      * **Name**: `Allow-FD-Backend`.
      * Click **Add**.

-----

## Part 4: Testing and Validation

1.  **Find your Front Door Endpoint URL**:
      * In the Azure Portal, navigate to your `fd-web-prod` profile and click **Endpoints**.
      * Copy the Hostname of your `fd-endpoint-prod` (e.g., `fd-endpoint-prod.azurefd.net`).
2.  Open a web browser and navigate to this URL.
3.  **Expected Result**: After a brief delay for DNS propagation, you should see the message from one of your backend web servers (e.g., "Hello from vm-web-01\!").
4.  Refresh the page multiple times. Front Door will forward the requests to the Application Gateway, which in turn performs its own load balancing to `vm-web-01` or `vm-web-02`. The result is that you are successfully accessing your application through the global Front Door endpoint.
5.  **Check Request Headers (Optional)**: If you inspect the request headers in your browser's developer tools, you will see headers like `x-azure-fdid` and `x-azure-fd-forwarded-for`, confirming that the request was processed by Front Door.

-----

## Part 5: Troubleshooting

  * **"Bad Gateway" or "404" Errors**: This typically means Front Door cannot reach or find a healthy origin.
      * **Check Public IP**: Ensure the public IP of your `appgw-web-prod` is correctly entered as the origin's hostname.
      * **Origin Health**: In the Front Door manager, check the **Origin groups** section to see if your `appgw-origin` is reported as healthy.
      * **NSG Rules**: Confirm the `Allow-FD-Backend` rule on your Application Gateway's NSG is correct.
  * **Traffic not load-balanced**: If you only see one VM's page, this is likely because the Application Gateway's load balancing rule is not set to round-robin, or you have session affinity enabled. This is a regional issue, not a Front Door issue.

-----

## Part 6: Cleanup Resources

To avoid incurring ongoing costs, it's crucial to delete the resources created in this lab.

**(Using Azure CLI - Recommended for comprehensive cleanup)**

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"

echo "Deleting resource group: $RESOURCE_GROUP and all its contents..."
az group delete --name $RESOURCE_GROUP --yes --no-wait
echo "Deletion process initiated. It may take some time to complete."
```

**(Using Azure Portal)**

1.  **Delete the Front Door Profile**:
      * Search for `Front Doors`.
      * Select `fd-web-prod` and click **Delete**.
2.  **Delete the Application Gateway**:
      * Select `appgw-web-prod` and click **Delete**.
3.  **Delete the VMs**:
      * Select `vm-web-01` and `vm-web-02` and click **Delete**.
4.  **Delete the Resource Group**:
      * If you're done with all labs in `rg-network-lab`, the most efficient way is to delete the entire resource group as detailed in previous labs.

-----

Congratulations\! You have successfully deployed and configured Azure Front Door, creating a global, multi-layered networking architecture that can accelerate and secure your web application.