# 🚀 Project 01: Multi-Tier Web Application Deployment with Basic DR

## Scenario: Contoso Corp's New Customer Portal

Contoso Corp, a growing e-commerce company, is launching a new customer self-service portal. This portal will allow customers to view their order history, update their profiles, and track shipments. The company needs a highly available solution with a focus on ease of management for the application tier and reliable data storage. While global distribution isn't an immediate requirement, they need a basic disaster recovery (DR) strategy to ensure business continuity in case of a regional outage.

The application is a standard 3-tier architecture:
* **Web Tier:** A public-facing web application.
* **Business Logic Tier:** APIs that handle business rules and interact with the database. (For simplicity, this might be integrated into the web application or a separate App Service).
* **Data Tier:** A relational database storing customer and order information.

### Key Requirements:

**Functional Requirements:**
* Customers can access the portal via a public URL.
* Customers can view their order history and update their profile.
* The application needs to communicate securely with the database.

**Non-Functional Requirements:**
* **High Availability (HA):** The application should remain available even if a single component fails within a region (e.g., VM instance, database replica).
* **Disaster Recovery (DR):** The application and its data must be recoverable in a different Azure region in the event of a primary region-wide outage, with an RTO (Recovery Time Objective) of less than 4 hours and an RPO (Recovery Point Objective) of less than 1 hour.
* **Scalability:** The web and business logic tiers must be able to scale out automatically to handle varying loads.
* **Security:**
    * Network traffic should be segmented between tiers.
    * Database access should be restricted to the application tier.
    * Public access should only be allowed to the web tier.
* **Manageability:** Minimize the management overhead for the application infrastructure.
* **Cost-Effectiveness:** Utilize PaaS services where appropriate to reduce operational costs.

### Learning Objectives:

Upon completing this project, you should be able to:

* Design a secure virtual network (VNet) topology with appropriate subnets and network security groups (NSGs) for a multi-tier application.
* Select appropriate Azure compute services (e.g., Azure App Service, Virtual Machine Scale Sets) for the web/business logic tier based on requirements.
* Choose a suitable Azure managed relational database service (e.g., Azure SQL Database) and configure it for high availability.
* Implement basic disaster recovery strategies for both the application and database tiers across Azure regions.
* Apply the principles of the Azure Well-Architected Framework (Reliability, Security, Operational Excellence, Cost Optimization) in a practical design.
* Understand and justify the trade-offs between different Azure service choices.

### Guidance/Hints:

* **Networking:** Consider a hub-spoke model or a simple flat VNet with multiple subnets. Each tier should ideally reside in its own subnet. Use NSGs to control traffic flow between subnets and to/from the internet.
* **Compute:** Azure App Service is a strong candidate for the web/business logic tier due to its PaaS benefits (reduced management, built-in scaling, deployment slots). How would you ensure its high availability?
* **Database:** Azure SQL Database (PaaS) offers various service tiers and deployment options. Look into its built-in HA and DR features like Active Geo-Replication or Failover Groups.
* **Public Access:** How will users access your web application? Consider services that provide global routing and potentially Web Application Firewall (WAF) capabilities.
* **DR Strategy:** Think about how you would achieve an RTO of < 4 hours and RPO of < 1 hour for both the application and the database. This might involve active-passive or active-active (where appropriate) setups.
* **Security Best Practices:** Remember the principle of least privilege for network access. How would the application connect securely to the database?

### Deliverables:

1.  **High-Level Architecture Diagram:** A visual representation of your proposed solution, showing the main Azure services and their connections. Use tools like draw.io, Visio, or any diagramming software you prefer.
2.  **Service Justification Document (Brief):** A short write-up (e.g., 1-2 pages) explaining:
    * Your choice of Azure services for each tier and their configuration.
    * How your design addresses each of the key requirements (HA, DR, Scalability, Security, Manageability, Cost-Effectiveness).
    * Any significant trade-offs you considered and why you made your final decisions.
3.  **(Optional) Basic Implementation Plan:** Outline the key steps you would take to deploy this architecture using either Azure Portal, ARM templates, Bicep, or Terraform (no actual code required at this stage, just the plan).

### Evaluation Criteria:

Your solution will be evaluated based on:

* **Completeness:** Does it address all key requirements?
* **Architectural Soundness:** Is the design robust, scalable, and secure?
* **Justification:** Are your service choices well-reasoned and do they align with the requirements and WAF principles?
* **DR Effectiveness:** Does the proposed DR strategy meet the RTO/RPO requirements?
* **Clarity of Diagram & Explanation:** Is your architecture easy to understand and well-documented?

### Advanced Challenges/Extensions (Optional):

For those who want to push further:

* Integrate **Azure Front Door** for global routing and WAF capabilities.
* Implement **private endpoint** connectivity between App Service and Azure SQL Database for enhanced security.
* Add **Azure Monitor** and **Application Insights** to the architecture for comprehensive monitoring.
* Consider **cost estimation** for your proposed solution using the Azure Pricing Calculator.
* Explore **Infrastructure as Code (IaC)** concepts for deploying this architecture (e.g., sketch out a Bicep structure).

---