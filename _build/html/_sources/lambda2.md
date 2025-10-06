# Part 3: Create Lambda 2 (JSON to HTML Report Generator)

## Create the Lambda Function
1. Return to the Lambda dashboard
2. Click the orange **"Create function"** button
3. Use the following options:
   - **Author from scratch**
   - **Function name**: Any unique name. Example: `yourname-json-to-html`
   - **Runtime**: Node.js 22.x
   - **Architecture**: x86_64
   - Expand **"Change default execution role"**
     - Choose **"Use an existing role"**
     - From the **Existing role** dropdown, choose **LabRole**
4. Click the orange **"Create function"** button
5. Copy the function name to your scratchpad as "Lambda 2 name"

## Upload the Lambda Code
1. [Download the `json-to-html-function.zip` file provided to you](https://github.com/mgmt-59000-cc/aws-final-project-assets/blob/main/json-to-html-function.zip){target="_blank" rel="noopener noreferrer"}
	- Choose the  **Download Raw File** button to download the file
    - DO NOT unzip the file – keep it in its zipped format
2. With the **Code** tab highlighted, click **"Upload from"** and choose **.zip file**
3. Click **Upload**, select the zip file, then click **Save**

## Configure Environment Variables
1. Choose the **Configuration** tab
2. Click **Environment variables** from the left menu
3. Click **Edit**
4. Add TWO environment variables:
   - Click **Add environment variable**
     - **Key**: `JSON_BUCKET`
     - **Value**: (paste your JSON bucket name from scratchpad)
   - Click **Add environment variable**
     - **Key**: `REPORT_BUCKET`
     - **Value**: (paste your reports bucket name from scratchpad)
5. Click **Save**

## Set Timeout
1. With the **Configuration** tab highlighted, click **General configuration** from the left menu
2. Click **Edit**
3. Update **Timeout** to **1 minute** (0 min 60 sec)
4. Click **Save**

## Add S3 Trigger
1. On the function overview page, click **+ Add trigger**
2. Use the following options:
   - **Select a source**: S3
   - **Bucket**: Select your **JSON bucket** from scratchpad
   - **Event type**: All object create events
   - **Suffix**: `.json` (only trigger on JSON files)
   - Check the box under **Recursive invocation** to acknowledge the warning
3. Click **Add**