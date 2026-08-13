# Amazon EC2 (Elastic Compute Cloud)

## What It Is

EC2 provides resizable virtual servers ("instances") in the cloud. Instead of buying physical hardware, you rent compute capacity by the second/hour, choosing the CPU, memory, storage, and networking that fit your workload.

## Core Concepts

| Concept | Description |
|---|---|
| **AMI (Amazon Machine Image)** | A template containing the OS and pre-installed software used to launch an instance |
| **Instance Type** | Defines hardware specs (CPU, RAM, network) — e.g., `t3.micro`, `m5.large` |
| **Instance Family** | Groups optimized for a purpose — `t` (burstable/general), `m` (balanced), `c` (compute-optimized), `r` (memory-optimized) |
| **Key Pair** | Public/private key used for SSH authentication instead of a password |
| **Security Group** | A virtual firewall controlling inbound/outbound traffic to the instance |
| **EBS (Elastic Block Store)** | Persistent block storage volume attached to an instance |
| **Elastic IP** | A static public IP you can attach/detach from instances |

## Instance Lifecycle

```
Pending → Running → Stopping → Stopped → Terminated
```

- **Stopped:** Instance is off, EBS data persists, you're not billed for compute (but still billed for storage)
- **Terminated:** Instance is permanently deleted; root EBS volume is deleted by default unless configured otherwise

## Pricing Models

| Model | Best For | Notes |
|---|---|---|
| **On-Demand** | Unpredictable, short-term workloads | Pay by the second, no commitment |
| **Reserved Instances** | Steady-state, predictable workloads | 1 or 3-year commitment, up to ~72% discount |
| **Spot Instances** | Fault-tolerant, flexible workloads | Up to 90% discount, can be reclaimed by AWS with 2-min notice |
| **Savings Plans** | Flexible long-term commitment | Commit to $/hour spend, applies across instance families |

## Basic Commands (AWS CLI)

```bash
# List running instances
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"

# Launch an instance
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.micro \
  --key-name my-key \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0

# Stop / Start / Terminate
aws ec2 stop-instances --instance-ids i-0123456789abcdef0
aws ec2 start-instances --instance-ids i-0123456789abcdef0
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0

# SSH into an instance
ssh -i my-key.pem ec2-user@<public-ip>
```

## Security Group vs. Network ACL

| | Security Group | Network ACL |
|---|---|---|
| Level | Instance level | Subnet level |
| Rules | Allow only (stateful) | Allow + Deny (stateless) |
| Evaluation | All rules evaluated | Rules processed in order |
| State | Stateful — return traffic auto-allowed | Stateless — must explicitly allow return traffic |

## Best Practices

- Never hardcode credentials on an instance — use an **IAM role** instead
- Use **Security Groups** as the primary firewall; keep NACLs for broad subnet-level rules
- Right-size instances — start small (`t3.micro`/`t3.small`) and scale based on actual CloudWatch metrics
- Use **Auto Scaling Groups** for production workloads instead of manually managing individual instances
- Tag every instance (`Name`, `Environment`, `Owner`) for cost tracking and organization
- Enable **termination protection** on production instances to prevent accidental deletion

## Interview Prep

**Q: What's the difference between stopping and terminating an instance?**
Stopping shuts the instance down but preserves the EBS volume and instance configuration — you can start it again later and it keeps its instance ID. Terminating permanently deletes the instance; by default the root EBS volume is deleted too, and the instance ID can never be reused.

**Q: When would you use Spot Instances vs. On-Demand?**
Spot Instances fit workloads that can tolerate interruption — batch processing, CI/CD runners, non-critical background jobs — because AWS can reclaim them with only a 2-minute warning in exchange for up to 90% cost savings. On-Demand suits workloads needing guaranteed availability, like production web servers with unpredictable traffic.

**Q: How do Security Groups differ from Network ACLs?**
Security Groups operate at the instance level and are stateful — if you allow inbound traffic, the response is automatically allowed out. Network ACLs operate at the subnet level, are stateless (you must explicitly allow both directions), and support explicit deny rules, which Security Groups don't.

**Q: How would you secure SSH access to an EC2 instance in production?**
Restrict the Security Group's inbound SSH rule to a specific IP range (not `0.0.0.0/0`), use key-pair authentication instead of passwords, consider AWS Systems Manager Session Manager to avoid opening port 22 entirely, and disable root login.
