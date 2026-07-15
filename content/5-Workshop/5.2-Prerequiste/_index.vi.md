---
title : "Các bước chuẩn bị"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

# PHẦN 1 - CHUẨN BỊ CODE (làm trong máy)

## 1A. BACKEND - sửa để chạy được trên Lambda

BE hiện tại là Express long-running (`src/server.ts` gọi `app.listen()`). Cần tách phần khởi tạo app ra khỏi phần `listen`, rồi thêm 1 file handler cho Lambda. **Logic nghiệp vụ (router, controller, model) giữ nguyên, không đụng.**

### Bước 1A.1 - Cài package cầu nối

```bash
cd smart-menu-api
npm i @codegenie/serverless-express
```

### Bước 1A.2 - Tách `src/server.ts` thành `app.ts` + `server.ts`

Hiện tại `src/server.ts` vừa dựng app (helmet, cors, compression, rate-limit, mount router `/api/v1`, error handler) vừa `app.listen()`. Tách:

**Tạo `src/app.ts`** - chuyển toàn bộ phần khởi tạo app từ `server.ts` sang đây, chỉ bỏ `app.listen()`, thêm `export default app`:

```ts
// src/app.ts
import express from 'express';
import helmet from 'helmet';
import cors from 'cors';
import compression from 'compression';
import cookieParser from 'cookie-parser';
import morgan from 'morgan';
// ...import các router, middleware error handler như hiện tại đang import trong server.ts

const app = express();

// BẮT BUỘC khi đứng sau CloudFront: để rate-limit đọc đúng client IP
// và cookie `secure` nhận đúng protocol qua header X-Forwarded-*
app.set('trust proxy', 1);

app.use(helmet());
app.use(cors({ origin: true, credentials: true })); // xem ghi chú CORS bên dưới
app.use(compression());
app.use(express.json());
app.use(cookieParser());
app.use(morgan('tiny'));

// Mount router y như hiện tại (giữ nguyên prefix /api/v1)
app.use('/api/v1', /* rootRouter của bạn từ src/router/v1 */);

// Global error handler giữ nguyên (đặt sau cùng)
app.use(/* errorHandler middleware của bạn */);

export default app;
```

**Sửa `src/server.ts`** - chỉ còn nhiệm vụ chạy local:

```ts
// src/server.ts
import app from './app';
import { connectDB } from './lib/db'; // đường dẫn thật tới file kết nối Mongo của bạn trong src/lib

const PORT = Number(process.env.PORT) || 3000;

connectDB()
  .then(() => {
    app.listen(PORT, () => {
      console.log(`SmartMenu API (local) running on http://localhost:${PORT}/api/v1`);
    });
  })
  .catch((err) => {
    console.error('Failed to start server:', err);
    process.exit(1);
  });
```

> Lưu ý: `connectDB` là hàm kết nối Mongo hiện có trong `src/lib/` của bạn. Nếu tên/đường dẫn khác, chỉnh import cho đúng. Nếu code cũ connect Mongo ngay trong `server.ts`, tách nó ra thành `src/lib/db.ts` như bước 1A.3.

### Bước 1A.3 - Cache MongoDB connection (chỗ dễ hỏng nhất trên Lambda)

Lambda tái sử dụng container giữa các request → **không** được `mongoose.connect()` mỗi lần gọi. Phải cache connection ở scope module.

**Sửa/tạo `src/lib/db.ts`:**

```ts
// src/lib/db.ts
import mongoose from 'mongoose';

let cached: typeof mongoose | null = null;

export async function connectDB() {
  // Nếu đã kết nối và socket còn sống thì tái dùng
  if (cached && mongoose.connection.readyState === 1) {
    return cached;
  }
  cached = await mongoose.connect(process.env.MONGOOSE_URL as string, {
    serverSelectionTimeoutMS: 5000,
    maxPoolSize: 5, // Lambda: giữ pool nhỏ để không vắt cạn connection Atlas
  });
  return cached;
}
```

> Dùng **MongoDB Atlas** (Mongo không chạy trên Lambda). Trong Atlas → **Network Access** → thêm `0.0.0.0/0` (đơn giản cho MVP; siết lại bằng VPC/NAT về sau nếu cần).

### Bước 1A.4 - Tạo entry cho Lambda: `src/lambda.ts`

Đây là **file duy nhất viết mới** cho Lambda:

```ts
// src/lambda.ts
import serverlessExpress from '@codegenie/serverless-express';
import app from './app';
import { connectDB } from './lib/db';

let cachedServer: ReturnType<typeof serverlessExpress> | null = null;

export const handler = async (event: any, context: any) => {
  // Không chờ event loop rỗng - nếu không Lambda treo vì socket Mongo còn mở
  context.callbackWaitsForEmptyEventLoop = false;

  await connectDB();

  if (!cachedServer) {
    cachedServer = serverlessExpress({ app });
  }
  return cachedServer(event, context);
};
```

### Bước 1A.5 - Sửa cookie refresh token cho production

BE của bạn set refresh token qua httpOnly cookie (endpoint `/auth/login`, `/auth/refresh-token`, `/auth/logout`). Vì FE và API **cùng domain** qua CloudFront (same-site), chỉ cần chỉnh chỗ set cookie (thường trong controller auth `src/controller/v1/`):

```ts
res.cookie('refreshToken', refreshToken, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production', // production bắt buộc HTTPS
  sameSite: 'lax',                                // cùng domain → 'lax' đủ
  path: '/api/v1/auth',                           // khớp path route auth
  maxAge: 7 * 24 * 60 * 60 * 1000,
});
```

> Vì same-domain nên **không cần** `sameSite: 'none'`. Chỉ dùng `'none'` nếu sau này tách FE/BE khác domain (khi đó bắt buộc kèm `secure: true`).

### Bước 1A.6 - Ghi chú CORS

Cùng domain qua CloudFront ⇒ request là same-origin ⇒ CORS gần như không kích hoạt. Cứ để `cors({ origin: true, credentials: true })` cho an toàn (không gây hại). Không cần whitelist domain riêng.

### Bước 1A.7 - Kiểm tra `tsconfig.json`

Đảm bảo có:
```json
{
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src",
    "module": "commonjs",
    "target": "ES2020",
    "esModuleInterop": true
  }
}
```
> Handler trên Lambda sẽ là `dist/lambda.handler`, nên `lambda.ts` phải build ra `dist/lambda.js`. Nếu bạn dùng path alias (`tsconfig-paths`), cân nhắc build bằng `esbuild`/`tsc` sao cho alias được resolve trong `dist` (xem ghi chú cuối phần).

### Bước 1A.8 - Build + đóng gói zip

```bash
npm run build            # tsc → ra dist/ (phải có dist/lambda.js)

# Cài lại chỉ dependencies production để zip gọn
rm -rf node_modules
npm ci --omit=dev

# Đóng gói
zip -r function.zip dist node_modules package.json
```

> Kết thúc 1A: bạn có **`function.zip`** - chứa `dist/`, `node_modules` (prod), `package.json`. Đây là artifact BE hoàn chỉnh để upload lên Lambda ở PHẦN 2.

> ⚠️ Nếu dự án dùng `tsconfig-paths` (alias `@/...`): `tsc` **không** rewrite alias trong output. Hai cách xử lý:
> 1. Build bằng esbuild bundle 1 file: `npx esbuild src/lambda.ts --bundle --platform=node --target=node20 --outfile=dist/lambda.js` rồi zip `dist` + `package.json` (không cần `node_modules` nếu bundle hết) - gọn và nhanh cold start nhất.
> 2. Hoặc thêm `tsconfig-paths/register` vào runtime - phức tạp hơn trên Lambda. Khuyên dùng cách esbuild.

---

## 1B. FRONTEND - build 3 app tĩnh

FE build ra 3 app qua Vite mode. Vì tất cả nằm chung 1 domain, API gọi bằng path tương đối `/api/v1`.

### Bước 1B.1 - Tạo file env production

Tạo `.env.production` (Vite tự nạp khi build production). Domain sẽ là domain CloudFront - nhưng lúc này chưa có, nên tạm để giá trị và **quay lại điền sau khi có domain CloudFront ở PHẦN 3** (rồi build lại). Hoặc nếu bạn đã có domain CloudFront/custom domain thì điền luôn.

```
# API cùng domain → dùng path tương đối
VITE_API_BASE_URL=/api/v1

# Origin của từng app - dùng để tablet sinh QR trỏ đúng app, và điều hướng chéo
# Điền domain CloudFront thật sau PHẦN 3, ví dụ:
VITE_LANDING_ORIGIN=https://dxxxx.cloudfront.net
VITE_OWNER_ORIGIN=https://dxxxx.cloudfront.net/owner
VITE_TABLET_ORIGIN=https://dxxxx.cloudfront.net/tablet
```

> ⚠️ Quan trọng: tablet app sinh **QR code cho từng bàn** trỏ về `VITE_TABLET_ORIGIN`. Nếu để sai (vd localhost), QR in ra sẽ không mở được. Phải là domain production thật.
>
> ⚠️ Đảm bảo **không** bật `VITE_USE_MOCK_API=true` trong bản production (mock API chỉ dùng khi dev). Để trống hoặc `false`.

### Bước 1B.2 - Build 3 app ra 3 thư mục riêng

`npm run build` mặc định build theo mode hiện tại. Cần build từng mode ra outDir riêng:

```bash
cd smart-menu-fe
npm ci

npm run build -- --mode landing --outDir dist/landing --emptyOutDir
npm run build -- --mode owner   --outDir dist/owner   --emptyOutDir
npm run build -- --mode tablet  --outDir dist/tablet  --emptyOutDir
```

> Nếu script `build` trong `package.json` đã cố định mode/outDir khiến cờ trên không ăn, mở `package.json` thêm 3 script:
> ```json
> "build:landing": "vite build --mode landing --outDir dist/landing --emptyOutDir",
> "build:owner":   "vite build --mode owner   --outDir dist/owner   --emptyOutDir",
> "build:tablet":  "vite build --mode tablet  --outDir dist/tablet  --emptyOutDir"
> ```
> rồi chạy `npm run build:landing && npm run build:owner && npm run build:tablet`.

Kết quả cần có 3 thư mục, mỗi thư mục là 1 SPA hoàn chỉnh:
```
dist/landing/index.html + assets/
dist/owner/index.html   + assets/
dist/tablet/index.html  + assets/
```

> ⚠️ Vấn đề base path: khi owner/tablet chạy ở sub-path (`/owner/`, `/tablet/`), Vite cần biết `base` để nạp đúng đường dẫn `assets/`. Thêm vào `vite.config.ts`:
> ```ts
> // vite.config.ts
> export default defineConfig(({ mode }) => ({
>   base:
>     mode === 'owner' ? '/owner/' :
>     mode === 'tablet' ? '/tablet/' :
>     '/',
>   // ...phần config còn lại giữ nguyên
> }));
> ```
> Không set `base`, các app con sẽ trắng trang vì tìm asset sai đường dẫn.

> Kết thúc 1B: bạn có **3 thư mục dist** sẵn sàng upload lên S3.

---

## 1C. AWS S3 BUCKET - Tạo Bucket lưu trữ Ảnh món ăn

Để ứng dụng có thể hiển thị và upload ảnh món ăn, chúng ta cần chuẩn bị một S3 Bucket với chính sách truy cập công khai (Public Read).

### Bước 1C.1 - Tạo S3 Bucket cho Ảnh
1. Truy cập AWS Console, tìm kiếm và chọn dịch vụ **S3**.
2. Nhấn nút **Create bucket**.
   ![S3 List](/images/5-Workshop/5.2-Prerequisite/s3_list.png)
3. Cấu hình thông tin Bucket:
   - **Bucket name:** Nhập tên duy nhất toàn cầu, ví dụ: `smartmenu-assets-2026`.
   - **AWS Region:** Chọn **Asia Pacific (Singapore) ap-southeast-1** (hoặc region bạn mong muốn).
   ![S3 Create Settings](/images/5-Workshop/5.2-Prerequisite/s3_create.png)
4. Cấu hình Quyền truy cập công khai:
   - Cuộn xuống phần **Block Public Access settings for this bucket**.
   - Bỏ tích chọn **Block all public access** (Cho phép truy cập công khai).
   - Tích chọn hộp thoại xác nhận **I acknowledge that the current settings might result in this bucket and the objects within becoming public**.
   ![S3 Public Access](/images/5-Workshop/5.2-Prerequisite/s3_public_access.png)
5. Nhấn **Create bucket** ở cuối trang. Bạn sẽ thấy thông báo tạo bucket thành công.
   ![S3 Created](/images/5-Workshop/5.2-Prerequisite/s3_created.png)

### Bước 1C.2 - Cấu hình Bucket Policy (Cho phép Đọc công khai)
1. Chọn bucket vừa tạo từ danh sách, chọn tab **Permissions**.
2. Cuộn xuống mục **Bucket policy**, nhấn **Edit**.
   ![S3 Policy Edit](/images/5-Workshop/5.2-Prerequisite/s3_policy_edit.png)
3. Dán đoạn JSON sau vào Policy Editor để cho phép mọi người đọc ảnh từ bucket (thay thế `smartmenu-assets-2026` bằng tên bucket của bạn):
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "PublicReadGetObject",
               "Effect": "Allow",
               "Principal": "*",
               "Action": "s3:GetObject",
               "Resource": "arn:aws:s3:::smartmenu-assets-2026/*"
           }
       ]
   }
   ```
   ![S3 Policy JSON](/images/5-Workshop/5.2-Prerequisite/s3_policy_json.png)
4. Nhấn **Save changes**. Trạng thái bucket sẽ chuyển sang **Public**.
   ![S3 Policy Success](/images/5-Workshop/5.2-Prerequisite/s3_policy_success.png)