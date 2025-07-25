# 🚀 Project 05: Data Lake & Batch Processing for Business Intelligence

## Scenario: AdventureWorks Cycles' Enterprise Data Platform

AdventureWorks Cycles, a global bicycle manufacturer, has data scattered across various disparate systems: operational databases (sales, inventory), ERP systems, CRM, web analytics logs, and external market data feeds. They currently struggle with inconsistent reporting, slow query performance, and the inability to combine all data sources for a holistic view of their business. They need to build a new enterprise data platform in Azure that can:
* Centralize all data, regardless of its format (structured, semi-structured, unstructured).
* Process large volumes of this data in batches to transform, clean, and aggregate it.
* Support complex analytical queries and ad-hoc reporting for business analysts.
* Provide data for dashboards and self-service BI tools.
* Be scalable, cost-effective, and secure.

### Key Requirements:

**Functional Requirements:**
* Ingest data from various sources (SQL databases, CSV files, JSON logs) into a central repository.
* Perform daily batch transformations, cleaning, and aggregations on the ingested data.
* Enable complex analytical queries and reporting by business analysts.
* Provide data to Power BI for interactive dashboards.
* Support data scientists who may need access to raw and processed data for machine learning.

**Non-Functional Requirements:**
* **Scalability:** The platform must handle petabytes of data and scale compute resources based on processing demands.
* **Cost-Effectiveness:** Optimize storage and compute costs for large data volumes and batch processing.
* **Performance:** Analytical queries and batch processing jobs should complete within defined SLAs (e.g., daily reports available by 8 AM).
* **Security:**
    * Data at rest and in transit must be encrypted.
    * Access to raw, processed, and curated data must be strictly controlled based on user roles (e.g., data scientists vs. business analysts).
    * Integration with existing corporate identity system.
* **Data Governance:** Implement data lineage tracking (conceptual) and ensure data quality.
* **Manageability:** Automate data ingestion and processing workflows.

### Learning Objectives:

Upon completing this project, you should be able to:

* Design a multi-zone (raw, refined, curated) data lake architecture using Azure Data Lake Storage Gen2.
* Select appropriate Azure services for orchestrating and executing large-scale batch data transformations (ETL/ELT).
* Choose a suitable data warehousing solution for analytical reporting.
* Implement data ingestion strategies from various sources.
* Design for data security, access control, and cost optimization within a big data context.
* Understand the roles of different components in a modern data platform.

### Guidance/Hints:

* **Central Storage:** Which Azure storage service is designed for petabyte-scale analytics workloads, supporting a hierarchical namespace? (Hint: ADLS Gen2). How would you structure folders for raw, processed, and curated data?
* **Data Ingestion & Orchestration:** What service can be used to connect to diverse data sources, orchestrate data movement, and trigger batch processing jobs? (Hint: It's an Azure ETL service).
* **Batch Processing Compute:** For transforming and cleaning large datasets, which managed Apache Spark-based analytics platforms in Azure are suitable? Consider **Azure Databricks** or **Azure Synapse Analytics Spark Pools**. Think about their respective strengths.
* **Analytical Query Engine:** For serving aggregated data to business analysts and Power BI, what managed service provides a highly performant, scalable SQL data warehouse experience? (Hint: Synapse Analytics Dedicated SQL Pool, or potentially a Serverless SQL pool for ad-hoc queries).
* **Security & Access Control:** How will you manage access to different layers of the data lake and the data warehouse? Consider Azure RBAC and POSIX ACLs for ADLS Gen2.
* **Data Quality/Transformation:** How would you conceptually ensure data quality during the ETL/ELT process?

### Deliverables:

1.  **High-Level Architecture Diagram:** A visual representation of your proposed enterprise data platform, showing the flow from various data sources through ingestion, storage layers, processing, and consumption. Clearly label the chosen Azure services.
2.  **Service Justification Document (Brief):** A short write-up (e.g., 1-2 pages) explaining:
    * Your choice of Azure services for each stage (data lake, ingestion, processing, warehousing/reporting) and their configurations (e.g., SKU, scale units).
    * How your design addresses each of the key requirements (Scalability, Cost-Effectiveness, Performance, Security, Data Governance, Manageability).
    * Your strategy for structuring the data lake.
    * Any significant trade-offs you considered (e.g., between Databricks and Synapse Spark, or different warehousing options).
3.  **(Optional) Data Flow Sketch:** A simple sketch showing a conceptual flow for a specific dataset (e.g., sales data) through the raw, refined, and curated layers.

### Evaluation Criteria:

Your solution will be evaluated based on:

* **Data Lake Design:** Is the ADLS Gen2 structure logical and scalable?
* **Pipeline Efficiency:** Does the proposed batch processing pipeline effectively handle large volumes and meet performance requirements?
* **Service Selection:** Are the chosen Azure services appropriate for an enterprise BI platform, especially regarding big data technologies?
* **Security & Access:** Is access control robust and granular across data layers?
* **Cost-Effectiveness:** Does the design consider optimization for large-scale data storage and compute?
* **Clarity of Diagram & Explanation:** Is your architecture easy to understand and well-documented?

### Advanced Challenges/Extensions (Optional):

For those who want to push further:

* Integrate **streaming data ingestion** alongside batch processing (e.g., using Event Hubs/Stream Analytics) for real-time analytics on a portion of the data.
* Propose a solution for **data cataloging and metadata management** (e.g., Azure Purview) within this architecture.
* Design for **data versioning** or schema evolution within the data lake.
* Consider using **Managed Virtual Network** and **Private Endpoints** within Azure Synapse Analytics for enhanced security and isolation.
* Outline how **Azure Machine Learning** could consume data from this platform for model training and deployment.

---