
# 📈 EC2 Auto Upgrade using Lambda, SNS, and CloudWatch

This project demonstrates how to automatically upgrade an EC2 instance from a `t2.micro` to a `t3.micro` when CPU utilization exceeds 90%. It uses **AWS Lambda**, **IAM Roles**, **SNS**, and **CloudWatch Alarms** to automate the process.

---

## 🛠️ Prerequisites

- AWS CLI configured
- IAM user with necessary permissions
- EC2 key pair
- Basic knowledge of AWS services

---

## 🔧 Step-by-Step Setup

### 1. Create an EC2 Instance

- Launch a `t2.micro` EC2 instance via AWS Console or CLI.
- Ensure:
  - The instance is in a VPC.
  - Monitoring is set to "detailed".
  - A key pair is selected for access.

---

### 2. Create an IAM Role for Lambda (LambdaEC2AccessRole)

- Attach the following AWS managed policies:
  - `AmazonEC2FullAccess`
  - `AWSLambdaBasicExecutionRole`
  - `CloudWatchFullAccess`
- Trust relationship policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}


### 3. Create a Lambda Function

- **Runtime**: Python 3.x  
- **Permissions**: Attach the IAM role created in Step 2  
- **Function logic**: Stops the instance, modifies the instance type to `t3.micro`, and restarts it.

```python
import boto3

INSTANCE_ID = 'i-0abc12345def67890'  # Replace with your instance ID
NEW_INSTANCE_TYPE = 't3.micro'       # Replace with your target instance type

ec2 = boto3.client('ec2')

def lambda_handler(event, context):
    # Stop instance
    print(f"Stopping instance: {INSTANCE_ID}")
    ec2.stop_instances(InstanceIds=[INSTANCE_ID])
    waiter = ec2.get_waiter('instance_stopped')
    waiter.wait(InstanceIds=[INSTANCE_ID])
    
    # Modify instance type
    print(f"Modifying instance type to {NEW_INSTANCE_TYPE}")
    ec2.modify_instance_attribute(
        InstanceId=INSTANCE_ID,
        InstanceType={'Value': NEW_INSTANCE_TYPE}
    )

    # Start instance
    print(f"Starting instance: {INSTANCE_ID}")
    ec2.start_instances(InstanceIds=[INSTANCE_ID])

    return {
        'statusCode': 200,
        'body': f'Instance {INSTANCE_ID} changed to {NEW_INSTANCE_TYPE} and restarted.'
    }


### 4. Create an SNS Topic

- **Name**: `HighCPUAlarmTopic`
- Add the Lambda function as a **subscriber**
- Ensure the Lambda has **SNS trigger enabled**

---

### 5. Create a CloudWatch Alarm

- **Metric**: `CPUUtilization`
- **Threshold**: >90% for 5 minutes
- **Action**: Send a notification to `HighCPUAlarmTopic`

---

## ✅ Outcome

When the EC2 instance exceeds **90% CPU usage**, CloudWatch triggers an **SNS notification**, which invokes the **Lambda function** to upgrade the instance type from `t2.micro` to `t3.micro`.
