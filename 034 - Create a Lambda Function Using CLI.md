# Create a Lambda Function Using CLI

---

<aside>

**Purpose:** Create an AWS Lambda function using the AWS CLI by packaging a Python handler into a zip file, then creating and invoking the function with an existing IAM execution role.

</aside>

---

### Procedure

1. Create the Python script:
    1. On the `aws-client` host, create the handler file:

```
cat << 'EOF' > lambda_function.py
def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": "Welcome to KKE AWS Labs!"
    }
EOF
```

1. Verify the file:

```
cat lambda_function.py
```

1. Zip the Python script:

```
zip function.zip lambda_function.py

# Check if zip file exists:

ls -l function.zip
```

1. Get the IAM role ARN:
    1. You must use the existing role `lambda_execution_role`.
    2. Run:

```
aws iam get-role \
  --role-name lambda_execution_role \
  --query "Role.Arn" \
  --output text
```

1. Copy the ARN output (you’ll use it in the next command).
2. Create the Lambda function using the zip file:

```
aws lambda create-function \
  --function-name <use lab provided name> \
  --runtime python3.9 \
  --role <from above query Role.Arn> \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```

- If successful, AWS will return a JSON output with function details.
1. Verify the Lambda function (optional but recommended):
    1. Invoke the function:

```
aws lambda invoke \
  --function-name labname-lambda-cli \
  response.json
```

1. Check the response:
    - Expected output: { "statusCode": 200, "body": "Welcome to KKE AWS Labs!" }

### Quick check

- **Expected result:** The invocation output returns:
    - `\{ "statusCode": 200, "body": "Welcome to KKE AWS Labs!" \}`