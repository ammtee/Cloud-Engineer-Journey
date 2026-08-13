# Technical Interview Prep — Consolidated

This file pulls together the interview questions scattered across each topic doc in this repository into a single study reference, organized by category. Each topic doc also has its own "Interview Prep" section with full context — this is meant for quick review and mock-interview practice.

## Networking

- What's the difference between TCP and UDP?
- Walk me through what happens when you type a URL into a browser.
- Why does private IP addressing matter in AWS?

*(Full answers: `01-Networking/networking-fundamentals.md`)*

## Linux

- What does `chmod 755` mean?
- What's the difference between a hard link and a symbolic link?
- How would you find out what's consuming disk space on a server?
- Why is `chmod 777` considered bad practice?
- Why is key-based SSH authentication preferred over passwords?
- How would you troubleshoot a service that isn't responding on its expected port?
- What's the difference between `systemctl start` and `systemctl enable`?
- What's the difference between `apt update` and `apt upgrade`?

*(Full answers: `02-Linux/filesystem-and-permissions.md`, `02-Linux/networking-and-services.md`)*

## Git & GitHub

- What's the difference between `git fetch` and `git pull`?
- What's the difference between `git reset` and `git revert`?
- Why would you use `.gitignore`?
- What's the difference between merging and rebasing?
- How do you resolve a merge conflict?
- What's a Pull Request, and why use one instead of pushing directly to `main`?
- What's the difference between `git branch -d` and `git branch -D`?

*(Full answers: `03-Git/git-basics.md`, `03-Git/branching-and-workflow.md`)*

## AWS — IAM

- What's the difference between an IAM user and an IAM role?
- What is the principle of least privilege, and why does it matter?
- How would you grant an EC2 instance access to an S3 bucket without hardcoding credentials?
- What's the difference between identity-based and resource-based policies?

## AWS — EC2

- What's the difference between stopping and terminating an instance?
- When would you use Spot Instances vs. On-Demand?
- How do Security Groups differ from Network ACLs?
- How would you secure SSH access to an EC2 instance in production?

## AWS — VPC

- What's the difference between a public and private subnet?
- Why would you put a database in a private subnet?
- What's the difference between a NAT Gateway and an Internet Gateway?
- How would you design a VPC for a 3-tier web application?

## AWS — S3

- What's the difference between S3 and EBS?
- How would you make an S3 bucket serve a static website securely?
- What are S3 storage classes, and why do they matter for cost?
- What does S3 versioning protect against, and what doesn't it protect against?

## AWS — RDS

- What's the difference between Multi-AZ and a Read Replica?
- How would you recover a database to a specific point in time before a bad deployment?
- Why should an RDS instance be in a private subnet?
- What's the tradeoff of enabling Multi-AZ?

## AWS — Lambda

- What is a cold start, and how would you reduce its impact?
- How is Lambda billed, and how does that affect design decisions?
- Walk me through a serverless API architecture using Lambda.
- When would you *not* use Lambda?

*(Full answers: `04-AWS/iam.md`, `ec2.md`, `vpc.md`, `s3.md`, `rds.md`, `lambda.md`)*

## Docker

- What's the difference between a Docker image and a container?
- Why are containers faster to start than virtual machines?
- How would you persist data used by a container beyond its lifecycle?
- Why avoid running processes as root inside a container?
- Why does instruction order matter in a Dockerfile?
- What problem does a multi-stage build solve?
- How do containers in the same Docker Compose file communicate with each other?
- What's the difference between `CMD` and `ENTRYPOINT`?

*(Full answers: `05-Docker/docker-basics.md`, `05-Docker/dockerfile-and-compose.md`)*

## Projects

- Walk me through the architecture of your Cloud Resume Challenge.
- Why use CloudFront in front of S3 instead of serving directly from the S3 website endpoint?
- Why DynamoDB instead of RDS for the visitor counter?
- Why keep the S3 bucket private when CloudFront is serving the site anyway?
- What's the benefit of using OIDC role assumption in GitHub Actions instead of storing AWS access keys as secrets?
- How does deploying to ECS Fargate differ from running Docker on an EC2 instance?

*(Full answers: `06-Projects/cloud-resume-challenge.md`, `portfolio-website.md`, `docker-deployment-lab.md`)*

## How to Use This File

1. Cover the answer sections in each topic doc, try answering out loud first
2. Time yourself — aim to answer each in under 90 seconds, the way a real interview moves
3. For scenario-style questions ("how would you design X"), practice sketching the architecture on a whiteboard or paper, not just describing it verbally
4. Revisit this list a few days before any interview as a fast full-repo refresher
