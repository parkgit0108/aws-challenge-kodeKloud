# Automating Infrastructure Deployment with AWS CloudFormation

---

<aside>

**Purpose:** Deploy a basic AWS Lambda function using CloudFormation, including an IAM execution role, then verify the function works by invoking it from the CLI.

</aside>

---

### Procedure

1. Create the CloudFormation template (aws-client host):
    1. Create/edit the file:
        
        ```
        vi /root/datacenter-lambda.yml
        ```
        
    2. Paste the template:
        
        ```
        AWSTemplateFormatVersion: '2010-09-09'
        Description: CloudFormation stack to create a basic Lambda function
        
        Resources:
          LambdaExecutionRole:
            Type: AWS::IAM::Role
            Properties:
              RoleName: lambda_execution_role
              AssumeRolePolicyDocument:
                Version: '2012-10-17'
                Statement:
                  - Effect: Allow
                    Principal:
                      Service: lambda.amazonaws.com
                    Action: sts:AssumeRole
              ManagedPolicyArns:
                - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
        
          DatacenterLambdaFunction:
            Type: AWS::Lambda::Function
            Properties:
              FunctionName: datacenter-lambda
              Runtime: python3.9
              Handler: index.lambda_handler
              Role: !GetAtt LambdaExecutionRole.Arn
              Timeout: 10
              Code:
                ZipFile: |
                  def lambda_handler(event, context):
                      return {
                          "statusCode": 200,
                          "body": "Welcome to KKE AWS Labs!"
                      }
        ```
        
2. Create the stack (aws-client host):
    
    ```
    aws cloudformation create-stack \
      --stack-name datacenter-lambda-app \
      --template-body file:///root/datacenter-lambda.yml \
      --capabilities CAPABILITY_NAMED_IAM
    ```
    
3. Wait for stack creation to complete (aws-client host):
    
    ```
    aws cloudformation wait stack-create-complete \
      --stack-name datacenter-lambda-app
    ```
    
4. Verify (invoke the function) (aws-client host):
    1. Invoke the function:
        
        ```
        aws lambda invoke \
          --function-name datacenter-lambda \
          response.json
        ```
        
    2. Inspect the CLI response (should include `StatusCode: 200`):
        
        ```
        cat response.json
        ```
        

### Quick check

- **Expected result:** `aws lambda invoke` returns `StatusCode: 200`, and `response.json` contains the response body.