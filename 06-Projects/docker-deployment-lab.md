# Project: Docker Deployment Lab

## Overview

A multi-container application deployed using Docker Compose locally, then containerized for deployment on AWS — demonstrating the path from local development to cloud-hosted containers. This project ties together the Docker documentation and AWS documentation in this repository into one applied piece of work.

## Architecture

```
Local Development                    AWS Deployment
──────────────────                   ──────────────

┌─────────────┐                     ┌──────────────────┐
│   Web App   │                     │   ECS Fargate     │
│  (Node/     │                     │   (or EC2 +        │
│   Flask)    │                     │    Docker)          │
└──────┬──────┘                     └────────┬───────────┘
       │                                       │
┌──────▼──────┐                     ┌─────────▼──────────┐
│   MySQL     │                     │   RDS (MySQL)        │
│  (container) │                     │   (managed)           │
└─────────────┘                     └────────────────────┘

docker-compose.yml                  Task Definition + Service
```

## Components & Services Used

| Component | Local | AWS Equivalent |
|---|---|---|
| App container | Docker (built from Dockerfile) | Same image pushed to Amazon ECR |
| Database | MySQL container | RDS MySQL (managed, not containerized) |
| Orchestration | Docker Compose | ECS Fargate (serverless containers) |
| Image registry | Local Docker daemon | Amazon ECR (Elastic Container Registry) |
| Networking | Compose's default bridge network | VPC with public/private subnets |

## Build Plan (Milestones)

1. **Local app + Dockerfile** — Build a simple web app (e.g., a Flask or Node API) with a Dockerfile
2. **Docker Compose for local dev** — Define the app + MySQL database as services in `docker-compose.yml`, confirm it runs end-to-end locally
3. **Push image to Amazon ECR** — Create an ECR repository, authenticate, tag, and push the built image
4. **Provision AWS infrastructure** — VPC with public/private subnets, RDS MySQL instance in a private subnet, ECS cluster
5. **ECS Task Definition** — Define the container, port mappings, environment variables (DB connection info via Secrets Manager, not hardcoded), and resource limits
6. **ECS Service** — Run the task behind an Application Load Balancer, so the app is reachable via a stable URL
7. **CI/CD** — GitHub Actions workflow: build image → push to ECR → update ECS service with the new image

## Pushing to Amazon ECR

```bash
# Authenticate Docker to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

# Build and tag
docker build -t my-app .
docker tag my-app:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest

# Push
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
```

## Why ECS Fargate (Not EC2 + Docker)

| | EC2 + Docker | ECS Fargate |
|---|---|---|
| Server management | You manage the EC2 instances | No servers to manage |
| Scaling | Manual or Auto Scaling Group setup | Built into ECS service configuration |
| Billing | Pay for the instance regardless of usage | Pay per task, based on CPU/memory allocated |

Fargate is the more "cloud-native" choice for this project — it directly builds on the EC2/VPC/IAM concepts already documented, while removing server management entirely, similar to how Lambda removes it for functions.

## Networking & Security Design

- ECS tasks run in **private subnets**, reachable only through an Application Load Balancer in public subnets
- RDS MySQL also lives in a **private subnet**, reachable only from the ECS task's security group
- Database credentials pulled from **Secrets Manager** at container startup, never baked into the image or task definition in plaintext
- ECS task execution role follows least privilege — only permissions needed to pull the image from ECR and read the specific secret

## Status Tracking

- [ ] App + Dockerfile built and tested locally
- [ ] docker-compose.yml running app + MySQL locally
- [ ] Image pushed to Amazon ECR
- [ ] VPC, subnets, RDS instance provisioned
- [ ] ECS cluster, task definition, and service running
- [ ] Application Load Balancer routing traffic to ECS tasks
- [ ] Secrets Manager used for DB credentials (not hardcoded)
- [ ] CI/CD pipeline building and deploying automatically

## Interview Prep

**Q: How does deploying to ECS Fargate differ from running Docker on an EC2 instance?**
With EC2, you provision and manage the underlying servers yourself — patching, scaling, capacity planning. With Fargate, AWS manages the underlying compute entirely; you just define the task (CPU/memory, image, networking) and AWS runs it, scaling and billing per task rather than per server. It's the container equivalent of the shift from EC2 to Lambda for serverless functions.

**Q: How would you securely pass a database password to a containerized application on ECS?**
Store it in AWS Secrets Manager (or Parameter Store), then reference it in the ECS task definition's `secrets` field rather than the `environment` field — ECS injects it securely at container startup without it ever being visible in the task definition itself or baked into the image.

**Q: Why containerize the app but not the database?**
Databases hold persistent state that doesn't fit well with the ephemeral, easily-replaced nature of containers — losing a container shouldn't risk losing data. RDS provides managed backups, Multi-AZ failover, and patching for the database tier, while the stateless application tier benefits fully from being containerized and easily scaled or replaced.
