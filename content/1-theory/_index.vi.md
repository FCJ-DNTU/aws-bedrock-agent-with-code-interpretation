---
title: "Giới thiệu Amazon Bedrock Agent"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

#### **Tổng quan về Amazon Bedrock**  
Amazon Bedrock là một dịch vụ được quản lý toàn diện, giúp các doanh nghiệp truy cập và sử dụng các **Foundation Models (FMs)** hiệu suất cao một cách dễ dàng và hiệu quả.  

---

#### **Giới thiệu về Amazon Bedrock Agent** 

Amazon Bedrock Agent giúp các ứng dụng **AI tạo sinh (Generative AI)** thực hiện các tác vụ nhiều bước bằng cách kết nối với **hệ thống, API và nguồn dữ liệu** của doanh nghiệp.  

Agent sử dụng khả năng **lập luận của Foundation Models (FMs)** kết hợp với API và dữ liệu để **phân tích yêu cầu của người dùng, truy xuất thông tin cần thiết và hoàn thành tác vụ một cách hiệu quả**.  

{{% notice note %}}
Trong workshop này, chúng ta sẽ sử dụng **Foundation Model [Claude 3 Sonnet](https://aws.amazon.com/blogs/aws/anthropics-claude-3-sonnet-foundation-model-is-now-available-in-amazon-bedrock/#:~:text=Introduction%20of%20Anthropic%E2%80%99s%20Claude%203%20Sonnet)**.
{{% /notice %}}


---

#### **Phân tích kiến trúc Amazon Bedrock Agent với Code Interpreter**  

Sơ đồ dưới đây minh họa cách **Amazon Bedrock Agent** kết hợp với **Code Interpreter** để xử lý các tác vụ phân tích dữ liệu, trực quan hóa và tính toán phức tạp.  

![architecture](/images/architecture-workshop-04-bedrock-agent.png)

**Luồng xử lý chính:**  
1. **Người dùng (Customer)** gửi câu hỏi hoặc tải lên tệp dữ liệu.  
2. **Analytics Agent** trong Amazon Bedrock tiếp nhận câu hỏi và tạo phản hồi dựa trên mô hình ngôn ngữ lớn (LLM). Nếu cần thực thi mã nguồn, agent sẽ chuyển tiếp nhiệm vụ đến **Code Interpreter**.  
3. **Code Interpreter** nhận tệp đầu vào hoặc yêu cầu xử lý, thực hiện tính toán, phân tích, hoặc trực quan hóa dữ liệu.  
4. Sau khi hoàn thành, **Code Interpreter** tạo ra các tệp đầu ra và gửi kết quả trở lại cho **Analytics Agent**.  
5. **Analytics Agent** tổng hợp thông tin và phản hồi lại cho người dùng.  

**Luồng xử lý tệp tin:**  
- Người dùng có thể **tải lên tệp dữ liệu** để xử lý.  
- **Analytics Agent** chia sẻ các tệp này với **Code Interpreter**.  
- **Code Interpreter** thực thi mã trên tệp đầu vào và tạo ra **tệp kết quả (output files)**.  
- Các tệp kết quả này sẽ được chia sẻ với người dùng để tải xuống hoặc tiếp tục sử dụng.  

---

#### **Các tính năng nổi bật và ứng dụng của Amazon Bedrock Agent**  

**1. Multi-agent collaboration – Hợp tác giữa nhiều Agent**  
Tính năng này giúp các nhà phát triển dễ dàng **tạo, triển khai và quản lý nhiều Agent chuyên biệt**, cho phép chúng làm việc cùng nhau để xử lý các quy trình kinh doanh phức tạp.  

![multi-agent](/images/1-theory/image.png)

**2. Retrieval-Augmented Generation (RAG) – Truy xuất dữ liệu để tăng cường khả năng sinh văn bản**  
Agent có thể **kết nối với các nguồn dữ liệu của doanh nghiệp** để cung cấp câu trả lời chính xác cho người dùng.  

🔹 **Ví dụ:** Người dùng hỏi: *"Lịch sử Python bắt đầu như thế nào?"*  
- **Bước 1:** AI tìm kiếm thông tin từ một nguồn bên ngoài (ví dụ: Wikipedia).  
- **Bước 2:** AI tổng hợp và tạo câu trả lời dựa trên dữ liệu tìm được.  
- **Kết quả:** *"Python được tạo ra bởi Guido van Rossum vào năm 1991 nhằm giúp lập trình dễ đọc hơn và hiệu quả hơn."*  

👉 **Lợi ích:** Giúp AI trả lời những câu hỏi mà dữ liệu không có sẵn trong mô hình, nhưng có thể lấy từ nguồn bên ngoài.  

![rag](/images/1-theory/image-1.png)

**3. Orchestrate and execute multistep tasks – Điều phối và thực hiện tác vụ nhiều bước**  
Người dùng có thể chọn một **mô hình AI** và viết hướng dẫn đơn giản, chẳng hạn:  
*"Bạn là một Agent quản lý kho hàng, giúp xác định sản phẩm có sẵn trong hệ thống."*  

Agent sẽ **phân tích nhiệm vụ, chia nhỏ thành các bước hợp lý** và tự động gọi các API cần thiết để hoàn thành yêu cầu.  

**4. Memory retention across interactions – Ghi nhớ ngữ cảnh trong các lần tương tác**  
Amazon Bedrock Agent có thể **ghi nhớ thông tin từ các lần tương tác trước**, giúp tạo ra trải nghiệm liền mạch và cá nhân hóa hơn.  

Ví dụ, nếu người dùng đã hỏi về **tồn kho sản phẩm X** trước đó, Agent có thể nhớ thông tin này và không yêu cầu nhập lại trong các truy vấn tiếp theo.  

![memory](/images/1-theory/image-2.png)

**5. Code interpretation – Diễn giải và thực thi mã nguồn**  
Agent có thể **tạo và thực thi mã nguồn trong môi trường an toàn**, giúp tự động hóa các truy vấn phân tích phức tạp.  

![code-interpretation](/images/1-theory/image-3.png)

👉 **Ứng dụng:**  
✅ **Phân tích dữ liệu**  
✅ **Trực quan hóa dữ liệu** (vẽ biểu đồ, đồ thị)  
✅ **Giải quyết bài toán toán học phức tạp**  

**6. Prompt engineering – Tạo và tối ưu hóa lời nhắc (prompt)**  
Agent có thể **tự động tạo các prompt template** từ **hướng dẫn của người dùng, nhóm hành động và kho kiến thức**.  
Người dùng cũng có thể **tinh chỉnh đầu vào, kế hoạch điều phối và phản hồi của mô hình** để cải thiện kết quả.  

---

#### **Cân nhắc về chi phí cho Amazon Bedrock Agent**

Trong workshop này, chúng ta sử dụng model **Claude 3 Sonnet**, với chi phí cụ thể như sau:  

| **Anthropic models** | **Giá cho 1,000 input tokens** | **Giá cho 1,000 output tokens** | **Giá cho 1,000 input tokens (batch)** | **Giá cho 1,000 output tokens (batch)** | **Giá cho 1,000 input tokens (cache write)** | **Giá cho 1,000 input tokens (cache read)** |
|----------------------|--------------------------------|----------------------------------|-----------------------------------------|------------------------------------------|--------------------------------------------|--------------------------------------------|
| **Claude 3 Sonnet**  | $0.003                         | $0.015                           | $0.0015                                  | $0.0075                                   | N/A                                        | N/A                                        |


{{% notice info %}}
📌 **Lưu ý:** Chi phí có thể thay đổi tùy theo mức độ sử dụng và phương thức triển khai. Hãy tham khảo [tài liệu chính thức của AWS](https://aws.amazon.com/bedrock/pricing/) để biết thêm thông tin chi tiết.  
{{% /notice %}}