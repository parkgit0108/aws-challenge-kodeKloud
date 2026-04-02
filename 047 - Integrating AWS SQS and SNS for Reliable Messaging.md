# Integrating AWS SQS and SNS for Reliable Messaging

---

<aside>

**Purpose:** Route messages by priority using an SNS topic with message attribute filtering, deliver them to separate SQS queues (high/low), and process them with a Lambda function that drains the high-priority queue first.

</aside>

---

### Architecture

- Publishers → SNS Topic (message attribute `priority`)
    - `priority=high` → High-priority SQS queue → Lambda (event source mapping)
    - `priority=low` → Low-priority SQS queue → Lambda (polled after high queue is empty)

### Procedure

1. Create the CloudFormation template (aws-client host):
    1. Create/edit the file:
        
        ```
        vi /root/datacenter-priority-stack.yml
        ```
        
    2. Paste the template:
        
        ```
        AWSTemplateFormatVersion: '2010-09-09'
        Description: SQS priority queues template
        
        Resources:
          SQSHighPriorityQueue:
            Type: AWS::SQS::Queue
            Properties:
              VisibilityTimeout: 180
              QueueName: datacenter-High-Priority-Queue
        
          SQSLowPriorityQueue:
            Type: AWS::SQS::Queue
            Properties:
              VisibilityTimeout: 180
              QueueName: datacenter-Low-Priority-Queue
        
          PriorityQueuesTopic:
            Type: AWS::SNS::Topic
            Properties:
              TopicName: datacenter-Priority-Queues-Topic
        
          SQSHighQueuePolicy:
            Type: AWS::SQS::QueuePolicy
            Properties:
              Queues:
                - !Ref SQSHighPriorityQueue
              PolicyDocument:
                Id: AllowIncomingMessageFromSNS
                Statement:
                  - Effect: Allow
                    Principal: '*'
                    Action:
                      - sqs:SendMessage
                    Resource:
                      - !GetAtt SQSHighPriorityQueue.Arn
                    Condition:
                      ArnEquals:
                        aws:SourceArn: !Ref PriorityQueuesTopic
        
          SQSLowQueuePolicy:
            Type: AWS::SQS::QueuePolicy
            Properties:
              Queues:
                - !Ref SQSLowPriorityQueue
              PolicyDocument:
                Id: AllowIncomingMessageFromSNS
                Statement:
                  - Effect: Allow
                    Principal: '*'
                    Action:
                      - sqs:SendMessage
                    Resource:
                      - !GetAtt SQSLowPriorityQueue.Arn
                    Condition:
                      ArnEquals:
                        aws:SourceArn: !Ref PriorityQueuesTopic
        
          SNSHighSubscription:
            Type: AWS::SNS::Subscription
            Properties:
              TopicArn: !Ref PriorityQueuesTopic
              Endpoint: !GetAtt SQSHighPriorityQueue.Arn
              Protocol: sqs
              RawMessageDelivery: true
              FilterPolicy: {"priority": ["high"]}
        
          SNSLowSubscription:
            Type: AWS::SNS::Subscription
            Properties:
              TopicArn: !Ref PriorityQueuesTopic
              Endpoint: !GetAtt SQSLowPriorityQueue.Arn
              Protocol: sqs
              RawMessageDelivery: true
              FilterPolicy: {"priority": ["low"]}
        
          LambdaRole:
            Type: AWS::IAM::Role
            Properties:
              RoleName: lambda_execution_role
              AssumeRolePolicyDocument:
                Statement:
                  - Action:
                      - sts:AssumeRole
                    Effect: Allow
                    Principal:
                      Service:
                        - lambda.amazonaws.com
                Version: 2012-10-17
              ManagedPolicyArns:
                - arn:aws:iam::aws:policy/AmazonSQSFullAccess
              Path: /
        
          LambdaFunction:
            Type: AWS::Lambda::Function
            Properties:
              FunctionName: datacenter-priorities-queue-function
              Description: Priority queue function
              Runtime: python3.9
              Code:
                ZipFile: >
                  import boto3
                  import os
                  sqs = boto3.client('sqs')
        
                  def delete_message(queue_url, receipt_handle, message):
                      response = sqs.delete_message(QueueUrl=queue_url, ReceiptHandle=receipt_handle)
                      return "Message " + "'" + message + "'" + " deleted"
        
                  def poll_messages(queue_url):
                      QueueUrl = queue_url
                      response = sqs.receive_message(
                          QueueUrl=QueueUrl,
                          AttributeNames=[],
                          MaxNumberOfMessages=1,
                          MessageAttributeNames=['All'],
                          WaitTimeSeconds=3
                      )
                      if "Messages" in response:
                          receipt_handle = response['Messages'][0]['ReceiptHandle']
                          message = response['Messages'][0]['Body']
                          delete_response = delete_message(QueueUrl, receipt_handle, message)
                          return delete_response
                      else:
                          return "No more messages to poll"
        
                  def lambda_handler(event, context):
                      response = poll_messages(os.environ['high_priority_queue'])
                      if response == "No more messages to poll":
                          response = poll_messages(os.environ['low_priority_queue'])
                      return response
        
              Handler: index.lambda_handler
              MemorySize: 128
              Timeout: 10
              Role:
                Fn::GetAtt:
                  - LambdaRole
                  - Arn
              Environment:
                Variables:
                  high_priority_queue: !Ref SQSHighPriorityQueue
                  low_priority_queue: !Ref SQSLowPriorityQueue
        
          HighPriorityEventSource:
            Type: AWS::Lambda::EventSourceMapping
            Properties:
              EventSourceArn: !GetAtt SQSHighPriorityQueue.Arn
              FunctionName: !Ref LambdaFunction
              BatchSize: 1
              Enabled: true
        
        Outputs:
          SNSTopicARN:
            Value: !Ref PriorityQueuesTopic
        ```
        
2. Create the CloudFormation stack (aws-client host):
    
    ```
    aws cloudformation create-stack \
      --stack-name datacenter-priority-stack \
      --template-body file:///root/datacenter-priority-stack.yml \
      --capabilities CAPABILITY_NAMED_IAM
    ```
    
3. Wait for stack creation to complete (aws-client host):
    
    ```
    aws cloudformation wait stack-create-complete \
      --stack-name datacenter-priority-stack
    ```
    
4. Test message routing (aws-client host):
    1. Publish a high-priority message:
        
        ```
        aws sns publish \
          --topic-arn <SNS_TOPIC_ARN> \
          --message "High priority test" \
          --message-attributes '{"priority":{"DataType":"String","StringValue":"high"}}'
        ```
        
    2. Publish a low-priority message:
        
        ```
        aws sns publish \
          --topic-arn <SNS_TOPIC_ARN> \
          --message "Low priority test" \
          --message-attributes '{"priority":{"DataType":"String","StringValue":"low"}}'
        ```
        
    3. Confirm Lambda processed the high-priority message first (Console → CloudWatch Logs).

### Quick check

- **Expected result:** Messages published with `priority=high` land in the high-priority queue and are processed first. When the high-priority queue is empty, the function polls and processes messages from the low-priority queue.