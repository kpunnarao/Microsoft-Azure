### **👁️ Module 7.6: Microsoft Sentinel (formerly Azure Sentinel) - Cloud-Native SIEM for Intelligent Security Analytics**

As organizations move more of their operations to the cloud and face an increasingly complex threat landscape, traditional Security Information and Event Management (SIEM) solutions often struggle to keep up with the scale, speed, and diversity of cloud data. 

**Microsoft Sentinel** (formerly Azure Sentinel) is Microsoft's answer to this challenge: a scalable, cloud-native SIEM with built-in Security Orchestration, Automation, and Response (SOAR) capabilities.

Microsoft Sentinel provides intelligent security analytics across your entire enterprise, offering a "bird's-eye view" of your security posture across users, devices, applications, and infrastructure, whether in Azure, on-premises, or in other clouds.

#### **I. What is Microsoft Sentinel?**

Microsoft Sentinel is a **cloud-native SIEM** (Security Information and Event Management) and **SOAR** (Security Orchestration, Automation, and Response) solution that:

* **Collects Data at Cloud Scale:** Gathers security data from virtually any source across your entire enterprise, regardless of where the data resides (on-premises, Azure, AWS, GCP, Microsoft 365, etc.).
* **Detects Threats:** Uses analytics rules, machine learning, and Microsoft's global threat intelligence to identify suspicious activities and sophisticated threats, minimizing false positives.
* **Investigates Threats:** Provides AI-powered investigation tools to explore and understand the scope of threats, correlate alerts into incidents, and visualize attack paths.
* **Responds to Incidents:** Enables rapid, automated responses to security incidents using playbooks, reducing the time to mitigate threats.
* **Proactive Threat Hunting:** Allows security analysts to proactively search for new and emerging threats in their environment using powerful query language (KQL).

#### **II. Key Capabilities and Features**

Microsoft Sentinel delivers an end-to-end security operations experience through its core capabilities:

**A. Data Collection**

1.  **Extensive Data Connectors:**
    * **Microsoft Sources:** Seamless, real-time integration with Microsoft services like Microsoft Entra ID (Azure AD), Office 365 (including SharePoint, Exchange, Teams), Microsoft Defender XDR products (Defender for Endpoint, Identity, Cloud Apps, Cloud), Azure Activity Logs, Azure Firewalls, Azure Storage, Azure SQL, and more.
    * **Non-Microsoft Sources:** Built-in connectors for a vast ecosystem of third-party security solutions (e.g., Palo Alto Networks, Fortinet, Check Point, Cisco ASA, Symantec, Zscaler, CrowdStrike, Qualys), cloud providers (AWS CloudTrail, S3, VPC Flow Logs; GCP Audit Logs), and generic formats (Syslog, Common Event Format (CEF), REST APIs).
    * **Custom Connectors:** Ability to build custom connectors using Azure Functions or Logic Apps for sources not natively supported.
2.  **Scalable Data Ingestion:**
    * Leverages Azure Monitor Log Analytics workspace as its data backend, providing massive scalability and cost-efficiency for ingesting and storing security logs and events.
3.  **Data Normalization and Enrichment:**
    * Sentinel can normalize raw logs into a consistent schema (using the Advanced Security Information Model - ASIM), making it easier to query and correlate data from diverse sources. It also enriches data with contextual information.

**B. Threat Detection and Analytics**

1.  **Analytics Rules:**
    * **Scheduled Queries:** The most common type, running Kusto Query Language (KQL) queries at defined intervals to detect specific patterns or thresholds that indicate threats.
    * **Near Real-time (NRT) Rules:** A subset of scheduled rules optimized to run every minute for highly time-sensitive detections.
    * **Microsoft Security Rules:** Built-in rules that leverage Microsoft's threat intelligence and security research, automatically detecting threats from connected Microsoft security solutions.
    * **Anomaly Rules / Machine Learning (ML) Rules:** Leverage built-in machine learning algorithms and behavioral analytics (User and Entity Behavior Analytics - UEBA) to detect unusual patterns, insider threats, compromised identities, and previously unknown (zero-day) threats without predefined signatures.
    * **Fusion (Multi-Stage Attack Detection):** A unique capability that uses Microsoft's proprietary scalable machine learning algorithm to automatically correlate alerts from various products and data sources into high-fidelity, actionable incidents. This helps identify complex, multi-stage attacks that might otherwise go unnoticed.
    * **Threat Intelligence Integration:** Ingests Indicators of Compromise (IoCs) from various threat intelligence platforms (e.g., TAXII, MISP, STIX) to enrich data and proactively detect malicious activities.
2.  **Incidents:**
    * Sentinel automatically correlates related alerts into unified **Incidents**, providing a centralized view for investigation. This significantly reduces alert fatigue by grouping noisy or fragmented alerts.

**C. Threat Investigation**

1.  **Investigation Graph:**
    * Provides an interactive visual graph that helps security analysts understand the scope and root cause of an incident by visualizing connections between suspicious entities (users, devices, IPs, files) and their activities.
2.  **Bookmarks:**
    * Allows analysts to save interesting queries and findings during threat hunting for later reference or to create new incidents.
3.  **Entity Behavior Analytics (UEBA):**
    * Built-in behavioral analytics profiles the normal behavior of users and entities, detecting deviations that could indicate a compromise or insider threat.
4.  **Workbooks:**
    * Integration with Azure Monitor Workbooks provides powerful, customizable dashboards and interactive reports for visualizing security data, compliance posture, and threat trends.

**D. Security Orchestration, Automation, and Response (SOAR)**

1.  **Playbooks (Azure Logic Apps):**
    * Microsoft Sentinel leverages Azure Logic Apps to create automated workflows (playbooks) that can respond to security incidents.
    * **Triggers:** Playbooks can be triggered automatically by Sentinel incidents or alerts, or manually by security analysts.
    * **Actions:** They can perform a wide range of actions, such as:
        * Notifying security teams (email, Teams message, ticketing system).
        * Enriching incident data (e.g., looking up IP reputation, querying threat intelligence).
        * Blocking malicious IPs in a firewall (e.g., Azure Firewall).
        * Isolating compromised endpoints (via integration with Microsoft Defender for Endpoint).
        * Disabling user accounts in Microsoft Entra ID.
        * Creating tickets in ITSM systems (e.g., ServiceNow).
        * Collecting forensic data.
    * **Benefits:** Automates repetitive tasks, accelerates incident response, reduces human error, and allows security analysts to focus on complex investigations.
2.  **Automation Rules:**
    * Allow you to define conditions and actions for incidents, including running playbooks, assigning owners, changing status, or classifying incidents. They provide greater flexibility in automating incident management.

**E. Proactive Threat Hunting**

1.  **Kusto Query Language (KQL):**
    * Empowers security analysts to write complex queries to proactively search for new and evolving threats in their ingested data.
2.  **Built-in Hunting Queries:**
    * Microsoft provides a rich set of pre-built hunting queries developed by security experts, categorized by MITRE ATT&CK tactics, to jumpstart investigations.
3.  **Notebooks (Jupyter):**
    * Integrates with Jupyter Notebooks (via Azure Machine Learning workspace) for advanced threat hunting, data analysis, visualization, and machine learning model development on top of Sentinel data.

#### **III. Microsoft Sentinel vs. Microsoft Defender for Cloud**

While both are crucial Microsoft security services, they serve distinct but complementary purposes:

| Feature/Service         | Microsoft Defender for Cloud (MDC)                                   | Microsoft Sentinel                                                                           |
| :---------------------- | :------------------------------------------------------------------- | :------------------------------------------------------------------------------------------- |
| **Primary Role** | **Cloud-Native Application Protection Platform (CNAPP):** CSPM (Posture Management) & CWP (Workload Protection). | **Cloud-Native SIEM & SOAR:** Security Information and Event Management, Orchestration, Automation, and Response. |
| **Focus** | **Posture:** Identifying misconfigurations, vulnerabilities, and hardening recommendations *for cloud workloads*. **Protection:** Detecting threats *within* cloud workloads (VMs, storage, SQL, containers, etc.). | **Visibility & Analytics:** Centralizing security data from *all sources* (cloud, on-prem, multi-cloud) for threat detection, investigation, and automated response. |
| **Data Source Focus** | Primarily focuses on data related to **Azure resources** (and connected AWS/GCP workloads) and their security configurations/threats. | Ingests data from **anywhere** – all Microsoft security products, third-party solutions, network devices, identity providers, on-premises servers, other clouds. |
| **Output** | Security **recommendations**, **security alerts** specific to cloud workloads, secure score. | Aggregates and correlates **alerts from all sources** into **incidents**, provides comprehensive **investigation tools**, enables **automated responses**. |
| **Relation** | **MDC is a primary data source for Sentinel.** MDC alerts and recommendations are ingested into Sentinel for broader context, correlation, and automated response. | **Sentinel provides the broader security operations center (SOC) capabilities.** It acts as the central brain that consumes alerts from MDC (and other Defender products) to provide a holistic view. |
| **Proactive Security** | Helps *prevent* attacks by hardening configurations and identifying vulnerabilities. | Helps *detect* active attacks, *investigate* them, and *respond* to them. Also supports proactive *threat hunting*. |

In simple terms: Microsoft Defender for Cloud tells you *what's wrong with your cloud resources* and *detects threats within them*. Microsoft Sentinel takes that information (and data from everywhere else) to provide *end-to-end threat detection, investigation, and automated response* for your entire organization. They work best when used together.

#### **IV. Pricing Model**

Microsoft Sentinel's pricing is primarily based on the volume of data ingested into the underlying Azure Monitor Log Analytics workspace.

* **Data Ingestion:** Charged per Gigabyte (GB) of data ingested into the Log Analytics workspace.
    * **Analytics Logs:** High-value security data optimized for real-time analytics and investigations.
    * **Basic Logs:** Lower-cost option for high-volume, low-fidelity logs (e.g., network and firewall logs) where full indexing is not always required for real-time analytics. Querying basic logs might incur additional search job costs.
    * **Auxiliary Logs (Preview):** Even lower-cost for specific high-volume, low-fidelity logs.
* **Data Retention:**
    * The first 90 days of data retention in the Log Analytics workspace are typically free for Analytics Logs.
    * Beyond 90 days, standard Log Analytics data retention costs apply per GB per month.
    * Basic Logs have a shorter free retention period (e.g., 8 days), after which retention charges apply.
    * Long-term archival to Azure Blob Storage is a more cost-effective option for compliance.
* **Commitment Tiers:**
    * Microsoft Sentinel offers "Commitment Tiers" (formerly Capacity Reservations) that provide discounted pricing for predictable daily data ingestion volumes (e.g., 100 GB/day, 200 GB/day, etc.). This offers significant savings compared to the Pay-As-You-Go model for consistent high data volumes.
* **Free Trial:** New Log Analytics workspaces often qualify for a free trial period with a certain amount of free data ingestion per day.
* **Free Data Sources:** Some specific data sources, like Azure Activity Logs and Office 365 Audit Logs, can be ingested into Sentinel at no additional cost.

By understanding your data volumes and retention needs, you can choose the most cost-effective pricing model for your organization.

---