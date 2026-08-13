# Amazon RDS (Relational Database Service)

## What It Is

RDS is a managed relational database service — AWS handles provisioning, patching, backups, and failover, so you focus on schema design and queries instead of database administration. Directly relevant given hands-on MySQL experience from prior infrastructure work.

## Supported Engines

- MySQL
- PostgreSQL
- MariaDB
- Oracle
- SQL Server
- Amazon Aurora (AWS's own MySQL/PostgreSQL-compatible engine, built for higher performance)

## Core Concepts

| Concept | Description |
|---|---|
| **DB Instance** | The actual running database server |
| **Multi-AZ Deployment** | A synchronous standby replica in a different AZ for automatic failover |
| **Read Replica** | An asynchronous, read-only copy used to scale read traffic (not for HA/failover) |
| **Parameter Group** | Database engine configuration settings |
| **Subnet Group** | The set of subnets (typically private) RDS can place the instance in |
| **Automated Backups** | Daily snapshots + transaction logs, enabling point-in-time recovery |

## Multi-AZ vs. Read Replicas

| | Multi-AZ | Read Replica |
|---|---|---|
| Purpose | High availability / failover | Read scalability |
| Replication | Synchronous | Asynchronous |
| Can serve reads? | No (standby is not accessible) | Yes |
| Automatic failover | Yes | No (must be manually promoted) |

These solve different problems and are often used together: Multi-AZ for resilience, Read Replicas for offloading read-heavy traffic.

## Basic Commands (AWS CLI)

```bash
# Create a MySQL RDS instance
aws rds create-db-instance \
  --db-instance-identifier my-app-db \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --master-username admin \
  --master-user-password 'ChangeMe123!' \
  --allocated-storage 20 \
  --vpc-security-group-ids sg-0123456789abcdef0 \
  --db-subnet-group-name my-private-subnet-group \
  --multi-az

# Create a read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier my-app-db-replica \
  --source-db-instance-identifier my-app-db

# Take a manual snapshot
aws rds create-db-snapshot \
  --db-instance-identifier my-app-db \
  --db-snapshot-identifier my-app-db-snapshot-1
```

## Security

- RDS instances should live in **private subnets**, never publicly accessible in production
- Use Security Groups to restrict inbound DB port access to only the application tier
- Enable **encryption at rest** (via KMS) at creation time — it can't be enabled retroactively on an existing unencrypted instance
- Use **IAM database authentication** where supported, instead of long-lived DB passwords
- Store credentials in **AWS Secrets Manager**, not in application code or environment files

## Backup & Recovery

- **Automated backups:** enabled by default, retained 1–35 days, support point-in-time recovery
- **Manual snapshots:** kept until explicitly deleted, useful before major changes (schema migrations, version upgrades)
- **Point-in-time recovery:** restore to any specific second within the retention window — creates a *new* DB instance, doesn't overwrite the existing one

## Best Practices

- Enable Multi-AZ for any production database
- Place RDS in private subnets with tightly scoped Security Groups
- Enable automated backups and test restores periodically — an untested backup isn't a real backup
- Use Read Replicas to offload reporting/analytics queries away from the primary
- Monitor with CloudWatch (CPU, storage, connections, replica lag) and set alarms proactively
- Right-size instance class based on actual load, not guesswork

## Interview Prep

**Q: What's the difference between Multi-AZ and a Read Replica?**
Multi-AZ maintains a synchronous standby in a different Availability Zone purely for failover — it's not accessible for reads and exists only for high availability. A Read Replica is an asynchronous copy that *can* serve read traffic, used to scale read-heavy workloads, but doesn't provide automatic failover on its own.

**Q: How would you recover a database to a specific point in time before a bad deployment?**
Use RDS's point-in-time recovery, which replays transaction logs up to the exact second you specify, within the backup retention window. This creates a new DB instance at that state — you'd then redirect the application to the new instance rather than overwriting the original.

**Q: Why should an RDS instance be in a private subnet?**
A database doesn't need to be reachable directly from the internet — application servers within the same VPC can reach it over the private network. Keeping it in a private subnet with no public accessibility and a tightly scoped Security Group significantly reduces the attack surface.

**Q: What's the tradeoff of enabling Multi-AZ?**
It roughly doubles the cost of the database instance (since you're paying for the standby too) in exchange for automatic failover and higher availability — a worthwhile tradeoff for production workloads, but often unnecessary for dev/test environments.
