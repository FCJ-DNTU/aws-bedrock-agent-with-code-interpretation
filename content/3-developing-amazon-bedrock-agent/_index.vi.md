---
title: "Triển khai Amazon Bedrock Agent"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

#### **1. Tạo IAM Policies**

Trước khi triển khai Amazon Bedrock Agent, chúng ta cần tạo các chính sách IAM (IAM Policies) để cấp quyền cho agent thực hiện gọi mô hình Bedrock.  

**Bước 1: Tạo IAM Policies**  

Chính sách IAM dưới đây cho phép agent gọi mô hình Bedrock cụ thể theo khu vực (region).  

```python
import json

# Tạo chính sách IAM cho Agent
bedrock_agent_bedrock_allow_policy_statement = {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AmazonBedrockAgentBedrockFoundationModelPolicy",
            "Effect": "Allow",
            "Action": "bedrock:InvokeModel",
            "Resource": [
                f"arn:aws:bedrock:{region}::foundation-model/{agent_foundation_model}"
            ]
        }
    ]
}

bedrock_policy_json = json.dumps(bedrock_agent_bedrock_allow_policy_statement)

agent_bedrock_policy = iam_client.create_policy(
    PolicyName=agent_bedrock_allow_policy_name,
    PolicyDocument=bedrock_policy_json
)
```

---

#### **2. Tạo IAM Role**

IAM Role sẽ được gán chính sách vừa tạo và cấp quyền cho Amazon Bedrock sử dụng agent.  

**Bước 2: Tạo IAM Role và gán quyền**  

```python
import time

# Định nghĩa chính sách Assume Role
assume_role_policy_document = {
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Principal": {
            "Service": "bedrock.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
    }]
}

assume_role_policy_document_json = json.dumps(assume_role_policy_document)

# Tạo IAM Role
agent_role = iam_client.create_role(
    RoleName=agent_role_name,
    AssumeRolePolicyDocument=assume_role_policy_document_json
)

# Chờ để đảm bảo vai trò được tạo trước khi gán chính sách
time.sleep(10)

# Gán chính sách cho IAM Role
iam_client.attach_role_policy(
    RoleName=agent_role_name,
    PolicyArn=agent_bedrock_policy['Policy']['Arn']
)
```

![iam-policies-and-role](/images/3-developing-amazon-bedrock-agent/image.png)

Kết quả hiển thị tại Amazon Console Management:

![result](/images/3-developing-amazon-bedrock-agent/image-1.png)

---

#### **3. Tạo Bedrock Agent**

Sau khi tạo IAM Role, chúng ta có thể sử dụng `Bedrock Agent Client` để khởi tạo agent.  

**Bước 3: Tạo Bedrock Agent**  

```python
# Tạo Bedrock Agent
response = bedrock_agent_client.create_agent(
    agentName=agent_name,
    agentResourceRoleArn=agent_role['Role']['Arn'],
    description=agent_description,
    idleSessionTTLInSeconds=1800,
    foundationModel=agent_foundation_model,
    instruction=agent_instruction
)
response

# Lưu ID của agent để sử dụng ở các bước tiếp theo
agent_id = response['agent']['agentId']
agent_id
```

![create-agent](/images/3-developing-amazon-bedrock-agent/image-2.png)

Kết quả hiển thị tại Amazon Management Console:

![result](/images/3-developing-amazon-bedrock-agent/image-3.png)

---

#### **4. Tạo Action Group cho Agent**

Trong Amazon Bedrock, **Action Groups** định nghĩa các công cụ mà agent có thể sử dụng. Ở đây, chúng ta sẽ tạo một Action Group cho **Code Interpreter**, cho phép agent chạy mã Python để xử lý dữ liệu và tạo file.  

**Bước 4: Tạo Agent Action Group (Code Interpreter)**  

```python
# Chờ để đảm bảo agent được khởi tạo
time.sleep(30)

# Cấu hình Action Group cho Code Interpreter
agent_action_group_response = bedrock_agent_client.create_agent_action_group(
    agentId=agent_id,       
    agentVersion='DRAFT',
    actionGroupName='code-interpreter',
    parentActionGroupSignature='AMAZON.CodeInterpreter',
    actionGroupState='ENABLED'
)

agent_action_group_response
```

Kết quả tạo Agent Action Group thành công:

![action-group-created](/images/3-developing-amazon-bedrock-agent/image-4.png)
