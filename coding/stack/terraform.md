# Terraform Patterns

## Module Structure

```
modules/
└── s3-bucket/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── README.md
```

## Variables

```hcl
variable "environment" {
  description = "Environment name (dev, staging, prod)"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "tags" {
  description = "Common tags for all resources"
  type        = map(string)
  default     = {}
}
```

## Outputs

```hcl
output "bucket_arn" {
  description = "ARN of the S3 bucket"
  value       = aws_s3_bucket.main.arn
}

output "bucket_name" {
  description = "Name of the S3 bucket"
  value       = aws_s3_bucket.main.id
}
```

## State Management

### Remote State (S3 + DynamoDB)
```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "project/environment/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

### State Locking
Always use DynamoDB for state locking in team environments.

## AWS Provider Patterns

### Provider Configuration
```hcl
provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "terraform"
      Project     = var.project_name
    }
  }
}
```

### Data Sources
```hcl
data "aws_caller_identity" "current" {}
data "aws_region" "current" {}

locals {
  account_id = data.aws_caller_identity.current.account_id
  region     = data.aws_region.current.name
}
```

## Best Practices

1. **Use modules** for reusable infrastructure
2. **Pin provider versions** in `versions.tf`
3. **Use `terraform fmt`** before committing
4. **Use `terraform validate`** in CI/CD
5. **Never store secrets** in state — use AWS Secrets Manager
6. **Use workspaces** or separate state files per environment
