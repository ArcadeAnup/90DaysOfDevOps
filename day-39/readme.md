# Day 39: Terraform and Infrastructure as Code (IaC)

## The Problem with Manual Infrastructure

### Clicking AWS Console (Bad)
- Create EC2 instance
- Configure security groups
- Create VPC
- Assign elastic IP
- Document everything manually (nobody does)
- Want to recreate? Click through it again
- Disaster recovery? Manual and slow
- Team members: "How do we set this up?"

### Infrastructure as Code (Good)
- Write code describing infrastructure
- Version control your infrastructure
- Recreate entire infrastructure with one command
- Team members: read the code
- Disaster recovery: one command
- Audit trail: git history

## What is Terraform?

Terraform is a tool that lets you:
1. Write infrastructure as code (HCL syntax)
2. Preview changes before applying
3. Apply infrastructure changes
4. Manage infrastructure state
5. Destroy infrastructure cleanly

Works with: AWS, Azure, GCP, Kubernetes, Docker, GitHub, Stripe, basically anything with an API.

## Terraform Workflow

### 1. Write Configuration (HCL)

```hcl
resource "aws_instance" "webserver" {
  ami           = "ami-0c2f25c1f66b1ff4d"
  instance_type = "t2.micro"
  
  tags = {
    Name = "web-server"
  }
}
```

### 2. Initialize

```bash
terraform init
```

Downloads provider plugins (AWS provider, Azure provider, etc.)

### 3. Plan

```bash
terraform plan
```

Preview what Terraform will create/modify/destroy. Read the output carefully.

### 4. Apply

```bash
terraform apply
```

Actually creates infrastructure. Terraform asks for confirmation (yes/no).

### 5. Verify

```bash
# Check state
terraform state list
terraform state show aws_instance.webserver

# AWS console shows created EC2 instance
```


## Terraform Commands Summary

```bash
# Initialize working directory
terraform init

# Format code
terraform fmt

# Validate syntax
terraform validate

# Preview changes
terraform plan

# Apply changes
terraform apply

# Show current state
terraform show

# List resources in state
terraform state list

# Show specific resource
terraform state show resource_type.resource_name

# Destroy infrastructure
terraform destroy

# Refresh state (sync with actual resources)
terraform refresh

# Taint resource (force recreation)
terraform taint resource_type.resource_name
```

## Terraform vs Imperative

### Imperative (Manual)
```bash
aws ec2 run-instances --image-id ami-0c2f25c1f66b1ff4d --instance-type t2.micro
aws ec2 create-security-group --group-name webserver-sg
aws ec2 authorize-security-group-ingress --group-name webserver-sg --protocol tcp --port 80 --cidr 0.0.0.0/0
# ... many more commands
# Documentation: hope someone wrote it down
# Disaster recovery: manually recreate
```

### Declarative (Terraform)
```hcl
resource "aws_vpc" "main" { ... }
resource "aws_subnet" "main" { ... }
resource "aws_security_group" "webserver" { ... }
resource "aws_instance" "webserver" { ... }
```

```bash
terraform apply
# Everything created according to code
# Documentation: the code itself
# Disaster recovery: terraform apply again
```




## Key Concepts

1. **Declarative** - Describe desired state, Terraform makes it happen
2. **Idempotent** - `terraform apply` twice = same result (safe)
3. **State** - Terraform tracks what exists
4. **Plan** - Always preview before applying
5. **Version control** - Infrastructure as code in Git
6. **Reproducible** - Same configuration = same infrastructure
7. **Destroy safely** - Clean up when done

## What's Next

1. **State management** - Store in S3, enable locking
2. **Variables and outputs** - Parameterize configurations
3. **Modules** - Create reusable components
4. **Remote state** - Team collaboration
5. **CI/CD integration** - Automated infrastructure deployments
6. **Terraform Cloud** - Managed Terraform hosting



**Progress: 39/90 days complete**

