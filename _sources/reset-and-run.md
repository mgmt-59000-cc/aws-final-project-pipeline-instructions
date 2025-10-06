# Part 7: Reset the Pipeline and Run the Full Project

## Reset the Pipeline
You will remove the test data from S3 so that you can upload multiple order files and see the consolidated results.
1. Go to **S3** → Your **JSON bucket**
2. Select all objects in the bucket (there should be 2, unless you have run the tests multiple times)
3. Click **Delete**
4. Follow the instructions on the page to permanently delete the objects

## Run the Full Project
1. [Download the test file `final_pipeline_orders.zip` provided to you](https://github.com/mgmt-59000-cc/aws-final-project-assets/blob/main/final_pipeline_orders.zip){target="_blank" rel="noopener noreferrer"}
	- Choose the  **Download Raw File** button to download the file
    - UNZIP this folder on your local machine
	- You should see a folder with 1,000 CSV files – each representing an individual order
2. Search for **S3** in AWS console
3. Click on your **upload bucket** name
4. Click the orange **"Upload"** button
5. Drag ALL of the unzipped CSV order files into the window
6. Click the orange **"Upload"** button at the bottom

## View the Results
1. Open the browser tab that displays your Order Analytics Report
2. Refresh the page to see the numbers change as new orders are processed
	- It will take ~10 minutes to process all the orders – *Remember! This is a demonstration and is not using the full processing power available to us.*

## Special Note - Results
Because this is a demo/prototype and we are using limited sandbox resources, you **MAY NOT** see 1,000 orders in your final Order Analytics Report.

We have not included any error handling or function retries in this code. This is for demonstration purposes only.

You will not be penalized if your final Order Analytics Report does not show 1,000 orders. For a production system, we would have much more layers of error handling in place.