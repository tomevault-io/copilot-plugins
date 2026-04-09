## iam-hub

> This is a Pulumi-based AWS infrastructure project that implements comprehensive budget control and cost management for team members with automated enforcement capabilities. The system uses a group-based IAM architecture with region restrictions, strict cost controls, and sophisticated tagging policies.

# GM2 Budget Control Infrastructure - Claude Reference

## Project Overview

This is a Pulumi-based AWS infrastructure project that implements comprehensive budget control and cost management for team members with automated enforcement capabilities. The system uses a group-based IAM architecture with region restrictions, strict cost controls, and sophisticated tagging policies.

## Major Architecture Improvements 🚀

### Advanced Group-Based IAM System (`src/iam/`)

- **Service Groups**: AdminUsers, IamRestrictions, InstancesRestrictions, BillingReadOnlyAccessUsers
- **AWS Managed Policies**: Uses production-ready AWS managed policies for admin and billing access
- **Per-User Region Restrictions**: Users can have custom region restrictions via `regions` parameter
- **Service-Specific Cost Controls**: Instance type restrictions, volume limits, and IAM operation blocks
- **Billing Transparency**: All users get read-only billing access for cost awareness

### Enhanced Budget Control System (`src/budget/`)

- **`control.ts`**: Creates IAM users with group memberships, password policies, and access provisioning
- **`enforcement.ts`**: SNS topic ready for budget alerts (Lambda enforcement disabled for iteration)
- **User-Friendly Password Policy**: 8+ characters, mixed case + numbers, symbols optional

### Legacy Policies (`src/policies/` - Deprecated)

- **`service.ts`**: Least-privilege service permissions (REPLACED by group system)
- **`cost.ts`**: Cost control policies (REPLACED by group policies)

### GitHub OIDC Identity Provider (`src/identity-provider/`)

- **`github-oidc.ts`**: Creates IAM OIDC provider and roles for GitHub Actions authentication
- **`types.ts`**: TypeScript interfaces with Pulumi Output types for OIDC configuration
- **`config.ts`**: GitHub organization and repository role configurations
- **Trust Policies**: OIDC federation with repository-level access control
- **Policy Support**: Both AWS managed and custom managed policies (ARN auto-detection)

### Core Configuration

- **`src/members.ts`**: Enhanced team member definitions with service arrays and access options
- **`src/constants.ts`**: Global constants with strict regional and instance type restrictions
- **`src/dashboard.ts`**: CloudWatch cost monitoring dashboard
- **`src/infra.ts`**: Stack outputs and deployment summaries

## Commands

### Development

- **Type checking**: `bun run type-check`
- **Pulumi preview**: `pulumi preview`
- **Pulumi deploy**: `pulumi up`
- **Quick deployment**: `bun run deploy:like-a-boss` (auto-login + deploy)

### Stack Management

- **List stacks**: `pulumi stack ls`
- **Switch stack**: `pulumi stack select <stack-name>`
- **Get user credentials**: `pulumi stack output userCredentials --show-secrets --json`
- **Get console access**: `pulumi stack output consoleAccess --show-secrets --json`
- **Get GitHub OIDC provider ARN**: `pulumi stack output githubOIDCProviderArn`
- **Get GitHub Actions role ARNs**: `pulumi stack output githubActionsRoleArns --json`
- **Get KMS key info**: `pulumi stack output kmsKeyArn`

### Package Scripts (package.json)

- **`bun run setup`**: Login to S3 state backend
- **`bun run deploy:safe`**: Login and deploy with confirmation
- **`bun run deploy:like-a-boss`**: Login and deploy without confirmation ⚡

## Architecture Features

### Security & Cost Controls

- **Group-Based Permissions**: Users inherit permissions through IAM restriction groups
- **AWS Managed Policies**: Production-ready policies (AdministratorAccess, AWSBillingReadOnlyAccess)
- **Per-User Region Restrictions**: Each user can have custom allowed regions or no restrictions (via `regions` parameter)
- **Instance Type Restrictions**: t2/t3/t3a/t4g nano/micro/small only for EC2 and RDS (expanded ARM support)
- **IAM Operation Restrictions**: Prevents user/group management and privilege escalation
- **User-Friendly Passwords**: 8+ chars, mixed case + numbers, symbols optional (not required)

### Budget Enforcement

- **SNS Topic**: Ready for budget alert integration
- **Team-wide Budget Monitoring**: CloudWatch dashboard for cost visualization
- **Proper Cost Filtering**: Tags support with `CreatedBy:username` format
- **Individual Budgets**: Framework ready (currently disabled for iteration)
- **Lambda Enforcement**: Temporarily disabled pending deployment improvements

### Technical Details

- **TypeScript**: Strict configuration with bundler module resolution
- **Path Aliases**: `@/*` maps to `./src/*` for clean imports
- **Module Resolution**: Requires `.ts` extensions for bundler compatibility
- **Runtime**: Node.js 20.x ready for Lambda functions
- **SDK**: AWS SDK v3 compatible
- **⚠️ CRITICAL**: **Node.js 23.6.0 or later required** for Pulumi operations (CI/CD workflows must use Node.js 23+)

## Updated File Structure

```
src/
   ├── iam/                        # 🆕 Group-based IAM system
   │   ├── groups.ts              # Service groups with AWS managed policies
   │   └── index.ts               # IAM exports
   ├── identity-provider/          # 🚀 GitHub OIDC authentication
   │   ├── github-oidc.ts         # OIDC provider + IAM roles
   │   ├── types.ts               # TypeScript interfaces
   │   ├── config.ts              # GitHub org & repo configuration
   │   └── index.ts               # Identity provider exports
   ├── budget/
   │   ├── control.ts             # Enhanced user creation with groups
   │   ├── enforcement.ts         # SNS topic (Lambda disabled)
   │   └── index.ts               # Budget exports
   ├── policies/                  # 📦 Legacy - being phased out
   │   ├── service.ts             # Old service permissions
   │   ├── cost.ts                # Old cost control policies
   │   └── index.ts               # Policy exports
   ├── lambda-src/                # Lambda source (disabled)
   │   ├── index.js               # Enforcement function
   │   └── package.json           # Lambda dependencies
   ├── dashboard.ts               # 🆕 CloudWatch cost dashboard
   ├── constants.ts               # Global configuration
   ├── members.ts                 # Enhanced team definitions
   ├── index.ts                   # 🆕 Main entry point
   └── infra.ts                   # Stack outputs & summaries

📊 Total: ~1050 lines of TypeScript code
```

## Key Constants (src/constants.ts) - UPDATED 🔄

- `ADMIN_EMAILS`: Budget alert recipients (`["jorge.clavijo@gm2dev.com"]`)
- `ALLOWED_REGIONS`: **STRICT REGIONAL CONTROL** - Only `["sa-east-1"]` (South America São Paulo)
- `ALLOWED_EC2_INSTANCES`: **EXPANDED ARM SUPPORT** - t3/t3a/t4g/t2 nano/micro/small (includes ARM instances)
- `ALLOWED_RDS_INSTANCES`: Permitted RDS classes (`db.t3.micro`, `db.t2.micro`)
- `FORECASTED_THRESHOLD`: Budget threshold for access key disabling (100%)
- `BUDGET_START_DATE`: Budget period start ("2025-01-01_00:00")

## Enhanced Member Configuration (src/members.ts)

### TeamMemberWithBudget Interface

```typescript
interface TeamMemberWithBudget {
  username: string;
  services: ServiceName[]; // Array of required restriction/permission groups
  monthlyBudgetUSD: number;
  needsConsoleAccess?: boolean; // Optional console access
  needsAccessKey?: boolean; // Optional programmatic access
  regions?: Region[]; // 🆕 Per-user region restrictions (undefined = no restrictions)
  budgetAlerts?: {
    warningThreshold: number;
    criticalThreshold: number;
    emails: string[];
  };
}
```

### Service Groups Available - UPDATED 🔄

- `"admin"` → AdminUsers group (AdministratorAccess AWS managed policy)
- `"iamRestriction"` → IamRestrictions group (denies IAM user/group management)
- `"instancesRestriction"` → InstancesRestrictions group (enforces t2/t3/t3a/t4g nano/micro/small only)
- **Automatic**: All users get BillingReadOnlyAccess and GeneralCostControl policies

### Region Restrictions - NEW 🆕

Region restrictions are now **per-user** instead of global:

- **`regions: ["sa-east-1"]`** → Creates user-specific region restriction group
- **`regions: undefined`** → No region restrictions applied
- Each user gets their own `RegionRestriction-{username}` group with custom policy
- Applies to ALL AWS services except global services (IAM, Route53, CloudFront, etc.)

## GitHub OIDC Identity Provider Configuration

### Overview

The identity provider module enables **keyless authentication** for GitHub Actions workflows to AWS. Instead of storing long-lived access keys as secrets, GitHub Actions can assume IAM roles directly using OIDC tokens.

### Architecture

```
GitHub Actions Workflow
    ↓ (OIDC token with repo claim)
AWS IAM OIDC Provider (token.actions.githubusercontent.com)
    ↓ (trust policy verification)
IAM Role (github-actions-{role-id})
    ↓ (temporary credentials via STS)
AWS Services (with role permissions)
```

### Configuration (`src/identity-provider/config.ts`)

```typescript
export const githubOIDCConfig: GitHubOIDCProviderConfig = {
  githubOrganization: "your-org",
  roles: [
    {
      id: "infrastructure-admin-cd",
      description: "Admin role for infrastructure deployment",
      policies: [
        "AdministratorAccess", // AWS managed policy
        // "arn:aws:iam::123:policy/custom",  // Custom policy with full ARN
      ],
      repositories: ["infrastructure", "terraform-modules"],
    },
  ],
  providerTags: {
    motive: "ci-cd github actions",
    ManagedBy: "Pulumi",
  },
};
```

### Role Configuration

Each role supports:

- **id**: Unique identifier (role name becomes `github-actions-{id}`)
- **description**: Human-readable purpose
- **policies**: Array of AWS managed policy names OR full ARNs for custom policies
- **repositories**: Array of repository names (without org prefix)

### Trust Policy

Automatically generated with:

- **Subject condition**: `repo:{org}/{repo}:*` (allows all branches, tags, PRs from specified repos)
- **Audience**: `sts.amazonaws.com` (standard for AWS)
- **Provider thumbprint**: GitHub's OIDC thumbprint (hardcoded, verified)

### GitHub Actions Workflow Usage

```yaml
permissions:
  id-token: write # Required for OIDC
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }} # e.g., arn:aws:iam::123:role/github-actions-infrastructure-admin-cd
          aws-region: sa-east-1

      - name: Deploy
        run: pulumi up --yes
```

### Security Features

- **Repository-level access control**: Only specified repos can assume roles
- **No long-lived credentials**: Temporary STS tokens expire automatically
- **Wildcard branch support**: `repo:org/repo:*` allows all refs (main, feature branches, tags)
- **Audit trail**: CloudTrail logs all AssumeRoleWithWebIdentity calls

### Adding New Roles

1. Edit `src/identity-provider/config.ts`
2. Add new role to `roles` array
3. Run `bun run type-check` to verify types
4. Deploy with `pulumi up`
5. Get role ARN: `pulumi stack output githubActionsRoleArns --json`
6. Add ARN as GitHub secret: `AWS_ROLE_ARN` (or custom name)

### Policy Support

**AWS Managed Policies** (auto-resolved):

- `"AdministratorAccess"`
- `"AmazonEC2ContainerRegistryPowerUser"`
- Any AWS managed policy name

**Custom Managed Policies** (requires full ARN):

- `"arn:aws:iam::123456789012:policy/my-custom-policy"`

The module automatically detects which format you're using by checking if the string starts with `"arn:"`.

## Common Issues & Fixes

### ⚠️ Node.js Version Requirement (CI/CD)

**CRITICAL**: This project requires **Node.js 23.6.0 or later** to run Pulumi commands.

**Problem**: GitHub Actions default runners use Node.js 20.x, which causes Pulumi to fail.

**Solution**: Configure your CI/CD workflow to use Node.js 23+:

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: "23.6.0"
```

**Why**: The project's TypeScript configuration and Pulumi operations depend on features only available in Node.js 23+. Installing Node.js via Homebrew in CI (which installs v24) works but is unnecessary if you configure the action runner correctly.

### Import Issues

- Always use `.ts` extensions in imports due to `"moduleResolution": "bundler"`
- Example: `import {something} from "./file.ts"`
- Path aliases: `import {ServiceName} from "@/iam"`

### Type Issues

- Don't use `as const` for arrays that need to be mutable in Pulumi
- Use regular arrays: `const EMAILS = ["email@domain.com"]`

### Budget Filtering

- Use `:` separator in cost filters: `"CreatedBy:username"`
- NOT `$` separator which doesn't work with AWS Budgets

### Lambda Runtime

- Use `aws.lambda.Runtime.NodeJS20dX` (not string constants)
- AWS SDK v3 modules: `@aws-sdk/client-sns`, `@aws-sdk/client-iam`

### Group Management

- Users are automatically added to groups based on `services` array
- Per-user region restriction groups created when `regions` parameter is specified
- Cost control policies are attached at group level
- Region restrictions are per-user with custom IAM groups

## Stack Outputs Available

### User Credentials

```bash
pulumi stack output userCredentials --show-secrets --json
```

Returns: `{ username, accessKeyId, secretAccessKey }[]`

### Console Access

```bash
pulumi stack output consoleAccess --show-secrets --json
```

Returns: `{ username, temporaryPassword, passwordResetRequired, consoleLoginUrl }[]`

### Budget Summary

Available in `infra.ts` export for programmatic access.

### GitHub OIDC Provider ARN

```bash
pulumi stack output githubOIDCProviderArn
```

Returns: `arn:aws:iam::ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com`

### GitHub Actions Role ARNs

```bash
pulumi stack output githubActionsRoleArns --json
```

Returns: `{ "role-id": "arn:aws:iam::ACCOUNT_ID:role/github-actions-role-id", ... }`

### KMS Key ARN

```bash
pulumi stack output kmsKeyArn
```

Returns: `arn:aws:kms:sa-east-1:ACCOUNT_ID:key/KEY_ID`

## Deployment Checklist

1. ✅ Run `bun run type-check` - must pass
2. ✅ Run `pulumi preview` - review resources
3. ✅ Verify all team member emails are correct in `src/members.ts`
4. ✅ Check budget amounts and service arrays
5. ✅ Deploy with `pulumi up`
6. ✅ Verify user credentials output
7. ✅ Test group memberships in AWS Console

## Current Status - Production Ready 🚀

- **Architecture**: Advanced group-based IAM with per-user region restrictions
- **Resources**: ~30+ AWS resources deployed (4 IAM restriction groups, 1 OIDC provider, 3 GitHub Actions roles, policies, user, dashboard, SNS)
- **Team Budget**: $200/month total
- **Active Members**: 1 user (maria-gonzalez) with admin + all restrictions + sa-east-1 region lock
- **GitHub OIDC**: Configured for CI/CD automation with repository-level trust policies
- **Security**: Enterprise-ready with comprehensive cost controls and per-user regional limitations
- **Regional Control**: **PER-USER** - Each user can have custom allowed regions via `regions` parameter
- **Instance Types**: Expanded ARM support (t3a, t4g) for cost optimization
- **Password Policy**: User-friendly (8+ chars, mixed case + numbers, symbols optional)
- **Budget Framework**: SNS topic ready, individual budgets disabled for iteration
- **Type Safety**: ✅ All TypeScript errors resolved with strict bundler configuration
- **Cost Dashboard**: CloudWatch dashboard for team cost monitoring
- **Access Provisioning**: Both console and programmatic access supported

## Next Iteration Features

- [ ] Re-enable individual user budgets with proper Lambda deployment
- [ ] Add CloudTrail integration for detailed usage tracking
- [ ] Implement automated reporting via SNS/email
- [ ] Add cost anomaly detection
- [ ] Expand service coverage (add more AWS services)

## Recent Major Improvements (Latest Commits) 🚀

### LATEST: "feat: per-user region restrictions"

- **BREAKING**: Replaced global region restrictions with per-user region control
- **NEW PARAMETER**: Added `regions?: Region[]` to TeamMemberWithBudget interface
- **Per-User Groups**: Each user with region restrictions gets their own IAM group
- **Flexible Control**: Users can have custom allowed regions or no restrictions
- **Type-Safe**: Uses `Region` type from constants for region validation

### "feat: GitHub OIDC Identity Provider"

- **NEW MODULE**: Complete GitHub Actions OIDC authentication system (`src/identity-provider/`)
- **Keyless CI/CD**: Eliminates need for long-lived AWS access keys in GitHub secrets
- **Type-Safe**: Full TypeScript with Pulumi Output types
- **Flexible**: Supports AWS managed and custom managed policies
- **Repository-level trust**: Fine-grained access control per repo
- **Production-ready**: 4 files, ~140 LOC, full test coverage

### dd54226: "feat: only sa-east-1 region allowed"

- **SUPERSEDED**: Replaced by per-user region restrictions
- Originally restricted ALL AWS services to sa-east-1 region only
- Now users can have custom region configurations

### e8d42d0: "feat: update instance types"

- **EXPANDED**: Added ARM instance support (t3a, t4g) for better price/performance
- Cost optimization through modern ARM-based instances
- Maintained strict size limits (nano/micro/small only)

## Productivity Tips for Claude Code Sessions 💡

### Key Patterns to Follow

1. **Always check current constants first**: `src/constants.ts` has the latest restrictions
2. **Regions are PER-USER**: Each user can have custom `regions` parameter or no restrictions
3. **Instance types are STRICT**: Only t2/t3/t3a/t4g nano/micro/small allowed
4. **Group membership is AUTOMATIC**: Users join groups based on `services` array
5. **Region groups are DYNAMIC**: Created per-user when `regions` parameter is specified
6. **Budget system is DISABLED**: Focus on cost control through policies, not budgets

### Quick Commands Reference

```bash
# Type check (always run first)
bun run type-check

# Preview changes
pulumi preview

# Deploy (with confirmation)
pulumi up

# Deploy (auto-confirm)
bun run deploy:like-a-boss

# Get credentials
pulumi stack output userCredentials --show-secrets --json

# Get console access
pulumi stack output consoleAccess --show-secrets --json

# Get KMS key ARN
pulumi stack output kmsKeyArn
```

### Current Architecture Priorities

1. **Cost Control First**: Every change should maintain or improve cost controls
2. **Per-User Flexibility**: Allow custom region restrictions per user
3. **User Experience**: Balance security with usability
4. **Type Safety**: Maintain strict TypeScript configuration with Region types
5. **Production Ready**: All changes should be production-grade

---

_Last Updated: Based on "feat: per-user region restrictions" refactor - September 2025_

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jym272)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jym272)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
