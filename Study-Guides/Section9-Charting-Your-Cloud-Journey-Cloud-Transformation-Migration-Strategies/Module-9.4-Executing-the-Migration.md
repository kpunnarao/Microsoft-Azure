### **⚙️ Module 9.4: Executing the Migration - Tools & Best Practices**

Executing a cloud migration involves more than just lifting and shifting; it's a carefully orchestrated process that demands the right tools, meticulous planning, and adherence to best practices to minimize disruption and ensure success. 

This module focuses on the practical aspects of moving your workloads to Azure.

#### **I. Understanding the Migration Workflow**

Regardless of the specific tool or workload type, a typical migration workflow often follows these high-level steps:

1.  **Preparation (Pre-Migration):**
    * **Source Environment Setup:** Ensure source machines/databases/apps meet prerequisites (e.g., OS compatibility, network access to Azure).
    * **Target Azure Environment Setup:** Provision necessary Azure resources (Virtual Networks, Subnets, Storage Accounts, Target SQL Instances/App Services).
    * **Network Connectivity:** Establish connectivity between on-premises and Azure (VPN Gateway, ExpressRoute, or internet over HTTPS).
    * **Tool Configuration:** Deploy and configure migration tools (e.g., Azure Migrate appliance, ASR replication agents, DMS instance).

2.  **Replication / Data Transfer:**
    * **Initial Replication (Full Sync):** Copy the entire workload/dataset from source to Azure. This is typically the longest phase.
    * **Ongoing Replication (Delta Sync):** After the initial copy, continuously synchronize changes (deltas) from the source to keep the Azure target up-to-date. This minimizes data loss during cutover.

3.  **Test Migration (Dry Run):**
    * Crucial step where you perform a simulated cutover in an isolated Azure environment. This validates the replicated workload, tests network connectivity, application functionality, and identifies issues without impacting production.

4.  **Cutover:**
    * **Planned Downtime/Read-Only:** Schedule a maintenance window. Stop write operations on the source, allow final delta syncs to complete.
    * **Final Sync:** Perform one last delta synchronization.
    * **DNS/Traffic Rerouting:** Change DNS entries, load balancer rules, or application configurations to direct user traffic to the newly migrated workload in Azure.
    * **Post-Cutover Validation:** Immediately verify functionality and performance in the Azure environment.

5.  **Post-Migration Optimization & Decommissioning:**
    * Optimize costs, performance, and security (as covered in Module 9.6).
    * Decommission the original on-premises resources.

#### **II. Key Migration Tools in Azure**

Azure provides a suite of tools tailored for different workload types and migration strategies:

**A. Server Migration (VMs & Physical Servers - Rehost Strategy)**

* **Azure Migrate: Server Migration:**
    * **Functionality:** This is the primary tool within the Azure Migrate hub for replicating and migrating Windows and Linux VMs (VMware, Hyper-V, other clouds) and physical servers. It leverages Azure Site Recovery technology.
    * **How it works:**
        * **Appliance/Agent Deployment:** For VMware/Hyper-V, the Azure Migrate appliance handles agentless replication. For physical servers or VMs on other clouds, a Mobility agent is installed on each server.
        * **Continuous Replication:** Changes on the source servers are continuously replicated to Azure Storage.
        * **Test Failovers:** Allows you to perform non-disruptive test migrations into an isolated Azure virtual network.
        * **Planned Failover/Cutover:** After testing, you execute a planned failover to cut over production traffic to the Azure VM.
    * **Benefits:** Minimal downtime, test migration capability, built-in consistency for multi-VM applications (replication groups).

**B. Database Migration (Replatform / Rearchitect Strategy)**

* **Azure Database Migration Service (DMS):**
    * **Functionality:** A fully managed service that facilitates seamless migrations from various database sources (SQL Server, MySQL, PostgreSQL, Oracle, MongoDB, etc.) to Azure data platforms (Azure SQL Database, Azure SQL Managed Instance, SQL Server on Azure VMs, Azure Cosmos DB, etc.).
    * **Migration Modes:** Supports both **offline** (downtime during migration) and **online** (minimal downtime, continuous sync during migration) migrations depending on the source/target pair.
    * **How it works:**
        * Configured within the Azure portal, DMS connects to your source database and target Azure database.
        * For SQL Server to Azure SQL Managed Instance (online migration), it leverages native backup/restore and log shipping for continuous synchronization.
        * For other scenarios, it uses different mechanisms appropriate for the database type.
    * **Benefits:** Reduces manual effort, handles schema and data transfer, provides a guided migration experience, supports heterogeneous migrations, and aims for minimal downtime with online options.

**C. Web Application Migration (Replatform Strategy)**

* **Azure App Service Migration Assistant:**
    * **Functionality:** A downloadable tool that assesses on-premises ASP.NET web applications running on IIS or Java web apps on Apache Tomcat for migration to Azure App Service.
    * **How it works:** Scans the web app, identifies any incompatibilities or required changes, and helps automate the deployment to an Azure App Service plan.
    * **Benefits:** Streamlines replatforming to PaaS, provides readiness assessment, and automates deployment.
* **Azure Migrate: Web App Assessment:** Integrates with the overall Azure Migrate hub to provide assessment capabilities for web apps before using the Migration Assistant.

**D. Large-Scale Data Migration**

For raw data, files, or large datasets that don't belong to a specific application stack, several options exist:

* **Azure Data Box Family:**
    * **Functionality:** A portfolio of physical devices (Data Box, Data Box Disk, Data Box Heavy) and a software solution (Data Box Gateway) for secure, cost-effective, and fast offline data transfer to Azure storage.
    * **How it works:** You order a device via the Azure portal, Microsoft ships it to you, you copy your data to it, ship it back, and Microsoft uploads the data to your designated Azure storage account.
    * **Benefits:** Ideal for very large datasets (terabytes to petabytes) or when network bandwidth is limited/expensive.
* **AzCopy:**
    * **Functionality:** A command-line utility designed for high-performance copying of data to and from Azure Blob storage, Azure Files, and Azure Table storage.
    * **Benefits:** Optimized for large numbers of small files or very large files, supports resume capabilities, and can synchronize directories. Excellent for online data transfer.
* **Azure Storage Explorer:**
    * **Functionality:** A graphical user interface (GUI) tool for managing Azure storage resources, including uploading and downloading blobs, files, queues, and tables. Built on AzCopy for transfers.
    * **Benefits:** User-friendly for managing data, supports drag-and-drop operations, and provides a visual representation of your storage.
* **Azure Data Factory (ADF):**
    * **Functionality:** A fully managed, serverless data integration service that can create, schedule, and orchestrate ETL/ELT workflows. It has connectors to various data sources and sinks, including on-premises databases and file systems.
    * **Benefits:** Ideal for complex, recurring data migrations, data transformations, and hybrid data integration scenarios.
* **RoboCopy (for Azure Files shares):**
    * **Functionality:** The robust Windows command-line utility (`Robocopy.exe`) can be used to copy files and directories, including their attributes, security information, and timestamps, to mounted Azure file shares.
    * **Benefits:** Familiar to Windows users, can handle large file sets and incremental copies, useful for mirroring shares.

#### **III. Best Practices for Minimizing Downtime During Migration**

Minimizing disruption to business operations is paramount.

1.  **Perform Thorough Test Migrations (Dry Runs):**
    * **Crucial Step:** Run test migrations of your workloads using tools like Azure Migrate: Server Migration's test failover feature.
    * **Validation:** Verify application functionality, network connectivity, and performance in the isolated Azure test environment.
    * **Iterate:** Identify and resolve issues found during testing *before* the actual cutover. This significantly reduces risk.
    * **Practice Cutover:** Use test migrations to refine your cutover runbook and timing.

2.  **Leverage Incremental Replication / Delta Sync:**
    * **Strategy:** For most online migrations (VMs, databases), perform an initial full replication, then continuously sync only the changes (deltas) from the source to the target.
    * **Tools:** Azure Migrate: Server Migration (continuous replication), Azure Database Migration Service (online modes), AzCopy (`--sync` or `--overwrite-if-newer`).
    * **Benefit:** Keeps the target in Azure almost perfectly in sync with the source, drastically reducing the final cutover window and data loss.

3.  **Plan and Schedule Cutovers During Maintenance Windows:**
    * **Timing:** Identify periods of lowest business activity (e.g., weekends, off-peak hours) for the final cutover.
    * **Communication:** Clearly communicate the planned downtime or read-only periods to all stakeholders and end-users.
    * **Detailed Runbook:** Develop a step-by-step cutover runbook outlining all actions, responsibilities, and rollback procedures.

4.  **Implement Application-Specific Downtime Minimization:**
    * **Databases:** Use online migration modes in DMS where available (e.g., SQL to Azure SQL MI) that leverage transaction log shipping for continuous synchronization, limiting downtime to the final cutover switch.
    * **Web Apps:** Use techniques like blue/green deployment or A/B testing with traffic managers to gradually shift users to the new Azure-hosted app.
    * **Load Balancers/Traffic Managers:** Use Azure Load Balancer or Azure Traffic Manager to smoothly reroute traffic to the Azure environment once validation is complete.

5.  **Group Workloads with Dependencies:**
    * Use Azure Migrate's dependency analysis to identify application groups and migrate them together in "waves" to ensure all interconnected components are moved and function correctly.

#### **IV. Crucial Pre-Migration and Post-Migration Considerations**

**A. Pre-Migration Checklist:**

* **Assessment Completion:** Ensure all workloads have been thoroughly assessed for readiness, sizing, and cost.
* **Network Design:** Plan your Azure VNet, subnets, DNS, and connectivity (VPN/ExpressRoute) to replicate or extend your on-premises network. Ensure firewall rules and NSGs are configured.
* **Identity Integration:** Plan Azure AD integration (Azure AD Connect for hybrid identity) for seamless user access post-migration.
* **Security Baseline:** Define and apply initial security policies (Azure Policy, RBAC) for migrated resources.
* **Licensing:** Understand Azure Hybrid Benefit for Windows Server and SQL Server to optimize costs.
* **Backup & DR Plan:** Ensure you have a current backup of your on-premises systems before starting.
* **Test Plan:** Develop a comprehensive test plan for post-migration validation.
* **Communication Plan:** Inform stakeholders about migration timelines and potential impacts.

**B. Post-Migration Validation & Optimization:**

* **Comprehensive Testing:**
    * **Functional Testing:** Verify all application features work as expected.
    * **Performance Testing:** Compare performance metrics against baselines.
    * **User Acceptance Testing (UAT):** Engage business users to validate functionality and user experience.
    * **Security Testing:** Conduct vulnerability scans and penetration tests.
* **Monitoring & Alerting:**
    * Implement Azure Monitor, Log Analytics, and Application Insights to continuously track the health, performance, and security of migrated workloads.
    * Set up alerts for critical events.
* **Cost Management:**
    * Regularly review Azure Cost Management reports.
    * Optimize resource sizing, consider Reserved Instances, and leverage auto-scaling.
* **Security Posture:**
    * Continuously improve security using Azure Security Center/Defender for Cloud recommendations.
    * Refine NSG rules, implement Azure Firewall, and apply security policies.
* **Governance & Compliance:**
    * Ensure migrated resources comply with Azure Policy definitions and organizational standards.
* **Decommissioning:**
    * Safely decommission on-premises infrastructure once the Azure migration is fully validated and production traffic has cut over.
* **Documentation:**
    * Update architectural diagrams, operational runbooks, and disaster recovery plans to reflect the new Azure environment.
* **Modernization Opportunities:**
    * Once stable, identify opportunities for further modernization (e.g., converting VMs to PaaS services, leveraging containers, serverless) to unlock greater cloud benefits.

---