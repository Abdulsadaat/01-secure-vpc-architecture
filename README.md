# 🔐 Secure AWS VPC Architecture

> **AWS Lab Project** | VPC · Subnets · Route Tables · NAT Gateway · Internet Gateway · Security Groups · Network ACLs

---

## 📋 Project Overview

In this project, I designed and deployed a secure, production-grade AWS Virtual Private Cloud (VPC) from scratch — following AWS Well-Architected Framework best practices. The environment isolates public-facing resources from private backend systems using a layered security model.

This is not a click-through tutorial. Every resource was manually configured in a timed challenge lab with no ability to pause or revert — simulating real-world production pressure.

---

## 🏗️ Architecture Diagram

```
Internet
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│                     AWS VPC (10.0.0.0/16)               │
│                                                         │
│  ┌─────────────────────────┐  ┌──────────────────────┐  │
│  │     PUBLIC SUBNET        │  │    PUBLIC SUBNET      │  │
│  │     10.0.1.0/24         │  │    10.0.2.0/24        │  │
│  │     (AZ: us-east-1a)    │  │    (AZ: us-east-1b)   │  │
│  │                         │  │                       │  │
│  │  ┌──────────────────┐   │  │  ┌─────────────────┐  │  │
│  │  │  Web Server (EC2)│   │  │  │ Web Server (EC2)│  │  │
│  │  │  Security Group  │   │  │  │ Security Group  │  │  │
│  │  └──────────────────┘   │  │  └─────────────────┘  │  │
│  └──────────┬──────────────┘  └────────────┬──────────┘  │
│             │  NAT Gateway                 │             │
│  ┌──────────▼──────────────────────────────▼──────────┐  │
│  │                  PRIVATE SUBNET                     │  │
│  │                  10.0.3.0/24                        │  │
│  │                  (App / Database Tier)              │  │
│  │                                                     │  │
│  │    ┌──────────────┐    ┌──────────────────┐         │  │
│  │    │  App Server  │    │   Database Layer  │         │  │
│  │    │  (EC2)       │    │   (RDS / EC2)     │         │  │
│  │    └──────────────┘    └──────────────────┘         │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │              Internet Gateway                    │   │
│  │         (Public subnets route 0.0.0.0/0 here)   │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## 🔧 AWS Services Used

| Service | Purpose |
|---|---|
| **VPC** | Isolated network environment (10.0.0.0/16 CIDR) |
| **Public Subnets** | Host web servers accessible from the internet |
| **Private Subnets** | Host app/database servers with no direct internet exposure |
| **Internet Gateway** | Allows public subnets to communicate with the internet |
| **NAT Gateway** | Allows private subnet instances to reach internet (updates, patches) without inbound exposure |
| **Route Tables** | Public subnet routes 0.0.0.0/0 → IGW; Private subnet routes 0.0.0.0/0 → NAT |
| **Security Groups** | Stateful firewall rules per instance (port 80/443 inbound for web; port 22 restricted) |
| **Network ACLs** | Stateless subnet-level firewall for additional traffic control |

---

## 🚀 What I Built — Step by Step

### Step 1: Create the VPC
- CIDR block: `10.0.0.0/16`
- Enabled DNS hostnames and DNS resolution

### Step 2: Create Subnets
- **Public Subnet A** — `10.0.1.0/24` in AZ us-east-1a
- **Public Subnet B** — `10.0.2.0/24` in AZ us-east-1b
- **Private Subnet** — `10.0.3.0/24` in AZ us-east-1a
- Enabled auto-assign public IP on public subnets

### Step 3: Internet Gateway
- Created and attached IGW to the VPC

### Step 4: NAT Gateway
- Deployed in Public Subnet A
- Allocated an Elastic IP

### Step 5: Route Tables
- **Public Route Table**: `0.0.0.0/0` → Internet Gateway → associated with public subnets
- **Private Route Table**: `0.0.0.0/0` → NAT Gateway → associated with private subnet

### Step 6: Security Groups
- **Web-SG**: Allow port 80 (HTTP) and 443 (HTTPS) from `0.0.0.0/0`; deny all else inbound
- **App-SG**: Allow port 8080 inbound from Web-SG only; no direct internet access
- **SSH-SG**: Allow port 22 from My IP only (or replaced with SSM — no open SSH ports)

### Step 7: Network ACLs
- Public NACL: Allow HTTP/HTTPS inbound; allow ephemeral ports outbound
- Private NACL: Allow traffic from VPC CIDR only; block all other inbound

### Step 8: Launch EC2 Instances
- Web server in public subnet using Web-SG
- App server in private subnet using App-SG
- Verified connectivity: internet → web server ✅ | internet → app server ✗ ✅

---

## 🛡️ Security Highlights

- **Zero direct internet access** to private subnet instances
- **Least-privilege security groups** — only required ports open, scoped to specific sources
- **Network ACLs** provide a second layer of traffic filtering at the subnet level
- **NAT Gateway** enables outbound-only internet access from private instances
- **No open SSH** — access via AWS Systems Manager Session Manager

---

## 📁 Repository Structure

```
01-secure-vpc-architecture/
├── README.md                    # This file
├── architecture/
│   └── vpc-diagram.md           # Architecture diagram reference
├── scripts/
│   ├── create-vpc.sh            # AWS CLI script to create VPC resources
│   ├── create-subnets.sh        # Subnet creation script
│   ├── create-security-groups.sh
│   └── create-nacls.sh
└── docs/
    └── lessons-learned.md       # Key takeaways and gotchas
```

---

## 💡 Key Takeaways

1. **Route tables are the brain of the VPC** — wrong association = broken connectivity
2. **NACLs are stateless** — you must explicitly allow both inbound AND return traffic (ephemeral ports)
3. **Security groups are stateful** — return traffic is automatically allowed
4. **NAT Gateway ≠ Bastion Host** — it enables outbound traffic, not inbound management access
5. **Always tag every resource** — makes cleanup and cost tracking manageable

---

## 🔗 Related Projects

- [02 - Highly Available Multi-Tier Web Application](../02-ha-multitier-webapp)
- [03 - Auto Scaling & Load Balancer](../03-auto-scaling-load-balancer)

---

*Built by Abdul Nazir Sadaat | AWS Cloud Engineer | [LinkedIn](#) | Rancho Cordova, CA*
