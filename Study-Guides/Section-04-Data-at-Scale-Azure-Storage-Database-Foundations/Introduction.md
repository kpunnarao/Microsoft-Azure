
### **Section 04: Data at Scale - Azure Storage & Database Foundations**

This section will dive deep into how data is managed and stored in Azure, covering everything from foundational storage accounts to various database solutions, both relational and NoSQL.

* **Module 4.1: Azure Storage Accounts - The Foundation of Cloud Storage**
    * What is an Azure Storage Account? (General Purpose v2, Premium accounts)
    * Types of data services within a Storage Account (Blobs, Files, Queues, Tables)
    * Redundancy options (LRS, ZRS, GRS, RA-GRS, GZRS, RA-GZRS) and their SLAs
    * Storage account types and their use cases (Standard vs. Premium)
    * Access and security (Shared Key, SAS, Azure AD/Entra ID integration, RBAC)
    * Networking for Storage Accounts (Public endpoints, Private Endpoints)

* **Module 4.2: Azure Blob Storage - Object Storage for Unstructured Data**
    * Deep dive into Blob Storage (Containers, Blobs - Block, Page, Append)
    * Access Tiers (Hot, Cool, Cold, Archive) - Features, cost implications, and use cases
    * Lifecycle Management policies
    * Blob versioning and soft delete
    * Immutability policies
    * Static websites with Blob Storage
    * Azure Data Lake Storage Gen2 (built on Blob Storage) as a big data analytics store

* **Module 4.3: Azure Files & Azure NetApp Files - Managed File Shares**
    * Azure Files: SMB/NFS file shares in the cloud
    * Use cases (lift-and-shift, application shares, diagnostic logs)
    * Azure File Sync for hybrid scenarios
    * Snapshots, backups, and security for Azure Files
    * Azure NetApp Files: High-performance, enterprise-grade NFS/SMB file shares
    * Service levels (Standard, Premium, Ultra) and use cases (HPC, SAP, VDI)

* **Module 4.4: Azure Queue Storage & Azure Table Storage - NoSQL Simplicity**
    * Azure Queue Storage: Purpose, features, use cases (decoupling components, asynchronous messaging, task distribution)
    * Azure Table Storage: NoSQL key-value store, schemaless data, use cases (web session state, structured non-relational data)

* **Module 4.5: Azure Disk Storage - Persistent Storage for VMs**
    * Types of Managed Disks (Standard HDD, Standard SSD, Premium SSD, Ultra Disks)
    * Performance characteristics and target workloads for each type
    * Disk sizes and IOPS/throughput
    * Shared Disks for clustered applications
    * Snapshots and Disk Encryption

---