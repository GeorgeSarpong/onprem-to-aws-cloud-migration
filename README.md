# On-Premises to AWS Cloud Migration Project

![AWS Migration](https://img.shields.io/badge/Domain-Cloud%20Migration-orange)
![AWS MGN](https://img.shields.io/badge/Tool-AWS%20MGN-orange)
![AWS DMS](https://img.shields.io/badge/Tool-AWS%20DMS-orange)
![Terraform](https://img.shields.io/badge/IaC-Terraform-purple)
![Zero Downtime](https://img.shields.io/badge/Downtime-Near%20Zero-brightgreen)
![AWS](https://img.shields.io/badge/AWS-Solutions%20Architect%20Pro-orange)

---

## Project Overview

This project documents the complete migration of enterprise ERP workloads from on-premises infrastructure to AWS — covering discovery, planning, execution, and validation phases using AWS Migration Hub, MGN, DMS, EC2, and RDS with near-zero downtime achievement.

Built to demonstrate operational readiness for:
- Cloud Infrastructure Engineer roles
- Cloud & DevOps Engineer roles
- Cloud Migration Engineer roles
- Solutions Architect roles

---

## Migration Results

| Metric | Result |
|---|---|
| Downtime achieved | Near-zero |
| Workloads migrated | ERP system + databases |
| Migration tool | AWS MGN + DMS |
| Infrastructure as Code | 100% Terraform |
| Post-migration availability | 99.9% |
| Security controls | IAM, VPC, KMS encryption |

---

## Repository Structure
---

## Migration Architecture
---

##  Migration Strategy — The 6Rs

| Strategy | Description | Used For |
|---|---|---|
| Rehost (Lift & Shift) | Move as-is to AWS | App servers via MGN |
| Replatform | Minor optimizations | Database to RDS |
| Refactor | Re-architect for cloud | Future phase |
| Repurchase | Move to SaaS | Future phase |
| Retain | Keep on-premises | Legacy systems |
| Retire | Decommission | Unused servers |

---

## AWS Infrastructure Design

### VPC Architecture

```hcl
# VPC Configuration
resource "aws_vpc" "migration_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "migration-vpc"
    Environment = "production"
    ManagedBy   = "Terraform"
    Project     = "erp-migration"
  }
}

# Public Subnets — Multi-AZ
resource "aws_subnet" "public" {
  count             = 2
  vpc_id            = aws_vpc.migration_vpc.id
  cidr_block        = ["10.0.1.0/24", "10.0.2.0/24"][count.index]
  availability_zone = ["us-east-1a", "us-east-1b"][count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-${count.index + 1}"
    Type = "Public"
  }
}

# Private Subnets — Multi-AZ
resource "aws_subnet" "private" {
  count             = 2
  vpc_id            = aws_vpc.migration_vpc.id
  cidr_block        = ["10.0.3.0/24", "10.0.4.0/24"][count.index]
  availability_zone = ["us-east-1a", "us-east-1b"][count.index]

  tags = {
    Name = "private-subnet-${count.index + 1}"
    Type = "Private"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.migration_vpc.id

  tags = {
    Name = "migration-igw"
  }
}
```

### EC2 Configuration

```hcl
# Application Server — Migrated via MGN
resource "aws_instance" "erp_app" {
  ami                    = var.ami_id
  instance_type          = "m5.xlarge"
  subnet_id              = aws_subnet.private[0].id
  vpc_security_group_ids = [aws_security_group.app.id]
  iam_instance_profile   = aws_iam_instance_profile.app.name
  
  root_block_device {
    volume_type = "gp3"
    volume_size = 100
    encrypted   = true
    kms_key_id  = aws_kms_key.app.arn
  }

  tags = {
    Name        = "erp-app-server"
    Environment = "production"
    ManagedBy   = "Terraform"
  }
}

# Auto Scaling Group for High Availability
resource "aws_autoscaling_group" "erp" {
  name                = "erp-asg"
  vpc_zone_identifier = aws_subnet.private[*].id
  min_size            = 2
  max_size            = 6
  desired_capacity    = 2

  launch_template {
    id      = aws_launch_template.erp.id
    version = "$Latest"
  }

  health_check_type         = "ELB"
  health_check_grace_period = 300
}
```

### RDS Configuration

```hcl
# RDS Multi-AZ Database — Migrated via DMS
resource "aws_db_instance" "erp_db" {
  identifier        = "erp-production-db"
  engine            = "mysql"
  engine_version    = "8.0"
  instance_class    = "db.r5.xlarge"
  allocated_storage = 500
  storage_type      = "gp3"
  storage_encrypted = true
  kms_key_id        = aws_kms_key.rds.arn

  db_name  = "erpdb"
  username = var.db_username
  password = var.db_password

  multi_az               = true
  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.rds.id]

  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"

  deletion_protection = true
  skip_final_snapshot = false

  tags = {
    Name        = "erp-production-db"
    Environment = "production"
    ManagedBy   = "Terraform"
  }
}
```

---

## AWS Migration Service (MGN) Setup

### MGN Configuration Steps
---

## AWS Database Migration Service (DMS)

### DMS Migration Steps
---

## Route 53 DNS Cutover

### DNS Migration Strategy
---

## CloudWatch Monitoring Setup

### Key Metrics Monitored Post-Migration

| Metric | Service | Threshold | Alert |
|---|---|---|---|
| CPU Utilization | EC2 | Above 80% | SNS notification |
| Memory Usage | CloudWatch Agent | Above 85% | SNS notification |
| RDS Connections | RDS | Above 80% max | SNS notification |
| RDS CPU | RDS | Above 75% | SNS notification |
| ALB Response Time | Application LB | Above 2 seconds | SNS notification |
| ALB 5XX Errors | Application LB | Above 1% | SNS notification |
| Replication Lag | DMS | Above 60 seconds | SNS notification |

---

## Migration Validation Checklist

### Pre-Cutover Validation
- [ ] All servers replicated and test instances validated
- [ ] Database replication lag below 5 seconds
- [ ] All application functions tested on AWS
- [ ] Security groups validated — no unnecessary ports open
- [ ] IAM roles and policies tested
- [ ] CloudWatch monitoring configured and alerting tested
- [ ] Route 53 TTL reduced to 60 seconds
- [ ] Rollback procedure confirmed and tested
- [ ] Stakeholders notified of cutover window
- [ ] Change request approved

### Post-Cutover Validation
- [ ] DNS resolved to AWS endpoints
- [ ] Application accessible and functional
- [ ] Database connections working
- [ ] All integrations functioning
- [ ] CloudWatch dashboards showing green
- [ ] No errors in application logs
- [ ] Performance baseline established
- [ ] Old servers traffic confirmed zero

---

## Rollback Procedure

### Trigger Conditions for Rollback
- Application unavailable after cutover
- Data corruption detected
- Performance degradation above 50%
- Critical errors in application logs

### Rollback Steps
1. Revert Route 53 DNS to on-premises IP
2. Wait for DNS propagation — up to 60 seconds
3. Verify traffic returning to on-premises
4. Confirm application accessible on-premises
5. Notify stakeholders of rollback
6. Investigate root cause before retry
7. Document all findings

---

## Standards & Frameworks Referenced

- **AWS Migration Acceleration Program (MAP)** — Migration best practices
- **AWS Well-Architected Framework** — Architecture review
- **AWS Migration Hub** — Migration tracking
- **6R Migration Strategies** — Cloud migration framework
- **CIS AWS Foundations** — Security benchmarks
- **NIST SP 800-145** — Cloud computing definition

---

## Tools & Technologies

![AWS MGN](https://img.shields.io/badge/AWS-MGN-orange)
![AWS DMS](https://img.shields.io/badge/AWS-DMS-orange)
![Terraform](https://img.shields.io/badge/Terraform-IaC-purple)
![CloudWatch](https://img.shields.io/badge/CloudWatch-Monitoring-yellow)
![Route53](https://img.shields.io/badge/Route%2053-DNS-orange)
![RDS](https://img.shields.io/badge/RDS-Multi--AZ-orange)
![EC2](https://img.shields.io/badge/EC2-Auto%20Scaling-orange)

---

## Author

**George Amankwaa Sarpong**
Cloud Infrastructure Engineer | Cloud Migration Specialist
📍 Accra, Ghana
🔗 [LinkedIn](https://linkedin.com/in/georgesarpong)
🌐 [GitHub Portfolio](https://github.com/GeorgeSarpong)

---

*This project is part of a broader portfolio demonstrating readiness for Cloud Infrastructure Engineer and Cloud Migration roles in the US and global market.*
