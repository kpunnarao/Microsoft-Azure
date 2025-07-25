### **🌍 Module 4.8: Azure Cosmos DB - Globally Distributed, Multi-model NoSQL Database**

Azure Cosmos DB is Microsoft's globally distributed, multi-model database service. 

It provides turn-key global distribution, guaranteeing low-latency data access (single-digit millisecond reads and writes), high availability, and elastic scalability. 

It's designed for modern cloud applications that demand global reach, rapid response times, and flexible data models.

#### **I. What is Azure Cosmos DB?**

* **Globally Distributed:** You can distribute your data across any number of Azure regions worldwide with the click of a button. Cosmos DB transparently handles data replication and data locality.
* **Multi-model Database:** It supports various popular NoSQL APIs and data models, allowing you to use the one that best fits your application's needs.
* **Guaranteed Performance:** Offers comprehensive Service Level Agreements (SLAs) for throughput, latency (at the 99th percentile), availability, and consistency.
* **Elastic Scalability:** Scales throughput and storage independently and elastically, globally across any number of Azure regions.
* **Serverless and Cost-Effective:** Offers a serverless capacity model, allowing you to pay only for the Request Units (RUs) consumed, or a provisioned throughput model.

#### **II. Core Capabilities: Global Distribution & Guaranteed Performance**

1.  **Turn-key Global Distribution:**
    * **Mechanism:** You choose the Azure regions where your data should be replicated. Cosmos DB automatically handles data synchronization and consistency across these regions.
    * **Multi-Master Writes:** You can enable multi-master writes, allowing your application to write data to any region configured for your Cosmos DB account. This reduces write latency for globally distributed users and enhances write availability.
    * **Automatic Geo-failover:** In the event of a regional outage, Cosmos DB automatically fails over to the next prioritized region (if configured for multiple regions), ensuring continuous availability with no downtime for your application.
    * **Benefits:** Low-latency access for users worldwide, enhanced resilience and business continuity, simpler development for global applications.

2.  **Guaranteed Low Latency (Single-digit Milliseconds):**
    * Cosmos DB commits to single-digit millisecond latency for reads and writes at the 99th percentile for simple operations, backed by comprehensive SLAs.

3.  **High Availability (99.999% SLA):**
    * When configured with multiple regions and multi-master writes, Cosmos DB offers an industry-leading 99.999% availability SLA, even during regional outages.

#### **III. Request Units (RUs) and Provisioned Throughput**

* **Request Unit (RU):**
    * **Concept:** The fundamental unit of throughput in Azure Cosmos DB. It's a performance currency that abstracts the system resources (CPU, memory, IOPS) required to perform database operations.
    * **Cost of Operations:** Every database operation (e.g., read, write, query, update) consumes a certain number of RUs. Complex operations (e.g., complex queries, large documents) consume more RUs.
    * **Why RUs?:** Simplifies performance planning. Instead of managing CPU, memory, etc., you provision RUs for your database or container.

* **Provisioning Throughput:**
    * You provision RUs per second (RU/s) at the database or container level. This is the throughput capacity reserved for your operations.
    * **Models:**
        * **Standard (Manual) Throughput:** You specify a fixed number of RU/s. You are billed for this provisioned throughput continuously, regardless of actual usage. Ideal for consistent, predictable workloads.
        * **Autoscale Provisioned Throughput:** Allows Cosmos DB to automatically scale your RU/s within a specified min/max range based on your workload. It scales down to your minimum RU/s during idle periods, saving costs. You are billed for the maximum RU/s reached in the hour. Ideal for burstable or unpredictable workloads.
        * **Serverless Capacity:** You pay only for the Request Units consumed by your database operations, and storage consumed. There is no minimum RU/s to provision. Cosmos DB automatically handles the underlying compute scale. Ideal for spiky, unpredictable, or infrequent workloads and dev/test environments.

#### **IV. Consistency Models**

Cosmos DB offers five well-defined consistency models, providing a spectrum of choices between strong consistency (data is immediately updated everywhere) and high availability/low latency (data updates might propagate eventually). You select the model that best fits your application's needs.

1.  **Strong:**
    * **Description:** The most consistent. All reads see the most recent committed write.
    * **Trade-off:** Highest latency, lowest availability (if a replica is down, writes stop until it recovers).
    * **Use Cases:** Financial transactions, user profile updates where all reads *must* see the latest write.

2.  **Bounded Staleness:**
    * **Description:** Reads are guaranteed to lag behind writes by at most 'K' versions (number of updates) or 'T' time interval.
    * **Trade-off:** Good balance of consistency and performance.
    * **Use Cases:** Collaborative applications, scenarios where slightly stale data is acceptable within a defined boundary.

3.  **Session:**
    * **Description:** The most commonly used. Guarantees read-your-own-writes consistency. For a single client session, all reads will see the writes made by that session. Other sessions may see data eventually.
    * **Trade-off:** Excellent balance of consistency, latency, and availability for user-centric applications.
    * **Use Cases:** Shopping carts, user sessions, most interactive web applications.

4.  **Consistent Prefix:**
    * **Description:** Reads are guaranteed to see a prefix of all writes. This means you will never see out-of-order writes, but you might see a subset of the latest writes.
    * **Trade-off:** Weaker than Session, but stronger than Eventual. Good availability and latency.
    * **Use Cases:** Tagging, comments, social feed where the order matters, but seeing all latest updates isn't critical immediately.

5.  **Eventual:**
    * **Description:** The least consistent. Reads may eventually see the latest writes. No ordering guarantee for different writes.
    * **Trade-off:** Highest availability, lowest latency.
    * **Use Cases:** Count of likes, retweets, IoT telemetry data, non-critical data where high throughput and availability are paramount.

#### **V. Supported APIs (Multi-model Capabilities)**

Cosmos DB's multi-model nature means it supports various APIs, each mapping to a different data model and allowing developers to use familiar SDKs and tools.

1.  **SQL API (Core API - Document Database):**
    * **Data Model:** Document-oriented (JSON documents).
    * **Query Language:** SQL-like query language (Azure Cosmos DB SQL dialect).
    * **Use Cases:** General-purpose applications, web/mobile apps, IoT, gaming, e-commerce, user profiles, content management. This is the native API and the most feature-rich.

2.  **MongoDB API:**
    * **Data Model:** Document-oriented (JSON documents).
    * **Query Language:** MongoDB query language.
    * **Use Cases:** Existing MongoDB applications can migrate with minimal or no code changes, leveraging Cosmos DB's global distribution and guaranteed performance.

3.  **Cassandra API:**
    * **Data Model:** Column-family database.
    * **Query Language:** Cassandra Query Language (CQL).
    * **Use Cases:** Large-scale, high-throughput applications migrating from Apache Cassandra, especially those requiring linear scalability and multi-region writes.

4.  **Gremlin API:**
    * **Data Model:** Graph database.
    * **Query Language:** Apache TinkerPop Gremlin traversal language.
    * **Use Cases:** Applications involving complex relationships (e.g., social networks, recommendation engines, fraud detection, supply chain management).

5.  **Table API:**
    * **Data Model:** Key-value store (similar to Azure Table Storage).
    * **Query Language:** REST API.
    * **Use Cases:** Applications migrating from Azure Table Storage seeking global distribution, guaranteed throughput, and lower latency. Offers a superset of Table Storage features.

#### **VI. Partitioning and Indexing**

* **Partitioning (Sharding):**
    * **Concept:** To achieve high scalability, Cosmos DB distributes your data horizontally across logical and physical partitions. You define a **partition key** for your container.
    * **Logical Partition:** All items with the same partition key value form a logical partition.
    * **Physical Partition:** Cosmos DB automatically manages physical partitions, placing one or more logical partitions on each.
    * **Impact on Performance:**
        * **Good Partition Key:** Distributes data evenly and requests across partitions, preventing "hot partitions" (partitions receiving disproportionately high traffic). This ensures efficient scaling and resource utilization.
        * **Bad Partition Key:** Can lead to hot partitions, limiting scalability and potentially causing throttling.
        * **Cross-Partition Queries:** Queries that span multiple partitions are more expensive (consume more RUs) than single-partition queries.
    * **Recommendation:** Choose a partition key with high cardinality and even distribution of access patterns.

* **Indexing:**
    * **Automatic Indexing:** Cosmos DB automatically indexes all properties of items by default without schema or index management from the developer. This significantly simplifies development.
    * **Indexing Policies:** You can customize the indexing policy to optimize performance and cost for your specific workload (e.g., exclude certain paths, include specific paths, change index type).
    * **Benefit:** Enables fast query execution even for large datasets.

#### **VII. Change Feed**

* **Concept:** A persistent, ordered, read-only log of changes to data in your Cosmos DB container. It records item creations, updates, and deletions.
* **How it Works:** The change feed is available per logical partition key and processes changes from that partition.
* **Change Feed Processor:** A component of the Cosmos DB SDK that simplifies reading from the change feed, automatically managing leases and distributing work across multiple consumers.
* **Use Cases:**
    * **Real-time Analytics:** Powering real-time dashboards or analytics.
    * **Materialized Views:** Creating denormalized views of your data for faster queries.
    * **Event Sourcing:** Building event-driven architectures.
    * **Data Synchronization:** Synchronizing data with other systems (e.g., search indexes like Azure Cognitive Search, data warehouses, caches).
    * **Triggering Events:** Notifying other services or microservices when data changes.

---