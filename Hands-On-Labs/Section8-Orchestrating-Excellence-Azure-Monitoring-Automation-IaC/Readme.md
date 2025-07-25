# 📊 Section 8: Orchestrating Excellence - Hands-On Labs

## Introduction to Operational Excellence in Practice

Welcome to the **Hands-On Labs** for Section 8: Orchestrating Excellence! This module is dedicated to providing you with practical experience in building and maintaining highly observable, automated, and consistently deployed Azure environments. As an Azure Solutions Architect, ensuring the operational health, efficiency, and reliability of your cloud solutions is paramount. This means actively monitoring performance, automating routine tasks, and codifying your infrastructure for repeatable, error-free deployments.

These labs are meticulously designed to give you direct, step-by-step experience with services like Azure Monitor for comprehensive observability, Azure Automation for task orchestration, and Infrastructure as Code (IaC) using Bicep. You will learn to collect logs and metrics, set up alerts, automate VM management, and deploy complex resource hierarchies using code. By actively engaging with these exercises, you will solidify your understanding of operational excellence best practices and build the practical skills necessary for designing and managing robust cloud operations.

### What You Will Achieve in These Labs:

Through the hands-on labs in this section, you will gain practical experience in:

1.  **Azure Monitor - Metrics & Logs:**
    * Navigating the **Azure Monitor dashboard** and exploring various metric charts for different resources (VMs, App Services, Storage Accounts).
    * Setting up **Diagnostic Settings** for an Azure resource to send logs and metrics to a Log Analytics Workspace.
    * Exploring and writing basic **Kusto Query Language (KQL)** queries in **Log Analytics Workspace** to analyze resource logs (e.g., VM performance, web server logs).
    * Using KQL to filter, project, and summarize log data for actionable insights.
2.  **Azure Monitor - Alerting & Action Groups:**
    * Creating a **metric alert rule** based on a common metric (e.g., CPU utilization of a VM exceeding a threshold).
    * Creating a **log search alert rule** based on a custom KQL query.
    * Configuring an **Action Group** to send email notifications when an alert fires.
    * (Optional) Integrating an Action Group with a Webhook or Logic App to trigger an automated response.
3.  **Azure Automation - Runbooks:**
    * Creating an **Azure Automation Account**.
    * Importing or creating a simple **PowerShell Runbook** (e.g., a runbook to start or stop an Azure Virtual Machine).
    * Configuring **Managed Identity** for the Automation Account and assigning it appropriate RBAC permissions to manage resources.
    * Scheduling the runbook to execute at specific times.
    * Reviewing runbook jobs and their output.
4.  **Infrastructure as Code (IaC) with Bicep:**
    * Installing the **Bicep CLI** and setting up the VS Code extension.
    * Writing a basic **Bicep template** to deploy a simple resource (e.g., a Storage Account or a Virtual Network).
    * Deploying the Bicep template using the **Azure CLI**.
    * Understanding **parameters and variables** in Bicep templates.
    * (Optional) Refactoring an existing ARM JSON template into a Bicep file (`az bicep decompile`).
    * (Optional) Deploying a slightly more complex multi-resource Bicep template (e.g., VNet, Subnet, VM with NIC).

### How These Labs Build Your Expertise:

These hands-on monitoring, automation, and IaC labs are fundamental for your development as an Azure Solutions Architect. They will enable you to:

* **Ensure Observability:** Gain practical skills in setting up comprehensive monitoring, collecting logs, and creating effective alerts to maintain application and infrastructure health.
* **Drive Automation:** Automate repetitive operational tasks, reducing manual effort, minimizing human error, and improving operational efficiency.
* **Implement Consistent Deployments:** Master Infrastructure as Code with Bicep to ensure your Azure environments are consistently and reliably deployed, reducing configuration drift.
* **Proactive Problem Solving:** Learn to interpret monitoring data and leverage alerts to identify and address issues before they impact users.
* **Optimize Operations:** Understand how automation and IaC contribute to operational excellence, allowing you to build more agile and efficient cloud solutions.
* **Translate Theory to Practice:** Directly apply concepts learned about metrics, logs, runbooks, and declarative infrastructure in a live Azure environment.

Approach these labs with a focus on efficiency and consistency. Mastering these tools is crucial for designing and managing cloud solutions that are not only powerful but also robust, observable, and easy to operate.