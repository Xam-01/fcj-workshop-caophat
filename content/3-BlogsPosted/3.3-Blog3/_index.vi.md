---
title: "Blog 3"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---
# Tối ưu chi phí lưu trữ Game Assets trên Amazon S3 bằng Lifecycle Policies

### Giới thiệu
Trong quá trình phát triển game, các studio phải lưu trữ một lượng dữ liệu khổng lồ bao gồm textures, model 3D, âm thanh, video, các bản cập nhật (patches) và các file sao lưu (backups). Ban đầu, những dữ liệu này thường được lưu trên lớp lưu trữ **Amazon S3 Standard** để đảm bảo tốc độ truy cập nhanh nhất. 

Tuy nhiên, sau một thời gian, nhiều tệp tin (như các phiên bản build cũ, backup của các tháng trước) không còn được truy cập thường xuyên nữa nhưng vẫn tiếp tục tiêu tốn chi phí lưu trữ ở mức giá cao của lớp Standard. 

**Amazon S3 Lifecycle Policies** là giải pháp giúp tự động hóa việc quản lý vòng đời của đối tượng, tự động chuyển đổi dữ liệu sang các lớp lưu trữ có chi phí thấp hơn hoặc xóa chúng đi khi không còn cần thiết, từ đó giúp doanh nghiệp tiết kiệm chi phí vận hành và đơn giản hóa việc quản trị.

### Amazon S3 Lifecycle hoạt động như thế nào?
S3 Lifecycle hoạt động dựa trên các quy tắc (Rules) cấu hình mốc thời gian lưu trữ của đối tượng kể từ ngày khởi tạo. 

Ví dụ về một quy trình tự động hóa vòng đời lưu trữ điển hình:

![Amazon S3 Lifecycle Process](/images/3-BlogsPosted/blog3.png)

* **30 ngày đầu:** Đối tượng được lưu trên lớp **Amazon S3 Standard** để đảm bảo hiệu năng tối đa cho người chơi và nhà phát triển.
* **Sau 30 ngày:** Đối tượng tự động chuyển sang **S3 Standard-IA (Infrequent Access)** vì tần suất truy cập giảm.
* **Sau 60 ngày:** Đối tượng được chuyển sang **S3 Glacier Flexible Retrieval** để lưu trữ dài hạn với mức giá cực kỳ rẻ.
* **Sau 365 ngày:** Hệ thống tự động xóa bỏ hoàn toàn đối tượng để giải phóng bộ nhớ.

### Lợi ích của Lifecycle Policy
1. **Giảm chi phí lưu trữ tối đa:** Đây là ưu điểm lớn nhất. Việc chuyển các dữ liệu ít dùng (như log, backup) từ S3 Standard sang S3 Glacier giúp tiết kiệm tới 70-80% chi phí lưu trữ.
2. **Tự động hóa hoàn toàn:** Nhà phát triển không cần tải dữ liệu về, tải lên lại các lớp lưu trữ khác hoặc viết script chạy định kỳ để dọn dẹp. Amazon S3 sẽ tự động xử lý mọi quy trình chuyển đổi và xóa theo đúng thiết lập cấu hình.

### Ứng dụng thực tiễn trong Game Development
Trong ngành phát triển game, Lifecycle Policies cực kỳ hữu ích cho các dự án có dữ liệu theo mùa hoặc có nhiều bản cập nhật định kỳ. Ví dụ:
- Các Assets đang hoạt động (textures, patch mới) được lưu trên S3 Standard để game clients tải nhanh.
- Dữ liệu sự kiện cũ hoặc các bản build thử nghiệm cũ được chuyển sang S3 Glacier để lưu trữ dự phòng với chi phí cực thấp, và chỉ khôi phục khi cần thiết.

### Một số lưu ý quan trọng
- **Chi phí truy xuất (Retrieval Costs):** Chuyển dữ liệu sang các lớp Standard-IA hay Glacier sẽ làm phát sinh chi phí khi bạn muốn đọc/truy xuất dữ liệu đó. Nếu dữ liệu vẫn được đọc thường xuyên, việc áp dụng Lifecycle Policy có thể làm tăng tổng chi phí thay vì tiết kiệm.
- **Giải pháp thay thế:** Đối với các dữ liệu khó dự đoán tần suất truy cập, AWS khuyến nghị nên sử dụng lớp **Amazon S3 Intelligent-Tiering** để hệ thống tự động tối ưu hóa dựa trên hành vi truy cập thực tế của người dùng.

### Kết luận
Amazon S3 Lifecycle Policies là một thực hành tốt (Best Practice) được AWS khuyến nghị khi thiết kế hệ thống lưu trữ trên đám mây. Việc cấu hình đúng đắn các quy tắc chuyển đổi lớp lưu trữ và dọn dẹp dữ liệu hết hạn giúp các nhà phát triển game vận hành hệ thống một cách hiệu quả, tiết kiệm chi phí và tập trung nguồn lực vào việc phát triển game.

---
**Tài liệu tham khảo:**
* [Manage Storage Costs with Amazon S3 Lifecycle Rules](https://dev.to/sachithmayantha/manage-storage-costs-with-amazon-s3-lifecycle-rules-1fnj)
* [AWS Blog - Optimize Storage Costs with New Amazon S3 Lifecycle Filters and Actions](https://aws.amazon.com/blogs/storage/optimize-storage-costs-with-new-amazon-s3-lifecycle-filters-and-actions/)