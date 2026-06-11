# TaskAndResolution.md

# Terraform Task: Create Public S3 Bucket Using Terraform

---

# Task Objective

As part of the cloud migration initiative, a public Amazon S3 bucket needed to be provisioned using Terraform.

### Requirements

* Create a public S3 bucket named `datacenter-s3-30573`
* Use Terraform only
* Use existing AWS provider configuration
* Create resources in `us-east-1`
* Ensure bucket is publicly accessible
* Configure proper ACL permissions
* Store Terraform code in `main.tf`

---

# Solution Architecture

```text
Terraform
    │
    ▼
AWS S3 Bucket
    │
    ├── Public ACL (public-read)
    │
    └── Public Access Block Disabled
```

---

# Terraform Configuration (main.tf)

```hcl
resource "aws_s3_bucket" "datacenter_bucket" {
  bucket = "datacenter-s3-30573"
}

resource "aws_s3_bucket_public_access_block" "public_access" {
  bucket = aws_s3_bucket.datacenter_bucket.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

resource "aws_s3_bucket_acl" "datacenter_bucket_acl" {
  depends_on = [
    aws_s3_bucket.datacenter_bucket,
    aws_s3_bucket_public_access_block.public_access
  ]

  bucket = aws_s3_bucket.datacenter_bucket.id
  acl    = "public-read"
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

Confirms Terraform code is being created in the correct working directory.

---

## Verify Existing Files

### Command

```bash
ls -ltr
```

### Output

```text
total 8
-rw-rw-r-- 1 bob bob 1116 May 13 2025 provider.tf
-rw-rw-r-- 1 bob bob 435 Jun 19 2025 README.MD
```

### Explanation

Verifies provider configuration already exists.

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
      ...
  }
}
```

### Explanation

Confirms:

* AWS Provider configured
* Region set to us-east-1
* LocalStack endpoint configured

No additional provider block required in main.tf.

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

Downloads and initializes:

* AWS Provider
* Terraform plugins
* State backend

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

Validates Terraform syntax before deployment.

---

# Terraform Planning

## Command

```bash
terraform plan
```

### Output

```text
Plan: 3 to add, 0 to change, 0 to destroy.
```

### Resources Planned

```text
aws_s3_bucket.datacenter_bucket
aws_s3_bucket_public_access_block.public_access
aws_s3_bucket_acl.datacenter_bucket_acl
```

### Explanation

Terraform calculates changes before execution.

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
aws_s3_bucket.datacenter_bucket: Creation complete

aws_s3_bucket_public_access_block.public_access: Creation complete

aws_s3_bucket_acl.datacenter_bucket_acl: Creation complete

Apply complete!
Resources: 3 added, 0 changed, 0 destroyed.
```

### Explanation

Successfully created:

1. S3 Bucket
2. Public Access Configuration
3. Public ACL

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
aws_s3_bucket.datacenter_bucket
aws_s3_bucket_acl.datacenter_bucket_acl
aws_s3_bucket_public_access_block.public_access
```

### Explanation

Confirms Terraform manages all required resources.

---

## Verify Public ACL

### Command

```bash
aws s3api get-bucket-acl --bucket datacenter-s3-30573
```

### Output

```json
{
  "Grants": [
    {
      "Permission": "FULL_CONTROL"
    },
    {
      "URI": "http://acs.amazonaws.com/groups/global/AllUsers",
      "Permission": "READ"
    }
  ]
}
```

### Explanation

Confirms public read access is granted.

---

## Verify Public Access Block Configuration

### Command

```bash
aws s3api get-public-access-block --bucket datacenter-s3-30573
```

### Output

```json
{
  "PublicAccessBlockConfiguration": {
    "BlockPublicAcls": false,
    "IgnorePublicAcls": false,
    "BlockPublicPolicy": false,
    "RestrictPublicBuckets": false
  }
}
```

### Explanation

Ensures AWS does not block public access.

---

## Verify Bucket Resource

### Command

```bash
terraform state show aws_s3_bucket.datacenter_bucket
```

### Output

```text
bucket = "datacenter-s3-30573"
region = "us-east-1"
arn    = "arn:aws:s3:::datacenter-s3-30573"
```

### Explanation

Confirms bucket exists in correct region.

---

## Verify ACL Resource

### Command

```bash
terraform state show aws_s3_bucket_acl.datacenter_bucket_acl
```

### Output

```text
acl = "public-read"
```

### Explanation

Confirms ACL configuration.

---

## Verify Public Access Block Resource

### Command

```bash
terraform state show aws_s3_bucket_public_access_block.public_access
```

### Output

```text
block_public_acls       = false
ignore_public_acls      = false
block_public_policy     = false
restrict_public_buckets = false
```

### Explanation

Confirms public access is fully enabled.

---

# Root Cause Analysis (Previous Failure)

## Problem

Validator reported:

```text
public s3 bucket doesn't exist
```

## Cause

Earlier implementation created:

```text
S3 Bucket
ACL
```

But omitted:

```text
Public Access Block Configuration
```

Result:

```text
ACL = Public
BUT
Public Access Block still enforced
```

Validator considered bucket non-public.

## Resolution

Added:

```hcl
resource "aws_s3_bucket_public_access_block"
```

and disabled all blocking settings.

Task passed successfully.

---

# Scenario Based Interview Questions & Answers

# L1 – Junior Engineer

## Q1. What is Terraform?

### Answer

Terraform is an Infrastructure as Code (IaC) tool developed by HashiCorp that allows infrastructure provisioning using declarative configuration files.

---

## Q2. What is an S3 bucket?

### Answer

An S3 bucket is a globally unique container used to store objects within Amazon S3.

---

## Q3. What is ACL?

### Answer

ACL stands for Access Control List.

It defines permissions for bucket access.

Example:

```text
private
public-read
public-read-write
```

---

## Q4. What is public-read?

### Answer

Allows anonymous users to read bucket objects.

---

# L2 – Mid-Level Engineer

## Q1. Difference between ACL and Bucket Policy?

### Answer

ACL

* Object/Bucket level permissions
* Limited granularity

Bucket Policy

* JSON-based permissions
* More flexible
* Supports conditions

---

## Q2. Why was ACL alone insufficient?

### Answer

AWS Public Access Block settings can override ACL permissions.

Therefore:

```text
ACL + Public Access Block Configuration
```

are both required.

---

## Q3. What does depends_on do?

### Answer

Forces Terraform dependency ordering.

Example:

```hcl
depends_on = [
  aws_s3_bucket_public_access_block.public_access
]
```

---

# L3 – Senior Engineer

## Q1. How would you secure public buckets?

### Answer

* Use bucket policies
* Enable CloudTrail
* Enable access logging
* Enable encryption
* Restrict access via conditions

---

## Q2. How would you detect public buckets?

### Answer

Using:

* AWS Config
* Security Hub
* Trusted Advisor
* Custom Lambda checks

---

## Q3. How would you enforce organization-wide standards?

### Answer

* SCPs
* Terraform Modules
* AWS Config Rules
* CI/CD policy validation

---

# L4 – Architect Level

## Q1. Design secure public content hosting architecture.

### Answer

```text
Users
  │
CloudFront
  │
Origin Access Control
  │
Private S3 Bucket
```

Avoid direct public buckets whenever possible.

---

## Q2. How would you manage thousands of buckets?

### Answer

* Terraform modules
* Tagging strategy
* AWS Organizations
* Centralized logging
* Security Hub

---

# AWS Scenario Based Questions

## Scenario 1

Public website hosted on S3 becomes inaccessible.

### Troubleshooting

1. Verify bucket policy
2. Verify ACL
3. Verify Public Access Block
4. Verify CloudFront
5. Check Route53

---

## Scenario 2

Users receive AccessDenied.

### Investigation

```bash
aws s3api get-bucket-policy
aws s3api get-bucket-acl
```

Check:

* Bucket policy
* IAM permissions
* SCP restrictions

---

# Real AWS Incident RCA

## Incident

Public static website unavailable.

### Impact

Customers unable to access application.

### Root Cause

Security team enabled:

```text
BlockPublicAcls = true
```

ACL became ineffective.

### Resolution

Disabled public access block and migrated to CloudFront with Origin Access Control.

### Preventive Measures

* Infrastructure testing
* CI/CD validation
* AWS Config monitoring

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

State tracks infrastructure resources managed by Terraform.

Stored in:

```text
terraform.tfstate
```

---

## Q3. Why use Remote State?

### Answer

Benefits:

* Team collaboration
* Locking
* Versioning
* Recovery

Example:

```text
S3 + DynamoDB Locking
```

---

## Q4. What is Infrastructure as Code?

### Answer

Managing infrastructure using code instead of manual configuration.

Benefits:

* Consistency
* Automation
* Version Control
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

1. Public ACL alone may not make bucket public.
2. Public Access Block settings can override ACLs.
3. Terraform state is critical for tracking resources.
4. Verification commands should always be executed after deployment.
5. Infrastructure as Code improves consistency and repeatability.
6. Security validation is mandatory for public cloud resources.

---

# Final Result

```text
Task Status : SUCCESS

Bucket Name : datacenter-s3-30573
Region      : us-east-1
ACL         : public-read
Public Access Block : Disabled
Terraform Validation : Passed
Terraform Apply : Successful
Verification : Successful
Challenge Status : PASSED
```

