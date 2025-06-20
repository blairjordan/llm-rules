# Terraform Development Rules

## Overview & Philosophy

- **Modular Architecture**: Clear separation of concerns with focused, reusable modules
- **State Management First**: Proper remote state setup with locking before any infrastructure
- **Security by Default**: Explicit resource configurations with security best practices
- **Environment Consistency**: Infrastructure as code with version control and reproducible deployments
- **Documentation-Driven**: Self-documenting configuration with clear variable descriptions

## Structure & Organization

### Project Architecture

- `main.tf` - Root module configuration with provider and module instantiation
- `variables.tf` - All input variables with comprehensive descriptions
- `versions.tf` - Provider and Terraform version constraints
- `backend.tf` - Remote state configuration (S3 + DynamoDB)
- `terraform.tfvars.example` - Example variable values (committed to VCS)
- `terraform.tfvars` - Actual variable values (excluded from VCS)
- `bootstrap/` - One-time state management infrastructure setup
- `modules/` - Reusable, focused modules for specific services
- `lambda/` - Serverless functions related to infrastructure (triggers, etc.)

### Module Structure

Each module follows consistent organization:

- `main.tf` - Resource definitions and core logic
- `variables.tf` - Input variables with types and descriptions
- `outputs.tf` - Return values for module consumers
- Single responsibility per module (cognito, s3, sqs, etc.)

### File Organization Principles

- Root level contains orchestration and configuration
- Modules contain implementation details
- Clear separation between infrastructure setup (bootstrap) and application infrastructure
- Lambda functions co-located when tightly coupled to infrastructure

## Naming Conventions

### Resources & Variables

- Resources use descriptive names that reflect their purpose
- Variables follow `snake_case` with comprehensive descriptions
- Output names match the resource attributes they expose
- Module names reflect the service they manage (`cognito`, `s3`, `sqs`)

### Tagging Strategy

- Consistent tagging across all resources:
  - `Name` - Resource identifier
  - `Environment` - Deployment environment
  - `Managed_By` - Always "terraform"
- Additional tags as needed for cost allocation and management

## Core Patterns & Best Practices

### Provider Configuration

- Explicit AWS provider configuration with region and profile
- Use named profiles for different environments/accounts
- Version constraints for providers to ensure consistency

```hcl
provider "aws" {
  region  = "ap-southeast-1"
  profile = "devmode"
}
```

### State Management

- **Bootstrap First**: Set up state infrastructure before application infrastructure
- Remote state with S3 backend and DynamoDB locking
- Separate bootstrap module for state management resources
- Enable encryption and versioning for state storage

```hcl
terraform {
  backend "s3" {
    bucket         = "project-terraform-state"
    key            = "project/terraform.tfstate"
    region         = "ap-southeast-1"
    profile        = "devmode"
    use_lockfile   = true
    dynamodb_table = "project-terraform-locks"
    encrypt        = true
  }
}
```

### Module Design

- **Single Responsibility**: Each module manages one service or logical grouping
- **Explicit Dependencies**: Pass ARNs and IDs between modules explicitly
- **Flexible Configuration**: Use variables for customizable settings
- **Complete Outputs**: Export all values that consumers might need

### Resource Configuration

- **Security First**: Enable encryption, restrict public access by default
- **Lifecycle Management**: Configure appropriate lifecycle rules
- **Force Destroy**: Use `force_destroy = true` for development environments
- **CORS Configuration**: Explicit CORS rules for S3 buckets with specific origins

### Variable Definitions

- Comprehensive descriptions for all variables
- Explicit type definitions
- Default values for optional configurations
- Logical grouping in variable files with comments

```hcl
# S3 Variables
variable "templates_bucket_name" {
  description = "Name of the S3 bucket for templates"
  type        = string
}
```

### Output Management

- **Descriptive Names**: Clear output names that indicate their purpose
- **Complete Information**: Include both names and ARNs where applicable
- **Legacy Compatibility**: Maintain backward compatibility when refactoring
- **Clear Descriptions**: Document what each output provides

### Inter-Module Communication

- Pass dependencies explicitly through variables
- Use module outputs for resource references
- Avoid hardcoded resource names or identifiers
- Clear dependency chains in module instantiation

## Error Handling & Quality

### Version Management

- **Lock Provider Versions**: Use `~>` constraints for predictable updates
- **Terraform Version**: Specify minimum required Terraform version
- **Lock File**: Commit `.terraform.lock.hcl` for consistent provider versions

### Configuration Validation

- **Type Safety**: Use proper variable types (string, list, map, etc.)
- **Validation Rules**: Add validation blocks for critical variables where appropriate
- **Required Variables**: Clearly distinguish required from optional variables

### Security Patterns

- **Least Privilege**: Configure minimal necessary permissions
- **Encryption**: Enable encryption at rest and in transit
- **Access Control**: Use IAM policies and resource-based policies appropriately
- **Public Access**: Block public access by default, enable only when needed

### Development Workflow

- **Plan Before Apply**: Always run `terraform plan` before `terraform apply`
- **Environment Separation**: Use different state files for different environments
- **Backup Strategy**: Ensure state backup and recovery procedures
- **Change Management**: Use version control for all configuration changes

## Documentation & Comments

### Configuration Documentation

- Comprehensive variable descriptions
- Clear module purposes and usage examples
- README files for complex modules or bootstrap procedures
- TODO tracking for technical debt and future improvements

### Inline Documentation

- Minimal inline comments - prefer self-documenting configuration
- Comments for complex resource interdependencies
- Explanation of non-obvious configuration choices
- Security consideration notes where relevant

## Maintenance Notes

- Bootstrap infrastructure must be set up before application infrastructure
- State management requires careful handling during team collaboration
- Provider updates should be tested in development before production
- Module refactoring may require state migration procedures
- Regular review of TODO items for infrastructure improvements
