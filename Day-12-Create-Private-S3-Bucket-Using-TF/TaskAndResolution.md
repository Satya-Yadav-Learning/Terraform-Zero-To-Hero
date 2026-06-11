# TaskAndResolution.md

# Terraform Task: Create Private S3 Bucket Using Terraform

---

# Task Objective

A cloud engineering team required a secure Amazon S3 bucket to store migration-related data. The bucket needed to remain private and block all forms of public access.

The solution was implemented entirely using Terraform.

---

# Requirements

* Create an S3 bucket named `nautilus-s3-21130`
* Create resources in `us-east-1`
* Use Terraform only
* Configure the bucket as private
* Block all public access
* Store configuration in `main.tf`

---

# Solution Architecture

```text
Terraform
    │
    ▼
Amazon S3 Bucket
    │
    ▼
Public Access Block
    │
    ├── BlockPublicAcls       = true
    ├── IgnorePublicAcls      = true
    ├── BlockPublicPolicy     = true
    └── RestrictPublicBuckets = true
    │
    ▼
Private Bucket
```

---

# Terraform Configuration (main.tf)

```hcl
resource "aws_s3_bucket" "private_bucket" {
  bucket = "nautilus-s3-21130"
}

resource "aws_s3_bucket_public_access_block" "private_access" {
  bucket = aws_s3_bucket.private_bucket.id

  block_public_acls       = true
  ignore_public_acls      = true
  block_public_policy     = true
  restrict_public_buckets = true
}
```

---

# Environment Verification

## Verify Working Directory

### Command

```bash
pwd
```

### Output

```text
/home/bob/terraform
```

### Explanation

Confirms Terraform files are created in the correct working directory.

---

## Verify Existing Files

### Command

```bash
ls -ltr
```

### Output

```text
total 8
-rw-rw-r-- 1 engineer engineer 1116 provider.tf
-rw-rw-r-- 1 engineer engineer  435 README.MD
```

### Explanation

Verifies the Terraform provider configuration already exists.

---

## Verify Provider Configuration

### Command

```bash
cat provider.tf
```

### Output

```hcl
provider "aws" {
  region = "us-east-1"

  skip_credentials_validation = true
  skip_requesting_account_id  = true

  endpoints {
    s3 = "http://aws:4566"
  }
}
```

### Explanation

Confirms:

* AWS Provider configured
* Region configured as us-east-1
* LocalStack endpoint configured

No additional provider block required.

---

# Terraform Initialization

## Command

```bash
terraform init
```

### Output

```text
Terraform has been successfully initialized!
```

### Explanation

Initializes:

* Terraform plugins
* AWS provider
* Terraform working directory

Creates:

```text
.terraform/
.terraform.lock.hcl
```

---

# Terraform Validation

## Command

```bash
terraform validate
```

### Output

```text
Success! The configuration is valid.
```

### Explanation

Validates Terraform syntax and resource definitions.

---

# Terraform Planning

## Command

```bash
terraform plan
```

### Output

```text
Plan: 2 to add, 0 to change, 0 to destroy.
```

### Planned Resources

```text
aws_s3_bucket.private_bucket
aws_s3_bucket_public_access_block.private_access
```

### Explanation

Terraform calculates infrastructure changes before deployment.

---

# Terraform Deployment

## Command

```bash
terraform apply
```

### Approval

```text
yes
```

### Output

```text
aws_s3_bucket.private_bucket: Creation complete

aws_s3_bucket_public_access_block.private_access: Creation complete

Apply complete!
Resources: 2 added, 0 changed, 0 destroyed.
```

### Explanation

Successfully created:

1. Private S3 Bucket
2. Public Access Block Configuration

---

# Verification Steps

---

## Verify Terraform Resources

### Command

```bash
terraform state list
```

### Output

```text
aws_s3_bucket.private_bucket
aws_s3_bucket_public_access_block.private_access
```

### Explanation

Confirms Terraform manages all required resources.

---

## Verify Bucket Exists

### Command

```bash
aws s3 ls
```

### Output

```text
2026-06-11 10:15:00 nautilus-s3-21130
```

### Explanation

Confirms bucket exists.

---

## Verify Public Access Block

### Command

```bash
aws s3api get-public-access-block --bucket nautilus-s3-21130
```

### Output

```json
{
  "PublicAccessBlockConfiguration": {
    "BlockPublicAcls": true,
    "IgnorePublicAcls": true,
    "BlockPublicPolicy": true,
    "RestrictPublicBuckets": true
  }
}
```

### Explanation

Confirms all public access is blocked.

---

## Verify Bucket Resource

### Command

```bash
terraform state show aws_s3_bucket.private_bucket
```

### Output

```text
bucket = "nautilus-s3-21130"
region = "us-east-1"
arn    = "arn:aws:s3:::nautilus-s3-21130"
```

### Explanation

Confirms bucket exists in correct region.

---

## Verify Public Access Block Resource

### Command

```bash
terraform state show aws_s3_bucket_public_access_block.private_access
```

### Output

```text
block_public_acls       = true
ignore_public_acls      = true
block_public_policy     = true
restrict_public_buckets = true
```

### Explanation

Confirms all public access restrictions are enabled.

---

# Security Analysis

## Why This Bucket Is Private

Public access is prevented by enabling:

```text
BlockPublicAcls
IgnorePublicAcls
BlockPublicPolicy
RestrictPublicBuckets
```

Even if someone attempts to:

* Add public ACLs
* Add public bucket policies

AWS will block access.

---

# Public vs Private Bucket Comparison

| Setting               | Public Bucket | Private Bucket |
| --------------------- | ------------- | -------------- |
| ACL                   | public-read   | private        |
| BlockPublicAcls       | false         | true           |
| IgnorePublicAcls      | false         | true           |
| BlockPublicPolicy     | false         | true           |
| RestrictPublicBuckets | false         | true           |
| Internet Access       | Allowed       | Blocked        |

---

# Root Cause Analysis Scenario

## Incident

Sensitive files became publicly accessible.

### Impact

Confidential data exposed.

### Root Cause

Public access block configuration disabled.

```text
BlockPublicPolicy = false
BlockPublicAcls  = false
```

### Resolution

Enabled:

```text
BlockPublicAcls       = true
IgnorePublicAcls      = true
BlockPublicPolicy     = true
RestrictPublicBuckets = true
```

### Preventive Actions

* AWS Config Rules
* Security Hub
* Terraform Policy Validation
* SCP Enforcement

---

# Scenario Based Interview Questions & Answers

# L1 – Junior Engineer

## Q1. What is Amazon S3?

### Answer

Amazon S3 is an object storage service used to store files, backups, logs, images, and application data.

---

## Q2. What is a bucket?

### Answer

A bucket is a logical container that stores objects in Amazon S3.

---

## Q3. What is Terraform?

### Answer

Terraform is an Infrastructure as Code (IaC) tool used to provision and manage cloud resources through code.

---

## Q4. Why block public access?

### Answer

To prevent unauthorized internet access to sensitive data.

---

# L2 – Mid-Level Engineer

## Q1. What is Public Access Block?

### Answer

A security feature that prevents public exposure of S3 buckets.

---

## Q2. Difference between ACL and Public Access Block?

### Answer

ACL grants permissions.

Public Access Block can override ACL permissions and prevent public access.

---

## Q3. Why use Terraform for S3?

### Answer

Benefits:

* Automation
* Repeatability
* Version Control
* Auditability

---

# L3 – Senior Engineer

## Q1. How would you secure enterprise S3 buckets?

### Answer

* Block Public Access
* Enable Versioning
* Enable Encryption
* Enable Logging
* Use Bucket Policies
* Enable AWS Config

---

## Q2. How would you identify public buckets?

### Answer

Using:

* AWS Config
* Security Hub
* Trusted Advisor
* IAM Access Analyzer

---

## Q3. How would you enforce standards?

### Answer

* Terraform Modules
* SCPs
* AWS Config Rules
* CI/CD Validation

---

# L4 – Architect Level

## Q1. Design a secure enterprise object storage architecture.

### Answer

```text
Applications
      │
      ▼
Private S3 Bucket
      │
      ├── Encryption
      ├── Versioning
      ├── Logging
      ├── Lifecycle Policies
      └── Public Access Block
```

---

## Q2. How would you manage thousands of buckets?

### Answer

* Centralized Terraform Modules
* Tagging Strategy
* SCP Enforcement
* Security Hub Monitoring
* Automated Compliance Checks

---

# AWS Scenario Based Questions

## Scenario 1

A bucket unexpectedly becomes public.

### Investigation

```bash
aws s3api get-bucket-policy
aws s3api get-bucket-acl
aws s3api get-public-access-block
```

Check:

* Bucket policy
* ACL
* Public access settings

---

## Scenario 2

Users cannot access bucket objects.

### Investigation

Check:

* IAM Policies
* Bucket Policy
* SCP Restrictions
* Encryption Permissions

---

# Real AWS Incident RCA

## Incident

A cloud storage bucket containing application backups became publicly accessible.

### Impact

Sensitive data exposure risk.

### Root Cause

Public access block disabled during maintenance.

### Resolution

Enabled:

```text
BlockPublicAcls       = true
IgnorePublicAcls      = true
BlockPublicPolicy     = true
RestrictPublicBuckets = true
```

### Lessons Learned

* Always enable Public Access Block.
* Continuously monitor bucket permissions.
* Automate security compliance.

---

# Full DevOps Mock Interview

## Q1. Explain Terraform Workflow

### Answer

```text
terraform init
terraform validate
terraform plan
terraform apply
terraform destroy
```

---

## Q2. What is Terraform State?

### Answer

Terraform state tracks infrastructure resources managed by Terraform.

Stored in:

```text
terraform.tfstate
```

---

## Q3. Why use Remote State?

### Answer

Benefits:

* Collaboration
* Locking
* Recovery
* Versioning

---

## Q4. What is Infrastructure as Code?

### Answer

Managing infrastructure using code instead of manual configuration.

Benefits:

* Automation
* Consistency
* Auditability
* Reusability

---

## Q5. Explain Terraform Resource Lifecycle

### Answer

```text
Create
Read
Update
Delete
```

Known as CRUD operations.

---

# Key Learnings

1. S3 buckets should be private by default.
2. Public Access Block provides an additional security layer.
3. Terraform enables repeatable infrastructure deployment.
4. Security controls should be enforced through code.
5. Verification steps are critical after deployment.
6. Infrastructure as Code improves compliance and governance.

---

# Final Result

```text
Task Status : SUCCESS

Bucket Name : nautilus-s3-21130
Region      : us-east-1

Public Access Block:
  BlockPublicAcls       = true
  IgnorePublicAcls      = true
  BlockPublicPolicy     = true
  RestrictPublicBuckets = true

Terraform Validation : Passed
Terraform Apply      : Successful
Verification         : Successful
Challenge Status     : PASSED
```

