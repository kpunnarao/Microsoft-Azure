### **Section 03: Powering Your Solutions - Azure Compute & Application Platforms**

This section will delve into the various ways you can host and run your applications in Azure, from traditional virtual machines to modern serverless and containerized solutions.

Here's the module list:

* **🌐 Module 3.1: Azure Virtual Machines (VMs) Deep Dive - IaaS Compute**
    * Understanding VM fundamentals, types (series & sizes), images, and storage options.
    * Deployment, networking (NICs, public/private IPs), and management.
    * High Availability options: Availability Sets, Availability Zones (recap/deep dive on how they relate to VMs).
    * Disaster Recovery (Azure Site Recovery basics) & Backup.
    * Cost optimization (Reserved Instances, Spot VMs).

* **⚖️ Module 3.2: Azure Virtual Machine Scale Sets (VMSS) Deep Dive**
    * Purpose and benefits of VMSS for highly available, scalable applications.
    * Automatic scaling (autoscale rules, metrics, instance limits).
    * Orchestration modes: Uniform vs. Flexible.
    * Deployment and management strategies.
    * Integration with Load Balancers and Application Gateways.

* **⚙️ Module 3.3: Azure App Service Deep Dive - PaaS Web Applications**
    * Overview of App Service for web apps, API apps, mobile backends, and Logic Apps.
    * App Service Plans (SKUs, pricing tiers, scaling).
    * Deployment slots for CI/CD, custom domains, and SSL bindings.
    * Networking integration (VNet Integration, Private Endpoints, App Service Environments).
    * Monitoring, diagnostics, and troubleshooting.

* **Functions Module 3.4: Azure Functions & Logic Apps Deep Dive - Serverless Compute & Workflow**
    * **Azure Functions:** Event-driven serverless compute (triggers, bindings, consumption vs. premium plans, Durable Functions).
    * **Azure Logic Apps:** Serverless workflow orchestration (triggers, actions, connectors, enterprise integration).
    * Use cases and benefits of serverless architectures.
    * Monitoring and troubleshooting serverless applications.

* **📦 Module 3.5: Azure Container Instances (ACI) Deep Dive**
    * Purpose: Running single containers quickly and easily without managing infrastructure.
    * Use cases (batch jobs, dev/test, burst workloads).
    * Deployment, networking, and storage options.
    * Comparison with other container services (AKS, Container Apps).

* **☸️ Module 3.6: Azure Kubernetes Service (AKS) Deep Dive - Managed Kubernetes**
    * Introduction to Kubernetes concepts (pods, deployments, services, namespaces, nodes).
    * AKS architecture (managed control plane, worker nodes).
    * Scaling, networking (Kubenet vs. Azure CNI), and storage.
    * Security (Azure AD integration, RBAC, Azure Policy for AKS).
    * Monitoring (Container Insights), logging, and CI/CD integration.
    * Advanced features: Virtual Nodes (ACI integration), Node Pools, AKS Day 2 operations.

* **🚀 Module 3.7: Azure Container Apps Deep Dive - Serverless Containers & Microservices**
    * Purpose: Running microservices and containerized applications on a serverless platform.
    * Core concepts: Revisions, traffic splitting, Dapr integration, KEDA for event-driven scaling.
    * Use cases (microservices, event-driven processing, long-running processes).
    * Networking (ingress, VNet integration).
    * Comparison with ACI, AKS, and App Service.

* **📊 Module 3.8: Azure Batch Deep Dive - Large-Scale Compute for HPC**
    * Purpose: Running large-scale parallel and high-performance computing (HPC) applications.
    * Key components: Pools, compute nodes, jobs, tasks.
    * Scaling policies, low-priority VMs for cost optimization.
    * Integration with Azure Storage.
    * Use cases (rendering, simulation, financial modeling).

---