## ses-receiving-terraform

> This document provides coding agents with essential information about this Terraform-based AWS SES inbound email pipeline project.

# Agent Guidelines for SES Receiving Terraform

This document provides coding agents with essential information about this Terraform-based AWS SES inbound email pipeline project.

## Project Overview

A Terraform configuration that provisions an AWS SES inbound email pipeline with S3 storage, SNS/SQS event processing, and Lambda-based email organization.

**Tech Stack:**

- Terraform (>= 1.6.0)
- AWS Provider (>= 5.50)
- Python 3.12 (Lambda runtime)
- AWS Services: SES, S3, SNS, SQS, Lambda, IAM

## Build, Test, and Deployment Commands

### Terraform Commands

```bash
# Initialize Terraform (download providers, set up backend)
terraform init

# Validate configuration syntax
terraform validate

# Format all .tf files to canonical style
terraform fmt

# Check formatting without making changes
terraform fmt -check

# Show execution plan
terraform plan

# Apply changes
terraform apply

# Apply with auto-approval (use with caution)
terraform apply -auto-approve

# Destroy all resources
terraform destroy

# Show current outputs
terraform output

# Show specific output
terraform output mx_record

# Run tflint to check for errors and best practices
tflint

# Initialize tflint (download plugins)
tflint --init
```

### TFLint

TFLint is a Terraform linter that checks for errors, best practices, and potential issues.

**Important:** Always run `tflint` after modifying any `.tf` file before committing or applying changes.

```bash
# Initialize tflint (required first time and after config changes)
tflint --init

# Run tflint on current directory
tflint

# Run with specific format
tflint --format compact

# Run recursively on all modules
tflint --recursive
```

### Lambda Testing

```bash
# Package Lambda function manually
cd lambda && zip -r ../lambda.zip . && cd ..

# Test Lambda locally (requires AWS SAM CLI or custom test harness)
# Note: This project doesn't include unit tests - consider adding them

# Invoke deployed Lambda function with test event
aws lambda invoke \
  --function-name ses-receive-app-markcallen-dev-move \
  --payload '{"Records":[{"ses":{"mail":{"messageId":"test123"},"receipt":{"recipients":["test@example.com"]}}}]}' \
  response.json
```

### AWS CLI Helpers

```bash
# Check SES receipt rule set status
aws ses describe-receipt-rule-set --rule-set-name ses-receive-app-markcallen-dev-rule-set

# List S3 bucket contents
aws s3 ls s3://ses-inbound-app-markcallen-dev/ --recursive

# View Lambda logs
aws logs tail /aws/lambda/ses-receive-app-markcallen-dev-move --follow

# Read SQS queue
aws sqs receive-message --queue-url <queue-url>
```

## Code Style Guidelines

### Terraform

**File Organization:**

- `main.tf` - Primary resource definitions
- `variables.tf` - Input variable declarations
- `outputs.tf` - Output value definitions
- Keep related resources grouped logically in main.tf

**Formatting:**

- Use `terraform fmt` before committing (2-space indentation)
- Run `tflint` after modifying any `.tf` file to check for errors and best practices
- One resource per block
- Blank line between resources
- Use snake_case for resource names and variables

**Resource Naming:**

- Pattern: `${var.project_tag}-<descriptor>`
- Examples: `ses-receive-app-markcallen-dev-lambda-role`, `ses-receive-app-markcallen-dev-queue`
- Keep names descriptive and consistent

**Variables:**

- Always include `description` and `type`
- Provide sensible defaults where appropriate
- Use descriptive variable names (e.g., `subdomain_fqdn`, not `domain`)

**Best Practices:**

- Use `depends_on` explicitly when resource dependencies aren't implicit
- Tag all resources with `{ Project = var.project_tag }`
- Use `jsonencode()` for IAM policies and complex JSON structures
- Use data sources (e.g., `data.aws_caller_identity`) instead of hardcoding account info
- Enable encryption, versioning, and public access blocks on S3 buckets

### Python (Lambda)

**Style:**

- Follow PEP 8 conventions
- 4-space indentation
- Use double quotes for strings
- Keep handler functions simple and focused

**Imports:**

- Standard library first (`import os`)
- Third-party libraries second (`import boto3`)
- Blank line between groups

**Naming:**

- snake_case for variables and functions
- UPPER_CASE for constants/environment variables
- Example: `BUCKET`, `INCOMING_PREFIX`, `message_id`

**Error Handling:**

- Lambda should handle exceptions gracefully
- Consider adding try/except blocks for S3 operations
- Return structured responses: `{"ok": True, ...}`

**Environment Variables:**

- Use `os.environ` for required vars
- Use `os.environ.get()` with defaults for optional vars
- Example: `BUCKET = os.environ["BUCKET"]`

**AWS SDK Usage:**

- Initialize boto3 clients at module level (outside handler)
- Use explicit parameter names in boto3 calls
- Example: `s3.copy_object(Bucket=BUCKET, Key=dest_key, CopySource=src)`

## Common Workflows

### Adding a New AWS Resource

1. Add resource block to `main.tf` in appropriate section
2. Add any required IAM permissions
3. Tag with `{ Project = var.project_tag }`
4. Run `terraform fmt` and `terraform validate`
5. Run `tflint` to check for errors and best practices
6. Run `terraform plan` to review changes
7. Add output to `outputs.tf` if resource info needs to be exposed

### Modifying Lambda Function

1. Edit `lambda/handler.py`
2. Test locally if possible or use safe test data
3. Run `terraform apply` (this will automatically re-zip and deploy)
4. Monitor CloudWatch logs for errors: `aws logs tail /aws/lambda/<function-name> --follow`

### Updating Variables

1. Modify `terraform.tfvars` (not tracked in git) or pass via CLI
2. Never hardcode account-specific values in .tf files
3. Use variables for anything that might change between environments

## Important Notes

- **Not a Git Repository:** This directory is not version-controlled. Consider initializing git.
- **No Automated Tests:** Lambda function lacks unit tests. Consider adding pytest-based tests.
- **State Management:** Terraform state is local. Consider using remote backend (S3 + DynamoDB) for team environments.
- **SES Sandbox:** Newly created AWS accounts have SES in sandbox mode - you can only send/receive from verified addresses.
- **S3 Cleanup:** The bucket is not configured with `force_destroy = true`, so destroy will fail if bucket contains objects.

## Security Best Practices

- Never commit `terraform.tfvars` or `.tfstate` files
- Use least-privilege IAM policies
- Keep AWS credentials out of code (use AWS CLI profiles or environment variables)
- Enable S3 bucket encryption (already configured with AES256)
- Use TLS for data in transit (SES rule uses `tls_policy = "Optional"` - consider "Require")
- Review IAM policies before applying

## Troubleshooting

**Terraform apply fails:**

- Check AWS credentials: `aws sts get-caller-identity`
- Ensure region supports SES receiving (us-east-1, us-east-2, us-west-2, eu-west-1)
- Verify unique bucket name globally

**Emails not arriving:**

- Verify MX record in DNS
- Check SES receipt rule is active: `aws ses describe-active-receipt-rule-set`
- Activate rule set: `aws ses set-active-receipt-rule-set --rule-set-name <name>`
- Verify domain in SES console

**Lambda not moving emails:**

- Check CloudWatch logs
- Verify Lambda has S3 permissions
- Ensure SES receipt rule has Lambda action enabled

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/markcallen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/markcallen)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
