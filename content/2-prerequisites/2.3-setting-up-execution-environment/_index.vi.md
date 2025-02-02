---
title: "Thiết lập môi trường thực thi code"
weight: 3
chapter: false
pre: " <b> 2.3 </b> "
---


#### **1. Giới thiệu**  
Để chạy và thử nghiệm Amazon Bedrock Agent, chúng ta cần thiết lập môi trường thực thi code. Có thể sử dụng **Google Colab** hoặc chạy trên **local**. Trong hướng dẫn này, chúng ta sẽ sử dụng **Jupyter Notebook trên Google Colab** làm môi trường lập trình.  

---

#### **2. Khởi tạo Jupyter Notebook trên Google Colab**  

**Bước 1: Tạo Notebook mới trên Google Colab**  
- Truy cập [Google Colab Console](https://colab.research.google.com/)  
- Chọn **"New Notebook"** để tạo một tệp Notebook mới.  

![google-colab](/images/2-prerequisites/2.3-setting-up-execution-environment/image-1.png)

- Tạo mới thành công

![new-notebook-successfully](/images/2-prerequisites/2.3-setting-up-execution-environment/image.png)

---

#### **3. Cài đặt & Cấu hình AWS CLI**  

**Bước 2: Cài đặt AWS CLI V2**  
Chạy lệnh sau để cài đặt AWS CLI:  

```python
!pip install awscli
```

**Bước 3: Cấu hình AWS CLI**  
Khi chạy lệnh `aws configure`, bạn cần nhập thông tin sau:  
- **AWS Access Key ID** – Access Key ID của IAM User  
- **AWS Secret Access Key** – Secret Access Key của IAM User  
- **Default region name** – Chọn khu vực thực thi (`us-east-1` được hỗ trợ model Claude 3.0 Sonnet)  
- **Default output format** – Chọn `json`  

```python
!aws configure
```

![aws-configure](/images/2-prerequisites/2.3-setting-up-execution-environment/image-2.png)

---

#### **4. Cập nhật packages**  

**Bước 4: Cài đặt và nâng cấp thư viện AWS SDK**  
Trước khi bắt đầu, hãy cập nhật các thư viện cần thiết để đảm bảo có phiên bản mới nhất:  

```python
!python3 -m pip install --upgrade -q boto3
!python3 -m pip install --upgrade -q botocore
!python3 -m pip install --upgrade -q awscli
```

Sau đó, kiểm tra phiên bản của `boto3`, `botocore`, và `awscli`. Phiên bản của `boto3` cần lớn hơn hoặc bằng **1.34.139**.  

```python
import boto3
import botocore
import awscli

print("Boto3 Version:", boto3.__version__)
print("Botocore Version:", botocore.__version__)
print("AWS CLI Version:", awscli.__version__)
```

![install-packages](/images/2-prerequisites/2.3-setting-up-execution-environment/image-3.png)

---

#### **5. Thiết lập Logger & Import Thư Viện Hỗ Trợ**  

**Bước 5: Import các thư viện cần thiết**  

```python
import json
import time
from io import BytesIO
import uuid
import pprint
import logging
```

**Bước 6: Cấu hình Logger để theo dõi tiến trình**  

```python
logging.basicConfig(
    format='[%(asctime)s] p%(process)s {%(filename)s:%(lineno)d} %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)
```

---

#### **6. Tạo Kết Nối với AWS Services**  

**Bước 7: Khởi tạo Boto3 Clients**  
Chúng ta cần tạo các **boto3 clients** để kết nối với các dịch vụ AWS như **IAM, Lambda, Bedrock Agent**.  

```python
# Khởi tạo boto3 clients
sts_client = boto3.client('sts')
iam_client = boto3.client('iam')
lambda_client = boto3.client('lambda')
bedrock_agent_client = boto3.client('bedrock-agent')
bedrock_agent_runtime_client = boto3.client('bedrock-agent-runtime')
```

---

#### **7. Thiết Lập Biến Cấu Hình**  

**Bước 8: Lấy thông tin tài khoản AWS**  
Chúng ta sẽ lấy thông tin **Region** và **Account ID** để sử dụng trong quá trình triển khai.  

```python
# Lấy thông tin tài khoản AWS
session = boto3.session.Session()
region = session.region_name
account_id = sts_client.get_caller_identity()["Account"]
region, account_id
```

**Bước 9: Thiết lập các biến cấu hình**  

Trong bước này, chúng ta sẽ thiết lập các biến cấu hình quan trọng cho Amazon Bedrock Agent. Các biến này xác định thông tin về agent, quyền truy cập, mô hình nền tảng, và hướng dẫn hoạt động.  

```python
# Declare environment variables
suffix = f"{region}-{account_id}"
agent_name = "assistant-w-code-interpret"
agent_bedrock_allow_policy_name = f"{agent_name}-ba-{suffix}"
agent_role_name = f'AmazonBedrockExecutionRoleForAgents_{agent_name}'
agent_foundation_model = "anthropic.claude-3-sonnet-20240229-v1:0"
agent_description = "Trợ lý hỗ trợ viết code và thực thi mã Python để trả lời câu hỏi và tạo tài liệu."
agent_instruction = """
Bạn là một trợ lý hỗ trợ người dùng trả lời câu hỏi và tạo tài liệu.
Bạn có quyền truy cập trình biên dịch mã để thực thi code Python.
Khi gặp nhiệm vụ có thể giải quyết bằng mã Python, hãy viết code tương ứng,
chuyển nó đến trình biên dịch để thực thi, sau đó trả về kết quả cho người dùng.
"""
agent_alias_name = f"{agent_name}-alias"
```

#### **Giải thích các biến cấu hình**  

- **`agent_name`**: Tên định danh của Agent.  
- **`agent_bedrock_allow_policy_name`**: Tên chính sách IAM cho phép Agent truy cập mô hình Bedrock.  
- **`agent_role_name`**: Tên vai trò IAM được gán cho Agent.  
- **`agent_foundation_model`**: Mô hình nền tảng mà Agent sẽ sử dụng (Claude 3 Sonnet).  
- **`agent_description`**: Mô tả về chức năng của Agent.  
- **`agent_instruction`**: Hướng dẫn hoạt động cho Agent, giúp xác định cách nó xử lý và thực thi mã Python.  
- **`agent_alias_name`**: Bí danh (alias) của Agent để sử dụng trong các yêu cầu sau này.  

📌 **Lưu ý quan trọng**:  
- Biến **`agent_instruction`** đóng vai trò quan trọng trong việc hướng dẫn Agent cách thức hoạt động.  
- Agent này có thể tự động tạo, thực thi code Python và sử dụng kết quả để hỗ trợ người dùng.  

---

#### **Hoàn thành thiết lập môi trường thực thi code**  

Môi trường đã sẵn sàng! Bây giờ bạn có thể sử dụng Jupyter Notebook để triển khai **Amazon Bedrock Agent** một cách dễ dàng. 🎯