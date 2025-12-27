# Terraform AWS CloudFormation Foundation

Shared backend infrastructure for Terraform and OpenTofu projects using CloudFormation for bootstrapping.

## Table of Contents

- [Why](#why)
- [How](#how)
- [What is Changed](#what-is-changed)
- [Quick Start](#quick-start)
- [AWS Well-Architected Framework](#aws-well-architected-framework)
- [Technologies Used](#technologies-used)
- [Copyright](#copyright)

## Why

Every Terraform or OpenTofu project needs three foundational components: remote state storage, state locking, and CI/CD authentication. Creating these manually for each project leads to inconsistency, security risks, and operational overhead. This project solves the bootstrapping problem by deploying shared backend infrastructure once, enabling all downstream projects to use consistent state management and secure OIDC-based authentication without long-lived credentials.

## How

The foundation uses CloudFormation to deploy infrastructure that Terraform projects depend on, avoiding the circular dependency of using Terraform to create Terraform backend resources. All configuration is published to SSM Parameter Store at predictable paths, enabling consuming projects to discover backend settings and OIDC providers without hardcoding values. The deployment script handles all complexity including prerequisite validation, metadata collection, parameter preparation, and stack deployment.

## What is Changed

### Resources Created

- S3 bucket for Terraform state storage with versioning and encryption
- S3 bucket for access logs with lifecycle policies
- DynamoDB table for state locking with point-in-time recovery
- OIDC provider for GitHub Actions authentication
- IAM role for terraform-aws-deployment-roles project
- SSM parameters publishing backend configuration and role ARNs
- CloudFormation stack managing all resources with consistent tagging

### Functional Changes

- Enables secure CI/CD authentication without long-lived credentials
- Provides centralized state management for multiple Terraform projects
- Establishes consistent resource naming and tagging patterns
- Creates foundation for project-specific IAM role deployment

## Quick Start

See [prerequisites](docs/prerequisites.md) for detailed requirements and [scripts directory](scripts/) for available operations.

```bash
git clone https://github.com/stephenabbot/terraform-aws-cfn-foundation.git
cd terraform-aws-cfn-foundation

# Configure environment
cp .env.example .env
# Edit .env with your settings

# Deploy foundation
./scripts/verify-prerequisites.sh
./scripts/deploy.sh

# Verify deployment
./scripts/list-deployed-resources.sh
```

## AWS Well-Architected Framework

This project demonstrates alignment with all six pillars of the [AWS Well-Architected Framework](https://aws.amazon.com/blogs/apn/the-6-pillars-of-the-aws-well-architected-framework/):

**Operational Excellence**: Automated deployment scripts with comprehensive error handling, idempotent operations, rollback handling, resource listing and status verification, git state validation, parameter store configuration distribution, comprehensive tagging, stack termination protection, and clear documentation.

**Security**: OIDC provider for secure CI/CD authentication without long-lived credentials, S3 bucket encryption with bucket key enabled, S3 public access blocked, IAM roles with least privilege, DynamoDB point-in-time recovery, resource-level tagging for governance, git repository validation, parameter store for secure configuration distribution, CloudFormation stack termination protection, and access logging for state bucket.

**Reliability**: S3 versioning for state recovery, DynamoDB with point-in-time recovery and deletion protection, multi-region OIDC provider support, comprehensive error handling in deployment scripts, orphaned resource detection and cleanup, stack rollback handling, resource retention policies, idempotent operations, import capability for orphaned resources, and automated prerequisite validation.

**Performance Efficiency**: S3 Intelligent Tiering for automatic optimization based on access patterns, DynamoDB on-demand billing with automatic scaling, CloudFormation for infrastructure as code, automated deployment scripts, parameter store for efficient configuration distribution, regional resource naming, and bucket key enabled for S3 encryption efficiency.

**Cost Optimization**: S3 Intelligent Tiering automatically moves objects to cheaper storage classes, DynamoDB on-demand billing eliminates provisioned capacity waste, S3 lifecycle policies for log retention, comprehensive tagging enables cost allocation and tracking, shared infrastructure reduces per-project overhead, OIDC eliminates costs of managing long-lived credentials, CloudFormation prevents resource drift and waste, and automated cleanup scripts prevent orphaned resource costs.

**Sustainability**: Serverless managed services reduce infrastructure overhead, S3 Intelligent Tiering reduces storage energy consumption, DynamoDB on-demand reduces idle resource consumption, automated operations reduce manual intervention, shared foundation reduces duplicate infrastructure across projects, CloudFormation prevents resource sprawl and waste, and lifecycle policies prevent indefinite data retention.

## Technologies Used

| Technology | Purpose |
|------------|---------|
| AWS CloudFormation | Infrastructure as code and stack management |
| AWS Systems Manager Parameter Store | Configuration distribution and service discovery |
| AWS Identity and Access Management OIDC Provider | Secure CI/CD authentication |
| AWS DynamoDB | Terraform state locking with point-in-time recovery |
| AWS S3 Intelligent Tiering | Automatic storage cost optimization |
| AWS S3 Server-Side Encryption | Data protection with bucket key efficiency |
| AWS S3 Versioning | State recovery and backup capabilities |
| AWS S3 Lifecycle Policies | Log retention and cost management |
| AWS IAM Roles | Least privilege access control |
| Bash Scripting | Deployment automation and validation |
| Git Repository Metadata | Resource naming and tagging automation |
| JSON Query (jq) | Data processing in deployment scripts |
| AWS CLI | Service interaction and authentication |
| OpenSSL | OIDC provider thumbprint calculation |

## Copyright

© 2025 Stephen Abbot - MIT License
