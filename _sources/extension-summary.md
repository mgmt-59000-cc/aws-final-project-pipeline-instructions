# Part 7: Adding Summary Aggregation (Lambda 3)

This part adds a third Lambda function that reads all valid JSON files, computes summary metrics, and writes a `summary.csv` file to your JSON bucket.

Follow the steps below in order.

---
(lambda-3)=
## 1. Create Lambda 3 Function

1. Go to AWS Console → **Lambda**  
2. Click the orange **"Create function"** button
3. Use the following options:
   - **Author from scratch**
   - **Function name**: `order-summary-aggregator` (or similar)  
   - **Runtime**: **Node.js 20.x**
   - **Architecture**: x86_64
   - Expand **"Change default execution role"**
     - Choose **"Use an existing role"**
     - From the **Existing role** dropdown, choose **LabRole**
4. Click the orange **"Create function"** button

---

## 2. Set Environment Variables

On the Lambda 3 function:

1. Go to **Configuration → Environment variables → Edit**
2. Add these keys:

| Key            | Value                                      |
|----------------|--------------------------------------------|
| `JSON_BUCKET`  | your JSON bucket name (e.g. `yourname-order-json`) |
| `JSON_PREFIX`  | (leave empty for root, or `json/` if you use a folder) |
| `REPORT_BUCKET`| same as `JSON_BUCKET` (or another bucket if desired) |
| `REPORT_KEY`   | `reports/summary.csv`                      |

3. Click **Save**

---

## 3. How Lambda 3 Differs from Lambda 2

While both Lambda 2 and Lambda 3 generate summaries of your data, they serve different purposes:

**Lambda 2 (HTML Report Generator):**
- Automatically triggered when JSON files are created in your JSON bucket
- Generates a visual HTML dashboard with charts, tables, and styled formatting
- Designed for human viewing in a web browser
- Shows detailed analytics including top products, top states, revenue breakdowns
- Updates automatically as new files are processed

**Lambda 3 (CSV Summary Aggregator):**
- Must be manually invoked (or triggered on a schedule)
- Generates a simple CSV file with key metrics in a structured format
- Designed for programmatic access and data analysis tools
- Provides basic aggregated statistics: total records, total subtotal, average subtotal, store vs online order counts
- Creates a machine-readable data file that can be imported into spreadsheets or other systems

In essence, Lambda 2 creates a **visual dashboard** for humans, while Lambda 3 creates a **data file** for systems and analysis tools.

---

## 4. Replace Lambda 3 Code

On Lambda 3:

1. Go to the **Code** tab  
2. Set **Runtime** to Node.js 20.x (if not already)  
3. Replace the entire contents of `index.mjs` / `index.js` with the code below  
4. Click **Deploy**

```javascript
import {
  S3Client,
  ListObjectsV2Command,
  GetObjectCommand,
  PutObjectCommand
} from "@aws-sdk/client-s3";

const s3Client = new S3Client({ region: process.env.AWS_REGION });

const JSON_BUCKET = process.env.JSON_BUCKET;
const JSON_PREFIX = process.env.JSON_PREFIX || "";
const REPORT_BUCKET = process.env.REPORT_BUCKET || JSON_BUCKET;
const REPORT_KEY = process.env.REPORT_KEY || "reports/summary.csv";

async function streamToString(stream) {
  const chunks = [];
  for await (const chunk of stream) {
    chunks.push(chunk);
  }
  return Buffer.concat(chunks).toString("utf8");
}

async function listAllJsonKeys(bucket, prefix) {
  let keys = [];
  let continuationToken = undefined;

  do {
    const cmd = new ListObjectsV2Command({
      Bucket: bucket,
      Prefix: prefix || undefined,
      ContinuationToken: continuationToken
    });
    const resp = await s3Client.send(cmd);

    const batchKeys =
      (resp.Contents || [])
        .map(obj => obj.Key)
        .filter(k => !!k)
        .filter(k => !k.toLowerCase().startsWith("invalid/"))
        .filter(k => k.endsWith(".json"));

    keys = keys.concat(batchKeys);
    continuationToken = resp.IsTruncated ? resp.NextContinuationToken : undefined;
  } while (continuationToken);

  console.log("JSON files found:", keys);
  return keys;
}

export const handler = async (event) => {
  console.log("Running Summary Aggregator...");

  if (!JSON_BUCKET) {
    throw new Error("JSON_BUCKET environment variable is required");
  }

  const keys = await listAllJsonKeys(JSON_BUCKET, JSON_PREFIX);

  let totalRecords = 0;
  let totalSubtotal = 0;
  let storeOrders = 0;
  let onlineOrders = 0;

  for (const key of keys) {
    console.log("Processing:", key);

    const getCmd = new GetObjectCommand({
      Bucket: JSON_BUCKET,
      Key: key
    });
    const obj = await s3Client.send(getCmd);
    const bodyText = await streamToString(obj.Body);

    if (!bodyText.trim()) {
      continue;
    }

    let records;
    try {
      records = JSON.parse(bodyText);
    } catch (err) {
      console.error("Failed to parse JSON for key:", key, err);
      continue;
    }

    if (!Array.isArray(records)) {
      continue;
    }

    for (const r of records) {
      if (!r || r.is_valid === false) continue;

      totalRecords += 1;

      const subtotal = Number(r.subtotal) || 0;
      totalSubtotal += subtotal;

      const orderType = (r.order_type || "").toString().toUpperCase();
      if (orderType === "S") storeOrders += 1;
      if (orderType === "E") onlineOrders += 1;
    }
  }

  const avgSubtotal =
    totalRecords > 0 ? totalSubtotal / totalRecords : 0;

  const csvLines = [
    "metric,value",
    `total_records,${totalRecords}`,
    `total_subtotal,${totalSubtotal.toFixed(2)}`,
    `avg_subtotal,${avgSubtotal.toFixed(2)}`,
    `store_orders,${storeOrders}`,
    `online_orders,${onlineOrders}`
  ];

  const csvContent = csvLines.join("\n");

  const putCmd = new PutObjectCommand({
    Bucket: REPORT_BUCKET,
    Key: REPORT_KEY,
    Body: csvContent,
    ContentType: "text/csv"
  });

  await s3Client.send(putCmd);

  console.log(
    `Summary CSV written to s3://${REPORT_BUCKET}/${REPORT_KEY}`
  );

  return {
    statusCode: 200,
    body: JSON.stringify({
      message: "Summary generated",
      total_records: totalRecords,
      total_subtotal: totalSubtotal,
      avg_subtotal: avgSubtotal,
      store_orders: storeOrders,
      online_orders: onlineOrders,
      report_location: `s3://${REPORT_BUCKET}/${REPORT_KEY}`
    })
  };
};
```

---

## 5. Test Lambda 3

1. In the Lambda 3 console, click **Test**
2. Create a new test event (any JSON, e.g. `{}`)  
3. Click **Test** again to invoke the function

You should see:

- Status: `200`  
- A log line like:  
  `Summary CSV written to s3://yourname-order-json/reports/summary.csv`

---

## 6. Verify `summary.csv` in S3

1. Open the **JSON bucket** in S3 (e.g. `yourname-order-json`)  
2. Navigate to the **`reports/`** folder  
3. You should see:

```text
reports/
    summary.csv
```

Download `summary.csv` and open it. It should contain:

```text
metric,value
total_records,…
total_subtotal,…
avg_subtotal,…
store_orders,…
online_orders,…
```

Lambda 3 is now correctly aggregating all valid JSON records into a single summary file.
