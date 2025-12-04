# Part 6: Adding Validation to Lambda 1 (CSV → JSON)

This part adds validation rules to your existing Lambda 1 function.  
With validation enabled, Lambda 1 separates rows into **valid** and **invalid**, saving invalid rows into the `Invalid/` folder in your JSON bucket.

Follow the steps below in order.

---

## 1. Open Lambda 1 Function Code

1. Go to AWS Console → Lambda  
2. Open your **CSV-to-JSON Lambda (Lambda 1)**  
3. Click **Code**  
4. Replace the existing code with the version in Step 4 (below)

---

## 2. Set Required Environment Variables

Go to: **Configuration → Environment variables → Edit**

Add:

| Key            | Value                      |
|----------------|----------------------------|
| `JSON_BUCKET`  | your JSON bucket name      |
| `INVALID_PREFIX` | `Invalid/`              |

Click **Save**.

---

## 3. How Validation Works

The validation function checks each CSV row against specific rules to determine if it's valid. Here's how it works:

When Lambda 1 processes a CSV file, it examines each row and checks:
- **`transaction_id`** - Must exist (cannot be empty)
- **`transaction_date`** - Must be a valid ISO timestamp format (e.g., "2024-01-15T10:30:00Z")
- **`price`** - Must be a numeric value (can be a number or numeric string)
- **`quantity`** - Must be a numeric value (can be a number or numeric string)
- **`email`** - Must exist (cannot be empty)
- **`subtotal`** - Must be numeric OR can be automatically calculated from `price × quantity` if missing

If a row passes all these checks, it's marked as **valid** and saved to `<filename>.json` in your JSON bucket. If any check fails, the row is marked as **invalid** and saved to `Invalid/<filename>-invalid.json`. The `Invalid/` folder will be created automatically by Lambda if it doesn't exist when invalid rows are detected.

This separation allows you to process valid data normally while keeping track of problematic records for review or correction.

---

## 4. Final Lambda 1 Code (with Validation)

> Copy and paste the entire code below into `index.mjs`  
> Runtime: **Node.js 20.x**  
> Format: **ES6 Modules (`import`)**

```javascript
import {
  S3Client,
  GetObjectCommand,
  PutObjectCommand,
  DeleteObjectCommand
} from "@aws-sdk/client-s3";
import Papa from "papaparse";

const s3Client = new S3Client({ region: process.env.AWS_REGION });

const JSON_BUCKET = process.env.JSON_BUCKET;
const INVALID_PREFIX = process.env.INVALID_PREFIX || "Invalid/";

// Validate ISO timestamp
function isIsoTimestamp(value) {
  if (!value) return false;
  const t = Date.parse(value);
  return !isNaN(t);
}

// Validate numeric field
function isNumeric(value) {
  return (
    typeof value === "number" ||
    (typeof value === "string" &&
      value.trim() !== "" &&
      !isNaN(Number(value)))
  );
}

// Check if a CSV row is valid and normalize subtotal if needed
function isRowValid(row) {
  if (!row.transaction_id) return false;
  if (!isIsoTimestamp(row.transaction_date)) return false;
  if (!isNumeric(row.price)) return false;
  if (!isNumeric(row.quantity)) return false;
  if (!row.email) return false;

  if (!isNumeric(row.subtotal)) {
    const price = Number(row.price);
    const qty = Number(row.quantity);
    if (isNaN(price) || isNaN(qty)) return false;
    row.subtotal = price * qty;
  }

  return true;
}

// Helper: stream S3 object body to string
async function streamToString(stream) {
  const chunks = [];
  for await (const chunk of stream) {
    chunks.push(chunk);
  }
  return Buffer.concat(chunks).toString("utf8");
}

export const handler = async (event) => {
  try {
    console.log("Lambda 1 (CSV → JSON with validation) starting...");

    const record = event.Records[0];
    const srcBucket = record.s3.bucket.name;
    const srcKey = decodeURIComponent(
      record.s3.object.key.replace(/\+/g, " ")
    );

    console.log(`Reading CSV from s3://${srcBucket}/${srcKey}`);

    // Get CSV from source bucket
    const getCmd = new GetObjectCommand({
      Bucket: srcBucket,
      Key: srcKey
    });
    const fileObj = await s3Client.send(getCmd);
    const csvContent = await streamToString(fileObj.Body);

    // Parse CSV
    const parsed = Papa.parse(csvContent, {
      header: true,
      dynamicTyping: true,
      skipEmptyLines: true
    });

    const rows = parsed.data;
    const valid = [];
    const invalid = [];

    for (let row of rows) {
      const ok = isRowValid(row);
      row.is_valid = ok;
      row.processed_timestamp = new Date().toISOString();

      if (ok) {
        valid.push(row);
      } else {
        invalid.push(row);
      }
    }

    console.log(
      `Validation complete: ${valid.length} valid, ${invalid.length} invalid`
    );

    // Write valid records JSON
    if (!JSON_BUCKET) {
      throw new Error("JSON_BUCKET environment variable is required");
    }

    const validKey = srcKey.replace(/\.csv$/i, ".json");
    await s3Client.send(
      new PutObjectCommand({
        Bucket: JSON_BUCKET,
        Key: validKey,
        Body: JSON.stringify(valid, null, 2),
        ContentType: "application/json"
      })
    );
    console.log(`Valid JSON written to s3://${JSON_BUCKET}/${validKey}`);

    // Write invalid records JSON (if any)
    if (invalid.length > 0) {
      const invalidKey = `${INVALID_PREFIX}${srcKey.replace(
        /\.csv$/i,
        "-invalid.json"
      )}`;
      await s3Client.send(
        new PutObjectCommand({
          Bucket: JSON_BUCKET,
          Key: invalidKey,
          Body: JSON.stringify(invalid, null, 2),
          ContentType: "application/json"
        })
      );
      console.log(
        `Invalid JSON written to s3://${JSON_BUCKET}/${invalidKey}`
      );
    } else {
      console.log("No invalid records found.");
    }

    // Delete original CSV from source bucket
    await s3Client.send(
      new DeleteObjectCommand({
        Bucket: srcBucket,
        Key: srcKey
      })
    );
    console.log(`Deleted source CSV s3://${srcBucket}/${srcKey}`);

    return {
      statusCode: 200,
      body: JSON.stringify({
        message: "CSV processed with validation",
        valid_records: valid.length,
        invalid_records: invalid.length,
        output_json: validKey
      })
    };
  } catch (err) {
    console.error("ERROR in Lambda 1:", err);
    return {
      statusCode: 500,
      body: err.message || "Internal Server Error"
    };
  }
};
```

---

## 5. Test the Validation

To test the validation, you can edit the `test_order_e.csv` file that you downloaded earlier:

1. Open `test_order_e.csv` in a text editor
2. Find a row and replace the `price` value with `0` (or remove it entirely)
3. Save the file
4. Upload the modified CSV file to your upload bucket

After Lambda 1 processes the file, check your JSON bucket:

- Valid rows should be in:  

  `yourname-order-json/<filename>.json`

- Invalid rows (including the row with price = 0) should be in:  

  `yourname-order-json/Invalid/<filename>-invalid.json`

---

## 6. Validation Complete

Lambda 1 now performs validation and splits rows into **valid** and **invalid** sets, ready for Lambda 2 and Lambda 3.
