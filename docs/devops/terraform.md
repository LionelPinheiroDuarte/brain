# Terraform

## What is Terraform?

**Terraform** is an open-source **Infrastructure as Code (IaC)** tool by HashiCorp. It lets you define infrastructure (servers, networks, DNS, etc.) in configuration files and manage it through a CLI.

Key properties:

- **Declarative** — you describe the desired state, Terraform figures out how to reach it
- **Idempotent** — running `apply` twice produces the same result
- **Provider-agnostic** — works with AWS, GCP, Azure, and hundreds of other APIs via **providers**

> ### 💡 **IaC vs manual provisioning**
>
> Instead of clicking in the AWS console, you write `.tf` files. The infrastructure becomes versionable, reproducible, and reviewable like code.

| Skill | Score |
|-------|:-----:|
| Explain what IaC is | 5 |
| Describe advantages of IaC patterns | 5 |
| Describe the concept of idempotency | 3 |

---

## File Structure

A standard Terraform project separates concerns across three files:

| File | Purpose |
|------|---------|
| `main.tf` | Core resources and data sources |
| `variables.tf` | Input variable declarations |
| `outputs.tf` | Values exposed after apply |
| `terraform.tfvars` | Actual variable values (never commit secrets) |

---

## Providers

A **provider** is a plugin that translates Terraform resources into API calls.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```

- Version constraints (`~> 5.0`) lock the minor version to avoid breaking changes
- The `.terraform.lock.hcl` file pins the exact resolved version

| Skill | Score |
|-------|:-----:|
| Differentiate Terraform vs other IaC tools | 4 |
| Explain multi-cloud and provider-agnostic benefits | 3 |
| Understand providers and provider configuration | 5 |

---

## Resources and Data Sources

**Resources** create and manage infrastructure:

```hcl
resource "aws_instance" "app" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
}
```

**Data sources** read existing infrastructure without managing it:

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical
}

data "aws_vpc" "default" {
  default = true
}
```

| Skill | Score |
|-------|:-----:|
| Understand resources and data sources | 5 |

---

## Variables and Outputs

**Variables** make configuration reusable:

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}
```

Values are provided via `terraform.tfvars`:

```hcl
aws_region = "eu-west-3"
my_ip      = "x.x.x.x/32"
```

**Outputs** expose values after apply:

```hcl
output "ssh_command" {
  value = "ssh ubuntu@${aws_instance.app.public_ip}"
}
```

| Skill | Score |
|-------|:-----:|
| Understand variables, outputs and locals | 4 |
| Understand `terraform.tfvars` and variable precedence | 5 |

---

## Core Workflow

```
terraform init      # Download providers and modules
terraform plan      # Preview changes
terraform apply     # Apply changes
terraform destroy   # Tear down infrastructure
```

- `init` must be run once per project (or after adding providers/modules)
- `plan` shows a diff: what will be created, changed, or destroyed
- State is stored in `terraform.tfstate` — never edit manually

| Skill | Score |
|-------|:-----:|
| Install and configure the Terraform CLI | 5 |
| Use `terraform init` | 5 |
| Use `terraform plan` | 5 |
| Use `terraform apply` | 5 |
| Use `terraform destroy` | 5 |
| Use `terraform fmt` and `terraform validate` | 2 |
| Use `terraform state` commands | 2 |
| Write → Plan → Apply lifecycle | 5 |
| Understand the dependency graph | 3 |

---

## Built-in Functions

```hcl
key_name = file(pathexpand("~/.ssh/id_rsa.pub"))
```

| Function | Description |
|----------|-------------|
| `file(path)` | Read a file's contents |
| `pathexpand(path)` | Expand `~` to home directory |
| `toset(list)` | Convert a list to a set (used with `for_each`) |
| `length(list)` | Count elements |
| `lookup(map, key)` | Safe map lookup with default |

| Skill | Score |
|-------|:-----:|
| Use built-in functions | 4 |
| Use `count` and `for_each` | 2 |
| Use `dynamic` blocks | 2 |

---

## State Management

**State** is how Terraform tracks what it has created. It is stored in `terraform.tfstate`.

- By default, state is **local** — stored on your machine
- In teams or CI/CD, use a **remote backend** (e.g. S3 + DynamoDB for locking)

```hcl
terraform {
  backend "s3" {
    bucket         = "my-tf-state"
    key            = "prod/terraform.tfstate"
    region         = "eu-west-3"
    dynamodb_table = "tf-lock"
  }
}
```

| Skill | Score |
|-------|:-----:|
| Understand the purpose of Terraform state | 4 |
| Configure remote backends | 2 |
| Handle state locking | 2 |
| Use `terraform import` | 2 |

---

## Modules

A **module** is a reusable group of resources. The **Terraform Registry** hosts public modules.

```hcl
module "ec2" {
  source        = "terraform-aws-modules/ec2-instance/aws"
  version       = "~> 5.0"
  instance_type = "t3.micro"
  ami           = data.aws_ami.ubuntu.id
}
```

| Skill | Score |
|-------|:-----:|
| Use public modules from the Terraform Registry | 2 |
| Understand module inputs and outputs | 2 |
| Create and call a local module | 2 |

---

## Terraform Cloud

**Terraform Cloud** is HashiCorp's managed platform for running Terraform remotely with collaboration features (remote state, run history, access control).

| Skill | Score |
|-------|:-----:|
| Describe Terraform Cloud features | 2 |
| Understand workspaces in Terraform Cloud | 2 |

---

### 🔑 **Important Keywords:**
`IaC`, `provider`, `resource`, `data source`, `state`, `backend`, `module`, `variables`, `outputs`, `tfvars`, `idempotent`, `declarative`, `HCL`, `terraform init`, `terraform apply`
