# Task: Configuring Secure SSH Access to an EC2 Instance

## The Scenario:

The Nautilus DevOps team needs to set up a new EC2 instance that can be accessed securely from their landing host
(aws-client). The instance should be of type t2.micro and named datacenter-ec2. A new SSH key with name id_rsa should be 
created on the aws-client host under the/root/.ssh/ folder, if it doesn't already exist. This key should then be added to
the root user's authorised keys on the EC2 instance, allowing passwordless SSH access from the aws-client host.

### Going about it

The goal:

Create:

- An EC2 instance:
- Name: xfusion-ec2
- Type: t2.micro

Then:

- Generate an SSH key on aws-client
- Add the public key to the EC2 instance
- Enable passwordless SSH access

## Steps:

1. Check if id_rsa already exists

In the aws-client(Kodekloud terminal provided by the lab), check if the access keys are present

```bash
ls -l /root/.ssh/id_rsa
```

If file exists, do not recreate it; if it doesn't exist then create


2. Generate SSH key pair

To generate the SSH keypair in the aws client, run:

```bash
ssh-keygen -t rsa -b 2048 -f /root/.ssh/id_rsa -N ""
```

| Part                   | Meaning                 |
| ---------------------- | ----------------------- |
| `ssh-keygen`           | Generates SSH keys      |
| `-t rsa`               | Use RSA encryption      |
| `-b 2048`              | Key size = 2048 bits    |
| `-f /root/.ssh/id_rsa` | Save key with this name |
| `-N ""`                | No passphrase           |


Two files are created;
| File                    | Purpose     |
| ----------------------- | ----------- |
| `/root/.ssh/id_rsa`     | PRIVATE key |
| `/root/.ssh/id_rsa.pub` | PUBLIC  key  |

For this task, the private key stays on the aws client, whereas the public key gets copied to the EC2 instance.


3. Create the EC2 instance

Create the EC2 instance via the terminal or the console.
- Via the terminal run:

```bash
aws ec2 run-instances \
  --image-id ami-xxxxxxxx \
  --instance-type t2.micro \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=datacenter-ec2}]'
```

- **Note**: Replace the xxxxxx with the prefered AMI id of choice (Like Ubuntu, CentOS, Amazon Linux or any other)


| Option                     | Meaning           |
| -------------------------- | ----------------- |
| `run-instances`            | Launch EC2        |
| `--image-id`               | OS image          |
| `--instance-type t2.micro` | Instance size     |
| `--tag-specifications`     | Set instance name |


4. Verify the instance

Verify that the instances have been created:

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=datacenter-ec2" \
  --query "Reservations[*].Instances[*].[InstanceId,PublicIpAddress,State.Name]" \
  --output table
```

The command fetches the instance ID, Public IP and the state of the instance and outputs it in a table.

5. Allow SSH into the instance

Ensure that the security groups allows ssh:
- In the console, open the instance,
- in the security tab, open the security group,
- Add inbound rule that allows ssh on port 22 from Anywhere IPv4

| Type | Port | Source    |
| ---- | ---- | --------- |
| SSH  | 22   | 0.0.0.0/0 |



6. Login to the EC2 

Login to the EC2 instance via the connect in the console

7. Become root user

- You have to be root because the task wants root user's authorized keys configured
```bash
sudo su -
```

8.  Copy the public key on aws-client

Back on the aws client, display the public key:

```bash
cat /root/.ssh/id_rsa.pub
```

- Copy the output and save it in a text editor

9. Add the public key to the EC2 authorized_keys

Create the directory /root/.ssh


```bash
mkdir -p /root/.ssh
chmod 700 /root/.ssh
```

Edit the authorized keys:

```bash
vi /root/.ssh/authorized_keys
```

Paste the copied public key into the authorized_keys, save and exit

**When aws-client connects, SSH compares keys, if they match login succeeds automatically**


10. Set Correct permissions

Set the permissions for the file:

```bash
chmod 600 /root/.ssh/authorized_keys
```

Giving the correct permissions are important since SSH refuses insecure files.



11. Test passwordless SSH
 
In the aws-client:
```bash
ssh -i /root/.ssh/id_rsa root@<PUBIC-IP>
```

Copy IP from the console or from the table from the previous step- described instance.

