# Lab: Deploying and Managing a Linux VM with Data Disks 🐧

## Lab Scenario 🧪

You are a cloud administrator tasked with provisioning a new Linux application server in Azure. The server requires additional storage for application data. 

This lab will guide you through the process of creating the VM, managing its state, attaching a new data disk, and preparing it for use within the operating system.

**Your specific objectives for this lab are:**

* Create a new Linux Virtual Machine.

* Manage the VM's power state (start/stop/deallocate).

* Attach an empty data disk to the VM.

* Connect to the VM and use Linux commands to partition, format, and mount the new disk.

* Clean up all the created resources to avoid costs.

## Part 1: Prerequisites & Architecture

* An active Azure subscription.

* **Azure Cloud Shell:** This lab uses both Azure CLI and Azure PowerShell, which are pre-installed in the Azure Cloud Shell. Access it from the Azure portal by clicking the `>_` icon.

* **SSH Client:** A local SSH client is required to connect to the Linux VM. The Cloud Shell has one built-in.

## Part 2: Common Setup (Create the Resource Group)

All three deployment methods require a resource group. We will create it using Azure CLI.

1.  **Open Azure Cloud Shell** and select **Bash** environment.

2.  **Set Variables**:

    ```bash
    RESOURCE_GROUP="rg-linux-lab"
    LOCATION="eastus" # Replace with a region close to you
    VM_NAME="linux-vm-01"
    ```

3.  **Create the Resource Group**:

    ```bash
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

-----

## Part 3: Create the Linux VM

Choose one of the three methods below to create your VM.

### Method 1: Azure Portal (GUI) 🌐

1.  Navigate to the Azure Portal and search for `Virtual machines`. Click **+ Create** -\> **Azure virtual machine**.
2.  **Basics Tab**:
      * **Subscription**: Select your subscription.
      * **Resource Group**: Choose `rg-linux-lab`.
      * **Virtual machine name**: `linux-vm-01-portal`.
      * **Region**: `(US) East US` (or the region you chose for your resource group).
      * **Image**: Select `Ubuntu Server 22.04 LTS`.
      * **Size**: `Standard_B1s` (cost-effective).
      * **Authentication type**: `SSH public key`.
      * **Username**: `azureuser`.
      * **SSH public key source**: Select `Generate new key pair`.
      * **Key pair name**: `my-linux-key`.
3.  Click **Review + create**.
4.  Review the settings, then click **Create**. A pop-up will prompt you to download the private key. Click **Download private key and create resource**. Save the `.pem` file to a secure location on your local machine.

### Method 2: Azure CLI 🖥️

1.  Open your Azure Cloud Shell and ensure you are in the **Bash** environment.
2.  **Set VM Name Variable**:
    ```bash
    VM_NAME="linux-vm-01-cli"
    ```
3.  **Create the VM**: This single command will automatically create all required network resources.
    ```bash
    az vm create \
      --resource-group $RESOURCE_GROUP \
      --name $VM_NAME \
      --image Ubuntu2204 \
      --admin-username azureuser \
      --size Standard_B1s \
      --generate-ssh-keys
    ```
    *The command will generate a new SSH key pair and save the private key in your Cloud Shell's `~/.ssh` directory.*

### Method 3: Azure PowerShell 💻

1.  Open your Azure Cloud Shell and ensure you are in the **PowerShell** environment.
2.  **Set Variables**:
    ```powershell
    $resourceGroup = "rg-linux-lab"
    $location = "eastus"
    $vmName = "linux-vm-01-ps"
    $vmSize = "Standard_B1s"
    $image = "Canonical:0001-com-ubuntu-server-jammy:22_04-lts:latest"
    $adminUsername = "azureuser"
    ```
3.  **Create SSH Key and VM**:
    ```powershell
    # The New-AzVM command can automatically generate a key pair
    New-AzVm -ResourceGroupName $resourceGroup `
        -Name $vmName `
        -Location $location `
        -Size $vmSize `
        -ImageName $image `
        -Credential (New-Object System.Management.Automation.PSCredential($adminUsername, (Read-Host "Enter Password" -AsSecureString))) `
        -GenerateSshKey
    ```
    *You will be prompted to enter a password for the VM. Remember it.*

-----

## Part 4: VM State Management

### Method 1: Azure Portal 🌐

1.  Navigate to your VM's **Overview** blade.
2.  Click the **Stop** button at the top to deallocate the VM. The VM will stop and you will no longer be charged for its compute time (storage costs will remain).
3.  Click the **Start** button to restart the VM.

### Method 2: Azure CLI 🖥️

```bash
# Stop the VM
az vm stop --resource-group $RESOURCE_GROUP --name linux-vm-01-cli

# Start the VM
az vm start --resource-group $RESOURCE_GROUP --name linux-vm-01-cli
```

### Method 3: Azure PowerShell 💻

```powershell
# Stop the VM
Stop-AzVM -ResourceGroupName $resourceGroup -Name linux-vm-01-ps -Force

# Start the VM
Start-AzVM -ResourceGroupName $resourceGroup -Name linux-vm-01-ps
```

-----

## Part 5: Attach and Prepare a Data Disk

Now, let's add a data disk to the VM you created. We will use the Azure CLI for this step, as it's a quick and efficient way to demonstrate the process. The preparation steps are done inside the VM and are identical regardless of the creation method.

### 5.1. Attach the Data Disk (using Azure CLI)

1.  **Attach a new 4 GiB disk** to your VM. Make sure the VM is running.

    ```bash
    # Let's use the VM created with the CLI
    VM_NAME="linux-vm-01-cli"
    az vm disk attach --resource-group $RESOURCE_GROUP --vm-name $VM_NAME --name data-disk-01 --size-gb 4 --new
    ```

2.  **Verify the disk is attached** by navigating to your VM resource in the Azure Portal and checking the **Disks** blade.

### 5.2. Connect to the VM and Prepare the Disk

1.  **Get the Public IP address** of your VM.

    ```bash
    az vm show --resource-group $RESOURCE_GROUP --name $VM_NAME --show-details --query "publicIps" -o tsv
    ```

2.  **SSH into the VM** using the public IP and your private key.

    ```bash
    # For CLI-created VM
    ssh -i ~/.ssh/id_rsa azureuser@<YOUR_PUBLIC_IP>

    # For Portal-created VM
    # ssh -i /path/to/my-linux-key.pem azureuser@<YOUR_PUBLIC_IP>
    ```

3.  **Find the new disk** using `lsblk`. Your new disk will likely be named `sdc`, `sdd`, or similar, with no partitions.

    ```bash
    lsblk
    ```

    *Output will look something like this:*

    ```
    NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
    sda       8:0   0   30G  0 disk
    ├─sda1    8:1   0 29.9G  0 part /
    ├─sda14   8:14  0    4M  0 part
    └─sda15   8:15  0  106M  0 part /boot/efi
    sdb       8:16  0    4G  0 disk
    ```

    The new disk is `sdb` in this example.

4.  **Create a partition** using `fdisk`.

    ```bash
    sudo fdisk /dev/sdb
    ```

    *Follow these steps in `fdisk`:*

      * Type `n` to create a new partition.
      * Press `Enter` to accept the default partition number.
      * Press `Enter` to accept the default first sector.
      * Press `Enter` to accept the default last sector (using the whole disk).
      * Type `w` to write the changes and exit.

5.  **Format the new partition** with an `ext4` filesystem.

    ```bash
    sudo mkfs.ext4 /dev/sdb1
    ```

6.  **Create a mount point** and mount the disk.

    ```bash
    sudo mkdir /datadrive
    sudo mount /dev/sdb1 /datadrive
    ```

7.  **Verify the mount** with `df -h`.

    ```bash
    df -h
    ```

    *You should see the `/datadrive` listed with a 4.0G size.*

8.  **Configure for automatic mounting on reboot**: To make the disk mount automatically, edit the `/etc/fstab` file. First, get the disk's UUID.

    ```bash
    sudo blkid
    ```

    *Copy the UUID of your new `/dev/sdb1` partition.*

    ```
    # Edit the fstab file. Use nano or vim.
    sudo nano /etc/fstab
    ```

    *Add this line to the end of the file, replacing `<UUID>` with the value you copied:*

    ```
    UUID=<UUID>   /datadrive   ext4   defaults,nofail   1   2
    ```

    *Save and close the file (Ctrl+X, Y, Enter for nano).*

-----

## Part 6: Cleanup Resources (Critical\!)

To avoid incurring ongoing costs, it is essential to delete all resources created in this lab.

### Method 1: Azure Portal 🌐

1.  In the Azure Portal, search for `Resource groups` and select it.
2.  Find and click on your resource group: `rg-linux-lab`.
3.  On the resource group's Overview blade, click **Delete resource group**.
4.  Type the resource group name (`rg-linux-lab`) to confirm, then click **Delete**.

### Method 2: Azure CLI 🖥️

```bash
az group delete --name rg-linux-lab --yes --no-wait
```

### Method 3: Azure PowerShell 💻

```powershell
Remove-AzResourceGroup -Name "rg-linux-lab" -Force
```

Congratulations\! You have successfully created and managed a Linux VM, attached a new data disk, and prepared it for use using all three primary Azure management tools.