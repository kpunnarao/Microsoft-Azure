# Azure Compute Products:

Microsoft Azure Offers various Virtual Machine based compute products designed to meet different workload requirements. 

Standarad Virtual Machines(VMs), Virtual Machine Scale Sets(VMSS), and Compute Fleet are some of the Compute Products that provide flexibility and Scalibility for Various Applications.

<hr style="border: 0; height: 3px; background: #0078D4; margin-top: 12px; margin-bottom: 12px;">

## Virtual Machines (VMs):

<p align="left">
  <img src="Images/Azure-VM.Default" alt="Azure VM" width="100">
</p>

**Description:** 

- VMs are the fundamental building blocks of Azure Compute. 

- They Offer full Control over the Operating System and applications, making them suitable for a wide range of workloads. 

- VMs provide the flexibility to choose the operating system, size and configuration that best fits your needs.

**Use Cases:** 

- Hosting traditional applications, databases, development and test environments, ow when you need specific OS configurations or software that isn't supported by PaaS.

**Control Level:** 

- IaaS (highest control).

<hr style="border: 0; height: 3px; background: #0078D4; margin-top: 12px; margin-bottom: 12px;">

## Virtual Machine Scale Sets(VMSS):

<p align="left">
  <img src="Images/Azure-VMSS.png" alt="Azure VMSS" width="100">
</p>

**Description:**

- VMSS extend the capabilities of VMs by enabling the deployment and management of a group of VMs. 

- This service is ideal for applications that require high availability or scalability.

- VMSS can ensure that your applications can handle varying loads efficiently by automatically increasing or decreasing the number of VMs in response to changes in demand.

**Use Cases:**

- Highly available and scalable applications, web servers, and distributed workloads.

**Control Level:**

- IaaS, with added management for Scaling

<hr style="border: 0; height: 3px; background: #0078D4; margin-top: 12px; margin-bottom: 12px;">

## Compute Fleet:

- `Compute Fleet` is designed for large-scale, distributed computing environments. It allows you to manage and orchestrate a fleet of VMs across multiple regions and availability zones.

- This service is particularly beneficial for high-performance computing(HPC) applications, big data processing, and other workloads that require significant computational power and distributed resources.

<hr style="border: 0; height: 3px; background: #0078D4; margin-top: 12px; margin-bottom: 12px;">

## Azure App Service:

<p align="left">
  <img src="Images/Azure-AppService.png" alt="Azure AppService" width="100">
</p>

**Description:**

-  A fully managed platform for building, deploying and scaling web apps, mobile app backends, and RESTful APIs.

- It supports various programming languages (e.g., .NET, Node.js, Java, Python, PHP).

**Use Cases:**

- Web applications, APIs, mobile backends, and other HTTP-based workloads where you want to focus on development rather than infrastructure management.

**Control Level:**

- PaaS

<hr style="border: 0; height: 3px; background: #0078D4; margin-top: 12px; margin-bottom: 12px;">

## Azure Functions:

<p align="left">
  <img src="Images/Azure-Functions.jpg" alt="Azure Functions" width="100">
</p>

**Description:**

- A serverless compute service that lets you run small pieces of code ("functions") in response to events (e.g., HTTP requests, database changes, timer events). 

- You only pay for the execution time.

**Use Cases:**

- Event-driven applications, real-time data processing, IoT backend, automated workflows, and microservices.

**Control Level:**

- FaaS/Serverless

<hr style="border: 0; height: 3px; background: #0078D4; margin-top: 12px; margin-bottom: 12px;">

## Azure Container Instances(ACI):

<p align="left">
  <img src="Images/Azure-ACI.png" alt="Azure ACI" width="100">
</p>

**Description:**

- A serverless container service that allows you to run containers directly on Azure without managing the underlying virtual machines or orchestration platforms like Kubernetes.

**Use Cases:**

- Running single containers or small groups of containers for quick deployments, development, testing, and burstable workloads.

**Control Level:**

- PaaS/Serverless (for containers).

<hr style="border: 0; height: 3px; background: #0078D4; margin-top: 12px; margin-bottom: 12px;">

## Azure Kubernetes service(AKS):

<p align="left">
  <img src="Images/Azure-AKS.jpg" alt="Azure AKS" width="100">
</p>

**Description:**

- A managed Kubernetes service that simplifies the deployment, management, and scaling of containerized applications using Kubernetes. 

- Azure handles the master nodes, and you manage the worker nodes.

**Use Cases:**

- Microservices architectures, complex containerized applications, and workloads requiring robust orchestration, self-healing, and scaling capabilities.

**Control Level:**

- PaaS (for Kubernetes orchestration).

<hr style="border: 0; height: 3px; background: #0078D4; margin-top: 12px; margin-bottom: 12px;">

## Azure Batch:

<p align="left">
  <img src="Images/Azure-Batch.png" alt="Azure Batch" width="100">
</p>

**Description:**

- A managed service for running large-scale parallel and high-performance computing (HPC) applications. 

- It efficiently provisions and manages compute resources on demand.

**Use Cases:**

- Data processing, simulations, rendering, and other compute-intensive tasks that can be broken down into parallel jobs.

<hr style="border: 0; height: 3px; background: #0078D4; margin-top: 12px; margin-bottom: 12px;">

## Azure Dedicated Host:

<p align="left">
  <img src="Images/Azure-DedicatedHost.png" alt="Azure DedicatedHost" width="100">
</p>

**Description:**

- Provides physical servers dedicated to your Azure subscription, giving you hardware isolation for compliance and performance reasons.

**Use Cases:**

- Meeting specific compliance requirements, licensing benefits, or when you need predictable performance for your VMs.

<hr style="border: 0; height: 3px; background: #0078D4; margin-top: 12px; margin-bottom: 12px;">

## Choosing The Right Service:

The best Azure compute service depends on your application's specific requirements, your desired level of control, and your operational overhead preferences.

- **High Control & Customization:** Azure VMs

- **Web Apps & APIs (managed):** Azure App Service

- **Event-Driven & Serverless Logic:** Azure Functions

- **Simple Container Deployment:** Azure Container Instances

- **Complex Container Orchestration (Kubernetes):** Azure Kubernetes Service

- **Large-Scale Parallel Processing:** Azure Batch

- **Hardware Isolation/Compliance:** Azure Dedicated Host

Azure Compute offers a flexible and comprehensive range of options to meet diverse computing needs, empowering businesses to build, deploy, and scale their applications efficiently in the cloud.
