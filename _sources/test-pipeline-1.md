# Part 4: Test Your Pipeline

## Test with E-commerce Order CSV
1. [Download the test file `test_order_e.csv` provided to you](https://github.com/mgmt-59000-cc/aws-final-project-assets/blob/main/test_order_e.csv){target="_blank" rel="noopener noreferrer"}
	- Choose the  **Download Raw File** button to download the file
2. Search for **S3** in AWS console
3. Click on your **upload bucket** name
4. Click the orange **"Upload"** button
5. Drag `test_order_e.csv` into the window
6. Click the orange **"Upload"** button at the bottom

## Verify Lambda 1 Execution
1. Search for **Lambda** in AWS console
2. Click on your **Lambda 1** (CSV to JSON) function name
3. Click the **Monitor** tab
4. Click **View CloudWatch logs**
5. Click on the most recent log stream
6. You should see logs showing:
   - "Processing: test_order_e.csv"
   - "Order type: E"
   - "Transformed X records"
   - "JSON written to [your-json-bucket]/test_order_e.json"

## Verify JSON File Was Created
1. Go to **S3** in AWS console
2. Click on your **JSON bucket** name
3. You should see a file named `test_order_e.json`
4. Click on the filename to preview the contents

## Verify Lambda 2 Execution
1. Go to **Lambda** in AWS console
2. Click on your **Lambda 2** (JSON to HTML) function name
3. Click the **Monitor** tab
4. Click **View CloudWatch logs**
5. Click on the most recent log stream
6. You should see logs showing:
   - "Generating HTML report"
   - "Found X JSON files"
   - "Total orders across all files: X"
   - "HTML report written to report bucket"

## View the HTML Report
1. Go to **S3** in AWS console
2. Click on your **reports bucket** name
3. You should see a file named `summary_report.html`
4. Click on the filename
5. In the **Object overview** section, find the **Object URL** 
6. Click the URL to open the HTML report in a new browser tab
7. You should see a styled dashboard with:
   - Total orders count
   - E-commerce vs Store order breakdown
   - Revenue statistics
   - Top products table
   - Top states table (for e-commerce orders)