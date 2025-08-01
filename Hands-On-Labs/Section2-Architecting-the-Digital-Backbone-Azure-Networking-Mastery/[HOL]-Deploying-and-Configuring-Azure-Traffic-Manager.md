# Lab: Deploying and Configuring Azure Traffic Manager: 🌐

Traffic Manager operates at the DNS level (Layer 3), whereas Front Door operates at the application level (Layer 7). This fundamental difference is crucial for a complete understanding of Azure's global networking capabilities.

* Traffic Manager is a DNS-based load balancer. It directs the user's browser to the correct endpoint's IP address, but the user connects directly to the endpoint, not through the Traffic Manager service itself. This is a key distinction from Front Door.

* It supports a variety of routing methods: `Priority`, `Weighted`, `Performance`, `Geographic`, `Multivalue`, and `Subnet`. For a lab, `Priority` or `Performance` are the most straightforward to demonstrate. `Performance` is particularly effective for showing how traffic is routed to the "closest" endpoint.

* It can use any Internet-facing service as an endpoint, including Azure services (App Service, public IPs), external endpoints, or nested profiles. This is perfect for our lab, where we can use the public IPs of our existing Application Gateway and Load Balancer to simulate two different regional deployments.

* It includes continuous health monitoring to ensure traffic is only directed to healthy endpoints, providing automatic failover.

* The `az network traffic-manager profile create` and `New-AzTrafficManagerProfile` commands, along with `az network traffic-manager endpoint create` and `New-AzTrafficManagerEndpoint`, are the correct tools for CLI and PowerShell.

* The documentation also highlights the importance of testing by checking the resolved DNS name with `nslookup` and then simulating a failure to see the failover in action.


This lab will guide you through the deployment and configuration of an Azure Traffic Manager profile. 

Traffic Manager is a DNS-based global traffic load balancer that enables you to distribute traffic to your public-facing applications across global Azure regions, while providing high availability and responsiveness.

You will learn how Traffic Manager works at the DNS level to direct users to the most appropriate endpoint, contrasting with the proxy-based approach of Azure Front Door.

## Lab Scenario 🧪

Your company's web application is now hosted across two different "regions" (represented by two separate, public-facing services from our previous labs). 

You need a DNS-based solution to intelligently direct user traffic to the closest or most available endpoint to improve performance and provide a failover mechanism. 

You will create a Traffic Manager profile that uses a `Performance` routing method to automatically direct users to the endpoint with the lowest latency.

**Your specific objectives for this lab are:**

* Create a new Azure Traffic Manager profile.

* Configure two "external" endpoints using the public IPs of our existing Application Gateway and Load Balancer.

* Use a `Performance` routing method to demonstrate latency-based traffic distribution.

* Test and validate that users are directed to one of the endpoints based on the DNS resolution.

* Simulate an endpoint failure and observe Traffic Manager's failover behavior.

## Part 1: Prerequisites

* An active Azure subscription.

* **Existing Endpoints**:

    * The **Public IP address** of your `appgw-web-prod` from the Application Gateway lab.

    * The **Public IP address** of the `az-load-balancer-public-pip` from the Azure Load Balancer lab.

    * Both Application Gateway and the public Load Balancer's backend services must be running and serving a web page on port 80.

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

-----

## Part 2: Architecture Diagram (Conceptual)

The following diagram illustrates how Traffic Manager will direct global traffic to our two distinct public endpoints:

```
+------------------------------------------------------------------------------------------------+
|                                        Global Internet                                         |
|                                                                                                |
|   +----------------------------------------------------------------------------------------+   |
|   |                       DNS Query for `<profile-name>.trafficmanager.net`                |   |
|   |                                                                                        |   |
|   |                               +----------------------------+                           |   |
|   +-----------------------------> | Azure Traffic Manager      | <--------------------------+   |
|          (User makes DNS query)   | (Performance Routing)      | (Traffic Manager responds with IP) |
|                                   +----------------------------+                           |   |
|                                         |      |                                           |   |
|                                         |      | (Based on low latency)                    |   |
|                                         v      v                                           |   |
|   +----------------------------------------------------------------------------------------+   |
|   |                                                                                        |   |
|   |     +----------------------------+        +----------------------------+                |   |
|   |     |    Endpoint 1              |        |     Endpoint 2             |                |   |
|   |     | (App Gateway's Public IP)  |        | (Load Balancer's Public IP)|                |   |
|   |     +----------------------------+        +----------------------------+                |   |
|   |          (Conceptually in              (Conceptually in a different                 |   |
|   |           Region 1)                      Region 2)                                 |   |
|   |                                                                                        |   |
+------------------------------------------------------------------------------------------------+
```

-----

## Part 3: Step-by-Step Configuration: Deploying Azure Traffic Manager

### 3.1 Create a Traffic Manager Profile

We will create the main profile and define our routing method. The `Performance` routing method is a great choice for this lab, as it directs users to the closest available endpoint based on network latency.

#### 3.1.1 Using Azure Portal

1.  In the Azure Portal search bar, type `Traffic Manager profiles` and select it.

2.  Click **+ Create** and fill in the details:

    * **Name**: `tm-web-prod` (or a globally unique name). The final URL will be `<name>.trafficmanager.net`.
    
    * **Routing method**: Select **Performance**.
    
    * **Subscription**: Your subscription.
    
    * **Resource group**: `rg-network-lab`.
    
    * **Location**: This is for the resource group metadata and has no effect on the global Traffic Manager service itself.

3.  Click **Create**.

#### 3.1.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
TM_PROFILE_NAME="tm-web-prod"  # Must be globally unique
ROUTING_METHOD="Performance"

echo "Creating Traffic Manager profile..."
az network traffic-manager profile create \
  --resource-group $RESOURCE_GROUP \
  --name $TM_PROFILE_NAME \
  --routing-method $ROUTING_METHOD \
  --unique-dns-name $TM_PROFILE_NAME \
  --output tsv
```

#### 3.1.3 Using Azure PowerShell

```powershell
# Define variables
$resourceGroupName = "rg-network-lab"
$tmProfileName = "tm-web-prod" # Must be globally unique
$routingMethod = "Performance"

Write-Host "Creating Traffic Manager profile..."
New-AzTrafficManagerProfile -ResourceGroupName $resourceGroupName -Name $tmProfileName `
  -TrafficRoutingMethod $routingMethod -RelativeDnsName $tmProfileName -Ttl 30
```

### 3.2 Add Endpoints

Now we'll add our two public-facing services as "external endpoints."

#### 3.2.1 Using Azure Portal

1.  Navigate to your newly created `tm-web-prod` Traffic Manager profile.
2.  In the left-hand menu, click **Endpoints**.
3.  Click **+ Add** and enter the following details for the first endpoint:
      * **Type**: `External endpoint`.
      * **Name**: `appgw-endpoint`.
      * **Target fully qualified domain name (FQDN) or IP**: Enter the Public IP address of your Application Gateway.
      * **Location**: Select the Azure region where your Application Gateway is located (e.g., East US). This is important for the `Performance` routing method.
4.  Click **Add**.
5.  Repeat the process for the second endpoint:
      * **Type**: `External endpoint`.
      * **Name**: `lb-endpoint`.
      * **Target FQDN or IP**: Enter the Public IP address of your Load Balancer.
      * **Location**: Select the Azure region where your Load Balancer is located (e.g., West Europe or another region).
6.  Click **Add**.

#### 3.2.2 Using Azure CLI

```bash
# Define variables
RESOURCE_GROUP="rg-network-lab"
TM_PROFILE_NAME="tm-web-prod"
APPGW_PUBLIC_IP=$(az network public-ip show -g $RESOURCE_GROUP -n "appgw-web-prod-pip" --query ipAddress -o tsv)
LB_PUBLIC_IP=$(az network public-ip show -g $RESOURCE_GROUP -n "az-load-balancer-public-pip" --query ipAddress -o tsv)
LOCATION_APPGW="East US" # Replace with your App Gateway's region
LOCATION_LB="West Europe"  # Replace with your Load Balancer's region

echo "Adding App Gateway endpoint..."
az network traffic-manager endpoint create \
  --resource-group $RESOURCE_GROUP \
  --profile-name $TM_PROFILE_NAME \
  --name "appgw-endpoint" \
  --type ExternalEndpoints \
  --target $APPGW_PUBLIC_IP \
  --endpoint-location "$LOCATION_APPGW" \
  --output tsv

echo "Adding Load Balancer endpoint..."
az network traffic-manager endpoint create \
  --resource-group $RESOURCE_GROUP \
  --profile-name $TM_PROFILE_NAME \
  --name "lb-endpoint" \
  --type ExternalEndpoints \
  --target $LB_PUBLIC_IP \
  --endpoint-location "$LOCATION_LB" \
  --output tsv
```

### 3.3 Configure Monitoring

Traffic Manager uses monitoring to check the health of its endpoints. We will configure it to probe the root path (`/`) on port 80.

1.  In the Traffic Manager profile's left-hand menu, click **Configuration**.
2.  **Protocol**: Select `HTTP`.
3.  **Port**: `80`.
4.  **Path**: `/`.
5.  Click **Save**.
6.  After a few moments, the `Monitoring Status` for your endpoints on the `Endpoints` blade should change from **Checking endpoint** to **Online**.

-----

## Part 4: Testing and Validation

### 4.1 Test DNS Resolution

1.  Navigate to your Traffic Manager profile's **Overview** blade and copy its DNS name (e.g., `tm-web-prod.trafficmanager.net`).
2.  Open a command prompt or terminal on your local machine.
3.  Run the `nslookup` command for the Traffic Manager DNS name:
    ```bash
    nslookup tm-web-prod.trafficmanager.net
    ```
4.  **Expected Result**: You should see the IP address of one of your two endpoints returned. Traffic Manager has used its `Performance` routing method to determine which endpoint is "closest" to your DNS resolver.

### 4.2 Test Endpoint Failover

1.  Choose one of your endpoints (e.g., the one that `nslookup` resolved to).
2.  Go into the backend VMs for that endpoint (e.g., `vm-web-01` and `vm-web-02` behind your Application Gateway) and **stop** them.
3.  Wait a few minutes. Traffic Manager's health probe will detect that the endpoint is no longer healthy.
4.  Go back to your `tm-web-prod` profile in the Azure Portal and check the **Endpoints** blade. The monitoring status for the failed endpoint should change to **Degraded**.
5.  Run the `nslookup` command again on your local machine:
    ```bash
    nslookup tm-web-prod.trafficmanager.net
    ```
6.  **Expected Result**: The DNS resolution should now return the IP address of the *other*, still-healthy endpoint. Traffic Manager has automatically failed over to the next available service.
7.  Once you've verified the failover, **restart** the VMs you stopped.

-----

## Part 5: Troubleshooting

  * **Endpoint Status is Degraded**:
      * Check the health probe settings (protocol, port, path).
      * Ensure the NSG rules for the endpoint's public IP or VNet subnet allow inbound traffic on the specified monitoring port (e.g., 80) from the `AzureTrafficManager` service tag.
      * Confirm the backend service itself is running and responsive.
  * **DNS Resolution Doesn't Change**: DNS results are often cached on your local machine and by your Internet Service Provider. This can delay the failover. You can try clearing your local DNS cache (`ipconfig /flushdns` on Windows) or using a public DNS resolver to confirm the change.

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

1.  **Delete the Traffic Manager Profile**:
      * Search for `Traffic Manager profiles`.
      * Select `tm-web-prod` and click **Delete**.
2.  **Delete the other Resources**:
      * You can now proceed to delete the Application Gateway and Load Balancer from previous labs if you are done with them, or simply keep them for future labs.
      * The most efficient way to clean up is to delete the entire `rg-network-lab` resource group, which will remove everything.

-----

Congratulations\! You have successfully deployed and configured Azure Traffic Manager, demonstrating how it provides intelligent, DNS-based routing and automatic failover for your global applications.