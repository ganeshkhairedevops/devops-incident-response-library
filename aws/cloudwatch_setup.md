# ☁️ AWS EC2 & CloudWatch Alarm Setup

## 📌 Task Overview

The DevOps team is tasked with:

- Launching an EC2 instance
- Creating a CloudWatch alarm to monitor CPU utilization
- Sending notifications using an SNS topic

---

# 🖥️ EC2 Instance Details

| Attribute | Value |
|----------|------|
| Instance Name | ubuntu-ec2 |
| AMI | Ubuntu Server |
| Instance Type | t2.micro |
| Region | us-east-1 |

---

# 🚀 Steps

## Step 1. Launch EC2 instance (ubuntu-ec2)
Navigate to:

`AWS Console → EC2 → Instances → Launch Instance`
- Choose AMI: Ubuntu Server
- Select Instance Type: t2.micro
- Configure Instance: Default settings
- Add Storage: Default
- Add Tags: Name = ubuntu-ec2
- VPC: Default
- Configure Security Group: Allow SSH (port 22) and HTTP (port 80)
- Review and Launch

Click Launch Instance and wait until status is Running.


## Step 2: Create CloudWatch Alarm
Navigate to:
`AWS Console → CloudWatch → Alarms → Create Alarm`
## Step 3: Select Metric
- Click Select Metric
- Go to:

EC2 → Per-Instance Metrics → CPUUtilization
- Select instance: ubuntu-ec2
- Click Select Metric

## Step 4: Configure Alarm
| Setting	| Value |
|-----------|------------|
| Statistic	| Average |
| Period	| 5 minutes |
|Threshold | Type	Static |
|Condition	| >= |
|Threshold Value	| 90 |

**Condition Expression**

CPUUtilization >= 90 for 1 datapoint within 5 minutes
## Step 5: Configure Notification
- Select:

Send notification to existing SNS topic

- Choose

SNS topic: ubuntu-sns-topic

## Step 6: Name the Alarm
- Alarm name: ubuntu-alarm
- Description: Alarm for high CPU utilization on ubuntu-ec2
- Click Create Alarm

## Step 7: Verification
- Ensure EC2 instance is running
- Verify alarm is created and in OK state
- Test alarm by simulating high CPU load (optional)

Navigate to:

CloudWatch → Alarms

Check:

Alarm Name: ubuntu-alarm

State: OK

Metric: CPUUtilization

Threshold: >= 90%

---
## Optional: Test the Alarm
To simulate high CPU load on the EC2 instance, you can SSH into the instance and run a CPU stress test:
```bash
sudo apt-get update
sudo apt-get install -y stress
# Generate CPU load:
stress --cpu 4 --timeout 300
```
---

# ✅ Result

- EC2 instance running
- Alarm created
- Notifications enabled
