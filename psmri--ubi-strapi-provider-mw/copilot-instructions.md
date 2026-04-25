## ubi-strapi-provider-mw

> NestJS-based middleware application serving as an intermediary between external systems and Strapi CMS for Universal Basic Income (UBI) benefit management. Implements ONDC-compliant financial services with comprehensive document management.

# UBI Strapi Provider Middleware - Cursor Rules

## Project Overview
NestJS-based middleware application serving as an intermediary between external systems and Strapi CMS for Universal Basic Income (UBI) benefit management. Implements ONDC-compliant financial services with comprehensive document management.

## Technology Stack
- **Framework**: NestJS with TypeScript
- **Database**: PostgreSQL with Prisma ORM
- **CMS**: Strapi with JWT authentication
- **File Storage**: AWS S3 / Local storage abstraction
- **Standards**: ONDC/DSEP compliance for financial services
- **Security**: AES-256-GCM encryption for sensitive data
- **Background Jobs**: Cron-based processing
- **Validation**: class-validator with comprehensive DTOs

## Module Structure
```
src/
├── benefits/              # ONDC benefit discovery and DSEP integration
├── applications/          # Application lifecycle with encryption and cron jobs
├── applicationFiles/      # Multi-provider file storage and verification
├── auth/                  # Strapi JWT integration
├── strapi-admin/         # Programmatic Strapi role/user management
├── verifications/        # Document verification workflows
├── services/storage-providers/ # S3/Local storage abstraction
└── common/middleware/    # Global auth middleware
```

## Key Architectural Patterns

### ONDC/DSEP Integration
- Domain: `"onest:financial-support"`
- DSEP endpoints: `/benefits/dsep/*`
- Transaction workflows: `init`, `confirm`, `status` with UUID tracking
- Message format: context + message structure with BPP/BAP configuration

### Strapi Integration
- JWT token authentication via `STRAPI_TOKEN`
- Content transformation from Strapi to ONDC format
- Admin API for user/role management
- Comprehensive field population for relationships

### Data Security
```typescript
// Encryption configuration
encryptionMap = {
  Applications: ['applicationData'] // Field-level AES-256-GCM
};
// Supports key rotation with OLD_ENCRYPTION_KEY fallback
```

### File Storage Abstraction
```typescript
// Dynamic provider selection
FILE_STORAGE_PROVIDER = 's3' | 'local'
// Factory pattern for storage service injection
```

### Authentication Middleware
```typescript
// Global optional authentication
consumer.apply(AuthMiddleware).forRoutes({ path: '/*', method: RequestMethod.ALL });
// Adds req.mw_userid when token present
```

## Environment Configuration
```bash
# Core Integration
DATABASE_URL=postgresql://...
STRAPI_URL=https://strapi.domain.com
STRAPI_TOKEN=your_strapi_token

# ONDC Configuration  
BPP_ID=provider.domain.com
BPP_URI=https://provider.domain.com/
BAP_ID=consumer.domain.com
BAP_URI=https://consumer.domain.com/

# File Storage
FILE_STORAGE_PROVIDER=s3|local
AWS_S3_BUCKET_NAME=bucket_name

# Security
ENCRYPTION_KEY=base64_encoded_32_byte_key

# Background Jobs
BENEFIT_CALCULATIONS_CRON_TIME=*/30 * * * *
ELIGIBILITY_CHECK_CRON_TIME=*/30 * * * *
```

## Business Logic Patterns

### Application Status Flow
- States: `pending`, `approved`, `rejected`, `in_progress`
- Audit trails with `updatedBy` and `actionLog`
- Encrypted `applicationData` field for sensitive information

### Benefits Processing
- ONDC-compliant message transformation
- Dynamic benefit amount calculation
- Eligibility checking with detailed results
- Provider catalog management through Strapi

### File Handling
- Base64 automatic decode and upload
- Signed URL generation for secure access
- External verification service integration
- Multi-provider storage support

## API Conventions
- **50MB request limit** for file uploads
- **Global validation** with `whitelist: true`
- **Consistent error format** via AllExceptionsFilter
- **ONDC endpoint structure**: `/benefits/dsep/*` for DSEP-specific operations

## Background Processing
- **Configurable cron expressions** from environment
- **Automatic scheduler registration** via `SchedulerRegistry`
- **Benefit calculations** and **eligibility checks** as separate jobs

## Development Patterns
- **DTO naming**: `Create*Dto`, `Update*Dto`, `List*Dto`
- **Service methods**: `findUniqueApplication`, `getBenefitsByIdStrapi`
- **Error handling**: Structured responses with security-aware logging
- **Database**: Prisma middleware for transparent encryption/decryption

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PSMRI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
