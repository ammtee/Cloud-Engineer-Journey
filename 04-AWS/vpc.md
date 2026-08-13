# Amazon VPC (Virtual Private Cloud)

## What It Is

A VPC is a logically isolated virtual network within AWS where you launch resources (EC2, RDS, etc.). It's your own private slice of the AWS network, with full control over IP ranges, subnets, route tables, and gateways — directly applying the networking fundamentals covered earlier in this repository.

## Core Concepts

| Concept | Description |
|---|---|
| **CIDR Block** | The IP address range for the VPC (e.g., `10.0.0.0/16`) |
| **Subnet** | A subdivision of the VPC's CIDR block, tied to a single Availability Zone |
| **Public Subnet** | Has a route to an Internet Gateway — resources can be reachable from the internet |
| **Private Subnet** | No direct route to the internet — used for databases, internal services |
| **Route Table** | Determines where network traffic from a subnet is directed |
| **Internet Gateway (IGW)** | Allows communication between the VPC and the internet |
| **NAT Gateway** | Allows private subnet resources to reach the internet (outbound only) without being publicly reachable |

## A Typical VPC Layout

```
VPC: 10.0.0.0/16
│
├── Public Subnet (10.0.1.0/24) — AZ-a
│     └── Web servers, Load Balancer, NAT Gateway
│
├── Private Subnet (10.0.2.0/24) — AZ-a
│     └── Application servers
│
└── Private Subnet (10.0.3.0/24) — AZ-b (database tier)
      └── RDS instance (Multi-AZ)
```

Spreading subnets across multiple Availability Zones (AZs) is the standard pattern for high availability.

## Public vs. Private Subnet — Routing

| | Public Subnet | Private Subnet |
|---|---|---|
| Route to `0.0.0.0/0` | via Internet Gateway | via NAT Gateway (outbound only) |
| Inbound from internet | Possible (if SG/NACL allow) | Not possible |
| Typical residents | Load balancers, bastion hosts | App servers, databases |

## Security Layers (Defense in Depth)

1. **Security Groups** — instance-level, stateful firewall
2. **Network ACLs** — subnet-level, stateless firewall
3. **Route Tables** — control where traffic is even allowed to go
4. **VPC Flow Logs** — capture network traffic metadata for auditing/troubleshooting

## Basic Commands (AWS CLI)

```bash
# Create a VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Create a subnet
aws ec2 create-subnet \
  --vpc-id vpc-0123456789abcdef0 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a

# Create and attach an Internet Gateway
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway \
  --vpc-id vpc-0123456789abcdef0 \
  --internet-gateway-id igw-0123456789abcdef0

# Create a route table entry pointing to the IGW
aws ec2 create-route \
  --route-table-id rtb-0123456789abcdef0 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-0123456789abcdef0
```

## VPC Peering & Endpoints

- **VPC Peering:** Connects two VPCs privately (no internet routing) — useful for multi-VPC architectures
- **VPC Endpoint:** Lets resources in a private subnet reach AWS services (like S3 or DynamoDB) without going through the internet or a NAT Gateway — both cheaper and more secure

## Best Practices

- Plan your CIDR block size carefully upfront — resizing later is painful (use `/16` for the VPC, `/24` for subnets as a common default)
- Keep databases and sensitive resources in private subnets, never public
- Use a NAT Gateway (not a NAT instance) for production — it's managed and highly available
- Enable VPC Flow Logs for security auditing and troubleshooting connectivity issues
- Use VPC Endpoints for AWS service access from private subnets to avoid unnecessary NAT Gateway data charges

## Interview Prep

**Q: What's the difference between a public and private subnet?**
It comes down to the route table. A public subnet's route table has a route to an Internet Gateway for `0.0.0.0/0`, making it reachable from (and able to reach) the internet directly. A private subnet has no such route — it can only reach the internet indirectly through a NAT Gateway (outbound only), and nothing can initiate a connection to it from outside the VPC.

**Q: Why would you put a database in a private subnet?**
Databases don't need to be directly reachable from the internet, and shouldn't be — keeping them in a private subnet with no route to an Internet Gateway drastically reduces the attack surface. Application servers in the same VPC can still reach the database over the private network.

**Q: What's the difference between a NAT Gateway and an Internet Gateway?**
An Internet Gateway allows two-way traffic between a VPC and the internet — resources with public IPs in a subnet routed to it can be reached from outside. A NAT Gateway allows one-way outbound traffic only — private subnet resources can reach the internet (e.g., for software updates) but can't be initiated against from outside.

**Q: How would you design a VPC for a 3-tier web application?**
Public subnets (across 2+ AZs) for the load balancer, private subnets for application servers reachable only from the load balancer's security group, and separate private subnets for the database tier reachable only from the app tier's security group — each layer only opens the minimum access the layer above it needs.
