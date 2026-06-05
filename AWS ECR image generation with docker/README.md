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

### Step 2: Authenticate Local Docker Client to AWS
Retrieve a secure registry token and tunnel it into the local Docker daemon:
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <YOUR-ACCOUNT-ID>.dkr.ecr.us-east-1.amazonaws.com
```

### Step 3: Build the Container Image
Navigate to the source directory and compile the optimized Dockerfile:
```bash
cd /root/pyapp
docker build -t nautilus-ecr .
```

### Step 4: Tag and Push the Artifact
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
