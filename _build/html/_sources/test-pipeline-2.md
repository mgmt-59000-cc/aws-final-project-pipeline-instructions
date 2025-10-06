# Part 5: Test with Store Order CSV

## Upload a Store Order
1. [Download the test file `test_order_s.csv` provided to you](https://github.com/mgmt-59000-cc/aws-final-project-assets/blob/main/test_order_s.csv){target="_blank" rel="noopener noreferrer"}
	- Choose the  **Download Raw File** button to download the file
2. Go to **S3** → Your **upload bucket**
3. Click **Upload**
4. Drag `test_order_s.csv` into the window
5. Click **Upload**

## Verify the Updated Report
1. Wait 10-15 seconds for processing
2. Go to **S3** → Your **reports bucket**
3. Click on `summary_report.html`
4. Click the **Object URL** to view the report
5. Refresh the page if it's already open
6. The report should now show:
   - Increased total orders
   - Both E-commerce AND Store order counts
   - Updated revenue figures
   - Combined statistics from both files