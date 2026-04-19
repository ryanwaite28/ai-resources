# Terraform Project Structure — Best Practices (with Cross-Module Data Sharing)

Source: [LinkedIn Post](https://www.linkedin.com/posts/alafonye-emmanuel_most-terraform-projects-fail-not-because-share-7451674800994848770-pa8b?utm_source=share&utm_medium=member_desktop&rcm=ACoAABluCTwB44tkeI2ie_AH9-HdTgMDEoJ5ys0)

## Overview

This structure organizes Terraform code into four logical layers to maximize scalability, reusability, governance, and **safe data sharing across modules and environments**:

* **Environments** → Environment-specific configurations (dev, prod)
* **Modules** → Reusable infrastructure components
* **Policies** → Governance, compliance, and security rules
* **Scripts** → Automation for Terraform workflows
* **Shared Data Patterns** → Controlled ways to pass outputs between modules and stacks

---

## 📁 Project Layout

```
your-terraform-project/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   └── prod/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       ├── terraform.tfvars
│       └── backend.tf
│
├── modules/
│   ├── networking/
│   ├── compute/
│   ├── database/
│   ├── security/
│   └── monitoring/
│
├── policies/
│   ├── sentinel/
│   └── opa/
│
├── scripts/
│
└── shared-data/              # (Optional) conventions & helpers
    ├── parameter-store.md
    └── naming-conventions.md
```

---

## 🌍 Environments Layer

### Purpose

Defines environment-specific infrastructure and orchestrates modules.

### Files

* **main.tf** → Calls modules with environment-specific configuration
* **variables.tf** → Declares variables
* **outputs.tf** → Exports values
* **terraform.tfvars** → Environment values (**no secrets in VCS**)
* **backend.tf** → Remote state config

### Rules

* One folder per environment
* Separate state per environment
* No hardcoded values
* Keep logic inside modules

---

## 🧩 Modules Layer

### Purpose

Reusable infrastructure building blocks.

### Standard Structure

```
module-name/
├── main.tf
├── variables.tf
├── outputs.tf
├── versions.tf
└── README.md
```

### Guidelines

* Single responsibility
* No environment-specific logic
* Well-defined inputs/outputs
* Outputs must be stable and predictable (used by other modules)

---

## 🔗 Cross-Module Data Sharing (CRITICAL)

Terraform modules **should not directly depend on each other’s internal resources**. Instead, use controlled data-sharing patterns:

---

### Pattern 1: Direct Output Passing (Same Root Module)

**Best for:** tightly coupled modules within the same environment

```
module "networking" {
  source = "../../modules/networking"
}

module "compute" {
  source = "../../modules/compute"

  vpc_id = module.networking.vpc_id
}
```

✔ Simple
✔ Native Terraform
❌ Not usable across separate states

---

### Pattern 2: Remote State Data Source

**Best for:** sharing data across environments or separate Terraform states

```
data "terraform_remote_state" "networking" {
  backend = "s3"

  config = {
    bucket = "my-terraform-state"
    key    = "networking/terraform.tfstate"
    region = "us-east-1"
  }
}

module "compute" {
  source = "../../modules/compute"

  vpc_id = data.terraform_remote_state.networking.outputs.vpc_id
}
```

✔ Works across stacks
❌ Tight coupling to state structure
❌ Requires backend access permissions

---

### Pattern 3: Parameter Store (Recommended for AWS)

**Best for:** decoupled, scalable, cross-module and cross-stack sharing

Uses Amazon Web Services **SSM Parameter Store**.

---

#### Step 1: Write Outputs to Parameter Store

Inside a module (e.g., networking):

```
resource "aws_ssm_parameter" "vpc_id" {
  name  = "/project/dev/networking/vpc_id"
  type  = "String"
  value = aws_vpc.main.id
}
```

---

#### Step 2: Read from Parameter Store in Another Module

```
data "aws_ssm_parameter" "vpc_id" {
  name = "/project/dev/networking/vpc_id"
}

module "compute" {
  source = "../../modules/compute"

  vpc_id = data.aws_ssm_parameter.vpc_id.value
}
```

---

### Naming Convention for Parameters

```
/<project>/<environment>/<module>/<output>
```

**Example:**

```
/myapp/prod/networking/vpc_id
/myapp/dev/database/db_endpoint
```

---

### Benefits of Parameter Store Approach

✔ Decouples modules completely
✔ No direct state access required
✔ Works across accounts (with IAM)
✔ Easier rotation and updates
✔ Centralized configuration

---

### When to Use What

| Scenario                                 | Recommended Pattern |
| ---------------------------------------- | ------------------- |
| Same root module                         | Direct outputs      |
| Separate Terraform states                | Remote state        |
| Large-scale / multi-team / multi-account | Parameter Store     |

---

## 🔐 Secrets & Sensitive Data

* Use **SecureString** in Parameter Store for secrets
* Or integrate with secrets managers
* Never expose sensitive outputs in plain Terraform outputs

Example:

```
resource "aws_ssm_parameter" "db_password" {
  name  = "/project/prod/database/password"
  type  = "SecureString"
  value = var.db_password
}
```

---

## 🛡️ Policies Layer

### Purpose

Governance and compliance enforcement.

### Sentinel

* Cost control
* Security rules
* Naming standards

### OPA

* Tag validation
* Network policies
* Compliance checks

---

## ⚙️ Scripts Layer

### Purpose

Standardizes Terraform execution.

### Scripts

* init.sh
* validate.sh
* plan.sh
* apply.sh
* destroy.sh

### Workflow

```
./scripts/init.sh
./scripts/validate.sh
./scripts/plan.sh
./scripts/apply.sh
```

---

## ✅ Best Practices

### State

* Use remote backends (S3, Azure, GCS)
* Enable locking

### Versioning

* Pin Terraform & providers

### Modularity

* Reuse modules
* Avoid duplication

### Isolation

* Separate state per environment

### Secrets

* Use Parameter Store / Vault
* Never hardcode

### Naming

* Consistent patterns across:

  * resources
  * parameters
  * modules

### Tagging

* Environment
* Owner
* Cost Center

### Documentation

* Maintain README files

### Backups

* Enable state versioning

### Plan First

* Always review before apply

### Least Privilege

* Minimal IAM access

---

## 🔁 Workflow Summary

1. Build reusable modules
2. Define outputs clearly
3. Publish shared values (Parameter Store preferred)
4. Reference shared values in dependent modules
5. Validate configuration
6. Plan changes
7. Apply safely

---

## 📌 Key Principles

* Loose coupling between modules
* Strong contracts via outputs
* Centralized shared configuration
* Security-first data sharing
* Scalable multi-environment design

---
