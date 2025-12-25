# Quick Start Guide

## AWS Infrastructure Automation - Complete Setup

This guide will help you set up AWS infrastructure automation in **two simple steps**: one-time bootstrap and then use for all future projects.

---

## Step 1: One-Time Bootstrap (5 minutes)

You only need to do this **once per AWS account**.

### 1.1 Create Temporary AWS Credentials

1. Log into AWS Console
2. Go to IAM → Users → Create user
3. Give it **AdministratorAccess** (temporary, we'll delete this)
4. Create access key → Download credentials

### 1.2 Add Temporary Secrets to GitHub

1. Go to your repository **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Add these TWO secrets:

```
Name: AWS_ACCESS_KEY_ID
Value: <your-access-key-id>

Name: AWS_SECRET_ACCESS_KEY  
Value: <your-secret-access-key>
```

### 1.3 Run Bootstrap Workflow

1. Go to **Actions** tab
2. Select **AWS Bootstrap Setup**
3. Click **Run workflow**
4. Fill in the inputs:
   - **aws_account_id**: Your 12-digit AWS account ID
   - **github_org**: `Schulman-Coaching`
   - **github_repo**: `laughing-octo-waffle`
   - **aws_region**: `us-east-1` (or your preferred region)
5. Click **Run workflow**

### 1.4 Complete Setup

After the workflow completes:

1. **Copy the Role ARN** from the workflow output
2. Go to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add:

```
Name: AWS_ROLE_ARN
Value: arn:aws:iam::YOUR_ACCOUNT_ID:role/GitHubActions-Infrastructure-Role
```

5. **DELETE the temporary secrets:**
   - Delete `AWS_ACCESS_KEY_ID`
   - Delete `AWS_SECRET_ACCESS_KEY`
6. **Delete the temporary IAM user** from AWS Console

✅ **Bootstrap Complete!** You never need to do this again.

---

## Step 2: Create Infrastructure for Any Project

Now you can create AWS infrastructure for **any project** without manual setup!

### 2.1 Run Infrastructure Workflow

1. Go to **Actions** tab
2. Select **AWS Infrastructure Setup**
3. Click **Run workflow**
4. Fill in project details:
   - **project_name**: e.g., `my-app`
   - **environment**: `dev`, `staging`, or `prod`
   - **aws_region**: e.g., `us-east-1`
5. Click **Run workflow**

### 2.2 What Gets Created

The workflow automatically creates:

- ✅ VPC with DNS support
- ✅ Public subnets (2x across AZs)
- ✅ Private subnets (2x across AZs)
- ✅ Internet Gateway
- ✅ Route tables
- ✅ Security groups (HTTP/HTTPS)
- ✅ S3 bucket for Terraform state
- ✅ All resources properly tagged

### 2.3 Use the Infrastructure

After completion, the workflow displays:
- VPC ID
- Subnet IDs (public & private)
- Security Group ID

Use these IDs to deploy your applications!

---

## Summary

### One Time:
1. Create temporary AWS credentials
2. Add as GitHub secrets
3. Run **AWS Bootstrap Setup** workflow
4. Add `AWS_ROLE_ARN` secret
5. Delete temporary credentials

### Every Project:
1. Run **AWS Infrastructure Setup** workflow
2. Provide project details
3. Get infrastructure IDs
4. Deploy your app!

---

## Troubleshooting

### "Error assuming role"
- Make sure `AWS_ROLE_ARN` secret is set correctly
- Verify the role exists in AWS IAM

### "Bucket already exists"
- Another project is using that name
- Choose a different `project_name`

### "Access denied"
- The IAM role might be missing permissions
- Re-run the bootstrap workflow

---

## Security Features

✅ **OIDC Authentication** - No long-lived AWS credentials stored  
✅ **Repository-Specific** - Role only works for this repo  
✅ **Encrypted State** - Terraform state in encrypted S3  
✅ **Version Control** - State file versioning enabled  
✅ **Public Access Blocked** - S3 buckets secured  

---

For detailed documentation, see [AWS-INFRASTRUCTURE-SETUP.md](./AWS-INFRASTRUCTURE-SETUP.md)
