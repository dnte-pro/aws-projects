# Configuring Secure SSH Access to an EC2 Instance

This project documents the AWS lab task for securely connecting to an EC2 Linux instance using SSH.

## Task Overview

Goal: configure SSH access to an EC2 instance using secure key-based authentication and tightly scoped network access.

## What Was Done

- Launched or used an existing EC2 instance.
- Generated a key pair (`.pem`) for SSH authentication.
- Set strict key permissions locally:

	```bash
	chmod 400 <key-name>.pem
	```

- Configured Security Group inbound rules to allow SSH (`TCP 22`) only from a trusted source IP (not `0.0.0.0/0` unless required temporarily).
- Connected to the instance with SSH:

	```bash
	ssh -i <key-name>.pem ec2-user@<public-ip>
	```

	> Use the correct default username based on AMI (for example: `ec2-user`, `ubuntu`, or `admin`).

## Security Practices Applied

- Used key-based authentication instead of password login.
- Restricted SSH access by source CIDR in Security Group.
- Avoided exposing private keys and enforced least-privilege access.
- Kept inbound rules minimal (SSH only when needed).

## Validation

- SSH connection succeeded from the approved client IP.
- SSH connection failed from non-approved IPs.
- Instance remained reachable only through intended secure path.

## Troubleshooting Notes

- **Permission denied (publickey):** verify username, key file, and `chmod 400`.
- **Connection timeout:** verify Security Group rule, public IP/DNS, subnet route table, and internet gateway.
- **Host key verification warning:** check you are connecting to the correct host.

## Reference

- Primary lab instructions and acceptance criteria are in `task.md`.
