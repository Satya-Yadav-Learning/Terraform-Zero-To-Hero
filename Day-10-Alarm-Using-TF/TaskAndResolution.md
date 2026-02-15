# TaskAndResolution.md

# Terraform – CloudWatch Alarm for EC2 CPU Monitoring

---

## 1. Task Description

The Nautilus DevOps team was required to configure monitoring in AWS using Terraform.

### Requirements:

- Create a CloudWatch alarm named `datacenter-alarm`
- Monitor EC2 `CPUUtilization`
- Trigger alarm when CPU exceeds 80%
- Set evaluation period to 5 minutes (300 seconds)
- Use a single evaluation period
- Entire configuration must be implemented using Terraform
- Only one file allowed: `main.tf`
- Working directory: `/home/bob/terraform`

---

## 2. Environment Details

- OS: Linux (iac-server)
- Terraform Version: v1.x
- AWS Provider: hashicorp/aws v6.32.1
- AWS Region: us-east-1
- Working Directory: `/home/bob/terraform`
- Configuration File: `main.tf`

---

## 3. Solution Strategy

Since no EC2 instance ID was provided:

1. Fetch the latest Amazon Linux AMI.
2. Create a new EC2 instance.
3. Create a CloudWatch alarm.
4. Attach the alarm to the created EC2 instance.
5. Ensure complete automation using Terraform only.

This guarantees:

- No manual variable input
- No dependency failures
- Fully reproducible infrastructure
- Proper resource dependency handling

---

## 4. main.tf Configuration

```hcl
############################################
# AWS Provider Configuration
############################################
provider "aws" {
  region = "us-east-1"
}

############################################
# Fetch Latest Amazon Linux AMI
############################################
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

############################################
# Create EC2 Instance
############################################
resource "aws_instance" "datacenter_instance" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t2.micro"

  tags = {
    Name = "datacenter-instance"
  }
}

############################################
# Create CloudWatch Alarm
############################################
resource "aws_cloudwatch_metric_alarm" "datacenter_alarm" {

  alarm_name          = "datacenter-alarm"
  alarm_description   = "Alarm when EC2 CPU exceeds 80% for 5 minutes"

  namespace           = "AWS/EC2"
  metric_name         = "CPUUtilization"
  statistic           = "Average"

  period              = 300
  evaluation_periods  = 1
  threshold           = 80

  comparison_operator = "GreaterThanThreshold"
  treat_missing_data  = "missing"

  dimensions = {
    InstanceId = aws_instance.datacenter_instance.id
  }
}
```

---

## 5. Commands Executed with Explanation

### Step 1 – Navigate to Working Directory

```bash
cd /home/bob/terraform
```

Purpose:  
Ensures Terraform runs inside the correct project directory.

---

### Step 2 – Initialize Terraform

```bash
terraform init
```

Explanation:  
- Downloads AWS provider  
- Initializes backend  
- Creates `.terraform.lock.hcl`  
- Prepares the working directory  

Output:

```
Terraform has been successfully initialized!
```

---

### Step 3 – Plan Infrastructure

```bash
terraform plan
```

Explanation:  
- Validates configuration  
- Shows execution plan  
- Displays resources to be created  

Output:

```
Plan: 2 to add, 0 to change, 0 to destroy.
```

Meaning:  
- 1 EC2 instance  
- 1 CloudWatch alarm  

---

### Step 4 – Apply Infrastructure

```bash
terraform apply
```

When prompted:

```
yes
```

Explanation:  
- Creates EC2 instance  
- Creates CloudWatch alarm  
- Automatically resolves dependencies  

Output:

```
aws_instance.datacenter_instance: Creation complete
aws_cloudwatch_metric_alarm.datacenter_alarm: Creation complete

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

---

## 6. Verification

```bash
aws cloudwatch describe-alarms --alarm-names datacenter-alarm
```

Important Output Section:

```
"AlarmName": "datacenter-alarm",
"MetricName": "CPUUtilization",
"Threshold": 80.0,
"Period": 300,
"EvaluationPeriods": 1,
"ComparisonOperator": "GreaterThanThreshold"
```

Initial Alarm State:

```
"StateValue": "INSUFFICIENT_DATA"
```

Explanation:  
The alarm is waiting for CPU metric data from EC2.

---

# 7. Scenario-Based Interview Questions & Answers

---

## L1 – Junior Level

**Q1: What does `period = 300` mean?**  
A: It defines the evaluation window in seconds. 300 seconds equals 5 minutes.

**Q2: What does `evaluation_periods = 1` mean?**  
A: The alarm triggers if the threshold is breached in one evaluation window.

**Q3: Why use namespace `AWS/EC2`?**  
A: Because CPUUtilization belongs to EC2 metrics.

---

## L2 – Mid-Level Engineer

**Q1: What happens if CPU spikes for 1 minute?**  
A: The alarm may not trigger because the metric is averaged over 5 minutes.

**Q2: What is `treat_missing_data`?**  
A: It defines how CloudWatch handles missing metrics.

**Q3: How does Terraform manage resource order?**  
A: Through implicit dependency graph based on resource references.

---

## L3 – Senior DevOps Engineer

**Q1: How would you make this production-ready?**  
A:
- Attach SNS notifications  
- Enable detailed monitoring (1-minute metrics)  
- Add Auto Scaling policies  
- Use remote backend (S3 + DynamoDB locking)  
- Apply standardized tagging  

**Q2: How to reduce false alarms?**  
A:
- Increase evaluation periods  
- Use anomaly detection  
- Use percentile statistics  

---

## L4 – Architect Level

**Q1: How would you monitor 500 EC2 instances?**  
A:
- Use `for_each` in Terraform  
- Centralize monitoring in a dedicated AWS account  
- Implement cross-account observability  
- Use EventBridge automation  
- Configure composite alarms  

**Q2: When use composite alarms?**  
A: When combining multiple conditions into a single alerting mechanism.

---

# 8. Real AWS Incident RCA Examples

---

## Incident 1 – High CPU Not Alerted

**Root Cause:**  
Threshold set too high (90%).

**Impact:**  
Application performance degradation unnoticed.

**Resolution:**  
Lowered threshold and added Auto Scaling.

---

## Incident 2 – Short CPU Spikes Missed

**Root Cause:**  
Basic monitoring (5-minute granularity).

**Resolution:**  
Enabled detailed monitoring (1-minute granularity).

---

## Incident 3 – Alarm Without Notification

**Root Cause:**  
Alarm created without SNS action.

**Impact:**  
Operations team did not receive alerts.

**Resolution:**  
Attached SNS topic with email subscription.

---

# 9. Terraform Concepts Demonstrated

- Provider configuration  
- Data sources  
- Resource creation  
- CloudWatch metric configuration  
- Implicit dependency graph  
- Infrastructure as Code best practices  
- Automated monitoring setup  

---

# 10. Cleanup Command

```bash
terraform destroy
```

Purpose:  
Removes all created resources to avoid unnecessary AWS charges.

---

# 11. Key Learnings

- Monitoring must include alerting integration.  
- Terraform automatically manages dependencies.  
- Correct namespace and dimensions are critical.  
- Always ensure monitored resources exist.  
- Production monitoring should integrate notifications and scaling mechanisms.  
- Infrastructure as Code ensures reproducibility and consistency.

