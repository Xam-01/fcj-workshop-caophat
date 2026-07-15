---
title : "Triển khai Frontend & CloudFront"
date : 2024-01-01 
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

# PHẦN 3 - TRIỂN KHAI FRONTEND VÀ CLOUDFRONT

Khi thông tin đăng nhập IAM đã sẵn sàng, chúng ta sẽ tạo một Amazon S3 Bucket để lưu trữ mã nguồn ứng dụng Web Frontend (`landing`, `owner`, và `tablet`), tải các tệp tin build lên S3, cấu hình Amazon CloudFront Distribution để phân phối mã nguồn trên toàn cầu, và thiết lập cấu hình định tuyến cho SPA (Single Page Application) hoạt động liền mạch với backend APIs.

## 3A. Tạo S3 Bucket & Tải lên Frontend Build

### Bước 3A.1 - Tạo S3 Bucket `smartmenu-fe-prod`
1. Truy cập AWS Console, tìm kiếm **S3** và nhấn **Create bucket**.
2. Thiết lập các thông số sau:
   - **AWS Region:** `ap-southeast-1` (Singapore) hoặc vùng bạn chọn.
   - **Bucket type:** `General purpose`
   - **Bucket namespace:** `Account Regional namespace` (khuyến nghị)
   - **Bucket name prefix:** `smartmenu-fe-prod`
3. Nhấn **Create bucket**.
![Create S3 Bucket](/images/5-Workshop/5.3-IAM-Uploader/s3_create_bucket.png)

### Bước 3A.2 - Cấu hình AWS CLI trên Máy cục bộ
Để tải các tệp tin lên S3 từ máy tính cá nhân:
1. Mở cửa sổ dòng lệnh (terminal) trong thư mục dự án frontend (`smart-menu-fe`).
2. Gõ lệnh cấu hình:
   ```bash
   aws configure
   ```
3. Nhập các thông tin đăng nhập IAM vừa tạo cho tài khoản `smartmenu-uploader` ở Phần 2:
   - **AWS Access Key ID:** `<Access Key ID của bạn>`
   - **AWS Secret Access Key:** `<Secret Key của bạn>`
   - **Default region name:** `ap-southeast-1`
   - **Default output format:** `json`
4. Kiểm tra lại tài khoản đang hoạt động bằng lệnh:
   ```bash
   aws sts get-caller-identity
   ```

### Bước 3A.3 - Tải tệp tin build Frontend lên S3
Vì dự án có nhiều thư mục (`landing`, `owner`, `tablet`), chúng ta build và tải chúng lên các thư mục tương ứng trong S3 bucket:
1. Đồng bộ tệp tin build trang landing vào thư mục gốc của bucket:
   ```bash
   aws s3 sync dist/landing s3://<tên-bucket-của-bạn>/ --delete
   ```
2. Đồng bộ tệp tin build bảng điều khiển owner vào thư mục `/owner/`:
   ```bash
   aws s3 sync dist/owner s3://<tên-bucket-của-bạn>/owner/ --delete
   ```
3. Đồng bộ tệp tin build bảng điều khiển tablet vào thư mục `/tablet/`:
   ```bash
   aws s3 sync dist/tablet s3://<tên-bucket-của-bạn>/tablet/ --delete
   ```
![S3 Sync Terminal](/images/5-Workshop/5.3-IAM-Uploader/s3_sync_terminal.png)

Kiểm tra lại cấu trúc thư mục trên giao diện điều khiển S3:
![S3 Bucket Objects](/images/5-Workshop/5.3-IAM-Uploader/s3_bucket_objects.png)

### Bước 3A.4 - Cấu hình Bucket Policy
Để cho phép truy cập đọc công khai các đối tượng trong bucket:
1. Tại trang chi tiết S3 bucket, chọn tab **Permissions**.
2. Cuộn xuống phần **Bucket policy**, chọn **Edit** và dán đoạn cấu hình JSON sau (thay thế bằng tên bucket thực tế của bạn):
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "Statement1",
               "Effect": "Allow",
               "Principal": "*",
               "Action": "s3:GetObject",
               "Resource": "arn:aws:s3:::<tên-bucket-của-bạn>/*"
           }
       ]
   }
   ```
3. Nhấn **Save changes**.
![S3 Bucket Policy](/images/5-Workshop/5.3-IAM-Uploader/s3_bucket_policy.png)

---

## 3B. Cấu hình CloudFront Distribution

Để bảo vệ origin (S3), tăng tốc độ phân phối tài nguyên tĩnh và chạy đồng thời Frontend/Backend trên cùng một tên miền, chúng ta sẽ tạo một CloudFront distribution.

### Bước 3B.1 - Tạo CloudFront Distribution
1. Tìm kiếm **CloudFront** trên AWS Console, nhấn **Distributions** và chọn **Create distribution**.
2. Tại màn hình **Choose a plan**, chọn gói **Free ($0/month)** và nhấn **Next**.
![CloudFront Choose Plan](/images/5-Workshop/5.3-IAM-Uploader/cf_choose_plan.png)

### Bước 3B.2 - Thiết lập Cấu hình Chung
1. Ở phần **Get started**:
   - **Distribution name:** `smartmenu`
   - **Distribution type:** `Single website or app`
2. Nhấn **Next**.
![CloudFront Get Started](/images/5-Workshop/5.3-IAM-Uploader/cf_get_started.png)

### Bước 3B.3 - Chọn Origin
1. Đối với **Origin domain**, nhấn **Browse S3** và chọn S3 bucket vừa tạo (`smartmenu-fe-prod`).
2. Hãy đảm bảo mục **Allow private S3 bucket access to CloudFront** được chọn (Khuyến nghị) và mục **Use recommended origin settings** được kích hoạt.
![CloudFront Select S3 Origin](/images/5-Workshop/5.3-IAM-Uploader/cf_select_s3.png)

### Bước 3B.4 - Cấu hình Security & Kiểm tra lại
1. Tại phần **Enable security**, đảm bảo các chức năng bảo vệ của WAF đã được bật nhằm ngăn chặn các lỗ hổng web phổ biến.
![CloudFront Enable Security](/images/5-Workshop/5.3-IAM-Uploader/cf_enable_security.png)
2. Trên màn hình **Review and create**, kiểm tra lại các thiết lập:
   - **S3 origin:** `smartmenu-fe-prod.s3.ap-southeast-1.amazonaws.com`
   - **Grant CloudFront access to origin:** `Yes`
   - **Enable Origin Shield:** `No`
3. Nhấn **Create distribution**.
![CloudFront Review Create](/images/5-Workshop/5.3-IAM-Uploader/cf_review_create.png)

### Bước 3B.5 - Cấu hình Default Root Object
1. Khi distribution đã được tạo thành công, bấm vào nó, chọn tab **General** và nhấn **Edit**.
2. Cuộn xuống phần **Default root object**, nhập giá trị `index.html`.
3. Nhấn **Save changes**.
![CloudFront Edit Settings](/images/5-Workshop/5.3-IAM-Uploader/cf_edit_settings.png)

---

## 3C. Kết nối Backend Origin & Định tuyến

Để chuyển tiếp các yêu cầu API từ giao diện frontend đến backend serverless, chúng ta cần thêm Lambda Function URL làm origin thứ hai và thiết lập định tuyến cho path.

### Bước 3C.1 - Thêm Origin cho Lambda Backend
1. Tại chi tiết của CloudFront distribution, di chuyển sang tab **Origins** và chọn **Create origin**.
2. Nhập URL của Lambda Function Backend vào mục **Origin domain**:
   - Định dạng: `lmczvvnxz4nqmcu6settpg2uxm0palnw.lambda-url.us-east-1.on.aws` (hãy dùng URL của Lambda của bạn và lược bỏ `https://`).
3. Đặt **Protocol** là `HTTPS only` và **Minimum Origin SSL protocol** thành `TLSv1.2`.
4. Nhấn **Create origin**.
![CloudFront Create Origin Lambda](/images/5-Workshop/5.3-IAM-Uploader/cf_create_origin_lambda.png)

### Bước 3C.2 - Tạo Behavior định tuyến cho API (`/api/*`)
1. Di chuyển sang tab **Behaviors** và chọn **Create behavior**.
2. Điền các cấu hình behavior:
   - **Path pattern:** `/api/*`
   - **Origin and origin groups:** Chọn origin Lambda Backend vừa tạo.
   - **Viewer protocol policy:** `Redirect HTTP to HTTPS`
   - **Allowed HTTP methods:** `GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE`
   - **Cache policy:** `CachingDisabled`
   - **Origin request policy:** `AllViewerExceptHostHeader`
3. Nhấn **Save changes**.
![CloudFront Create Behavior](/images/5-Workshop/5.3-IAM-Uploader/cf_create_behavior.png)

---

## 3D. Thiết lập SPA URL Rewrite bằng CloudFront Functions

Bởi vì ứng dụng có cấu trúc Single Page Application (SPA) với định tuyến phía Client (ví dụ `/owner/dashboard`, `/tablet/menu`), việc S3 trả lỗi 404 cho các đường dẫn con này là rất phổ biến. Chúng ta cần một CloudFront Function để viết lại (rewrite) các đường dẫn này về tệp tin `index.html` tương ứng ngay tại Edge.

### Bước 3D.1 - Tạo CloudFront Function
1. Ở menu bên trái CloudFront, chọn mục **Functions** và nhấn **Create function**.
2. Cung cấp chi tiết:
   - **Name:** `spa-rewrite`
   - **Runtime:** `cloudfront-js-2.0`
3. Nhấn **Create function**.
![CloudFront Create Function](/images/5-Workshop/5.3-IAM-Uploader/cf_create_function.png)

### Bước 3D.2 - Viết mã nguồn cho URL Rewrite
1. Ở khung soạn thảo mã nguồn trong tab **Development**, dán đoạn code javascript sau:
   ```javascript
   function handler(event) {
       var req = event.request;
       var uri = req.uri;
       
       // Giữ nguyên các tệp tin có đuôi định dạng (ví dụ .js, .css, .png)
       if (uri.match(/\.\w+$/)) return req;
       
       // Luật viết lại đường dẫn cho từng module
       if (uri.startsWith('/owner')) {
           req.uri = '/owner/index.html';
       } else if (uri.startsWith('/tablet')) {
           req.uri = '/tablet/index.html';
       } else if (!uri.startsWith('/api')) {
           req.uri = '/index.html';
       }
       return req;
   }
   ```
2. Nhấn **Save changes**.
![CloudFront Function Code](/images/5-Workshop/5.3-IAM-Uploader/cf_function_code.png)

### Bước 3D.3 - Publish và Liên kết Function
1. Di chuyển sang tab **Publish** và nhấn **Publish function**.
![CloudFront Publish Function](/images/5-Workshop/5.3-IAM-Uploader/cf_publish_function.png)
2. Cuộn xuống phần **Function associations** để liên kết function với behavior:
   - **Event type:** `Viewer request`
   - **Function type:** `CloudFront Functions`
   - **Function name:** `spa-rewrite`
3. Nhấn **Save changes** để áp dụng cơ chế viết lại đường dẫn cho các yêu cầu đến.
![CloudFront Function Association](/images/5-Workshop/5.3-IAM-Uploader/cf_function_association.png)

---

## 3E. (Tùy chọn) Origin thứ 3 - S3 ảnh cũ

Nếu muốn phục vụ ảnh menu qua CDN:
1. Tạo origin trỏ bucket ảnh cũ (OAC tương tự 3A).
2. Behavior `/images/*` → origin ảnh → Cache policy **CachingOptimized**.

## 3F. Kiểm tra THỨ TỰ behavior (rất quan trọng)

CloudFront match từ trên xuống. **Path cụ thể phải nằm trên `Default (*)`**:

| # | Path pattern | Origin | Cache policy | Ghi chú |
|---|---|---|---|---|
| 1 | `/api/*` | Lambda FURL | CachingDisabled | + Origin request **AllViewerExceptHostHeader** |
| 2 | `/images/*` | S3 ảnh | CachingOptimized | tùy chọn |
| 3 | `Default (*)` | S3 FE | CachingOptimized | |

Nếu `Default (*)` bị đẩy lên trên `/api/*`, mọi request API sẽ đi nhầm vào S3 → trả HTML hoặc 403. Dùng nút **Move up/down** để sửa.

## 3G. Deploy + lấy domain + build lại FE với domain thật

1. Chờ distribution status = **Deployed** (5 - 15 phút). Lấy domain `dxxxx.cloudfront.net`.
2. **Quay lại** `.env.production` của FE, điền domain thật vào `VITE_*_ORIGIN` (bước 1B.1), rồi **build lại 3 app + sync lại S3** (bước 1B.2 + 2E).
3. Xóa cache CloudFront để thấy bản mới:
   ```bash
   aws cloudfront create-invalidation --distribution-id EXXXX --paths "/*"
   ```

## 3H. Test end-to-end

```bash
# Landing
curl -I https://dxxxx.cloudfront.net/
# Owner
curl -I https://dxxxx.cloudfront.net/owner/
# Tablet
curl -I https://dxxxx.cloudfront.net/tablet/
# API qua CloudFront
curl https://dxxxx.cloudfront.net/api/v1/health
# SPA fallback (route con phải trả 200)
curl -I https://dxxxx.cloudfront.net/owner/some/deep/route
```

Mở trình duyệt → test đăng nhập owner (kiểm tra cookie refresh token set được, F5 không mất session), test tablet quét QR → tạo session → đặt món.
