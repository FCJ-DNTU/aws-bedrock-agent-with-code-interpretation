---
title: "Invoking the Code Interpreter"
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

### **Invoking the Code Interpreter**  

We request the agent to generate a random string, which forces it to use the code interpreter. By using the `show_code_use` flag, we can see that the agent invokes the code interpreter to evaluate the Python code it generates.  

```python
## create a random id for session initiator id
session_id:str = str(uuid.uuid1())
memory_id:str = 'TST_MEM_ID'
query = "Please generate a 10 character long string of random characters"
invoke_agent_helper(query, session_id, agent_id, agent_alias_id, enable_trace=False, memory_id=memory_id, show_code_use=True)
```  

![generate-string](/images/4-invoking-agent/4.1-invoking-code-interpreter/image.png)  

Similarly, the agent will write Python code and invoke the code interpreter to solve a mathematical problem.  

```python
query = "What is 75 * sin(.75)?"
invoke_agent_helper(query, session_id, agent_id, agent_alias_id, enable_trace=False, memory_id=memory_id,
                    show_code_use=True)
```  
![complex-math](/images/4-invoking-agent/4.1-invoking-code-interpreter/image-1.png)  

On the other hand, actions where the model does not need to execute code will not invoke the code interpreter.  

```python
query = "thank you!"
invoke_agent_helper(query, session_id, agent_id, agent_alias_id, enable_trace=False, memory_id=memory_id, show_code_use=True)
```  

![non-invoke-code](/images/4-invoking-agent/4.1-invoking-code-interpreter/image-2.png)  