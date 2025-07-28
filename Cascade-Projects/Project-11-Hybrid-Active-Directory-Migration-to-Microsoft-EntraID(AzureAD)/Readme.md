# Project Document: Hybrid Active Directory Migration to Microsoft Entra ID (Azure AD)

**Version:** 1.0
**Date:** July 28, 2025

---

## 1. Executive Summary

This document outlines the strategy, step-by-step migration plan, and operational considerations for establishing a hybrid identity solution by synchronizing our on-premises Active Directory (AD DS) with Microsoft Entra ID (formerly Azure Active Directory). 

This approach ensures business continuity, leverages cloud security features like Multi-Factor Authentication (MFA), and provides a seamless user experience across both on-premises and cloud resources. 

We will maintain both on-premises AD DS and Microsoft Entra ID, with the former remaining the authoritative source for identity management.

---

## 2. Introduction to Hybrid Identity

A hybrid identity solution integrates your on-premises Active Directory Domain Services (AD DS) with Microsoft Entra ID. 

This means that user accounts, groups, and device information created and managed in your on-premises AD DS are synchronized to Microsoft Entra ID. 

This synchronization allows users to access both on-premises applications and cloud-based services (like Microsoft 365, Azure resources, and SaaS applications) using a single set of credentials.

**Key benefits of a Hybrid AD setup:**
* **Centralized Identity Management:** Manage identities from a single, familiar on-premises AD DS.
* **Seamless User Experience (SSO):** Users can log in once and access both on-premises and cloud applications without re-entering credentials.
* **Enhanced Security:** Leverage Microsoft Entra ID's advanced security features like Multi-Factor Authentication (MFA) and Conditional Access for cloud applications.
* **Scalability and Resilience:** Distribute authentication workloads and provide redundancy.
* **Future-Proofing:** Paves the way for eventual cloud-native identity management if desired.

---

## 3. Theoretical Concepts and Workflow

#### 3.1. How Multi-Factor Authentication (MFA) Works in this Scenario:

In a hybrid setup, MFA is primarily enforced by Microsoft Entra ID for access to cloud-based applications and resources.

1.  **User attempts to access a cloud application (e.g., Microsoft 365, Azure portal).**
2.  **The application redirects the authentication request to Microsoft Entra ID.**
3.  **Microsoft Entra ID verifies the user's identity.**
    * If using **Password Hash Synchronization (PHS)** or **Pass-through Authentication (PTA)**: Microsoft Entra ID handles the initial password verification (either against the synced hash or by passing it through to on-premises AD DS).
    * If using **Federation (AD FS)**: Microsoft Entra ID redirects the authentication request to the on-premises AD FS servers for password validation.
4.  **After successful primary authentication, if MFA is enabled and triggered by a Conditional Access policy, Microsoft Entra ID prompts the user for a second factor.** This could be a notification to the Microsoft Authenticator app, a code from a hardware token, an SMS code, or a phone call.
5.  **Upon successful MFA, Microsoft Entra ID issues a token, allowing the user access to the cloud application.**

**Note:** MFA applied through Microsoft Entra ID typically protects access to cloud resources. For on-premises resources, you would still need on-premises MFA solutions (e.g., NPS extension for MFA, or integrating with AD FS and an MFA provider). However, for hybrid-joined devices accessing on-premises resources with a Primary Refresh Token (PRT) and cloud Kerberos trust, MFA from Entra ID can also satisfy on-premises MFA requirements.

#### 3.2. How Communication Happens Between On-Premises and Azure

The primary communication channel for identity synchronization is established by **Azure AD Connect**.

* **Azure AD Connect Server:** A dedicated server (physical or virtual) in your on-premises network hosts the Azure AD Connect software. This server acts as the bridge.
* **Outbound Communication:** Azure AD Connect initiates outbound connections over standard HTTPS (port 443) to Microsoft Entra ID for synchronization. This means you typically don't need to open inbound firewall ports to your on-premises network.
* **LDAP/Kerberos:** Azure AD Connect communicates with your on-premises domain controllers using standard Active Directory protocols like LDAP and Kerberos to read identity information.
* **Authentication Flow:**
    * **Password Hash Synchronization (PHS):** Azure AD Connect periodically hashes user passwords on-premises and synchronizes these hashes to Microsoft Entra ID. Actual passwords are never sent to the cloud. When a user authenticates to Microsoft Entra ID, the provided password's hash is compared against the synced hash.
    * **Pass-through Authentication (PTA):** Azure AD Connect agents are installed on on-premises servers. When a user tries to authenticate to Microsoft Entra ID, the request is routed to an agent, which then validates the credentials directly against your on-premises AD DS. No password hashes are stored in the cloud.
    * **Federation (AD FS):** For organizations with existing AD FS infrastructure, Microsoft Entra ID redirects authentication requests to the on-premises AD FS farm. AD FS then handles the authentication with your on-premises AD DS.

#### 3.3. How Sync Happens Between Both ADs (On-Premises AD DS and Microsoft Entra ID)

**Azure AD Connect** is the tool responsible for this synchronization.

1.  **Installation and Configuration:** Azure AD Connect is installed on a domain-joined server in your on-premises environment. During configuration, you specify which Organizational Units (OUs) or domains you want to synchronize.
2.  **Initial Synchronization:** After configuration, Azure AD Connect performs an initial full synchronization. It reads all selected user, group, and contact objects from your on-premises AD DS and provisions them in Microsoft Entra ID.
3.  **Delta Synchronization:** After the initial sync, Azure AD Connect uses a scheduler (default every 30 minutes) to perform delta synchronizations. It only synchronizes changes (creations, deletions, modifications) that have occurred in on-premises AD DS since the last sync cycle.
4.  **Synchronization Rules:** Azure AD Connect uses pre-defined and customizable synchronization rules to determine which attributes are synchronized and how they are transformed. For example, `sAMAccountName` and `userPrincipalName` are critical for user login.
5.  **Unidirectional Flow (Default):** By default, synchronization is unidirectional from on-premises AD DS to Microsoft Entra ID. Changes made in Microsoft Entra ID for synced objects (e.g., password reset via Self-Service Password Reset with password writeback enabled) can be written back to on-premises AD DS, but primary identity management remains on-premises.

#### 3.4. When a User Tries to Access, Which AD Receives the Request First?

This depends on **what the user is trying to access** and the **authentication method** chosen for Azure AD Connect.

* **Accessing Cloud Resources (e.g., Microsoft 365, Azure services, SaaS apps integrated with Microsoft Entra ID):**
    * The request goes to **Microsoft Entra ID first**.
    * Based on your Azure AD Connect authentication method:
        * **PHS or PTA:** Microsoft Entra ID attempts to authenticate the user directly or passes the request to the on-premises PTA agent, which then validates against on-premises AD DS.
        * **Federation (AD FS):** Microsoft Entra ID redirects the user's browser to the on-premises AD FS server for authentication.
* **Accessing On-Premises Resources (e.g., file shares, internal applications, domain-joined computers):**
    * The request goes to **on-premises AD DS first**. Users authenticate directly against their on-premises domain controllers using standard Kerberos or NTLM.
    * For hybrid-joined devices, a Primary Refresh Token (PRT) obtained from Microsoft Entra ID can be used for seamless access to both cloud and *some* on-premises resources without re-prompting for credentials, assuming cloud Kerberos trust is configured.

**In summary: Cloud resource access initiates with Microsoft Entra ID. On-premises resource access initiates with on-premises AD DS.**

---

## 4. Step-by-Step Migration Document

This plan assumes you have a healthy on-premises Active Directory environment.

#### Phase 1: Planning and Prerequisites (Critical)

1.  **Define Project Scope and Goals:**
    * Clearly define what users, groups, and devices need to be synchronized.
    * Identify business-critical applications and services that rely on on-premises AD and ensure their compatibility with hybrid identity.
    * Determine the chosen authentication method (PHS, PTA, or AD FS) and justify the choice. (PHS is generally recommended for simplicity and resilience unless specific requirements dictate otherwise).
    * Establish success criteria and rollback procedures.
2.  **Azure Subscription and Tenant:**
    * Ensure you have an active Azure subscription.
    * Verify your Microsoft Entra ID tenant is set up and configured.
    * **Verify Custom Domains:** Add and verify your public domain names (e.g., `yourcompany.com`) in your Microsoft Entra ID tenant. These should match your on-premises User Principal Names (UPNs) for a seamless experience. If your on-premises AD uses a non-routable UPN suffix (e.g., `.local`), you *must* add a routable UPN suffix to your on-premises AD and update user UPNs before synchronization.
3.  **Network Connectivity:**
    * Ensure reliable and secure network connectivity between your on-premises data center and Azure (e.g., Site-to-Site VPN or Azure ExpressRoute). This is crucial for replication, management, and potentially for users accessing resources.
    * Ensure your on-premises firewall allows outbound HTTPS (port 443) traffic from the Azure AD Connect server to Microsoft Entra ID endpoints.
4.  **On-Premises Active Directory Health Check:**
    * Run `DCDiag` and `Repadmin` on all domain controllers to ensure AD DS health (replication, DNS, FSMO roles). Resolve any issues.
    * Clean up any stale or duplicate objects in on-premises AD.
    * Ensure DNS is correctly configured on-premises and can resolve Azure endpoints.
5.  **Azure AD Connect Server Requirements:**
    * **Dedicated Server:** Provision a dedicated Windows Server (2016 or later) for Azure AD Connect. It should *not* be a domain controller.
    * **Hardware:** Sufficient CPU, RAM, and disk space as per Microsoft's recommendations (depends on directory size).
    * **Network:** Join the server to your on-premises domain. It needs network access to domain controllers and outbound internet access.
    * **Service Account:** Identify or create a dedicated service account in your on-premises AD with sufficient permissions to read directory changes (typically a member of "Enterprise Admins" or "Domain Admins" during initial setup, then downgraded to a standard user with replication permissions).
    * **Global Administrator Account:** Have a Microsoft Entra ID Global Administrator account ready for the Azure AD Connect configuration.
6.  **Identity Strategy Review:**
    * **User Principal Names (UPNs):** Standardize UPNs in on-premises AD to match verified domains in Microsoft Entra ID (e.g., `user@yourcompany.com` instead of `user@yourcompany.local`).
    * **Attribute Filtering:** Decide if you need to filter any users or groups from synchronizing to Microsoft Entra ID (e.g., service accounts, disabled accounts).
    * **Password Policy:** Understand how on-premises and Microsoft Entra ID password policies will interact. PHS and PTA allow for a unified password experience.
7.  **Backup and Recovery Plan:**
    * Perform a full backup of your on-premises Active Directory environment before starting.
    * Have a clear rollback plan in case of issues.

#### Phase 2: Azure AD Connect Installation and Configuration

1.  **Download Azure AD Connect:**
    * Log in to the Azure portal (portal.azure.com) with your Global Administrator account.
    * Navigate to **Microsoft Entra ID > Azure AD Connect**.
    * Download the latest version of Azure AD Connect.
2.  **Install Azure AD Connect:**
    * Run the `AzureADConnect.msi` on your dedicated on-premises server.
    * **Express Settings (Recommended for simple topologies):** For a single forest setup with default synchronization options, choose "Express settings." This is the fastest way to get started.
    * **Custom Installation (For advanced scenarios):** If you need to:
        * Synchronize from multiple forests.
        * Filter specific OUs or attributes.
        * Choose a specific authentication method (PTA, AD FS).
        * Specify the service account for synchronization.
        * Install on a SQL Server instead of LocalDB.
        * **For your scenario of keeping both ADs with specific filtering, you will likely opt for Custom Installation.**
3.  **Connect to Microsoft Entra ID:**
    * During the wizard, you will be prompted to sign in with your Microsoft Entra ID Global Administrator credentials.
4.  **Connect to AD DS (On-Premises Active Directory):**
    * Provide the credentials for an Enterprise Administrator or a specially prepared service account for your on-premises AD.
    * Select the on-premises forest(s) you wish to synchronize.
5.  **Configure User Sign-in Method:**
    * **Password Hash Synchronization (PHS):** Simplest, most resilient, and recommended. It synchronizes a hash of the user's password to Microsoft Entra ID.
    * **Pass-through Authentication (PTA):** Provides simple password validation against on-premises AD DS. Requires agents installed on separate servers.
    * **Federation with AD FS:** Requires deploying and configuring AD FS servers on-premises. More complex but offers advanced features.
    * **Select PHS or PTA based on your planning.**
6.  **Configure Synchronization Options (Custom Installation):**
    * **Domain and OU Filtering:** Select the specific domains and OUs you want to synchronize. **This is crucial to prevent synchronizing unwanted objects.** Only select OUs containing users, groups, and devices that need to be in Microsoft Entra ID.
    * **Attribute Filtering:** (Advanced) If specific attributes should not be synchronized, this can be configured.
    * **Optional Features:** Enable features like:
        * **Password writeback:** Allows users to reset their password in Microsoft Entra ID and have it synchronized back to on-premises AD DS. (Requires Microsoft Entra ID P1/P2 license).
        * **Device writeback:** Writes devices from Microsoft Entra ID back to on-premises AD DS (required for Hybrid Azure AD Join).
        * **Group writeback:** Writes Microsoft 365 groups back to on-premises AD DS.
7.  **Start Synchronization Process:**
    * Once configured, the wizard will prompt you to begin the initial synchronization. This can take time depending on the size of your directory.
    * **Enable Staging Mode (Optional but Recommended):** For initial setup, consider enabling staging mode. This allows you to install and configure Azure AD Connect, perform a full synchronization, and review the results without actually exporting changes to Microsoft Entra ID. Once you're confident, you can disable staging mode and promote it to active.

#### Phase 3: Verification and Testing

1.  **Verify Synchronization Status:**
    * **Azure AD Connect Health:** Monitor the synchronization status using Azure AD Connect Health in the Azure portal (Microsoft Entra ID > Azure AD Connect > Azure AD Connect Health). This provides insights into sync errors, performance, and agent health.
    * **Synchronization Service Manager:** On the Azure AD Connect server, open the "Synchronization Service Manager" (from Start Menu). Review the "Operations" tab for successful imports, synchronizations, and exports. Look for any warnings or errors.
2.  **Verify User and Group Synchronization:**
    * In the Azure portal, navigate to **Microsoft Entra ID > Users** and **Microsoft Entra ID > Groups**.
    * Confirm that expected users and groups from your on-premises AD are present in Microsoft Entra ID.
    * Check the "Sync status" attribute for users, which should indicate "Synced from on-premises."
    * Verify key attributes like UPN, display name, and email address are correctly synchronized.
3.  **Test Authentication:**
    * Have a test user account (synced from on-premises) attempt to log in to various Microsoft cloud services (e.g., Office 365 portal, Azure portal).
    * Verify that they can authenticate successfully using their on-premises credentials.
4.  **Test Password Writeback (if enabled):**
    * For a test user, try resetting their password through the Microsoft 365 or Azure AD user portal. Verify that the password change propagates back to on-premises AD DS.
5.  **Test Hybrid Azure AD Join (if applicable):**
    * Ensure devices are successfully Hybrid Azure AD Joined. In the Azure portal, navigate to **Microsoft Entra ID > Devices**. Devices should show a "Join Type" of "Hybrid Azure AD joined."
    * Test user logins on these devices to ensure seamless access to cloud resources.
6.  **Test MFA (if configured):**
    * For test users, enable MFA in Microsoft Entra ID and attempt to log in to a cloud application. Verify that the MFA challenge is presented and authentication succeeds.

#### Phase 4: Post-Migration and Ongoing Management

1.  **Decommission Staging Mode (if used):** If you used staging mode, once testing is complete and you're confident, disable staging mode on the primary Azure AD Connect server to make it active.
2.  **Implement Monitoring and Alerting:**
    * Configure alerts in Azure AD Connect Health for critical synchronization errors or agent health issues.
    * Monitor event logs on the Azure AD Connect server for any related issues.
3.  **Regular Maintenance:**
    * Keep the Azure AD Connect software updated.
    * Regularly review synchronization health reports.
    * Monitor disk space on the Azure AD Connect server.
4.  **Disaster Recovery:**
    * Ensure your Azure AD Connect server is properly backed up.
    * Understand the process for deploying a new Azure AD Connect server in case of failure.
    * Consider deploying a secondary Azure AD Connect server in staging mode for high availability and quick failover.
5.  **User Education:**
    * Inform users about the changes, especially regarding password resets and MFA enrolment.
    * Provide clear instructions and support channels.
6.  **Documentation:**
    * Document all configuration settings, filters, and any custom synchronization rules.
    * Maintain a record of the migration process, including dates and any issues encountered.

### 5. Licensing Considerations

* **Microsoft Entra ID Free:** Provides basic synchronization capabilities.
* **Microsoft Entra ID P1/P2:** Required for advanced features like:
    * Password writeback (for Self-Service Password Reset).
    * Conditional Access policies (critical for advanced MFA scenarios).
    * Microsoft Entra ID Protection (identity risk detection).
    * Group writeback.

Ensure you have the appropriate licenses for the features you intend to use.

### 6. Security Considerations

* **Azure AD Connect Server Security:** Treat the Azure AD Connect server as a Tier 0 asset. Implement strong security measures, including:
    * Dedicated server (no other roles).
    * Principle of Least Privilege for service accounts.
    * Regular patching and security updates.
    * Strong firewall rules.
    * Antivirus/antimalware software.
* **MFA:** Enforce MFA for all administrative accounts in both on-premises AD and Microsoft Entra ID. Extend MFA to all users for cloud access.
* **Conditional Access:** Implement Conditional Access policies in Microsoft Entra ID to control access to cloud applications based on user location, device compliance, sign-in risk, etc.
* **Privileged Identity Management (PIM):** Utilize PIM for just-in-time access to privileged roles in Microsoft Entra ID.
* **Monitoring and Auditing:** Implement robust monitoring and auditing of sign-ins and directory changes in both on-premises AD and Microsoft Entra ID.

### 7. Contingency and Rollback Plan

While a full rollback to a pre-hybrid state is generally complex and rarely needed if proper planning and testing are done, here are key considerations:

* **Pre-Migration Backups:** Ensure complete and verified backups of your on-premises domain controllers.
* **Azure AD Connect Staging Mode:** Utilize staging mode for initial deployment to prevent accidental changes to your Microsoft Entra ID tenant.
* **Disabling Synchronization:** In case of critical issues, synchronization can be temporarily disabled from the Azure AD Connect server or via PowerShell.
* **Object Deletion Threshold:** Configure and understand the object deletion threshold in Azure AD Connect to prevent accidental mass deletions in Microsoft Entra ID.
* **Phased Rollout:** Consider a phased rollout for synchronization, starting with a small group of test users before synchronizing the entire directory.

### 8. Timeline (Example)

| Phase                          | Estimated Duration | Key Activities                                                                                                                                                                                                                                                                                             |
| :----------------------------- | :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Planning & Prerequisites** | 2-4 Weeks          | Project scoping, Azure tenant setup, network connectivity review, on-premises AD health check & cleanup, UPN standardization, Azure AD Connect server provisioning, security review, backup planning.                                                                                                    |
| **2. Installation & Config** | 1 Week             | Download and install Azure AD Connect (staging mode recommended), connect to Azure AD and on-premises AD, configure synchronization scope (OUs), select authentication method, enable optional features (password writeback, device writeback), initial full synchronization.                               |
| **3. Verification & Testing** | 2-3 Weeks          | Monitor sync status via Connect Health & Sync Service Manager, verify user/group presence in Azure AD, test authentication for cloud services with test users, test password writeback, test Hybrid Azure AD Join (if applicable), test MFA.                                                            |
| **4. Post-Migration & Ops** | Ongoing            | Disable staging mode, implement ongoing monitoring & alerting, establish maintenance routines, update disaster recovery plan, user communication & education, continuous security posture review. |

### 9. Conclusion

Implementing a hybrid Active Directory solution by integrating on-premises AD DS with Microsoft Entra ID is a strategic move that enhances security, streamlines identity management, and prepares our organization for future cloud adoption. 

By carefully following the outlined steps and considering the theoretical aspects, we can ensure a smooth and successful migration that delivers significant benefits to our users and IT operations.

---