# Part 8: Understanding the Pipeline

## What Happens When You Upload a CSV?

1. **File Upload**
   - You upload a CSV file (ending with `_e` or `_s`) to the upload bucket
   
2. **Lambda 1 Triggered**
   - S3 event triggers Lambda 1
   - Lambda reads the filename to determine order type (E or S)
   - Downloads and parses the CSV
   - Adds `order_type` field to each record
   - Converts to JSON format
   - Writes JSON to the JSON bucket
   - Deletes CSV from upload bucket

3. **Lambda 2 Triggered**
   - JSON file creation triggers Lambda 2
   - Lambda reads ALL JSON files in the bucket
   - Calculates comprehensive statistics:
     - Total orders, revenue, averages
     - E-commerce vs Store breakdowns
     - Top products by revenue
     - Top states by revenue (e-commerce only)
   - Generates styled HTML report
   - Overwrites `summary_report.html` in reports bucket

4. **View Results**
   - Access the HTML report via the S3 object URL
   - Report updates automatically with each new file processed

> [!IMPORTANT]
> **What about Lambda #3?**
> 
> Although you have created 3 Lambda functions, only *the first two* are truly a part of your "pipeline" (meaning: they will run automatically when documents are uploaded to your "uploads" bucket). The third Lambda that creates the summary CSV file is *manually triggered* and, thus, isn't a part of the automated pipeline!

## Key Concepts Demonstrated

- **Event-driven architecture**: S3 events trigger Lambda functions automatically
- **Serverless computing**: No servers to manage, functions run on-demand
- **Data transformation**: CSV → JSON with enrichment
- **Data aggregation**: Multiple files combined for analytics
- **Multi-stage pipeline**: Sequential processing through multiple buckets
- **Static web hosting**: S3 serving HTML reports