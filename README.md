# Fraud Detection with AWS and Hazelcast Cloud

## Overview
This repository contains the implementation of a fraud detection system that uses AWS services and Hazelcast Cloud. The system validates transactions against stored data to identify anomalies, ensuring enhanced security for financial operations.

The following steps detail how to set up and execute the system.

---

## Prerequisites
- An active AWS account
- Basic understanding of AWS Lambda, API Gateway, and S3
- Node.js installed on your system
- AWS CLI installed and configured
- Access to Hazelcast Cloud

---

## Step-by-Step Guide

### 1. Setting Up the Project Files

1. Clone the GitHub repository:
   ```bash
   git clone https://github.com/hazelcast-guides/serverless-fraud-detection.git
   ```
2. Navigate to the `code/` directory and install dependencies:
   ```bash
   npm install
   ```
3. Download the TLS certificate files from the Hazelcast Cloud console and save them in the `code/` directory.

---

### 2. Configuring AWS CLI

1. Install the AWS CLI.
2. Configure the CLI:
   ```bash
   aws configure
   ```
   Provide your AWS credentials and set the default region to `us-west-2`.
3. Verify the setup by listing S3 buckets:
   ```bash
   aws s3 ls
   ```

---

### 3. Creating a Hazelcast Viridian Cluster

1. Log in to the [Hazelcast Cloud console](https://cloud.hazelcast.com).
2. Create a new cluster in the `us-west-2` region.
3. Note the cluster name and discovery token for later use.

---

### 4. Creating an S3 Bucket and IAM Roles

1. Create an S3 bucket to store airport data:
   ```bash
   aws s3api create-bucket --bucket hazelcast-fraud-bucket-12345 \
       --region us-west-2 --create-bucket-configuration LocationConstraint=us-west-2
   ```
2. Create an IAM role for Lambda with the necessary permissions.
3. Attach the role to the Lambda functions.

---

### 5. Ingesting Airport Data

1. Create a Lambda function named `ImportAirportsFn` to read data from S3 and write to the Hazelcast cluster.
2. Zip the required files:
   ```bash
   zip -r import.zip import.js hazelcast.js node_modules ca.pem cert.pem key.pem
   ```
3. Deploy the Lambda function:
   ```bash
   aws lambda create-function --function-name ImportAirportsFn \
       --role <ROLE_ARN> --zip-file fileb://import.zip --handler import.handle \
       --runtime nodejs16.x --region us-west-2
   ```
4. Set up S3 triggers to invoke the function on file uploads.
5. Upload the `airports.json` file to the S3 bucket:
   ```bash
   aws s3 cp airports.json s3://hazelcast-fraud-bucket-12345/
   ```

---

### 6. Validating Transactions

1. Develop the fraud detection logic in `validate.js`.
2. Zip the files and deploy the `ValidateFn` Lambda function:
   ```bash
   zip -r validate.zip validate.js hazelcast.js node_modules ca.pem cert.pem key.pem
   aws lambda create-function --function-name ValidateFn \
       --role <ROLE_ARN> --zip-file fileb://validate.zip --handler validate.handle \
       --runtime nodejs16.x --region us-west-2
   ```
3. Test the function with sample data in the AWS Lambda console.

---

### 7. Creating the API Gateway Endpoint

1. Create a REST API in API Gateway named `ServerlessFraudDetectionGateway`.
2. Add a resource `/ValidateTransactions` with a POST method linked to `ValidateFn`.
3. Deploy the API to a stage named `Test`.
4. Test the API endpoint using tools like `httpie` or Postman.

---

### 8. Testing the System

1. Send valid and invalid transactions to the API endpoint.
2. Confirm the system’s responses.
3. Monitor CloudWatch logs for debugging and performance tracking.

---

## Common Issues and Solutions

1. **Permission Denied Errors**:
   - Ensure the IAM role has `AmazonS3FullAccess`, `AWSLambdaFullAccess`, and `IAMFullAccess` policies attached.

2. **Missing Authentication Token**:
   - Verify the API Gateway endpoint and ensure it is deployed to a stage.

3. **Syntax Errors in `validate.js`**:
   - Ensure all `await` statements are inside `async` functions.( it's working in this repo no worries !! )

4. **Data Not Loaded into Hazelcast**:
   - Check S3 triggers and CloudWatch logs for issues.

---

## Conclusion
This project demonstrates a scalable approach to fraud detection using AWS and Hazelcast Cloud. The guide ensures smooth implementation while highlighting potential pitfalls and their solutions. With the flexibility of this architecture, you can extend the system to handle more complex fraud detection scenarios.

For more information, refer to the [official Hazelcast tutorial](https://docs.hazelcast.com/tutorials/serverless-fraud-detection).
