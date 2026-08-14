## aeims-control

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

AEIMS-Control is the infrastructure-as-code (IaC) and deployment automation repository for the AEIMS (Adult Entertainment & Interactive Management System) platform. This repository manages cloud infrastructure, deployment pipelines, and configuration management for a comprehensive VoIP telephony platform with interactive device control capabilities.

## Development Commands

### Terraform Infrastructure Management
```bash
# Initialize Terraform
terraform init

# Plan infrastructure changes
terraform plan

# Apply infrastructure changes
terraform apply

# Destroy infrastructure (use with caution)
terraform destroy

# Format Terraform files
terraform fmt

# Validate Terraform configuration
terraform validate
```

### AWS CLI Commands
```bash
# Configure AWS credentials
aws configure

# Check EC2 instances
aws ec2 describe-instances --filters "Name=tag:Project,Values=AEIMS"

# Check ALB health
aws elbv2 describe-target-health --target-group-arn [TARGET_GROUP_ARN]

# SSM commands for remote instance management
aws ssm send-command --instance-ids [INSTANCE_ID] --document-name "AWS-RunShellScript"

# ECR repository management
aws ecr describe-repositories
```

### Deployment Scripts
```bash
# Build and deploy all services
./deploy-multi-site.sh

# Clean deployment (removes existing containers)
./clean-deploy-all.sh

# Test all virtual hosts
./test-all-vhosts.sh

# Complete deployment verification
./comprehensive-validation.js
```

## Architecture Overview

### Core Infrastructure Components

- **VPC & Networking** (`vpc.tf`): Multi-AZ VPC with public/private subnets, NAT gateways
- **Auto Scaling Group** (`ec2-autoscaling.tf`): Ubuntu 22.04 LTS instances with Ansible integration
- **Application Load Balancer** (`alb.tf`): Multi-target load balancer for web traffic
- **ECR Repositories** (`ecr.tf`): Container registries for microservices
- **Security Groups** (`security-groups.tf`): Firewall rules for VoIP and web traffic
- **KMS Encryption** (`kms.tf`): Encryption keys for data at rest
- **CloudWatch Monitoring** (`cloudwatch.tf`): Logging and metrics collection

### Key Directories

- `terraform/` - Main Terraform configuration files
- `ansible/` - Ansible playbooks for configuration management
- `user-data/` - Instance bootstrap scripts
- `scripts/` - Deployment and utility scripts
- `docker/` - Docker configurations and Dockerfiles

### Multi-Site Architecture

The platform supports multiple client sites:
- **flirts.nyc** - Dating platform client site
- **aeims.app** - Operator portal and admin interface
- **sexacomms.com** - Adult entertainment platform control panel
- **nycflirts.com** - Legacy client site (separate infrastructure)

### Technology Stack

- **Infrastructure**: AWS (EC2, ALB, VPC, ECR, KMS)
- **Operating System**: Ubuntu 22.04 LTS
- **Configuration Management**: Ansible
- **Containerization**: Docker with ECR
- **Load Balancing**: Application Load Balancer (ALB)
- **Monitoring**: CloudWatch, SSM
- **CI/CD**: Terraform + Ansible automation

## Environment Configuration

### AWS Infrastructure Settings

The system uses environment-based configuration:
- **Production**: `aeims-production` environment
- **ALB**: `aeims-alb-production-1271381208.us-east-1.elb.amazonaws.com`
- **Target Group**: `aeims-app-tg-production`
- **Auto Scaling Group**: `aeims-asg-production`
- **Region**: `us-east-1`

### Key Configuration Files

- `terraform.tfvars` - Environment-specific variables
- `ansible/deploy.yml` - Main deployment playbook
- `user-data/ubuntu-bootstrap.sh` - Instance initialization script

## VoIP Platform Architecture

### Core VoIP Components

- **Asterisk PBX**: On-premises VoIP server for call routing
- **AWS Connect Integration**: Cloud-based call center capabilities
- **SIP Protocol**: Port 5060-5061 for signaling
- **RTP Media**: Ports 10000-20000 for voice streams
- **High-Volume Calling**: 1000+ concurrent call capacity

### Adult Entertainment Features

- **Interactive Device Control**: WebSocket-based device management
- **Pay-per-minute Billing**: Real-time billing integration
- **Operator Management**: Multi-role user system
- **Multi-site Support**: White-label platform capabilities

## Security Architecture

### Network Security

- **Security Groups**: Restrictive firewall rules
- **VPC Isolation**: Private subnets for backend services
- **SSL/TLS**: End-to-end encryption for web traffic
- **KMS Encryption**: Data encryption at rest

### Access Control

- **IAM Roles**: Service-specific permissions
- **SSM Access**: Secure shell access without SSH keys
- **VPN Access**: Controlled admin access to private resources

## Deployment Strategy

### Infrastructure Deployment

1. **Terraform Apply**: Create/update AWS infrastructure
2. **Instance Refresh**: Rolling update of EC2 instances
3. **Health Checks**: Verify ALB target health
4. **Service Deployment**: Deploy microservices via Ansible

### Multi-Service Deployment

The platform consists of multiple microservices:
- **admin-service**: Administrative interface (PHP)
- **user-service**: User management API
- **billing-service**: Payment processing
- **call-service**: VoIP call management
- **analytics-service**: Metrics and reporting
- **operator-service**: Operator dashboard
- **conference-service**: Conference calling
- **React Frontend**: User interface (Node.js)
- **aeimsLib**: Interactive device control library

### Health Monitoring

- **ALB Health Checks**: `/health` endpoint monitoring
- **Target Group Health**: Instance-level health verification
- **CloudWatch Metrics**: Performance and error monitoring
- **Automated Recovery**: Auto Scaling Group instance replacement

## Common Operations

### Deployment Troubleshooting

```bash
# Check ALB health
curl -k https://aeims-alb-production-1271381208.us-east-1.elb.amazonaws.com/health

# Check target health
aws elbv2 describe-target-health --target-group-arn arn:aws:elasticloadbalancing:us-east-1:515966511618:targetgroup/aeims-app-tg-production/50c36fa193066446

# Check instance refresh status
aws autoscaling describe-instance-refreshes --auto-scaling-group-name aeims-asg-production

# SSM command execution
aws ssm list-command-invocations --command-id [COMMAND_ID]
```

### Virtual Host Configuration

Virtual hosts are configured via nginx for multi-site support:
- Sites require proper DNS pointing to ALB
- nginx virtual host configurations deployed via SSM
- Health endpoints must respond correctly for ALB checks

### Emergency Procedures

```bash
# Emergency nginx fix deployment
aws ssm send-command --instance-ids [INSTANCE_IDS] --document-name "AWS-RunShellScript" --parameters 'commands=["nginx -t && systemctl reload nginx"]'

# Force instance refresh
aws autoscaling start-instance-refresh --auto-scaling-group-name aeims-asg-production

# Scale out for high load
aws autoscaling set-desired-capacity --auto-scaling-group-name aeims-asg-production --desired-capacity [COUNT]
```

## Integration Points

### AEIMS Telephony Platform Integration

- **Service Integration**: Connects to main AEIMS application
- **Database Sharing**: Shared user and billing data
- **API Integration**: REST APIs for cross-service communication

### AeimsLib Integration

- **Device Control**: WebSocket protocol for interactive devices
- **Docker Integration**: Containerized deployment
- **Ubuntu Compatibility**: Aligned with main infrastructure

## Best Practices

### Infrastructure Management

- Always run `terraform plan` before `terraform apply`
- Use feature flags for gradual rollouts
- Monitor ALB health during deployments
- Keep Terraform state files secure
- Use encrypted AMIs for instances

### Security Practices

- Rotate IAM access keys regularly
- Use least-privilege IAM policies
- Enable CloudTrail for audit logging
- Encrypt all data at rest and in transit
- Regular security group audits

### Deployment Guidelines

- Test in staging environment first
- Use blue-green deployment patterns
- Monitor metrics during rollouts
- Have rollback procedures ready
- Document all configuration changes

## Troubleshooting Guide

### Common Issues

1. **ALB 502 Errors**: Check target health and nginx configuration
2. **SSM Connection Issues**: Verify IAM roles and SSM agent status
3. **Deployment Failures**: Check Ansible logs and instance user data
4. **VoIP Issues**: Verify security group ports and Asterisk configuration
5. **SSL Certificate Issues**: Check ACM certificate status

### Log Locations

- **Instance Logs**: `/var/log/cloud-init-output.log`
- **Nginx Logs**: `/var/log/nginx/error.log`
- **Application Logs**: Service-specific log directories
- **CloudWatch Logs**: Centralized logging for all services

## Regulatory Compliance

The platform adheres to adult entertainment industry compliance requirements:
- **Age Verification**: Integrated identity verification
- **Content Filtering**: Automated content moderation
- **Privacy Controls**: GDPR/CCPA compliance features
- **Audit Logging**: Comprehensive activity tracking

---
> Source: [straticus1/aeims-control](https://github.com/straticus1/aeims-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
