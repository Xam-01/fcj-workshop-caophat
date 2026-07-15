---
title : "Frontend & CloudFront Deployment"
date : 2024-01-01 
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

# PART 3 - FRONTEND & CLOUDFRONT DEPLOYMENT

Now that our IAM credentials are ready, we will create an Amazon S3 Bucket to host our Frontend web applications (`landing`, `owner`, and `tablet`), upload the build files, configure an Amazon CloudFront Distribution to distribute the files globally, and configure routing rules so the SPA (Single Page Application) works seamlessly with the backend APIs.

## 3A. Create S3 Bucket & Upload Frontend Build

### Step 3A.1 - Create S3 Bucket `smartmenu-fe-prod`
1. Go to the AWS Console, search for **S3**, and click **Create bucket**.
2. Set the following settings:
   - **AWS Region:** `ap-southeast-1` (Singapore) or your preferred region.
   - **Bucket type:** `General purpose`
   - **Bucket namespace:** `Account Regional namespace` (recommended)
   - **Bucket name prefix:** `smartmenu-fe-prod`
3. Click **Create bucket**.
![Create S3 Bucket](/images/5-Workshop/5.3-IAM-Uploader/s3_create_bucket.png)

### Step 3A.2 - Configure AWS CLI on Local Machine
To upload the files from your local terminal:
1. Open your terminal in the frontend project folder (`smart-menu-fe`).
2. Run the configuration command:
   ```bash
   aws configure
   ```
3. Enter the IAM credentials generated for your `smartmenu-uploader` user in Part 2:
   - **AWS Access Key ID:** `<Your Access Key>`
   - **AWS Secret Access Key:** `<Your Secret Key>`
   - **Default region name:** `ap-southeast-1`
   - **Default output format:** `json`
4. Verify your login identity:
   ```bash
   aws sts get-caller-identity
   ```

### Step 3A.3 - Upload Frontend Build Files to S3
Since the project has multiple folders (`landing`, `owner`, `tablet`), we build and upload them to their respective directories in the S3 bucket:
1. Sync the landing page build files to the root of the bucket:
   ```bash
   aws s3 sync dist/landing s3://<your-bucket-name>/ --delete
   ```
2. Sync the owner panel build files to the `/owner/` directory:
   ```bash
   aws s3 sync dist/owner s3://<your-bucket-name>/owner/ --delete
   ```
3. Sync the tablet panel build files to the `/tablet/` directory:
   ```bash
   aws s3 sync dist/tablet s3://<your-bucket-name>/tablet/ --delete
   ```
![S3 Sync Terminal](/images/5-Workshop/5.3-IAM-Uploader/s3_sync_terminal.png)

Verify that the files and folders are structured correctly in your S3 bucket console:
![S3 Bucket Objects](/images/5-Workshop/5.3-IAM-Uploader/s3_bucket_objects.png)

### Step 3A.4 - Configure Bucket Policy
To allow public read access to the objects:
1. In your S3 bucket page, go to the **Permissions** tab.
2. Under **Bucket policy**, click **Edit** and paste the following policy (replace with your actual bucket name):
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "Statement1",
               "Effect": "Allow",
               "Principal": "*",
               "Action": "s3:GetObject",
               "Resource": "arn:aws:s3:::<your-bucket-name>/*"
           }
       ]
   }
   ```
3. Click **Save changes**.
![S3 Bucket Policy](/images/5-Workshop/5.3-IAM-Uploader/s3_bucket_policy.png)

---

## 3B. Configure CloudFront Distribution

To secure our origin, speed up asset delivery, and serve frontend/backend from a single domain name, we will create a CloudFront distribution.

### Step 3B.1 - Create CloudFront Distribution
1. Search for **CloudFront** in the AWS Console, click **Distributions**, then click **Create distribution**.
2. On the **Choose a plan** page, select the **Free ($0/month)** option and click **Next**.
![CloudFront Choose Plan](/images/5-Workshop/5.3-IAM-Uploader/cf_choose_plan.png)

### Step 3B.2 - Set Distribution General Options
1. Under **Get started**:
   - **Distribution name:** `smartmenu`
   - **Distribution type:** `Single website or app`
2. Click **Next**.
![CloudFront Get Started](/images/5-Workshop/5.3-IAM-Uploader/cf_get_started.png)

### Step 3B.3 - Specify Origin
1. For **Origin domain**, click **Browse S3** and select the S3 bucket we just created (`smartmenu-fe-prod`).
2. Make sure **Allow private S3 bucket access to CloudFront** is checked (Recommended) and **Use recommended origin settings** is selected.
![CloudFront Select S3 Origin](/images/5-Workshop/5.3-IAM-Uploader/cf_select_s3.png)

### Step 3B.4 - Configure Security & Review
1. Under **Enable security**, ensure WAF protections are enabled to protect your web application from common vulnerabilities.
![CloudFront Enable Security](/images/5-Workshop/5.3-IAM-Uploader/cf_enable_security.png)
2. On the **Review and create** page, verify all settings:
   - **S3 origin:** `smartmenu-fe-prod.s3.ap-southeast-1.amazonaws.com`
   - **Grant CloudFront access to origin:** `Yes`
   - **Enable Origin Shield:** `No`
3. Click **Create distribution**.
![CloudFront Review Create](/images/5-Workshop/5.3-IAM-Uploader/cf_review_create.png)

### Step 3B.5 - Configure Default Root Object
1. Once the distribution is created, click on it, and click **Edit** under **General Settings**.
2. Scroll down to find the **Default root object** field and enter `index.html`.
3. Save changes.
![CloudFront Edit Settings](/images/5-Workshop/5.3-IAM-Uploader/cf_edit_settings.png)

---

## 3C. Connect Backend Origin & Routing

To route API requests from the frontend to our serverless backend, we need to add the Backend Lambda URL as a second origin and configure path patterns.

### Step 3C.1 - Add Lambda Backend Origin
1. Inside your CloudFront distribution details, go to the **Origins** tab and click **Create origin**.
2. Enter your Backend Lambda Function URL in **Origin domain**:
   - Format: `lmczvvnxz4nqmcu6settpg2uxm0palnw.lambda-url.us-east-1.on.aws` (use your actual Lambda function URL without `https://`).
3. Set **Protocol** to `HTTPS only` and **Minimum Origin SSL protocol** to `TLSv1.2`.
4. Click **Create origin**.
![CloudFront Create Origin Lambda](/images/5-Workshop/5.3-IAM-Uploader/cf_create_origin_lambda.png)

### Step 3C.2 - Create Behavior for Backend Routing (`/api/*`)
1. Go to the **Behaviors** tab and click **Create behavior**.
2. Enter the following behavior settings:
   - **Path pattern:** `/api/*`
   - **Origin and origin groups:** Select the Lambda Backend Origin.
   - **Viewer protocol policy:** `Redirect HTTP to HTTPS`
   - **Allowed HTTP methods:** `GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE`
   - **Cache policy:** `CachingDisabled`
   - **Origin request policy:** `AllViewerExceptHostHeader`
3. Click **Create behavior**.
![CloudFront Create Behavior](/images/5-Workshop/5.3-IAM-Uploader/cf_create_behavior.png)

---

## 3D. Setup SPA URL Rewrite with CloudFront Functions

Because we have a Single Page Application (SPA) with path-based routing (e.g., `/owner/dashboard`, `/tablet/menu`), returning a 404 from S3 for these sub-paths is common. We need a CloudFront Function to rewrite these sub-paths to the respective `index.html` at the edge.

### Step 3D.1 - Create CloudFront Function
1. In the CloudFront left navigation bar, click **Functions**, and then click **Create function**.
2. Set the details:
   - **Name:** `spa-rewrite`
   - **Runtime:** `cloudfront-js-2.0`
3. Click **Create function**.
![CloudFront Create Function](/images/5-Workshop/5.3-IAM-Uploader/cf_create_function.png)

### Step 3D.2 - Write URL Rewrite Code
1. In the **Function code** editor under the **Development** tab, paste the following JS router script:
   ```javascript
   function handler(event) {
       var req = event.request;
       var uri = req.uri;
       
       // Keep files with extensions (e.g. .js, .css, .png) unchanged
       if (uri.match(/\.\w+$/)) return req;
       
       // Rewrite rules for each module
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
2. Click **Save changes**.
![CloudFront Function Code](/images/5-Workshop/5.3-IAM-Uploader/cf_function_code.png)

### Step 3D.3 - Publish and Associate Function
1. Switch to the **Publish** tab and click **Publish function**.
![CloudFront Publish Function](/images/5-Workshop/5.3-IAM-Uploader/cf_publish_function.png)
2. Scroll down to **Function associations**, and configure the association:
   - **Event type:** `Viewer request`
   - **Function type:** `CloudFront Functions`
   - **Function name:** `spa-rewrite`
3. Save changes. This will apply the rewrite logic to requests arriving at the default cache behavior.
![CloudFront Function Association](/images/5-Workshop/5.3-IAM-Uploader/cf_function_association.png)

---

## 3E. (Optional) Origin 3 - Old S3 Images Bucket

If you want to serve menu images via CDN:
1. Create an origin pointing to the old images bucket (same OAC settings as 3A).
2. Create a behavior for `/images/*` pointing to this images origin, with Cache Policy **CachingOptimized**.

## 3F. Verify Behavior Priority (Crucial)

CloudFront matches behaviors from top to bottom. **Specific path patterns must be above the Default (*)**:

| # | Path pattern | Origin | Cache policy | Notes |
|---|---|---|---|---|
| 1 | `/api/*` | Lambda FURL | CachingDisabled | + Origin Request Policy **AllViewerExceptHostHeader** |
| 2 | `/images/*` | S3 Images | CachingOptimized | Optional |
| 3 | `Default (*)` | S3 FE | CachingOptimized | |

If `Default (*)` is above `/api/*`, all API requests will go to S3 instead, resulting in HTML files or 403 errors. Use the **Move up/down** buttons to adjust.

## 3G. Deploy, Retrieve Domain, and Rebuild Frontend with Real Domain

1. Wait until the distribution status becomes **Deployed** (approx. 5 - 15 minutes). Copy the distribution domain name, e.g., `dxxxx.cloudfront.net`.
2. **Go back** to the frontend `.env.production` file, fill in this real domain for `VITE_*_ORIGIN` (Step 1B.1 in the preparation section), and then **re-build the 3 apps and sync to S3** (Steps 1B.2 + 2E).
3. Invalidate the CloudFront cache to apply the changes:
     ```bash
     aws cloudfront create-invalidation --distribution-id EXXXX --paths "/*"
     ```

## 3H. End-to-End Testing

```bash
# Landing
curl -I https://dxxxx.cloudfront.net/
# Owner
curl -I https://dxxxx.cloudfront.net/owner/
# Tablet
curl -I https://dxxxx.cloudfront.net/tablet/
# API via CloudFront
curl https://dxxxx.cloudfront.net/api/v1/health
# SPA fallback (deep routes should return 200)
curl -I https://dxxxx.cloudfront.net/owner/some/deep/route
```

Open the browser, test logging in as owner (check if refresh token cookie is set and session is kept on refresh), test tablet scanning the QR code, starting a session, and ordering.
