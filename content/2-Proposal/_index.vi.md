---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---


# SmartMenu - Hệ thống Menu Điện tử Thông minh tích hợp AI trên nền tảng AWS Serverless

### 1. Tóm tắt
SmartMenu là một hệ thống menu điện tử dành cho nhà hàng và quán ăn, được xây dựng trên nền tảng điện toán đám mây AWS theo mô hình Serverless. Hệ thống cho phép khách hàng truy cập menu trực tuyến thông qua thiết bị di động, xem thông tin món ăn, nhận hỗ trợ dịch ngôn ngữ và tư vấn món ăn bằng trí tuệ nhân tạo, đồng thời thực hiện đặt món nhanh chóng.
Bên cạnh đó, SmartMenu cung cấp giao diện quản trị giúp chủ nhà hàng quản lý thực đơn, theo dõi đơn hàng và cập nhật dữ liệu theo thời gian thực. Việc ứng dụng kiến trúc Serverless giúp hệ thống có khả năng mở rộng linh hoạt, giảm chi phí vận hành và không yêu cầu quản lý máy chủ.

### 2. Mục tiêu
**Mục tiêu tổng quát:**
Xây dựng một hệ thống menu điện tử thông minh trên AWS nhằm nâng cao trải nghiệm khách hàng và hỗ trợ nhà hàng quản lý hiệu quả hơn.

**Mục tiêu cụ thể:**
- Xây dựng hệ thống menu điện tử hoạt động trên nền tảng AWS Serverless.
- Hỗ trợ xem menu và đặt món trực tuyến.
- Hỗ trợ dịch menu sang nhiều ngôn ngữ.
- Tư vấn món ăn bằng AI.
- Quản lý menu và đơn hàng theo thời gian thực.
- Tối ưu chi phí vận hành.
- Đảm bảo khả năng mở rộng và tính sẵn sàng cao.

### 3. Vấn đề cần giải quyết
Hiện nay nhiều nhà hàng vẫn sử dụng menu giấy và quy trình phục vụ truyền thống, dẫn đến các vấn đề:
- Khách hàng phải chờ nhân viên để gọi món.
- Khó cập nhật thực đơn khi thay đổi giá hoặc thêm món mới.
- Tốn chi phí in ấn menu.
- Khách quốc tế gặp rào cản ngôn ngữ.
- Khách có dị ứng thực phẩm khó tìm thông tin về thành phần món ăn.
- Chủ nhà hàng khó quản lý đơn hàng trong giờ cao điểm.
- Hệ thống truyền thống khó mở rộng khi số lượng khách tăng.

Do đó cần xây dựng một hệ thống số hóa toàn bộ quy trình xem menu và đặt món.

### 4. Kiến trúc hệ thống

{{% notice info %}}
Tạm thời dự án sẽ không triển khai DynamoDB và Bedrock AI do trong lúc thực hiện gặp một vài vấn đề trục trặc nên sẽ thay bằng MongoDB và API Gemini.
{{% /notice %}}

**Công nghệ sử dụng:**
- **Frontend Web:** ReactJS + Vite, xây dựng giao diện cho khách hàng (Table Menu Client) và quản trị viên (Manager Client).
- **Backend:** AWS Lambda (Node.js), triển khai theo mô hình Serverless để xử lý các nghiệp vụ của hệ thống.
- **AI:** Amazon Bedrock, hỗ trợ dịch đa ngôn ngữ và tư vấn món ăn.
- **Database:** Amazon DynamoDB, lưu trữ dữ liệu menu và đơn hàng.

**Các dịch vụ AWS chính:**
- **Amazon API Gateway:** Tiếp nhận và định tuyến các yêu cầu từ ứng dụng đến các dịch vụ backend.
- **AWS Lambda:** Xử lý các nghiệp vụ như quản lý thực đơn, đặt món và dịch ngôn ngữ.
- **Amazon DynamoDB:** Lưu trữ dữ liệu menu, đơn hàng và các thông tin của hệ thống.
- **Amazon S3:** Lưu trữ giao diện web và các tài nguyên tĩnh như hình ảnh, CSS và JavaScript.
- **Amazon CloudFront:** Phân phối nội dung từ Amazon S3 với độ trễ thấp và tăng tốc độ truy cập.
- **Amazon Bedrock:** Cung cấp các mô hình AI phục vụ chức năng dịch và tư vấn món ăn.
- **AWS IAM:** Quản lý người dùng, vai trò và phân quyền truy cập giữa các dịch vụ AWS.


![SmartMenu Architecture](/images/2-Proposal/smartmenu_architecture.png)

### 5. Triển khai kỹ thuật
#### 5.1. Các giai đoạn triển khai
- **Giai đoạn 1:** Tìm hiểu các dịch vụ AWS nền tảng thông qua CloudJourney Labs, bao gồm IAM, VPC, EC2, RDS, S3, Lambda, API Gateway, CloudFormation và DynamoDB.
- **Giai đoạn 2:** Phân tích yêu cầu của hệ thống SmartMenu, thiết kế kiến trúc Serverless, xây dựng sơ đồ Solution Architecture và thiết lập môi trường phát triển.
- **Giai đoạn 3:** Phát triển các chức năng chính của hệ thống, bao gồm giao diện khách hàng và quản trị viên, các dịch vụ quản lý thực đơn, quản lý đơn hàng, dịch ngôn ngữ và tích hợp AI.
- **Giai đoạn 4:** Triển khai hệ thống lên AWS, thực hiện kiểm thử, tối ưu hiệu năng, giám sát hệ thống và hoàn thiện tài liệu dự án.

#### 5.2. Yêu cầu kỹ thuật
**Yêu cầu công nghệ:**
- **Frontend:** ReactJS, Vite, HTML5, CSS3, JavaScript.
- **Backend:** AWS Lambda (Node.js).
- **Database:** Amazon DynamoDB.
- **AI Services:** Amazon Bedrock.
- **API:** Amazon API Gateway (REST API).

**Yêu cầu hạ tầng AWS:**
- Amazon S3 và CloudFront để triển khai giao diện web.
- AWS Lambda để xử lý nghiệp vụ.
- Amazon API Gateway để cung cấp API.
- Amazon DynamoDB để lưu trữ dữ liệu.
- AWS IAM để bảo mật hệ thống.


**Yêu cầu phi chức năng:**
- Hệ thống có khả năng mở rộng tự động theo số lượng người dùng.
- Thời gian phản hồi API dưới 2 giây trong điều kiện hoạt động bình thường.
- Dữ liệu được sao lưu và có khả năng khôi phục khi xảy ra sự cố.
- Hệ thống đảm bảo tính bảo mật và phân quyền truy cập phù hợp.

### 6. Lộ trình triển khai
#### Giai đoạn 1: Học tập và tìm hiểu nền tảng AWS
*Thời gian: 05/05/2026 - 31/05/2026 (Tuần 1 - Tuần 4)*
- **Tuần 1 (05/05 - 10/05):**
  - Tìm hiểu tổng quan về điện toán đám mây (Cloud Computing) và Amazon Web Services (AWS).
  - Làm quen với AWS Management Console và AWS CLI.
- **Tuần 2 (11/05 - 17/05):**
  - Thực hành các lab về: AWS IAM, Amazon VPC, Amazon EC2, Amazon RDS.
- **Tuần 3 (18/05 - 24/05):**
  - Thực hành các lab về: Amazon S3, AWS Lambda, Amazon API Gateway.
- **Tuần 4 (25/05 - 31/05):**
  - Thực hành các lab về: AWS CloudFormation, Amazon DynamoDB.
  - Tìm hiểu AWS Well-Architected Framework và kiến trúc Serverless trên AWS.

#### Giai đoạn 2: Phân tích, thiết kế và phát triển hệ thống SmartMenu
*Thời gian: 01/06/2026 - 28/06/2026 (Tuần 5 - Tuần 8)*
- **Tuần 5 (01/06 - 07/06):**
  - Tìm hiểu yêu cầu nghiệp vụ của hệ thống SmartMenu.
  - Phân tích bài toán và xác định các chức năng chính.
  - Thiết lập môi trường phát triển.
- **Tuần 6 (08/06 - 14/06):**
  - Thiết kế kiến trúc AWS cho hệ thống.
  - Xây dựng sơ đồ AWS Solution Architecture.
  - Thiết kế cơ sở dữ liệu trên DynamoDB.
- **Tuần 7 (15/06 - 21/06):**
  - Phát triển giao diện Table Menu Client và Manager Client.
  - Xây dựng Menu Service và Order Service.
- **Tuần 8 (16/06 - 28/06):**
  - Phát triển Translation Service.
  - Tích hợp Amazon Bedrock cho chức năng dịch ngôn ngữ và tư vấn món ăn.
  - Kết nối các dịch vụ với DynamoDB.

#### Giai đoạn 3: Triển khai, kiểm thử và hoàn thiện hệ thống
*Thời gian: 29/06/2026 - 30/07/2026 (Tuần 9 - Tuần 12)*
- **Tuần 9 (29/06 - 05/07):**
  - Triển khai frontend lên Amazon S3 và Amazon CloudFront.
  - Triển khai API bằng Amazon API Gateway và AWS Lambda.
  - Cấu hình IAM và tích hợp các dịch vụ AWS.
- **Tuần 10 (06/07 - 12/07):**
  - Thực hiện Unit Test và Integration Test.
  - Kiểm thử hiệu năng và tối ưu Lambda, DynamoDB.
- **Tuần 11 (13/06 - 19/07):**
  - Tối ưu giao diện người dùng và trải nghiệm sử dụng.
  - Tối ưu chi phí vận hành trên AWS.
  - Viết tài liệu kỹ thuật và tài liệu kiến trúc.
- **Tuần 12 (20/07 - 30/07):**
  - Kiểm thử toàn bộ hệ thống.
  - Sửa lỗi và hoàn thiện sản phẩm.

### 7. Ước tính chi phí
Chi phí vận hành hệ thống được ước tính dựa trên AWS Pricing Calculator, với giả định hệ thống phục vụ khoảng 100 - 200 khách hàng mỗi ngày, tương đương khoảng 5.000 lượt truy cập và 2.000 - 3.000 đơn hàng mỗi tháng.

#### Chi phí hạ tầng

| Dịch vụ AWS | Cấu hình giả định | Chi phí/tháng (USD) | Chi phí/tháng (VNĐ) |
|---|---|---|---|
| **AWS Lambda** | 5.000 request, 512 MB RAM | 0,02 USD | ~500 VNĐ |
| **Amazon API Gateway** | 5.000 API request | 0,02 USD | ~500 VNĐ |
| **Amazon DynamoDB** | Lưu trữ menu và đơn hàng | 2,50 USD | ~65.000 VNĐ |
| **Amazon S3 Standard** | 5 GB lưu trữ | 0,20 USD | ~5.200 VNĐ |
| **Amazon CloudFront** | 10 GB truyền dữ liệu | 1,50 USD | ~39.000 VNĐ |
| **Amazon Bedrock** | Dịch ngôn ngữ và tư vấn AI | 8,00 USD | ~208.000 VNĐ |
| **Amazon CloudWatch** | Giám sát và lưu log | 0,50 USD | ~13.000 VNĐ |
| **AWS IAM** | Quản lý người dùng và phân quyền | Miễn phí | 0 VNĐ |
| **Tổng chi phí dự kiến** | | **~12,74 USD** | **~331.000 VNĐ** |

- **Chi phí trong 12 tháng:** 12,74 × 12 = 152,88 USD/năm (tương đương khoảng **3.975.000 VNĐ/năm**).

#### Chi phí phần cứng
Do hệ thống SmartMenu được triển khai hoàn toàn trên nền tảng điện toán đám mây AWS theo mô hình Serverless, không cần đầu tư máy chủ vật lý hoặc thiết bị phần cứng riêng.
- **Chi phí phần cứng:** 0 VNĐ

### 8. Đánh giá rủi ro
#### Các rủi ro chính
- **Quá tải hệ thống:** Lượng người dùng tăng đột biến có thể làm tăng số lượng request đến API Gateway và Lambda.
- **Mất dữ liệu:** Lỗi ứng dụng hoặc thao tác sai có thể ảnh hưởng đến dữ liệu menu và đơn hàng trong DynamoDB.
- **Tấn công DDoS/Bot:** Các request giả mạo có thể làm giảm hiệu năng và tăng chi phí vận hành.
- **Chi phí AWS vượt dự kiến:** Chi phí có thể tăng khi số lượng request đến Lambda, API Gateway hoặc Amazon Bedrock tăng cao.

#### Biện pháp giảm thiểu rủi ro
- Sử dụng kiến trúc Serverless để tự động mở rộng tài nguyên.
- Kích hoạt Point-in-Time Recovery (PITR) cho DynamoDB và sao lưu dữ liệu định kỳ.
- Sử dụng AWS WAF, Rate Limiting và nguyên tắc Least Privilege của IAM để tăng cường bảo mật.
- Thiết lập AWS Budgets và CloudWatch Billing Alarm để giám sát và kiểm soát chi phí.

#### Kế hoạch dự phòng (Contingency Plans)
- Khôi phục dữ liệu từ DynamoDB hoặc bản sao lưu trên Amazon S3 khi xảy ra sự cố.
- Trong trường hợp Amazon Bedrock không khả dụng, hệ thống vẫn duy trì chức năng xem menu và đặt món, tạm thời vô hiệu hóa các tính năng AI.

### 9. Kết quả mong đợi
#### Cải tiến kỹ thuật
- Xây dựng thành công hệ thống SmartMenu trên kiến trúc AWS Serverless.
- Triển khai các chức năng chính: xem menu, đặt món, quản lý thực đơn và tư vấn món ăn bằng AI.
- Đảm bảo hệ thống có khả năng mở rộng, tính sẵn sàng cao và chi phí vận hành thấp.
- Áp dụng các dịch vụ AWS như API Gateway, Lambda, DynamoDB, S3, CloudFront và Bedrock vào một dự án thực tế.

#### Giá trị dài hạn
- Giúp nhà hàng số hóa quy trình xem menu và đặt món, giảm thời gian phục vụ và chi phí vận hành.
- Nâng cao trải nghiệm khách hàng thông qua hỗ trợ đa ngôn ngữ và tư vấn bằng AI.
- Tạo nền tảng có thể mở rộng thêm các tính năng trong tương lai như thanh toán trực tuyến, phân tích dữ liệu và quản lý chuỗi nhà hàng.
- Có thể tiếp tục phát triển thành sản phẩm thương mại trong tương lai.
