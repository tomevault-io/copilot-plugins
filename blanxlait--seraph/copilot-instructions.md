## seraph

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

SERAPH (Security Event Response & Autonomous Protection Handler) is a Security Lake Dashboard that provides monitoring and querying capabilities for AWS Security Lake data. It's a web application deployed to AWS using CDK.

## Architecture

```
Frontend (React/Vite)
        │
        ▼
CloudFront Distribution
    ├── S3 (static files)
    └── API Gateway → Lambda
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
  Security Lake     Glue Catalog      Athena
      API                              Queries
                         │
                         ▼
              Security Lake S3 Data
```

**Default configuration:**
- Region: us-east-1
- Security Lake: 7-day retention
- Sources: CloudTrail management events, Security Hub findings

**Account Structure:**
| Account | ID | Purpose |
|---------|-----|---------|
| Log Archive | 779315395440 | Security Lake data |
| Security | 429971481640 | SERAPH dashboard (cross-account) |

## Common Commands

```bash
# Install dependencies
uv pip install -e ".[dev,cdk]"
cd frontend && npm install

# Deploy infrastructure
cd infrastructure
AWS_PROFILE=logarchive-admin uv run cdk deploy seraph-web

# Deploy frontend
cd frontend
npm run build
AWS_PROFILE=logarchive-admin aws s3 sync dist/ s3://<BUCKET_NAME> --delete --region us-east-1

# Run tests
python -m pytest tests/ -v

# Lint
ruff check .
cd frontend && npm run lint
```

## Key Components

### CDK Stack (`infrastructure/stacks/web_stack.py`)

Single stack that creates:
- S3 bucket for static website
- CloudFront distribution
- API Gateway REST API
- Lambda function for API
- Athena results S3 bucket
- IAM permissions for Security Lake, Glue, Athena

### Lambda API (`lambdas/api/`)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/securitylake/status` | GET | Security Lake configuration |
| `/api/securitylake/sources` | GET | Configured log sources |
| `/api/securitylake/tables` | GET | Glue tables in Security Lake database |
| `/api/securitylake/queries` | GET | Available pre-canned queries |
| `/api/securitylake/query` | POST | Execute an Athena query |

### Frontend (`frontend/`)

React + Vite + Tailwind + React Query
- `SecurityLakeStatus` - Shows lake config and data sources
- `QueryRunner` - Dropdown to select and run queries

## CRITICAL: Lake Formation Permissions

**Security Lake uses AWS Lake Formation for access control. IAM permissions alone are NOT sufficient.**

After deploying with CDK, you MUST grant Lake Formation permissions manually:

```bash
# Get Lambda role ARN from CDK output or AWS Console
LAMBDA_ROLE_ARN="arn:aws:iam::<ACCOUNT_ID>:role/seraph-web-APIFunctionServiceRole..."

# Grant database DESCRIBE
aws lakeformation grant-permissions \
  --principal DataLakePrincipalIdentifier="$LAMBDA_ROLE_ARN" \
  --resource '{"Database":{"Name":"amazon_security_lake_glue_db_us_east_1"}}' \
  --permissions DESCRIBE \
  --region us-east-1

# Grant table SELECT and DESCRIBE
aws lakeformation grant-permissions \
  --principal DataLakePrincipalIdentifier="$LAMBDA_ROLE_ARN" \
  --resource '{"Table":{"DatabaseName":"amazon_security_lake_glue_db_us_east_1","TableWildcard":{}}}' \
  --permissions SELECT DESCRIBE \
  --region us-east-1
```

**Why can't this be automated in CDK?**
- `AWS::LakeFormation::PrincipalPermissions` requires the deploying role to be a Lake Formation admin
- Security Lake creates its database with specific Lake Formation permissions
- The CDK execution role is not a Lake Formation admin by default
- This is the AWS-recommended approach for same-account Security Lake access

**Symptoms if permissions are missing:**
- `/api/securitylake/tables` returns empty `[]`
- Queries fail with "Insufficient permissions to execute the query"
- Errors mentioning `lakeformation:GetDataAccess`

## Pre-canned Queries (`lambdas/api/queries.py`)

Based on AWS Security Lake documentation:
- `cloudtrail-event-count` - Event volume by day
- `unauthorized-attempts` - Access denied errors
- `iam-activity` - IAM API calls
- `failed-records` - Failed operations
- `sh-medium-severity` - Security Hub findings >= Medium
- `sh-products-count` - Findings by product
- `data-freshness` - Most recent event per source

## Testing

```bash
# Unit tests
python -m pytest tests/unit/ -v

# Test API locally (after deployment)
curl https://<API_URL>/api/securitylake/status
curl -X POST https://<API_URL>/api/securitylake/query \
  -H "Content-Type: application/json" \
  -d '{"queryId": "cloudtrail-event-count"}'
```

## Deployment Notes

### Full Deployment Checklist

1. **Deploy CDK stack**
   ```bash
   cd infrastructure
   AWS_PROFILE=logarchive-admin uv run cdk deploy seraph-web
   ```

2. **Grant Lake Formation permissions** (see above)

3. **Build and deploy frontend**
   ```bash
   cd frontend && npm run build
   AWS_PROFILE=logarchive-admin aws s3 sync dist/ s3://<BUCKET> --delete --region us-east-1
   AWS_PROFILE=logarchive-admin aws cloudfront create-invalidation --distribution-id <DIST_ID> --paths "/*" --region us-east-1
   ```

4. **Verify**
   - Open CloudFront URL
   - Check Security Lake status shows "enabled"
   - Check tables list is populated
   - Run a test query

### Cross-Account Deployment (Security Account)

For deploying to the Security account with data in Log Archive:

1. **Create Security Lake subscriber** (Log Archive Console)
   - Subscriber name: `seraph-dashboard`
   - Access type: Query access (Lake Formation)
   - Principal: `429971481640`

2. **Accept RAM share** (Security Account Console)

3. **Create resource link database**
   ```bash
   AWS_PROFILE=security-admin aws glue create-database \
     --database-input '{
       "Name": "seraph_security_lake",
       "TargetDatabase": {
         "CatalogId": "779315395440",
         "DatabaseName": "amazon_security_lake_glue_db_us_east_1"
       }
     }' \
     --region us-east-1
   ```

4. **Deploy CDK stack**
   ```bash
   cd infrastructure
   AWS_PROFILE=security-admin uv run cdk bootstrap aws://429971481640/us-east-1
   AWS_PROFILE=security-admin uv run cdk deploy seraph-web \
     -c security_lake_account_id=779315395440 \
     -c security_lake_database=seraph_security_lake
   ```

5. **Grant Lake Formation permissions**
   ```bash
   LAMBDA_ROLE_ARN="arn:aws:iam::429971481640:role/seraph-web-APIFunctionServiceRole..."

   AWS_PROFILE=security-admin aws lakeformation grant-permissions \
     --principal DataLakePrincipalIdentifier="$LAMBDA_ROLE_ARN" \
     --resource '{"Database":{"Name":"seraph_security_lake"}}' \
     --permissions DESCRIBE \
     --region us-east-1

   AWS_PROFILE=security-admin aws lakeformation grant-permissions \
     --principal DataLakePrincipalIdentifier="$LAMBDA_ROLE_ARN" \
     --resource '{"Table":{"DatabaseName":"seraph_security_lake","TableWildcard":{}}}' \
     --permissions SELECT DESCRIBE \
     --region us-east-1
   ```

6. **Deploy frontend**
   ```bash
   cd frontend && npm run build
   AWS_PROFILE=security-admin aws s3 sync dist/ s3://<BUCKET> --delete --region us-east-1
   AWS_PROFILE=security-admin aws cloudfront create-invalidation --distribution-id <DIST_ID> --paths "/*"
   ```

### Deployment Info

After deployment, note these values from CDK outputs:
- `WebsiteURL` - CloudFront distribution URL
- `APIEndpoint` - API Gateway URL
- `BucketName` - S3 bucket for frontend
- Lambda role ARN (for Lake Formation permissions)

These can be stored as GitHub secrets for CI/CD workflows.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BLANXLAIT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
