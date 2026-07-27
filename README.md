terraform-project/
├── terragrunt.hcl          ← ROOT (shared backend config)
├── modules/
│   └── vpc/
│       ├── main.tf         ← VPC module code
│       ├── variables.tf
│       └── outputs.tf
└── live/
    ├── terragrunt.hcl      ← live config
    ├── dev/
    │   └── vpc/
    │       └── terragrunt.hcl
    ├── staging/
    │   └── vpc/
    │       └── terragrunt.hcl
    └── production/
        └── vpc/
            └── terragrunt.hcl

            Use plain Terraform when:
──────────────────────────
When to use Terragrunt:

✅ Small project
✅ Single environment
✅ Learning Terraform
✅ Simple infrastructure

Use Terragrunt when:
─────────────────────
✅ Multiple environments
✅ Large teams
✅ Lots of modules
✅ Don't want code duplication
✅ Production-grade infrastructure



Terragrunt is not a replacement for Terraform it is a wrapper around Terraform (and OpenTofu) that helps you manage large,
multi-environment infrastructure with less duplication and better organization.

What is Terragrunt?

Terragrunt is an open-source tool developed by Gruntwork that provides extra functionality on top of Terraform.
Think of it like this:

You write Terraform code
        ↑
   Terragrunt manages it
        ↑
You run terragrunt commands

Instead of running:

terraform init
terraform plan
terraform apply

You run:

terragrunt init
terragrunt plan
terragrunt apply

Terragrunt invokes Terraform behind the scenes.
Why was Terragrunt created?

Imagine you have three environments:

Development
QA
Production

Without Terragrunt, your directory might look like this:
terraform/

dev/
    vpc/
    ec2/
    rds/

qa/
    vpc/
    ec2/
    rds/

prod/
    vpc/
    ec2/
    rds/
    
    Each folder contains nearly identical Terraform code, differing only in values like instance sizes or CIDR blocks. This leads to lots of duplicated code.
Terragrunt lets you keep a single reusable Terraform module and separate only the configuration for each environment.

Typical Project Structure
live/
│
├── dev/
│     ├── vpc/
│     ├── ec2/
│     └── rds/
│
├── qa/
│     ├── vpc/
│     ├── ec2/
│     └── rds/
│
└── prod/
      ├── vpc/
      ├── ec2/
      └── rds/

modules/
│
├── vpc/
├── ec2/
└── rds/
The modules/ directory contains reusable Terraform code. Each environment contains only Terragrunt configuration pointing to those modules.

Example

Terraform module:
modules/ec2

main.tf
variables.tf
outputs.tf

Terragrunt configuration:

terraform {
  source = "../../modules/ec2"
}

inputs = {
  instance_type = "t3.micro"
}

For production:

terraform {
  source = "../../modules/ec2"
}

inputs = {
  instance_type = "m5.large"
}

The Terraform code is identical; only the inputs change.
******************************************
Benefits of Terragrunt
1. DRY (Don't Repeat Yourself)

Without Terragrunt:

backend.tf
provider.tf
versions.tf

might be copied into every module.

With Terragrunt:

One configuration

↓

Shared across every module

No duplication.

2. Shared Remote State Configuration

Without Terragrunt:

Every module contains:

terraform {
  backend "s3" {
    bucket = "terraform-state"
    key    = "dev/vpc.tfstate"
    region = "us-east-1"
  }

  Repeated many times.

With Terragrunt:
remote_state {
  backend = "s3"

  config = {
    bucket = "terraform-state"
    region = "us-east-1"
  }
}
All child modules can inherit one configuration.
*******************************************
3. Automatic Dependency Management

   VPC

↓

EKS

↓

Applications

EKS needs the VPC ID.

Terragrunt allows:

dependency "vpc" {
  config_path = "../vpc"
}

Then:

inputs = {
  vpc_id = dependency.vpc.outputs.vpc_id
}

Terragrunt ensures the VPC is applied before EKS.

4. Apply Multiple Modules Together

Instead of:

cd vpc
terraform apply

cd ../ec2
terraform apply

cd ../rds
terraform apply

You can run:

terragrunt run-all apply

Terragrunt applies modules in dependency order.

5. Automatic Backend Creation

If your S3 bucket or DynamoDB table for state locking doesn't exist yet, Terragrunt can create them automatically.

Terraform alone expects the backend to already exist.

6. Inheritance

Parent configuration:

locals {
  region = "us-east-1"
}

Child:

include {
  path = find_in_parent_folders()
}

Every child inherits the shared configuration.

7. Environment-Specific Variables
dev

instance = t3.micro

qa

instance = t3.small

prod

instance = m5.large

The underlying Terraform module stays the same.

Real-World Example

Suppose you're deploying:

VPC
ALB
Auto Scaling Group
RDS
Redis

Without Terragrunt:

dev/
    backend.tf
    provider.tf
    versions.tf

qa/
    backend.tf
    provider.tf
    versions.tf

prod/
    backend.tf
    provider.tf
    versions.tf

A lot of repetition.

With Terragrunt:

live/

terragrunt.hcl

dev/

terragrunt.hcl

qa/

terragrunt.hcl

prod/

terragrunt.hcl

Each environment contains only the values that differ.
*****************************
When Should You Use Terragrunt?

It's especially useful when you have:

Multiple environments (dev, QA, prod)
Many Terraform modules
Shared backend configuration
Shared provider configuration
Module dependencies
Large teams managing infrastructure

For a small project with just one or two modules, plain Terraform is usually sufficient.

