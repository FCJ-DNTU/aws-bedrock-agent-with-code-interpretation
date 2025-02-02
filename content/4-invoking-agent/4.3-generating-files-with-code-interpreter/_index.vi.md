---
title: "Tạo File với Trình Thông Dịch Mã"
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

#### **Tạo File với Trình Thông Dịch Mã**

Các **Amazon Bedrock Agent** có khả năng tạo và trả về file cho người dùng. Chúng có thể tạo file bằng cách sử dụng trí thông minh tích hợp của mô hình để tạo các loại file như `.CSV`, hoặc bằng cách viết mã sử dụng **trình thông dịch mã (Code Interpreter)** để tạo các file nhị phân, chẳng hạn như biểu đồ dữ liệu. Các file này được trả về thông qua **dòng phản hồi (response stream)**.

---

#### **Dòng Phản Hồi (Response Stream) của Agent Bedrock**

Dòng phản hồi bao gồm các sự kiện được định dạng theo **JSON**, truyền tải thông tin chi tiết về quá trình suy nghĩ và hành động của Agent khi thực hiện theo [mô hình ReAct](https://aws.amazon.com/blogs/aws/preview-enable-foundation-models-to-complete-tasks-with-agents-for-amazon-bedrock/) (Reasoning and Acting). Dưới đây là một số trường quan trọng:

- **'files'**: Chứa các file được tạo ra tự động bởi mô hình LLM của Agent.
- **'trace'**: Chứa thông tin về quá trình suy nghĩ và các bước thực hiện của Agent. Có nhiều loại sự kiện trace:
    - **'modelInvocationInput'**: Chứa các thông tin đầu vào khi gọi mô hình.
    - **'rationale'**: Chứa lý do (rationale) mà Agent đưa ra để thực hiện hành động.
    - **'invocationInput'**: Chứa chi tiết về các tham số khi gọi nhóm hành động.
        - **'codeInterpreterInvocationInput'**: Chứa mã nguồn mà mô hình tạo ra và chuyển đến trình thông dịch mã.
    - **'observation'**: Chứa các quan sát quan trọng, bao gồm:
        - **'codeInterpreterInvocationOutput'**: Chứa kết quả cụ thể từ việc thông dịch mã:
            - **'executionOutput'**: Kết quả thực thi mã.
            - **'executionError'**: Lỗi nếu có trong quá trình thực thi mã.
            - **'files'**: Các file được tạo ra từ việc thông dịch mã.
        - **'finalResponse'**: Phản hồi cuối cùng của Agent.

Chúng ta sẽ định nghĩa lại **helper function** để thu thập các file từ dòng phản hồi. Sau đó, sử dụng nó để lưu các file được tạo bởi Agent, dù thông qua trí thông minh của mô hình hay thông qua trình thông dịch mã, và trả về cho người dùng.

---

#### **Định Nghĩa Lại Helper Function**

Chúng ta sẽ định nghĩa lại hàm `process_response` để thu thập và hiển thị chi tiết phong phú hơn từ dòng phản hồi. Để hỗ trợ hiển thị trong notebook, chúng ta sử dụng thư viện `IPython.display` để hiển thị các file trả về một cách trực quan, chẳng hạn như nhúng hình ảnh hoặc hiển thị mã nguồn.

```python
from IPython.display import display, Markdown
import matplotlib.pyplot as plt
import matplotlib./images/4-invoking-agent/4.3-generating-files-with-code-interpreter/image as mpimg
```

```python
def process_response(resp, enable_trace:bool=True, show_code_use:bool=False):
    if resp['ResponseMetadata']['HTTPStatusCode'] != 200:
        print(f"API Response was not 200: {resp}")

    event_stream = resp['completion']
    for event in event_stream:
        if 'files' in event.keys():
            files_event = event['files']
            display(Markdown("### Files"))
            files_list = files_event['files']
            for this_file in files_list:
                print(f"{this_file['name']} ({this_file['type']})")
                file_bytes = this_file['bytes']

                # Lưu byte vào file
                file_name = os.path.join('output', this_file['name'])
                with open(file_name, 'wb') as f:
                    f.write(file_bytes)
                if this_file['type'] == '/images/4-invoking-agent/4.3-generating-files-with-code-interpreter/image/png' or this_file['type'] == '/images/4-invoking-agent/4.3-generating-files-with-code-interpreter/image/jpeg':
                    img = mpimg.imread(file_name)
                    plt.imshow(img)
                    plt.show()

        if 'trace' in event.keys() and enable_trace:
            trace_event = event.get('trace')['trace']['orchestrationTrace']

            if 'modelInvocationInput' in trace_event.keys():
                pass

            if 'rationale' in trace_event.keys():
                rationale = trace_event['rationale']['text']
                display(Markdown(f"### Rationale\n{rationale}"))

            if 'invocationInput' in trace_event.keys() and show_code_use:
                inv_input = trace_event['invocationInput']
                if 'codeInterpreterInvocationInput' in inv_input:
                    gen_code = inv_input['codeInterpreterInvocationInput']['code']
                    code = f"```python\n{gen_code}\n```"
                    display(Markdown(f"### Generated code\n{code}"))

            if 'observation' in trace_event.keys():
                obs = trace_event['observation']
                if 'codeInterpreterInvocationOutput' in obs:
                    if 'executionOutput' in obs['codeInterpreterInvocationOutput'].keys() and show_code_use:
                        raw_output = obs['codeInterpreterInvocationOutput']['executionOutput']
                        output = f"```\n{raw_output}\n```"
                        display(Markdown(f"### Output from code execution\n{output}"))

                    if 'executionError' in obs['codeInterpreterInvocationOutput'].keys():
                        display(Markdown(f"### Error from code execution\n{obs['codeInterpreterInvocationOutput']['executionError']}"))

                    if 'files' in obs['codeInterpreterInvocationOutput'].keys():
                        display(Markdown("### Files generated\n"))
                        display(Markdown(f"{obs['codeInterpreterInvocationOutput']['files']}"))

                if 'finalResponse' in obs:                    
                    final_resp = obs['finalResponse']['text']
                    display(Markdown(f"### Final response\n{final_resp}"))
                    return final_resp
```

---

#### **Tạo File sử dụng Code Generation**

Chúng ta sẽ yêu cầu Agent tạo một file, và file này sẽ được trả về thông qua dòng phản hồi.

```python
query = """
Please generate a list of the 10 greatest books of all time. Return it as a CSV file. Always return the file, even if you have provided it before.
"""

invoke_agent_helper(query, session_id, agent_id, agent_alias_id, enable_trace=False, session_state=sessionState,
                    memory_id=memory_id, show_code_use=True)
```

![generates-files](/images/4-invoking-agent/4.3-generating-files-with-code-interpreter/image.png)

---

#### **Tạo Biểu Đồ bằng Cách Thông Dịch Mã**

Chúng ta sẽ gửi cùng một file dữ liệu giá cổ phiếu như trước, nhưng lần này yêu cầu Agent tạo một biểu đồ. Agent sẽ viết mã Python để tạo biểu đồ, và trình phân tích dòng phản hồi sẽ hiển thị biểu đồ trực tiếp trong notebook.

```python
# Gọi Agent và xử lý dòng phản hồi
query = "Given the attached price data file, please make me a chart with moving average in red and actual data in blue"

sessionState=add_file_to_session_state(stock_file, 'CODE_INTERPRETER')

invoke_agent_helper(query, session_id, agent_id, agent_alias_id, enable_trace=True, session_state=sessionState,
                    memory_id=memory_id, show_code_use=True)
```

![gen-charts](/images/4-invoking-agent/4.3-generating-files-with-code-interpreter/image-1.png)

---

#### **Tạo Dữ Liệu Tổng Hợp và Phân Tích**

Chúng ta sẽ yêu cầu Agent tạo hai file CSV và thực hiện phân tích dữ liệu.

```python
# Gọi Agent và xử lý dòng phản hồi
query = """
generate two csv files for me. 
one called SALES, with 3 columns: COMPANY_ID, COMPANY_NAME, and SALES_2024. 
the other called DETAILS, with 3 columns: COMPANY_ID, COMPANY_STATE_CODE. 
follow these rules:
1) each file should contain 200 companies, and share the same company ID’s. 
2) use human readable english words in the names (not random strings of letters and digits), 
3) use ID’s of the form: C00001. 
4) Only use states that are generally considered to be near the east coast or near the west coast. 
5) Make the revenue from each eastern company range from 0 to $700,000, 
6) Make revenue from each western company range from $500,000 up to $2,000,000. 
When done, test to be sure you have followed each of the above rules, 
and produce a chart comparing sales per company in the two regions using box plots.
"""

invoke_agent_helper(query, session_id, agent_id, agent_alias_id, enable_trace=True, session_state=sessionState,
                    memory_id=memory_id, show_code_use=True)
```

![analytics](/images/4-invoking-agent/4.3-generating-files-with-code-interpreter/image-2.png)
