# AWS-assignment
An AWS assignment


# S3 HTTP Service

## Overview

This project provides an HTTP service that lists the contents of an S3 bucket. The service is developed using Python with Flask and deployed using Terraform on AWS. The service exposes an endpoint that returns the content of an S3 bucket for a specified path in JSON format.

## Features

- Lists the contents of an S3 bucket.
- Supports listing contents for specified paths.
- Handles errors gracefully for invalid or non-existing paths.

## Prerequisites

- **Python** and **pip** installed on your machine.
- **AWS CLI** installed and configured with your AWS credentials.
- **Terraform** installed.

## Setup Instructions

### 1. Local Setup

1. **Clone the repository**:
   ```bash
   git clone <your-repo-url>
   cd s3-http-service
   ```

2. **Install dependencies**:
   ```bash
   sudo yum install python-pip -y
   pip install flask boto3
   ```

3. **Configure AWS CLI**:
   ```bash
   aws configure
   ```

4. **Create and run the Flask application**:
   ```bash
   sudo nano app.py
   ```
   Paste the following code:
   ```python
   from flask import Flask, jsonify
   import boto3

   app = Flask(__name__)

   s3 = boto3.client('s3')
   BUCKET_NAME = 'your-bucket-name'

   @app.route('/list-bucket-content/', defaults={'path': ''})
   @app.route('/list-bucket-content/<path:path>')
   def list_bucket_content(path):
       try:
           response = s3.list_objects_v2(Bucket=BUCKET_NAME, Prefix=path)
           contents = [item['Key'] for item in response.get('Contents', [])]
           return jsonify({"content": contents})
       except Exception as e:
           return jsonify({"error": str(e)}), 400

   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   ```
   Replace `your-bucket-name` with your actual S3 bucket name.

5. **Run the application**:
   ```bash
   python app.py
   ```

### 2. Deployment with Terraform

1. **Navigate to your Terraform directory**:
   ```bash
   mkdir terraform-deployment
   cd terraform-deployment
   ```

2. **Create `main.tf`**:
   ```bash
   sudo nano main.tf
   ```
   Paste the following code:
   ```hcl
   provider "aws" {
     region = "us-west-2"
   }

   resource "aws_instance" "web" {
     ami           = "ami-0c55b159cbfafe1f0" # Use an appropriate AMI ID for your region
     instance_type = "t2.micro"

     tags = {
       Name = "HTTP Service"
     }

     provisioner "remote-exec" {
       inline = [
         "sudo yum update -y",
         "sudo yum install -y python3",
         "sudo yum install -y python3-pip",
         "pip3 install flask boto3",
         "echo 'from flask import Flask, jsonify' > app.py",
         "echo 'import boto3' >> app.py",
         "echo 'app = Flask(__name__)' >> app.py",
         "echo 's3 = boto3.client(\"s3\")' >> app.py",
         "echo 'BUCKET_NAME = \"your-bucket-name\"' >> app.py",
         "echo '@app.route(\"/list-bucket-content/\", defaults={\"path\": \"\"})' >> app.py",
         "echo '@app.route(\"/list-bucket-content/<path:path>\")' >> app.py",
         "echo 'def list_bucket_content(path):' >> app.py",
         "echo 'try:' >> app.py",
         "echo 'response = s3.list_objects_v2(Bucket=BUCKET_NAME, Prefix=path)' >> app.py",
         "echo 'contents = [item[\"Key\"] for item in response.get(\"Contents\", [])]' >> app.py",
         "echo 'return jsonify({\"content\": contents})' >> app.py",
         "echo 'except Exception as e:' >> app.py",
         "echo 'return jsonify({\"error\": str(e)}), 400' >> app.py",
         "echo 'app.run(host=\"0.0.0.0\", port=5000)' >> app.py",
         "python3 app.py &"
       ]
     }
   }

   resource "aws_security_group" "web_sg" {
     name        = "web_sg"
     description = "Allow inbound traffic on port 5000"

     ingress {
       from_port   = 5000
       to_port     = 5000
       protocol    = "tcp"
       cidr_blocks = ["0.0.0.0/0"]
     }

     egress {
       from_port   = 0
       to_port     = 0
       protocol    = "-1"
       cidr_blocks = ["0.0.0.0/0"]
     }
   }

   resource "aws_s3_bucket" "bucket" {
     bucket = "your-bucket-name"
   }
   ```

3. **Initialize Terraform and apply**:
   ```bash
   terraform init
   terraform apply
   ```

## Testing the Service

1. **Make a GET request** to list the bucket contents:
   ```bash
   curl http://<instance-ip>:5000/list-bucket-content
   ```
   Replace `<instance-ip>` with the public IP of your EC2 instance.

2. **Check different paths**:
   ```bash
   curl http://<instance-ip>:5000/list-bucket-content/dir1
   ```

## Cleaning Up

1. **Terminate the EC2 instance** and **delete the S3 bucket**:
   ```bash
   terraform destroy
   ```

## Bonus Points

- **Secure the service with HTTPS**:
  - Use AWS Certificate Manager (ACM) to create a certificate.
  - Configure an Elastic Load Balancer (ELB) with the certificate.

- **Provide a comprehensive video demo** of the working solution, along with screenshots of the S3 bucket structure and API responses.

## Design Decisions

- Used Python with Flask for simplicity and quick development.
- Deployed the service on an EC2 instance using Terraform.
- Configured security groups to allow inbound traffic on port 5000.
- Provisioned an S3 bucket for storing and retrieving files.

## Challenges Faced

- Ensuring the correct configuration of AWS resources using Terraform.
- Handling exceptions and providing informative error messages.

---
