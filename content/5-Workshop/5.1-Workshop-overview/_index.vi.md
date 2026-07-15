---
title : "Giới thiệu"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

{{% notice info %}}
Hiện tại dự án demo sẽ không sử dụng DynamoDB và Bedrock AI
{{% /notice %}}

#### Giới thiệu về AWS Serverless & AI

+ **AWS Serverless** là mô hình kiến trúc điện toán đám mây cho phép bạn xây dựng và chạy ứng dụng mà không cần quản lý hay vận hành máy chủ vật lý. Hệ thống tự động mở rộng và chỉ tính phí theo mức độ sử dụng thực tế.
+ **Trí tuệ nhân tạo (AI)** tích hợp qua Amazon Bedrock cung cấp các mô hình ngôn ngữ lớn (LLMs) mạnh mẽ để xử lý các tính năng thông minh như dịch đa ngôn ngữ và tư vấn món ăn trực tiếp cho khách hàng.

#### Tổng quan về workshop
Trong workshop này, bạn sẽ xây dựng hệ thống **SmartMenu** theo kiến trúc Serverless tích hợp AI trên AWS với các thành phần chính:
+ **Frontend:** Giao diện khách hàng và quản trị viên được phát triển bằng ReactJS & Vite, triển khai trên Amazon S3 và phân phối qua Amazon CloudFront.
+ **Backend & API:** API Gateway đóng vai trò tiếp nhận request, định tuyến tới AWS Lambda xử lý các nghiệp vụ (quản lý thực đơn, đặt món, hỗ trợ dịch ngôn ngữ).
+ **Database:** Amazon DynamoDB lưu trữ dữ liệu thực đơn và đơn hàng với hiệu năng cao và độ trễ thấp.
+ **AI Integration:** Sử dụng Amazon Bedrock để kết nối với các mô hình Foundation Models phục vụ dịch thuật và tư vấn món ăn tự động.

![overview](/images/5-Workshop/5.1-Workshop-overview/smartmenu_architecture.png)
