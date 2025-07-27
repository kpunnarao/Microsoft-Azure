### **🗃️ Module 4.6: Azure SQL Database - Managed Relational Database (PaaS)**

Azure SQL Database is a fully managed platform-as-a-service (PaaS) database engine that handles most of the database management functions such as upgrading, patching, backups, and monitoring without user involvement. 

It provides a highly available, scalable, and secure SQL Server database in the cloud, allowing you to focus on application development rather than infrastructure management.

#### **I. Understanding Azure SQL Database**

* **PaaS Offering:** As a PaaS service, Azure SQL Database provides a managed database instance without requiring you to manage the underlying operating system, hardware, or SQL Server software installation. Microsoft takes care of patching, backups, and high availability.
* **Built on SQL Server:** It's built on the latest stable version of the SQL Server database engine, ensuring compatibility with your existing SQL Server tools and applications.
* **Scalability:** Offers flexible scaling options for compute and storage resources, both independently and together.
* **High Availability & Disaster Recovery:** Built-in high availability (HA) and disaster recovery (DR) capabilities are standard features.
* **Security:** Comprehensive security features for data protection, access control, and threat detection.

#### **II. Deployment Models**

Azure SQL Database offers distinct deployment models tailored for different scenarios:

1.  **Single Database:**
    * **Description:** Represents a fully managed, isolated database with its own set of resources. It's similar to a contained database in SQL Server.
    * **Use Cases:** Ideal for modern cloud applications and microservices that need a single, dedicated data source with independent resource allocation and scaling.
    * **Resource Allocation:** Each single database has its own guaranteed amount of compute, memory, and storage resources, which can be dynamically scaled up and down.

2.  **Elastic Pool:**
    * **Description:** A collection of single databases with a shared set of resources (e.g., vCores, memory, storage). Databases can be moved into and out of an elastic pool.
    * **Use Cases:** Designed for multi-tenant applications or software-as-a-service (SaaS) providers with many databases that have varying and unpredictable resource demands. It optimizes cost-efficiency by allowing databases to share a common pool of resources.
    * **Benefit:** Prevents over-provisioning for individual databases, as resources are distributed dynamically based on need, maximizing resource utilization and potentially saving costs.

#### **III. Purchasing Models**

Azure SQL Database offers two primary purchasing models, allowing you to choose how you pay for your database resources:

1.  **vCore-based Purchasing Model (Recommended for new deployments):**
    * **Concept:** Offers more control and transparency over the underlying hardware and resources. You choose the number of virtual cores (vCores), memory, and storage independently.
    * **Transparency:** Provides a clear understanding of the compute resources allocated (e.g., number of vCores, memory, I/O limits).
    * **Flexibility:** Allows you to independently scale compute and storage. You can also choose hardware generations.
    * **Cost Savings:** Supports Azure Hybrid Benefit (if you have existing SQL Server licenses) and Reserved Capacity.
    * **Compute Tiers within vCore:**
        * **Provisioned:** You provision a fixed amount of compute resources (vCores) that are always available, regardless of workload. You pay a fixed hourly rate for this provisioned capacity. Ideal for applications with consistent, predictable demand.
        * **Serverless:** Automatically scales compute resources based on current workload activity. Databases automatically pause during inactive periods (you only pay for storage) and resume when activity returns. You are billed per second for the compute used. Ideal for databases with intermittent, unpredictable usage patterns. Available in General Purpose and Hyperscale tiers.

2.  **DTU-based Purchasing Model (Legacy/Simpler Workloads):**
    * **Concept:** A bundled measure of compute, storage, and I/O resources (CPU, memory, reads, and writes) into a single unit called a Database Transaction Unit (DTU). For elastic pools, it's called an eDTU.
    * **Simplicity:** Provides preconfigured bundles of resources, making it simpler to understand and manage for common workloads.
    * **Limitations:** Less granular control over individual resources. You cannot independently scale CPU, memory, or I/O.
    * **Service Tiers within DTU:** Basic, Standard, and Premium (these map roughly to General Purpose, and Business Critical in vCore).
    * **Use Cases:** Simpler, smaller-scale applications, or if you prefer the simplicity of a preconfigured bundle and fixed monthly payments.

#### **IV. Service Tiers within the vCore Model**

The vCore model offers three main service tiers, each designed for different application requirements regarding performance, availability, and storage:

1.  **General Purpose (GP):**
    * **Architecture:** Separates compute (stateless SQL Server process) and storage (database files on Azure Blob Storage). This design makes it cost-efficient and flexible.
    * **Performance:** Provides a good balance of cost and performance for most business workloads. Uses remote storage, so it has slightly higher latency than local SSDs.
    * **Storage:** Up to 4 TB for single databases.
    * **Availability:** 99.99% SLA. Local redundancy within the data center, with the option for zone redundancy (ZRS for data files) for resilience against zonal failures.
    * **Use Cases:** Most common business applications, dev/test environments, medium-sized web applications, SaaS apps requiring a balance of performance and cost.

2.  **Business Critical (BC):**
    * **Architecture:** Integrates compute and storage on the same node, using locally attached super-fast SSDs for very low latency. It uses Always On Availability Groups technology with multiple synchronous replicas for high resilience.
    * **Performance:** Designed for high-transaction rate applications requiring the highest resilience and lowest I/O latency (1-2 ms).
    * **Storage:** Up to 4 TB for single databases.
    * **Availability:** 99.995% SLA. Achieves high availability and fast recovery through multiple hot standby replicas (3+ replicas by default). Supports zone redundancy (replicas spread across AZs).
    * **Read-Scale Out:** Includes a free-of-charge readable secondary replica for offloading read-only workloads (e.g., reporting, analytics).
    * **Use Cases:** Mission-critical applications, online transaction processing (OLTP) workloads with high concurrency, applications needing fast recovery and low latency, and scenarios where a readable secondary is beneficial for reporting.

3.  **Hyperscale (HS):**
    * **Architecture:** A unique, highly scalable storage architecture that decouples compute and storage. It leverages a multi-tier caching system and "page servers" that store database pages, allowing the database size to grow far beyond the limits of a single node. The transaction log is also highly scalable.
    * **Performance:** Designed for extremely large databases (up to 128 TB) with high throughput and performance. It offers rapid scaling of compute and storage independently.
    * **Storage:** Scales up to 128 TB, irrespective of the compute size.
    * **Availability:** Highly resilient, with data durability for committed transactions regardless of compute replica availability. Supports zone redundancy. Offers multiple types of secondary replicas (high-availability replicas, named replicas for read-scale out, geo-replicas).
    * **Fast Operations:** Fast database backups (snapshot-based, seconds to minutes) and restores (minutes, independent of database size).
    * **Use Cases:** Very large OLTP databases, high-throughput analytics workloads, large data warehouses, and any application that anticipates massive data growth with fluctuating resource needs. Customers starting small and growing rapidly can also benefit.

#### **V. High Availability and Disaster Recovery (HA/DR)**

Azure SQL Database includes robust built-in HA and DR capabilities as part of its PaaS nature:

1.  **High Availability (within a region):**
    * **Built-in Redundancy:** Azure SQL Database automatically maintains multiple copies of your data and logs.
    * **Automatic Failover:** In case of a failure (hardware, software, network), the service automatically fails over to a healthy replica with minimal downtime (typically seconds).
    * **Local Redundancy:** By default, data and compute are replicated within a single data center (LRS for storage).
    * **Zone Redundancy:** For enhanced resilience, you can enable zone redundancy (available in GP, BC, and HS tiers in regions with Availability Zones). This places database replicas in different Azure Availability Zones, protecting against zonal outages.

2.  **Automated Backups:**
    * **Description:** Azure SQL Database automatically performs full backups weekly, differential backups daily, and transaction log backups every 5-10 minutes.
    * **Storage:** Backups are stored in geo-redundant storage (GRS) by default, providing protection against regional outages.
    * **Retention:** Default retention is 7 days, configurable up to 35 days (up to 10 years for long-term retention).
    * **Recovery:** Supports point-in-time restore to any second within the retention period.

3.  **Disaster Recovery (across regions):**
    * **Geo-Restore:** Allows you to recover from a regional outage by restoring from geo-replicated automated backups to any existing server in any Azure region. RPO (Recovery Point Objective) is typically up to 1 hour, RTO (Recovery Time Objective) depends on database size (hours).
    * **Active Geo-Replication:**
        * **Description:** Continuously synchronizes a readable secondary database to a primary database in a different Azure region. Up to four readable secondaries can be configured.
        * **Benefit:** Provides a high RPO (typically less than 5 seconds) and RTO (seconds to minutes) for regional disaster recovery.
        * **Manual Failover:** You manually initiate a failover to the secondary database.
        * **Read-Scale:** Secondary replicas are readable, enabling read-scale out.
    * **Auto-Failover Groups (formerly Failover Groups):**
        * **Description:** Builds on Active Geo-Replication by allowing you to manage replication and failover for a group of databases on a logical server to another logical server in a different region.
        * **Benefit:** Provides read-write and read-only listener endpoints that remain unchanged after failover, simplifying application connectivity. Supports automatic failover with configurable policy.
        * **RPO/RTO:** Similar to Active Geo-replication (RPO typically <5s, RTO within 60s for automatic failover).
        * **Use Cases:** Mission-critical applications requiring minimal downtime during regional disasters and simplified failover management.

#### **VI. Security Features and Capabilities**

Azure SQL Database offers a comprehensive suite of security features:

1.  **Network Security (Firewall & VNet Integration):**
    * **Firewall:** By default, all access is blocked. You configure IP firewall rules (server-level or database-level) or Virtual Network (VNet) rules to allow traffic from specific IPs or subnets.
    * **Private Link:** Recommended for secure and private access, routing traffic over the Azure backbone network by providing a private IP address for the database within your VNet.

2.  **Authentication:**
    * **SQL Authentication:** Traditional SQL Server logins (username and password).
    * **Microsoft Entra ID (formerly Azure Active Directory) Authentication:** Recommended for centralized identity management, single sign-on, and MFA. Supports Azure AD users, groups, and applications.

3.  **Authorization:**
    * **Role-Based Access Control (RBAC):** Manage access to the Azure SQL Database *resource* itself (e.g., who can create/delete databases).
    * **SQL Permissions:** Standard SQL Server permissions for controlling access to *data* within the database (users, roles, object-level permissions).

4.  **Encryption:**
    * **Encryption in Transit (TLS/SSL):** All connections to Azure SQL Database are encrypted by default using Transport Layer Security (TLS).
    * **Encryption at Rest (Transparent Data Encryption - TDE):** Encrypts the entire database, backups, and transaction log files. TDE is enabled by default for all new databases using Microsoft-managed keys. You can also use customer-managed keys (CMK) stored in Azure Key Vault.
    * **Always Encrypted:** Client-side encryption that protects sensitive data inside the database columns from unauthorized access, even by database administrators. The data remains encrypted while in use by the client application.

5.  **Data Protection & Governance:**
    * **Auditing:** Tracks database events and writes them to an audit log in an Azure Storage account, Log Analytics, or Event Hubs for compliance and security analysis.
    * **Data Masking:** Obscures sensitive data in real-time without modifying the actual data in the database. Useful for development/test environments or for non-privileged users.
    * **Row-Level Security (RLS):** Enables control over access to rows in a database table based on the characteristics of the user executing a query (e.g., group membership, execution context).
    * **Azure Defender for SQL:** An advanced security offering that detects unusual activities indicating potential threats to databases (e.g., SQL injection, brute force attacks) and provides security vulnerability assessments.
    * **Ledger (Immutable Ledger):** Provides cryptographic proof of data integrity, protecting data from tampering. New feature, especially useful for audit trails.

---