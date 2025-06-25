# 🚦 MLOps Computer Vision Pipeline on AWS

## Overview

This project implements a production-grade MLOps pipeline for computer vision, deploying a React frontend and two Flask-based ML microservices (YOLO object detection and DepthAnything depth estimation) on AWS using:

* Docker for containerization
* AWS ECR for private image registry
* AWS CodeBuild for CI/CD
* AWS ECS (EC2 launch type) for orchestration
* AWS Application Load Balancer (ALB) for HTTP routing
* Terraform for infrastructure as code

## Architecture

```
User
 │
 │  HTTP(S)
 ▼
[Application Load Balancer]
 ├── /detect*        → YOLO backend (port 5005)
 ├── /predict_depth* → Depth backend (port 5050)
 └── /*              → React frontend (port 80)
```

All services run as ECS tasks on an EC2 cluster (m5.large instances, auto-scaled).

## Prerequisites

* AWS account with administrator permissions
* AWS CLI configured (`aws configure`)
* Terraform ≥ 1.0
* Docker (for local builds/tests)
* Git

## Repository Structure

```
.
├── object-detection-react-app/      # React frontend
├── yolo-v5-flask-app/              # YOLO backend (Flask)
├── depth-anything-flask-app/       # Depth backend (Flask)
├── buildspec.yml                   # CodeBuild pipeline config
├── main.tf, variables.tf, outputs.tf # Terraform IaC
└── README.md                       # This guide
```

## Step-by-Step Deployment Guide

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/your-repo.git
cd your-repo
```

### 2. Configure AWS Credentials

```bash
aws configure
# Enter AWS Access Key, Secret Key, region (e.g., us-east-1), output format
```

### 3. Create ECR Repositories

```bash
aws ecr create-repository --repository-name mlops/frontend --region us-east-1
aws ecr create-repository --repository-name mlops/yolo-backend --region us-east-1
aws ecr create-repository --repository-name mlops/depth-backend --region us-east-1
```

### 4. Deploy Infrastructure with Terraform

```bash
terraform init
terraform apply
# Type 'yes' to confirm
```

This will provision:

* VPC, subnets, security groups
* ECS cluster (EC2, m5.large instances)
* Application Load Balancer (ALB)
* Target groups and listener rules for each service
* ECS task definitions and services for frontend, yolo, and depth
* CloudWatch log groups

> **Note:** Outputs include your ALB DNS name (e.g., `mlops-alb-xxxx.elb.amazonaws.com`).

### 5. Set Up CodeBuild for CI/CD

1. Open the AWS CodeBuild console.
2. Create a new project:

   * **Source:** Connect to your GitHub repo.
   * **Environment:** `aws/codebuild/standard:7.0` (Privileged mode Enabled).
   * **Service Role:** New or existing with ECR and ECS permissions.
   * **Environment variables:** As defined in `buildspec.yml`.
3. Use the provided `buildspec.yml` for build instructions.


### 6. Verify ECS and ALB

* **ECS console:** Confirm all three services are running and healthy.
* **EC2 console (Target Groups):** Ensure at least one healthy target per group.
* **ALB console:** Confirm your DNS name is active.

## Application URLs

* **Frontend:** `http://<ALB_DNS>`
* **YOLO API:** `http://<ALB_DNS>/detect`
* **Depth API:** `http://<ALB_DNS>/predict_depth`

## Health Checks and Troubleshooting

* **Health Checks:** ALB expects HTTP 200 or 405 at `/` for all services.
* **Logs:** All container logs are sent to CloudWatch (`/ecs/frontend`, `/ecs/yolo`, `/ecs/depth`).

**Common issues:**

* **502/503 errors:** Check target group health, container ports, and logs.
* **CannotPullContainerError:** Verify ECR image tags and IAM permissions.

## How Load Balancing Works

* ALB inspects each incoming HTTP request.
* **Path-based routing:**

  * `/detect*` → YOLO backend
  * `/predict_depth*` → Depth backend
  * `/*` → React frontend
* Target groups ensure traffic is routed only to healthy ECS tasks.

## Auto Scaling

* **EC2 Auto Scaling Group:** Cluster scales from 1 to 2 `m5.large` instances.
* **ECS Services:** Each service runs 1 task by default (can be scaled manually or via ECS service autoscaling).

## Cleanup

To avoid AWS charges, destroy all resources:

```bash
terraform destroy
# Type 'yes' to confirm
```

Also delete ECR repositories and CodeBuild projects if no longer needed.

## Credits

* Project for CSYE 7220 DevOps, Prof. Dino Konstantopoulos
* Based on `object-detection-react-app`, `yolo-v5-flask-app`, `depth-anything-flask-app`


