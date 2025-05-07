* 📈 EC2 Auto Upgrade with Lambda, SNS, and CloudWatch *

  This project demonstrates how to automatically upgrade an EC2 instance from a t2.micro to a t3.micro instance when CPU utilization exceeds 90%. It uses Lambda, IAM Roles, SNS, and CloudWatch Alarms to automate the process.

  Prerequisites
AWS CLI configured
IAM user with necessary permissions
EC2 key pair
Basic understanding of AWS services

1. Create an EC2 Instance
Launch a t2.micro instance from the AWS Management Console or CLI.

Create an IAM Role for Lambda (LambdaEC2AccessRole)
Attach the following managed policies:

AmazonEC2FullAccess

CloudWatchFullAccess

AWSLambdaBasicExecutionRole

Trust relationship should allow Lambda service: 
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

3. Create a Lambda Function
Runtime: Python 3.x

Permissions: Use the IAM role created in Step 2

Function logic: Stop the instance, modify its type to t3.micro, then start it.

    import boto3

    ec2 = boto3.client('ec2')

    def lambda_handler(event, context):
  
       instance_id = 'i-xxxxxxxxxxxxxxxxx'  # Replace with your instance ID
       ec2.stop_instances(InstanceIds=[instance_id])
       waiter = ec2.get_waiter('instance_stopped')
       waiter.wait(InstanceIds=[instance_id])

       ec2.modify_instance_attribute(InstanceId=instance_id, Attribute='instanceType', Value='t3.micro')

       ec2.start_instances(InstanceIds=[instance_id])
       return {'status': 'Instance upgraded to t3.micro'}

4. Create an SNS Topic
Name: HighCPUAlarmTopic

Add Lambda function as a subscriber

Ensure the Lambda has SNS trigger enabled

5. Create a CloudWatch Alarm
Metric: CPUUtilization

Threshold: >90% for 5 minutes

Action: Send a notification to HighCPUAlarmTopic

✅ Outcome
When the EC2 instance exceeds 90% CPU usage, CloudWatch triggers an SNS notification, which invokes the Lambda function to upgrade the instance type from t2.micro to t3.micro.





