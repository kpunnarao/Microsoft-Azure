### **💿 Module 4.5: Azure Disk Storage - Persistent Storage for VMs**

Azure Disk Storage provides highly durable and available block-level storage for Azure Virtual Machines (VMs). 

These "managed disks" are virtualized disks that abstract away the complexity of underlying storage accounts, making disk management for VMs much simpler and more scalable. 

They are the equivalent of a physical disk on an on-premises server, but with the benefits of cloud scalability, durability, and resilience.

#### **I. Azure Managed Disks: The Evolution of VM Storage**

Azure Managed Disks simplify disk management by handling the storage accounts, ensuring data durability, and providing built-in integration with Azure Availability Sets and Availability Zones.

* **Managed vs. Unmanaged Disks (Brief Recap):**
    * **Unmanaged Disks (Legacy):** Required you to create and manage the storage accounts where your VM's VHD files (disks) resided. This meant manually dealing with storage account IOPS/throughput limits and ensuring disks for VMs in an Availability Set were spread across different storage accounts to avoid single points of failure.
    * **Managed Disks (Recommended):** Azure handles all the complexities of the underlying storage accounts. You just specify the disk type and size, and Azure provisions and manages the disk. This eliminates storage account limits, provides better resiliency, and simplifies scaling. **All new VM deployments should use Managed Disks.**

* **Benefits of Azure Managed Disks:**
    * **Simplified Management:** No need to manage storage accounts for VM disks.
    * **Higher Scale:** Supports up to 50,000 disks of each type per region per subscription, enabling thousands of VMs in a single subscription.
    * **Better Reliability for Availability Sets/Zones:** Disks for VMs in an Availability Set are automatically placed in different storage scale units (stamps) to avoid single points of failure. For Availability Zones, disks are replicated within the chosen zone.
    * **High Durability & Availability:** Designed for 99.999% availability, with three replicas of your data for high tolerance against failures.
    * **Granular Access Control:** Integrates with Azure RBAC for fine-grained permissions.
    * **Integration with Azure Backup:** Seamlessly integrate with Azure Backup service for automated backups and recovery.
    * **Snapshot Capabilities:** Easily create point-in-time snapshots for data protection.

#### **II. Types of Managed Disks and their Performance Characteristics**

Azure offers several types of managed disks, each optimized for different workload performance and cost requirements.

1.  **Standard HDD (Hard Disk Drive):**
    * **Media:** Traditional spinning platters (HDD).
    * **Performance:** Lowest cost, lowest performance. Best for infrequently accessed data, development/test workloads, and non-critical applications that are not sensitive to performance variations.
    * **IOPS/Throughput:** Up to 2,000 IOPS and 500 MB/s (for largest sizes).
    * **Cost:** Transaction-based billing for operations, plus storage cost.

2.  **Standard SSD (Solid State Drive):**
    * **Media:** Solid State Drives (SSD).
    * **Performance:** More consistent performance and reliability than HDDs. Good balance of cost and performance for web servers, lightly used enterprise applications, and dev/test environments.
    * **IOPS/Throughput:** Up to 6,000 IOPS and 750 MB/s (for largest sizes).
    * **Cost:** Transaction-based billing for operations, plus storage cost.

3.  **Premium SSD:**
    * **Media:** High-performance SSDs.
    * **Performance:** Designed for I/O-intensive production workloads, offering high throughput and low latency. Performance scales with disk size (higher IOPS/throughput for larger disks). Can leverage "disk bursting" for short periods of higher performance.
    * **IOPS/Throughput:** Up to 20,000 IOPS and 900 MB/s (for largest sizes, P80).
    * **Cost:** Fixed cost based on provisioned disk size/performance tier, regardless of usage.
    * **Use Cases:** SQL Server, Oracle, MongoDB, Cassandra, high-performance transactional databases, data warehouses.

4.  **Premium SSD v2:**
    * **Media:** Next-generation high-performance SSDs.
    * **Performance:** Offers even higher performance than Premium SSDs with lower latency. Crucially, it allows **independent configuration** of capacity, IOPS, and throughput for greater flexibility and cost efficiency. Performance can be adjusted dynamically without detaching the disk.
    * **IOPS/Throughput:** Up to 80,000 IOPS and 1,200 MB/s (can scale even higher by configuring).
    * **Cost:** Billed for provisioned capacity, IOPS, and throughput.
    * **Use Cases:** Mission-critical production workloads, very large databases, high-performance transactional systems, and other workloads requiring highly granular performance control. Generally replaces the need for Ultra Disks in many scenarios.

5.  **Ultra Disks:**
    * **Media:** The highest-performing SSD option.
    * **Performance:** Offers the highest throughput, IOPS, and consistent sub-millisecond latency. Like Premium SSD v2, allows **independent configuration** of IOPS and throughput, and can be adjusted dynamically.
    * **IOPS/Throughput:** Up to 400,000 IOPS and 10,000 MB/s (for largest configurations).
    * **Cost:** Billed for provisioned capacity, IOPS, and throughput. Most expensive option.
    * **Limitations:** Cannot be used as an OS disk. Does not support Availability Sets (only Single VM or Availability Zone deployment). Limited regional availability. No disk caching.
    * **Use Cases:** Extremely I/O-intensive workloads like SAP HANA, top-tier databases (SQL, Oracle), and other transaction-heavy applications requiring the absolute highest performance.

#### **III. Key Features of Azure Disk Storage**

1.  **Disk Caching (Host Caching):**
    * **Purpose:** Improves disk performance by caching frequently accessed data on the VM host's RAM and SSDs.
    * **Options:**
        * **Read-only:** Recommended for data disks that are primarily read-intensive.
        * **Read/Write:** Recommended for OS disks.
        * **None:** For disks that are write-intensive or require low latency and don't benefit from caching (e.g., database log files, Ultra Disks).
    * **Supported Disks:** Standard HDD, Standard SSD, Premium SSD. Not supported for Ultra Disks or Shared Disks.

2.  **Snapshots:**
    * **Description:** A read-only, crash-consistent, point-in-time full copy of a managed disk.
    * **Purpose:** Used for quick backups, creating new VMs from a specific disk state, or migrating disks.
    * **Cost:** Stored as standard managed disks, so billed at the standard rate.

3.  **Shared Disks:**
    * **Description:** A feature that allows you to attach a single managed disk to multiple Azure VMs simultaneously.
    * **Purpose:** Enables migration or deployment of clustered applications to Azure, such as Windows Server Failover Cluster (WSFC) environments (e.g., SQL Server Failover Cluster Instances, Scale-out File Servers, SAP applications).
    * **Mechanism:** Uses SCSI Persistent Reservations (SCSI PR) to manage concurrent access and write locking, which is an industry standard for clustered environments.
    * **Supported Disks:** Currently, Ultra Disks, Premium SSD v2, Premium SSD, and Standard SSDs can be used as shared disks.
    * **Configuration:** You define a `maxShares` value on the disk, specifying the maximum number of VMs that can simultaneously attach it.
    * **Considerations:** Requires a cluster manager (like WSFC or Pacemaker for Linux) to coordinate access and ensure data consistency. Host caching is not supported for shared disks.

4.  **Disk Encryption:**
    Azure offers multiple options for encrypting your managed disks:
    * **Server-Side Encryption (SSE) with Platform-Managed Keys (PMK):**
        * **Description:** All managed disks are encrypted at rest by default using Microsoft-managed keys. This is transparent to you and provides strong protection.
        * **Standard:** Applies to all managed disks, snapshots, and images.
    * **Server-Side Encryption (SSE) with Customer-Managed Keys (CMK):**
        * **Description:** You provide your own encryption keys, stored in Azure Key Vault. This gives you full control over the encryption keys, including the ability to revoke access.
        * **Control:** You manage the lifecycle of the encryption keys.
        * **Mechanism:** Uses "Disk Encryption Sets" to manage CMKs and apply them to multiple disks.
    * **Azure Disk Encryption (ADE):**
        * **Description:** Provides full disk encryption for OS and data disks by integrating with industry-standard encryption technologies: BitLocker for Windows and dm-crypt for Linux.
        * **Control:** Allows encryption with customer-managed keys (stored in Azure Key Vault).
        * **Purpose:** Encrypts the entire OS volume, data volumes, and can encrypt the temporary disk. Provides "encryption in transit" from the VM to the storage backend (if using CMK).
        * **Considerations:** Not compatible with Ultra Disks. Can be more complex to manage than SSE.

#### **IV. Use Cases for Azure Disk Storage**

* **Operating System Disks:** Every Azure VM has an OS disk (boot disk).
* **Data Disks:** Attach additional disks to VMs for storing application data, databases, logs, etc.
* **Databases:** High-performance Premium SSDs, Premium SSD v2, or Ultra Disks are essential for production-grade SQL Server, Oracle, MySQL, PostgreSQL, and NoSQL databases running on VMs.
* **Enterprise Applications:** Running business-critical applications (e.g., SAP, ERP systems) that require consistent and reliable storage.
* **Clustered Applications:** Using Shared Disks to deploy highly available clustered applications like SQL Server Failover Cluster Instances.
* **Dev/Test Environments:** Cost-effective Standard HDDs or Standard SSDs are often sufficient.
* **Backup & Disaster Recovery:** Snapshots can be used for quick backups, and disks can be replicated to other regions for DR.

---