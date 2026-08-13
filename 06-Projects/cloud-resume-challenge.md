# Project: Cloud Resume Challenge

## Overview

The Cloud Resume Challenge is a widely recognized hands-on project in the cloud community, designed specifically to demonstrate practical AWS skills to recruiters. It takes a simple idea — host a resume online — and builds a full serverless, multi-service architecture around it, touching nearly every core AWS service covered in this repository.

**Why it matters for a Junior Cloud Engineer application:** it's not a tutorial screenshot — it's a live, working project a recruiter can visit, backed by a public GitHub repo showing real infrastructure code and CI/CD, which speaks louder than a certification alone.

## Architecture

```
                    ┌─────────────┐
   Visitor  ──────▶ │  CloudFront │  (HTTPS, caching, custom domain)
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  S3 Bucket  │  (static HTML/CSS/JS resume)
                    └──────┬──────┘
                           │ (JS fetch on page load)
                    ┌──────▼──────┐
                    │ API Gateway │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │   Lambda    │  (visitor counter logic)
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  DynamoDB   │  (stores visitor count)
                    └─────────────┘

   Route 53 → DNS for custom domain, points to CloudFront
```

## Components & Services Used

| Component | AWS Service | Purpose |
|---|---|---|
| Frontend hosting | S3 (static website) | Serves the actual resume HTML/CSS/JS |
| CDN + HTTPS | CloudFront | Global caching, HTTPS termination, custom domain support |
| DNS | Route 53 | Maps a custom domain to CloudFront |
| Visitor counter API | API Gateway + Lambda | Serverless backend logic |
| Data storage | DynamoDB | Stores and increments the visitor count |
| Infrastructure as Code | (Optional) Terraform / AWS SAM / CloudFormation | Reproducible, version-controlled infrastructure |
| CI/CD | GitHub Actions | Auto-deploy frontend/backend on every push to `main` |

## Build Plan (Milestones)

1. **Static resume site** — Write the resume as plain HTML/CSS, host it on S3 with static website hosting enabled
2. **HTTPS + custom domain** — Put CloudFront in front of the S3 bucket, request a certificate via ACM, point a Route 53 domain at it
3. **Visitor counter backend** — Build a Lambda function that reads/increments a count in DynamoDB, expose it via API Gateway
4. **Frontend integration** — Add JavaScript to the resume page that calls the API on load and displays the count
5. **Infrastructure as Code** — Define the S3 bucket, CloudFront distribution, Lambda, API Gateway, and DynamoDB table in Terraform or AWS SAM instead of clicking through the console
6. **CI/CD pipeline** — GitHub Actions workflow that runs tests, deploys frontend changes to S3, and deploys backend changes via SAM/Terraform automatically on push
7. **Testing** — Unit tests for the Lambda function (e.g., using `pytest` for Python)

## Sample Lambda: Visitor Counter

```python
import boto3
import json

dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table("visitor-count")

def lambda_handler(event, context):
    response = table.update_item(
        Key={"id": "resume-visits"},
        UpdateExpression="SET visits = if_not_exists(visits, :start) + :inc",
        ExpressionAttributeValues={":inc": 1, ":start": 0},
        ReturnValues="UPDATED_NEW"
    )
    return {
        "statusCode": 200,
        "headers": {"Access-Control-Allow-Origin": "*"},
        "body": json.dumps({"visits": int(response["Attributes"]["visits"])})
    }
```

## What This Project Demonstrates to Recruiters

- Practical, applied knowledge of S3, CloudFront, Route 53, API Gateway, Lambda, and DynamoDB — not just exam theory
- Understanding of serverless architecture patterns
- Infrastructure as Code discipline (if the Terraform/SAM step is completed)
- CI/CD literacy — an increasingly expected skill even for junior cloud roles
- A live, linkable project on a resume/LinkedIn/GitHub profile

## Status Tracking

- [ ] Static site live on S3
- [ ] CloudFront + HTTPS + custom domain configured
- [ ] Lambda + API Gateway + DynamoDB visitor counter working
- [ ] Frontend calling the API and displaying live count
- [ ] Infrastructure defined as code (Terraform/SAM)
- [ ] CI/CD pipeline deploying automatically on push
- [ ] Public repo README documenting the architecture (link back to this file)

## Interview Prep

**Q: Walk me through the architecture of your Cloud Resume Challenge.**
A static resume site is hosted on S3 and served globally through CloudFront for HTTPS and caching, with Route 53 handling the custom domain. A visitor counter is implemented serverlessly: the frontend calls an API Gateway endpoint on page load, which triggers a Lambda function that increments and reads a count from DynamoDB, returning it to be displayed on the page.

**Q: Why use CloudFront in front of S3 instead of serving directly from the S3 website endpoint?**
CloudFront adds HTTPS (S3 static website hosting alone doesn't support HTTPS on custom domains), caches content at edge locations for faster global load times, and lets you restrict the S3 bucket from being publicly accessible directly — improving both performance and security.

**Q: Why DynamoDB instead of RDS for the visitor counter?**
The workload is a simple key-value increment operation with no complex relational queries — DynamoDB's low-latency, serverless, pay-per-request model fits that access pattern far better than provisioning and managing a relational database for a single counter value.
