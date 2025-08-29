# Workshop: Serverless DevOps CI/CD Pipeline on AWS

## 🚀 Overview

This repository demonstrates how to build and deploy a **Serverless DevOps CI/CD pipeline on AWS** using Docker, Amazon ECR, ECS, and Application Load Balancer.

The pipeline covers the complete workflow:

1. Launching an **EC2 instance**
2. Cloning this repository into the instance
3. Building a **Docker image** from the project
4. Pushing the Docker image to **Amazon Elastic Container Registry (ECR)**
5. Creating and attaching an **IAM Role** to EC2 for AWS permissions
6. Deploying the containerized application on **Amazon ECS (Fargate/EC2)**
7. Exposing the application via **Application Load Balancer (ALB)**

This setup ensures a scalable, containerized deployment workflow with minimal manual effort.

---

## 🛠️ Architecture

![AWS Architecture](https://d1.awsstatic.com/architecture-diagrams/ArchitectureDiagrams/containers-ecs-architecture-diagram.c7c3c162f48d2f6b3e8d7e6e6e7d82a9d3428.png)

Components:

* **AWS EC2** → Build server to prepare and push Docker image
* **Docker** → Containerization of the application
* **Amazon ECR** → Private container image registry
* **IAM Role** → Secure access to ECR and ECS services
* **Amazon ECS** → Orchestration of containers as tasks and services
* **Application Load Balancer** → Routing traffic to ECS tasks

---

## 📦 Prerequisites

* AWS account with administrative access
* AWS CLI installed & configured
* Docker installed on EC2 instance
* Git installed on EC2 instance

---

## ⚙️ Setup & Deployment

### 1. Launch an EC2 Instance

* Launch an **Amazon Linux 2** instance (t2.micro or higher).
* SSH into the instance.

```bash
ssh -i your-key.pem ec2-user@your-ec2-public-ip
```

### 2. Install Dependencies

```bash
sudo yum update -y
sudo yum install git -y
sudo yum install docker -y
sudo service docker start
sudo usermod -aG docker ec2-user
```

### 3. Clone Repository

```bash
git clone https://github.com/vipdanish/workshop.git
cd workshop
```

### 4. Build Docker Image

```bash
docker build -t workshop-app .
```

### 5. Authenticate & Push to Amazon ECR

* Create an ECR repository:

```bash
aws ecr create-repository --repository-name workshop-app
```

* Authenticate Docker with ECR:

```bash
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
```

* Tag and push image:

```bash
docker tag workshop-app:latest <account-id>.dkr.ecr.<region>.amazonaws.com/workshop-app:latest
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/workshop-app:latest
```

### 6. Create IAM Role & Attach to EC2

* Create IAM role with policies:

  * `AmazonEC2ContainerRegistryFullAccess`
  * `AmazonECSFullAccess`
* Attach role to EC2 instance.

### 7. Create ECS Cluster, Task, and Service

```bash
aws ecs create-cluster --cluster-name workshop-cluster
```

Define ECS task definition with the ECR image.
Then create a service with **Application Load Balancer** attached.

---

## 📊 Workflow Summary

1. **Code Commit** → Repo cloned on EC2
2. **Docker Build** → Image built from Dockerfile
3. **ECR Push** → Image stored in AWS registry
4. **IAM Role** → Grants permissions for ECS & ECR
5. **ECS Deployment** → Task & service created
6. **ALB Exposure** → App accessible via public DNS

---

## 🌐 Accessing the App

Once deployed, get the **ALB DNS name** from AWS console:

```bash
aws elbv2 describe-load-balancers --names workshop-lb --query 'LoadBalancers[*].DNSName' --output text
```

Open it in your browser to access the application.

---

## 📌 Next Steps

* Automate pipeline using **GitHub Actions / AWS CodePipeline**
* Add **CloudWatch logging & monitoring**
* Enable **auto-scaling** for ECS tasks

---
