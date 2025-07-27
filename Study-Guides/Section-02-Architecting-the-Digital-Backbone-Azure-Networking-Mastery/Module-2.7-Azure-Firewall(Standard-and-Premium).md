### **🛡️ Module 2.7: Azure Firewall (Standard & Premium) Deep Dive**

Azure Firewall is a managed, cloud-based network security service that provides threat protection for your Azure Virtual Network resources. It's a fully stateful firewall as a service with built-in high availability and unrestricted cloud scalability. It offers both Layer 3-7 filtering capabilities and advanced threat protection features, making it a critical component for securing your network perimeter in Azure.

#### **What is Azure Firewall?**

Azure Firewall is designed to centralize network security policies across your virtual networks, allowing you to define, enforce, and log application and network connectivity rules for traffic flowing between subnets, virtual networks, and the Internet. It's a core component in hub-and-spoke network topologies, acting as the central point of ingress and egress inspection.

* **Key Features and Concepts (Applicable to both Standard and Premium):**
    * **Stateful Firewall as a Service:** Azure Firewall operates as a fully stateful firewall, meaning it tracks the state of network connections and makes decisions based on the context of the traffic (e.g., allowing return traffic for an established outbound connection).
    * **Built-in High Availability & Scalability:** Azure Firewall is deployed with built-in high availability, scaling automatically to handle varying network traffic loads up to 30 Gbps (Standard) or 100 Gbps (Premium). No additional load balancers are required. It can also be deployed across Availability Zones for increased resilience.
    * **Centralized Logging:** Integrates seamlessly with Azure Monitor, allowing you to centralize firewall logs (network rule logs, application rule logs, threat intelligence logs, IDPS logs, TLS inspection logs) into Log Analytics workspaces, Storage Accounts, or Event Hubs for analysis and auditing.
    * **Forced Tunneling:** You can configure User Defined Routes (UDRs) on your VNet subnets to force all internet-bound traffic to be routed through the Azure Firewall. This ensures all outbound traffic is inspected and filtered by the firewall.
    * **SNAT (Source Network Address Translation):** By default, Azure Firewall performs SNAT for all outbound traffic. This means the private IP addresses of your internal VMs/resources are translated to the firewall's public IP address(es) when they initiate connections to the internet or other networks that are not directly connected (e.g., via VNet peering without firewall transit). This hides your internal network topology.
        * **SNAT Port Exhaustion:** A common issue with high-volume outbound connections. Azure Firewall allocates 2,496 SNAT ports per public IP. To mitigate exhaustion, you can add multiple public IP addresses to the firewall or use Azure NAT Gateway for SNAT.
    * **DNAT (Destination Network Address Translation):** Configured via DNAT rules, this allows inbound internet traffic destined for the firewall's public IP address and a specific port to be translated to a private IP address and port of a backend resource within your VNet. This enables services like RDP or SSH access to VMs, or web server access. DNAT rules implicitly create corresponding network rules to allow the translated traffic.
    * **Rule Processing Logic (Order of Precedence):**
        1.  **DNAT Rules:** Processed first. If a match is found, the traffic is translated and immediately allowed. No further rules are processed.
        2.  **Network Rules:** Processed next if no DNAT match. These filter traffic based on Layer 3/4 properties (source/destination IP, port, protocol).
        3.  **Application Rules:** Processed last, only if no network rule match AND the protocol is HTTP/HTTPS or MS-SQL. These filter traffic based on FQDNs, URLs, and HTTP/HTTPS headers (Layer 7).
        * **Default Deny:** If no rule matches, traffic is implicitly denied by default.
    * **Rule Collections:** Rules are organized into Rule Collections (DNAT, Network, Application). Each collection has a priority (lower number = higher priority) and an action (Allow/Deny).
    * **Rule Collection Groups (Firewall Policy):** (Applicable when using Firewall Policy, which is the modern management method) Rule collections are further organized into Rule Collection Groups, also with priorities.
    * **Service Tags:** Represent a group of IP address prefixes for well-known Azure services (e.g., `AzureCloud`, `Storage`, `Sql`). You can use Service Tags in network rules to simplify rule creation and avoid maintaining lists of IP ranges. Azure automatically updates the IP ranges associated with a service tag.
    * **FQDN Tags:** Represent a group of Fully Qualified Domain Names (FQDNs) associated with well-known Microsoft services (e.g., `WindowsUpdate`, `AzureKubernetesService`, `Microsoft365`). You can use FQDN Tags in application rules to allow outbound traffic to these services easily, without needing to list individual FQDNs. Microsoft manages and updates these tags.

#### **Azure Firewall Standard SKU Deep Dive**

Azure Firewall Standard is a Layer 3-Layer 7 firewall providing enterprise-grade security and filtering for network traffic.

* **Features:**
    * **Stateful Filtering:** Filters Layer 3 (IP, Protocol) and Layer 4 (Port) network traffic.
    * **Application FQDN Filtering:** Filters outbound HTTP/S and Azure SQL traffic to a specified list of FQDNs (e.g., `*.google.com`, `myblob.blob.core.windows.net`). Does not require TLS termination.
    * **Network Filtering:** Allows or denies traffic based on source/destination IP address, port, and protocol (TCP/UDP/ICMP/Any).
    * **Threat Intelligence-based filtering:** Can be enabled to alert on or deny traffic from/to known malicious IP addresses and domains, sourced from Microsoft Threat Intelligence.
    * **DNS Proxy:** Can process and forward DNS queries, enabling reliable FQDN filtering in network rules even if VMs use a custom DNS server.
    * **Availability Zones support:** Can be deployed across zones for higher resilience.

* **Use Cases for Standard SKU:**
    * Securing basic hub-and-spoke topologies.
    * Controlling outbound access to the internet for VMs and applications.
    * Implementing centralized network security policies for multiple VNets.
    * Filtering inbound traffic for publicly exposed applications via DNAT.
    * Meeting basic compliance requirements for network segmentation.

#### **Azure Firewall Premium SKU Deep Dive**

Azure Firewall Premium builds upon the capabilities of the Standard SKU, offering advanced threat protection features required for highly sensitive and regulated environments.

* **Key Differences & Advanced Features:**
    1.  **TLS Inspection (HTTPS Inspection):**
        * **Purpose:** Decrypts outbound and East-West (intra-Azure/hybrid) HTTPS traffic, allowing the firewall to inspect the unencrypted data for threats, and then re-encrypts it before sending it to the destination.
        * **Mechanism:** Acts as a transparent proxy. For outbound TLS inspection, Azure Firewall presents its own certificate to the client, effectively performing a "man-in-the-middle" operation. The client must trust the root Certificate Authority (CA) used by the firewall.
        * **Certificate Requirements:** Requires an Intermediate CA certificate stored in Azure Key Vault, which is then configured on the Firewall Policy. Clients that will have their traffic inspected must trust the root CA that signed this intermediate CA.
        * **Benefits:** Crucial for detecting malware or exploits hidden within encrypted traffic that would otherwise bypass traditional firewalls.
    2.  **IDPS (Intrusion Detection and Prevention System):**
        * **Purpose:** Monitors network activities for malicious activity, logs information, and optionally attempts to block it. It uses signature-based detection to identify specific patterns (e.g., byte sequences, known attack instructions).
        * **Signatures:** Over 67,000 rules across more than 50 categories (malware, phishing, botnets, exploits, etc.). New rules are released daily.
        * **Modes:**
            * **Alert mode:** Detects malicious activity and generates alerts in logs. Traffic is *not* blocked by IDPS in this mode (it might still be blocked by other firewall rules).
            * **Alert & Deny mode:** Detects malicious activity, generates alerts, *and* blocks the suspicious traffic. This provides active prevention.
        * **Customization:** You can customize specific signature rules (disable, set to Alert, or Alert & Deny).
    3.  **URL Filtering:**
        * **Purpose:** Extends the FQDN filtering capability to consider the *entire URL* (e.g., `www.contoso.com/a/b/c`) for HTTP and HTTPS traffic (when TLS inspection is enabled for HTTPS).
        * **Benefit:** Provides more granular control over web access compared to FQDN filtering, which only considers the domain name.
    4.  **Web Categories:**
        * **Purpose:** Allows administrators to allow or deny user access to website categories such as gambling, social media, news, adult content, etc.
        * **Mechanism:** Utilizes a continuously updated database that classifies websites into categories based on their content. Premium SKU categorizes based on the *entire URL* for both HTTP and HTTPS (with TLS inspection), while Standard only uses the FQDN.
        * **Benefits:** Simplifies web access management by allowing policies to be applied to categories rather than individual FQDNs or URLs.

* **Use Cases for Premium SKU:**
    * **Highly Sensitive Applications:** Protecting applications in regulated environments (e.g., finance, healthcare) that require deep packet inspection.
    * **Preventing Malware/Viruses:** Detecting and preventing the spread of malware and viruses across networks, including within encrypted traffic.
    * **Advanced Content Filtering:** Implementing granular URL-based filtering and web category filtering for outbound internet access.
    * **Zero Trust Implementations:** As a key component for validating and securing all traffic flows, even within your trusted network.
    * **Securing East-West Traffic:** Inspecting traffic between different application tiers or subnets within your VNet (micro-segmentation).

#### **Deployment Architectures with Azure Firewall**

Azure Firewall is typically deployed in a hub-and-spoke network topology, where the hub VNet acts as a central point of connectivity and security, and spoke VNets host workloads.

1.  **Hub-and-Spoke with Azure Firewall:**
    * **Architecture:**
        * **Hub VNet:** Contains the Azure Firewall in its own `AzureFirewallSubnet`. It can also host shared services like VPN Gateways, ExpressRoute Gateways, and Bastion.
        * **Spoke VNets:** Host your application workloads. These VNets are peered with the Hub VNet.
        * **Routing:** User-Defined Routes (UDRs) are typically configured on the subnets within the spoke VNets to force all traffic (especially internet-bound and potentially cross-spoke traffic) to traverse the Azure Firewall in the hub.
    * **Traffic Flow:**
        * **Spoke to Internet:** Spoke VM -> Spoke VNet -> Peering -> Hub VNet -> Azure Firewall (SNAT applies) -> Internet.
        * **Internet to Spoke:** Internet -> Firewall Public IP -> Azure Firewall (DNAT applies) -> Hub VNet -> Peering -> Spoke VNet -> Spoke VM.
        * **Spoke to Spoke:** Spoke VM A -> Spoke A VNet -> Peering -> Hub VNet -> Azure Firewall -> Peering -> Spoke B VNet -> Spoke VM B. (Requires a UDR in each spoke to send traffic for other spokes via the firewall).
        * **On-premises to Spoke:** On-premises -> VPN/ExpressRoute Gateway (in Hub) -> Hub VNet -> Azure Firewall -> Peering -> Spoke VNet -> Spoke VM.
    * **Benefits:** Centralized security enforcement, simplified routing, easier management for large-scale deployments, consistent policy application.

2.  **Forced Tunneling Scenario:**
    * **Architecture:** Similar to hub-and-spoke, but all outbound internet traffic is *not* allowed to go directly to the internet from Azure. Instead, it is forced to go through the Azure Firewall, which then forwards it to an on-premises proxy or firewall via a VPN/ExpressRoute tunnel.
    * **Mechanism:** UDRs on Azure subnets direct `0.0.0.0/0` (all traffic) to the Azure Firewall's private IP. The Azure Firewall then has a default route or a network rule that forwards `0.0.0.0/0` to the VPN Gateway or ExpressRoute Gateway in the hub, sending traffic back to on-premises.
    * **Benefits:** Ensures all Azure-originated internet traffic is subject to on-premises security controls and logging, aiding compliance for highly regulated workloads.

#### **Benefits of Azure Firewall:**

* **Centralized Security:** Provides a single point of control for network security across your Azure environment.
* **Built-in Scalability & HA:** Automatically scales to meet demand and offers high availability without manual configuration.
* **Reduced Operational Overhead:** Managed service reduces the need to deploy, configure, and maintain individual firewall VMs.
* **Advanced Threat Protection:** Premium SKU offers deep packet inspection, IDPS, URL filtering, and web categories to counter sophisticated threats.
* **Integration with Azure Services:** Seamlessly integrates with VNets, Log Analytics, Azure Monitor, Private Link, and other security services.
* **Simplified Rule Management:** Uses Service Tags and FQDN Tags to simplify policy creation.

#### **Limitations/Considerations of Azure Firewall:**

* **Cost:** Can be a significant cost component, especially the Premium SKU and for high data volumes.
* **Learning Curve:** Understanding rule processing, SNAT/DNAT, and certificate management for TLS inspection can be complex.
* **Shared Resource:** It's a shared security boundary for your hub-and-spoke. For very specific, granular micro-segmentation *within* a subnet, NSGs or Application Security Groups (ASGs) might still be required in conjunction with the firewall.
* **Throughput Limits:** While highly scalable, there are still SKU-based throughput limits that need to be considered for extremely high-volume traffic.

---