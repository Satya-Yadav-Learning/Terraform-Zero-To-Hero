# TaskAndResolution.md

# Terraform Task: Create IAM User Using Terraform

---

# Task Objective

As part of cloud identity and access management automation, an IAM user needed to be provisioned using Terraform.

The objective was to create an IAM user through Infrastructure as Code (IaC) while ensuring the deployment remained repeatable, auditable, and version-controlled.

---

# Requirements

* Create an IAM User
* User Name: `clouduser_mariyam`
* Use Terraform only
* Create resources in `us-east-1`
* Store configuration in `main.tf`
* Use existing provider configuration

---

# Solution Architecture

```text
Terraform
    │
    ▼
AWS IAM
    │
    ▼
IAM User
    │
    ▼
clouduser_mariyam
```

---

# Terraform Configuration (main.tf)

```hcl
resource "aws_iam_user" "clouduser_mariyam" {
  name = "clouduser_mariyam"
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

Verifies that Terraform files are created in the correct working directory.

---

## Verify Existing Files

### Command

```bash
ll
```

### Output

```text
total 20

README.MD
provider.tf
```

### Explanation

Confirms provider configuration already exists.

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

Confirms:

* AWS Provider configured
* Region configured as us-east-1
* Terraform can communicate with AWS services

---

# Create Terraform Configuration

## Command

```bash
cat > main.tf <<'EOF'
resource "aws_iam_user" "clouduser_mariyam" {
  name = "clouduser_mariyam"
}
EOF
```

### Explanation

Creates the Terraform configuration file responsible for provisioning the IAM user.

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

Downloads provider plugins and prepares Terraform for execution.

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

  # aws_iam_user.clouduser_mariyam will be created

Plan: 1 to add, 0 to change, 0 to destroy.
```

### Explanation

Shows the infrastructure changes Terraform intends to make.

---

# Terraform Deployment

## Command

```bash
terraform apply -auto-approve
```

### Output

```text
aws_iam_user.clouduser_mariyam: Creating...

aws_iam_user.clouduser_mariyam: Creation complete after 0s

Apply complete!
Resources: 1 added, 0 changed, 0 destroyed.
```

### Explanation

Creates the IAM user without interactive approval.

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
aws_iam_user.clouduser_mariyam
```

### Explanation

Confirms Terraform is managing the IAM user.

---

## Verify Resource Details

### Command

```bash
terraform state show aws_iam_user.clouduser_mariyam
```

### Output

```text
arn       = "arn:aws:iam::000000000000:user/clouduser_mariyam"

id        = "clouduser_mariyam"

name      = "clouduser_mariyam"

path      = "/"

unique_id = "dhxl69vfth24u7bmvpx2"
```

### Explanation

Confirms:

* IAM user exists
* User name is correct
* ARN assigned
* Terraform resource successfully created

---

# Resource Breakdown

## IAM User

```hcl
resource "aws_iam_user" "clouduser_mariyam" {
  name = "clouduser_mariyam"
}
```

### Purpose

Creates an IAM identity that can later be used for:

* Console access
* Programmatic access
* Role assumption
* AWS service integrations

---

# Terraform Workflow

```text
main.tf
   │
terraform init
   │
terraform validate
   │
terraform plan
   │
terraform apply
   │
AWS IAM User Created
```

---

# Security Considerations

### Best Practices

* Use IAM Groups instead of direct permissions
* Follow Least Privilege Principle
* Enable MFA
* Rotate Access Keys
* Prefer IAM Roles when possible

---

# Scenario Based Interview Questions & Answers

# L1 – Junior Engineer

## Q1. What is IAM?

### Answer

IAM (Identity and Access Management) is an AWS service used to manage users, groups, roles, and permissions.

---

## Q2. What is an IAM User?

### Answer

An IAM User is a permanent identity within an AWS account that can be assigned permissions.

---

## Q3. Why use Terraform for IAM?

### Answer

Terraform automates IAM resource creation and ensures consistency across environments.

---

## Q4. What is Infrastructure as Code?

### Answer

Infrastructure managed through code instead of manual console operations.

---

# L2 – Mid-Level Engineer

## Q1. Difference between IAM User and IAM Role?

### Answer

### IAM User

```text
Permanent Identity
```

Examples:

```text
Developers
Administrators
Automation Accounts
```

### IAM Role

```text
Temporary Identity
```

Examples:

```text
EC2 Instances
Lambda Functions
Cross Account Access
```

---

## Q2. Why are IAM Roles preferred?

### Answer

Roles eliminate long-term credentials and improve security.

---

## Q3. What is Least Privilege?

### Answer

Grant only the permissions necessary to perform required tasks.

---

# L3 – Senior Engineer

## Q1. How would you manage thousands of IAM users?

### Answer

* IAM Groups
* Terraform Modules
* AWS SSO
* Identity Federation
* Automated User Lifecycle Management

---

## Q2. How would you audit IAM users?

### Answer

Using:

```text
AWS Config
CloudTrail
IAM Access Analyzer
Security Hub
```

---

## Q3. How would you secure IAM access?

### Answer

* MFA Enforcement
* Password Policies
* Role-Based Access
* SCP Enforcement
* Credential Rotation

---

# L4 – Architect Level

## Q1. Design Enterprise IAM Architecture

### Answer

```text
Corporate Identity Provider
           │
           ▼
AWS IAM Identity Center
           │
           ▼
Permission Sets
           │
           ▼
AWS Accounts
```

Benefits:

* Centralized Authentication
* SSO
* Reduced IAM User Sprawl

---

## Q2. Why avoid large numbers of IAM users?

### Answer

Because:

* Difficult to manage
* Credential rotation overhead
* Increased security risk

Modern enterprises prefer:

```text
Federation
IAM Identity Center
Temporary Credentials
```

---

# AWS Scenario Based Questions

## Scenario 1

Developer cannot access AWS resources.

### Investigation

```bash
aws iam get-user
aws iam list-attached-user-policies
```

Check:

* User Exists
* Policies Attached
* Group Membership
* SCP Restrictions

---

## Scenario 2

User receives AccessDenied.

### Troubleshooting

Check:

```text
IAM Policy
Permission Boundary
Resource Policy
SCP
Session Policy
```

---

## Scenario 3

Access Keys Compromised.

### Resolution

1. Disable keys immediately
2. Create new credentials
3. Audit CloudTrail logs
4. Rotate secrets
5. Review permissions

---

# Real AWS Incident RCA

## Incident

Developer account compromised due to leaked access key.

### Impact

Unauthorized API calls executed.

### Root Cause

Access key stored in source code repository.

### Resolution

* Disabled access keys
* Rotated credentials
* Audited CloudTrail logs
* Enforced secret scanning

### Prevention

```text
IAM Roles
AWS Secrets Manager
MFA
Least Privilege
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
What Exists
What Changed
What Needs Creation
What Needs Deletion
```

---

## Q3. What Happens During terraform plan?

### Answer

Terraform compares:

```text
Current State
vs
Desired State
```

and generates an execution plan.

---

## Q4. What Happens During terraform apply?

### Answer

Terraform executes the changes generated by the plan.

---

## Q5. Why Use Version Control for Terraform?

### Answer

Benefits:

```text
Auditability
Rollback
Collaboration
Change Tracking
Compliance
```

---

# Key Learnings

1. IAM users can be provisioned using Terraform.
2. Infrastructure as Code improves consistency.
3. Terraform state tracks deployed resources.
4. IAM should follow least privilege principles.
5. IAM Roles are preferred over long-term credentials.
6. Verification should always be performed after deployment.

---

# Final Result

```text
Task Status        : SUCCESS

Resource Type      : IAM User
User Name          : clouduser_mariyam
Terraform Init     : Successful
Terraform Validate : Successful
Terraform Plan     : Successful
Terraform Apply    : Successful
Verification       : Successful

Challenge Status   : PASSED
```

