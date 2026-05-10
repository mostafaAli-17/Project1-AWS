# 🌐 Designing & Deploying a Custom VPC for a Multi-Tier Web Application on AWS

> A production-ready, multi-tier web application infrastructure built on AWS using a custom Virtual Private Cloud (VPC) with full network isolation, high availability, and scalability.
---

## 🚀 Project Overview

This project demonstrates the design and deployment of a **custom AWS VPC** to host a secure, scalable, and highly available multi-tier web application. The architecture follows AWS best practices by separating the infrastructure into distinct tiers:

| Tier | Description |
|------|-------------|
| **Web Tier** | Public-facing EC2 instances behind an Application Load Balancer |
| **App Tier** | Private EC2 instances handling business logic |
| **Data/Storage Tier** | S3 for static assets and object storage |

---

## ☁️ AWS Services Used

| Service | Purpose |
|---------|---------|
| **VPC** | Custom isolated network with full control over IP ranges |
| **Public & Private Subnets** | Network segmentation across multiple Availability Zones |
| **EC2 Instances** | Web and application servers |
| **Application Load Balancer (ALB)** | Distributes traffic across EC2 instances |
| **NAT Gateway** | Allows private subnet instances to access the internet securely |
| **Auto Scaling Group (ASG)** | Automatically scales EC2 instances based on demand |
| **Security Groups** | Instance-level firewall rules |
| **Network ACLs (NACLs)** | Subnet-level stateless traffic filtering |
| **Amazon S3** | Object storage for static files and assets |

---

## 🌍 Network Design

### VPC Configuration
- **CIDR Block:** `10.0.0.0/16`
- **Region:** `us-east-1` (or your chosen region)
- **Availability Zones:** 2 AZs for high availability

### Subnet Layout

| Subnet | Type | CIDR | AZ | Purpose |
|--------|------|------|----|---------|
| Public Subnet 1 | Public | `10.0.1.0/24` | AZ-1a | ALB, NAT Gateway |
| Public Subnet 2 | Public | `10.0.2.0/24` | AZ-1b | ALB, NAT Gateway |
| Private Subnet 1 | Private | `10.0.3.0/24` | AZ-1a | EC2 App Servers |
| Private Subnet 2 | Private | `10.0.4.0/24` | AZ-1b | EC2 App Servers |

### Routing
- **Public Subnets** → Internet Gateway (IGW) for direct internet access
- **Private Subnets** → NAT Gateway for outbound internet (no inbound access)

---

## 🔐 Security Design

### Security Groups

| Security Group | Inbound Rules | Outbound Rules |
|----------------|---------------|----------------|
| **ALB-SG** | HTTP (80), HTTPS (443) from `0.0.0.0/0` | All traffic to Web-SG |
| **Web/App-SG** | HTTP (80) from ALB-SG only | All traffic outbound |
| **Bastion-SG** | SSH (22) from trusted IP only | All traffic outbound |

### Network ACLs (NACLs)
- Public subnets: Allow HTTP/HTTPS inbound, deny all other unsolicited traffic
- Private subnets: Allow traffic only from within VPC CIDR, block direct internet access

### Key Security Principles Applied
- ✅ Principle of Least Privilege on all Security Groups
- ✅ No direct SSH access to private instances (Bastion Host / SSM)
- ✅ Private subnets have no route to IGW
- ✅ NACLs as an additional stateless layer of defense

---

## 📈 High Availability & Scalability

### Application Load Balancer (ALB)
- Deployed across **2 public subnets** in different AZs
- Performs **health checks** on EC2 targets
- Routes traffic only to healthy instances

### Auto Scaling Group (ASG)
- **Minimum capacity:** 2 instances
- **Desired capacity:** 2 instances
- **Maximum capacity:** 4 instances
- Scales based on **CPU utilization** (e.g., scale out at 70%)
- Instances distributed across multiple AZs for fault tolerance

### NAT Gateway
- Deployed in **each public subnet** per AZ for high availability
- Enables private EC2 instances to pull updates/packages from the internet

---

## 🛠️ Getting Started

### Prerequisites
- AWS Account with appropriate IAM permissions
- AWS CLI installed and configured
- Basic knowledge of AWS networking concepts

### Deployment Steps

1. **Create the VPC**
   ```bash
   aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=MyCustomVPC}]'
   ```

2. **Create Subnets** (Public & Private across 2 AZs)
   ```bash
   aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.1.0/24 --availability-zone us-east-1a
   aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.3.0/24 --availability-zone us-east-1a
   ```

3. **Attach Internet Gateway**
   ```bash
   aws ec2 create-internet-gateway
   aws ec2 attach-internet-gateway --vpc-id <vpc-id> --internet-gateway-id <igw-id>
   ```

4. **Create NAT Gateway** (in each public subnet)
   ```bash
   aws ec2 allocate-address --domain vpc
   aws ec2 create-nat-gateway --subnet-id <public-subnet-id> --allocation-id <eip-id>
   ```

5. **Configure Route Tables**
   - Public route table → `0.0.0.0/0` → IGW
   - Private route table → `0.0.0.0/0` → NAT Gateway

6. **Launch EC2 Instances** with the appropriate Security Groups in private subnets

7. **Create ALB** targeting EC2 instances in private subnets

8. **Configure Auto Scaling Group** with launch template

---

## 🎯 Key Takeaways

- Proper **network segmentation** using public/private subnets is critical for security
- **NAT Gateway** enables secure outbound internet access for private resources without exposing them
- **Multi-AZ deployment** ensures fault tolerance and high availability
- **ALB + ASG** together create a self-healing, scalable architecture
- **Defense in depth** using both Security Groups and NACLs provides layered security

---

## 👤 Author

**Your Name**
- GitHub: [MostafaAli](https://github.com/mostafaAli-17)
- LinkedIn: [Your LinkedIn](https://www.linkedin.com/in/mostafa-ali-goda378)

---

> ⭐ If you found this project helpful, feel free to star the repository!
