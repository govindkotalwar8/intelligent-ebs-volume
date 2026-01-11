# Intelligent EBS Volume Optimization using AWS Serverless Services

### Architecture

**Label:** `screenshots/architecture.png`

## Overview

This project implements an automated, serverless solution on AWS to identify Amazon EBS volumes of type **gp2** and convert them to **gp3** based on explicit tagging.
The workflow is fully orchestrated using AWS Step Functions and provides monitoring, logging, notifications, and an audit trail.

The solution is designed to demonstrate real-world AWS skills including event-driven architecture, IAM least privilege, workflow orchestration, and operational visibility.

---

## Objective

* Automatically identify EBS volumes of type `gp2`
* Ensure only explicitly approved volumes are modified using tags
* Convert approved volumes to `gp3`
* Maintain an audit trail of all changes
* Notify stakeholders upon successful conversion
* Provide full observability via CloudWatch

---

## Architecture

### High-Level Flow

1. A scheduled trigger (manual or EventBridge) starts the Step Function
2. Lambda scans for eligible EBS volumes
3. Identified volumes are logged to DynamoDB
4. Volumes are converted from gp2 to gp3
5. Notifications are sent using SNS

### AWS Services Used

* Amazon EC2 / EBS
* AWS Lambda
* AWS Step Functions
* Amazon DynamoDB
* Amazon SNS
* Amazon EventBridge (optional scheduling)
* Amazon CloudWatch
* AWS IAM

---

## Workflow Details

### Step 1: Filter Volumes

A Lambda function scans all EBS volumes and filters based on:

* Volume type = `gp2`
* Tag `AutoConvert=true`

Only volumes meeting both conditions are selected.

### Step 2: Log Volume Metadata

Eligible volumes are logged into DynamoDB with:

* Volume ID
* Instance ID (if attached)
* Volume size
* Previous volume type
* Region
* Timestamp

This provides a complete audit trail.

### Step 3: Convert Volumes

The workflow converts each identified volume from `gp2` to `gp3` using the EC2 ModifyVolume API.

### Step 4: Notify

After conversion, an SNS notification is sent containing:

* Volume ID
* Region
* Conversion status

---

## Step Functions State Machine

The workflow is orchestrated using AWS Step Functions with the following states:

* FilterVolumes
* LogVolumes
* ConvertVolumes
* Notify

The state machine ensures controlled execution, error visibility, and extensibility.

---

## IAM Security

The solution follows **least privilege access** principles:

* Lambda functions have scoped permissions for EC2, DynamoDB, SNS, and CloudWatch Logs
* No wildcard permissions beyond required service actions
* Step Functions execution role only allows invoking specific Lambda functions

---

## How to Execute

1. Ensure the EBS volume you want to convert has the tag:

   ```
   Key: AutoConvert
   Value: true
   ```
2. Open the Step Functions state machine
3. Click **Start execution**
4. Provide input:

   ```json
   {}
   ```
5. Monitor execution in the Step Functions console

---

## Verification Checklist

After execution, verify the following:

* Step Functions execution completes successfully
* DynamoDB contains log entries for converted volumes
* SNS notification is received
* EBS volume type changes from gp2 to gp3
* CloudWatch logs show successful Lambda execution

---


### Screenshot 1: Step Functions Execution Flow

**Label:** `screenshots/step-functions-execution.png`
Shows the full execution path with all states completed successfully.

### Screenshot 2: DynamoDB Audit Log

**Label:** `screenshots/dynamodb-log.png`
Shows items stored with volume metadata and timestamps.

### Screenshot 3: SNS Notification

**Label:** `screenshots/sns-notification.png`
Shows the email or SMS notification received after conversion.

### Screenshot 4: CloudWatch Logs

**Label:** `screenshots/cloudwatch-logs.png`
Shows Lambda execution logs with volume processing details.

---

## Real-World Considerations

* Tag-based controls prevent accidental infrastructure changes
* Idempotent volume modifications avoid repeated conversions
* The workflow can be extended with retry, wait, and verification states
* EventBridge can be used for scheduled execution

---

## Future Enhancements

* Add Wait and Verify step using DescribeVolumeModifications
* Add retry and catch logic for error handling
* Generate cost-savings reports
* Extend support to multiple AWS regions
* Infrastructure as Code using Terraform or CloudFormation
