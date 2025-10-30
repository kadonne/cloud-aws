# Cloud Computing CI/CD Pipeline with Terraform & GitHub Actions

Automated CI/CD pipeline for deploying the Umami web analytics application to AWS using Infrastructure as Code (Terraform) and GitHub Actions workflows. This project demonstrates modern DevOps practices including infrastructure automation, containerization, and continuous deployment.

## Project Overview

This project automates the complete lifecycle of deploying a web application to AWS cloud infrastructure. It provisions AWS resources using Terraform, containerizes the Umami analytics application with Docker, and orchestrates the entire deployment process through GitHub Actions workflows.

**Deployed Application**: [Umami](https://umami.is/) - Open-source web analytics platform (privacy-friendly alternative to Google Analytics)

## Team Members
* **Ammar**: Lead developer - Terraform implementation, Umami analytics utilizer.
* **Rima**: CI/CD pipeline - assisted in writing Github Actions file.
## Architecture

### Infrastructure Components

```
┌─────────────────────────────────────────────────────────────┐
│                         AWS Cloud                            │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  VPC (10.0.0.0/16)                                     │ │
│  │  ┌──────────────────────────────────────────────────┐ │ │
│  │  │  Public Subnet (10.0.1.0/24)                     │ │ │
│  │  │  ┌────────────────────────────────────────────┐  │ │ │
│  │  │  │  EC2 Instance (t2.micro)                   │  │ │ │
│  │  │  │  ┌──────────────────────────────────────┐  │  │ │ │
│  │  │  │  │  Docker Container                    │  │  │ │ │
│  │  │  │  │  - Umami Application                 │  │  │ │ │
│  │  │  │  │  - PostgreSQL Database               │  │  │ │ │
│  │  │  │  └──────────────────────────────────────┘  │  │ │ │
│  │  │  └────────────────────────────────────────────┘  │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  │                                                         │ │
│  │  Internet Gateway ←→ Route Table                       │ │
│  │  Security Group (SSH, HTTP, All Traffic)               │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                            ↑
                            │
                ┌───────────┴───────────┐
                │   GitHub Actions      │
                │   - Build Docker      │
                │   - Terraform Apply   │
                │   - Deploy to EC2     │
                └───────────────────────┘
```

### CI/CD Pipeline Flow

```
GitHub Push/Manual Trigger
         ↓
   GitHub Actions Workflow
         ↓
    ┌────┴────┐
    ↓         ↓
Build Docker  Skip Build
    ↓         ↓
Push to Hub   ↓
    ↓         ↓
    └────┬────┘
         ↓
  Terraform Init
         ↓
  Terraform Validate
         ↓
  Terraform Apply
         ↓
  Provision AWS Resources
   - VPC & Subnet
   - Internet Gateway
   - Security Groups
   - EC2 Instance
         ↓
  SSH into EC2
         ↓
  Deploy Docker Compose
         ↓
  Application Live ✓
```

## Features

### Infrastructure as Code
* **Terraform**: Complete AWS infrastructure definition
* **State Management**: Remote state storage in S3
* **Version Control**: Infrastructure changes tracked in Git
* **Idempotent**: Repeatable deployments with consistent results

### Automated CI/CD
* **GitHub Actions**: Workflow automation
* **Manual Triggers**: Workflow dispatch for controlled deployments
* **Multi-stage Pipeline**: Build, deploy, and destroy options
* **Secret Management**: Secure credential storage

### Security
* **VPC Isolation**: Custom Virtual Private Cloud
* **Security Groups**: Granular ingress/egress rules
* **SSH Key Authentication**: Secure EC2 access
* **AWS IAM**: Least privilege access controls

### Containerization
* **Docker**: Application containerization
* **Docker Compose**: Multi-container orchestration
* **Docker Hub**: Container registry integration
* **Automated Builds**: CI pipeline builds and pushes images

## Repository Structure

```
cloud-computing-project/
├── terraform/
│   └── main.tf                    # Complete infrastructure definition
├── umami/
│   ├── Dockerfile                 # Umami application container
│   └── docker-compose.yml         # Multi-container setup
├── .github/
│   └── workflows/
│       └── deploy.yml             # CI/CD workflow
└── README.md
```

## Prerequisites

### Required Tools
* Terraform >= 1.0
* Docker & Docker Compose
* AWS CLI
* Git
* SSH client

### Required Accounts
* AWS Account with programmatic access
* Docker Hub account
* GitHub repository with Actions enabled

### Required Secrets

Configure these secrets in GitHub repository settings:

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `AWS_ACCESS_KEY` | AWS IAM access key ID | `AKIAIOSFODNN7EXAMPLE` |
| `AWS_SECRET_ACCESS_KEY` | AWS IAM secret access key | `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` |
| `DOCKER_HUB_TOKEN` | Docker Hub access token | `dckr_pat_...` |
| `EC2_KEY` | Private SSH key for EC2 | `-----BEGIN RSA PRIVATE KEY-----...` |

## Installation & Setup

### 1. Clone Repository

```bash
git clone https://github.com/yourusername/cloud-computing-project.git
cd cloud-computing-project
```

### 2. Configure AWS Credentials

**Create IAM User with permissions:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "vpc:*",
        "s3:*"
      ],
      "Resource": "*"
    }
  ]
}
```

### 3. Create S3 Bucket for Terraform State

```bash
aws s3 mb s3://ammar-bucket --region us-west-2
```

### 4. Generate SSH Key Pair

```bash
# Generate new key pair
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ec2_key

# Get public key (add to terraform main.tf)
cat ~/.ssh/ec2_key.pub

# Get private key (add to GitHub Secrets as EC2_KEY)
cat ~/.ssh/ec2_key
```

### 5. Configure GitHub Secrets

Navigate to: `Repository → Settings → Secrets and variables → Actions → New repository secret`

Add all four required secrets listed above.

### 6. Update Terraform Configuration

Edit `terraform/main.tf`:

```hcl
terraform {
  backend "s3" {
    bucket = "your-bucket-name"  # Update this
    key = "state/main.tf.tfstate"
    region = "us-west-2"
  }
}

resource "aws_key_pair" "deployer" {
  key_name   = "d-key"
  public_key = "YOUR_PUBLIC_KEY_HERE"  # Update this
}
```

## Usage

### Deployment Options

The GitHub Actions workflow provides three deployment modes via manual dispatch:

#### Option 1: Build Image + Deploy

1. Go to `Actions` tab in GitHub
2. Select `Deploy to AWS` workflow
3. Click `Run workflow`
4. Select:
   * Docker: `build image`
   * Action: `apply`
5. Click `Run workflow`

**This will:**
* Build Docker image from source
* Push to Docker Hub
* Provision AWS infrastructure
* Deploy application to EC2

#### Option 2: Deploy Only (Skip Build)

1. Go to `Actions` tab
2. Select `Deploy to AWS` workflow
3. Click `Run workflow`
4. Select:
   * Docker: `N/A`
   * Action: `apply`
5. Click `Run workflow`

**This will:**
* Skip Docker build
* Use existing Docker image
* Provision AWS infrastructure
* Deploy application to EC2

#### Option 3: Destroy Infrastructure

1. Go to `Actions` tab
2. Select `Deploy to AWS` workflow
3. Click `Run workflow`
4. Select:
   * Docker: `N/A`
   * Action: `destroy`
5. Click `Run workflow`

**This will:**
* Destroy all AWS resources
* Clean up infrastructure
* Remove EC2, VPC, etc.

### Accessing the Application

After successful deployment:

1. Check GitHub Actions workflow output for public IP
2. Access Umami at: `http://<PUBLIC_IP>:3000`
3. Default credentials (if using Umami defaults):
   * Username: `admin`
   * Password: `umami`

### Manual Terraform Commands

For local development/testing:

```bash
# Navigate to terraform directory
cd terraform/

# Initialize Terraform
terraform init

# Validate configuration
terraform validate

# Plan infrastructure changes
terraform plan

# Apply changes
terraform apply

# Get outputs
terraform output public_ip

# Destroy infrastructure
terraform destroy
```

### Manual Docker Operations

```bash
# Build Docker image locally
cd umami/
docker build -t kadonne55/cloud_computing:latest .

# Test locally
docker compose up -d

# Check running containers
docker ps

# View logs
docker compose logs -f

# Stop containers
docker compose down
```

## Infrastructure Details

### AWS Resources Created

**VPC Configuration:**
* CIDR Block: `10.0.0.0/16`
* Public Subnet: `10.0.1.0/24`
* Internet Gateway attached
* Route table with default route to IGW

**EC2 Instance:**
* Instance Type: `t2.micro` (Free tier eligible)
* AMI: Amazon Linux 2 (`ami-005e54dee72cc1d00`)
* Region: `us-west-2` (Oregon)
* Public IP: Auto-assigned
* CPU Credits: Unlimited (burstable performance)

**Security Group Rules:**

| Type | Protocol | Port | Source | Description |
|------|----------|------|--------|-------------|
| Ingress | TCP | 22 | 0.0.0.0/0 | SSH access |
| Ingress | All | All | 0.0.0.0/0 | Application access |
| Egress | All | All | 0.0.0.0/0 | Outbound traffic |

**⚠️ Security Note**: The security group allows all inbound traffic (`0.0.0.0/0`). For production, restrict to specific IPs/ports.

### Terraform Resource Graph

```hcl
aws_vpc
  └── aws_subnet
      ├── aws_internet_gateway
      │   └── aws_route_table
      │       └── aws_route_table_association
      └── aws_security_group
          └── aws_instance
              └── aws_key_pair
```

### State Management

Terraform state is stored remotely in S3:
* **Bucket**: `ammar-bucket`
* **Key**: `state/main.tf.tfstate`
* **Region**: `us-west-2`

**Benefits:**
* Team collaboration
* State locking (with DynamoDB)
* Disaster recovery
* Version history

## CI/CD Workflow Details

### Workflow Triggers

```yaml
on:
  workflow_dispatch:  # Manual trigger only
```

### Job: Build Docker Image

**Runs when**: `docker == 'build image'`

```yaml
Steps:
1. Checkout repository
2. Build Docker image from Dockerfile
3. Login to Docker Hub
4. Push image to Docker Hub (kadonne55/cloud_computing:latest)
```

### Job: Terraform Apply

**Runs when**: `action == 'apply'`

**Dependencies**: `needs: build` (or skips if N/A)

```yaml
Steps:
1. Checkout repository
2. Configure AWS credentials
3. Setup Terraform CLI
4. Initialize Terraform (download providers, configure backend)
5. Validate Terraform configuration
6. Apply infrastructure changes (auto-approve)
7. SSH into EC2 instance
8. Clone application repository
9. Install Docker & Docker Compose
10. Deploy containers with docker-compose up
11. Output public IP address
```

### Job: Terraform Destroy

**Runs when**: `action == 'destroy'`

```yaml
Steps:
1. Checkout repository
2. Configure AWS credentials
3. Setup Terraform CLI
4. Initialize Terraform
5. Destroy all infrastructure (auto-approve)
```

## Docker Configuration

### Umami Application Stack

The application uses Docker Compose with two services:

**Service 1: PostgreSQL Database**
* Image: `postgres:15-alpine`
* Port: 5432 (internal)
* Environment: Database name, user, password
* Volume: Persistent data storage

**Service 2: Umami Application**
* Image: `kadonne55/cloud_computing:latest`
* Port: 3000 (exposed)
* Environment: Database connection string
* Depends on: PostgreSQL service

### Dockerfile Structure

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

## Troubleshooting

### Common Issues

**Issue: Terraform state locked**
```bash
Solution: 
terraform force-unlock <LOCK_ID>
# Or wait for previous operation to complete
```

**Issue: EC2 connection timeout**
```bash
Solution:
1. Verify security group allows your IP
2. Check EC2 instance is running
3. Wait 2-3 minutes after creation for SSH to be ready
4. Verify SSH key permissions: chmod 600 ec2_key.pem
```

**Issue: Docker containers not starting**
```bash
Solution:
# SSH into EC2
ssh -i ec2_key.pem ubuntu@<PUBLIC_IP>

# Check Docker status
sudo systemctl status docker

# Check container logs
cd team-8/umami
sudo docker compose logs

# Restart containers
sudo docker compose down
sudo docker compose up -d
```

**Issue: Workflow fails at Terraform apply**
```bash
Solution:
1. Check AWS credentials are valid
2. Verify S3 bucket exists
3. Check IAM permissions
4. Review Terraform logs in Actions output
```

**Issue: Port 3000 not accessible**
```bash
Solution:
1. Verify security group allows inbound traffic
2. Check application is running: sudo docker ps
3. Test locally on EC2: curl localhost:3000
4. Check EC2 instance has public IP
```

### Debug Commands

**Check Terraform state:**
```bash
terraform state list
terraform state show aws_instance.foo
```

**Check EC2 instance:**
```bash
aws ec2 describe-instances --instance-ids <INSTANCE_ID>
```

**Check Docker status:**
```bash
ssh -i ec2_key.pem ubuntu@<PUBLIC_IP>
sudo docker ps -a
sudo docker compose logs -f
```

**Check security group rules:**
```bash
aws ec2 describe-security-groups --group-ids <SG_ID>
```

## Security Best Practices

### Implemented

* ✅ Remote state storage in S3
* ✅ SSH key authentication
* ✅ AWS IAM for access control
* ✅ Secrets stored in GitHub Secrets
* ✅ VPC isolation

### Recommended Improvements

* [ ] Restrict security group to specific IPs
* [ ] Enable S3 bucket encryption
* [ ] Add DynamoDB for state locking
* [ ] Implement AWS KMS for encryption
* [ ] Use private subnets with NAT gateway
* [ ] Add CloudWatch monitoring
* [ ] Implement SSL/TLS certificates
* [ ] Enable VPC Flow Logs
* [ ] Use AWS Systems Manager Session Manager instead of SSH
* [ ] Implement least-privilege IAM policies

### Production Considerations

```hcl
# Restrict SSH access
ingress {
  from_port   = 22
  to_port     = 22
  protocol    = "tcp"
  cidr_blocks = ["YOUR_IP/32"]  # Your IP only
}

# Restrict application access
ingress {
  from_port   = 3000
  to_port     = 3000
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]  # Or specific ranges
}

# Remove unrestricted ingress
# Delete the 0.0.0.0/0 all-traffic rule
```

## Cost Considerations

### AWS Free Tier (First 12 months)

* **EC2**: 750 hours/month of t2.micro
* **VPC**: No charge
* **S3**: 5GB storage, 20,000 GET requests
* **Data Transfer**: 100GB outbound

### Estimated Monthly Cost (After Free Tier)

| Resource | Cost |
|----------|------|
| t2.micro EC2 (24/7) | ~$8.50 |
| EBS Storage (8GB) | ~$0.80 |
| Data Transfer | ~$1-5 |
| **Total** | **~$10-15/month** |

### Cost Optimization

```bash
# Stop instance when not in use
aws ec2 stop-instances --instance-ids <INSTANCE_ID>

# Start when needed
aws ec2 start-instances --instance-ids <INSTANCE_ID>

# Or use Terraform destroy/apply
terraform destroy  # Remove all resources
terraform apply    # Recreate when needed
```

## Monitoring & Logging

### GitHub Actions Logs

* View workflow runs in Actions tab
* Download logs for offline analysis
* Check step-by-step execution details

### EC2 Logs

```bash
# SSH into instance
ssh -i ec2_key.pem ubuntu@<PUBLIC_IP>

# View system logs
sudo journalctl -u docker -f

# View application logs
cd team-8/umami
sudo docker compose logs -f umami
sudo docker compose logs -f postgres
```

### AWS CloudWatch (Optional)

Enable CloudWatch for advanced monitoring:

```hcl
resource "aws_cloudwatch_log_group" "ec2_logs" {
  name = "/aws/ec2/umami"
  retention_in_days = 7
}
```

## Future Enhancements

* [ ] Add HTTPS with Let's Encrypt SSL certificates
* [ ] Implement Auto Scaling Group for high availability
* [ ] Add Application Load Balancer
* [ ] Set up CloudWatch alarms and SNS notifications
* [ ] Implement blue-green deployments
* [ ] Add automated testing in CI pipeline
* [ ] Use AWS Parameter Store for secrets
* [ ] Implement infrastructure drift detection
* [ ] Add disaster recovery with cross-region replication
* [ ] Create custom AMI with pre-installed dependencies
* [ ] Implement container health checks
* [ ] Add backup automation for PostgreSQL
* [ ] Use AWS ECS/EKS for container orchestration

## Learning Objectives

This project demonstrates:

* **Infrastructure as Code**: Terraform for AWS provisioning
* **CI/CD Pipelines**: GitHub Actions workflow automation
* **Containerization**: Docker and Docker Compose
* **Cloud Computing**: AWS VPC, EC2, security groups
* **DevOps Practices**: Automated deployments, IaC
* **Version Control**: Git for infrastructure and code
* **Security**: IAM, SSH keys, secret management

## Contributing

This is an educational project. For questions or improvements:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## License

Educational project for learning cloud computing and DevOps practices.

## Acknowledgments

* [Umami Analytics](https://umami.is/) - Open-source web analytics
* [Terraform](https://www.terraform.io/) - Infrastructure as Code
* [GitHub Actions](https://github.com/features/actions) - CI/CD platform
* AWS Free Tier for development resources
