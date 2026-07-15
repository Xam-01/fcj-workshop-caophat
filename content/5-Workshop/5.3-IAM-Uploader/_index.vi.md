---
title : "Cấu hình IAM & Tài khoản Uploader"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

# PHẦN 2 - CẤU HÌNH IAM & TÀI KHOẢN UPLOAD

Để backend của ứng dụng SmartMenu có thể tải ảnh món ăn lên S3 Bucket vừa tạo, chúng ta cần cấu hình một IAM User với quyền ghi thích hợp và sinh Access Key.

## 2A. Cấu hình IAM User & Policy cho Backend

### Bước 2A.1 - Tạo IAM User `smartmenu-uploader`
1. Trên AWS Console, tìm kiếm dịch vụ **IAM**.
2. Chọn mục **IAM users** ở menu bên trái, sau đó nhấn **Create user**.
   ![IAM List](/images/5-Workshop/5.3-IAM-Uploader/iam_list.png)
3. Điền thông tin User details:
   - **User name:** `smartmenu-uploader`
   - *Không* tích chọn "Provide user access to the AWS Management Console" (vì tài khoản này chỉ dùng cho backend qua API).
   - Nhấn **Next**.
   ![Create IAM User](/images/5-Workshop/5.3-IAM-Uploader/iam_create_user.png)

### Bước 2A.2 - Cấu hình Quyền (Permissions) bằng Inline Policy
1. Trong màn hình cấu hình quyền, nhấn **Next** để chuyển sang màn hình chi tiết của user (hoặc chọn **Attach policies directly**). 
2. Ở tab **Permissions**, nhấn nút **Add permissions** và chọn **Create inline policy**.
   ![Add Permission](/images/5-Workshop/5.3-IAM-Uploader/iam_add_permission.png)
   ![Permission Options](/images/5-Workshop/5.3-IAM-Uploader/iam_perm_options.png)
3. Trong giao diện Policy Editor, chọn tab **JSON**.
   ![Policy Editor](/images/5-Workshop/5.3-IAM-Uploader/iam_policy_editor.png)
4. Dán đoạn cấu hình JSON sau để cho phép user tải lên (`PutObject`) và xóa (`DeleteObject`) tệp tin trong bucket `smartmenu-assets-2026` (thay thế bằng tên bucket của bạn):
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "s3:PutObject",
                   "s3:DeleteObject"
               ],
               "Resource": "arn:aws:s3:::smartmenu-assets-2026/*"
           }
       ]
   }
   ```
   ![Policy JSON](/images/5-Workshop/5.3-IAM-Uploader/iam_policy_json.png)
5. Nhấn **Next**. 
6. Đặt tên Policy là `SmartMenuS3UploadPolicy`, kiểm tra lại các thiết lập và nhấn **Create policy**.
   ![Policy Name](/images/5-Workshop/5.3-IAM-Uploader/iam_policy_name.png)

---

## 2B. Khởi tạo Access Key & Upload Kiểm thử

### Bước 2B.1 - Tạo Access Key
1. Quay lại trang chi tiết của user `smartmenu-uploader`, chọn tab **Security credentials**.
2. Cuộn xuống phần **Access keys**, nhấn **Create access key**.
   ![Create Access Key](/images/5-Workshop/5.3-IAM-Uploader/iam_access_key.png)
3. Chọn mục **Application running outside AWS** làm Use case (vì Backend của chúng ta chạy trên Lambda hoặc máy local kết nối trực tiếp đến AWS). Nhấn **Next**.
   ![Access Key Use Case](/images/5-Workshop/5.3-IAM-Uploader/iam_access_key_usecase.png)
4. Đặt tag mô tả, ví dụ: `SmartMenu API`. Nhấn **Create access key**.
   ![Access Key Tag](/images/5-Workshop/5.3-IAM-Uploader/iam_access_key_tag.png)
5. **QUAN TRỌNG:** Lưu lại thông tin **Access Key ID** và **Secret Access Key** được hiển thị. Đây là những thông tin mật để điền vào cấu hình Backend.

### Bước 2B.2 - Kiểm tra Upload ảnh thực tế
Khi backend cấu hình đúng các thông tin:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_S3_BUCKET_NAME`

Hệ thống sẽ tải ảnh món ăn lên thư mục `menu-items/` trong S3 Bucket của bạn mỗi khi tạo mới/sửa món ăn. Kết quả tệp tin được lưu trữ và truy cập công khai như dưới đây:
![Upload Verification](/images/5-Workshop/5.3-IAM-Uploader/s3_uploaded_verification.png)
