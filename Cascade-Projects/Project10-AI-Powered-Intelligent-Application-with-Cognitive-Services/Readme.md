# 🚀 Project 10: AI-Powered Intelligent Application with Cognitive Services

## Scenario: Fabrikam's Customer Feedback Analysis System

Fabrikam, a large consumer electronics company, receives a massive volume of customer feedback daily through various channels: product reviews on their website, social media comments, and support email transcripts. Manually sifting through this feedback is time-consuming and often leads to missed insights. They want to build an intelligent application that can:
* Automatically ingest customer feedback from different sources.
* Analyze the sentiment of the feedback to quickly identify positive, negative, or neutral opinions.
* Extract key phrases and entities (e.g., product names, features, locations) from the text.
* Categorize feedback topics (e.g., battery life, screen quality, customer service).
* Provide a summary dashboard for product managers and customer support leads.
* (Bonus) Automatically route negative feedback to the appropriate support team.

### Key Requirements:

**Functional Requirements:**
* Ingest text data from various sources (simulated via file uploads or API calls).
* Perform sentiment analysis on incoming text to classify it as positive, negative, or neutral.
* Extract key entities (people, places, organizations) and phrases.
* Identify the dominant language of the text.
* Categorize feedback based on its content (e.g., 'technical issue', 'feature request', 'delivery problem').
* Display analyzed insights (sentiment trends, top entities, common phrases) on a dashboard.
* (Bonus) Trigger an action (e.g., create a support ticket, send an email) for negative feedback.

**Non-Functional Requirements:**
* **Scalability:** The system must scale to handle thousands of feedback submissions per day.
* **Accuracy:** The AI models should provide reasonably accurate sentiment and entity extraction.
* **Low Latency:** Analysis should be performed near real-time, allowing insights to appear on the dashboard quickly.
* **Security:**
    * Secure access to the API endpoints for submitting feedback.
    * Secure access to the Cognitive Services keys.
    * Data at rest and in transit must be encrypted.
* **Cost-Effectiveness:** Leverage managed AI services to minimize development and operational overhead.
* **Manageability:** The solution should be easy to monitor and maintain.

### Learning Objectives:

Upon completing this project, you should be able to:

* Design an architecture that integrates Azure Cognitive Services (Azure AI Services) into an application workflow.
* Select appropriate Cognitive Services for specific AI tasks (e.g., sentiment analysis, key phrase extraction, language detection).
* Understand the capabilities and limitations of pre-trained AI models.
* Design a data pipeline for ingesting and processing unstructured text data.
* Implement a solution for storing and visualizing AI-derived insights.
* Secure access to Cognitive Services endpoints and keys.
* Apply the principles of the Azure Well-Architected Framework (Performance Efficiency, Operational Excellence, Cost Optimization) in an AI context.

### Guidance/Hints:

* **Ingestion:** How will feedback enter your system? Consider a simple web API (Azure Functions or App Service) or direct ingestion into a storage account.
* **Core AI Services:**
    * Which **Azure AI Service (Cognitive Service)** API is best suited for analyzing sentiment and extracting key phrases/entities from text? (Hint: Text Analytics or Language Service).
    * Which API can automatically detect the language of the input text? (Also part of Language Service).
    * For **categorization**, if it's based on specific keywords or patterns, you might use rules within an Azure Function. For more complex, flexible categorization, you could explore **Custom Text Classification** within the Language Service, or even a basic Azure Machine Learning custom model if you have labeled data. Start with rule-based/pre-trained if possible.
* **Orchestration/Processing:** Once data is ingested, what service can act as the glue to call the Cognitive Services APIs, process their responses, and store the results? (Hint: Azure Functions, Logic Apps, or even an App Service backend).
* **Storage for Insights:** Where will the analyzed results (e.g., sentiment score, extracted entities, categorization) be stored for querying and dashboarding? Consider a NoSQL database like **Azure Cosmos DB** for its flexibility and scalability for JSON-like data, or **Azure SQL Database** if you want a more structured approach for reporting.
* **Dashboarding:** How will product managers visualize the sentiment trends and top entities? **Power BI** is an excellent choice, connecting to your chosen insights database.
* **Security:** How will your application securely call the Cognitive Services APIs? (Hint: API keys, Managed Identities if calling from Azure services).
* **Scalability:** Ensure the ingestion and processing components (e.g., App Service plan, Function App plan) can scale. Cognitive Services APIs themselves are highly scalable.

### Deliverables:

1.  **High-Level Architecture Diagram:** A visual representation of your proposed intelligent application, showing the data flow from feedback sources through ingestion, AI processing, storage of insights, and finally to the dashboard. Clearly label all chosen Azure services.
2.  **Service Justification Document (Brief):** A short write-up (e.g., 1-2 pages) explaining:
    * Your choice of Azure AI Services (Cognitive Services) APIs for each required AI capability and their configuration.
    * Your selected services for ingestion, processing orchestration, and insights storage.
    * How your design addresses scalability, security, and low latency.
    * Any significant trade-offs considered (e.g., different database choices, different orchestration tools).
3.  **(Optional) Data Model Sketch:** A simple sketch of how you would structure the analyzed feedback data in your chosen database (e.g., JSON structure for Cosmos DB).

### Evaluation Criteria:

Your solution will be evaluated based on:

* **AI Integration:** Is the use of Azure AI Services appropriate and effective for the specified requirements?
* **Architectural Soundness:** Is the overall data flow logical, scalable, and resilient?
* **Service Selection:** Are the chosen Azure services well-suited for an AI-powered application?
* **Security:** Are API keys and data access handled securely?
* **Insights & Visualization:** Does the design provide a clear path to generate and present valuable business insights?
* **Clarity of Diagram & Explanation:** Is your architecture easy to understand and well-documented?

### Advanced Challenges/Extensions (Optional):

For those who want to push further:

* Integrate an **image recognition** (Computer Vision) Cognitive Service to process feedback that includes images (e.g., photos of damaged products).
* Implement a **custom text classification model** using Azure Machine Learning if the provided Cognitive Services categorization isn't sufficient for Fabrikam's specific needs (requires labeled data).
* Add a **chatbot (Azure Bot Service)** that interacts with customers to collect structured feedback and potentially provide instant resolutions.
* Explore **Azure AI Search** for creating a searchable index of all feedback, enabling powerful natural language queries on the dataset.
* Consider **Responsible AI** principles: How would you address potential biases in the AI models or ensure transparency if custom models are used?

---

This concludes the initial set of 10 Cascade Projects! Each project is designed to tackle different facets of the Solutions Architect role and challenge you to think holistically. Good luck, and keep learning!