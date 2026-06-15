# TaskAndResolution.md

# Terraform Task: Create IAM Policy for Read-Only EC2 Console Access

---

# Task Objective

As part of cloud identity and access management automation, an IAM Policy needed to be provisioned using Terraform.

The objective was to create a reusable read-only IAM policy that allows users to view Amazon EC2 resources such as instances, AMIs, snapshots, volumes, security groups, and VPC information without granting modification permissions.

---

# Requirements

* Create an IAM Policy
* Policy Name: `readonly_ec2_policy`
* Region: `us-east-1`
* Use Terraform only
* Policy must provide read-only access to Amazon EC2 resources
* Store Terraform configuration in `main.tf`

---

# Solution Architecture

```text
Terraform
    │
    ▼
AWS IAM
    │
    ▼
IAM Policy
    │
    ▼
Read Only EC2 Access
    │
    ├── View Instances
    ├── View AMIs
    ├── View Snapshots
    ├── View Volumes
    ├── View Security Groups
    └── View VPC Resources
```

---

# Terraform Configuration (main.tf)

```hcl
resource "aws_iam_policy" "readonly_ec2_policy" {
  name        = "readonly_ec2_policy"
  description = "Read-only access to EC2 resources"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "ec2:Describe*"
        ]
        Resource = "*"
      }
    ]
  })
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
/home/devops/terraform
```

### Explanation

Confirms Terraform files are being created in the correct working directory.

---

## Verify Existing Files

### Command

```bash
ll
```

### Output

```text
README.MD
provider.tf
```

### Explanation

Confirms AWS provider configuration already exists.

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
}
```

### Explanation

Verifies:

* AWS Provider configured
* Region configured
* Terraform can communicate with AWS services

---

# Create Terraform Configuration

## Command

```bash
cat > main.tf <<'EOF'
resource "aws_iam_policy" "readonly_ec2_policy" {
  name        = "readonly_ec2_policy"
  description = "Read-only access to EC2 resources"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "ec2:Describe*"
        ]
        Resource = "*"
      }
    ]
  })
}
EOF
```

### Explanation

Creates Terraform configuration for provisioning the IAM policy.

---

# Terraform Initialization

## Command

```bash
terraform init
```

### Output

```text
Initializing the backend...

Initializing provider plugins...

Terraform has been successfully initialized!
```

### Explanation

Downloads provider plugins and prepares Terraform execution environment.

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
Terraform will perform the following actions:

# aws_iam_policy.readonly_ec2_policy will be created

Plan: 1 to add, 0 to change, 0 to destroy.
```

### Explanation

Shows infrastructure changes before deployment.

---

# Terraform Deployment

## Command

```bash
terraform apply -auto-approve
```

### Output

```text
aws_iam_policy.readonly_ec2_policy: Creating...

aws_iam_policy.readonly_ec2_policy: Creation complete after 0s

Apply complete!
Resources: 1 added, 0 changed, 0 destroyed.
```

### Explanation

Creates IAM policy automatically without requiring manual approval.

---

# Verification Steps

## Verify Terraform State

### Command

```bash
terraform state list
```

### Output

```text
aws_iam_policy.readonly_ec2_policy
```

### Explanation

Confirms Terraform is managing the IAM policy.

---

## Verify IAM Policy Details

### Command

```bash
terraform state show aws_iam_policy.readonly_ec2_policy
```

### Output

```text
arn              = "arn:aws:iam::000000000000:policy/readonly_ec2_policy"

description      = "Read-only access to EC2 resources"

name             = "readonly_ec2_policy"

path             = "/"
```

### Explanation

Confirms:

* IAM Policy exists
* Policy name is correct
* Description is correct
* Resource is managed by Terraform

---

## Verify Policy Document

### Command

```bash
terraform state show aws_iam_policy.readonly_ec2_policy
```

### Output

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*"
      ],
      "Resource": "*"
    }
  ]
}
```

### Explanation

The policy grants read-only access to EC2 resources.

---

# Permission Analysis

## Allowed Operations

```text
ec2:DescribeInstances
ec2:DescribeImages
ec2:DescribeSnapshots
ec2:DescribeVolumes
ec2:DescribeSecurityGroups
ec2:DescribeSubnets
ec2:DescribeVpcs
ec2:DescribeRouteTables
ec2:DescribeAddresses
ec2:DescribeNetworkInterfaces
```

Users can view resources.

---

## Blocked Operations

```text
ec2:RunInstances
ec2:TerminateInstances
ec2:StartInstances
ec2:StopInstances
ec2:RebootInstances
ec2:CreateSnapshot
ec2:DeleteSnapshot
ec2:CreateVolume
ec2:DeleteVolume
```

Users cannot modify resources.

---

# Why Use ec2:Describe*

The EC2 Console requires multiple API calls to display information.

Instead of granting individual permissions:

```text
ec2:DescribeInstances
ec2:DescribeImages
ec2:DescribeSnapshots
```

Terraform uses:

```text
ec2:Describe*
```

which covers all read-only EC2 console operations.

---

# Security Benefits

```text
Read Access Only
No Resource Creation
No Resource Modification
No Resource Deletion
Least Privilege Principle
```

---

# IAM Policy Evaluation Flow

```text
User
  │
  ▼
IAM Policy
  │
  ▼
ec2:Describe*
  │
  ▼
Read Access Granted
```

---

# Scenario Based Interview Questions & Answers

# L1 – Junior Engineer

## Q1. What is an IAM Policy?

### Answer

An IAM Policy is a JSON document that defines permissions for AWS resources.

---

## Q2. What does Allow mean?

### Answer

Allow grants permission to perform an action on AWS resources.

---

## Q3. What is Resource "*"?

### Answer

The permission applies to all resources.

---

## Q4. What is ec2:Describe*?

### Answer

A wildcard permission that allows viewing EC2 resources.

---

# L2 – Mid-Level Engineer

## Q1. Difference Between IAM User, Group and Policy?

### Answer

### IAM User

```text
Identity
```

### IAM Group

```text
Collection of Users
```

### IAM Policy

```text
Permission Document
```

---

## Q2. Why use Read-Only Policies?

### Answer

Provides visibility without risk of modification.

---

## Q3. What is Least Privilege?

### Answer

Grant only the permissions required to perform a task.

---

# L3 – Senior Engineer

## Q1. How would you audit IAM policies?

### Answer

Use:

```text
IAM Access Analyzer
AWS Config
CloudTrail
Security Hub
```

---

## Q2. What risks exist with overly permissive policies?

### Answer

```text
Privilege Escalation
Data Exposure
Unauthorized Changes
Security Incidents
```

---

## Q3. How would you manage IAM policies at scale?

### Answer

Using:

```text
Terraform Modules
Version Control
Policy Libraries
CI/CD Validation
```

---

# L4 – Architect Level

## Q1. Design Enterprise IAM Permission Architecture

### Answer

```text
Identity Provider
        │
        ▼
AWS IAM Identity Center
        │
        ▼
Permission Sets
        │
        ▼
IAM Roles
        │
        ▼
AWS Resources
```

Benefits:

* Centralized Access
* SSO
* Reduced Credential Management

---

## Q2. Why prefer Roles over Users?

### Answer

Roles provide temporary credentials and improve security.

---

# AWS Scenario Based Questions

## Scenario 1

A developer can view instances but cannot start them.

### Root Cause

Policy contains:

```text
ec2:Describe*
```

but not:

```text
ec2:StartInstances
```

---

## Scenario 2

A support engineer needs visibility but no modification rights.

### Solution

Attach read-only policy.

---

## Scenario 3

An auditor requires access to EC2 inventory.

### Solution

Grant:

```text
ec2:Describe*
```

without write permissions.

---

# Real AWS Incident RCA

## Incident

An engineer accidentally terminated production instances.

### Impact

Application outage.

### Root Cause

Excessive IAM permissions granted.

```text
ec2:*
```

instead of:

```text
ec2:Describe*
```

### Resolution

Implemented read-only roles for support teams.

### Prevention

```text
Least Privilege
IAM Reviews
Access Analyzer
Terraform Policy Reviews
```

---

# Full DevOps Mock Interview

## Q1. Explain Terraform State

### Answer

Terraform state tracks infrastructure resources managed by Terraform.

File:

```text
terraform.tfstate
```

---

## Q2. Why is Terraform State Important?

### Answer

Terraform uses state to determine:

```text
Current State
Desired State
Infrastructure Changes
```

---

## Q3. What Happens During terraform plan?

### Answer

Terraform compares current infrastructure with desired infrastructure and generates an execution plan.

---

## Q4. What Happens During terraform apply?

### Answer

Terraform executes changes generated by the plan.

---

## Q5. Why Use Infrastructure as Code?

### Answer

Benefits:

```text
Automation
Consistency
Version Control
Auditability
Repeatability
```

---

# Key Learnings

1. IAM Policies define AWS permissions.
2. ec2:Describe* provides EC2 read-only access.
3. Least Privilege reduces security risks.
4. Terraform automates IAM policy management.
5. IAM Policies should be version controlled.
6. Verification is essential after deployment.

---

# Final Result

```text
Task Status        : SUCCESS

Resource Type      : IAM Policy

Policy Name        : readonly_ec2_policy

Permissions:
  ec2:Describe*

Access Level:
  Read Only

Terraform Init     : Successful
Terraform Validate : Successful
Terraform Plan     : Successful
Terraform Apply    : Successful

Verification       : Successful

Challenge Status   : PASSED
```

