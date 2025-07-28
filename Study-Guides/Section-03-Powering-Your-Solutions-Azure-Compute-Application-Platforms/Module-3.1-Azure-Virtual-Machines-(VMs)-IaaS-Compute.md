# **💻 Module 3.1: Azure Virtual Machines (VMs) Deep Dive - IaaS Compute**

Azure Virtual Machines (VMs) are one of the most fundamental and flexible compute services in Azure. 

They provide Infrastructure-as-a-Service (IaaS), allowing you to create and manage virtualized computers that run on Microsoft's global infrastructure. 

VMs give you the most control over the operating system, applications, and networking components, making them ideal for lift-and-shift migrations, custom application development, and scenarios requiring specific software or operating system configurations.

---

## **What are Azure Virtual Machines (VMs)?**

An Azure VM is a virtualized instance of a computer that you can configure and control much like a physical server. It includes a virtual hard disk (VHD) that stores the operating system and data, virtual network interface cards (NICs) for connectivity, and a set of compute resources (CPU, RAM). You pay for the compute capacity you consume on an hourly basis.

* **Key Features and Core Functionality:**
    * **Full OS Control:** You have full administrative access to the operating system (Windows, Linux, custom OS).
    * **Customization:** Install any software, configure specific drivers, or set up unique network configurations.
    * **Scalability:** Easily scale up (more CPU/RAM) or scale out (more instances) to meet demand, though scaling out is more effectively handled by Virtual Machine Scale Sets (VMSS) (covered in Module 3.2).
    * **High Availability & Disaster Recovery:** Deployable across Availability Zones and Availability Sets for high availability, and support for Azure Site Recovery for disaster recovery.
    * **Integration:** Seamlessly integrates with other Azure services like Azure Networking, Azure Storage, Azure Monitor, and Azure Backup.
    * **Cost Efficiency:** Options like Reserved Instances and Spot VMs provide significant cost savings.

### **Key Components of an Azure VM Deployment**

When you provision an Azure VM, several Azure resources are typically created and associated with it:

1.  **Virtual Machine (VM) Resource:** The core compute resource itself, defining the VM size, operating system image, administrative credentials, and other properties.
2.  **Managed Disk(s):**
    * **Description:** Virtual hard disks (VHDs) that store the VM's operating system (OS Disk) and any data (Data Disks). Azure Managed Disks are fully managed by Azure, simplifying storage management, improving performance, and enhancing availability.
    * **Types (SKUs):**
        * **Standard HDD (Hard Disk Drive):** Cost-effective storage for infrequent access.
        * **Standard SSD (Solid State Drive):** Cost-effective for web servers, lightly used enterprise applications.
        * **Premium SSD:** High-performance, low-latency disk support for I/O-intensive workloads (e.g., databases). Guaranteed IOPS and throughput.
        * **Ultra Disks:** The highest performance SSDs, offering unprecedented and flexible performance for the most demanding workloads (e.g., SAP HANA, SQL Server, NoSQL databases). Allows you to independently configure IOPS and throughput.
    * **Managed vs. Unmanaged (Legacy):** Always use Managed Disks for new deployments. Unmanaged disks required manual management of storage accounts and VHDs.
3.  **Virtual Network (VNet):** The private network in Azure where your VM is deployed. VMs reside in subnets within a VNet.
4.  **Network Interface Card (NIC):** The virtual network adapter that enables the VM to communicate with other resources within the VNet, on-premises networks (via VPN/ExpressRoute), and the internet. Each VM typically has at least one NIC.
5.  **IP Addresses:**
    * **Private IP Address:** Assigned to the NIC from the VNet's subnet range. Used for internal communication. Can be dynamic or static.
    * **Public IP Address (Optional):** Assigned to the NIC (or a Load Balancer/NAT Gateway in front of it) to enable direct inbound/outbound communication with the internet. Can be dynamic or static, Basic or Standard SKU.
6.  **Network Security Groups (NSGs):** Used to filter network traffic to and from the VM's NIC or the subnet it resides in. Essential for securing VM network access.
7.  **Availability Set (Optional but Recommended):** A logical grouping capability for VMs that ensures fault isolation and redundancy across different fault domains (physical server racks) and update domains (groups of VMs that can be rebooted at the same time during maintenance) within a single Azure data center.
8.  **Availability Zones (Optional but Recommended):** Physically separate locations within an Azure region, each with independent power, cooling, and networking. Deploying VMs across Availability Zones protects against datacenter-level failures.
9.  **Storage Account (for Boot Diagnostics):** A storage account is used to store boot diagnostics logs for VMs, which can help troubleshoot boot failures.
10. **Resource Group:** A logical container that holds related Azure resources for a solution. All resources for a VM deployment are typically placed in the same resource group.

---

## **VM Series and Sizes**

Azure offers a vast array of VM series and sizes, optimized for different workloads and performance needs. VM sizes determine the number of vCPUs, memory, disk capacity, and network bandwidth.

* **General Purpose (e.g., Dv3, Dasv5, B-series):**
    * **Description:** Good for dev/test, small to medium databases, and low to medium traffic web servers. Balances CPU-to-memory ratio.
    * **B-series:** Burstable VMs. Low-cost option for workloads that don't need continuous full CPU performance but may burst occasionally.
* **Compute Optimized (e.g., Fsv2-series):**
    * **Description:** High CPU-to-memory ratio. Ideal for medium traffic web servers, network appliances, batch processes, and application servers.
* **Memory Optimized (e.g., Ev3, Easv5, M-series):**
    * **Description:** High memory-to-CPU ratio. Excellent for relational database servers, in-memory analytics, and large caches. M-series offers ultra-high memory for very large databases (e.g., SAP HANA).
* **Storage Optimized (e.g., Lsv2-series):**
    * **Description:** High disk throughput and IOPS. Ideal for NoSQL databases, data warehousing, and large transactional systems that require direct access to local high-speed storage.
* **GPU Optimized (e.g., NC, ND, NV-series):**
    * **Description:** Specialized VMs with Graphics Processing Units (GPUs). Perfect for heavy graphics rendering, video editing, deep learning, and predictive analytics.
* **High Performance Compute (HPC) (e.g., H-series):**
    * **Description:** Fastest and most powerful CPU VMs with optional high-throughput network interfaces (InfiniBand). Designed for computationally intensive workloads like scientific modeling, simulations, and large-scale parallel processing.
* **Confidential Compute (e.g., DC-series, EC-series):**
    * **Description:** Encrypts data *in use* (while being processed by the CPU) using hardware-based trusted execution environments (TEEs). Provides the highest level of data confidentiality and integrity.
    * **Use Cases:** Highly sensitive data processing, blockchain nodes, secure multi-party computation.

---

## **High Availability and Resilience for VMs**

Ensuring your VMs and applications remain available is critical. Azure provides several options to build highly available VM-based solutions:

1.  **Availability Sets:**
    * **Description:** A logical grouping capability that ensures your VMs are spread across different physical hardware (fault domains) and automatically rebooted at different times during maintenance (update domains) within a single Azure data center.
    * **Fault Domains:** A physical rack of servers. Spreading VMs across multiple fault domains protects against power, cooling, or network failures impacting a single rack. (Typically 2 or 3 fault domains).
    * **Update Domains:** Logical groups of VMs that can be rebooted or undergo maintenance simultaneously. Spreading VMs across multiple update domains ensures that only a single update domain is offline at any given time during planned maintenance. (Typically 5 or 20 update domains).
    * **Requirement:** To be effective, you need at least two VMs in an Availability Set. It is highly recommended to have at least three VMs for production workloads across three fault domains.
    * **SLA:** Provides a 99.95% VM uptime SLA if you have two or more VMs within an Availability Set.

2.  **Availability Zones:**
    * **Description:** Physically separate locations *within an Azure region*, each with independent power, cooling, and networking. Deploying VMs across Availability Zones protects against entire datacenter failures.
    * **Zonal Deployment:** You deploy specific VM instances into specific zones (e1: Zone 1, e2: Zone 2, e3: Zone 3).
    * **Zone-Redundant Deployment:** For services like Load Balancers, you can configure them to be zone-redundant, distributing resources across multiple zones automatically.
    * **SLA:** Provides a 99.99% VM uptime SLA if you deploy two or more VMs across two or more Availability Zones in the same region.
    * **Considerations:** Not all regions have Availability Zones. Services must be explicitly configured to use them.

3.  **Azure Site Recovery (ASR) (Basic Introduction):**
    * **Description:** A disaster recovery as a service (DRaaS) solution that enables replication of VMs (and on-premises physical/virtual servers) to an Azure region or to an on-premises secondary site.
    * **Purpose:** Ensures business continuity by orchestrating replication, failover, and failback, minimizing downtime during major outages.
    * **How it relates to VMs:** ASR is commonly used to protect Azure VMs by replicating them to a different Azure region, allowing for quick failover in case of a regional disaster.

4.  **Azure Backup:**
    * **Description:** A native Azure service that provides backup and restore capabilities for Azure VMs (and other Azure services/on-premises data).
    * **Purpose:** Protects your VM data from accidental deletion, corruption, or malware attacks.
    * **Key Features:** Application-consistent backups (for Windows), file-level recovery, long-term retention.

---

## **Cost Optimization Strategies for Azure VMs**

Managing VM costs is crucial. Azure offers several options:

1.  **Reserved Virtual Machine Instances (RIs):**
    * **Description:** Pre-purchase VM capacity for one or three years in a specified region. You commit to a certain VM size (or family) and quantity.
    * **Benefits:** Significant discounts (up to 72% compared to pay-as-you-go).
    * **Ideal for:** Predictable, long-running workloads.
    * **Flexibility:** Can apply to different VMs within the same size family, and can be exchanged or canceled (with fees).

2.  **Azure Spot Virtual Machines:**
    * **Description:** Unused Azure compute capacity available at deeply discounted rates (up to 90% savings). Spot VMs can be evicted by Azure with a 30-second notice if Azure needs the capacity back.
    * **Benefits:** Extreme cost savings for flexible workloads.
    * **Ideal for:** Fault-tolerant applications, batch processing, dev/test environments, rendering jobs, and other workloads that can handle interruptions.
    * **Considerations:** Not suitable for critical, continuous production workloads.

3.  **Azure Hybrid Benefit (AHB):**
    * **Description:** Use your existing on-premises Windows Server and SQL Server licenses with Software Assurance (SA) or eligible subscription licenses to save on VM costs in Azure.
    * **Benefits:** Reduces the cost of Windows Server/SQL Server VMs by covering only the base compute rate, not the software license portion.
    * **Ideal for:** Organizations migrating existing licensed Windows/SQL workloads to Azure.

4.  **Right-Sizing VMs:**
    * **Description:** Continuously review and adjust VM sizes to match actual workload demands. Use Azure Monitor and Advisor recommendations.
    * **Benefits:** Avoids over-provisioning and reduces unnecessary spending.

5.  **Auto-Shutdown:**
    * **Description:** For dev/test VMs, configure automatic shutdown schedules (e.g., shut down at night and on weekends) to save costs when VMs are not in use.

---

## **Benefits of Using Azure VMs**

* **Infrastructure as a Service (IaaS):** Provides maximum control over your operating system and software stack.
* **Flexibility:** Supports a wide range of operating systems (Windows, various Linux distros, custom images).
* **Hybrid Capabilities:** Easily integrate with on-premises networks using VPN or ExpressRoute.
* **Scalability:** Can be scaled up/down vertically, or horizontally using VMSS.
* **Security:** Integrated with Azure security features (NSGs, Azure Security Center/Defender for Cloud).
* **Cost Efficiency:** Multiple options for cost savings (RIs, Spot VMs, Hybrid Benefit).

---

## **Common Use Cases for Azure VMs**

* **Lift-and-Shift Migrations:** Migrating existing on-premises applications and servers to Azure without re-architecting.
* **Custom Applications:** Running applications that require specific OS configurations, legacy software, or custom drivers.
* **Development and Test Environments:** Providing flexible and scalable environments for developers and testers.
* **High-Performance Computing (HPC):** Running complex simulations, scientific modeling, or data processing with specialized VM series.
* **Domain Controllers & Directory Services:** Hosting Active Directory Domain Controllers for hybrid identity.
* **Network Virtual Appliances (NVAs):** Deploying third-party firewalls, load balancers, or other network appliances.
* **Virtual Desktops (VDI):** Hosting virtualized desktop environments (e.g., with Azure Virtual Desktop).

---