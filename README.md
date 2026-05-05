# Secure Private Container Registry with Image Lifecycle Policies

## Project Overview

This project demonstrates how to implement a secure private container registry using AWS services. It ensures secure storage, controlled access, and automated cleanup of Docker images.

The system uses **Amazon ECR** as a private registry, **IAM** for access control, and **lifecycle policies** for automatic image management.

---

## Objectives

* Create a private container registry
* Push multiple Docker image versions
* Implement IAM-based access control
* Configure lifecycle policies for cleanup
* Validate security using unauthorized access

---

## Technologies Used

* Docker
* AWS EC2
* Amazon ECR
* AWS IAM
* AWS CLI

---

## Architecture Workflow

Developer → Build Image → Push to ECR → Apply Lifecycle Policy → Secure Access via IAM

---

## Step 1: Launch EC2 Instance

* Launch Ubuntu EC2
* Connect via SSH
* Install Docker & AWS CLI

### Commands:

```
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

---

## Step 2: Configure AWS CLI

```
aws configure
```

Provide Access Key, Secret Key, Region, Output format

---

## Step 3: Create ECR Repository

* Go to AWS Console → ECR
* Create Private Repository
* Name: `my-secure-repo`

---

## Step 4: Docker Application

### app.py

```
print("Hello DevOps Secure Registry Project")
```

### Dockerfile

```
FROM python:3.9-slim
WORKDIR /app
COPY app.py .
CMD ["python","app.py"]
```

---

##  Step 5: Build Image

```
docker build -t secure-app:v1 .
```

---

##  Step 6: Login to ECR

```
aws ecr get-login-password --region us-east-1 | \
docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com
```

---

##  Step 7: Tag Image

```
docker tag secure-app:v1 <repo-url>:v1
```

---

##  Step 8: Push Image

```
docker push <repo-url>:v1
```

Multiple versions pushed:
`v1, v2, v3, v4, v5`

---

##  Step 9: Lifecycle Policy

### lifecycle-policy.json

```
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep last 3 images",
      "selection": {
        "tagStatus": "tagged",
        "countType": "imageCountMoreThan",
        "countNumber": 3
      },
      "action": { "type": "expire" }
    }
  ]
}
```

### Result:

Before: v1, v2, v3, v4, v5
After: v3, v4, v5

---

##  Step 10: Repository Policy

### repository-policy.json

```
{
  "Version": "2008-10-17",
  "Statement": [
    {
      "Sid": "AllowPushPullFromDevOpsRole",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<account-id>:role/DevOpsRole"
      },
      "Action": [
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:BatchCheckLayerAvailability",
        "ecr:PutImage"
      ]
    }
  ]
}
```

---

##  Step 11: Security Validation

* Unauthorized IAM user tried to push image
* Result: **AccessDeniedException**

 Confirms access restriction works

---

##  Results

* Secure image storage
* Controlled access using IAM
* Automated cleanup using lifecycle policy
* Reduced storage usage

---

##  Conclusion

This project successfully demonstrates secure container image management with automated lifecycle handling and strong access control using AWS services.

---

##  Key Learnings

* ECR as private registry
* IAM for security
* Docker image versioning
* Lifecycle policy for cost optimization

---
