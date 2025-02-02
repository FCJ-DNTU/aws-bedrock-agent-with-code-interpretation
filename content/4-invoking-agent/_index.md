---
title: "Invoking The Amazon Bedrock Agent"
weight: 4
chapter: false
pre: " <b> 4. </b> "
---

#### **Invoking the Amazon Bedrock Agent**  

We will define a helper function to invoke the agent and parse its responses, then invoke it to see how it utilizes code invocation.  

#### **Preparing the DRAFT Version of the Agent**  

First, we will create a **DRAFT** version of the agent for internal testing.  

```python
response = bedrock_agent_client.prepare_agent(
    agentId=agent_id
)
print(response)
```  

```python
# Pause to make sure agent is prepared
time.sleep(30)

# Extract agentAliasId from response
agent_alias_id = "TSTALIASID" # This ID is for testing purpose
```  

![create-draft-version](/images/4-invoking-agent/image.png)  

---  

#### **Helper Function to Invoke the Agent**  

This helper function can invoke the agent and parse the stream of returned responses.  

The `invoke_agent_helper` function allows a user to send a `query` to the agent with a `session_id`. A session defines an exchange between the user and the agent. The agent can remember the entire context within a session. Once the user ends the session, this context is erased.  

Users can:  

- Enable or disable tracing using the `enable_trace` boolean.  
- Pass session state as a dictionary via `session_state`.  

If a new `session_id` is provided, the agent will create a **new conversation** without any prior context. If the same `session_id` is reused, the agent will retain the context from that session.  

If `enable_trace = True`, each response from the agent will include a **trace**, detailing the steps the agent orchestrated to generate the final response (*reasoning via Chain of Thoughts prompting*).  

The `memory_id` parameter handles memory capabilities. When a session ends, it summarizes the content into a new session ID as part of `memory_id`.  

#### **Session State**  

Users can pass session context using `session_state`, with the following options:  

- **`sessionAttributes`**: Attributes persist throughout a session between the user and the agent. All `invokeAgent` calls with the same `session_id` belong to the same session and share `sessionAttributes`, as long as the session has not expired or ended. `sessionAttributes` are only available in the Lambda function and **are not** included in the agent prompt.  
- **`promptSessionAttributes`**: Attributes persist only for a **single** `invokeAgent` call. Prompt attributes are added to the prompt and the Lambda function.  
- **`invocationId`**: The ID returned by the agent in `ReturnControlPayload` when a return control invocation occurs.  
- **`returnControlInvocationResults`**: Results obtained from an invocation action external to Amazon Bedrock Agents.  

If `show_code_use` is set to `True`, the helper function will print a message when the code interpreter is invoked. It also **automatically enables tracing** to accomplish this.  

We will use the test `agent_alias_id` as `"TSTALIASID"`, a default value for testing the agent during development. When deploying the agent, you can create a new version and obtain a new agent alias ID.  

```python
def invoke_agent_helper(
    query, session_id, agent_id, alias_id, enable_trace=False, memory_id=None, session_state=None, end_session=False, show_code_use=False
):
    
    if not session_state:
        session_state = {}

    # Invoke agent API
    agent_response = bedrock_agent_runtime_client.invoke_agent(
        inputText=query,
        agentId=agent_id,
        agentAliasId=alias_id,
        sessionId=session_id,
        enableTrace=(enable_trace | show_code_use), # Force tracing on if showing code use
        endSession=end_session,
        memoryId=memory_id,
        sessionState=session_state
    )
    return process_response(agent_response, enable_trace=enable_trace, show_code_use=show_code_use)
```  

---  

#### **Helper Function to Process Agent Responses**  

```python
def process_response(resp, enable_trace: bool = False, show_code_use: bool = False):
    if enable_trace:
        logger.info(pprint.pprint(resp))

    event_stream = resp['completion']
    try:
        for event in event_stream:
            if 'chunk' in event:
                data = event['chunk']['bytes']
                if enable_trace:
                    logger.info(f"Final answer ->\n{data.decode('utf8')}")
                agent_answer = data.decode('utf8')
                return agent_answer
            elif 'trace' in event:
                if 'codeInterpreterInvocationInput' in json.dumps(event['trace']):
                    if show_code_use:
                        print("Invoked code interpreter")
                if enable_trace:
                    logger.info(json.dumps(event['trace'], indent=2))
            else:
                raise Exception("Unexpected event.", event)
    except Exception as e:
        raise Exception("Unexpected event.", e)
```  
