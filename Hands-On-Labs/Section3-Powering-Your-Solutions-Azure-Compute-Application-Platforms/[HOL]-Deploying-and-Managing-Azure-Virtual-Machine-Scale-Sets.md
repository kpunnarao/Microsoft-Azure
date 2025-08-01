# Lab: Deploying and Managing Azure Virtual Machine Scale Sets 🚀

This lab will guide you through the deployment and management of an Azure Virtual Machine Scale Set (VMSS) using the Azure Portal, Azure CLI, and Azure PowerShell. 

VMSS provides a simple way to deploy and manage a group of identical, load-balanced virtual machines.

## Lab Scenario 🧪

You are a cloud administrator tasked with deploying a highly scalable web application. 

To handle unpredictable traffic, you need a solution that can automatically adjust the number of virtual machines to meet demand. 

This lab will demonstrate how to create a VMSS that can scale out and scale in based on CPU usage.

**Your specific objectives for this lab are:**

* Understand the key components of a VMSS.

* Deploy a VMSS using the Azure Portal, Azure CLI, and Azure PowerShell.

* Configure basic autoscaling rules.

* Perform manual scaling and verify the changes.

* Clean up all the created resources to avoid costs.

## Part 1: Prerequisites & Architecture

* An active Azure subscription.

* **Azure Cloud Shell:** This lab uses both Azure CLI and Azure PowerShell, which are pre-installed in the Azure Cloud Shell. Access it from the Azure portal by clicking the `>_` icon.

The following diagram illustrates the resources you will deploy. All three methods will result in this same logical architecture.

```
+------------------------------------------------------------------------------------------------+
|                             Azure Resource Group: `rg-vmss-lab`                                |
|                                                                                                |
|   +----------------------------------------------------------------------------------------+   |
|   |   Public Internet IP Address                                                           |   |
|   |   +-------------------+                                                                |   |
|   |   | Load Balancer     |                                                                |   |
|   |   +---------+---------+                                                                |   |
|   |             | (Traffic Distribution)                                                   |   |
|   |             |                                                                          |   |
|   |             v                                                                          |   |
|   |   +--------------------------------------------------------------------------------+   |   |
|   |   |                 Azure Virtual Machine Scale Set (`vmss-lab`)                   |   |   |
|   |   |                                                                                |   |   |
|   |   |   +---------------+   +---------------+   +---------------+                    |   |   |
|   |   |   |   VM Instance 1 |   |   VM Instance 2 |   |   VM Instance 3 |    (N instances)   |   |
|   |   |   +---------------+   +---------------+   +---------------+                    |   |   |
|   |   |                                 ^  (Autoscaling Rules)                         |   |   |
|   |   |---------------------------------|----------------------------------------------|   |   |
|   |                       (Monitors CPU % and other metrics)                             |   |
|   |                                                                                      |   |
|   +----------------------------------------------------------------------------------------+   |
|                                                                                                |
+------------------------------------------------------------------------------------------------+
```

-----

## Part 2: Common Setup (Create the Resource Group)

All three deployment methods require a resource group. We will create it using the Azure CLI.

1.  **Set Variables**:

    ```bash
    RESOURCE_GROUP="rg-vmss-lab"
    LOCATION="eastus" # Replace with a region close to you
    VMSS_NAME="vmss-lab-01"
    ```

2.  **Create the Resource Group**:

    ```bash
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

-----

## Part 3: Method 1: Deploying via the Azure Portal (GUI)

This method is the most visual and beginner-friendly.

1.  **Navigate to the Azure Portal**: Go to `portal.azure.com`.

2.  **Create a Virtual Machine Scale Set**: In the search bar at the top, type `Virtual machine scale sets` and select it. Then, click **+ Create**.

3.  **Basics Tab**:
      * **Subscription**: Select your subscription.
      * **Resource Group**: Choose `rg-vmss-lab`.
      * **Virtual machine scale set name**: `vmss-lab-01-portal`.
      * **Region**: `(US) East US`.
      * **Orchestration mode**: `Uniform`.
      * **Image**: Select `Ubuntu Server 22.04 LTS`.
      * **Size**: `Standard_B1s`.
      * **Authentication type**: `SSH public key`.
      * **Username**: `azureuser`.
      * **SSH public key source**: Select `Generate new key pair`.
      * **Key pair name**: `labvmsskey`.
4.  **Scaling Tab**:
      * **Initial instance count**: `2`.
      * **Scaling policy**: `Custom`.
      * **Scale out rule**:
          * **Metric source**: `Current`.
          * **Metric**: `CPU percentage`.
          * **Operator**: `Greater than`.
          * **Threshold**: `75`.
          * **Duration (minutes)**: `5`.
          * **Action**: `Increase count by 1`.
      * **Scale in rule**:
          * **Metric**: `CPU percentage`.
          * **Operator**: `Less than`.
          * **Threshold**: `25`.
          * **Duration (minutes)**: `5`.
          * **Action**: `Decrease count by 1`.
      * **Instance limits**: `Minimum instances: 2`, `Maximum instances: 5`.
5.  **Networking Tab**: The default settings are fine.
      * **Public inbound ports**: `Allow selected ports`.
      * **Select inbound ports**: `SSH (22)`.
6.  **Review + create**: Click this button, review the settings, and then click **Create**. The portal will download a private key file named `labvmsskey.pem`. Save this file.
7.  **Deployment**: The deployment will take several minutes as it creates the VMSS, load balancer, and two VM instances.

-----

## Part 4: Method 2: Deploying via Azure CLI

This method is fast, efficient, and great for automation. We will create a new VMSS to avoid conflicts.

1.  **Set VMSS Name Variable**:

    ```bash
    VMSS_NAME="vmss-lab-01-cli"
    ```

2.  **Create the Virtual Machine Scale Set**: This single command will automatically create all required resources.

    ```bash
    az vmss create \
      --resource-group $RESOURCE_GROUP \
      --name $VMSS_NAME \
      --image Ubuntu2204 \
      --admin-username azureuser \
      --instance-count 2 \
      --size Standard_B1s \
      --generate-ssh-keys \
      --lb-sku Standard \
      --output table
    ```

* The command will generate a new SSH key pair and save the private key in the `~/.ssh` directory of your Cloud Shell.

3.  **Deployment**: The deployment will take a few minutes. The output will provide a table confirming the creation of the VMSS.

-----

## Part 5: Method 3: Deploying via Azure PowerShell

This method is more detailed and object-oriented, giving you fine-grained control over each resource. We will use the simplified parameter set.

1.  **Set Variables**:

    ```powershell
    $resourceGroup = "rg-vmss-lab"
    $location = "eastus"
    $vmssName = "vmss-lab-01-ps"
    $vmSize = "Standard_B1s"
    $image = "UbuntuServer"
    $credential = Get-Credential # You will be prompted to enter a username and password
    ```

2.  **Create the Virtual Machine Scale Set**:

    ```powershell
    New-AzVmss -ResourceGroupName $resourceGroup `
        -VMScaleSetName $vmssName `
        -Location $location `
        -SkuName $vmSize `
        -ImageName $image `
        -InstanceCount 2 `
        -Credential $credential
    ```

3.  **Deployment**: The deployment will take several minutes. PowerShell will output the details of the created VMSS.

-----

## Part 6: Managing and Scaling the Scale Set

1.  **View Instances**: You can see the instances in the Azure Portal by navigating to your VMSS resource. You can also use the CLI:

    ```bash
    az vmss list-instances --resource-group $RESOURCE_GROUP --name vmss-lab-01-cli --output table
    ```

2.  **Manually Scale the VMSS**:

      * **Scale up to 3 instances**:
        ```bash
        az vmss scale --resource-group $RESOURCE_GROUP --name vmss-lab-01-cli --new-capacity 3
        ```
      * **Scale down to 2 instances**:
        ```bash
        az vmss scale --resource-group $RESOURCE_GROUP --name vmss-lab-01-cli --new-capacity 2
        ```

3.  **Verify Autoscaling Rules (Conceptual)**: To test the autoscaling rules you set up in the portal, you would need to generate load on the VMs. For example, using a tool like `stress-ng` from within a VM. You would then monitor the instance count. We will not do this in this lab, but you can view the scaling history in the Azure Portal under your VMSS resource -\> **Scaling** -\> **Run History**.

-----

## Part 7: Cleanup Resources

This is a crucial step to avoid unnecessary costs. Deleting the resource group will remove all VMs, load balancers, and other associated resources.

1.  **Delete the Resource Group**:
      * **Azure Portal**: Go to the `rg-vmss-lab` resource group and click **Delete resource group**.
      * **Azure CLI**: `az group delete --name $RESOURCE_GROUP --yes --no-wait`
      * **Azure PowerShell**: `Remove-AzResourceGroup -Name $RESOURCE_GROUP -Force`

Congratulations\! You have successfully deployed and managed Azure Virtual Machine Scale Sets using three different methods, demonstrating your ability to provision scalable and resilient infrastructure.