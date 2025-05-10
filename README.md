# Event-Driven Order Notification System Using AWS

## Overview

This project implements a simplified event-driven backend system for an e-commerce platform using AWS services. The system processes incoming orders, stores them in a database, and sends notifications, ensuring reliable message handling even in failure scenarios.

### Key AWS Services Used

- **Amazon SNS** – Publishes order notifications.
- **Amazon SQS** – Queues messages for processing.
- **AWS Lambda** – Processes messages and stores order data.
- **Amazon DynamoDB** – Stores order details.
- **Dead-Letter Queue (DLQ)** – Handles failed message processing.

---

## Setup Instructions

### Step 1 - DynamoDB Setup
1. Go to **DynamoDB** console
2. Create table named `Orders`
3. Configure:
   - **Partition Key**: `orderId` (String)
   - **Attributes**: 
     - `userId` (String)
     - `itemName` (String) 
     - `quantity` (Number)
     - `status` (String)
     - `timestamp` (String)

### Step 2 - Create SNS Topic
1. Go to **SNS** console
2. Create topic named `OrderTopic`
3. Add subscription:
   - Protocol: **Amazon SQS**
   - Endpoint: Select `OrderQueue` after completing step 3
   - Enable raw message delivery in order to simplify the testing process and lambda function code

### Step 3 - Create SQS Queue
1. Go to **SQS** console
2. Create standard queue named `OrderQueue`
3. Subscribe it to `OrderTopic`
4. Configure Dead-Letter Queue:
   - Create `OrderDLQ` queue
   - In `OrderQueue` → Redrive Policy:
     - Set **Dead-letter queue** = `OrderDLQ`
     - Set **maxReceiveCount** = 3

### Step 4 - Deploy Lambda Function
1. Go to **Lambda** console
2. Create function:
   - Runtime: Python 3.13
   - Permissions: Auto-create basic execution role or create a custom role
3. Add trigger:
   - Select `OrderQueue`
4. Paste code from [lambda_function.py]

### Step 5 - Test the Flow
1. Publish test JSON message to SNS:
{
  "orderId": "O1234",
  "userId": "U123",
  "itemName": "Laptop",
  "quantity": 1,
  "status": "new",
  "timestamp": "2025-05-03T12:00:00Z"
}'

## Architecture Diagram (Text)

```text
+-------------+       +-------------+       +-------------+       +-------------+
|  Publisher  | --->  |   SNS Topic | --->  |   SQS Queue | --->  |   Lambda     |
| (Test Order)|       | (OrderTopic)|       | (OrderQueue)|       |   Function   |
+-------------+       +-------------+       +-------------+       +-------------+
                                                                   |
                                                                   v
                                                           +---------------+
                                                           |  DynamoDB     |
                                                           |   Orders Table|
                                                           +---------------+

[Dead-Letter Queue (DLQ) attached to OrderQueue with maxReceiveCount = 3]


