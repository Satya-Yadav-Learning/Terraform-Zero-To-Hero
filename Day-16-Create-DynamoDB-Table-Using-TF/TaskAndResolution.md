# TaskAndResolution.md

# Terraform Task: Create DynamoDB Table Using Terraform

---

# Project Overview

As part of a cloud-native application modernization initiative, the engineering team required a scalable NoSQL database for storing application user information.

The solution involved provisioning an Amazon DynamoDB table using Terraform with:

* On-Demand Billing (PAY_PER_REQUEST)
* String-based Partition Key
* Fully managed infrastructure deployment using Infrastructure as Code (IaC)

The objective was to create a highly scalable, serverless database solution without managing capacity manually.

---

# Business Requirements

| Requirement        | Value             |
| ------------------ | ----------------- |
| Service            | Amazon DynamoDB   |
| Table Name         | application-users |
| Primary Key        | user_identifier   |
| Key Type           | String            |
| Billing Mode       | PAY_PER_REQUEST   |
| Deployment Method  | Terraform         |
| Configuration File | main.tf           |

---

# Solution Architecture

```text
                Application
                      │
                      ▼
               DynamoDB Table
                      │
                      ▼
              application-users
                      │
                      ▼
            Partition Key (PK)
               user_identifier
                  Type:String
```

---

# Why DynamoDB?

Amazon DynamoDB is a fully managed NoSQL database service that provides:

```text
High Availability
Automatic Scaling
Low Latency
Serverless Operations
Multi-AZ Replication
```

Benefits:

* No server management
* No capacity planning
* Pay only for actual usage
* Millisecond response times

---

# Terraform Configuration

## main.tf

```hcl
resource "aws_dynamodb_table" "application_users" {

  name         = "application-users"

  billing_mode = "PAY_PER_REQUEST"

  hash_key     = "user_identifier"

  attribute {
    name = "user_identifier"
    type = "S"
  }
}
```

---

# Configuration Breakdown

## Table Name

```hcl
name = "application-users"
```

Creates DynamoDB table.

---

## Billing Mode

```hcl
billing_mode = "PAY_PER_REQUEST"
```

AWS automatically scales capacity.

No need to configure:

```text
Read Capacity Units (RCU)
Write Capacity Units (WCU)
```

---

## Primary Key

```hcl
hash_key = "user_identifier"
```

Acts as the Partition Key.

Every item must contain this attribute.

---

## Attribute Definition

```hcl
attribute {
  name = "user_identifier"
  type = "S"
}
```

Type:

```text
S = String
```

Other DynamoDB types:

```text
S = String
N = Number
B = Binary
```

---

# Execution Steps

---

# Verify Working Directory

## Command

```bash
pwd
```

## Output

```text
/home/devops/terraform
```

### Explanation

Confirms Terraform files are being created in the correct workspace.

---

# Create Terraform Configuration

## Command

```bash
cat > main.tf <<'EOF'
resource "aws_dynamodb_table" "application_users" {

  name         = "application-users"

  billing_mode = "PAY_PER_REQUEST"

  hash_key     = "user_identifier"

  attribute {
    name = "user_identifier"
    type = "S"
  }
}
EOF
```

### Explanation

Creates the Terraform configuration file.

---

# Initialize Terraform

## Command

```bash
terraform init
```

## Output

```text
Terraform has been successfully initialized!
```

### Explanation

Downloads providers and initializes the Terraform working directory.

---

# Validate Terraform Configuration

## Command

```bash
terraform validate
```

## Output

```text
Success! The configuration is valid.
```

### Explanation

Verifies Terraform syntax and resource configuration.

---

# Generate Execution Plan

## Command

```bash
terraform plan
```

## Output

```text
Terraform will perform the following actions:

# aws_dynamodb_table.application_users will be created

Plan: 1 to add, 0 to change, 0 to destroy.
```

### Explanation

Shows planned infrastructure changes before deployment.

---

# Deploy Infrastructure

## Command

```bash
terraform apply -auto-approve
```

## Output

```text
aws_dynamodb_table.application_users: Creating...

aws_dynamodb_table.application_users: Creation complete after 3s

Apply complete!

Resources: 1 added, 0 changed, 0 destroyed.
```

### Explanation

Creates the DynamoDB table automatically.

---

# Verification Steps

---

# Verify Terraform State

## Command

```bash
terraform state list
```

## Output

```text
aws_dynamodb_table.application_users
```

### Explanation

Confirms Terraform manages the DynamoDB table.

---

# Verify Table Configuration

## Command

```bash
terraform state show aws_dynamodb_table.application_users
```

## Output

```text
name         = "application-users"

billing_mode = "PAY_PER_REQUEST"

hash_key     = "user_identifier"
```

---

# Verify Attribute Definition

## Output

```text
attribute {
    name = "user_identifier"
    type = "S"
}
```

### Explanation

Confirms:

* Primary Key Exists
* Data Type = String

---

# Verify Billing Mode

## Output

```text
billing_mode = "PAY_PER_REQUEST"
```

### Explanation

AWS automatically manages:

```text
Read Capacity
Write Capacity
Scaling
```

No manual capacity configuration required.

---

# Understanding PAY_PER_REQUEST

Traditional DynamoDB:

```text
Provisioned Mode
     │
     ▼
Specify RCU
Specify WCU
Monitor Capacity
Scale Capacity
```

PAY_PER_REQUEST:

```text
Application Traffic
          │
          ▼
AWS Automatically Scales
          │
          ▼
Pay Only For Usage
```

---

# Resource Architecture

```text
Terraform
    │
    ▼
DynamoDB Table
    │
    ▼
application-users
    │
    ▼
user_identifier (PK)
```

---

# DynamoDB Data Example

Example Item:

```json
{
  "user_identifier": "1001",
  "username": "john",
  "email": "john@example.com"
}
```

Partition Key:

```text
1001
```

Used to uniquely identify the record.

---

# Scenario Based Interview Questions & Answers

# L1 – Junior Engineer

## Q1. What is DynamoDB?

### Answer

DynamoDB is AWS's fully managed NoSQL database service.

---

## Q2. What is a Primary Key?

### Answer

A Primary Key uniquely identifies each item in a DynamoDB table.

---

## Q3. What is a Partition Key?

### Answer

A Partition Key determines where data is stored internally.

---

## Q4. What does type "S" mean?

### Answer

```text
S = String
```

---

# L2 – Mid-Level Engineer

## Q1. Difference Between SQL and DynamoDB?

### SQL

```text
Structured Data
Joins
Fixed Schema
```

### DynamoDB

```text
NoSQL
Flexible Schema
Serverless
```

---

## Q2. What is PAY_PER_REQUEST?

### Answer

On-demand billing mode where AWS automatically scales capacity.

---

## Q3. When should PAY_PER_REQUEST be used?

### Answer

Ideal for:

```text
Unpredictable Workloads
Variable Traffic
Development Environments
```

---

# L3 – Senior Engineer

## Q1. How does DynamoDB scale?

### Answer

AWS automatically partitions data across storage nodes.

---

## Q2. What causes hot partitions?

### Answer

Poor partition key selection.

Example:

```text
Same Partition Key
High Request Volume
```

---

## Q3. How would you secure DynamoDB?

### Answer

Use:

```text
IAM Policies
Encryption
VPC Endpoints
CloudTrail
```

---

# L4 – Architect Level

## Q1. Design Large Scale User Database

### Answer

```text
Application Layer
        │
        ▼
DynamoDB
        │
        ▼
Global Tables
        │
        ▼
Multi Region Replication
```

Benefits:

```text
High Availability
Low Latency
Disaster Recovery
```

---

## Q2. How would you optimize DynamoDB costs?

### Answer

Use:

```text
PAY_PER_REQUEST
TTL
Sparse Indexes
Efficient Key Design
```

---

# AWS Scenario Based Questions

## Scenario 1

Application traffic suddenly increases 10x.

### Solution

With:

```text
PAY_PER_REQUEST
```

AWS automatically scales.

No manual intervention required.

---

## Scenario 2

Application performance degrades.

### Investigation

Check:

```text
Partition Key Design
Hot Partitions
CloudWatch Metrics
```

---

## Scenario 3

Users report missing records.

### Investigation

Check:

```text
Application Logic
Partition Keys
IAM Permissions
CloudTrail Logs
```

---

# Real AWS Production Incident RCA

# Incident

E-commerce application experienced DynamoDB throttling.

---

## Impact

```text
Slow Checkout
API Timeouts
Customer Complaints
```

---

## Root Cause

Table used:

```text
Provisioned Capacity
```

Traffic increased unexpectedly.

---

## Resolution

Migrated to:

```text
PAY_PER_REQUEST
```

---

## Preventive Measures

```text
CloudWatch Monitoring
Auto Scaling
Capacity Planning
Terraform Governance
```

---

# Full DevOps Mock Interview

## Q1. What is Infrastructure as Code?

### Answer

Infrastructure provisioning through code rather than manual actions.

---

## Q2. Why Terraform?

### Answer

Benefits:

```text
Automation
Repeatability
Version Control
Consistency
```

---

## Q3. What is Terraform State?

### Answer

Terraform State tracks infrastructure resources.

File:

```text
terraform.tfstate
```

---

## Q4. What Happens During terraform plan?

### Answer

Terraform compares:

```text
Current State
Desired State
```

and generates an execution plan.

---

## Q5. What Happens During terraform apply?

### Answer

Terraform executes planned changes.

---

# Key Learnings

1. DynamoDB is a fully managed NoSQL database.
2. Partition Keys uniquely identify records.
3. PAY_PER_REQUEST eliminates capacity planning.
4. Terraform automates infrastructure deployment.
5. Verification is critical after deployment.
6. Proper key design prevents hot partitions.
7. Infrastructure as Code improves consistency and governance.

---

# Final Result

```text
Task Status          : SUCCESS

Resource Type        : DynamoDB Table

Table Name           : application-users

Primary Key          : user_identifier

Key Type             : String (S)

Billing Mode         : PAY_PER_REQUEST

Terraform Init       : Successful
Terraform Validate   : Successful
Terraform Plan       : Successful
Terraform Apply      : Successful

Verification         : Successful

Challenge Status     : PASSED
```

