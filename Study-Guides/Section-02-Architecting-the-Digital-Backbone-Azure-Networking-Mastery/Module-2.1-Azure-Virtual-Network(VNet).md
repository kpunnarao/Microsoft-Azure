# **Module 2.1: Azure Virtual Network (VNet) Deep Dive**

The Azure Virtual Network (VNet) is the foundational building block for your private network in Azure. 

It enables Azure resources to securely communicate with each other, the internet, and on-premises networks. 

This module provides an in-depth exploration of VNet, its components, features, and best practices for planning and implementation.

## **What is an Azure Virtual Network (VNet)?**

An Azure VNet is a representation of your own network in the cloud. 

It's a logically isolated network in Azure, dedicated to your subscription. 

With VNet, you can define your own private IP address space, segment your network into subnets, and configure routes, DNS servers, and network security groups to control traffic flow.

### **Key Features and Components of an Azure VNet**

1.  **IP Address Space:**
    
    * **Description:** When you create a VNet, you define a private IP address range using CIDR (Classless Inter-Domain Routing) notation (e.g., `10.0.0.0/16`, `172.16.0.0/12`, `192.168.0.0/16`). This address space is private and routable only within the VNet and any directly connected networks (e.g., via VNet peering, VPN Gateway).
    
    * **Planning:** It's crucial to plan your IP address space carefully to avoid overlaps with your on-premises networks or other Azure VNets, especially if you anticipate hybrid connectivity or VNet peering in the future.
    
    * **Maximum Size:** A VNet can have a large address space (e.g., `/8` allows for millions of IPs).

2.  **Subnets:**
    
    * **Description:** A VNet can be divided into one or more subnets. Each subnet is a logical subdivision of the VNet's IP address space. Resources are deployed into specific subnets.
    
    * **Purpose:**
    
    * **Segmentation:** Isolate different types of resources (e.g., web servers in a "front-end" subnet, database servers in a "back-end" subnet).
    
    * **Security:** Apply different Network Security Groups (NSGs) to different subnets to control traffic flow between them.
    
    * **Delegation:** Certain Azure services (e.g., Azure SQL Managed Instance, Azure Container Instances, Azure Bastion) require dedicated subnets, sometimes called "delegated subnets," where the service takes control of network policies for that subnet.
    
    * **Reserved IPs:** Azure reserves the first four and last IP addresses in each subnet for internal use (e.g., network address, default gateway, Azure DNS).
    
    * **Minimum Size:** The smallest subnet you can create is a `/29` (5 usable IPs). For delegated subnets, `/27` or larger is often required.

3.  **Network Interface Cards (NICs):**
    
    **Description:** A NIC enables an Azure Virtual Machine (VM) to communicate with the internet, Azure, and on-premises resources. A VM can have one or more NICs.
    
    **IP Configuration:** Each NIC is assigned one or more IP configurations:
    
    * **Private IP Address:** Assigned from the subnet's address range. Can be dynamic (default) or static.
    
    * **Public IP Address (Optional):** Can be associated with the NIC to enable direct internet inbound/outbound communication.
    
    * **Association:** A NIC must be attached to a subnet within a VNet.
    
    * **Network Security Groups (NSGs):** NSGs can be associated directly with a NIC (in addition to or instead of a subnet).

4.  **IP Addressing (Deep Dive - Public vs. Private):**
    
    * **Private IP Addresses:**
    
        * **Purpose:** Used for communication within a VNet, between peered VNets, or with on-premises networks connected via VPN/ExpressRoute. Not directly routable over the public internet.
    
        * **Allocation Methods:**
    
            * **Dynamic (Default):** Assigned when the resource is created or started, and can change upon deallocation/reallocation.
    
            * **Static:** Assigned when the resource is created and remains the same even after the resource is stopped/started. Essential for resources that need a consistent internal IP (e.g., domain controllers, load balancer backends).
    * **Public IP Addresses:**
    
        * **Purpose:** Used for communication between Azure resources and the internet. Globally routable.
    
        * **SKUs (Stock Keeping Units):**
    
            * **Basic:** Simpler, supports dynamic or static allocation, no Availability Zone support, default outbound SNAT (Source Network Address Translation).
    
            * **Standard:** Recommended for production workloads. Supports static allocation only, Zone-redundant or Zonal options for high availability, secure by default (closed to inbound traffic until NSG rules allow), explicit outbound connectivity via SNAT or Load Balancer rules.
    
        * **Allocation Methods:**
            * **Dynamic:** Default for Basic SKU, changes on deallocation.
            * **Static:** Available for Basic and Standard SKUs, persists across resource stop/start.
    
        * **Use Cases:** Public-facing web servers, VPN gateways, public load balancers, Azure Firewall.

5.  **DNS Servers:**
    
    **Description:** VNets use DNS servers to resolve hostnames to IP addresses.
    
    **Options:**
    
    * **Azure-provided DNS (Default):** Automatic name resolution for resources within the VNet. Ideal for basic scenarios.
    
    * **Custom DNS Servers:** You can specify your own DNS servers (e.g., Active Directory Domain Controllers in Azure VMs, on-premises DNS servers connected via VPN/ExpressRoute, or third-party DNS solutions). Necessary for resolving names in on-premises domains or complex hybrid scenarios.
    
    * **Azure Private DNS Zones:** A managed service for private DNS resolution within and across VNets without deploying your own DNS servers (covered in Module 2.1's advanced networking topics).

6.  **Routing:**
    
    **System Routes (Default):** Azure automatically creates default routes for your VNet, enabling communication:
    
    * Between subnets within the VNet.
    
    * From the VNet to the internet.
    
    * Between the VNet and on-premises networks (if connected via gateway).
    
    **User-Defined Routes (UDRs):** You can create custom routes to override Azure's default routing or to route traffic to a network virtual appliance (NVA) like a firewall. Applied to subnets.
    
    * **Route Tables:** Collections of UDRs that are associated with subnets.

## **Benefits of Using Azure Virtual Networks**

**Isolation and Security:** 

* VNets provide a private and isolated environment, logically separated from other Azure tenants. You control network topology and traffic flow.

**Hybrid Connectivity:** 

* Seamlessly connect your Azure cloud resources with your on-premises infrastructure.

**Scalability and Flexibility:** 

* Easily expand your network, add subnets, and deploy resources as your needs grow.

**Granular Control:** 

* Use NSGs and UDRs to define precise traffic flow rules and routing.

**Optimized Performance:** 

* Traffic within a VNet or between peered VNets leverages Azure's high-speed backbone.

**Resource Organization:** 

* Group related resources within a VNet for easier management and lifecycle control.

## **Common Use Cases for Azure VNets**

**Hosting Multi-Tier Applications:** 

* Deploying web servers in one subnet, application servers in another, and databases in a third, with NSGs controlling traffic between tiers.

**Extending On-premises Data Centers:** 

* Creating a seamless hybrid environment where Azure acts as an extension of your existing network.

**Isolated Environments:** 

* Creating separate development, testing, and production environments, each within its own VNet, with controlled connectivity between them.

**Secure PaaS Service Integration:** 

* Using VNet integration or Private Link to securely connect Azure PaaS services (like App Service, Azure SQL Database) to your private VNet.

**Migrating On-premises Workloads ("Lift and Shift"):** 

* Moving existing applications and servers to Azure while maintaining similar network configurations.

---