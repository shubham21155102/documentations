# 🌍 Terraform

> Frequently used Terraform commands for infrastructure as code.

[← Back to Home](../README.md)

---

## 📋 Table of Contents

- [Installation & Setup](#installation--setup)
- [Core Workflow](#core-workflow)
- [State Management](#state-management)
- [Workspaces](#workspaces)
- [Modules](#modules)
- [Variables & Outputs](#variables--outputs)
- [Import & Move](#import--move)
- [Debugging](#debugging)
- [Common HCL Patterns](#common-hcl-patterns)

---

## Installation & Setup

```bash
# Install Terraform (Ubuntu)
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Verify
terraform version

# Enable tab completion
terraform -install-autocomplete
```

---

## Core Workflow

```bash
# Initialize working directory (download providers & modules)
terraform init
terraform init -upgrade          # upgrade providers

# Format code
terraform fmt
terraform fmt -recursive

# Validate configuration
terraform validate

# Plan (preview changes)
terraform plan
terraform plan -out=tfplan       # save plan
terraform plan -var="region=us-east-1"
terraform plan -var-file="prod.tfvars"

# Apply changes
terraform apply
terraform apply tfplan           # apply saved plan
terraform apply -auto-approve    # skip confirmation
terraform apply -var="region=us-east-1"

# Destroy infrastructure
terraform destroy
terraform destroy -auto-approve
terraform destroy -target=aws_instance.web

# Apply/destroy specific resource
terraform apply -target=aws_instance.web
terraform destroy -target=module.vpc
```

---

## State Management

```bash
# Show current state
terraform show
terraform show -json            # JSON format

# List resources in state
terraform state list

# Show a specific resource
terraform state show aws_instance.web

# Move resource in state
terraform state mv aws_instance.web aws_instance.app

# Remove resource from state (without destroying)
terraform state rm aws_instance.web

# Pull remote state
terraform state pull

# Push local state to remote
terraform state push terraform.tfstate

# Refresh state from real infrastructure
terraform refresh
```

---

## Workspaces

```bash
# List workspaces
terraform workspace list

# Create workspace
terraform workspace new staging

# Select workspace
terraform workspace select production

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete staging
```

---

## Modules

```bash
# Initialize (downloads modules)
terraform init

# Get modules only
terraform get
terraform get -update           # update modules

# Module source types:
# Local:   source = "./modules/vpc"
# Git:     source = "git::https://github.com/org/repo.git//modules/vpc?ref=v1.0"
# Registry:source = "hashicorp/consul/aws"  version = "0.1.0"
```

---

## Variables & Outputs

```bash
# Pass variable on command line
terraform apply -var="instance_type=t3.micro"

# Use var file
terraform apply -var-file="prod.tfvars"

# Show outputs
terraform output
terraform output instance_ip
terraform output -json
```

**`variables.tf` example:**

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}
```

**`outputs.tf` example:**

```hcl
output "instance_ip" {
  description = "Public IP of the instance"
  value       = aws_instance.web.public_ip
}
```

---

## Import & Move

```bash
# Import existing resource into state
terraform import aws_instance.web i-1234567890abcdef0

# Generate config for imported resource (Terraform 1.5+)
terraform plan -generate-config-out=generated.tf
```

---

## Debugging

```bash
# Enable debug logging
TF_LOG=DEBUG terraform apply
TF_LOG=TRACE terraform plan

# Log to file
TF_LOG=DEBUG TF_LOG_PATH=./terraform.log terraform apply

# Show provider version constraints
terraform providers

# Taint resource (force replacement on next apply) — deprecated in 1.x
terraform taint aws_instance.web    # old
# Use -replace flag instead:
terraform apply -replace="aws_instance.web"
```

---

## Common HCL Patterns

**`main.tf` — AWS EC2:**

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket = "my-tfstate-bucket"
    key    = "prod/terraform.tfstate"
    region = "us-east-1"
  }
}

provider "aws" {
  region = var.region
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type

  tags = {
    Name = "web-server"
    Env  = "production"
  }
}
```

**Locals:**

```hcl
locals {
  env         = "production"
  common_tags = {
    Environment = local.env
    ManagedBy   = "Terraform"
  }
}
```

**Data sources:**

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-22.04-amd64-server-*"]
  }
}
```

**For-each:**

```hcl
resource "aws_instance" "server" {
  for_each      = toset(["web1", "web2", "web3"])
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  tags = { Name = each.key }
}
```

---

[← Back to Home](../README.md)
