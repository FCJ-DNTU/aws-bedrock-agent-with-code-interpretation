---
title: "Thực thi Amazon Bedrock Agent"
weight: 4
chapter: false
pre: " <b> 4. </b> "
---

#### **Thực thi Agent**  

Chúng ta sẽ định nghĩa một helper function để invoke agent và parse responses của nó, sau đó invoke nó để xem cách nó sử dụng code invocation.  

#### **Chuẩn bị DRAFT Version của Agent**  

Trước tiên, chúng ta sẽ tạo một DRAFT version của agent để phục vụ internal testing.  

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

#### **Helper function xử lý invoke agent**  

Helper function này có thể invoke agent và parse stream của returned responses.  

Function `invoke_agent_helper` cho phép user gửi một `query` đến agent với một `session_id`. Một session định nghĩa một lượt trao đổi qua lại giữa user và agent. Agent có thể ghi nhớ toàn bộ context trong một session. Khi user kết thúc session, context này sẽ bị xóa.  

User có thể:  

- Bật hoặc tắt trace bằng cách sử dụng `enable_trace` boolean.  
- Truyền session state như một dictionary qua `session_state`.  

Nếu một `session_id` mới được cung cấp, agent sẽ tạo một new conversation mà không có context trước đó. Nếu cùng `session_id` được reuse, conversation context liên quan đến session đó sẽ được agent ghi nhớ.  

Nếu `enable_trace = True`, mỗi response từ agent sẽ đi kèm với một **trace** detailing step mà agent đã orchestrate để đưa ra final response (*reasoning via Chain of Thoughts prompting*).  

Parameter `memory_id` được sử dụng để handle memory capabilities. Khi một session kết thúc, nó sẽ summarize content vào một new session id như một phần của `memory_id`.  

##### **Session State**  

User có thể pass session context bằng cách sử dụng `session_state`, với các options sau:  

- **`sessionAttributes`**: attributes tồn tại trong suốt một session giữa user và agent. Tất cả invokeAgent calls với cùng `session_id` thuộc về cùng một session và sẽ share `sessionAttributes` với nhau, miễn là session chưa hết hạn hoặc user chưa end session. `sessionAttributes` chỉ khả dụng trong Lambda function và **không** được thêm vào agent prompt.  
- **`promptSessionAttributes`**: attributes tồn tại trong một single invokeAgent call. Prompt attributes được thêm vào prompt và Lambda function.  
- **`invocationId`**: ID được agent trả về trong `ReturnControlPayload` khi có return control invocation.  
- **`returnControlInvocationResults`**: results thu được từ invocation action bên ngoài Amazon Bedrock Agents.  

Nếu `show_code_use` được pass là `True`, helper sẽ print một message khi code interpreter được invoke. Nó cũng tự động bật tracing để thực hiện điều này.  

Chúng ta sẽ sử dụng test `agent_alias_id` là `"TSTALIASID"`, một default value để test agent trong quá trình development. Khi deploy agent, bạn có thể tạo một new version và lấy một agent alias id mới.  

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

#### **Helper function xử lý dữ liệu trả về của Agent**  

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