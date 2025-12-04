# Troubleshooting

**Note:** For Parts 6 and 7, additional troubleshooting may be needed for Lambda 3 and validation features.

## Lambda 1 Not Triggering
- Check CloudWatch logs for errors
- Verify CSV filename ends with `_e.csv` or `_s.csv`
- Verify S3 trigger is configured correctly on Lambda 1
- Check IAM permissions (LabRole should have S3 access)

## JSON File Not Appearing
- Check Lambda 1 CloudWatch logs for errors
- Verify `JSON_BUCKET` environment variable is correct
- Check for CSV parsing errors in logs

## Lambda 2 Not Triggering
- Verify S3 trigger is configured on Lambda 2 for JSON bucket
- Check CloudWatch logs for Lambda 2

## HTML Report Not Loading
- Verify bucket policy allows public read access
- Check that static website hosting is enabled
- Ensure you're using the Object URL, not the S3 console preview

## Report Shows Zero Orders
- Verify JSON files exist in the JSON bucket
- Check Lambda 2 logs to see if files were read correctly
- Verify `JSON_BUCKET` environment variable matches actual bucket name