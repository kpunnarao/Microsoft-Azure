### **🏢 Module 4.7: Azure SQL Managed Instance & SQL Server on Azure VMs - Hybrid & IaaS SQL**

This module delves into two distinct but equally important ways to deploy SQL Server in Azure, bridging the gap between a fully managed PaaS database (Azure SQL Database) and full infrastructure control.

#### **I. Azure SQL Managed Instance: The Near-100% Compatible PaaS Offering**

Azure SQL Managed Instance is a fully managed, intelligent, and scalable cloud database service that combines the broadest SQL Server engine compatibility (near 100%) with all the benefits of a fully managed and evergreen Platform-as-a-Service (PaaS). It's designed for customers looking to lift and shift their on-premises applications to the cloud with minimal application and database changes.

* **What it is:** A dedicated, fully isolated instance of the SQL Server database engine deployed within your Azure Virtual Network (VNet). It offers a rich set of SQL Server features and instance-scoped capabilities that are not available in Azure SQL Database (single database/elastic pool).

* **Key Features & Benefits:**
    * **Near 100% Compatibility with On-Premises SQL Server:**
        * Supports instance-scoped features like SQL Server Agent, Common Language Runtime (CLR), Service Broker, Database Mail, Linked Servers, Change Data Capture (CDC), and native backup/restore (`.bak` files).
        * Allows for cross-database queries and transactions, which are challenging with single Azure SQL Databases.
        * This high compatibility makes it ideal for lift-and-shift migrations of existing SQL Server applications with minimal to no code changes.
    * **PaaS Benefits:** Automatic patching and version updates, automated backups, built-in high availability, and reduced management overhead compared to IaaS.
    * **Native VNet Integration:** Deployed directly into a dedicated subnet within your Azure VNet, providing network isolation and a private IP address. This enables secure, private connectivity and addresses common security concerns for hybrid environments.
    * **Dedicated Resources:** Each Managed Instance is dedicated, ensuring resource isolation and predictable performance.
    * **Security:** Inherits Azure SQL Database's robust security features (TDE, Always Encrypted, Azure AD authentication, Auditing, Threat Detection) and adds VNet-level security.
    * **Scalability:** Uses the vCore purchasing model (General Purpose and Business Critical tiers) allowing independent scaling of compute and storage.
    * **Managed Instance Link:** A new feature that enables bidirectional data replication between SQL Server 2022 on-premises and Azure SQL Managed Instance, allowing for hybrid scenarios like read-scale out to Azure or disaster recovery.

* **High Availability & Disaster Recovery for Managed Instance:**
    * **Built-in HA:** Achieves 99.99% SLA.
        * **General Purpose:** Uses remote storage (Azure Premium Storage) and an Azure Service Fabric-managed stateless compute node. If a node fails, Service Fabric creates a new compute node and attaches the storage.
        * **Business Critical:** Uses a cluster of SQL Server database engine processes, with data replicated to three or more availability replicas. It leverages Always On Availability Groups technology for synchronous replication and automatic failover, providing 99.995% SLA.
    * **Zone Redundancy:** Both General Purpose and Business Critical tiers can be deployed with zone redundancy in regions with Availability Zones, spreading replicas across different zones for resilience against zonal outages.
    * **Automated Backups:** Full, differential, and transaction log backups are automatically taken and stored in geo-redundant storage (GRS). Point-in-time restore is available.
    * **Failover Groups:** Provides similar functionality to Azure SQL Database Failover Groups for cross-region disaster recovery, enabling continuous asynchronous replication to a secondary Managed Instance in another region with automatic failover capabilities.
    * **Geo-Restore:** Restore a Managed Instance from geo-replicated backups to any existing instance in any Azure region in case of a regional outage.

* **When to Choose Azure SQL Managed Instance:**
    * **Lift-and-Shift Migrations:** When migrating existing on-premises SQL Server applications that require instance-scoped features or broader SQL Server surface area compatibility.
    * **Hybrid Scenarios:** When you need a cloud database that can integrate seamlessly with on-premises environments via VPNs or ExpressRoute, especially if you have a VNet-centric architecture.
    * **Consolidation:** Consolidating multiple on-premises databases into a single cloud instance.
    * **Enterprise Applications:** For business-critical applications that need a fully managed PaaS solution but cannot use Azure SQL Database due to feature limitations.

#### **II. SQL Server on Azure Virtual Machines: The IaaS Option**

SQL Server on Azure Virtual Machines (VMs) allows you to run SQL Server in a fully managed virtual machine in Azure. This is an Infrastructure-as-a-Service (IaaS) offering, giving you complete control over the operating system and SQL Server installation.

* **What it is:** You provision a Windows or Linux VM in Azure and manually install SQL Server onto it, just as you would on-premises. Azure provides SQL Server images in the Marketplace to simplify deployment.

* **Key Features & Benefits:**
    * **100% SQL Server Compatibility:** Provides the highest level of compatibility with on-premises SQL Server. All SQL Server features, versions, and editions are supported, including features like SQL Server Reporting Services (SSRS), SQL Server Analysis Services (SSAS), Master Data Services (MDS), PolyBase, and FileStream, which might not be fully available or managed in PaaS options.
    * **Full OS and SQL Server Control:** You have full administrative control over the operating system (Windows or Linux), SQL Server configuration, patching, security hardening, and installation of third-party software.
    * **Customization:** Ideal for highly customized or legacy applications that require specific OS configurations, SQL Server versions, or third-party software installations directly on the database server.
    * **Licensing Flexibility:** Supports Pay-As-You-Go licensing (SQL Server license included in VM cost) and Azure Hybrid Benefit (use your existing SQL Server licenses with Software Assurance to save costs).

* **High Availability & Disaster Recovery for SQL Server on Azure VMs:**
    Unlike PaaS offerings, HA/DR for SQL Server on Azure VMs requires manual configuration, similar to on-premises deployments.

    * **High Availability (within a region):**
        * **Always On Availability Groups (AGs):** The recommended solution for high availability. Multiple SQL Server VMs configured in an AG across Availability Zones or within an Availability Set for fault tolerance. Requires Windows Server Failover Clustering (WSFC).
        * **Failover Cluster Instances (FCIs):** Uses shared Azure Disk Storage (e.g., Premium SSD Shared Disks) for HA within a single Availability Zone.
        * **Log Shipping:** For less critical workloads or as a basic DR solution.
    * **Disaster Recovery (across regions):**
        * **Always On Availability Groups (AGs):** Extend AGs to multiple regions for cross-region DR.
        * **Azure Site Recovery (ASR):** Replicates entire SQL Server VMs to a secondary region.
        * **Backup to URL / Geo-Replicated Storage:** Regularly back up databases to Azure Blob Storage (GRS) and restore to a VM in another region.

* **Licensing for SQL Server on Azure VMs:**
    * **Pay-As-You-Go:** The cost of the SQL Server license is included in the hourly rate of the Azure VM. Simplest option, no upfront license investment.
    * **Azure Hybrid Benefit (AHB):**
        * **Requirement:** Requires active SQL Server licenses with Software Assurance.
        * **Benefit:** Allows you to use your existing SQL Server licenses on Azure VMs, reducing the VM cost to just the base compute rate (you pay for Windows Server or Linux, plus VM hardware). This can lead to significant cost savings.
        * **Activation:** Activated via the Azure portal, PowerShell, or CLI. You confirm that you have eligible licenses.
        * **Dual-use rights:** You get 180 days of dual-use rights to facilitate migration, allowing you to run SQL Server on-premises and in Azure simultaneously during the transition period.
        * **Note:** For SQL Server Enterprise Edition licenses, one core license can often be converted to four vCores in Azure SQL Database (General Purpose or Hyperscale) or Azure SQL Managed Instance. For SQL Server on Azure VMs, it's typically a 1:1 core mapping, but with the benefit of not paying the SQL license portion of the VM cost.

* **When to Choose SQL Server on Azure VMs:**
    * **Maximal Control:** When you need full administrative control over the OS and SQL Server instance for highly specific configurations or legacy application requirements.
    * **Unsupported Features:** When your application relies on SQL Server features not available in PaaS options (e.g., SSRS, SSAS, MDS, PolyBase, specific third-party tools that require direct OS access).
    * **Custom OS:** If you need to run SQL Server on a specific version of Windows Server or Linux, or require custom OS hardening.
    * **Lift-and-Shift (True IaaS):** If you are doing a pure lift-and-shift of an entire server, or if a PaaS option won't meet specific compatibility requirements.
    * **Specialized Licensing:** If you have highly specific or complex licensing agreements that make IaaS more cost-effective (e.g., certain volume licensing deals).

#### **III. Comparison: Azure SQL Database vs. SQL Managed Instance vs. SQL Server on Azure VMs**

Here's a quick summary to help differentiate between the three primary Azure SQL offerings:

| Feature/Service         | Azure SQL Database (PaaS)                               | Azure SQL Managed Instance (PaaS+)                       | SQL Server on Azure VMs (IaaS)                                |
| :---------------------- | :------------------------------------------------------ | :------------------------------------------------------- | :------------------------------------------------------------ |
| **Management Overhead** | Lowest (Fully managed by Microsoft)                     | Low (Managed by Microsoft, some instance-level control)  | Highest (You manage OS, SQL Server, HA/DR, backups)           |
| **Compatibility** | Limited (Database-level, no instance features)          | Near 100% (Instance-level features, VNet integration)    | 100% (Full control over all SQL Server features)              |
| **Networking** | Public Endpoint, Private Link, VNet Service Endpoint    | VNet-injected (dedicated subnet), Private Endpoint       | Full VNet Control, NSGs, Public/Private IPs                   |
| **HA/DR** | Built-in (Automatic failover, geo-replication, failover groups) | Built-in (Automatic failover, geo-replication, failover groups) | Manual setup (Always On AGs, FCIs, Log Shipping, ASR)         |
| **Pricing Model** | vCore (Provisioned/Serverless), DTU                     | vCore (Provisioned)                                      | Pay-As-You-Go or Azure Hybrid Benefit (VM + SQL license)      |
| **Best For** | Cloud-native apps, new dev, multi-tenant SaaS           | Lift-and-shift, hybrid scenarios, enterprise apps needing instance features | Highly customized apps, legacy apps, specific OS/SQL versions, full control |
| **Database Size** | Up to 128 TB (Hyperscale)                               | Up to 16 TB                                              | Limited only by underlying VM disks (up to 64 TB with M-series VMs) |

---