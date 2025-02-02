---  
title: "Sending Files to the Agent"  
weight: 2  
chapter: false  
pre: " <b> 4.2. </b> "  
---  

### **Sending Files to the Agent**  

We can send files to the agent for use in normal conversations or with the code interpreter. To send files, we attach them to the session state.  

#### **Defining Helper Functions**  

We will define helper functions to handle different file types, set the correct media type, call the agent, and process responses.  

The function below adds files to the session state. Each file will have the following attributes:  

- **name** (file name)  
- **sourceType** ('s3' or 'byte_content' for local files)  
- **mediaType** (supported formats: CSV, XLS, XLSX, YAML, JSON, DOC, DOCX, HTML, MD, TXT, PDF)  
- **data** (file content)  
- **useCase** (specifies how the model should use the file: `CHAT` or `CODE_INTERPRETER`)  

Refer to the [session state documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/Agents-test-code-interpretation.html) for more details.  

```python
# Returns session state containing the list of added files
def add_file_to_session_state(file_name, use_case='CODE_INTERPRETER', session_state=None):
    if use_case not in ["CHAT", "CODE_INTERPRETER"]:
        raise ValueError("Use case must be 'CHAT' or 'CODE_INTERPRETER'")
    if not session_state:
        session_state = {"files": []}

    type = file_name.split(".")[-1].upper()
    name = file_name.split("/")[-1]

    if type == "CSV":
        media_type = "text/csv" 
    elif type in ["XLS", "XLSX"]:
        media_type = "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
    else:
        media_type = "text/plain"

    named_file = {
        "name": name,
        "source": {
            "sourceType": "BYTE_CONTENT", 
            "byteContent": {
                "mediaType": media_type,
                "data": open(file_name, "rb").read()
            }
        },
        "useCase": use_case
    }
    session_state['files'].append(named_file)

    return session_state
```  

---

### **Sending Files for Normal Chat (No Code Execution Required)**  

In this case, we send a local CSV file and ask the model to interpret the data in the file. When adding the file to the session state, we set `useCase='CHAT'` instead of `'CODE_INTERPRETER'`. Additionally, we enable `show_code_use=True` to check whether the model uses the code interpreter. The results show that the model does not use the code interpreter but instead evaluates the information based on its natural language understanding.  

First, we check the file contents. The file contains historical stock price data.  

```python
import base64 

# Encode the CSV file as base64
with open(stock_file, "rb") as file_name:
    data = file_name.read()
    encoded_file = data  # base64.b64encode(data)

# Display the first 100 characters of the encoded file
encoded_file[0:100]
```  

![base-64-encoded](/images/4-invoking-agent/4.2-sending-files-to-agent/image-2.png)

Next, we invoke the agent to analyze the data in the file and provide insights. The agent identifies the contents of the file (containing simulated stock price data for 'FAKECO') and provides information about the type of data and the date range. Notice that the agent does not need to use the code interpreter to accomplish this task.  

```python
# Invoke the agent and process the response stream
query = "What is the data in this file?"

sessionState = add_file_to_session_state(stock_file, 'CHAT')

invoke_agent_helper(query, session_id, agent_id, agent_alias_id, enable_trace=False, session_state=sessionState,
                    memory_id=memory_id, show_code_use=True)
```

![normal-chat](/images/4-invoking-agent/4.2-sending-files-to-agent/image.png)  

---

### **Sending Files for Code Interpreter Execution**  

Now that we know the file contains stock market data, we can ask financial-related questions about it. This will require the model to invoke the **Code Interpreter**.  

Here, we reinitialize the session data and specify **use case** as `'CODE_INTERPRETER'`.  

```python
# Invoke the Agent and process the response
query = "Based on the attached stock price data, what is the total percentage growth of the closing price over the entire time series? What were the prices on the first and last day?"

sessionState = add_file_to_session_state(stock_file, 'CODE_INTERPRETER')

invoke_agent_helper(query, session_id, agent_id, agent_alias_id, enable_trace=False, session_state=sessionState,
                    memory_id=memory_id, show_code_use=True)
```

![invoked-code-chat](/images/4-invoking-agent/4.2-sending-files-to-agent/image-1.png)  
