# Create a Lambda Function

---

<aside>

**Purpose:** Create an AWS Lambda function with a basic Python handler, using an IAM execution role with CloudWatch Logs permissions.

</aside>

---

### Procedure

1. Create the IAM role (`lambda_execution_role`):
    1. Go to **AWS Console → IAM**.
    2. Click **Roles → Create role**.
    3. Under **Trusted entity type**, select **AWS service**.
    4. Under **Use case**, choose **Lambda**.
    5. Click **Next**.
    6. Under **Permissions**, attach:
        - `AWSLambdaBasicExecutionRole`
    7. Click **Next**.
    8. Under **Role name**, enter:
        - `lambda_execution_role`
    9. Click **Create role**.
2. Create the Lambda function:
    1. Go to **AWS Console → Lambda**.
    2. Click **Create function**.
    3. Choose **Author from scratch**.
    4. Under **Basic information**:
        - **Function name:** `nautilus-lambda`
        - **Runtime:** **Python 3.x** (any available version)
    5. Under **Permissions → Change default execution role**:
        1. Select **Use an existing role**.
        2. Choose `lambda_execution_role`.
    6. Click **Create function**.
3. Add the Lambda function code:
    1. In the **Code** tab, replace the default code with:

```python
def lambda_handler(event, context):
	return {
		"statusCode": 200,
		"body": "Welcome to KKE AWS Labs!"
	}
```

1. Click **Deploy**.
2. Test the Lambda function:
    1. Click **Test**.
    2. Create a new test event:
        - **Event name:** `test`
        - Keep the default JSON
    3. Click **Test**.

### Quick check

- **Location:** AWS Console → Lambda → Functions → `nautilus-lambda`
- **Expected result:** The test output returns:
    - `{ "statusCode": 200, "body": "Welcome to KKE AWS Labs!" }`