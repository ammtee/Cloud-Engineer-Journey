# AWS Certified Solutions Architect – Associate (SAA-C03)

**Status:** 🔄 In Progress
**Prerequisite recommendation:** Complete CLF-C02 first (see `clf-c02-cloud-practitioner.md`)

## Exam Overview

| | |
|---|---|
| Format | 65 questions, multiple choice / multiple response |
| Duration | 130 minutes |
| Passing score | 720 / 1000 |
| Cost | $150 USD |
| Validity | 3 years |

## Domain Breakdown (Exam Weighting)

| Domain | Weight |
|---|---|
| Design Secure Architectures | 30% |
| Design Resilient Architectures | 26% |
| Design High-Performing Architectures | 24% |
| Design Cost-Optimized Architectures | 20% |

## Study Plan Checklist

### Domain 1: Design Secure Architectures
- [ ] IAM deep dive — policies, roles, cross-account access (see `04-AWS/iam.md`)
- [ ] VPC security — Security Groups vs. NACLs (see `04-AWS/vpc.md`)
- [ ] Encryption — KMS, encryption at rest and in transit
- [ ] Secrets Manager vs. Systems Manager Parameter Store
- [ ] S3 bucket policies, Block Public Access, Object Lock

### Domain 2: Design Resilient Architectures
- [ ] Multi-AZ vs. Multi-Region architectures
- [ ] RDS Multi-AZ and Read Replicas (see `04-AWS/rds.md`)
- [ ] Auto Scaling Groups
- [ ] Elastic Load Balancing (ALB vs. NLB vs. CLB)
- [ ] SQS, SNS for decoupled architectures
- [ ] Backup and disaster recovery strategies (RPO/RTO concepts)

### Domain 3: Design High-Performing Architectures
- [ ] EC2 instance types and right-sizing (see `04-AWS/ec2.md`)
- [ ] Caching strategies — CloudFront, ElastiCache
- [ ] Storage performance — EBS volume types (gp3, io2), S3 performance patterns
- [ ] Database performance — read replicas, DynamoDB partition keys, Aurora
- [ ] Serverless performance patterns — Lambda concurrency, cold starts (see `04-AWS/lambda.md`)

### Domain 4: Design Cost-Optimized Architectures
- [ ] EC2 pricing models — On-Demand, Reserved, Spot, Savings Plans
- [ ] S3 storage classes and lifecycle policies (see `04-AWS/s3.md`)
- [ ] Right-sizing and Auto Scaling for cost efficiency
- [ ] AWS Cost Explorer, Budgets, Trusted Advisor cost checks

## Key Architectural Patterns to Know Cold

**3-Tier Web Application:**
```
ALB (public subnet) → EC2/ECS Auto Scaling Group (private subnet) → RDS Multi-AZ (private subnet)
```

**Serverless API:**
```
API Gateway → Lambda → DynamoDB
```

**Static Website with Global Delivery:**
```
Route 53 → CloudFront → S3 (private, via OAC)
```

**Decoupled Processing:**
```
Producer → SQS Queue → Consumer (EC2/Lambda) — absorbs traffic spikes, prevents overload
```

## High-Yield Comparison Tables (Common Exam Traps)

| ALB | NLB |
|---|---|
| Layer 7 (HTTP/HTTPS) | Layer 4 (TCP/UDP) |
| Content-based routing | Extreme performance, static IP support |

| SQS | SNS |
|---|---|
| Queue — pull-based, one consumer processes each message | Pub/sub — push-based, many subscribers receive each message |

| Standard SQS | FIFO SQS |
|---|---|
| At-least-once delivery, best-effort ordering | Exactly-once processing, strict ordering |

## Practice Resources

- AWS Skill Builder official practice exam
- Tutorials Dojo practice tests (widely recommended in the community)
- Hands-on labs — actually building the 3-tier and serverless patterns above in the AWS Console/CLI reinforces retention far more than reading alone

## Notes / Weak Areas

*(Update this section as study progresses.)*

-
-
-
