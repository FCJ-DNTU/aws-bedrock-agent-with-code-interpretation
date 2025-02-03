---

title: "Introduction to Amazon Bedrock Agent"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

#### **Overview of Amazon Bedrock**  
Amazon Bedrock is a fully managed service that enables businesses to easily and efficiently access and use high-performance **Foundation Models (FMs)**.  

---

#### **Introduction to Amazon Bedrock Agent**  

Amazon Bedrock Agent enables **Generative AI** applications to perform multi-step tasks by connecting to **enterprise systems, APIs, and data sources**.  

The Agent leverages the **reasoning capabilities of Foundation Models (FMs)** combined with APIs and data to **analyze user requests, retrieve necessary information, and complete tasks efficiently**.  

{{% notice note %}}
In this workshop, we will use the **Foundation Model [Claude 3 Sonnet](https://aws.amazon.com/blogs/aws/anthropics-claude-3-sonnet-foundation-model-is-now-available-in-amazon-bedrock/#:~:text=Introduction%20of%20Anthropic%E2%80%99s%20Claude%203%20Sonnet)**.
{{% /notice %}}


---

#### **Architecture Analysis of Amazon Bedrock Agent with Code Interpreter**  

The diagram below illustrates how **Amazon Bedrock Agent** integrates with **Code Interpreter** to handle data analysis, visualization, and complex computation tasks.  

![architecture](/images/architecture-workshop-04-bedrock-agent.svg)

**Main Workflow:**  
1. **Customer** submits a query or uploads a data file.  
2. **Analytics Agent** in Amazon Bedrock receives the query and generates a response based on the Large Language Model (LLM). If code execution is required, the agent forwards the task to **Code Interpreter**.  
3. **Code Interpreter** receives the input file or processing request, performs calculations, analysis, or data visualization.  
4. Upon completion, **Code Interpreter** generates output files and sends the results back to **Analytics Agent**.  
5. **Analytics Agent** consolidates the information and responds to the customer.  

**File Processing Workflow:**  
- Customers can **upload data files** for processing.  
- **Analytics Agent** shares these files with **Code Interpreter**.  
- **Code Interpreter** executes code on the input files and generates **output files**.  
- These output files are shared with the customer for download or further use.  

---

#### **Key Features and Applications of Amazon Bedrock Agent**  

**1. Multi-agent Collaboration**  
This feature allows developers to easily **create, deploy, and manage specialized Agents**, enabling them to work together to handle complex business processes.  

![alt text](/images/1-theory/image.png)

**2. Retrieval-Augmented Generation (RAG)**  
The Agent can **connect to enterprise data sources** to provide accurate responses to customer queries.  

🔹 **Example:** A customer asks: *"How did the history of Python begin?"*  
- **Step 1:** The AI searches for information from an external source (e.g., Wikipedia).  
- **Step 2:** The AI synthesizes and generates a response based on the retrieved data.  
- **Result:** *"Python was created by Guido van Rossum in 1991 to make programming more readable and efficient."*  

👉 **Benefit:** Enables the AI to answer questions for which data is not available in the model but can be retrieved from external sources.  

![alt text](/images/1-theory/image-1.png)

**3. Orchestrate and Execute Multi-step Tasks**  
Customers can select an **AI model** and provide simple instructions, such as:  
*"You are a warehouse management Agent, help identify available products in the system."*  

The Agent will **analyze the task, break it down into logical steps**, and automatically call the necessary APIs to complete the request.  

**4. Memory Retention Across Interactions**  
Amazon Bedrock Agent can **retain information from previous interactions**, providing a seamless and personalized experience.  

For example, if a customer previously inquired about **inventory for product X**, the Agent can remember this information and avoid requesting it again in subsequent queries.  

![alt text](/images/1-theory/image-2.png)

**5. Code Interpretation**  
The Agent can **generate and execute code in a secure environment**, automating complex analytical queries.  

![alt text](/images/1-theory/image-3.png)

👉 **Applications:**  
✅ **Data analysis**  
✅ **Data visualization** (charts, graphs)  
✅ **Solving complex mathematical problems**  

**6. Prompt Engineering**  
The Agent can **automatically generate prompt templates** from **customer instructions, action groups, and knowledge bases**.  
Customers can also **refine inputs, orchestration plans, and model responses** to improve results.  

---

#### **Cost Considerations for Amazon Bedrock Agent**

In this workshop, we use the **Claude 3 Sonnet** model, with specific costs as follows:  

| **Anthropic models** | **Price per 1,000 input tokens** | **Price per 1,000 output tokens** | **Price per 1,000 input tokens (batch)** | **Price per 1,000 output tokens (batch)** | **Price per 1,000 input tokens (cache write)** | **Price per 1,000 input tokens (cache read)** |
|----------------------|--------------------------------|----------------------------------|-----------------------------------------|------------------------------------------|--------------------------------------------|--------------------------------------------|
| **Claude 3 Sonnet**  | $0.003                         | $0.015                           | $0.0015                                  | $0.0075                                   | N/A                                        | N/A                                        |


{{% notice info %}}
📌 **Note:** Costs may vary depending on usage and deployment methods. Refer to the [official AWS documentation](https://aws.amazon.com/bedrock/pricing/) for more details.  
{{% /notice %}}