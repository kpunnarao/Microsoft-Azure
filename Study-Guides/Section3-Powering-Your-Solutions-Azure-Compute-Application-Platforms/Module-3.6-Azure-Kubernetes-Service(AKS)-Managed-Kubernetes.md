### **☸️ Module 3.6: Azure Kubernetes Service (AKS) Deep Dive - Managed Kubernetes**

Azure Kubernetes Service (AKS) is Microsoft's managed Kubernetes offering, simplifying the deployment, management, and operations of Kubernetes clusters in Azure. 

Kubernetes is an open-source container orchestration system for automating application deployment, scaling, and management. 

AKS takes away the complexity and operational overhead of managing the Kubernetes control plane, allowing you to focus on developing and running your containerized applications.

#### **What is Azure Kubernetes Service (AKS)?**

AKS provides a fully managed Kubernetes control plane, meaning Microsoft handles the master nodes, API server, scheduler, and `etcd` (the cluster's database) at no cost. 

You only pay for the worker nodes (Virtual Machines) that run your application workloads.

* **Key Features and Core Functionality:**
    * **Managed Control Plane:** Azure handles patching, upgrades, and health monitoring of the Kubernetes control plane.
    * **Worker Nodes (Node Pools):** You provision and manage the Virtual Machines (VMs) that act as worker nodes, which are grouped into node pools.
    * **Automated Scaling:** Supports both Horizontal Pod Autoscaler (HPA) for scaling pods based on resource utilization and Cluster Autoscaler (CA) for scaling the number of nodes.
    * **Networking Integration:** Seamless integration with Azure Virtual Networks (VNets), offering various networking models.
    * **Integrated Security:** Deep integration with Azure Active Directory (Azure AD/Entra ID) for identity and access management, Kubernetes RBAC, Azure Policy, and network security features.
    * **Persistent Storage:** Supports Azure Disks, Azure Files, and Azure NetApp Files for stateful applications.
    * **DevOps Integration:** Seamlessly integrates with Azure DevOps, GitHub Actions, and other CI/CD tools.
    * **Monitoring:** Integrated with Azure Monitor (Container Insights) for comprehensive cluster and application monitoring.
    * **Multi-Node Pools:** Run different types of workloads (e.g., CPU-intensive vs. GPU-intensive) on distinct node pools within the same cluster.
    * **Windows Server Container Support:** Beyond Linux, AKS supports Windows Server containers.

#### **Key Components and Architecture of an AKS Cluster**

An AKS cluster is primarily composed of two components:

1.  **Control Plane (Managed by Azure):**
    * **Description:** The "brain" of the Kubernetes cluster. It's a collection of core Kubernetes components that manage the cluster's state, scheduling, and overall operation. Azure fully manages and maintains the control plane.
    * **Components:**
        * **Kube-API Server:** Exposes the Kubernetes API, the frontend for the Kubernetes control plane.
        * **etcd:** A consistent and highly available key-value store that stores all cluster data (cluster state, configuration, and metadata).
        * **Kube-Scheduler:** Watches for newly created pods with no assigned node and selects a node for them to run on.
        * **Kube-Controller-Manager:** Runs controller processes (e.g., Node Controller, Replication Controller, Endpoints Controller, Service Account Controller) that regulate the cluster's desired state.
    * **Connectivity:** The control plane typically communicates with worker nodes over a private network. You can also configure a "private cluster" where the API server's endpoint is only accessible within your VNet.

2.  **Nodes (Managed by You, on Azure VMs):**
    * **Description:** These are the Virtual Machines (VMs) that run your containerized applications (pods). They are provisioned in **Node Pools**.
    * **Components on each Node:**
        * **Kubelet:** An agent that runs on each node in the cluster. It ensures that containers are running in a pod, as specified by the API server.
        * **Kube-proxy:** A network proxy that runs on each node and maintains network rules on nodes, allowing network communication to your pods from inside or outside the cluster.
        * **Container Runtime:** The software that runs containers (e.g., containerd, Docker Engine - though containerd is the standard in modern Kubernetes).
    * **Node Pools:** Collections of VMs that have the same configuration.
        * **System Node Pool:** Hosts critical system pods (e.g., CoreDNS, `kube-proxy`, `metrics-server`). It's recommended not to run application workloads on system node pools. Must be Linux.
        * **User Node Pool:** Dedicated to hosting your application pods. You can have multiple user node pools, each with different VM sizes, OS types (Linux/Windows), or GPU capabilities.

#### **Networking Models in AKS: Kubenet vs. Azure CNI**

AKS offers flexible networking models to integrate your cluster with Azure Virtual Networks. The choice impacts IP address allocation, network performance, and integration with VNet features.

1.  **Kubenet (Basic Networking):**
    * **How it Works:** Pods receive an IP address from a logically separate address space that is *not* part of your Azure VNet. When pods need to communicate with resources outside the cluster, Kubenet uses Network Address Translation (NAT) to route traffic through the node's IP address.
    * **IP Address Management:**
        * **Node IPs:** Consume IP addresses from the Azure VNet subnet.
        * **Pod IPs:** Assigned from a private CIDR block (e.g., 10.244.0.0/16) that is *not* part of the VNet.
        * **Efficiency:** Conserves VNet IP address space because only nodes consume VNet IPs.
    * **Performance:** Introduces a slight overhead due to NAT for external communication.
    * **Features:** Basic Kubernetes network policies (Calico only), no Windows containers support. Limited to 400 nodes.
    * **Use Cases:**
        * Small-scale clusters.
        * Dev/test environments.
        * Workloads with limited VNet IP address space.
        * When most pod communication is internal to the cluster.

2.  **Azure CNI (Container Network Interface) (Advanced Networking):**
    * **How it Works:** Pods receive an IP address directly from the VNet subnet where the AKS nodes are deployed. This means pods are first-class citizens in the VNet.
    * **IP Address Management:**
        * **Node IPs:** Consume IP addresses from the VNet subnet.
        * **Pod IPs:** Each pod also consumes a distinct, routable IP address from the VNet subnet.
        * **Efficiency:** Requires careful VNet IP planning as each pod consumes an IP.
    * **Performance:** Direct pod-to-pod and pod-to-VNet communication, resulting in lower latency and better performance.
    * **Features:** Supports Azure Network Policies (or Calico/Cilium), Windows containers, Virtual Nodes (ACI integration), and direct VNet integration features like Service Endpoints and Private Link. Highly scalable.
    * **Variants of Azure CNI:**
        * **Traditional Azure CNI:** Pods directly consume IPs from the node subnet.
        * **Azure CNI with Dynamic IP Allocation:** Uses a separate, dedicated subnet for pod IPs, which are dynamically allocated, conserving the main VNet subnet space.
        * **Azure CNI Overlay:** Pods receive IPs from a private CIDR range *outside* the VNet, but routing is handled by the Azure network fabric without NAT for intra-cluster communication. It combines the IP efficiency of Kubenet with some of the performance benefits of Azure CNI.
        * **Azure CNI powered by Cilium:** Offers advanced network security, observability, and often better performance.
    * **Use Cases:**
        * Enterprise-grade production workloads.
        * Scenarios requiring direct pod access to VNet resources (e.g., Azure SQL DB, Azure Storage).
        * Hybrid connectivity scenarios with ExpressRoute or VPN.
        * Larger clusters with extensive IP address planning.
        * Windows Server container workloads.

#### **Scaling Options in AKS**

AKS provides multiple layers of scaling to ensure your applications can meet demand:

1.  **Manual Scaling (Pods & Nodes):**
    * Manually adjust the number of pod replicas in a deployment or the number of nodes in a node pool. Useful for testing or fixed-size environments.

2.  **Horizontal Pod Autoscaler (HPA):**
    * **Purpose:** Automatically scales the number of *pods* in a deployment or replica set based on observed metrics (e.g., CPU utilization, memory utilization, or custom metrics).
    * **Mechanism:** Watches the Metrics API and adjusts the replica count of your application pods.

3.  **Cluster Autoscaler (CA):**
    * **Purpose:** Automatically scales the number of *nodes* (VMs) in your AKS node pools.
    * **Mechanism:** If pods cannot be scheduled due to insufficient resources, the cluster autoscaler increases the number of nodes in the node pool. If nodes are underutilized for a period, it decreases the number of nodes.
    * **Integration:** Often used in conjunction with HPA: HPA scales pods, and if more pods need more nodes, CA scales the nodes.

4.  **Kubernetes Event-Driven Autoscaling (KEDA) Add-on:**
    * **Purpose:** Scales applications based on the number of events requiring processing. KEDA extends HPA to allow scaling based on metrics from 50+ event sources (e.g., Azure Service Bus queues, Kafka topics, Azure Event Hubs, Redis queues).
    * **Benefits:** Ideal for truly event-driven, "bursty" workloads where scaling based on CPU/memory might not be granular enough or reactive enough to event backlogs. Scales to zero and from zero.

5.  **Burst to Azure Container Instances (ACI) with Virtual Nodes:**
    * **Purpose:** Provides rapid bursting capabilities for your AKS cluster without provisioning new VMs.
    * **Mechanism:** Virtual Nodes (a feature of AKS) allow AKS to provision pods into ACI directly. These pods start in seconds and are billed per second by ACI.
    * **Benefits:** Eliminates the need to wait for VM node scale-out, handles unpredictable traffic spikes, and is highly cost-efficient for intermittent, bursty workloads that exceed current cluster capacity.
    * **Integration:** ACI becomes a secured, logical extension of your AKS cluster's compute capacity.

#### **Security Features and Best Practices for AKS**

Security in AKS is multi-layered, combining Kubernetes-native features with Azure's robust security capabilities.

1.  **Identity and Access Management (IAM) - Azure AD/Entra ID Integration:**
    * **Kubernetes RBAC:** Use Kubernetes Role-Based Access Control (RBAC) to define granular permissions within the cluster (who can do what to which Kubernetes resources).
    * **Azure AD Integration:** Integrate AKS with Azure AD (now Microsoft Entra ID) to manage access to the AKS cluster using existing corporate identities and groups. This centralizes authentication.
    * **Azure RBAC for AKS:** Beyond Kubernetes RBAC, Azure RBAC can control access to AKS *resources* in Azure (e.g., who can create/delete clusters, manage node pools). It also offers specific roles to control access to the Kubernetes API plane.
    * **Workload Identity (Recommended):** Enables Kubernetes service accounts to authenticate with Azure AD to access Azure resources (e.g., Azure Key Vault, Azure Storage) using managed identities, eliminating the need to manage secrets or connection strings in pod code.

2.  **Cluster Security:**
    * **Private Clusters:** Deploy an AKS cluster with a private API server endpoint, meaning the Kubernetes API server is only accessible via a private IP address within your Azure VNet or connected networks (e.g., via VPN/ExpressRoute/VNet peering), significantly reducing attack surface.
    * **Authorized IP Ranges:** Restrict public API server access to specific public IP ranges if a private cluster is not feasible.
    * **Regular Upgrades:** Keep your AKS cluster and Kubernetes versions up-to-date to benefit from the latest security patches and features.
    * **Network Security Groups (NSGs):** AKS automatically manages NSG rules on the node subnets. Avoid manual modification.
    * **Azure Policy for AKS:** Apply built-in or custom Azure Policies to enforce compliance and security standards on your AKS clusters (e.g., enforce specific container image registries, disable privileged containers, ensure HTTPS ingress).

3.  **Pod Security and Credential Protection:**
    * **Pod Security Standards (PSS):** Kubernetes provides PSS (formerly Pod Security Policies) to enforce security configurations on pods (e.g., preventing privileged containers, requiring read-only root filesystems). AKS enables PSS by default using admission controllers.
    * **Azure Key Vault Integration:** Use Azure Key Vault to store sensitive information (secrets, keys, certificates) and securely inject them into pods using the Azure Key Vault Provider for CSI Secrets Store driver. This avoids hardcoding secrets in container images or Kubernetes manifests.
    * **Container Image Security:** Use trusted container registries (like Azure Container Registry with vulnerability scanning) and regularly scan your images for vulnerabilities.

4.  **Network Security (within the cluster):**
    * **Network Policies:** Implement Kubernetes Network Policies (using Azure Network Policy Manager or Calico/Cilium) to control inbound and outbound network traffic between pods within the cluster, enabling micro-segmentation.
    * **Ingress Controllers:** Use secure Ingress controllers (e.g., NGINX Ingress, Azure Application Gateway Ingress Controller) for Layer 7 routing, SSL termination, and WAF capabilities.
    * **Egress Control:** Control outbound traffic from your AKS cluster using Azure Firewall or network virtual appliances (NVAs) to restrict access to external services.

#### **Advanced Features & Capabilities**

* **Node Pools for Different Workloads:** Create distinct node pools (e.g., general-purpose, GPU-enabled, memory-optimized) to efficiently run diverse workloads within a single cluster.
* **Persistent Volumes:** Integrate with Azure Disks (for single pod access) or Azure Files (for shared access) to provide persistent storage for stateful applications.
* **Container Insights (Azure Monitor):** Collect and visualize performance metrics, logs, and health status for your AKS cluster and containerized applications.
* **Azure Advisor:** Provides recommendations for optimizing cost, performance, security, and reliability of your AKS resources.
* **DevSpaces (Legacy/Limited):** Provides a rapid, iterative development experience for microservices on Kubernetes. (Often replaced by local development tools with remote debugging).
* **Confidential Compute Nodes (Preview):** Run your containers in hardware-based trusted execution environments (TEEs) for enhanced data confidentiality and integrity, even for data *in use*.
* **Pod Sandboxing (Preview):** Provides an isolation boundary between the container application and the shared kernel/compute resources of the container host.

#### **Common Use Cases for Azure Kubernetes Service (AKS)**

* **Microservices Architectures:** The primary use case for AKS, providing a highly scalable and resilient platform for deploying and managing complex microservice-based applications.
* **Lift and Shift of Existing Applications:** Containerizing legacy applications and migrating them to a managed Kubernetes environment for improved scalability and simplified operations.
* **Batch Processing:** Running large-scale, parallelizable batch jobs (e.g., financial simulations, data analysis) that require dynamic resource allocation.
* **Machine Learning and AI Workloads:** Deploying and scaling machine learning models, training jobs, and AI inference services, often utilizing GPU-enabled node pools.
* **IoT Solutions:** Processing high volumes of real-time data from IoT devices, performing edge computing, and orchestrating connected services.
* **DevOps and CI/CD:** Integrating with CI/CD pipelines to automate the build, test, and deployment of containerized applications, enabling faster release cycles.
* **Hybrid Cloud Solutions:** Extending on-premises Kubernetes deployments or integrating with on-premises resources via Azure Arc.
* **Gaming Servers:** Hosting and scaling online game servers that require low latency and high concurrency.

---