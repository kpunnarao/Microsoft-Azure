### **Module 1.7: Identity, Access & Security Fundamentals**

Security is paramount in the cloud, and identity and access management (IAM) forms its cornerstone. This module introduces the fundamental concepts and services Azure provides to control who can do what with your cloud resources.

#### **Microsoft Entra ID (formerly Azure Active Directory)**

Microsoft Entra ID (formerly known as Azure Active Directory or Azure AD) is Microsoft's cloud-based identity and access management service. It provides a comprehensive solution for managing user identities and controlling access to both Azure resources and external applications (like Microsoft 365, Salesforce, and thousands of other SaaS apps).

* **Tenants:**
    * **Description:** A Microsoft Entra ID tenant is a dedicated and isolated instance of Microsoft Entra ID for your organization. It's the first thing you get when you sign up for a Microsoft cloud service (like Azure, Microsoft 365, or Dynamics 365).
    * **Purpose:** It acts as the identity boundary for your organization, where your users, groups, and application registrations reside. Each tenant is unique and logically isolated from other tenants.
    * **Example:** `yourcompany.onmicrosoft.com` (default domain name).

* **Users:**
    * **Description:** Represent individuals or service accounts that need to access resources protected by Microsoft Entra ID.
    * **Types:**
        * **Member users:** Typically internal users within your organization.
        * **Guest users:** External users (from other Microsoft Entra ID tenants, personal Microsoft accounts, or social identity providers) invited to collaborate on specific resources.
    * **Management:** Users are managed within the Microsoft Entra ID tenant.

* **Groups:**
    * **Description:** Collections of users (and sometimes other groups or service principals) that simplify permissions management. Instead of assigning permissions to individual users, you assign permissions to a group, and all members of that group inherit those permissions.
    * **Types:**
        * **Security groups:** Used for granting access to resources (e.g., Azure subscriptions, resource groups, applications).
        * **Microsoft 365 groups:** Used for collaboration (email, calendar, SharePoint) and can also be used for access management.
    * **Benefits:** Easier administration, consistency, and reduced error when managing access at scale.

* **Authentication vs. Authorization:** These are two distinct but related concepts in security:
    * **Authentication:**
        * **Description:** The process of *verifying the identity* of a user, service, or device. It's about proving "who you are."
        * **Mechanism:** Typically involves providing credentials (username/password, multi-factor authentication, biometric scan, certificates).
        * **Outcome:** If successful, the user is authenticated and issued a token (e.g., an access token).
    * **Authorization:**
        * **Description:** The process of *determining what an authenticated entity is allowed to do* within a system or on a resource. It's about defining "what you can do."
        * **Mechanism:** Based on roles, permissions, policies, or access control lists (ACLs).
        * **Outcome:** Determines whether an authenticated user has the necessary rights to perform a requested action (e.g., read a file, delete a VM, modify a database).

#### **Role-Based Access Control (RBAC)**

Role-Based Access Control (RBAC) is the primary authorization system in Azure. It provides fine-grained access management for Azure resources, allowing you to define *what* users can do within *specific scopes*.

* **What it is:**
    * RBAC is built on Microsoft Entra ID. It allows you to manage who has access to Azure resources, what they can do with those resources, and what areas they have access to.
    * An RBAC assignment consists of three elements: **Security Principal** (who/what), **Role Definition** (what actions), and **Scope** (where).
* **Built-in Roles:** Azure provides many pre-defined roles that cover common access scenarios.
    * **Owner:** Has full access to all resources and can delegate access to others.
    * **Contributor:** Can create and manage all types of Azure resources, but cannot grant access to others.
    * **Reader:** Can view all Azure resources, but cannot make any changes.
    * **User Access Administrator:** Can manage user access to Azure resources.
    * Many service-specific roles (e.g., "Virtual Machine Contributor," "SQL DB Contributor").
* **Custom Roles:** If built-in roles don't meet your specific needs, you can create custom roles by defining a specific set of permissions. This adheres to the principle of *least privilege* (granting only the necessary permissions).
* **Scope:** RBAC permissions can be applied at different levels of the Azure hierarchy:
    * **Management Group:** Permissions apply to all subscriptions under the management group.
    * **Subscription:** Permissions apply to all resource groups and resources within that subscription.
    * **Resource Group:** Permissions apply to all resources within that resource group.
    * **Resource:** Permissions apply only to that specific resource.
    * Permissions are inherited down the hierarchy (e.g., if a user is a "Reader" at the subscription level, they are a "Reader" for all resource groups and resources within that subscription).

#### **Azure Key Vault (Secret Management)**

Azure Key Vault is a cloud service that provides secure storage for secrets, cryptographic keys, and SSL/TLS certificates. It helps solve the problem of hardcoding sensitive information directly into application code or configuration files.

* **Purpose:**
    * **Secrets Management:** Securely store and tightly control access to tokens, passwords, API keys, connection strings, etc.
    * **Key Management:** Create and control the encryption keys used to encrypt your data (e.g., for Azure Storage, SQL Database encryption).
    * **Certificate Management:** Provision, manage, and deploy SSL/TLS certificates for your integrated Azure services.
* **Benefits:**
    * **Centralized Secure Storage:** Secrets are stored in a highly available and secure vault.
    * **Access Control:** Access to Key Vault itself is controlled using RBAC, and access to individual secrets/keys/certificates within a vault can be controlled via access policies.
    * **Auditability:** All access to Key Vault is logged.
    * **Managed Identity Integration:** Azure resources (like App Services, VMs, Azure Functions) can use Managed Identities to authenticate to Key Vault, eliminating the need to manage credentials for these services.
    * **Hardware Security Module (HSM) Backed:** For the highest security level, keys can be stored in FIPS 140-2 Level 2 validated HSMs (Premium tier).

#### **Azure Security Center (now Microsoft Defender for Cloud) (Introduction)**

Microsoft Defender for Cloud (formerly Azure Security Center) is a unified infrastructure security management system that strengthens the security posture of your cloud and on-premises workloads. It provides tools to assess vulnerabilities, protect against threats, and respond to attacks.

* **Purpose:**
    * **Security Posture Management:** Provides continuous security assessment, recommendations, and a "Secure Score" to help you improve your security posture across your hybrid cloud environment.
    * **Threat Protection:** Offers advanced threat protection capabilities for various Azure resources (VMs, SQL databases, storage accounts, etc.) and hybrid workloads.
    * **Compliance Management:** Helps you meet regulatory and industry compliance requirements by assessing your environment against various benchmarks (e.g., ISO 27001, PCI DSS).
* **Key Capabilities:**
    * **Secure Score:** A quantifiable measure of your security posture, helping you prioritize security recommendations.
    * **Security Recommendations:** Actionable steps to improve your security configuration (e.g., "Enable MFA," "Encrypt disks").
    * **Vulnerability Assessment:** Identifies security vulnerabilities in VMs, SQL databases, container registries, etc.
    * **Adaptive Network Hardening:** Recommends NSG rules to secure traffic between your VMs.
    * **Just-in-Time VM Access:** Reduces exposure to brute force attacks by only opening network access to VMs when needed.
    * **Advanced Threat Protection:** Detects anomalous activities, SQL injection attempts, suspicious logins, etc.
    * **Centralized Security Monitoring:** Collects security data from various sources and presents it in a unified dashboard.

---