---
title: "Blog 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---
# Amazon CloudWatch là gì? Giám sát Game Server trên AWS Thông qua Amazon CloudWatch

### Giới thiệu
Khi vận hành một trò chơi trực tuyến, việc triển khai game server thành công chỉ là bước khởi đầu. Nhà phát triển còn cần theo dõi sát sao tình trạng hoạt động của hệ thống để phát hiện sớm các sự cố như CPU quá tải, bộ nhớ RAM gần đầy hoặc số lượng kết nối của người chơi tăng đột biến để kịp thời xử lý.

Amazon CloudWatch là dịch vụ giám sát (Monitoring) và quan sát (Observability) của AWS, giúp thu thập và hiển thị các chỉ số hoạt động của tài nguyên đám mây theo thời gian thực. Nhờ đó, đội ngũ vận hành hệ thống có thể nhanh chóng phát hiện các hành vi bất thường và đưa ra phương án khắc phục phù hợp.

### Amazon CloudWatch là gì?
Amazon CloudWatch là một dịch vụ quản lý giám sát tập trung cho các tài nguyên AWS như Amazon EC2, Amazon S3, AWS Lambda, Amazon GameLift, và các ứng dụng chạy trên chúng. 

Hoạt động của CloudWatch xoay quanh ba trụ cột chính:
- **Chỉ số (Metrics):** Thu thập các số liệu về hiệu năng (ví dụ: Tỷ lệ sử dụng CPU, dung lượng đĩa đọc/ghi, lưu lượng mạng).
- **Nhật ký (Logs):** Lưu trữ và phân tích nhật ký hoạt động từ hệ điều hành và ứng dụng.
- **Cảnh báo (Alarms):** Thiết lập các ngưỡng giới hạn. Ví dụ: nếu CPU của Game Server vượt quá 80% trong 5 phút liên tiếp, CloudWatch sẽ tự động kích hoạt cảnh báo gửi qua Amazon SNS (Simple Notification Service) đến email hoặc kênh chat của quản trị viên.

### Ứng dụng giám sát Game Server
Đối với các game nhiều người chơi (multiplayer), việc kết hợp Amazon CloudWatch giúp giám sát sức khỏe của máy chủ game theo thời gian thực. Các chỉ số quan trọng thường được giám sát bao gồm:

![Amazon CloudWatch Dashboard](/images/3-BlogsPosted/blog2.png)
1. **CPU Utilization:** Đảm bảo game server không bị giật lag do xử lý logic game quá tải.
2. **Memory Usage:** Phát hiện lỗi rò rỉ bộ nhớ (memory leaks) của mã nguồn game.
3. **Network Traffic (In/Out):** Giám sát băng thông kết nối từ người chơi, phát hiện các đợt lưu lượng truy cập bất thường.
4. **Disk Usage:** Theo dõi không gian ổ đĩa để tránh lỗi đầy bộ nhớ lưu trữ cục bộ.
5. **Số lượng Game Session & Active Players:** Nắm bắt số trận đấu đang diễn ra và số lượng người dùng thực tế.
6. **Nhật ký hoạt động (Logs):** Thu thập log lỗi hệ thống để phục vụ công tác gỡ lỗi (debugging).

### Lợi ích nổi bật
- **Bảng điều khiển tập trung (Dashboards):** Hiển thị biểu đồ trực quan giúp theo dõi toàn diện sức khỏe hệ thống chỉ trên một giao diện duy nhất.
- **Tự động hóa hành động:** Kết hợp cảnh báo Alarms với EC2 Auto Scaling để tự động tăng số lượng máy chủ khi tải cao và giảm đi khi rảnh rỗi.
- **Khả năng khắc phục sự cố nhanh:** Lưu trữ logs và công cụ phân tích CloudWatch Logs Insights giúp rút ngắn thời gian điều tra nguyên nhân sự cố.

### Kết luận
Amazon CloudWatch đóng vai trò thiết yếu giúp nâng cao tính ổn định và độ tin cậy của game server trực tuyến trên AWS. Bằng cách thu thập Metrics, quản lý Logs và kích hoạt Alarms tự động, CloudWatch giúp nhà phát triển vận hành hệ thống một cách thông minh, từ đó cải thiện và tối ưu hóa trải nghiệm chơi game của người dùng.

---
**Tài liệu tham khảo:**
* [AWS Blog - Game Server Observability with Amazon GameLift and Amazon CloudWatch](https://aws.amazon.com/blogs/gametech/game-server-observability-with-amazon-gamelift-and-amazon-cloudwatch/)