# AWS ECR Containerization Pipeline (Nautilus DevOps Task)

## 🚀 Scenario Overview
The Nautilus DevOps team was tasked with setting up a secure, containerized deployment workflow for a critical Python web service. To ensure absolute privacy, low latency, and granular access control, the infrastructure required a private container registry. 

As a Junior DevOps Engineer, I implemented this by spinning up an isolated AWS Elastic Container Registry (ECR), configuring local client authentication, writing an optimized multi-stage build configuration, and implementing automated image cleanup retention rules to control storage expenditure.

## 🛠️ Objectives Completed
1. **Private Registry Provisioning**: Created a private Amazon ECR repository named `nautilus-ecr`.
2. **Local Environment Authentication**: Securely logged into the remote AWS container registry using the local Docker client.
3. **Multi-Stage Build Optimization**: Built a highly compact Docker image from a local configuration workspace.
4. **Secure Image Distribution**: Tagged and shipped the runtime image to the private registry using the `latest` tag matrix.
5. **Cost Optimization & Lifecycles**: Created lifecycle policies within the AWS Management Console to prevent stale, untagged images from incurring unnecessary cloud costs.

---

## 💻 Step-by-Step Implementation

### Step 1: Create the AWS ECR Private Repository
Using the AWS Management Console or AWS CLI, provision the private tracking repository:
```bash
aws ecr create-repository --repository-name nautilus-ecr --region us-east-1
```
- Via the console, create the private repository, click the Create repository button on the top right.
Configure the settings:
- Visibility settings: Select Private.
- Repository name: Enter exactly nautilus-ecr.

Keep all other settings (like Tag immutability and Image scanning) at their defaults.Click Create repository at the bottom of the page


### Step 2: Get the Push Commands from the Console
1. In the ECR console list, click on your newly created nautilus-ecr repository.
2. Look at the top right of the screen and click the View push commands button.
3. A popup window will appear showing you the exact commands pre-filled with your AWS Account ID and Region. Leave this window open to copy the commands.

![](https://github.com/user-attachments/assets/38386ac8-8707-406a-b7d4-7d7186cdc31b)



### Step 3: Run the Commands on your aws-client Host
Retrieve a secure registry token and tunnel it into the local Docker daemon:
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <YOUR-ACCOUNT-ID>.dkr.ecr.us-east-1.amazonaws.com
```

### Step 4: Build the Container Image
Navigate to the source directory and compile the optimized Dockerfile:
```bash
cd /root/pyapp
docker build -t nautilus-ecr .
```
- Ensure that the Dockerfile is well configured

![](https://github.com/user-attachments/assets/5bcf2157-58a2-41e2-bcad-4240e7e8281c)

### Step 5: Tag and Push the Artifact
Map the local build image signature to the remote cloud storage path and execute the upload pipeline:
```bash
# Tag the image
docker tag nautilus-ecr:latest <YOUR-ACCOUNT-ID>://

# Push to Amazon ECR
docker push <YOUR-ACCOUNT-ID>://
```

---

## 📈 Cost Optimization Strategy (ECR Lifecycle Policies)
To automate asset management and minimize standard S3 storage fees, an ECR Lifecycle Policy was attached to the repository using the following governance logic:


| Rule Priority | Description | Target Image Status | Match Criteria | Consequence |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Expunge Stale Images | Untagged | > 14 days since push | **Expire / Delete** |
| **2** | Keep Fresh Build History | Any | Count exceeds 10 | **Expire / Delete** |

This guarantees that whenever a new `latest` build replaces an older deployment, the deprecated untagged layers are swept away within two weeks, keeping infrastructure lean.

![](https://github.com/user-attachments/assets/d0af39ea-18f8-4e4e-a525-cdb51e228bc6)
![](https://github.com/user-attachments/assets/fda08ffa-2808-4607-9d5e-de7649f2fa60)