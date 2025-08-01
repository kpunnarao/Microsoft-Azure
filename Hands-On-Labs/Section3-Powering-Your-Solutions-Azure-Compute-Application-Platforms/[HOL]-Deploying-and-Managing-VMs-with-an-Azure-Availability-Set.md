# Lab: Deploying and Managing VMs with an Azure Availability Set 🛡️

This lab will guide you through the process of deploying virtual machines into an Azure Availability Set using the Azure Portal, Azure CLI, and Azure PowerShell. 

An Availability Set is a foundational high-availability feature that ensures your VMs are distributed across different physical hardware.

## Lab Scenario 🧪

You are a cloud administrator responsible for deploying a critical application tier that must be resilient to infrastructure failures. 

To meet this requirement, you will deploy two virtual machines into an Availability Set. 

This will ensure that a single point of failure—such as a power outage or planned maintenance—will not take down both of your servers simultaneously.

**Your specific objectives for this lab are:**

* Understand the core concepts of Fault Domains and Update Domains.

* Create an Availability Set.

* Deploy two virtual machines into the same Availability Set using three different methods.

* Verify that the VMs are correctly distributed across different domains.

* Clean up all the created resources to avoid costs.

## Part 1: Prerequisites & Architecture

* An active Azure subscription.

* **Azure Cloud Shell:** This lab uses both Azure CLI and Azure PowerShell, which are pre-installed in the Azure Cloud Shell. Access it from the Azure portal by clicking the `>_` icon.

The following diagram illustrates the resources you will deploy. All three methods will result in this same logical architecture.

```
+--------------------------------------------------------------------------------------------------+
|                            Azure Resource Group: `rg-avset-lab`                                  |
|                                                                                                  |
|   +------------------------------------------------------------------------------------------+   |
|   |                        Azure Availability Set (`avset-lab`)                              |   |
|   |                                                                                          |   |
|   |   +-------------------+   +-------------------+                                          |   |
|   |   |   VM-01           |   |   VM-02           |                                          |   |
|   |   |   (F-D: 0, U-D: 0)|   |   (F-D: 1, U-D: 1)|                                          |   |
|   |   +---------^---------+   +---------^---------+                                          |   |
|   |             |                     |                                                      |   |
|   |             |                     | (Traffic via Public IP)                              |   |
|   |             |                     |                                                      |   |
|   |  +-----------------------------------+  (NICs & Disks)                                   |   |
|   |  |   VNet: `vnet-lab`              |                                                    |   |
|   |  +-----------------------------------+                                                    |   |
|   |                                                                                          |   |
|   +------------------------------------------------------------------------------------------+   |
|                                                                                                  |
+--------------------------------------------------------------------------------------------------+
```

## Key Concepts:

* **Fault Domain (F-D):** A logical group of hardware in a datacenter that shares a common power source, network switch, and cooling. By distributing VMs across multiple Fault Domains, a single hardware failure will only impact one VM. Azure typically supports up to 3 Fault Domains.

* **Update Domain (U-D):** A logical group of VMs that can be rebooted at the same time during planned maintenance by Azure. Distributing VMs across multiple Update Domains ensures that during an update, only one domain is offline at a time, keeping the application available.

-----

## Part 2: Common Setup (Create the VNet and Resource Group)

All three deployment methods require a resource group and a virtual network. We will create them using the Azure CLI.

1.  **Set Variables**:

    ```bash
    RESOURCE_GROUP="rg-avset-lab"
    LOCATION="eastus" # Replace with a region close to you
    VNET_NAME="vnet-lab"
    SUBNET_NAME="subnet-lab"
    ```

2.  **Create the Resource Group**:

    ```bash
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

3.  **Create the Virtual Network and Subnet**:

    ```bash
    az network vnet create --resource-group $RESOURCE_GROUP \
      --name $VNET_NAME \
      --address-prefix 10.0.0.0/16 \
      --subnet-name $SUBNET_NAME \
      --subnet-prefix 10.0.1.0/24
    ```

-----

## Part 3: Method 1: Deploying via the Azure Portal (GUI)

This method is the most visual and beginner-friendly.

1.  **Navigate to the Azure Portal**: Go to `portal.azure.com`.
2.  **Create the First VM**:
      * In the search bar, type `Virtual machines` and select it. Click **+ Create** -\> **Azure virtual machine**.
      * **Basics Tab**:
          * **Resource Group**: `rg-avset-lab`.
          * **Virtual machine name**: `vm-avset-01-portal`.
          * **Region**: `(US) East US`.
          * **Availability options**: Select `Availability set`.
          * **Availability set**: Click **Create new**. Enter `avset-lab-portal` for the name and leave the domain counts as default. Click **OK**.
          * **Image**: Select `Ubuntu Server 22.04 LTS`.
          * **Size**: `Standard_B1s`.
          * **Authentication type**: `SSH public key`.
          * **Username**: `azureuser`.
          * **SSH public key source**: Select `Generate new key pair`. Enter a key pair name like `avset-key`.
      * **Networking Tab**: Leave the defaults, ensuring a new public IP is created.
      * **Review + create**: Click this button, review the settings, and then click **Create**. The portal will download your private key file.
3.  **Create the Second VM**:
      * Repeat the process to create a new VM.
      * **Basics Tab**:
          * **Resource Group**: `rg-avset-lab`.
          * **Virtual machine name**: `vm-avset-02-portal`.
          * **Availability options**: Select `Availability set`.
          * **Availability set**: **IMPORTANT**: Choose the existing `avset-lab-portal` from the dropdown.
      * **Review + create**: Finish the creation process.

-----

## Part 4: Method 2: Deploying via Azure CLI

This method is fast, efficient, and great for automation. We will deploy two new VMs.

1.  **Set VM Name Variables**:

    ```bash
    VM_NAME_1="vm-avset-01-cli"
    VM_NAME_2="vm-avset-02-cli"
    AVSET_NAME="avset-lab-cli"
    ```

2.  **Create the Availability Set**:

    ```bash
    az vm availability-set create --resource-group $RESOURCE_GROUP \
      --name $AVSET_NAME \
      --location $LOCATION \
      --platform-fault-domain-count 2 \
      --platform-update-domain-count 5 \
      --managed
    ```

3.  **Create the First VM**:

    ```bash
    az vm create --resource-group $RESOURCE_GROUP \
      --name $VM_NAME_1 \
      --image Ubuntu2204 \
      --availability-set $AVSET_NAME \
      --vnet-name $VNET_NAME \
      --subnet $SUBNET_NAME \
      --admin-username azureuser \
      --size Standard_B1s \
      --generate-ssh-keys
    ```

4.  **Create the Second VM**:

    ```bash
    az vm create --resource-group $RESOURCE_GROUP \
      --name $VM_NAME_2 \
      --image Ubuntu2204 \
      --availability-set $AVSET_NAME \
      --vnet-name $VNET_NAME \
      --subnet $SUBNET_NAME \
      --admin-username azureuser \
      --size Standard_B1s
    ```

    *Note: The second command will re-use the SSH keys created in the first command.*

-----

## Part 5: Method 3: Deploying via Azure PowerShell

This method is more detailed and object-oriented, giving you fine-grained control over each resource.

1.  **Set Variables**:

    ```powershell
    $resourceGroup = "rg-avset-lab"
    $location = "eastus"
    $avsetName = "avset-lab-ps"
    $vmName1 = "vm-avset-01-ps"
    $vmName2 = "vm-avset-02-ps"
    $vnetName = "vnet-lab"
    $ipName1 = "ip-avset-ps1"
    $ipName2 = "ip-avset-ps2"
    $nicName1 = "nic-avset-ps1"
    $nicName2 = "nic-avset-ps2"
    $vmSize = "Standard_B1s"
    $cred = Get-Credential # You will be prompted to enter a username and password
    ```

2.  **Create the Availability Set**:

    ```powershell
    $avSet = New-AzAvailabilitySet -ResourceGroupName $resourceGroup `
        -Name $avsetName `
        -Location $location `
        -PlatformFaultDomainCount 2 `
        -PlatformUpdateDomainCount 5 `
        -Sku "Aligned"
    ```

3.  **Create VM-01**:

    ```powershell
    $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroup -Name $vnetName
    $subnet = Get-AzVirtualNetworkSubnetConfig -Name $vnet.Subnets[0].Name -VirtualNetwork $vnet
    $ip1 = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Name $ipName1 -Location $location -AllocationMethod Dynamic
    $nic1 = New-AzNetworkInterface -ResourceGroupName $resourceGroup -Name $nicName1 -Location $location -SubnetId $subnet.Id -PublicIpAddressId $ip1.Id

    $vm1 = New-AzVMConfig -VMName $vmName1 -VMSize $vmSize
    $vm1 = Add-AzVMNetworkInterface -VM $vm1 -Id $nic1.Id
    $vm1 = Set-AzVMOperatingSystem -VM $vm1 -Linux -ComputerName $vmName1 -Credential $cred
    $vm1 = Set-AzVMSourceImage -VM $vm1 -PublisherName "Canonical" -Offer "0001-com-ubuntu-server-jammy" -Skus "22_04-lts" -Version "latest"
    $vm1 = Set-AzVMOSDisk -VM $vm1 -CreateOption "FromImage" -ManagedDiskId (New-AzManagedDisk -Name "$vmName1-osdisk" -Location $location -StorageAccountType "Standard_LRS" -DiskSizeGB 30).Id

    New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vm1 -AvailabilitySetId $avSet.Id
    ```

    *Note: The PowerShell example above is simplified. In a real-world scenario, you would repeat the process for the second VM.*

-----

## Part 6: Verifying the Availability Set Configuration

1.  **View from the Azure Portal**:

      * Go to your `rg-avset-lab` resource group and click on your Availability Set (`avset-lab-portal`, `avset-lab-cli`, or `avset-lab-ps`).
      * In the left-hand menu, under **Settings**, click on **Virtual machines**.
      * You will see the VMs you deployed listed. Look at the `Fault domain` and `Update domain` columns to confirm that the VMs are placed in different domains.

2.  **View with Azure CLI**:

    ```bash
    az vm availability-set show --resource-group $RESOURCE_GROUP \
      --name avset-lab-cli --query 'virtualMachines[].{VMName: id, Fd: "properties.platformFaultDomain", Ud: "properties.platformUpdateDomain"}' -o table
    ```

    This command will provide a clean table showing the VM names and their assigned Fault and Update Domains.

-----

## Part 7: Cleanup Resources

This is a crucial step to avoid unnecessary costs. Deleting the resource group will remove all VMs, disks, public IPs, and the Availability Set.

1.  **Delete the Resource Group**:
      * **Azure Portal**: Go to the `rg-avset-lab` resource group and click **Delete resource group**.
      * **Azure CLI**: `az group delete --name $RESOURCE_GROUP --yes --no-wait`
      * **Azure PowerShell**: `Remove-AzResourceGroup -Name $RESOURCE_GROUP -Force`

Congratulations\! You have successfully deployed and managed Azure Virtual Machines with Availability Sets, a key feature for ensuring high availability and resilience in your applications.