---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Triển khai Full-Stack lên AWS - Qua dự án SmartMenu

#### Tổng quan

**SmartMenu** là hệ thống menu điện tử thông minh tích hợp trí tuệ nhân tạo (AI), được xây dựng trên kiến trúc Serverless tiên tiến của AWS.

Trong bài lab này, chúng ta sẽ học cách triển khai và cấu hình một ứng dụng Full-stack hoàn chỉnh sử dụng các dịch vụ cloud của AWS để đảm bảo tính sẵn sàng cao, bảo mật và tối ưu chi phí.

Chúng ta sẽ tiếp cận việc triển khai thông qua các thành phần cốt lõi của giải pháp:
+ **Frontend:** Sử dụng Amazon S3 và CloudFront để lưu trữ và phân phối ứng dụng ReactJS một cách nhanh chóng với độ trễ thấp toàn cầu.
+ **Backend & Database:** Triển khai các API serverless với API Gateway và AWS Lambda kết nối dữ liệu tới Amazon DynamoDB.
+ **AI & Security:** Tích hợp mô hình AI từ Amazon Bedrock và thiết lập phân quyền bảo mật IAM chặt chẽ.

#### Nội dung

1. [Tổng quan về workshop](5.1-Workshop-overview/)
2. [Chuẩn bị](5.2-Prerequiste/)
3. [Cấu hình IAM & Tài khoản Uploader](5.3-IAM-Uploader/)
4. [Triển khai Frontend & CloudFront](5.4-Frontend-Cloudfront/)
5. [Demo sản phẩm](5.5-Demo/)
