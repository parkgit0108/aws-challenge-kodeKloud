# Event-Driven Processing with Amazon S3 and Lambda

---

<aside>

**Purpose:** Trigger an AWS Lambda function on object upload to a public S3 bucket, copy the object to a private S3 bucket, and write a log record to DynamoDB.

</aside>

---

### Architecture

- Public S3 Bucket → (upload/PUT) → Lambda Function
    - Copies file → Private S3 Bucket
    - Writes log → DynamoDB

### Procedure

1. Create the public S3 bucket (Console → S3):
    1. Go to S3 → Create bucket.
    2. Bucket name: `nautilus-public-24772`.
    3. Region: same region you will use for Lambda.
    4. Block Public Access: uncheck Block all public access (acknowledge the warning).
    5. Create bucket.
2. Allow public read on the public bucket (Console → S3):
    1. Open the bucket → Permissions.
    2. Add this Bucket Policy:
        
        ```
        {
          "Version": "2012-10-17",
          "Statement": [
            {
              "Effect": "Allow",
              "Principal": "*",
              "Action": "s3:GetObject",
              "Resource": "arn:aws:s3:::nautilus-public-24772/*"
            }
          ]
        }
        ```
        
    3. Save changes.
3. Create the private S3 bucket (Console → S3):
    1. Go to S3 → Create bucket.
    2. Bucket name: `nautilus-private-31548`.
    3. Block Public Access: keep enabled.
    4. Create bucket.
4. Create the DynamoDB table (Console → DynamoDB):
    1. Go to DynamoDB → Create table.
    2. Table name: `nautilus-S3CopyLogs`.
    3. Partition key: `LogID` (String).
    4. Create table and wait until Status = ACTIVE.
5. Create an IAM role for Lambda (Console → IAM):
    1. Go to IAM → Roles → Create role.
    2. Trusted entity: AWS service.
    3. Use case: Lambda.
    4. Attach policies:
        - `AWSLambdaBasicExecutionRole`
        - `AmazonS3FullAccess`
        - `AmazonDynamoDBFullAccess`
    5. Role name: `lambda_execution_role`.
    6. Create role.
    
    <aside>
    
    For production, replace FullAccess policies with least-privilege permissions scoped to the specific buckets/table.
    
    </aside>
    
6. Prepare the Lambda code (aws-client host):
    1. Create/edit the file:
        
        ```
        vi lambda-function.py
        ```
        
    2. Set these values in the code:
        - `DYNAMODB_TABLE = "nautilus-S3CopyLogs"`
        - `DESTINATION_BUCKET = "nautilus-private-31548"`
    3. Save and exit `vi` (type `:wq!` then Enter).
7. Create the Lambda function (Console → Lambda):
    1. Go to Lambda → Create function.
    2. Choose Author from scratch.
    3. Function name: `nautilus-copyfunction`.
    4. Runtime: Python 3.10
    5. Execution role: Use existing role → `lambda_execution_role`.
    6. Create function.
    7. In Code source, paste/upload the contents of `lambda-function.py`.
    8. Click Deploy.
8. Add the S3 trigger to Lambda (Console → Lambda):
    1. In the function, choose Add trigger.
    2. Trigger configuration: S3.
    3. Bucket: `nautilus-public-24772`.
    4. Event type: PUT.
    5. Acknowledge the permission prompt.
    6. Add trigger.
9. Test by uploading a file to the public bucket (aws-client host):
    
    ```
    aws s3 cp /root/sample.zip s3://nautilus-public-24772/
    ```
    
10. Verify the object was copied to the private bucket (Console → S3):
    1. Open `nautilus-private-31548`.
    2. Confirm `sample.zip` exists.
11. Verify the DynamoDB log entry exists (Console → DynamoDB):
    1. Open Tables → `nautilus-S3CopyLogs`.
    2. Explore table items.
    3. Confirm a new item exists with the source bucket, destination bucket, and object key.

### Quick check

- **Expected result:** Uploading an object to `nautilus-public-24772` triggers Lambda, the object appears in `nautilus-private-31548`, and a corresponding log entry is written to `nautilus-S3CopyLogs`.