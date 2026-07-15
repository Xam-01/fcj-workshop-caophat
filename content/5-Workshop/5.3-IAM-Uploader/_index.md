---
title : "IAM & Uploader Configuration"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

# PART 2 - IAM & UPLOADER CONFIGURATION

To allow the SmartMenu Backend to upload menu images to our newly created S3 Bucket, we need to configure an IAM User with write permissions and generate Access Keys.

## 2A. Configure IAM User & Policy for Backend

### Step 2A.1 - Create IAM User `smartmenu-uploader`
1. In the AWS Console, search for the **IAM** service.
2. Select **IAM users** in the left menu, then click **Create user**.
   ![IAM List](/images/5-Workshop/5.3-IAM-Uploader/iam_list.png)
3. Enter User Details:
   - **User name:** `smartmenu-uploader`
   - *Do not* tick "Provide user access to the AWS Management Console" (since this account will only be used programmatically by the backend API).
   - Click **Next**.
   ![Create IAM User](/images/5-Workshop/5.3-IAM-Uploader/iam_create_user.png)

### Step 2A.2 - Configure Permissions via Inline Policy
1. On the permissions setup screen, click **Next** to proceed to the user details page (or select **Attach policies directly**).
2. Under the **Permissions** tab, click **Add permissions** and select **Create inline policy**.
   ![Add Permission](/images/5-Workshop/5.3-IAM-Uploader/iam_add_permission.png)
   ![Permission Options](/images/5-Workshop/5.3-IAM-Uploader/iam_perm_options.png)
3. In the Policy Editor interface, select the **JSON** tab.
   ![Policy Editor](/images/5-Workshop/5.3-IAM-Uploader/iam_policy_editor.png)
4. Paste the following JSON policy configuration to allow the user to upload (`PutObject`) and delete (`DeleteObject`) files within the `smartmenu-assets-2026` bucket (replace with your actual bucket name):
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
5. Click **Next**.
6. Name the policy `SmartMenuS3UploadPolicy`, review the configuration settings, and click **Create policy**.
   ![Policy Name](/images/5-Workshop/5.3-IAM-Uploader/iam_policy_name.png)

---

## 2B. Generate Access Keys & Test Upload

### Step 2B.1 - Create Access Key
1. Go back to the user details page of `smartmenu-uploader`, select the **Security credentials** tab.
2. Scroll down to the **Access keys** section and click **Create access key**.
   ![Create Access Key](/images/5-Workshop/5.3-IAM-Uploader/iam_access_key.png)
3. Select **Application running outside AWS** as the Use case (as our Backend runs on Lambda or local machine connecting directly to AWS). Click **Next**.
   ![Access Key Use Case](/images/5-Workshop/5.3-IAM-Uploader/iam_access_key_usecase.png)
4. Set a description tag, e.g., `SmartMenu API`. Click **Create access key**.
   ![Access Key Tag](/images/5-Workshop/5.3-IAM-Uploader/iam_access_key_tag.png)
5. **IMPORTANT:** Save the displayed **Access Key ID** and **Secret Access Key**. This is secret credentials to configure in your Backend env settings.

### Step 2B.2 - Verify Image Upload
Once the backend is configured with the correct settings:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_S3_BUCKET_NAME`

The system will upload menu images to the `menu-items/` folder in your S3 Bucket whenever a menu item is created or updated. The uploaded file will be stored and publicly accessible as shown below:
![Upload Verification](/images/5-Workshop/5.3-IAM-Uploader/s3_uploaded_verification.png)
