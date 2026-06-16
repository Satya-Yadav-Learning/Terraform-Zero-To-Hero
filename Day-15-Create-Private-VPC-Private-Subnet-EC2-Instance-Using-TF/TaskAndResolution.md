# TaskAndResolution.md

# Terraform Task: Create Private VPC, Private Subnet and EC2 Instance Using Terraform

---

# Project Overview

As part of a cloud infrastructure modernization initiative, the platform engineering team needed to provision a completely isolated network environment within AWS.

The requirement was to create:

* A private VPC
* A private subnet
* A security group allowing traffic only from within the VPC
* A private EC2 instance deployed inside the subnet

The objective was to ensure that workloads remain inaccessible from the public internet while allowing secure communication within the VPC.

All infrastructure was provisioned using Terraform following Infrastructure as Code (IaC) principles.

---

# Business Requirements

| Component             | Requirement                       |
| --------------------- | --------------------------------- |
| VPC Name              | private-app-vpc                   |
| VPC CIDR              | 10.0.0.0/16                       |
| Subnet Name           | private-app-subnet                |
| Subnet CIDR           | 10.0.1.0/24                       |
| Auto Assign Public IP | Disabled                          |
| EC2 Name              | private-app-server                |
| Instance Type         | t2.micro                          |
| Security Group        | Allow traffic only from VPC CIDR  |
| Terraform Files       | main.tf, variables.tf, outputs.tf |

---

# Solution Architecture

```text
                    AWS Cloud
                         │
                         ▼
                 private-app-vpc
                   10.0.0.0/16
                         │
                         ▼
              private-app-subnet
                  10.0.1.0/24
                         │
                         ▼
                Security Group
          Allow: 10.0.0.0/16 Only
                         │
                         ▼
                private-app-server
                     t2.micro
```

---

# Terraform Variables File

## variables.tf

```hcl
variable "KKE_VPC_CIDR" {
  default = "10.0.0.0/16"
}

variable "KKE_SUBNET_CIDR" {
  default = "10.0.1.0/24"
}
```

---

# Terraform Main Configuration

## main.tf

```hcl
resource "aws_vpc" "private_app_vpc" {
  cidr_block = var.KKE_VPC_CIDR

  tags = {
    Name = "private-app-vpc"
  }
}

resource "aws_subnet" "private_app_subnet" {
  vpc_id                  = aws_vpc.private_app_vpc.id
  cidr_block              = var.KKE_SUBNET_CIDR
  map_public_ip_on_launch = false

  tags = {
    Name = "private-app-subnet"
  }
}

resource "aws_security_group" "private_app_sg" {
  name        = "private-app-sg"
  description = "Allow traffic only from VPC"
  vpc_id      = aws_vpc.private_app_vpc.id

  ingress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = [var.KKE_VPC_CIDR]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "private_app_server" {
  ami                    = "ami-0c101f26f147fa7fd"
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.private_app_subnet.id
  vpc_security_group_ids = [aws_security_group.private_app_sg.id]

  tags = {
    Name = "private-app-server"
  }
}
```

---

# Terraform Outputs

## outputs.tf

```hcl
output "KKE_vpc_name" {
  value = aws_vpc.private_app_vpc.tags["Name"]
}

output "KKE_subnet_name" {
  value = aws_subnet.private_app_subnet.tags["Name"]
}

output "KKE_ec2_private" {
  value = aws_instance.private_app_server.tags["Name"]
}
```

---

# Execution Steps

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

Ensures Terraform files are being created in the correct workspace.

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

Confirms AWS provider configuration exists.

---

## Create Variables File

### Command

```bash
vi variables.tf
```

### Explanation

Stores reusable CIDR variables.

---

## Create Main Configuration

### Command

```bash
vi main.tf
```

### Explanation

Defines VPC, Subnet, Security Group and EC2 resources.

---

## Create Outputs File

### Command

```bash
vi outputs.tf
```

### Explanation

Returns resource names after deployment.

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

Downloads providers and initializes Terraform.

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

Checks Terraform syntax.

---

# Terraform Plan

## Command

```bash
terraform plan
```

### Output

```text
Plan: 4 to add, 0 to change, 0 to destroy.
```

### Resources Planned

```text
aws_vpc.private_app_vpc
aws_subnet.private_app_subnet
aws_security_group.private_app_sg
aws_instance.private_app_server
```

---

# Terraform Apply

## Command

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

### Output

```text
KKE_ec2_private = "private-app-server"

KKE_subnet_name = "private-app-subnet"

KKE_vpc_name = "private-app-vpc"
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
aws_instance.private_app_server
aws_security_group.private_app_sg
aws_subnet.private_app_subnet
aws_vpc.private_app_vpc
```

### Explanation

Confirms Terraform manages all resources.

---

## Verify Subnet Is Private

### Command

```bash
terraform state show aws_subnet.private_app_subnet
```

### Important Output

```text
map_public_ip_on_launch = false
```

### Explanation

Instances launched in this subnet will not receive public IP addresses automatically.

This makes the subnet private.

---

## Verify Security Group Restriction

### Command

```bash
terraform state show aws_security_group.private_app_sg
```

### Important Output

```text
cidr_blocks = [
  "10.0.0.0/16"
]
```

### Explanation

Only resources inside the VPC can access the EC2 instance.

No external traffic is allowed.

---

# Resource Breakdown

---

# VPC

```hcl
resource "aws_vpc" "private_app_vpc"
```

Purpose:

* Provides isolated networking environment
* Controls IP ranges
* Acts as a security boundary

---

# Subnet

```hcl
resource "aws_subnet" "private_app_subnet"
```

Purpose:

* Segments network
* Hosts private workloads
* Prevents automatic public IP assignment

---

# Security Group

```hcl
resource "aws_security_group" "private_app_sg"
```

Purpose:

* Stateful firewall
* Allows traffic only from inside VPC

---

# EC2 Instance

```hcl
resource "aws_instance" "private_app_server"
```

Purpose:

* Hosts application workload
* Runs inside private subnet
* Protected by VPC security controls

---

# Why map_public_ip_on_launch = false ?

Without this:

```text
EC2 Instance
    │
    ▼
Receives Public IP
```

With this:

```text
EC2 Instance
    │
    ▼
Private IP Only
```

Benefits:

* Better security
* Reduced attack surface
* Internal-only communication

---

# Network Flow

```text
Application Server A
       │
       ▼
10.0.0.0/16
       │
       ▼
Security Group
       │
       ▼
Application Server B
```

External internet traffic:

```text
Internet
   │
   ▼
DENIED
```

---

# Scenario Based Interview Questions & Answers

# L1 – Junior Engineer

## Q1. What is a VPC?

### Answer

A Virtual Private Cloud (VPC) is a logically isolated network in AWS.

---

## Q2. What is a Subnet?

### Answer

A subnet is a segmented IP range inside a VPC.

---

## Q3. What is a Security Group?

### Answer

A Security Group acts as a virtual firewall for AWS resources.

---

## Q4. What makes a subnet private?

### Answer

A subnet becomes private when resources inside it do not receive public IP addresses and do not have internet routing.

---

# L2 – Mid-Level Engineer

## Q1. Difference Between Public and Private Subnet?

### Public Subnet

```text
Has Public IP
Internet Accessible
```

### Private Subnet

```text
Private IP Only
Internal Access Only
```

---

## Q2. Why disable auto-assign public IP?

### Answer

To prevent instances from becoming publicly reachable.

---

## Q3. Why restrict Security Group traffic to VPC CIDR?

### Answer

To ensure only internal resources communicate with the server.

---

# L3 – Senior Engineer

## Q1. How would you secure a production private subnet?

### Answer

Implement:

```text
Private Subnets
Security Groups
NACLs
VPC Endpoints
IAM Roles
Encryption
```

---

## Q2. What are common mistakes in VPC design?

### Answer

```text
Overlapping CIDRs
Open Security Groups
Public Workloads in Private Zones
Missing Route Tables
```

---

## Q3. How would you troubleshoot connectivity inside a VPC?

### Answer

Check:

```text
Security Groups
Route Tables
NACLs
Subnet CIDRs
VPC Peering
DNS Resolution
```

---

# L4 – Architect Level

## Q1. Design a Highly Secure Enterprise VPC

### Answer

```text
Internet
    │
    ▼
Public ALB
    │
    ▼
Private Application Tier
    │
    ▼
Private Database Tier
```

Benefits:

* Layered Security
* Zero Trust Principles
* Reduced Attack Surface

---

## Q2. How would you design multi-region networking?

### Answer

Use:

```text
Transit Gateway
VPC Peering
PrivateLink
Direct Connect
```

---

# AWS Scenario Based Questions

## Scenario 1

A server inside the private subnet cannot access another instance.

### Troubleshooting

Check:

```text
Security Groups
NACLs
Route Tables
Subnet Association
```

---

## Scenario 2

An EC2 instance unexpectedly receives a public IP.

### Root Cause

```text
map_public_ip_on_launch = true
```

---

## Scenario 3

A private server is exposed to the internet.

### Root Cause

Security Group contains:

```text
0.0.0.0/0
```

on inbound rules.

---

# Real AWS Production Incident RCA

## Incident

Private database server became internet-accessible.

### Impact

Sensitive data exposure risk.

---

## Root Cause

Security Group configured:

```text
0.0.0.0/0
```

for database port.

---

## Resolution

* Restricted CIDR
* Implemented Security Reviews
* Added Terraform Validation

---

## Prevention

```text
Least Privilege
Code Reviews
Security Scanning
Terraform Policy Checks
```

---

# Full DevOps Mock Interview

## Q1. What is Terraform State?

### Answer

Terraform state stores metadata about managed resources.

---

## Q2. Why is Terraform State Important?

### Answer

Terraform uses state to track:

```text
Existing Resources
Changes Required
Dependencies
```

---

## Q3. What Happens During terraform plan?

### Answer

Terraform compares current state with desired state.

---

## Q4. What Happens During terraform apply?

### Answer

Terraform creates, modifies, or removes infrastructure.

---

## Q5. Why Use Infrastructure as Code?

### Answer

Benefits:

```text
Automation
Consistency
Version Control
Repeatability
Compliance
```

---

# Key Learnings

1. VPC provides network isolation.
2. Private subnets improve security.
3. Security Groups act as stateful firewalls.
4. Disabling public IP assignment creates private workloads.
5. Terraform automates infrastructure provisioning.
6. Outputs provide resource information after deployment.
7. Infrastructure should be verified after deployment.

---

# Final Result

```text
Task Status          : SUCCESS

Resources Created:

1. VPC
   Name : private-app-vpc
   CIDR : 10.0.0.0/16

2. Subnet
   Name : private-app-subnet
   CIDR : 10.0.1.0/24
   Public IP Assignment : Disabled

3. Security Group
   Access : Internal VPC Only

4. EC2 Instance
   Name : private-app-server
   Type : t2.micro

Terraform Init       : Successful
Terraform Validate   : Successful
Terraform Plan       : Successful
Terraform Apply      : Successful

Verification         : Successful

Challenge Status     : PASSED
```

