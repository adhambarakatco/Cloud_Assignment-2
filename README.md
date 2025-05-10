# Event-Driven Order Notification System on AWS

This project implements an **event-driven order processing system** using AWS services including DynamoDB, SNS, SQS, Lambda, and CloudWatch.

The system architecture enables **asynchronous order processing**, fault tolerance, and scalability by decoupling the message producer (order publisher) from the message consumer (order processor).

---

## Note:
You can read the Event-Driven Order Notification System – Assignment Report.pdf for full descreption 
Also, you will find screenshots of every step in AWS Assignment 2 Screenshots.pdf

##  **What the CloudFormation Template Does**

The provided CloudFormation template (`template.yaml`) automatically creates and configures:

1. **DynamoDB Table**  
   - Name: `Orders`  
   - Partition Key: `orderId`

2. **SNS Topic**  
   - Name: `OrderTopic`

3. **SQS Queue**  
   - Name: `OrderQueue-CF`

4. **Dead Letter Queue (DLQ)**  
   - Name: `OrderQueueDLQ-CF`  
   - Configured to receive messages after 3 failed processing attempts

5. **SNS Subscription**  
   - Subscribes `OrderQueue-CF` to `OrderTopic`

6. **IAM Role**  
   - Grants Lambda permissions for DynamoDB, SQS, and CloudWatch Logs

7. **Lambda Function** (`ProcessOrderLambda`)  
   - Runtime: Python 3.12  
   - Triggered by `OrderQueue-CF`  
   - Reads messages from SQS  
   - Parses SNS-wrapped messages  
   - Logs order ID  
   - Stores order in DynamoDB

---
## Understanding Visibility Timeout and Dead Letter Queue (DLQ)

Visibility Timeout is a key configuration in Amazon SQS that ensures message processing happens safely and without duplication. When a message is fetched from the queue by a Lambda function, it becomes temporarily hidden from other consumers for a defined period (commonly 30 seconds). This ensures that only one instance of the function processes the message. If the message is handled successfully and deleted, it will not be processed again. However, if the function fails or times out, the message reappears in the queue once the visibility timeout expires, allowing it to be retried.

Dead Letter Queue (DLQ) is a separate queue used to capture messages that repeatedly fail to process. In this system, if a message cannot be processed successfully after three attempts, it is redirected to the DLQ. This prevents problematic messages from blocking the main queue and allows for easier inspection and troubleshooting of errors.

Combined, the visibility timeout and DLQ mechanisms contribute to a more robust, fault-tolerant, and maintainable processing pipeline by ensuring reliable retries and isolating failed messages for review.

##  **How the System Works**

1. A message is published to the **SNS topic** (`OrderTopic`).
2. SNS forwards the message to the **SQS queue** (`OrderQueue-CF`).
3. The **Lambda function** is automatically triggered by SQS when a new message arrives.
4. Lambda reads the message → logs the `orderId` → saves the order to **DynamoDB**.
5. If Lambda fails to process the message **3 times**, it is sent to the **Dead Letter Queue**.

---

##  **How to Deploy**

1. Go to **AWS Console → CloudFormation → Create Stack**.
2. Upload `template.yaml` and create the stack.
3. Wait for stack status to become **CREATE_COMPLETE**.
4. The system will be fully provisioned and ready.

---

##  **Notes**

- All resources created by this template are named with `-CF` suffix to avoid conflicts.
- Make sure to delete the stack after use to avoid unnecessary charges.

---

##  **Outputs**

After deployment, the stack outputs:
- DynamoDB table name
- SNS topic ARN
- SQS queue URLs
- Lambda function name

---

##  **Description**

This project demonstrates a serverless, event-driven architecture for reliable and scalable order processing using AWS managed services.

