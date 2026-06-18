# TaskAndResolution.md

# Terraform Task: Create DynamoDB Table, IAM Role and Read-Only IAM Policy

---

# Project Overview

As part of a cloud security and data governance initiative, the infrastructure team needed to provision a secure DynamoDB environment and implement fine-grained access control using AWS Identity and Access Management (IAM).

The objective was to:

* Create a DynamoDB table for application data.
* Create an IAM role for trusted AWS services.
* Create a read-only IAM policy.
* Restrict access to the specific DynamoDB table.
* Attach the policy to the IAM role.
* Manage the complete infrastructure using Terraform.

This approach ensures secure access to data while following the Principle of Least Privilege.

---

# Business Requirements

| Component           | Requirement                                         |
| ------------------- | --------------------------------------------------- |
| DynamoDB Table      | application-data-table                              |
| IAM Role            | application-read-role                               |
| IAM Policy          | dynamodb-readonly-policy                            |
| Access Type         | Read Only                                           |
| Allowed Actions     | GetItem, Scan, Query                                |
| Restriction         | Specific DynamoDB Table Only                        |
| Deployment Tool     | Terraform                                           |
| Configuration Files | main.tf, variables.tf, terraform.tfvars, outputs.tf |

---

# Solution Architecture

```text
                    Terraform
                         │
                         ▼
                DynamoDB Table
               application-data-table
                         │
                         ▼
                Read-Only IAM Policy
                         │
                         ▼
                 IAM Role Attached
                         │
                         ▼
                 Trusted AWS Service
                         │
                         ▼
                 Read Access Only
```

---

# Terraform Variables

## variables.tf

```hcl
variable "KKE_TABLE_NAME" {
  type = string
}

variable "KKE_ROLE_NAME" {
  type = string
}

variable "KKE_POLICY_NAME" {
  type = string
}
```

---

# Variable Values

## terraform.tfvars

```hcl
KKE_TABLE_NAME  = "application-data-table"
KKE_ROLE_NAME   = "application-read-role"
KKE_POLICY_NAME = "dynamodb-readonly-policy"
```

---

# Main Terraform Configuration

## main.tf

```hcl
resource "aws_dynamodb_table" "application_table" {
  name         = var.KKE_TABLE_NAME
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "id"

  attribute {
    name = "id"
    type = "S"
  }
}

resource "aws_iam_role" "application_role" {
  name = var.KKE_ROLE_NAME

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"

        Principal = {
          Service = "ec2.amazonaws.com"
        }

        Action = "sts:AssumeRole"
      }
    ]
  })
}

resource "aws_iam_policy" "readonly_policy" {
  name = var.KKE_POLICY_NAME

  policy = jsonencode({
    Version = "2012-10-17"

    Statement = [
      {
        Effect = "Allow"

        Action = [
          "dynamodb:GetItem",
          "dynamodb:Scan",
          "dynamodb:Query"
        ]

        Resource = aws_dynamodb_table.application_table.arn
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "policy_attach" {
  role       = aws_iam_role.application_role.name
  policy_arn = aws_iam_policy.readonly_policy.arn
}
```

---

# Terraform Outputs

## outputs.tf

```hcl
output "kke_dynamodb_table" {
  value = aws_dynamodb_table.application_table.name
}

output "kke_iam_role_name" {
  value = aws_iam_role.application_role.name
}

output "kke_iam_policy_name" {
  value = aws_iam_policy.readonly_policy.name
}
```

---

# Deployment Steps

## Verify Working Directory

### Command

```bash
pwd
```

### Output

```text
/home/devops/terraform
```

### Explanation

Verifies that Terraform files are being created in the correct working directory.

---

# Create Variables File

### Command

```bash
vi variables.tf
```

### Explanation

Defines reusable variables.

---

# Create Variable Values

### Command

```bash
vi terraform.tfvars
```

### Example Content

```hcl
KKE_TABLE_NAME  = "application-data-table"
KKE_ROLE_NAME   = "application-read-role"
KKE_POLICY_NAME = "dynamodb-readonly-policy"
```

---

# Create Main Configuration

### Command

```bash
vi main.tf
```

### Explanation

Defines DynamoDB table, IAM role, IAM policy and policy attachment.

---

# Create Outputs File

### Command

```bash
vi outputs.tf
```

### Explanation

Displays important resource names after deployment.

---

# Initialize Terraform

### Command

```bash
terraform init
```

### Output

```text
Terraform has been successfully initialized!
```

### Explanation

Downloads required provider plugins.

---

# Validate Configuration

### Command

```bash
terraform validate
```

### Output

```text
Success! The configuration is valid.
```

### Explanation

Validates Terraform syntax and configuration.

---

# Generate Execution Plan

### Command

```bash
terraform plan
```

### Output

```text
Plan: 4 to add, 0 to change, 0 to destroy.
```

### Resources Planned

```text
aws_dynamodb_table.application_table
aws_iam_role.application_role
aws_iam_policy.readonly_policy
aws_iam_role_policy_attachment.policy_attach
```

---

# Deploy Infrastructure

### Command

```bash
terraform apply -auto-approve
```

### Output

```text
Apply complete!

Resources: 4 added, 0 changed, 0 destroyed.
```

---

# Terraform Outputs

### Command

```bash
terraform output
```

### Output

```text
kke_dynamodb_table = "application-data-table"

kke_iam_policy_name = "dynamodb-readonly-policy"

kke_iam_role_name = "application-read-role"
```

---

# Verification Steps

---

## Verify Terraform State

### Command

```bash
terraform state list
```

### Output

```text
aws_dynamodb_table.application_table
aws_iam_policy.readonly_policy
aws_iam_role.application_role
aws_iam_role_policy_attachment.policy_attach
```

---

## Verify Policy Configuration

### Command

```bash
terraform state show aws_iam_policy.readonly_policy
```

### Output

```json
{
  "Action": [
    "dynamodb:GetItem",
    "dynamodb:Scan",
    "dynamodb:Query"
  ],

  "Effect": "Allow",

  "Resource": "arn:aws:dynamodb:region:account-id:table/application-data-table"
}
```

---

# Permission Analysis

## Allowed Actions

```text
dynamodb:GetItem
dynamodb:Scan
dynamodb:Query
```

---

## Blocked Actions

```text
dynamodb:PutItem
dynamodb:UpdateItem
dynamodb:DeleteItem
dynamodb:BatchWriteItem
dynamodb:CreateTable
dynamodb:DeleteTable
```

---

# Security Design

## Why Restrict Access to a Specific Table?

Bad Practice:

```json
{
  "Resource": "*"
}
```

Allows access to all DynamoDB tables.

---

Good Practice:

```json
{
  "Resource": "Specific DynamoDB Table ARN"
}
```

Provides least-privilege access.

---

# IAM Access Flow

```text
AWS Service
     │
     ▼
IAM Role
     │
     ▼
IAM Policy
     │
     ▼
GetItem
Scan
Query
     │
     ▼
DynamoDB Table
```

---

# DynamoDB Read Operations

## GetItem

Reads one item using primary key.

---

## Scan

Reads all table data.

---

## Query

Reads filtered data using partition key.

---

# Scenario Based Interview Questions & Answers

# L1 – Junior Engineer

## Q1. What is DynamoDB?

### Answer

DynamoDB is a fully managed NoSQL database service provided by AWS.

---

## Q2. What is an IAM Role?

### Answer

An IAM Role is a temporary identity that can be assumed by AWS services or users.

---

## Q3. What is an IAM Policy?

### Answer

An IAM Policy is a JSON document that defines permissions.

---

## Q4. What does GetItem do?

### Answer

Retrieves a single item from a DynamoDB table.

---

# L2 – Mid-Level Engineer

## Q1. Difference Between IAM User and IAM Role?

### IAM User

```text
Permanent Identity
Long-Term Credentials
```

### IAM Role

```text
Temporary Identity
Assumed Dynamically
```

---

## Q2. Why Restrict Policy to Table ARN?

### Answer

To ensure users can access only required resources.

---

## Q3. Why Use Read-Only Access?

### Answer

To prevent accidental modification or deletion of data.

---

# L3 – Senior Engineer

## Q1. How would you secure DynamoDB access?

### Answer

Implement:

```text
IAM Roles
Least Privilege
Encryption
CloudTrail
VPC Endpoints
```

---

## Q2. How would you audit access?

### Answer

Use:

```text
CloudTrail
IAM Access Analyzer
AWS Config
Security Hub
```

---

## Q3. What risks exist with Resource "*"?

### Answer

```text
Unauthorized Access
Data Leakage
Privilege Escalation
Compliance Violations
```

---

# L4 – Architect Level

## Q1. Design Enterprise Data Access Architecture

### Answer

```text
Application
     │
     ▼
IAM Role
     │
     ▼
Fine-Grained IAM Policies
     │
     ▼
DynamoDB
```

Benefits:

* Least Privilege
* Scalability
* Security
* Auditability

---

## Q2. How would you secure multi-account environments?

### Answer

Use:

```text
AWS Organizations
IAM Identity Center
Cross-Account Roles
Service Control Policies
```

---

# AWS Scenario Based Questions

## Scenario 1

Developer cannot read data from DynamoDB.

### Troubleshooting

Check:

```text
IAM Role
Policy Attachment
Table ARN
Trust Relationship
```

---

## Scenario 2

Developer can read all tables unexpectedly.

### Root Cause

```json
{
  "Resource": "*"
}
```

Used in IAM Policy.

---

## Scenario 3

Application receives AccessDenied errors.

### Troubleshooting

Verify:

```text
IAM Policy
Assume Role Permissions
CloudTrail Logs
```

---

# Real AWS Production Incident RCA

## Incident

Application gained access to sensitive customer data.

---

## Impact

```text
Data Exposure
Compliance Violation
Security Incident
```

---

## Root Cause

IAM Policy used:

```json
{
  "Action": "dynamodb:*",
  "Resource": "*"
}
```

---

## Resolution

Implemented:

```text
Read-Only Policies
Resource-Level Restrictions
IAM Reviews
```

---

## Preventive Measures

```text
Least Privilege
Terraform Code Reviews
Access Analyzer
Security Audits
```

---

# Full DevOps Mock Interview

## Q1. What is Terraform State?

### Answer

Terraform State tracks deployed infrastructure resources.

---

## Q2. Why is Terraform State Important?

### Answer

Terraform uses state to determine:

```text
Current Infrastructure
Desired Infrastructure
Required Changes
```

---

## Q3. What Happens During terraform plan?

### Answer

Terraform compares state with configuration and generates an execution plan.

---

## Q4. What Happens During terraform apply?

### Answer

Terraform provisions or updates infrastructure resources.

---

## Q5. Why Use Infrastructure as Code?

### Answer

Benefits:

```text
Automation
Consistency
Repeatability
Version Control
Compliance
```

---

# Key Learnings

1. DynamoDB provides scalable NoSQL storage.
2. IAM Roles provide secure temporary credentials.
3. IAM Policies enforce fine-grained permissions.
4. Resource-level restrictions improve security.
5. Least Privilege reduces attack surface.
6. Terraform automates infrastructure provisioning.
7. Verification is critical after deployment.

---

# Final Result

```text
Task Status              : SUCCESS

Resources Created:

1. DynamoDB Table
   Name : application-data-table

2. IAM Role
   Name : application-read-role

3. IAM Policy
   Name : dynamodb-readonly-policy

4. Policy Attachment
   Status : Attached Successfully

Allowed Actions:
  GetItem
  Scan
  Query

Restricted Resource:
  Specific DynamoDB Table ARN

Terraform Init         : Successful
Terraform Validate     : Successful
Terraform Plan         : Successful
Terraform Apply        : Successful

Verification           : Successful

Challenge Status       : PASSED
```

