# Project: Portfolio Website (AWS Static Hosting)

## Overview

A personal portfolio site — separate from the Cloud Resume Challenge — showcasing projects, background, and MotionCut Productions work, deployed using the same AWS static hosting pattern. Where the Cloud Resume Challenge is deliberately backend-heavy (to demonstrate serverless skills), this project focuses on clean deployment practices and a proper CI/CD pipeline for a real-world static site.

## Architecture

```
GitHub Repo (source)
     │
     │ push to main
     ▼
GitHub Actions (build + deploy)
     │
     ▼
S3 Bucket (private, static assets)
     │
     ▼
CloudFront (HTTPS, custom domain, caching)
     │
     ▼
Route 53 (DNS)
```

## Components & Services Used

| Component | Service | Purpose |
|---|---|---|
| Hosting | S3 (private bucket) | Stores built static site files |
| CDN + HTTPS | CloudFront with Origin Access Control (OAC) | Serves content globally, keeps S3 bucket private |
| DNS | Route 53 | Custom domain routing |
| SSL Certificate | AWS Certificate Manager (ACM) | Free HTTPS certificate for the custom domain |
| CI/CD | GitHub Actions | Automated build and deploy on every push |

## Build Plan (Milestones)

1. **Design & build the site** — Simple HTML/CSS/JS, or a static site generator, showcasing projects (including MotionCut work) and background
2. **S3 bucket (private)** — Create the bucket with Block Public Access enabled — no direct public access
3. **CloudFront with OAC** — Configure CloudFront to be the only entity allowed to read from the bucket, using Origin Access Control
4. **ACM certificate + Route 53** — Request a certificate for the custom domain in ACM (must be in `us-east-1` for CloudFront), attach it to the distribution, and point Route 53 at CloudFront
5. **GitHub Actions pipeline** — On push to `main`: sync files to S3, then invalidate the CloudFront cache so changes go live immediately

## Sample GitHub Actions Workflow

```yaml
name: Deploy Portfolio Site

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-deploy-role
          aws-region: us-east-1

      - name: Sync to S3
        run: aws s3 sync ./site s3://my-portfolio-bucket --delete

      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ABCDEF123456 \
            --paths "/*"
```

Note the use of **OIDC role assumption** (`role-to-assume`) instead of storing long-lived AWS access keys as GitHub secrets — the modern, more secure approach for CI/CD authentication to AWS.

## Why a Private Bucket + OAC Instead of a Public Bucket

A common beginner mistake is making the S3 bucket public for static hosting. The better pattern:
- Keep **Block Public Access enabled** on the bucket (default, secure)
- Use CloudFront's **Origin Access Control** so only CloudFront can read from the bucket
- Everything goes through CloudFront, which also means the site benefits from HTTPS and caching automatically

## Status Tracking

- [ ] Site content built (projects, background, MotionCut work)
- [ ] Private S3 bucket created with Block Public Access on
- [ ] CloudFront distribution with OAC configured
- [ ] ACM certificate issued and validated
- [ ] Route 53 domain pointed at CloudFront
- [ ] GitHub Actions pipeline auto-deploying on push
- [ ] Cache invalidation working correctly after deploys

## Interview Prep

**Q: Why keep the S3 bucket private when CloudFront is serving the site anyway?**
It removes an entire attack surface — if the bucket were public, anyone could bypass CloudFront entirely and hit S3 directly, skipping caching, HTTPS enforcement, and any access logging configured at the CloudFront layer. Origin Access Control ensures CloudFront is the *only* path to the content.

**Q: What's the benefit of using OIDC role assumption in GitHub Actions instead of storing AWS access keys as secrets?**
Long-lived access keys stored as GitHub secrets are a standing credential that could leak or be misused indefinitely. OIDC lets GitHub Actions assume a specific IAM role with a short-lived token issued just for that workflow run — no long-term secret exists at all, which is significantly more secure and aligns with least-privilege principles.

**Q: Why is cache invalidation needed after a deploy?**
CloudFront caches content at edge locations to improve performance, but that means updated files won't be visible to visitors until the cache expires or is explicitly invalidated. Running a cache invalidation as part of the deploy pipeline ensures changes go live immediately instead of waiting for the cache TTL to expire naturally.
