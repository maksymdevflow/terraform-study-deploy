# Terraform Learning Roadmap

## Phase 1: Basic Terraform Understanding

### Theory
- [ ] Understand what IaC (Infrastructure as Code) is and why Terraform
- [ ] Learn how providers work
- [ ] Understand what resources are
- [ ] Learn about variables
- [ ] Learn about outputs
- [ ] Understand commands: `terraform init`, `terraform plan`, `terraform apply`, `terraform destroy`

### Practice
- [ ] Create 1 EC2 instance
- [ ] Create 1 S3 bucket
- [ ] Delete created resources using `terraform destroy`

**Status**: ✅ Mostly done (EC2 and S3 already exist)

---

## Phase 2: Structure and Best Practices

### Theory
- [ ] Learn to organize code into separate files:
  - [ ] `main.tf` - main resources
  - [ ] `variables.tf` - variables
  - [ ] `outputs.tf` - outputs
  - [ ] `providers.tf` or `terraform.tf` - provider configuration
- [ ] Understand `.tfvars` files
- [ ] Learn about workspaces
- [ ] Learn basic VPC structure (subnets, routes)

### Practice
- [ ] Reorganize current project:
  - [ ] Create `variables.tf`
  - [ ] Create `outputs.tf`
  - [ ] Create `terraform.tfvars.example`
- [ ] Create your first VPC with:
  - [ ] 1 public subnet
  - [ ] 1 private subnet
  - [ ] Internet Gateway
  - [ ] Route Table

---

## Phase 3: AWS, Networking, RDS, EC2

### Theory
- [ ] Deep dive into Security Groups
- [ ] Study EC2 in detail:
  - [ ] AMI selection
  - [ ] Key pairs
  - [ ] Instance types
- [ ] Learn RDS basics (PostgreSQL/MySQL)
- [ ] Optional: ASG + Launch Template

### Practice
- [ ] Create EC2 in private subnet
- [ ] Connect NAT Gateway (so EC2 has internet access)
- [ ] Create RDS in private zone
- [ ] Configure Security Groups so EC2 can connect to RDS

**Status**: 🟡 Partially done (EC2, RDS, Security Groups exist)

---

## Phase 4: Modules

### Theory
- [ ] Understand what modules are in Terraform
- [ ] Learn module structure
- [ ] Learn to create custom modules

### Practice
- [ ] Create `modules/vpc/` module
- [ ] Create `modules/ec2/` module
- [ ] Create `modules/rds/` module
- [ ] Break down current infrastructure into modules
- [ ] Reuse modules in a new project
- [ ] Extract variables and outputs from modules

---

## Phase 5: Terraform Cloud + S3 Backend

### Theory
- [ ] Learn about remote S3 backend
- [ ] Understand locking via DynamoDB
- [ ] Learn Terraform Cloud workspaces
- [ ] Learn GitHub Actions integration
- [ ] Understand `.terraform.lock.hcl`

### Practice
- [ ] Set up S3 backend for Terraform state
- [ ] Set up DynamoDB for state locking
- [ ] Create Terraform Cloud workspace
- [ ] Set up GitHub Actions for:
  - [ ] Automatic `terraform plan` on PR
  - [ ] Manual approval for `terraform apply`
- [ ] Add `.terraform.lock.hcl` to repository

---

## Phase 6: IAM and Security

### Theory
- [ ] Learn IAM policies
- [ ] Learn IAM roles
- [ ] Understand least privilege principle
- [ ] Learn EC2 instance roles instead of secrets
- [ ] Learn GitHub Actions OIDC → AWS integration

### Practice
- [ ] Create IAM role for EC2
- [ ] Attach IAM role to EC2 instance
- [ ] Remove AWS keys usage from variables
- [ ] Set up GitHub Actions with OIDC for secure AWS access
- [ ] Create IAM policies with least privilege principle

---

## Additional Goals (Optional)

### Portfolio and Documentation
- [ ] Create "senior-level Terraform template" for portfolio
- [ ] Add project to GitHub as portfolio
- [ ] Write detailed project documentation
- [ ] Add module usage examples

### Advanced Topics
- [ ] Terraform Cloud/Enterprise
- [ ] Terraform testing (terratest)
- [ ] Multi-cloud deployments
- [ ] Terraform best practices and code review

---

## Useful Resources

- [ ] HashiCorp Learn - Terraform tutorials
- [ ] AWS Terraform Provider documentation
- [ ] Terraform Registry for ready-made modules

---

## Important Concepts to Review

- [ ] State management
- [ ] Workspaces vs environments
- [ ] Variable precedence
- [ ] Data sources
- [ ] Locals
- [ ] Conditional expressions

---

## Completion Checklist

- [ ] All practical tasks completed
- [ ] Project structured with modules
- [ ] Remote backend configured
- [ ] IAM roles instead of secrets
- [ ] CI/CD pipeline set up
- [ ] Project added to portfolio
