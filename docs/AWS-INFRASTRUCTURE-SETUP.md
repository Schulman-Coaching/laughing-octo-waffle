# AWS Infrastructure Setup Workflow

## Overview

This GitHub Actions workflow automates the creation of basic AWS infrastructure for new projects using Terraform. It sets up a complete VPC environment with public and private subnets, internet gateway, route tables, and security groups.

## Features

The workflow creates the following AWS resources:

- **VPC**: A virtual private cloud with DNS support enabled
- **Public Subnets**: 2 public subnets across different availability zones
- **Private Subnets**: 2 private subnets across different availability zones
- **Internet Gateway**: For public subnet internet access
- **Route Tables**: Configured for public subnet routing
- **Security Groups**: Default security group with HTTP/HTTPS access
- **Resource Tagging**: All resources are tagged with project name, environment, and management method

## Prerequisites

Before using this workflow, you need to set up the following:

### 1. AWS IAM Role for GitHub Actions

Set up OIDC authentication between GitHub and AWS:

```bash
# Create an IAM OIDC identity provider for GitHub
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --client-id-list sts.amazonaws.com \
  --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1
```

Create an IAM role with the following trust policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:YOUR_ORG/YOUR_REPO:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

Attach the following AWS managed policies to the role:
- `AmazonVPCFullAccess`
- `AmazonEC2FullAccess`
- Access to S3 for Terraform state (see next section)

### 2. Terraform State Backend

Create an S3 bucket to store Terraform state:

```bash
# Create S3 bucket for Terraform state
aws s3api create-bucket \
  --bucket YOUR_PROJECT_NAME-terraform-state \
  --region us-east-1

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket YOUR_PROJECT_NAME-terraform-state \
  --versioning-configuration Status=Enabled

# Enable encryption
aws s3api put-bucket-encryption \
  --bucket YOUR_PROJECT_NAME-terraform-state \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }]
  }'
```

### 3. GitHub Secrets

Add the following secret to your GitHub repository:

1. Go to your repository Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. Add:
   - **Name**: `AWS_ROLE_ARN`
   - **Value**: `arn:aws:iam::YOUR_ACCOUNT_ID:role/YOUR_ROLE_NAME`

## Usage

### Running the Workflow

1. Navigate to the **Actions** tab in your GitHub repository
2. Select **AWS Infrastructure Setup** from the workflows list
3. Click **Run workflow**
4. Fill in the required inputs:
   - **project_name**: Name for your project (used in resource naming)
   - **environment**: Choose from dev, staging, or prod
   - **aws_region**: AWS region (default: us-east-1)
5. Click **Run workflow** to start the process

### Workflow Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|----------|
| `project_name` | Project name used for resource naming | Yes | - |
| `environment` | Environment type (dev/staging/prod) | Yes | - |
| `aws_region` | AWS region for resources | Yes | us-east-1 |

## Infrastructure Details

### Network Configuration

- **VPC CIDR**: 10.0.0.0/16
- **Public Subnets**: 
  - Subnet 1: 10.0.0.0/24
  - Subnet 2: 10.0.1.0/24
- **Private Subnets**:
  - Subnet 1: 10.0.10.0/24
  - Subnet 2: 10.0.11.0/24

### Security Group Rules

**Ingress**:
- Port 443 (HTTPS) from 0.0.0.0/0
- Port 80 (HTTP) from 0.0.0.0/0

**Egress**:
- All traffic to 0.0.0.0/0

### Resource Naming Convention

All resources follow this naming pattern:
```
{project_name}-{environment}-{resource_type}
```

Example: `myapp-dev-vpc`, `myapp-prod-public-1`

## Outputs

After successful execution, the workflow displays:

- **vpc_id**: ID of the created VPC
- **public_subnet_ids**: List of public subnet IDs
- **private_subnet_ids**: List of private subnet IDs
- **security_group_id**: ID of the default security group

These outputs can be used for deploying applications or additional infrastructure.

## Customization

To customize the infrastructure:

1. Edit the Terraform configuration in the workflow file (`.github/workflows/aws-infrastructure.yml`)
2. Modify the `Create Terraform Configuration` step
3. Update resource definitions as needed

### Common Customizations

**Change VPC CIDR**:
```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.1.0.0/16"  # Change this
  # ...
}
```

**Add more subnets**:
```hcl
resource "aws_subnet" "public" {
  count = 3  # Change from 2 to 3
  # ...
}
```

**Modify security group rules**:
```hcl
ingress {
  from_port   = 22
  to_port     = 22
  protocol    = "tcp"
  cidr_blocks = ["YOUR_IP/32"]
}
```

## Troubleshooting

### Common Issues

1. **"Error assuming role"**
   - Verify AWS_ROLE_ARN secret is correctly set
   - Check IAM role trust policy allows your repository

2. **"Terraform state bucket not found"**
   - Create the S3 bucket before running the workflow
   - Ensure bucket name matches `{project_name}-terraform-state`

3. **"Insufficient permissions"**
   - Verify IAM role has necessary AWS permissions
   - Check policies attached to the role

4. **"Resource already exists"**
   - Another resource with the same name exists
   - Use a different project_name or delete existing resources

## Best Practices

1. **Use separate environments**: Always use different environment values (dev/staging/prod) for isolation
2. **Backup state files**: Enable S3 versioning for Terraform state bucket
3. **Review before apply**: Check the Terraform plan output before applying
4. **Tag resources**: The workflow automatically tags resources for easier management
5. **Cost monitoring**: Monitor AWS costs, especially in production environments

## Cleanup

To destroy infrastructure created by this workflow:

1. Manually run Terraform destroy:
```bash
terraform init
terraform destroy
```

Or create a separate workflow for destruction (recommended for safety).

## Security Considerations

- The workflow uses OIDC authentication (no long-lived credentials)
- All traffic rules should be reviewed and restricted as needed
- Consider using private subnets for application servers
- Enable VPC Flow Logs for network monitoring
- Review security group rules regularly

## Support

For issues or questions:
1. Check the Actions tab for workflow logs
2. Review AWS CloudWatch logs
3. Consult Terraform documentation
4. Open an issue in this repository

## License

This workflow is part of the repository and follows the same license.
