# Project Overview
In this project, you will build a serverless data processing pipeline using AWS Lambda and S3. The pipeline will:
1. Accept CSV files uploaded to S3
2. Transform them to JSON format with additional metadata
3. Generate an HTML analytics report with statistics from all processed files

**Note:** This project includes Parts 6 and 7 that add validation and summary aggregation features to enhance the pipeline.

## Scenario
You work for an online clothing retailer that is owned by a holding company: BTFU Group. 

BTFU has decided to buy another clothing retailer that has both physical stores and a fledgling e-commerce operation. The company will likely divest the physical stores and integrate the e-commerce functions into your group. 

The other clothing retailer used a very outdated order tracking system. The only way to export the historic order data is via CSV files. You will create a project using AWS that will help ingest, sort, and convert the data into JSON data for later processing. You will also create a "dashboard" that shows an overview of the data contained in the files. 

## Project Setup
You have been provided with a subset of the complete dataset: 1,000 CSV files containing sample transaction records from the new company. Each CSV file contains ONE item included in an order; if a customer ordered multiple items, those items will each be in a separate CSV file with the same transaction_id. 

The CSV files are named as follows: 

**20002_3113307591-800-3_s.csv**
- 20002 is the transaction_id. Files that begin with the same transaction_id are part of the same order 
- 3113307591-800-3 is the sku of the item in the order 
- Each filename ends with “s” or “e”. “s” indicates it is an order from a physical store. “e” indicates it is an online order 

## Getting Started
**THIS PROJECT MUST BE COMPLETED IN A SINGLE SITTING** We will use the "Sandbox" function of AWS Academy. The sandbox resources are reset when the sandbox is terminated and *you WILL lose all your work*!

1. Open the AWS Academy LMS
2. From **Modules**, scroll down to find the **Sandbox Environment**
3. The sandbox will open in a new tab. Click **Start Lab** at the top of the screen.
    - The lab may take ~5 minutes to launch
4. When the sandbox shows "created", click the **AWS** button at the top of the screen to load the AWS console
    - You'll perform the remainder of the instructions in the AWS console

```{tableofcontents}
```
