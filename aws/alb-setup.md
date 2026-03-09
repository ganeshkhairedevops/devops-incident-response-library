# AWS Application Load Balancer Setup for DevOps

## Problem Statement

The DevOps team is currently working on setting up a simple application on the AWS cloud. They aim to establish an Application Load Balancer (ALB) in front of an EC2 instance where an Nginx server is currently running. While the Nginx server currently serves a sample page, the team plans to deploy the actual application later.

### Requirements

1. Set up an **Application Load Balancer** named **application-alb**.
2. Create a **target group** named **application-tg**.
3. Create a **security group** named **application-sg** to open **port 80** for the public.
4. Attach this security group to the **ALB**.
5. The **ALB should route traffic on port 80 to port 80** of the **application-ec2** instance.
6. Make appropriate changes in the **default security group attached to the EC2 instance if necessary**.
---
Below is a clear step-by-step guide in the AWS Console to complete this lab. The goal is:

ALB: `application-alb`

Target Group: `application-tg`

Security Group: `application-sg`

Route: ALB 80 → EC2 80

Instance: `application-ec2`

---

# Solution

# Step 1: Login to AWS Console

1. Make sure the region is set to **us-east-1 (N. Virginia)**.

---

# Step 2: Create Security Group for ALB

1. Go to **EC2 Dashboard**.
2. Click **Security Groups**.
3. Click **Create Security Group**.

Fill the details:

- **Security group name:** `application-sg`
- **Description:** Allow HTTP traffic
- **VPC:** Select the same VPC used by the EC2 instance.

### Inbound Rule
Add:
| Type | Protocol | Port | Source |
|-----|-----|-----|-----|
| HTTP | TCP | 80 | 0.0.0.0/0 |

Leave **Outbound Rules** as default `All traffic`.

Click **Create Security Group**.

---

# Step 3: Create Target Group

1. Go to **EC2 → Target Groups**.
2. Click **Create Target Group**.

Configuration:

- **Target type:** Instances
- **Target group name:** `application-tg`
- **Protocol:** HTTP
- **Port:** 80
- **VPC:** Same VPC as the EC2 instance.

Click **Next**.

### Register Target

1. Select the instance **nautilus-ec2**.
2. Click **Include as pending below**.
3. Click **Create Target Group**.

---

# Step 4: Create Application Load Balancer

1. Go to **EC2 → Load Balancers**.
2. Click **Create Load Balancer**.
3. Choose **Application Load Balancer**.

Fill the configuration:

### Basic Configuration

- **Name:** `application-alb`
- **Scheme:** Internet-facing
- **IP address type:** IPv4

### Network Mapping

- Select the **same VPC as EC2**.
- Select **at least two subnets**.

### Security Groups

- Remove default security group.
- Attach **application-sg**.

### Listener and Routing

- **Listener Protocol:** HTTP
- **Port:** 80
- **Forward to:** `application-tg`

Click **Create Load Balancer**.

---

# Step 5: Update EC2 Security Group

The EC2 instance must allow traffic from the ALB.

1. Go to **EC2 → Instances**.
2. Select **application-ec2**.
3. Click the **Security Group** attached to it.
4. Edit **Inbound Rules**.

Add rule:

| Type | Port | Source |
|-----|-----|-----|
| HTTP | 80 | application-sg |

Save rules.

---

# Step 6: Verify Target Health

1. Go to **EC2 → Target Groups**.
2. Open **application-tg**.
3. Check the **Targets tab**.

The instance status should become:

```
healthy
```

after some time.

---

# Step 7: Test the Load Balancer

1. Go to **EC2 → Load Balancers**.
2. Select **application-alb**.
3. Copy the **DNS Name**.

Example:

```
http://application-alb-123456.us-east-1.elb.amazonaws.com
```

4. Open it in a browser.

You should see the **Nginx default page**.

---

# Final Architecture

```
Internet
   │
   ▼
Application Load Balancer (application-alb)
   │
   ▼
Target Group (application-tg)
   │
   ▼
EC2 Instance (application-ec2)
   │
   ▼
Nginx Web Server (Port 80)
```

---

# Result

The **Application Load Balancer is successfully configured** and routing traffic from the internet to the **Nginx server running on the EC2 instance**.