### **🔒 Module 7.5: Azure Network Security - Firewalls, DDoS Protection & Web Application Firewall (WAF)**

Network security is a cornerstone of any cloud architecture. 

In Azure, you have a suite of services designed to protect your network infrastructure from various threats, from volumetric attacks to sophisticated application-layer exploits. 

This module focuses on three critical services: **Azure Firewall**, **Azure DDoS Protection**, and **Azure Web Application Firewall (WAF)**, and how they work together to create a robust, layered defense.

#### **I. Understanding the Layered Defense (Defense in Depth)**

A strong network security strategy in Azure follows the principle of **Defense in Depth**, employing multiple layers of security controls to protect your assets. These services operate at different layers of the OSI model:

* **Azure DDoS Protection:** Primarily operates at Network Layer (Layer 3) and Transport Layer (Layer 4) to protect against volumetric and protocol attacks.
* **Azure Firewall:** A stateful firewall that operates at Network Layer (Layer 3) and Transport Layer (Layer 4), and Application Layer (Layer 7) for FQDN filtering, providing centralized network security.
* **Azure Web Application Firewall (WAF):** Operates at the Application Layer (Layer 7) to protect web applications from common web vulnerabilities.

By combining these services, you create a comprehensive defense from the network edge to your application workloads.

#### **II. Azure Firewall**

Azure Firewall is a managed, cloud-native network security service that provides threat protection for your Azure Virtual Network resources. It's a fully stateful firewall as a service with built-in high availability and unrestricted cloud scalability.

**A. Key Features & Capabilities:**

1.  **Stateful Firewall as a Service (FaaS):**
    * Maintains the state of connections, allowing it to make intelligent filtering decisions for both inbound and outbound traffic based on the context of the traffic flow.
    * Offers built-in high availability and autoscales to meet varying traffic demands, ensuring performance and continuous protection.
2.  **Centralized Policy Management:**
    * Allows you to centrally create, enforce, and log application and network connectivity policies across multiple subscriptions and virtual networks.
    * Uses **Rule Collections** (Network Rules, Application Rules, DNAT Rules) with definable priorities.
        * **Network Rules:** Filter traffic based on source/destination IP addresses, ports, and protocols (Layer 3/4).
        * **Application Rules:** Filter outbound HTTP/S traffic based on Fully Qualified Domain Names (FQDNs), even for non-HTTP/S protocols (e.g., SQL FQDNs).
        * **DNAT Rules:** Translate inbound public IP addresses to private IP addresses within your VNet for server access.
3.  **Threat Intelligence-based Filtering:**
    * Integrates with Microsoft's real-time threat intelligence feeds to automatically alert on or deny traffic from/to known malicious IP addresses and domains.
4.  **Availability Zones Support:**
    * Can be deployed across Azure Availability Zones for enhanced resiliency.
5.  **Integration with Azure Monitor & SIEM:**
    * Provides comprehensive logging of traffic processing, threat intelligence alerts, and FQDN filtering, which can be sent to Azure Monitor Log Analytics, Storage Accounts, or a SIEM like Microsoft Sentinel for further analysis and alerting.
6.  **Service Tags & FQDN Tags:**
    * Simplifies rule creation by allowing you to specify abstract groups of IP addresses (Service Tags for Azure services like Storage, SQL) or known FQDNs (e.g., Windows Update) instead of individual IPs.
7.  **Forced Tunneling:**
    * Can route all internet-bound traffic from Azure virtual networks to a specified on-premises firewall via a VPN Gateway or ExpressRoute, ensuring all traffic is inspected by your existing security appliances.

**B. Azure Firewall SKUs:**

* **Azure Firewall Basic:**
    * Designed for SMBs.
    * Fixed scale unit (two VM instances), recommended for up to 250 Mbps throughput.
    * Includes L3-L7 filtering, SNAT/DNAT, Service/FQDN tags.
    * Supports Threat Intel in *alert only* mode.
* **Azure Firewall Standard:**
    * Recommended for most enterprise scenarios.
    * Autoscales up to 30 Gbps throughput.
    * Includes all Basic features plus: Threat Intelligence-based filtering (alert *and* deny), DNS Proxy, Custom DNS, Web Categories (for FQDN filtering), and enhanced logging.
* **Azure Firewall Premium:**
    * Designed for highly sensitive and regulated environments.
    * Autoscales up to 100 Gbps throughput.
    * Includes all Standard features plus advanced threat protection capabilities:
        * **TLS Inspection:** Decrypts outbound and East-West HTTPS traffic, inspects it for threats, and then re-encrypts it. Crucial for detecting threats within encrypted traffic.
        * **Intrusion Detection and Prevention System (IDPS):** Signature-based IDPS to detect and block network intrusions by identifying malicious patterns (signatures) in network traffic. Over 67,000 signatures, regularly updated.
        * **URL Filtering:** Extends FQDN filtering to inspect the entire URL path for HTTP/S traffic, providing more granular control over web access.

**C. Common Use Cases:**

* **Hub-and-Spoke Network Topology:** Centralizing outbound and East-West traffic inspection in a hub VNet.
* **Segmenting Virtual Networks:** Enforcing strict network segmentation between different application tiers or environments.
* **Outbound Internet Access Control:** Filtering and logging all outbound traffic from Azure VMs and services to the internet.
* **Hybrid Cloud Security:** Protecting traffic flowing between on-premises networks and Azure.

#### **III. Azure DDoS Protection**

Azure DDoS Protection helps protect your Azure resources from Distributed Denial of Service (DDoS) attacks. It's a critical service for applications that need to be highly available and resilient to large-scale network attacks.

**A. Key Features & Capabilities:**

1.  **Always-on Monitoring and Automatic Attack Mitigation:**
    * Continuously monitors incoming traffic for signs of DDoS attacks.
    * Automatically initiates mitigation when attack patterns are detected, without any user intervention.
    * Leverages Azure's global network scale to absorb and mitigate even very large volumetric attacks.
2.  **Adaptive Real-time Tuning:**
    * Learns the legitimate traffic patterns of your applications over time to fine-tune protection policies. This helps minimize false positives and ensures effective mitigation tailored to your specific application.
3.  **DDoS Attack Analytics, Telemetry, and Alerting:**
    * Provides rich telemetry via Azure Monitor, including attack mitigation reports, flow logs (details of dropped and forwarded traffic), and key metrics.
    * Allows you to configure alerts for attack detection, mitigation start/stop, and duration.
4.  **Cost Protection:**
    * Offers service credits for resource scale-out costs incurred during a documented DDoS attack.
5.  **DDoS Rapid Response (DRR) Support:**
    * For DDoS Network Protection customers, access to Microsoft's DDoS Rapid Response team for expert assistance during active attacks.
6.  **Integration with WAF:**
    * While DDoS Protection focuses on Layer 3/4 attacks, it's designed to work in conjunction with a Web Application Firewall (WAF) to provide comprehensive Layer 7 protection.

**B. Azure DDoS Protection Tiers:**

* **Basic (Infrastructure) Protection:**
    * This is **always-on and free** for all Azure services with public IPv4/IPv6 addresses. It protects the Azure infrastructure itself.
    * Provides platform-level protection against large-scale network attacks that might impact the Azure platform and customers.
    * No user configuration or telemetry is provided at this tier.
* **DDoS IP Protection / DDoS Network Protection (Paid Tiers):**
    * **DDoS IP Protection:** A newer, more granular tier where you enable protection on individual Standard Public IP addresses. Ideal for protecting a smaller number of critical IPs.
    * **DDoS Network Protection:** Provides protection for an entire virtual network, safeguarding all public IPs within that VNet.
    * **Key features beyond Basic:** Adaptive real-time tuning, attack analytics, telemetry, alerting, cost protection, and DDoS Rapid Response.
    * **Pricing:** Fixed monthly charge per DDoS protection plan that covers up to 100 public IP addresses within the enabled virtual networks or for the explicitly protected IPs.

**C. Common Use Cases:**

* **Protecting Internet-Facing Applications:** Safeguarding web applications, APIs, and services exposed to the internet from volumetric attacks (e.g., SYN floods, UDP floods, HTTP floods at L3/4).
* **Ensuring Business Continuity:** Maintaining the availability and performance of critical business services during a DDoS attack.

#### **IV. Azure Web Application Firewall (WAF)**

Azure WAF is a cloud-native service that provides centralized protection of your web applications and APIs from common web exploits and vulnerabilities. It primarily operates at Layer 7 (Application Layer) of the OSI model.

**A. Key Features & Capabilities:**

1.  **Protects against Common Web Vulnerabilities:**
    * Defends against attacks listed in the OWASP (Open Web Application Security Project) Top 10, such as SQL injection, Cross-Site Scripting (XSS), cross-site request forgery, file inclusion, and more.
2.  **Managed Rule Sets:**
    * Microsoft provides and automatically updates managed rule sets (based on OWASP Core Rule Set - CRS 2.2.9, CRS 3.0, CRS 3.1) to detect and block common threats without requiring manual tuning.
3.  **Custom Rules:**
    * Allows you to create your own custom rules to block or allow traffic based on specific conditions (e.g., IP addresses, HTTP headers, request methods, query strings). These can supplement managed rules or address specific application-level threats not covered by default.
4.  **Detection and Prevention Modes:**
    * **Detection Mode:** Logs requests that match WAF rules but still allows them to pass. Useful for testing and understanding potential false positives before enforcing prevention.
    * **Prevention Mode:** Blocks requests that match WAF rules and logs them. This is the production-ready mode for active protection.
5.  **Rate Limiting and Geo-filtering:**
    * Can configure rules to limit the number of requests from a specific IP address within a time period (rate limiting) or block traffic from specific geographic locations (geo-filtering).
6.  **Bot Protection (on Front Door WAF):**
    * Managed bot protection rules that automatically identify and block known malicious bots while allowing legitimate bots.
7.  **Integration and Deployment Options:**
    * Azure WAF is not a standalone service but is integrated as a feature of other Azure services:
        * **Azure Application Gateway (Regional):** Best for protecting web applications deployed within a single Azure region. Provides application-level load balancing, SSL offloading, and WAF capabilities.
        * **Azure Front Door (Global):** Best for protecting global web applications, providing content delivery network (CDN) capabilities, global load balancing, and WAF at the edge (closest to the user). Offers lower latency and protection at Azure's network edge.
8.  **Logging and Monitoring:**
    * WAF logs are integrated with Azure Monitor Log Analytics, allowing for detailed analysis of blocked requests and security insights.

**B. WAF on Application Gateway vs. Azure Front Door:**

| Feature         | Azure Application Gateway WAF                             | Azure Front Door WAF                                |
| :-------------- | :-------------------------------------------------------- | :-------------------------------------------------- |
| **Scope** | **Regional** (within a single Azure region/VNet)          | **Global** (at Azure's network edge)                |
| **Primary Use** | Protecting web apps within a VNet in a single region      | Protecting global web apps/APIs, content delivery   |
| **Load Bal. Type**| Regional Layer 7 load balancer                          | Global Layer 7 load balancer / CDN                  |
| **Latency** | Higher latency for geographically dispersed users         | Lower latency for global users (Anycast routing)    |
| **IP Address** | Public IP (can be internal too)                           | Azure Front Door's anycast IP                       |
| **TLS Offloading** | At the Application Gateway                              | At the edge of Azure's network (closest to user)  |
| **Bot Protection** | Limited (via custom rules)                              | Managed Bot protection set (Premium)                |
| **Cost Model** | Per-hour pricing + data processed, WAF fixed fee per gateway. | Per-hour pricing + data processed, WAF included in Premium tier. |

**C. Common Use Cases:**

* **Protecting Web Applications:** Safeguarding public-facing websites, e-commerce platforms, and web APIs.
* **Compliance Requirements:** Helping to meet compliance standards (e.g., PCI DSS) that require WAF protection for web applications.
* **Mitigating Application-Layer Attacks:** Blocking common attacks like SQL injection, XSS, and broken authentication attempts.

---