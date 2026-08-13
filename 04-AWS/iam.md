# AWS IAM (Identity and Access Management)

## What It Is

IAM controls **who** can do **what** in your AWS account. It manages authentication (verifying identity) and authorization (what actions that identity can perform) across every AWS service.

## Core Concepts

| Concept | Description |
|---|---|
| **User** | Represents a person or application with long-term credentials |
| **Group** | A collection of users — permissions applied to the group apply to all members |
| **Role** | A temporary identity assumed by users, services, or applications (no long-term credentials) |
| **Policy** | A JSON document defining permissions (allow/deny actions on resources) |
| **Root User** | The account owner with unrestricted access — should almost never be used day-to-day |

## Policy Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "IpAddress": { "aws:SourceIp": "203.0.113.0/24" }
      }
    }
  ]
}
```

- **Effect:** `Allow` or `Deny`
- **Action:** The API operation(s) covered (e.g., `s3:GetObject`, `ec2:StartInstances`)
- **Resource:** The specific ARN(s) the policy applies to
- **Condition:** (Optional) Extra constraints — IP range, MFA required, time window, etc.

## Users vs. Roles

| | IAM User | IAM Role |
|---|---|---|
| Credentials | Long-term (access key + secret) | Temporary (STS-issued, auto-expiring) |
| Use case | Human operators, occasionally CI/CD | EC2 instances, Lambda functions, cross-account access |
| Best practice | Use sparingly, enforce MFA | Preferred for anything running inside AWS |

**Rule of thumb:** if a workload runs *inside* AWS (EC2, Lambda, ECS), it should assume a **role** — never store access keys on the resource itself.

## Policy Types

- **Identity-based policies:** Attached to users, groups, or roles
- **Resource-based policies:** Attached directly to a resource (e.g., an S3 bucket policy)
- **Managed policies:** Reusable, standalone (AWS-managed or customer-managed)
- **Inline policies:** Embedded directly into a single user/group/role (not reusable)

## The Principle of Least Privilege

Grant only the permissions required to perform a task — nothing more. In practice:
- Start with zero access, add permissions incrementally as needed
- Use specific resource ARNs instead of `"Resource": "*"` where possible
- Avoid `"Action": "*"` (full admin) except for genuinely privileged break-glass roles
- Regularly review IAM Access Analyzer / unused permissions reports

## Basic Commands (AWS CLI)

```bash
# Create a user
aws iam create-user --user-name jane-doe

# Attach a managed policy
aws iam attach-user-policy \
  --user-name jane-doe \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# Create a role for EC2
aws iam create-role \
  --role-name ec2-s3-read-role \
  --assume-role-policy-document file://trust-policy.json

# List a user's attached policies
aws iam list-attached-user-policies --user-name jane-doe
```

## MFA (Multi-Factor Authentication)

Always enable MFA on the **root user** and any privileged IAM users. AWS supports virtual MFA apps (Google Authenticator, Authy), hardware tokens, and security keys (FIDO2/WebAuthn).

## Best Practices

- Never use the root account for daily operations — create an admin IAM user/role instead
- Enable MFA on all human users, especially anyone with elevated permissions
- Use IAM roles for EC2/Lambda/ECS instead of embedding access keys
- Rotate access keys regularly and delete unused ones
- Use groups to manage permissions at scale instead of attaching policies per-user
- Enable AWS CloudTrail to audit who did what, when

## Interview Prep

**Q: What's the difference between an IAM user and an IAM role?**
A user has long-term credentials (access key/secret) tied to a specific person or application. A role has no long-term credentials — it's assumed temporarily and issues short-lived credentials via AWS STS. Roles are the recommended way to grant permissions to AWS resources like EC2 instances or Lambda functions.

**Q: What is the principle of least privilege, and why does it matter?**
It means granting only the minimum permissions needed to perform a specific task. It matters because it limits the blast radius if credentials are compromised — an attacker with a narrowly-scoped role can do far less damage than one with broad or admin access.

**Q: How would you grant an EC2 instance access to an S3 bucket without hardcoding credentials?**
Create an IAM role with a policy granting the needed S3 permissions (e.g., `s3:GetObject` on a specific bucket ARN), then attach that role to the EC2 instance via an instance profile. The instance automatically receives temporary credentials through the metadata service — no keys stored on disk.

**Q: What's the difference between identity-based and resource-based policies?**
Identity-based policies attach to a user, group, or role and define what that identity can do. Resource-based policies attach directly to a resource (like an S3 bucket or SQS queue) and define who can access that resource — useful for cross-account access without needing to modify the requester's own permissions.
