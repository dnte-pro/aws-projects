# Task: Enable Internet Access for a Private EC2 Using a NAT Instance


## Objective

The goal was to provide internet access to an existing private EC2 instance (nautilus-priv-ec2) running in a private subnet (nautilus-priv-subnet) within the VPC nautilus-priv-vpc.

Instead of using a NAT Gateway, a NAT Instance was deployed to reduce costs. Once connectivity was established, the private EC2 instance would automatically upload a test file (nautilus-test.txt) to the S3 bucket nautilus-nat-5631 via an existing cron job.

## Task

The following components already exist in the environment: 
1) A VPC named nautilus-priv-vpc and a private subnet named nautilus-priv-subnet have been created. 
2) An EC2 instance named nautilus-priv-ec2 is already running in the private subnet. 
3) The EC2 instance is configured with a cron job that uploads a test file to the S3 bucket nautilus-nat-5631 every minute. Upload will only succeed once internet access is established. 

Your task is to: 
- Create a new public subnet named nautilus-pub-subnet in the existing VPC. 
- Launch a NAT Instance in the public subnet using an Amazon Linux 2023 AMI and name it nautilus-nat-instance. 
- Configure this instance to act as a NAT instance. Make sure to use a custom security group for this instance. After the configuration, verify that the test file nautilus-test.txt appears in the S3 bucket nautilus-nat-5631. 

This indicates successful internet access from the private EC2 instance via the NAT Instance. Note: iptables is not installed by default on Amazon Linux 2023. You will need to install and enable it before configuring NAT setup.


## Architecture


```text
+----------------------+
|      Internet        |
+----------+-----------+
           |
           |
+----------v-----------+
|   Internet Gateway   |
+----------+-----------+
           |
           |
+----------v-----------+
|  Public Subnet       |
|  nautilus-pub-subnet |
|                      |
|  NAT Instance        |
|  nautilus-nat-instance
+----------+-----------+
           |
           |
+----------v-----------+
|  Private Subnet      |
|  nautilus-priv-subnet|
|                      |
|  nautilus-priv-ec2   |
+----------+-----------+
           |
           |
+----------v-----------+
|      S3 Bucket       |
|  nautilus-nat-5631   |
+----------------------+

```

### 1. Create a Public Subnet

- Create the subnet
VPC → Subnets → Create Subnet:

Configuration:


| Parameter | Value               |
| --------- | ------------------- |
| Name      | nautilus-pub-subnet |
| VPC       | nautilus-priv-vpc   |
| CIDR      | 10.1.2.0/24         |

The VPC CIDR is ```10.1.0.0/24```

![]()


### 2. Enable Auto-Assign Public IP
Navigate to VPC → Subnets,


Select nautilus-pub-subnet, 


Choose nautilus-pub-subnet,


Enable: ```Auto-assign public IPv4 address ```


### 3. Create and Attach an Internet Gateway

Navigate to:
```text
VPC → Internet Gateways
```

Create:
```text
nautilus-igw
```


Attach it to:
```text
nautilus-priv-vpc
```

### 4. Create a Public Route Table

Navigate to:
```text
VPC → Route Tables
```

Create:
```text
nautilus-pub-rt
```

Add route:
| Destination | Target           |
| ----------- | ---------------- |
| 0.0.0.0/0   | Internet Gateway |



Associate:

```text
nautilus-pub-subnet
```

### 5. Create a Security Group for the NAT Instance

Navigate to:

```text
EC2 → Security Groups
```
Create:
```text
nautilus-nat-sg
```

| Type        | Source      |
| ----------- | ----------- |
| SSH         | 0.0.0.0/0   |
| All Traffic | 10.1.0.0/16 |


### 6. Launch the NAT Instance

Navigate to:
```text
EC2 → Launch Instance
```

Configuration:
| Parameter      | Value                 |
| -------------- | --------------------- |
| Name           | nautilus-nat-instance |
| AMI            | Amazon Linux 2023     |
| Instance Type  | t2.micro              |
| VPC            | nautilus-priv-vpc     |
| Subnet         | nautilus-pub-subnet   |
| Public IP      | Enabled               |
| Security Group | nautilus-nat-sg       |

### 7. Disable Source/Destination Check

AWS EC2 instances normally validate that traffic is destined for themselves. A NAT Instance forwards traffic on behalf of other instances, so this validation must be disabled.

Navigate to:
```text
EC2 → Instances → nautilus-nat-instance
```

Choose:

```text
Actions → Networking → Change source/destination check
```

Set:
```text
Disabled
```


### 8. Install iptables

SSH into the NAT instance or use the EC@ Instance Connect:
- Become root
```bash
sudo su -
```

Install IPtables

```bash
dnf install -y iptables-services
```

Enable the service:

```bash
systemctl enable iptables
systemctl start iptables
```

Verify:
```bash
systemctl status iptables
```


### 9. Enable IP forwarding
Check current status:
```bash
sysctl net.ipv4.ip_forward
```

Enable:
```bash
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```

Verify:
```bash
sysctl net.ipv4.ip_forward
```

Output:
```text
net.ipv4.ip_forward = 1
```

### 10. Configure NAT Rules
Add masquerading:
```bash
iptables -t nat -A POSTROUTING -o ens5 -j MASQUERADE
```

Allow forwarding:
```bash
iptables -A FORWARD -i ens5 -m state --state RELATED,ESTABLISHED -j ACCEPT

iptables -A FORWARD -o ens5 -j ACCEPT
```

Save:
```bash
iptables-save > /etc/sysconfig/iptables
```

Verify:
```bash
iptables-save > /etc/sysconfig/iptables
```

Output:
```text
MASQUERADE
```

### 11. Configure Private Route Table
Locate the route table associated with:
```text
nautilus-priv-subnet
```

Add route:
| Destination | Target           |
| ----------- | ---------------- |
| 0.0.0.0/0   | NAT Instance ENI |

AWS displays the NAT instance as its Network Interface (ENI).

### 12 Final Validation
Wait approximately:
```text
1–2 minutes
```
for the cron job on ```nautilus-priv-ec2``` to execute.

Navigate to:
```bash
S3 → nautilus-nat-5631
```
Verify the presence of:
```text
nautilus-test.txt
```



>Problem Encountered
After completing all configuration steps, the file:
```text
nautilus-test.txt
```
was still not appearing in the S3 bucket.

Initial verification showed:
```bash
sysctl net.ipv4.ip_forward
```
Output:
```text
net.ipv4.ip_forward = 1
```
And:
```bash
iptables -t nat -L -n
```
showed:
```text
MASQUERADE
```
Both appeared correct.
> Root Cause
Inspection of the FORWARD chain revealed:
```bash
iptables -L FORWARD -n -v
```
Output:
```bash
REJECT all -- * * 0.0.0.0/0 0.0.0.0/0 reject-with icmp-host-prohibited

ACCEPT all -- ens5 * state RELATED,ESTABLISHED

ACCEPT all -- * ens5
```
The REJECT rule appeared before the ACCEPT rules.

Because iptables evaluates rules from top to bottom, all forwarded traffic was being rejected before reaching the NAT rules.

>Fix

Display numbered rules:
```bash
iptables -L FORWARD --line-numbers
```
Delete the REJECT rule:
```bash
iptables -D FORWARD 1
```
Verify:
```bash
iptables -L FORWARD -n -v
```
Result:

ACCEPT all -- ens5 * state RELATED,ESTABLISHED

ACCEPT all -- * ens5

Save changes:

iptables-save > /etc/sysconfig/iptables

Restart:

systemctl restart iptables