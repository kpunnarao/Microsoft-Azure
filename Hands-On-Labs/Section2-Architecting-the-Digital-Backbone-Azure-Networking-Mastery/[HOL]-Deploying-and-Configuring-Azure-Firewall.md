# Lab Guide: Deploying and Configuring Azure Firewall 🛡️

This lab will guide you through deploying and configuring Azure Firewall, a stateful, managed firewall as a service.

You will learn how to secure your virtual network by centrally managing and controlling all inbound and outbound network traffic using a combination of DNAT, Network, and Application rules.

## Lab Scenario 🧪

Your security team has mandated that all outbound internet traffic from your Azure Virtual Network must pass through a single, managed firewall. 

Additionally, you need to allow inbound RDP access to a virtual machine (VM) via a single public IP address, and control which websites users on your network can access. 

You will achieve this by deploying and configuring an Azure Firewall.

**Your specific objectives for this lab are:**

* Deploy an Azure Firewall into a dedicated subnet.

* Configure a route table to force all outbound traffic from your workload subnet through the firewall.

* Create a DNAT rule to allow inbound RDP access to a VM.

* Create a Network rule to allow outbound `ping` traffic.

* Create an Application rule to allow outbound web traffic to a specific FQDN.

* Validate that all rules are working as expected.

## Part 1: Prerequisites

* An active Azure subscription.

* **Existing VNet and VM**: You will use a VNet (e.g., `vnet-lab` with address space `10.0.0.0/16`) and a VM (e.g., `vm-app-01`) from previous labs.

* **Azure Cloud Shell / Azure CLI / Azure PowerShell**: For managing Azure resources.

* **A note on pricing**: Azure Firewall is a paid service. Be mindful of costs and delete the resources as soon as you are done with the lab.

-----

## Part 2: Architecture Diagram

The following diagram illustrates the network flow you will configure in this lab:

```
+------------------------------------------------------------------------------------------------+
|                                    Azure Virtual Network: `vnet-lab`                             |
|                                                                                                |
|   +--------------------------+                                                               |
|   |   Public Internet        |                                                               |
|   |                          |                                                               |
|   |  +-------------------+   |   (DNAT Rule)                                                 |
|   |  | Azure Firewall    |   | <----------------------------------------------------------+  |
|   |  | Public IP         |   |                                                           |  |
|   |  +-------------------+   |                                                           |  |
|   |            |             |                                                           |  |
|   |            v             |    (AzureFirewallSubnet)      (Route Table)               |  |
|   |   +------------------+   |   +--------------------------+      +-----------------+  |  |
|   |   | Azure Firewall   |   |   | Firewall Private IP      |  --> |    0.0.0.0/0    |  |  |
|   |   | (`fw-lab`)       |---+---+ `10.0.255.4`             |  |   | Next Hop: `VA`  |  |  |
|   |   | Private IP       |   |   +--------------------------+  |   +-----------------+  |  |
|   |   +------------------+   |                                   |                     |  |
|   |            ^             |                                   v                     |  |
|   |            |             |    (default subnet)               |                     |  |
|   |   +------------------+   |   +--------------------------+  |   +------------------+  |
|   |   |   `vm-app-01`    |   |   | Private IP: `10.0.1.4`   |  +-->|   ping www.bing.com  |
|   |   |                  |---+---|                          |------>|   HTTP/S Allowed   |
|   |   +------------------+   |   +--------------------------+  |   +------------------+
|   |   |                        |                                   |
|   |   |                        +-----------------------------------+
|   |   |
+------------------------------------------------------------------------------------------------+
```

-----

## Part 3: Step-by-Step Configuration: Deploying and Configuring Azure Firewall

### 3.1 Prepare the Azure VNet and Deploy the Firewall

Azure Firewall requires a dedicated subnet named `AzureFirewallSubnet`.

#### 3.1.1 Using Azure Portal

1.  Navigate to your `vnet-lab` virtual network.
2.  Under **Settings**, select **Subnets**.
3.  Click **+ Subnet** and fill in the details:
      * **Name**: `AzureFirewallSubnet` (This name is mandatory).
      * **Subnet address range**: `10.0.255.0/26` (A `/26` or larger is required).
4.  Click **Save**.
5.  Now, deploy the firewall: Search for `Firewalls` and click **+ Create**.
      * **Subscription**: Your subscription.
      * **Resource Group**: `rg-network-lab`.
      * **Firewall name**: `fw-lab`.
      * **Region**: The same region as your `vnet-lab`.
      * **Firewall SKU**: `Standard`.
      * **Firewall management**: Select `Use a Firewall Policy` and create a new policy named `fw-lab-policy`.
      * **Virtual network**: Select your `vnet-lab`.
      * **Public IP address**: Create a new public IP with name `fw-lab-pip`.
6.  Click **Review + create**, then **Create**. This deployment will take 20-30 minutes.

### 3.2 Configure a Default Route Table

To force all outbound traffic from your VM through the firewall, you must create and associate a route table with its subnet.

#### 3.2.1 Using Azure Portal

1.  While the firewall is deploying, search for `Route tables` and click **+ Create**.
      * **Subscription**: Your subscription.
      * **Resource Group**: `rg-network-lab`.
      * **Region**: The same region as your other resources.
      * **Name**: `firewall-rt`.

2.  Click **Review + create**, then **Create**.

3.  Once deployed, go to the `firewall-rt` resource and select **Subnets**.

4.  Click **Associate**, select your `vnet-lab` and the subnet where your VM resides (e.g., `default`). Click **OK**.

5.  Now, go back to the firewall resource `fw-lab` and note its **Private IP address** from the Overview pane.

6.  Navigate back to the `firewall-rt` route table and select **Routes**.

7.  Click **+ Add** and fill in the details:
      * **Route name**: `fw-default-route`.
      * **Destination type**: `IP Addresses`.
      * **Destination IP addresses/CIDR ranges**: `0.0.0.0/0`.
      * **Next hop type**: `Virtual appliance`.
      * **Next hop address**: Enter the private IP of your firewall.

8.  Click **Add**.

### 3.3 Configure Firewall Rules

Now we will create the rules in the `fw-lab-policy` to control traffic.

#### 3.3.1 Using Azure Portal

1.  Navigate to your `fw-lab-policy` firewall policy resource.

2.  **DNAT Rule**: To allow inbound RDP/SSH.
      * Under **Settings**, select **DNAT Rules**.
      * Click **+ Add a rule collection**.
      * **Name**: `dnat-rules`. **Priority**: `100`.
      * Under **Rules**, fill in the details:
          * **Name**: `dnat-vm-app-01`.
          * **Rule collection group**: `DefaultDnatRuleCollectionGroup`.
          * **Source type**: `IP Address`. **Source**: `*`.
          * **Protocol**: `TCP`.
          * **Destination type**: `IP Address`. **Destination**: Enter the public IP address of your firewall.
          * **Destination ports**: `3389` (for RDP).
          * **Translated address**: The private IP of your `vm-app-01`.
          * **Translated port**: `3389`.
      * Click **Add**.

3.  **Network Rule**: To allow outbound ping.
      * Under **Settings**, select **Network Rules**.
      * Click **+ Add a rule collection**.
      * **Name**: `network-rules`. **Priority**: `200`. **Rule collection group**: `DefaultNetworkRuleCollectionGroup`.
      * **Rules**:
          * **Name**: `allow-ping`.
          * **Protocol**: `ICMP`.
          * **Source type**: `IP Address`. **Source**: `10.0.1.0/24` (the subnet of your VM).
          * **Destination type**: `IP Address`. **Destination**: `*`.
          * **Destination ports**: `*`.
      * Click **Add**.

4.  **Application Rule**: To allow outbound HTTP/S to a specific FQDN.
      * Under **Settings**, select **Application Rules**.
      * Click **+ Add a rule collection**.
      * **Name**: `app-rules`. **Priority**: `300`. **Rule collection group**: `DefaultApplicationRuleCollectionGroup`.
      * **Rules**:
          * **Name**: `allow-bing`.
          * **Protocol**: `http:80,https:443`.
          * **Source type**: `IP Address`. **Source**: `10.0.1.0/24`.
          * **Target FQDNs**: `www.bing.com`.
      * Click **Add**.

-----

## Part 4: Testing and Validation

1.  **Test Inbound Connectivity (DNAT)**: From your local machine, open a remote desktop client and try to connect to the **public IP address of your firewall** on port `3389`. This should successfully connect you to your `vm-app-01`.

2.  **Test Outbound Connectivity (Network & Application)**:
      * From within `vm-app-01`, open a command prompt.
      * Try to ping a public IP address: `ping 8.8.8.8` (Google's DNS server). This should **succeed** because of your Network rule.
      * Now, try to ping an FQDN: `ping www.google.com`. This should **fail** because your firewall policy does not allow it.
      * Open a web browser on the VM and try to navigate to `www.bing.com`. This should **succeed** because of your Application rule.
      * Now, try to navigate to `www.google.com`. This should **fail**.

-----

## Part 5: Troubleshooting

* **DNAT rule is not working**: Ensure the route table is correctly associated, and the `Next hop` is the firewall's private IP. Also, check that the destination IP in the DNAT rule is the firewall's public IP.

* **Outbound traffic is blocked**: Check that the `Next hop address` in your route table is the correct private IP of the firewall. Remember that all outbound traffic is denied by default unless you have an explicit rule to allow it.

-----

## Part 6: Cleanup Resources

1.  **Delete the Route Table Association**: Navigate to the `firewall-rt` route table, go to **Subnets**, select the `default` subnet, and click **Dissociate**.

2.  **Delete the Firewall and Route Table**: Delete the `fw-lab` firewall and the `firewall-rt` route table.

3.  **Delete the Subnet**: Navigate to your `vnet-lab`, select **Subnets**, and delete the `AzureFirewallSubnet`.

4.  Finally, delete the `rg-network-lab` resource group to remove all remaining resources.

Congratulations\! You have successfully deployed and configured Azure Firewall to secure network traffic in your virtual network.