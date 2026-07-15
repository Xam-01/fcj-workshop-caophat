---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---
# Amazon GameLift Servers - Giải pháp quản lý Dedicated Game Server trên AWS

### Giới thiệu
Trong những năm gần đây, các tựa game multiplayer như MOBA, FPS, Battle Royale hay MMORPG ngày càng trở nên phổ biến. Điểm chung của các trò chơi này là yêu cầu một hệ thống máy chủ (Dedicated Game Server) hoạt động ổn định để xử lý kết nối giữa hàng nghìn người chơi cùng lúc.

Nếu tự xây dựng hạ tầng, nhà phát triển sẽ phải giải quyết rất nhiều vấn đề như:
- Triển khai và quản lý nhiều máy chủ.
- Tự động mở rộng khi lượng người chơi tăng (Auto Scaling).
- Giảm số lượng máy chủ khi ít người chơi để tiết kiệm chi phí.
- Đảm bảo độ trễ thấp cho người chơi ở nhiều khu vực khác nhau.
- Theo dõi tình trạng hoạt động của game server.

Đây chính là lý do Amazon GameLift Servers được AWS phát triển nhằm giúp các studio game tập trung vào phát triển gameplay thay vì dành quá nhiều thời gian quản lý hạ tầng.

### Kiến trúc tổng quan
Kiến trúc cơ bản của GameLift có thể mô tả theo quy trình kết nối và xử lý yêu cầu sau:

![Amazon GameLift Architecture](/images/3-BlogsPosted/blog1.png)

**Quy trình diễn ra như sau:**
1. Người chơi gửi yêu cầu tham gia trận đấu từ client.
2. Backend xử lý thông tin người chơi.
3. Backend gửi yêu cầu tạo game session đến Amazon GameLift.
4. GameLift tìm kiếm hoặc khởi chạy máy chủ phù hợp từ nhóm tài nguyên (Fleet).
5. Sau khi cấp phát thành công, GameLift trả về thông tin kết nối (IP và Port) cho Backend để gửi lại cho Client.
6. Người chơi kết nối trực tiếp đến Dedicated Game Server thông qua thông tin IP/Port đó để bắt đầu chơi.

*Điểm đáng chú ý là GameLift chỉ chịu trách nhiệm quản lý và cấp phát máy chủ. Sau khi người chơi nhận được địa chỉ IP và cổng kết nối, dữ liệu gameplay sẽ được truyền trực tiếp giữa Game Client và Dedicated Game Server, không đi qua GameLift để giảm thiểu độ trễ tối đa.*

### Ưu điểm của Amazon GameLift Servers
- **Triển khai nhanh:** Studio không cần xây dựng hệ thống quản lý Dedicated Server từ đầu.
- **Tự động co giãn (Auto Scaling):** Tự động thêm/bớt máy chủ theo số lượng người chơi thời gian thực giúp tiết kiệm chi phí.
- **Độ sẵn sàng cao:** Hỗ trợ Multi-Region để phân phối game toàn cầu.
- **Giảm độ trễ:** Kết nối người chơi tới các game server có khoảng cách địa lý gần nhất.
- **Tích hợp sâu:** Dễ dàng kết nối với các dịch vụ AWS khác như Amazon DynamoDB, Lambda, CloudWatch.

### Hạn chế
Mặc dù rất mạnh, GameLift không phải lúc nào cũng là lựa chọn tối ưu cho tất cả dự án:
- **Thời gian làm quen:** Cần tích hợp GameLift SDK vào mã nguồn game server (C++, C#, Go, v.v.), đòi hỏi thời gian nghiên cứu cấu hình Fleet và Queue.
- **Chi phí:** Có thể tương đối cao đối với các nhà phát triển game độc lập (indie) hoặc dự án nhỏ.
- **Độ tương thích:** Chỉ thực sự phát huy hiệu quả với game multiplayer dùng Dedicated Server. Với các game co-op nhỏ (chỉ vài người chơi), sử dụng máy chủ Amazon EC2 thông thường sẽ đơn giản và tiết kiệm hơn.

### Kết luận
Amazon GameLift Servers không chỉ là dịch vụ triển khai máy chủ thông thường mà là một nền tảng quản lý toàn bộ vòng đời của Dedicated Game Server. Những tính năng như Fleet, Queue và Auto Scaling giúp giảm đáng kể công sức xây dựng hạ tầng, đặc biệt đối với các studio muốn tập trung nguồn lực phát triển gameplay.

---
**Tài liệu tham khảo:**
* [AWS Blog - Introducing Amazon GameLift Anywhere](https://aws.amazon.com/blogs/aws/introducing-amazon-gamelift-anywhere-run-your-game-servers-on-your-own-infrastructure/)