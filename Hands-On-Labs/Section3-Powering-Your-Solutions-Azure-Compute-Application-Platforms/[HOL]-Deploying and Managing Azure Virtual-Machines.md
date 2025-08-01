# Lab Guide: Deploying and Managing Azure Virtual Machines 💻

## Lab Scenario 🧪

You are a cloud administrator tasked with provisioning a new application server in Azure. The server needs to be a Linux-based virtual machine, accessible from the internet for initial setup and configuration. 

This lab will walk you through the entire process, from creation to deletion, using different tools.

**Your specific objectives for this lab are:**

* Understand the core components of an Azure VM.

* Create a resource group as a container for your resources.

* Deploy a VM using the Azure Portal, Azure CLI, and Azure PowerShell.

* Connect to the VM using SSH.

* Perform basic management tasks (e.g., stopping and starting the VM).

* Clean up all the created resources to avoid costs.

## Part 1: Prerequisites & Architecture

* An active Azure subscription.

* **Azure Cloud Shell:** This lab uses both Azure CLI and Azure PowerShell, which are pre-installed in the Azure Cloud Shell. Access it from the Azure portal by clicking the `>_` icon.

* **Basic knowledge of SSH:** Familiarity with the SSH protocol is helpful.

The following diagram illustrates the resources you will deploy. All three methods will result in this same logical architecture.

```
+------------------------------------------------------------------------------------------------+
|                             Azure Resource Group: `rg-vm-lab`                                  |
|                                                                                                |
|   +----------------------------------------------------------------------------------------+   |
|   |                        Azure Virtual Network: `vnet-vm-lab`                            |   |
|   |                                                                                        |   |
|   |      +-------------------+                                +-------------------+       |   |
|   |      |  Public IP Address |                                |  `vm-lab-01`      |       |   |
|   |      |  (e.g., 20.x.x.x)  +--------------------------------> (OS Disk & Data)  |       |   |
|   |      +---------+---------+   (Network Interface, NSG)       +-------------------+       |   |
|   |                |                                                                        |   |
|   |    (Internet Traffic)                                                                    |   |
|   |                                                                                        |   |
|   +----------------------------------------------------------------------------------------+   |
|                                                                                                |
+------------------------------------------------------------------------------------------------+
```

-----

## Part 2: Common Setup (Create the Resource Group)

All three deployment methods require a resource group. We will create it using Azure CLI.

1.  **Set Variables**:

    ```bash
    RESOURCE_GROUP="rg-vm-lab"
    LOCATION="eastus" # Replace with a region close to you
    VM_NAME="vm-lab-01"
    ```

2.  **Create the Resource Group**:

    ```bash
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

-----

## Part 3: Method 1: Deploying via the Azure Portal (GUI)

This method is the most visual and beginner-friendly.

1.  **Navigate to the Azure Portal**: Go to `portal.azure.com`.
2.  **Create a Virtual Machine**: In the search bar at the top, type `Virtual machines` and select it. Then, click **+ Create** -\> **Azure virtual machine**.
3.  **Basics Tab**:
      * **Subscription**: Select your subscription.
      * **Resource Group**: Choose `rg-vm-lab`.
      * **Virtual machine name**: `vm-lab-01-portal`.
      * **Region**: `(US) East US`.
      * **Image**: Select `Ubuntu Server 22.04 LTS`.
      * **Size**: `Standard_B1s`.
      * **Authentication type**: `SSH public key`.
      * **Username**: `azureuser`.
      * **SSH public key source**: Select `Generate new key pair`.
      * **Key pair name**: `labkey`.
4.  **Networking Tab**: The default settings are fine for this lab. Azure will automatically create a VNet, subnet, public IP, and NSG.
      * **Public inbound ports**: `Allow selected ports`.
      * **Select inbound ports**: `SSH (22)`.
5.  **Review + create**: Click this button, review the settings, and then click **Create**. The portal will download a private key file named `labkey.pem`. Save this file.
6.  **Deployment**: The deployment will take a few minutes. Once complete, navigate to the `vm-lab-01-portal` resource.

-----

## Part 4: Method 2: Deploying via Azure CLI

This method is fast, efficient, and great for automation. We will create a new VM to avoid conflicts with the previous one.

1.  **Set VM Name Variable**:

    ```bash
    VM_NAME="vm-lab-01-cli"
    ```

2.  **Create the Virtual Machine**: This single command will automatically create all required resources (VNet, public IP, NSG).

    ```bash
    az vm create \
      --resource-group $RESOURCE_GROUP \
      --name $VM_NAME \
      --image Ubuntu2204 \
      --admin-username azureuser \
      --size Standard_B1s \
      --generate-ssh-keys
    ```

      * The command will generate a new SSH key pair and save the public and private keys in the `~/.ssh` directory of your Cloud Shell.

3.  **Deployment**: The deployment will take a few minutes. The output will provide a JSON object with the VM details.

-----

## Part 5: Method 3: Deploying via Azure PowerShell

This method is more detailed and object-oriented, giving you fine-grained control over each resource.

1.  **Set Variables**:

    ```powershell
    $resourceGroup = "rg-vm-lab"
    $location = "eastus"
    $vmName = "vm-lab-01-ps"
    $vnetName = "vnet-ps"
    $subnetName = "subnet-ps"
    $ipName = "ip-ps"
    $nicName = "nic-ps"
    $vmSize = "Standard_B1s"
    $image = "Canonical:0001-com-ubuntu-server-jammy:22_04-lts:latest"
    $cred = Get-Credential # You will be prompted to enter a username and password
    ```

    *The `$cred` variable will pop up a window asking for a username and password. This is for RDP/SSH access.*

2.  **Create Network Resources**:

    ```powershell
    $vnet = New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroup -Location $location -AddressPrefix "10.0.0.0/16"
    $subnet = Add-AzVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix "10.0.1.0/24" -VirtualNetwork $vnet
    $vnet | Set-AzVirtualNetwork
    $pip = New-AzPublicIpAddress -Name $ipName -ResourceGroupName $resourceGroup -Location $location -AllocationMethod Dynamic
    $nic = New-AzNetworkInterface -Name $nicName -ResourceGroupName $resourceGroup -Location $location -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id
    ```

3.  **Create VM Configuration**:

    ```powershell
    $vmConfig = New-AzVMConfig -VMName $vmName -VMSize $vmSize
    $vmConfig = Set-AzVMSourceImage -VM $vmConfig -PublisherName "Canonical" -Offer "0001-com-ubuntu-server-jammy" -Skus "22_04-lts" -Version "latest"
    $vmConfig = Set-AzVMOSDisk -VM $vmConfig -CreateOption "FromImage" -StorageAccountType "Standard_LRS"
    $vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id
    $vmConfig = Set-AzVMOperatingSystem -VM $vmConfig -Linux -ComputerName $vmName -Credential $cred
    ```

4.  **Create the VM**:

    ```powershell
    New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig
    ```

-----

## Part 6: Management & Cleanup

1.  **Retrieve Public IP**:

      * **CLI**: `az vm show --resource-group $RESOURCE_GROUP --name $VM_NAME --query "publicIps" -o tsv`
      * **PowerShell**: `(Get-AzPublicIpAddress -ResourceGroupName $RESOURCE_GROUP).IpAddress`

2.  **Stop/Start VM**:

      * **CLI**: `az vm stop --resource-group $RESOURCE_GROUP --name $VM_NAME` and `az vm start --resource-group $RESOURCE_GROUP --name $VM_NAME`
      * **PowerShell**: `Stop-AzVM -ResourceGroupName $RESOURCE_GROUP -Name $VM_NAME` and `Start-AzVM -ResourceGroupName $RESOURCE_GROUP -Name $VM_NAME`

3.  **Delete Resources**:

      * **Azure Portal**: Go to the `rg-vm-lab` resource group and click **Delete resource group**.
      * **Azure CLI**: `az group delete --name $RESOURCE_GROUP --yes --no-wait`
      * **Azure PowerShell**: `Remove-AzResourceGroup -Name $RESOURCE_GROUP -Force`