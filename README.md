
# Static Website Hosting on AWS S3 bucket using Terraform

## What's on the agenda?

We'll be deploying a static website to an AWS S3 bucket using Terraform, and it can be achieved in a straightforward 5-step process. All tasks will be performed exclusively through Terraform, utilizing the terraform-s3 approach.

## Prerequisites

Before starting the deployment process, ensure the following prerequisites are met:

1. **AWS Account:** You should have an AWS account. If you don't have one, you can create it [here](https://aws.amazon.com/).

2. **AWS CLI Configuration:** Make sure you have the AWS CLI installed and configured with the necessary access credentials. You can configure the AWS CLI using the following command:

   ```bash
   aws configure

## Step 1: Provider File

Begin by creating a provider.tf file, where you specify your provider (AWS in this case) and include the necessary access keys for AWS console access.

```bash
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.33.0"
    }
  }
}

provider "aws" {
  # Configuration options
  region = var.region
}
```

## Step 2: Variable and tfvars File

Avoid hardcoding variables in the provider file. Declare variable names and their types in a variable.tf file (with optional default values). Use a terraform.tfvars file to store the variable values.
> variable.tf
```bash
#varible are declared here
variable "region" {
    type = string
}

variable "s3_bucket" {
    type = string
}
```
> terraform.tfvars
```bash
#values of varible are declared here
region = "us-east-1"
s3_bucket = "mystaticwebsitehostingbucket"
```

## Step 3: main file

Create a main.tf file containing all the resources and their configurations, such as creating an S3 bucket, changing ownership, making it publicly accessible, enabling access control, and creating S3 objects.

> creating s3 bucket
```bash
#create S3 bucket
resource "aws_s3_bucket" "mybucket" {
  bucket = var.s3_bucket

  tags = {
    Name        = "My bucket"
    Environment = "Dev"
  }
}
```

> changing ownership 
```bash
resource "aws_s3_bucket_ownership_controls" "mybucket" {
  bucket = aws_s3_bucket.mybucket.id

  rule {
    object_ownership = "BucketOwnerPreferred"
  }
}
```

> making it publicly accessible
```bash
resource "aws_s3_bucket_public_access_block" "mybucket" {
  bucket = aws_s3_bucket.mybucket.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}
```

> enabling access control to public-read for s3
```bash
resource "aws_s3_bucket_acl" "mybucket" {
  depends_on = [
    aws_s3_bucket_ownership_controls.mybucket,
    aws_s3_bucket_public_access_block.mybucket,
  ]

  bucket = aws_s3_bucket.mybucket.id
  acl    = "public-read"
}
```

>  create s3 objects index.html, error.html and profile.png which is required for website hosting
> [Click to see index.html and error.html](https://github.com/vasugupta32/Terraform-Project-Static-Hosting/tree/main)

```bash
resource "aws_s3_object" "index" {
    bucket = aws_s3_bucket.mybucket.id
    key = "index.html"
    source = "index.html"
    acl = "public-read"
    content_type = "text/html"
}

resource "aws_s3_object" "error" {
    bucket = aws_s3_bucket.mybucket.id
    key = "error.html"
    source = "error.html"
    acl = "public-read"
    content_type = "text/html"
}

resource "aws_s3_object" "profile" {
    bucket = aws_s3_bucket.mybucket.id
    key = "profile.png"
    source = "profile.png"
    acl = "public-read"
}
```

> adding aws S3 bucket website configuration
```bash
resource "aws_s3_bucket_website_configuration" "example" {
  bucket = aws_s3_bucket.mybucket.id

  index_document {
    suffix = "index.html"
  }

  error_document {
    key = "error.html"
  }

  depends_on = [ aws_s3_bucket_acl.mybucket ]
}
```

## Step 4: Run Terraform Commands
Execute the following Terraform commands:
```bash
terraform init - Initialize the Terraform backend.
terraform plan - Preview changes that Terraform plans to make.
terraform apply - Execute the proposed actions.
```

* terraform init - Initialize the Terraform backend.
![image](https://github.com/vasugupta32/Terraform-Project-Static-Hosting/assets/51460014/c9ef3e83-62cb-46f0-bfe1-f271d2e7f387)

* terraform plan - Preview changes that Terraform plans to make.
![image](https://github.com/vasugupta32/Terraform-Project-Static-Hosting/assets/51460014/70c9b469-b54a-496f-9ed5-e688863357b5)
![image](https://github.com/vasugupta32/Terraform-Project-Static-Hosting/assets/51460014/f5c97ac7-c807-4d8e-b495-5d18c5c81e96)

* terraform apply - Execute the proposed actions.
![image](https://github.com/vasugupta32/Terraform-Project-Static-Hosting/assets/51460014/b956c172-e758-4166-ba62-e29d32b12e56)
![image](https://github.com/vasugupta32/Terraform-Project-Static-Hosting/assets/51460014/c2cf9312-c533-4685-ab8c-81668d24d07b)
## Step 5:  All Done!

Check the AWS console output, inspect the bucket with its objects, and find the website link in the properties section. Click the link to view your hosted resume.
![image](https://github.com/vasugupta32/Terraform-Project-Static-Hosting/assets/51460014/044d73cf-6bb7-4ae4-9d70-a557e0a2f47b)
![image](https://github.com/vasugupta32/Terraform-Project-Static-Hosting/assets/51460014/15b0102e-298b-4d44-9d2b-99813640464c)
![image](https://github.com/vasugupta32/Terraform-Project-Static-Hosting/assets/51460014/f6a2aab2-c6b5-4339-9ea1-927e2aac5157)

## Step 6:  Terraform Destroy (Optional)
If needed, use terraform destroy to conveniently remove all remote objects managed by the Terraform configuration.
![image](https://github.com/vasugupta32/Terraform-Project-Static-Hosting/assets/51460014/45639438-f393-4b5d-b509-0dd183dac837)
![image](https://github.com/vasugupta32/Terraform-Project-Static-Hosting/assets/51460014/a2b4398d-5123-4a51-9b80-5ec4b8944902)


### Congratulations! You've successfully learned how to host a static website on AWS S3 using Terraform. 👏


## Authors

- [@vasugupta32](https://github.com/vasugupta32)


