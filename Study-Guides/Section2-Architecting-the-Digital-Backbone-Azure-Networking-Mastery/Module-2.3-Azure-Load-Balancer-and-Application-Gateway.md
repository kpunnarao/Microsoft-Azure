### **Module 2.3: Azure Load Balancer & Application Gateway Deep Dive**

This module provides an in-depth understanding of Azure Load Balancer and Azure Application Gateway, two critical regional load balancing services. We will explore their functionalities, components, use cases, and how they differ to help you choose the right solution for your application delivery needs.

#### **Azure Load Balancer (Layer 4 - Transport Layer)**

Azure Load Balancer is a high-performance, ultra-low-latency Layer 4 (TCP, UDP) load balancing service. It distributes incoming network traffic across healthy virtual machines or instances within the same Azure virtual network. It is ideal for balancing non-HTTP/HTTPS traffic or for simple, high-performance web applications where advanced Layer 7 features are not required.

* **Purpose and Core Functionality:**
    * **Traffic Distribution:** Distributes incoming flow connections from the frontend to instances within the backend pool based on load balancing rules.
    * **High Availability:** Ensures high availability by distributing traffic only to healthy instances.
    * **Outbound Connectivity:** Provides outbound connectivity for VMs inside a virtual network by translating their private IP addresses to the load balancer's public IP address (SNAT).
    * **Port Forwarding (Inbound NAT rules):** Allows direct access to specific ports on individual VMs in the backend pool.
    * **Protocol Support:** Operates at Layer 4, supporting TCP and UDP protocols. It does not inspect the content of the traffic (e.g., HTTP headers).

* **SKUs (Stock Keeping Units):**
    * **Basic SKU:**
        * **Scenario:** Designed for small-scale applications that do not require high availability, redundancy, or advanced features.
        * **Features/Limitations:**
            * Supports only Basic Public IPs.
            * Does not support Availability Zones.
            * Limited backend pool size (up to 300 instances).
            * SLA: Not provided.
            * Outbound connectivity uses automatic SNAT, which can lead to SNAT port exhaustion for high-volume outbound connections.
            * Management operations can be slower (60-90+ seconds).
            * No support for HA Ports, TCP Reset on Idle, Multi-frontends (outbound), Global VNet Peering, NAT Gateway, or Private Link.
            * **Important:** Basic Load Balancer is retiring on **September 30, 2025**. Migration to Standard SKU is highly recommended.
    * **Standard SKU:**
        * **Scenario:** Recommended for production workloads requiring high availability, zone redundancy, and advanced features.
        * **Features/Advantages:**
            * Supports only Standard Public IPs.
            * **Availability Zone Support:** Can be zone-redundant (distributes across multiple zones) or zonal (tied to a specific zone) for high resiliency.
            * **High Performance:** Built for high performance and ultra-low latency.
            * **SLA:** 99.99% availability SLA.
            * **Outbound Rules:** Provides declarative outbound NAT configuration with explicit control over SNAT behavior, mitigating port exhaustion issues.
            * **HA Ports:** A single rule can load balance on *all* ports simultaneously, useful for Network Virtual Appliances (NVAs).
            * **TCP Reset on Idle:** Helps manage connections more gracefully.
            * **Multiple Frontends:** Supports multiple frontend IP configurations for both inbound and outbound traffic.
            * **Global VNet Peering Support:** Compatible with Global VNet Peering for connecting internal load balancers across regions.
            * **Integration:** Supports NAT Gateway and Private Link.
            * Faster management operations (typically < 30 seconds).
        * **Recommendation:** Always use Standard SKU for new deployments.

* **Key Components of Azure Load Balancer:**
    1.  **Frontend IP Configuration:** The IP address(es) where incoming traffic arrives. Can be a Public IP (for external load balancing) or a Private IP (for internal load balancing).
    2.  **Backend Pool:** A group of virtual machines or instances that will receive the load-balanced traffic. Can include:
        * Individual Azure Virtual Machines
        * Virtual Machine Scale Sets
        * IP addresses within the VNet
    3.  **Health Probes:**
        * **Description:** A mechanism to monitor the availability of backend instances. The load balancer uses these probes to determine which instances are healthy and can receive traffic. If an instance fails the health probe, the load balancer stops sending new connections to it.
        * **Probe Types:**
            * **TCP:** Attempts to establish a TCP connection to a specified port on the backend instance. If the connection is established, the instance is considered healthy.
            * **HTTP/HTTPS:** Sends an HTTP/HTTPS GET request to a specified path and expects a 200 OK response. This allows for application-level health checks.
        * **Parameters:** Configurable interval (how often to probe), unhealthy threshold (number of consecutive failures before marking unhealthy), and probe timeout.
        * **Source IP:** Health probes originate from a specific Azure-internal IP address (168.63.129.16) and must not be blocked by NSGs on the backend instances.
    4.  **Load Balancing Rules:**
        * **Description:** Defines how inbound traffic arriving at the frontend IP and port is distributed to the backend pool based on protocol and backend port.
        * **Configuration:** Maps a frontend IP/port/protocol to a backend pool/port/protocol.
        * **Distribution Modes:** By default, it uses a 5-tuple hash (Source IP, Source Port, Destination IP, Destination Port, Protocol Type) to distribute traffic. Session affinity (source IP based) can also be configured.
    5.  **Inbound NAT Rules (Port Forwarding):**
        * **Description:** Maps a specific frontend IP and port to a specific port on an individual backend instance.
        * **Purpose:** Allows direct access to a specific VM in the backend pool for management or specific services, bypassing load balancing. Often used for RDP/SSH access to individual VMs.
    6.  **Outbound Rules (Standard SKU only):**
        * **Description:** Defines how outbound connections are made from the backend instances to the internet.
        * **Purpose:** Provides explicit control over SNAT behavior, allowing you to scale outbound connectivity, use multiple frontend IPs for SNAT, and manage SNAT port exhaustion.

* **Use Cases for Azure Load Balancer:**
    * **Load Balancing Internal Applications:** Distributing traffic between different tiers of an N-tier application (e.g., web tier to application tier).
    * **Basic Public-Facing Applications:** Simple web servers or services that need to be accessible from the internet but don't require advanced Layer 7 features like WAF or URL-based routing.
    * **High Availability for VMs:** Ensuring continuous availability of applications by routing traffic away from unhealthy VMs.
    * **Outbound Connectivity Management:** Providing and managing outbound internet access for VMs that do not have their own public IPs.
    * **Network Virtual Appliance (NVA) Insertion:** Using HA Ports to load balance traffic through firewalls or other NVAs.
    * **Gaming Servers:** High-performance, low-latency UDP traffic distribution.

#### **Azure Application Gateway (Layer 7 - Application Layer)**

Azure Application Gateway is a web traffic load balancer that enables you to manage traffic to your web applications. It operates at Layer 7 (HTTP/HTTPS) and provides advanced routing, security (Web Application Firewall), and optimization features, making it suitable for complex web application deployments.

* **Purpose and Core Functionality:**
    * **Application Delivery Controller (ADC) as a Service:** Provides features typically found in on-premises ADCs.
    * **Layer 7 Load Balancing:** Makes routing decisions based on HTTP request attributes like URL path, host headers, and cookies.
    * **SSL/TLS Termination (SSL Offloading):** Decrypts incoming TLS traffic at the gateway, significantly offloading the decryption burden from backend web servers. It can also re-encrypt for end-to-end TLS.
    * **Web Application Firewall (WAF):** Integrated protection against common web vulnerabilities (e.g., SQL injection, cross-site scripting) based on OWASP Core Rule Sets.
    * **Multi-site Hosting:** Host multiple web applications (with different domain names) on the same Application Gateway instance.
    * **URL-based Routing:** Direct traffic to different backend pools based on the URL path in the request.
    * **Session Affinity (Cookie-based):** Ensures requests from the same user session are routed to the same backend server.
    * **Autoscaling (v2 SKU):** Automatically scales capacity up or down based on traffic load.

* **SKUs (Stock Keeping Units):**
    * **v1 SKU (Standard/WAF):**
        * **Deprecation:** V1 SKU was announced for deprecation on April 28, 2023. **It is not supported after September 30, 2025.** All new deployments should use v2.
        * **Features:** Basic Layer 7 load balancing, SSL offloading, WAF (separate tier).
        * **Limitations:** Fixed size, no autoscaling, no zone redundancy, dynamic public IP, slower deployment/update times.
    * **v2 SKU (Standard_v2/WAF_v2):**
        * **Recommendation:** The latest and recommended SKU for all new deployments.
        * **Key Enhancements over v1:**
            * **Autoscaling:** Automatically scales based on traffic, removing the need to choose instance count or size.
            * **Zone Redundancy:** Can span multiple Availability Zones for higher resilience.
            * **Static VIP:** Uses a static public IP address for the gateway's lifetime.
            * **Performance Enhancements:** Up to 5x better TLS offload performance, faster deployment and update times.
            * **Header Rewrite:** Add, remove, or update HTTP request/response headers.
            * **Key Vault Integration:** Seamless integration with Azure Key Vault for managing TLS certificates.
            * **Mutual Authentication (mTLS):** Supports client certificate authentication.
            * **Private Link Support:** Enables private connectivity to the Application Gateway from other VNets/subscriptions.
            * **TCP/TLS Proxy (Preview):** In addition to HTTP/HTTPS, can proxy Layer 4 TCP/TLS traffic.
            * **Deeper WAF Integration:** WAF policies are more flexible and powerful.
        * **WAF_v2 Tier:** Includes all Standard_v2 features plus the Web Application Firewall.

* **Key Components of Azure Application Gateway:**
    1.  **Frontend IP Configuration:** The IP address(es) where the Application Gateway receives client requests. Can be Public, Private, or both. For v2, static IPs are standard.
    2.  **Listeners:**
        * **Description:** A logical entity that checks for incoming connection requests based on specified port, protocol (HTTP/HTTPS/TLS/TCP), host, and IP address.
        * **Purpose:** Determines how Application Gateway handles incoming traffic. Can be configured for SSL/TLS termination.
        * **Types:** Basic (single site) or Multi-site (for multiple hostnames).
    3.  **Request Routing Rules:**
        * **Description:** Connects a listener to a backend pool. Defines how traffic arriving at a listener is routed to the appropriate backend.
        * **Types:**
            * **Basic (Path-based):** Simple mapping of all traffic from a listener to a single backend pool.
            * **Path-based Routing:** Routes requests based on URL path patterns (e.g., `/images/*` to an image backend pool, `/video/*` to a video backend pool).
            * **Multi-site Routing:** Routes requests based on HTTP host headers (different domains to different backend pools).
    4.  **Backend Pools:**
        * **Description:** A collection of backend targets where the Application Gateway routes traffic.
        * **Backend Targets:** Can include:
            * Azure Virtual Machines
            * Virtual Machine Scale Sets
            * IP addresses or FQDNs (e.g., on-premises servers, external endpoints)
            * Azure App Service
            * Azure Kubernetes Service (AKS)
    5.  **HTTP Settings (Backend Settings):**
        * **Description:** Defines how Application Gateway connects to the backend servers.
        * **Configuration:** Includes settings like port, protocol (HTTP/HTTPS), cookie-based affinity, connection draining, request timeout, and whether to use end-to-end TLS encryption.
        * **End-to-End TLS:** Application Gateway can re-encrypt traffic before sending it to the backend servers, ensuring encryption throughout the path.
    6.  **Health Probes:**
        * **Description:** Monitors the health of the backend instances to ensure traffic is only sent to healthy servers.
        * **Types:** Default (basic ping) or Custom (recommended, allowing for specific URL paths, HTTP status codes, and timeouts).
        * **Integration:** Custom probes are associated with backend HTTP settings.
    7.  **Web Application Firewall (WAF):**
        * **Description:** An optional, integrated feature (WAF SKU) that provides centralized protection of your web applications from common exploits and vulnerabilities.
        * **Rule Sets:** Uses OWASP Core Rule Sets (CRS) to detect and block attacks (e.g., SQL injection, cross-site scripting, HTTP protocol violations).
        * **Modes:**
            * **Detection Mode:** Monitors and logs all detected threats without blocking them.
            * **Prevention Mode:** Blocks detected threats and logs them.
        * **Custom Rules:** Define your own rules to allow, block, or log traffic based on specific conditions (e.g., IP address ranges, HTTP headers, request attributes).
        * **Exclusion Lists:** Allow specific request attributes to bypass WAF inspection for certain rules, useful for false positive mitigation.
        * **Bot Protection:** (Premium/WAF v2 only) Managed rule set to protect against known malicious bots and crawlers.

* **Use Cases for Azure Application Gateway:**
    * **Securing Web Applications:** Protecting internet-facing web applications with a WAF against common web attacks.
    * **SSL/TLS Offloading:** Centralizing certificate management and offloading SSL decryption from backend web servers.
    * **Multi-site Hosting:** Hosting multiple distinct web applications on the same Application Gateway instance, using different domain names.
    * **URL-based Routing (Microservices):** Routing traffic to different backend microservices or tiers based on the URL path.
    * **Session Affinity:** Maintaining user sessions on the same backend server for stateful applications.
    * **High Availability for Web Applications:** Distributing traffic and ensuring failover for web workloads within a region.
    * **Integration with PaaS Services:** Load balancing traffic to Azure App Services, Azure Kubernetes Service (AKS) as an Ingress Controller, or other web-enabled services.
    * **Custom Error Pages:** Displaying custom error pages to users instead of default server errors.

---