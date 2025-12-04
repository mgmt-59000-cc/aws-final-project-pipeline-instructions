# Part 1: Create S3 Buckets

**Note:** This project includes Parts 6 and 7 that add validation and summary aggregation features. These are covered in separate sections.

## Create Bucket 1: Upload Bucket (for CSV files)
1. Search for **S3** in AWS console
2. From the dashboard, click the orange **"Create bucket"** button
3. Use the following options. If the option is not noted below, use the default value.
   - **General purpose**
   - **Bucket name**: A *unique* bucket name. Example: `yourname-order-upload`
   - Copy the bucket name into your scratchpad as "Upload bucket name"
4. Click the orange **"Create bucket"** button

## Create Bucket 2: JSON Storage Bucket
1. From the S3 dashboard, click the orange **"Create bucket"** button
2. Use the following options:
   - **General purpose**
   - **Bucket name**: A *unique* bucket name. Example: `yourname-order-json`
   - Copy the bucket name into your scratchpad as "JSON bucket name"
3. Click the orange **"Create bucket"** button

## Create Bucket 3: Reports Bucket (for HTML)
1. From the S3 dashboard, click the orange **"Create bucket"** button
2. Use the following options:
   - **General purpose**
   - **Bucket name**: A *unique* bucket name. Example: `yourname-order-reports`
   - Copy the bucket name into your scratchpad as "Reports bucket name"
3. **Enable static website hosting:**
   - After creating the bucket, click on the bucket name
   - Go to the **Properties** tab
   - Scroll down to **Static website hosting**
   - Click **Edit**
   - Select **Enable**
   - **Index document**: `summary_report.html`
   - Click **Save changes**
4. **Make bucket publicly readable (so you can view the HTML report):**
   - Go to the **Permissions** tab
   - Under **Block public access (bucket settings)**, click **Edit**
   - Uncheck **"Block all public access"**
   - Click **Save changes**
   - Type `confirm` when prompted
   - Under **Bucket policy**, click **Edit**
   - Paste the following policy (replace `YOUR-BUCKET-NAME` with your reports bucket name):
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
       }
     ]
   }
   ```
   - Click **Save changes**