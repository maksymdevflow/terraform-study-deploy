# Terraform AWS Infrastructure Guide

This guide is a collection of notes and explanations from my Terraform learning journey. It explains how to set up AWS infrastructure using Terraform, and I hope it might be useful for others who are learning Terraform as well. We use variables to keep sensitive information like access keys and passwords secure.

## Table of Contents

1. [Installation](#installation)
2. [Getting Started with Terraform Commands](#getting-started-with-terraform-commands)
3. [Terraform Configuration](#terraform-configuration)
4. [AWS Provider Configuration](#aws-provider-configuration)
5. [VPC Configuration](#vpc-configuration)
6. [AMI Selection](#ami-selection)
7. [SSH Key Pair](#ssh-key-pair)
8. [Security Groups](#security-groups)
9. [EC2 Instance](#ec2-instance)
10. [RDS Database Instance](#rds-database-instance)
11. [S3 Bucket](#s3-bucket)

---

## Installation

Before you can use Terraform, you need to install it on your computer. Terraform is available for Windows, macOS, and Linux.

**Official installation guide**: [Install Terraform](https://developer.hashicorp.com/terraform/install)

The official HashiCorp website provides detailed installation instructions for all operating systems, including:
- Package manager installations (Homebrew for macOS, apt/yum for Linux)
- Binary downloads for direct installation
- Instructions for different system architectures (AMD64, ARM64, etc.)

After installation, verify that Terraform is working by running:

```bash
terraform version
```

This command will show you the installed Terraform version. Make sure it meets the minimum version requirement specified in your `terraform.tf` file (>= 1.2).

---

## Getting Started with Terraform Commands

Once Terraform is installed and your configuration files are ready, you need to run two main commands to deploy your infrastructure.

### 1. terraform init

The first command you must run is `terraform init`. This command initializes your Terraform working directory.

```bash
terraform init
```

**What this command does:**

- Downloads the required providers (like the AWS provider) specified in your `terraform.tf` file
- Creates a `.terraform` directory that stores provider plugins
- Initializes the backend for storing Terraform state
- Prepares your working directory for other Terraform commands

**When to use it:**

- When you first start working with a Terraform project
- After adding new providers to your configuration
- After updating provider versions in `terraform.tf`
- When you clone a Terraform project from a repository

**Important**: You must run `terraform init` before running any other Terraform commands like `terraform plan` or `terraform apply`.

### 2. terraform apply

After running `terraform init`, you can use `terraform apply` to create or update your infrastructure.

```bash
terraform apply
```

**What this command does:**

- Reads your Terraform configuration files (`main.tf`, `terraform.tf`, etc.)
- Creates an execution plan showing what resources will be created, modified, or destroyed
- Asks for your confirmation before making any changes
- Creates, updates, or destroys AWS resources according to your configuration

**What happens when you run it:**

1. Terraform shows you a plan of all the changes it will make
2. You review the plan to see what will be created (EC2 instance, RDS database, S3 bucket, etc.)
3. You type `yes` to confirm and apply the changes, or `no` to cancel
4. Terraform creates all the resources in AWS
5. Terraform saves the state of your infrastructure in a `terraform.tfstate` file

**Before running terraform apply, make sure:**

- You have set up your variables file (`terraform.tfvars`) with:
  - AWS access key and secret key
  - Your IP addresses (`my_ip`, `other_ip`)
  - Database username and password
- You have AWS credentials with the necessary permissions
- You have enough AWS account limits for the resources you're creating

**Example workflow:**

```bash
# 1. Initialize Terraform (first time only, or after changes to providers)
terraform init

# 2. Review what will be created (optional, but recommended)
terraform plan

# 3. Apply the configuration to create resources
terraform apply
```

**Note**: The `terraform plan` command (optional) shows you what changes will be made without actually applying them. This is useful for reviewing changes before applying them.

---

## Terraform Configuration

The `terraform.tf` file is a special configuration file that tells Terraform which providers and versions to use. This file should be in your project root.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.92"
    }
  }

  required_version = ">= 1.2"
}
```

**What this file does:**

- **terraform block**: This is the main configuration block for Terraform itself.
- **required_providers**: Specifies which providers (plugins) Terraform needs to download and use.
  - **aws**: The AWS provider that allows Terraform to manage AWS resources.
  - **source**: `"hashicorp/aws"` - The official source of the AWS provider (from HashiCorp).
  - **version**: `"~> 5.92"` - This means Terraform will use version 5.92 or any newer version in the 5.92.x range, but not 6.0. This ensures compatibility while allowing minor updates.
- **required_version**: `">= 1.2"` - Specifies the minimum version of Terraform itself that is needed to run this configuration. This ensures everyone uses a compatible version.

**Why this is important**: This file ensures that everyone working on the project uses the same provider versions, which prevents errors and makes the infrastructure more reliable.

---

## AWS Provider Configuration

```hcl
provider "aws" {
  region     = "us-east-1"
  access_key = var.aws_access_key
  secret_key = var.aws_secret_key
}
```

This block initializes the connection to AWS. Here we configure:

- **region**: The AWS region where all resources will be created (e.g., `us-east-1`). This region will be used for all services like S3, RDS, and EC2.
- **access_key** and **secret_key**: Your AWS credentials stored in variables for security.

**Note**: You can create IAM users manually, but it's better to create them with Terraform. This way, you can manage permissions more easily. I spent a lot of time manually adding permissions one by one 😅. For more information on creating IAM users with Terraform, see the [official Terraform AWS IAM User documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_user).

---

## VPC Configuration

```hcl
data "aws_vpc" "default" {
  default = true
}
```

This tells Terraform to use your default VPC (Virtual Private Cloud). The VPC is like a private network in AWS where your resources will run.

---

## AMI Selection

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }

  owners = ["099720109477"]
}
```

This is a very important part of creating an EC2 instance. Here we select the newest version of **Ubuntu**.

- **most_recent = true**: Terraform automatically selects the newest AMI that matches the filter.
- **filter**: We search for AMIs by the `name` attribute. The `values` field contains the pattern of the Ubuntu image we want.
- **owners**: The ID of the AMI owner (`099720109477` is the official Ubuntu account). This ensures Terraform only picks **official Ubuntu AMIs** and ignores third-party images.

---

## SSH Key Pair

```hcl
resource "aws_key_pair" "ssh_key" {
  key_name   = "ssh_dev"
  public_key = file("~/.ssh/id_rsa_terraform.pub")
}
```

This resource creates an SSH key pair for connecting to your server. You can connect to the server using:

```bash
ssh -i "path_to_private_key" ubuntu@server_ip
```

**To generate an SSH key**, use:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
```

- **key_name**: A name to identify this key in AWS.
- **public_key**: The path to your public key file (the `.pub` file).

---

## Security Groups

Security Groups are very important in AWS. They act like virtual firewalls for your resources. With them, you can control access to EC2 instances, RDS databases, and other services by specifying which ports are open and which IPs or resources can access them.

### EC2 Security Group

```hcl
resource "aws_security_group" "ec2_sg" {
  name   = "ec2-sg"
  vpc_id = data.aws_vpc.default.id

  description = "Allow SSH from my IP, HTTP/HTTPS for all"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.my_ip, var.other_ip]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

**Configuration details:**

- **name**: `ec2-sg` - A name to identify this security group.
- **vpc_id**: Links the security group to the VPC we use.
- **description**: Explains the purpose of the security group.

**Ingress rules (incoming traffic):**

- **Port 22 (SSH)**: Allows SSH access only from your IPs (`var.my_ip`, `var.other_ip`).
- **Port 80 (HTTP)**: Allows HTTP traffic from anywhere (`0.0.0.0/0`).
- **Port 443 (HTTPS)**: Allows HTTPS traffic from anywhere.

**Egress rules (outgoing traffic):**

- All traffic is allowed to anywhere (ports 0-0, protocol -1).

### RDS Security Group

```hcl
resource "aws_security_group" "rds_sg" {
  name        = "rds-sg"
  vpc_id      = data.aws_vpc.default.id
  description = "Allow Postgres from EC2 only"

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.ec2_sg.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

**Configuration details:**

- **name**: `rds-sg` - Identifies the security group for the database.
- **vpc_id**: Uses the same VPC as the EC2 instance.
- **description**: Restricts access to the Postgres database.

**Ingress rules:**

- **Port 5432 (PostgreSQL)**: Allows database connections only from resources in the EC2 security group (`ec2_sg.id`). This means only your EC2 instance can connect to the database.

**Egress rules:**

- All outgoing traffic is allowed to anywhere.

---

## EC2 Instance

```hcl
resource "aws_instance" "app_server" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = "t3.micro"
  vpc_security_group_ids = [aws_security_group.ec2_sg.id]
  key_name               = aws_key_pair.ssh_key.key_name

  tags = {
    Name = "learn-terraform"
  }
}
```

This resource creates and configures an EC2 instance (a virtual server).

**Configuration details:**

- **ami**: Uses the Ubuntu AMI we selected from the `aws_ami` data source (the latest official version).
- **instance_type**: Specifies the type of EC2 instance (`t3.micro`), which defines CPU, memory, and performance.
- **vpc_security_group_ids**: Attaches the EC2 instance to the security group we created earlier (`ec2_sg`). This controls access via firewall rules.
- **key_name**: The SSH key we created. This is required to connect to the instance securely.
- **tags**: Metadata for the instance. In this case, `Name = "learn-terraform"` helps identify it in the AWS console.

**In summary**: This block creates a Linux server (EC2), attaches the proper firewall rules, and allows SSH access using the specified key.

---

## RDS Database Instance

```hcl
resource "aws_db_instance" "postgres" {
  allocated_storage      = 20
  engine                 = "postgres"
  engine_version         = "15.15"
  instance_class         = "db.t3.micro"
  db_name                = "your_db_name"
  username               = var.db_username
  password               = var.db_password
  parameter_group_name   = "default.postgres15"
  skip_final_snapshot    = true
  publicly_accessible    = false
  vpc_security_group_ids = [aws_security_group.rds_sg.id]
}
```

This resource creates a PostgreSQL database instance on AWS RDS.

**Configuration details:**

- **allocated_storage**: `20` - Defines the disk size for the database in GB.
- **engine**: `"postgres"` - The type of database (PostgreSQL).
- **engine_version**: `"15.15"` - Specifies the version of the database engine.
- **instance_class**: `"db.t3.micro"` - Chooses the compute resources (CPU/memory) for the database instance.
- **db_name**: `"your_name"` - The name of the initial database created.
- **username**: `var.db_username` - Admin username for connecting to the database (stored in a variable).
- **password**: `var.db_password` - Password for the admin user (stored in a variable for security).
- **parameter_group_name**: `"default.postgres15"` - Database parameters configuration (using default settings).
- **skip_final_snapshot**: `true` - Does not create a final snapshot when deleting the database. **Use with caution** - snapshots are useful for backups.
- **publicly_accessible**: `false` - The database is not accessible from the internet, only from resources inside the VPC.
- **vpc_security_group_ids**: `[aws_security_group.rds_sg.id]` - Attaches the RDS instance to the RDS security group, restricting which resources can connect (only your EC2 instance in this case).

---

## S3 Bucket

```hcl
resource "aws_s3_bucket" "spacelift_dev_study_s3" {
  bucket = "spacelift-dev-study-s3"
}
```

This resource creates an S3 bucket. S3 (Simple Storage Service) is AWS's object storage service where you can store files, backups, static website content, and more.

**Configuration details:**

- **resource**: `"aws_s3_bucket"` - The type of resource we're creating (an S3 bucket).
- **spacelift_dev_study_s3**: A local name for this resource in Terraform (you can reference it later as `aws_s3_bucket.spacelift_dev_study_s3`).
- **bucket**: `"spacelift-dev-study-s3"` - The actual name of the bucket in AWS. This name must be globally unique across all AWS accounts.

**Important notes:**

- S3 bucket names must be unique worldwide - no two AWS accounts can have a bucket with the same name.
- Bucket names can only contain lowercase letters, numbers, hyphens, and dots.
- Once created, you can use this bucket to store files, host static websites, or use it for backups.

**Common use cases for S3 buckets:**

- Storing application files and assets
- Backing up databases
- Hosting static websites
- Storing logs and data files
- Serving as a storage backend for applications

---

## Summary

This Terraform configuration creates:

1. An EC2 instance running Ubuntu that can be accessed via SSH
2. A PostgreSQL database that is only accessible from the EC2 instance
3. Security groups that control network access to both resources
4. An S3 bucket for storing files and data
5. All resources are created in the default VPC in the specified AWS region

Remember to set up your variables file (`variables.tf` and `terraform.tfvars`) with your AWS credentials, IP addresses, and database credentials before running `terraform apply`.

---

## Tips and Additional Commands

### terraform fmt

Before committing your Terraform code, it's a good practice to format it. The `terraform fmt` command automatically formats your Terraform configuration files to a consistent style.

```bash
terraform fmt
```

**What this command does:**

- Formats all `.tf` files in the current directory
- Makes your code more readable and consistent
- Follows Terraform's standard formatting conventions
- Can be run on specific files: `terraform fmt main.tf`

**When to use it:**

- Before committing code to version control
- After making changes to your Terraform files
- To ensure consistent code style across your project

**Note**: This command only formats files and doesn't make any changes to your infrastructure. It's safe to run anytime.

### About terraform init

The `terraform init` command can be confusing at first, especially if you're new to Terraform. Here are some important points to remember:

- **You must run it first**: Before any other Terraform command (`plan`, `apply`, `destroy`), you need to run `terraform init`
- **It downloads providers**: This command downloads the AWS provider (and other providers) that your configuration needs
- **It's not destructive**: Running `terraform init` doesn't create or change any AWS resources - it only prepares your local environment
- **Run it again when needed**: You should run it again if you add new providers or change provider versions in your `terraform.tf` file

If you see errors about missing providers or plugins, running `terraform init` usually fixes them.
