# Setting Up an Application Load Balancer for an EC2 Instance

## Scenario:
The aim is to establish an Application Load Balancer (ALB) in front of an EC2 instance where an Nginx server is currently running. While the Nginx server currently serves a sample page, the team plans to deploy the actual application later.

Set up an Application Load Balancer named datacenter-alb.
Create a target group named datacenter-tg.
Create a security group named datacenter-sg to open port 80 for the public.
Attach this security group to the ALB.
The ALB should route traffic on port 80 to port 80 of the datacenter-ec2 instance.
Make appropriate changes in the default security group attached to the EC2 instance if necessary.


## Step-by-step Solution

### 1. Launching the EC2 instance with nginx showing the default welcome page
In the AWS Console, click launch instance with the configurations:

| Setting       | Value                                     |
| ------------- | ----------------------------------------- |
| Name          | datacenter-ec2                            |
| AMI           | Amazon Linux 2023                         |
| Instance Type | t2.micro (or any free tier)               |
| Key Pair      | Proceed without one - for this task       |


In the network settings section, click edit and configure it:


| Setting               | Value                              |
| --------------------- | ---------------------------------- |
| VPC                   | default VPC                        |
| Auto-assign Public IP | Enable                             |
| Security Group        | Use the default                    |


In the user data section, install the nginx :
Expand the advanced deteils section and in the User data section, paste:

```bash
#!/bin/bash

dnf update -y

dnf install nginx -y

systemctl enable nginx
systemctl start nginx
```

Click launch instance


### 2. Create Security Group for ALB

Navigate to:

EC2 → Security Groups

Click Create security group

Basic details
| Field               | Value                      |
| ------------------- | -------------------------- |
| Security group name | datacenter-sg              |
| Description         | Allow HTTP traffic to ALB  |
| VPC                 | Same VPC as datacenter-ec2 |

Inbound Rules

Add:

| Type | Protocol | Port | Source                    |
| ---- | -------- | ---- | ------------------------- |
| HTTP | TCP      | 80   | Anywhere-IPv4 (0.0.0.0/0) |

Outbound Rules

Leave default:

| Type        | Destination |
| ----------- | ----------- |
| All Traffic | 0.0.0.0/0   |


Click Create security group


![SecurityGroup](https://github.com/user-attachments/assets/0bf38399-12d3-491b-8832-0d23195df87c)

### 3. Create Target Group

Navigate to:

EC2 → Target Groups

Click Create target group

Configure


| Setting               | Value                      |
| --------------------- | -------------------------- |
| Target type           | Instances                  |
| Target group name     | datacenter-tg              |
| Protocol              | HTTP                       |
| Port                  | 80                         |
| VPC                   | Same VPC as datacenter-ec2 |
| Health check protocol | HTTP                       |
| Health check path     | /                          |


Click Next

Register Targets

Select:

datacenter-ec2

Click:

Include as pending below

Click:

Create target group


### 4. Create Application Load Balancer

Navigate to:

EC2 → Load Balancers

Click Create Load Balancer

Choose:

Application Load Balancer

Click Create

Configurations:

| Setting            | Value           |
| ------------------ | --------------- |
| Load Balancer Name | datacenter-alb  |
| Scheme             | Internet-facing |
| IP Address Type    | IPv4            |


Network Mapping

Select:

Same VPC as datacenter-ec2(default vpc)
At least 2 Availability Zones/Subnets since AWS requires ALB to span multiple AZs.


Security Groups

Select:

datacenter-sg

Uncheck the default security group


Listener

| Protocol | Port |
| -------- | ---- |
| HTTP     | 80   |


Forward Action

Choose:

datacenter-tg
 
Click Create Load Balancer

Wait until status becomes:
>active
![LoadBalancer](https://github.com/user-attachments/assets/99a70c34-bb3b-4d58-aa02-bbcaa4fb6993)

### 5. Modify EC2 Security Group

The ALB can only forward traffic if the EC2 instance allows HTTP traffic.

Navigate:

EC2 → Instances → datacenter-ec2

Open the attached security group ("default").

Click Edit inbound rules

Ensure there is a rule allowing inbound traffic from the datacenter security group:


| Type          | Protocol          | Port          | Source        |
| --------------| ------------------| --------------| ------------- |
| HTTP          | TCP               | 80            | datacenter-sg |

This ensures only the ALB can reach the EC2 instance.

Save the changes

![SGmodification](https://github.com/user-attachments/assets/0234d9a3-faa5-4abc-bcc5-90098962c4b4)

### 6. Verify Health Checks

Navigate:

Target Groups → datacenter-tg → Targets

Wait a minute.

You should see:
```yaml
datacenter-ec2
Healthy
```
![Healthchecks](https://github.com/user-attachments/assets/305fb364-bdfa-43d7-ae63-82ac20f2d11a)

### 6. Test the ALB

Navigate:

EC2 → Load Balancers

Open:

datacenter-alb

Copy the DNS Name and paste it in the browser:
```yaml
Welcome to nginx!
```

![](https://github.com/user-attachments/assets/dae42a21-ed80-4c39-9f82-43f277a844b5)

## Flow Chart:

| Resource                  | Name                    |
| ------------------------- | ----------------------- |
| Application Load Balancer | datacenter-alb          |
| Target Group              | datacenter-tg           |
| Security Group            | datacenter-sg           |
| Listener                  | HTTP :80                |
| Target                    | datacenter-ec2          |
| Health Status             | Healthy                 |
| Traffic Flow              | ALB → datacenter-ec2:80 |

