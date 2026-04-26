## viewdocs-cloud

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Viewdocs Cloud Migration** is a multi-tenant, serverless document management system migrating from on-premise Java/Spring/Tomcat/Oracle stack to AWS serverless architecture. The system provides document viewing, search, download, and management capabilities across multiple archive systems (IESC, IES, CMOD).

### Key Characteristics
- **Multi-tenant**: Pool model with logical isolation (5-500 tenants)
- **Serverless**: AWS Lambda, API Gateway, DynamoDB, S3, CloudFront
- **Hybrid**: Integrates with on-premise systems (IES, CMOD, FRS, HUB) via Direct Connect
- **Multi-region**: Primary in ap-southeast-2, DR in ap-southeast-4 (Active-Passive failover)
- **Scale**: 500 concurrent users, 10-1000 users per tenant

## Technology Stack

### Backend
- **Language**: TypeScript
- **Runtime**: Node.js on AWS Lambda
- **API**: API Gateway (REST)
- **Database**: DynamoDB with Global Tables
- **Storage**: S3 (temporary document storage, bulk downloads)
- **Authentication**: AWS Cognito with SAML 2.0 federation
- **Event Processing**: EventBridge, Step Functions, SQS

### Frontend
- **Framework**: Angular
- **Hosting**: S3 + CloudFront

### Infrastructure as Code
- **Tool**: AWS CDK (TypeScript)
- **Deployment**: CDK Pipelines

### Integrations
- **IESC**: REST API (AWS-hosted, per-tenant stack)
- **IES**: SOAP API (on-premise via Direct Connect)
- **CMOD**: SOAP API (IBM on-premise via Direct Connect)
- **FRS Proxy**: SOAP API (AWS proxy to on-premise FRS/IBM MQ via Direct Connect)
- **IDM**: SAML 2.0 IdP (AWS-hosted) + support for external IdPs
- **Email**: IDM Email Service (current), Email Platform REST (future)
- **MailRoom Backend**: REST API (AWS-hosted, independent document routing and assignment microservice)

## Architecture Principles

### Multi-Tenancy (Pool Model)
- **Logical Isolation**: Single shared infrastructure with tenant_id partitioning
- **Tenant Identification**: Subdomain-based routing (tenant1.viewdocs.example.com)
- **Data Isolation**: DynamoDB partition key design with tenant_id prefix
- **No Cross-Tenant Access**: Strict authorization checks on every request

### Serverless & Non-VPC (ADR-012)
- **Lambda WITHOUT VPC**: All Lambda functions deployed without VPC for cost savings ($1,992/year) and performance (faster cold starts)
- **Public Endpoints**: Lambda connects to AWS services (DynamoDB, S3, Secrets Manager) via public endpoints with IAM authentication
- **Direct Connect**: Public VIF with IPsec VPN tunnel to on-premise systems (IES, CMOD, FRS)
- **No NAT Gateways**: No VPC infrastructure to manage (no subnets, route tables, security groups)
- **Equivalent Security**: TLS 1.2+ encryption + IPsec VPN + IAM roles + KMS encryption

### Security
- **Data Residency**: All data in Australia (ap-southeast-2, ap-southeast-4)
- **Encryption at Rest**: DynamoDB, S3 with KMS
- **Encryption in Transit**: TLS 1.2+ for AWS services, IPsec VPN for Direct Connect
- **Authentication**: Cognito with SAML 2.0, support for external IdPs
- **Authorization**: Role-based access control (RBAC) with ACLs stored in DynamoDB
- **Audit**: All document operations logged to DynamoDB with TTL (6mo prod, 1mo UAT, 1wk dev)

### Performance & Resilience
- **Caching**: CloudFront for static assets, DynamoDB for configuration/ACLs
- **Async Processing**: Step Functions for bulk downloads (up to 5GB)
- **Retry Logic**: Built into FRS Proxy for on-premise integrations
- **Concurrency**: Lambda concurrency controls to prevent noisy neighbor

### Observability
- **Logging**: CloudWatch Logs (centralized per tenant)
- **Metrics**: CloudWatch Metrics
- **Tracing**: X-Ray for distributed tracing
- **Events**: EventBridge for real-time events to HUB

## Common Development Commands

### Source Control (Bitbucket)
```bash
# Create feature branch
git checkout -b feature/JIRA-123-description

# Commit changes
git add .
git commit -m "feat: add bulk download feature"

# Push to Bitbucket
git push origin feature/JIRA-123-description

# Create Pull Request via Bitbucket UI
# After PR approval, merge to develop triggers Jenkins pipeline
```

### CDK Infrastructure
```bash
# Install dependencies
npm install

# Bootstrap CDK (first time only per account/region)
cdk bootstrap aws://ACCOUNT-ID/ap-southeast-2
cdk bootstrap aws://ACCOUNT-ID/ap-southeast-4

# Synthesize CloudFormation templates
cdk synth

# Deploy to dev environment (via Jenkins or manual)
cdk deploy --all --context env=dev

# Deploy to UAT environment (via Jenkins or manual)
cdk deploy --all --context env=uat

# Deploy to prod environment (via Jenkins only - requires Heat approval)
cdk deploy --all --context env=prod

# Destroy stack (dev/uat only)
cdk destroy --all --context env=dev
```

### Backend Lambda Development
```bash
# Install dependencies
cd backend
npm install

# Run unit tests
npm test

# Run single test file
npm test -- --testPathPattern=document-service.test.ts

# Run tests with coverage
npm test -- --coverage

# Lint code
npm run lint

# Fix linting issues
npm run lint:fix

# Build for deployment
npm run build

# Run locally with SAM (if configured)
sam local start-api
```

### Frontend Angular Development
```bash
# Install dependencies
cd frontend
npm install

# Run dev server
npm start
# Access at http://localhost:4200

# Run unit tests
npm test

# Run single test file
npm test -- --include='**/document-viewer.component.spec.ts'

# Run e2e tests
npm run e2e

# Build for production
npm run build:prod

# Lint
npm run lint
```

## Repository Structure

```
/
├── docs/
│   └── architecture/          # Architecture documentation (TOGAF + C4)
│       ├── 00-architecture-overview.md
│       ├── 01-business-architecture.md
│       ├── 02-application-architecture.md
│       ├── 03-data-architecture.md
│       ├── 04-technology-architecture.md
│       ├── 05-security-architecture.md
│       ├── 06-deployment-architecture.md
│       ├── 07-infrastructure-architecture.md
│       ├── 08-cost-architecture.md
│       ├── 10-decision-log.md
│       └── diagrams/          # Mermaid + draw.io diagrams
├── infrastructure/            # AWS CDK code
│   ├── bin/                   # CDK app entry point
│   ├── lib/                   # CDK stack definitions
│   │   ├── stacks/
│   │   │   ├── api-stack.ts
│   │   │   ├── auth-stack.ts
│   │   │   ├── data-stack.ts
│   │   │   ├── frontend-stack.ts
│   │   │   ├── event-stack.ts
│   │   │   └── monitoring-stack.ts
│   │   └── constructs/        # Reusable CDK constructs
│   ├── test/                  # CDK tests
│   └── cdk.json
├── backend/                   # Lambda functions (TypeScript)
│   ├── src/
│   │   ├── functions/         # Lambda handlers
│   │   ├── services/          # Business logic
│   │   ├── models/            # Data models
│   │   ├── middleware/        # Auth, logging, error handling
│   │   └── utils/             # Shared utilities
│   ├── test/
│   └── package.json
├── frontend/                  # Angular application
│   ├── src/
│   │   ├── app/
│   │   │   ├── core/          # Singleton services, guards
│   │   │   ├── shared/        # Shared components, pipes, directives
│   │   │   ├── features/      # Feature modules
│   │   │   │   ├── documents/
│   │   │   │   ├── search/
│   │   │   │   ├── admin/
│   │   │   │   └── ...
│   │   │   └── app.module.ts
│   │   ├── assets/
│   │   └── environments/
│   └── package.json
└── intent-statement.md        # Business requirements
```

## Key Architecture Patterns

### 1. Tenant Isolation Pattern
```typescript
// Every Lambda function extracts tenant_id from request
const tenantId = extractTenantIdFromSubdomain(event.headers.host);
// All DynamoDB queries include tenant_id partition key
const params = {
  TableName: 'viewdocs-config',
  Key: { PK: `TENANT#${tenantId}`, SK: `CONFIG#archive` }
};
```

### 2. Archive Abstraction Pattern
```typescript
// Factory pattern for different archive types
interface ArchiveClient {
  search(params): Promise<SearchResult>;
  getDocument(docId): Promise<Document>;
}

class IESCClient implements ArchiveClient { /* REST */ }
class IESClient implements ArchiveClient { /* SOAP */ }
class CMODClient implements ArchiveClient { /* SOAP */ }
```

### 3. MailRoom Integration Pattern (ADR-013)
```typescript
// Backend for Frontend (BFF) + Anti-Corruption Layer
// MailRoom Wrapper Service translates between Viewdocs and MailRoom formats

class MailRoomWrapperService {
  async getMailItems(tenantId: string, userId: string, filters: any) {
    // 1. Authorization: Check Viewdocs ACLs
    await this.checkUserAccess(tenantId, userId);

    // 2. Tenant Isolation: Inject tenant_id into MailRoom call
    const mailroomRequest = {
      tenant_id: tenantId,
      filters: this.translateFilters(filters)
    };

    // 3. Call MailRoom Backend API
    const response = await this.mailroomClient.getItems(mailroomRequest);

    // 4. Response Translation: Map MailRoom format → Viewdocs format
    return this.translateResponse(response);

    // 5. Audit Logging: Log to Viewdocs audit table
    await this.logAudit(tenantId, userId, 'MailRoom:GetItems');
  }
}

// MailRoom UI integrated into Viewdocs Angular app (unified UX)
// MailRoom Backend remains independent (reusable by other clients)
```

### 4. Event-Driven Pattern
```typescript
// All document operations emit events to EventBridge
await eventBridge.putEvents({
  Entries: [{
    Source: 'viewdocs',
    DetailType: 'DocumentViewed',
    Detail: JSON.stringify({ tenantId, userId, documentId })
  }]
});
// EventBridge rule forwards to FRS Proxy → HUB
```

### 5. Bulk Download Pattern
```typescript
// Step Functions orchestration
{
  "StartAt": "ValidateRequest",
  "States": {
    "ValidateRequest": { /* Check ACLs */ },
    "FanOutDocuments": { /* Map over document IDs */ },
    "FetchDocument": { /* Lambda per document, concurrency=1 */ },
    "AggregateToS3": { /* Zip documents */ },
    "NotifyUser": { /* Email via IDM Email Service */ }
  }
}
```

## DynamoDB Table Design

### Single-Table Design with GSIs

**Main Table**: `viewdocs-data` (Global Table)

#### Access Patterns:
1. Get tenant configuration
2. Get user's role-to-ACL mapping
3. Get folder ACLs
4. Query audit logs by tenant + time range
5. Query comments by document
6. Get bulk download job status

#### Key Schema:
```
PK (Partition Key)         SK (Sort Key)                    Entity Type
-----------------------------------------------------------------------------------
TENANT#<tenantId>          CONFIG#archive                   Archive config
TENANT#<tenantId>          ROLE#<roleId>#ACL                Role-to-ACL mapping
TENANT#<tenantId>          FOLDER#<folderId>#ACL            Folder ACL
TENANT#<tenantId>          AUDIT#<timestamp>#<eventId>      Audit event
TENANT#<tenantId>          DOWNLOAD#<jobId>                 Bulk download job
DOC#<docId>                COMMENT#<timestamp>#<commentId>  Document comment
```

#### GSIs:
- **GSI1**: For querying documents by tenant (PK=TENANT#<tenantId>, SK=DOC#<docId>)
- **GSI2**: For user activity queries (PK=USER#<userId>, SK=AUDIT#<timestamp>)

## Security Guidelines

### Authentication Flow
1. User accesses `https://<tenant>.viewdocs.example.com`
2. CloudFront → S3 Angular app loads
3. Angular initiates Cognito-hosted UI login
4. Cognito redirects to appropriate IdP (IDM SAML or external SAML)
5. After SAML assertion, Cognito issues JWT tokens
6. Frontend includes JWT in Authorization header for API calls
7. API Gateway validates JWT with Cognito authorizer

### Authorization Flow
1. Lambda extracts `tenantId` from subdomain + `userId` from JWT
2. Query DynamoDB for user's roles (from IdP claim mapped to Viewdocs roles)
3. For document operation, check folder ACLs against user roles
4. If authorized, proceed; else return 403

### Secrets Management
- **Never hardcode**: Archive endpoints, API keys, credentials
- **Use AWS Secrets Manager**: Store per-tenant archive credentials
- **Rotation**: Enable automatic rotation for credentials
- **Access**: Lambda execution role with least-privilege access to secrets

## Testing Strategy

### Unit Tests
- **Coverage**: Minimum 80% for backend services
- **Mocking**: Mock DynamoDB, archive clients, EventBridge
- **Framework**: Jest

### Integration Tests
- **Scope**: Test Lambda + DynamoDB + SQS interactions
- **Environment**: Dedicated test DynamoDB tables
- **Cleanup**: Tear down test data after each run

### E2E Tests
- **Scope**: Frontend → API Gateway → Lambda → DynamoDB
- **Framework**: Cypress or Playwright
- **Environment**: UAT environment with test tenants

### Load Tests
- **Tool**: Artillery or Locust
- **Scenarios**: 500 concurrent users, document search/view/download
- **Metrics**: p95/p99 latency, error rate, Lambda throttles

## Deployment Strategy

### CI/CD Pipeline (Bitbucket + Jenkins)
- **Source Control**: Bitbucket
- **CI/CD**: Jenkins with dedicated agents per environment
- **Deployment**: CDK → CloudFormation
- **Approvals**: Heat System (Change Control) for UAT and Production

### Pipeline Flow
```
Bitbucket PR (merged to develop)
  → Jenkins Webhook Trigger
  → Build & Test
  → Deploy Dev (jenkins-dev-agent)
  → Integration Tests
  → Heat Call Approval (UAT)
  → Deploy UAT (jenkins-uat-agent)
  → E2E Tests + Load Tests
  → Heat Call Approval (Production)
  → Deploy Prod Blue-Green (jenkins-prod-agent)
  → Monitor Canary → Gradual Rollout
```

### Environments
- **Dev**: Auto-deploy on merge to `develop` branch
- **UAT**: Heat System approval required
- **Prod**: Heat System approval + scheduled deployment window (off-peak hours)

### Blue-Green Deployment for Multi-Tenant
1. Deploy new version to "green" API Gateway stage (via Jenkins prod agent)
2. Select 1-2 canary tenants, route via DNS to green stage
3. Monitor for 24-48 hours (CloudWatch alarms)
4. Gradual rollout: 10% → 25% → 50% → 100% of tenants
5. If issues, instant rollback by reverting DNS/API Gateway stage

### Rollback Procedure
```bash
# Immediate rollback via API Gateway stage swap
aws apigateway update-stage --rest-api-id <api-id> --stage-name prod --patch-operations op=replace,path=/deploymentId,value=<previous-deployment-id>

# Or rollback CDK deployment
cdk deploy --all --context env=prod --rollback
```

### Heat System Integration
- UAT deployments require Heat Call approval (24-48 hour turnaround)
- Production deployments require Heat Change Control with CAB review (48-72 hour turnaround)
- Deployment windows scheduled for off-peak hours (e.g., Saturday 2:00-6:00 AM AEST)

## Monitoring & Alerting

### Key Metrics to Monitor
- **API Latency**: p95/p99 response times per endpoint
- **Error Rate**: 4xx/5xx errors per tenant
- **Lambda Duration**: Near timeout warnings
- **DynamoDB Throttles**: Read/write capacity exceeded
- **Archive Failures**: IESC/IES/CMOD API errors
- **Bulk Download Queue**: SQS queue depth, age of oldest message

### CloudWatch Alarms
```typescript
// Example: Lambda error rate > 5%
new cloudwatch.Alarm(this, 'HighErrorRate', {
  metric: lambdaFunction.metricErrors(),
  threshold: 5,
  evaluationPeriods: 2,
  datapointsToAlarm: 2,
  alarmActions: [snsTopic]
});
```

### X-Ray Tracing
- Enable on all Lambda functions
- Trace archive API calls to identify slow integrations
- Correlate errors across services

## Cost Optimization

### Lambda
- Right-size memory (start at 512MB, tune based on CloudWatch metrics)
- Use Lambda Insights for memory/CPU utilization
- Reserved concurrency only for critical functions

### DynamoDB
- On-demand pricing for dev/UAT (unpredictable load)
- Provisioned capacity for prod (predictable, cheaper at scale)
- Enable DynamoDB auto-scaling for provisioned mode
- TTL for audit logs (automatic deletion)

### S3
- Lifecycle policies: Move bulk download files to Glacier after 7 days, delete after 30 days
- Intelligent-Tiering for infrequently accessed documents

### CloudFront
- Optimize cache hit ratio (longer TTLs for static assets)
- Enable compression

### Direct Connect
- Shared with other FBDMS systems to amortize cost

## Disaster Recovery

### RPO: 2 hours | RTO: 24 hours (Active-Passive)

#### Backup Strategy
- **DynamoDB Global Tables**: Continuous replication to ap-southeast-4
- **S3 Cross-Region Replication**: Bulk download buckets to DR region
- **Secrets Manager**: Replicated to DR region
- **CloudFront**: Multi-origin failover (automatic)

#### Failover Procedure
1. Detect primary region failure (Route 53 health checks)
2. Route 53 automatically fails over to DR region (ap-southeast-4)
3. Verify DynamoDB Global Tables in DR region are healthy
4. Scale up Lambda concurrency in DR region if needed
5. Monitor for 24 hours, then fail back to primary region

## Common Pitfalls to Avoid

1. **Lambda Cold Starts**: Use provisioned concurrency for latency-sensitive functions (non-VPC Lambdas have faster cold starts ~200-500ms)
2. **DynamoDB Hot Partitions**: Design partition keys with high cardinality (tenant_id + random suffix)
3. **Timeout Cascades**: Set Lambda timeout < API Gateway timeout (29s max)
4. **CORS Issues**: Configure API Gateway CORS properly for Angular app
5. **Large Lambda Packages**: Use Lambda Layers for shared dependencies (AWS SDK, utilities)
6. **Hardcoded Tenant IDs**: Always extract dynamically from request context
7. **Missing Authorization Checks**: Every Lambda must validate tenant + user access
8. **Unencrypted Secrets**: Use Secrets Manager, never environment variables for sensitive data
9. **DO NOT add Lambda to VPC**: Architecture uses non-VPC Lambdas per ADR-012 (cost, performance, simplicity)

## References

- **AWS Well-Architected Framework**: https://aws.amazon.com/architecture/well-architected/
- **AWS Serverless Multi-Tenancy**: https://aws.amazon.com/solutions/implementations/saas-identity-cognito/
- **DynamoDB Best Practices**: https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html
- **TOGAF**: https://www.opengroup.org/togaf
- **C4 Model**: https://c4model.com/

## Decision Log

See [docs/architecture/10-decision-log.md](docs/architecture/10-decision-log.md) for Architecture Decision Records (ADRs).

## Project Status & History

### Current Phase: Architecture Complete ✅

**Status**: Architecture documentation complete and ready for implementation

**What Has Been Completed** (January 2025):

#### Phase 1: Business Requirements Gathering ✅
- Stakeholder interviews and requirement analysis
- Business case and ROI calculation
- Use case documentation (View Document, Bulk Download, Tenant Onboarding)
- Created `intent-statement.md` with business and technical requirements

#### Phase 2: Architecture Design ✅
- **Complete TOGAF + C4 Model Architecture** (~270KB documentation)
- 9 core architecture documents covering all aspects:
  - 00-architecture-overview.md (HLSD - High-Level Solution Design)
  - 01-business-architecture.md (Use cases, stakeholders, business rules, ROI)
  - 02-application-architecture.md (Services, APIs, integration patterns)
  - 03-data-architecture.md (DynamoDB schema, access patterns)
  - 04-technology-architecture.md (Tech stack, tools, SDKs)
  - 05-security-architecture.md (Auth, encryption, compliance, threat model)
  - 06-deployment-architecture.md (CI/CD, blue-green deployment)
  - 08-cost-architecture.md (Cost breakdown, optimization strategies)
  - 10-decision-log.md (13 Architecture Decision Records)

#### Phase 3: Diagrams & Visualizations ✅
- **C4 Model Diagrams** (Mermaid format):
  - Level 1: System Context diagram
  - Level 2: Container diagram (AWS services)
  - Sequence diagrams (6 flows: auth, document view, bulk download, search, admin, comments)
  - Deployment topology (multi-region with DR)
- **Draw.io Instructions**: Step-by-step guide for creating AWS icon diagrams

#### Phase 4: Key Decisions Made ✅
- **ADR-001**: Pool multi-tenancy model (vs silo) - Cost: $2.96/tenant/month
- **ADR-002**: AWS Cognito with SAML 2.0 (vs custom JWT)
- **ADR-003**: DynamoDB Global Tables (vs RDS Aurora)
- **ADR-004**: Hybrid sync/async archive integration
- **ADR-005**: EventBridge for event processing
- **ADR-006**: S3 + CloudFront (vs Amplify)
- **ADR-007**: AWS CDK with TypeScript (vs Terraform)
- **ADR-008**: Node.js 20.x + TypeScript (vs Python/Go)
- **ADR-009**: REST API (vs GraphQL)
- **ADR-010**: Blue-green deployment with canary rollout
- **ADR-011**: CloudWatch + X-Ray (vs ELK/Datadog)
- **ADR-012**: Lambda WITHOUT VPC (vs VPC) - Cost savings: $1,992/year
- **ADR-013**: MailRoom Backend-Only Platform with Viewdocs UI Wrapper (BFF + Anti-Corruption Layer pattern)

#### Phase 5: Cost Analysis ✅
- **Production Monthly Cost**: $1,282/month (primary + DR)
  - 500 tenants, 500 concurrent users
  - Cost per tenant: $2.96/month
- **Cost Savings**: 69% reduction vs on-premise ($300K/year → $94K/year)
- **ROI**: 2.5-year payback period
- **Reserved Capacity Potential**: Additional $321/month savings

#### Repository Setup ✅
- Git repository initialized
- GitHub repository created: https://github.com/onesphereai/viewdocs-cloud
- All documentation pushed (20 files, 8,765 lines)
- .gitignore configured for Node.js/TypeScript/AWS CDK

### Next Steps: Implementation Roadmap

#### Phase 6: Foundation Setup (Weeks 1-4) 🔄 NEXT
- [ ] Set up AWS accounts (dev, uat, prod)
- [ ] Bootstrap AWS CDK in all regions (ap-southeast-2, ap-southeast-4)
- [ ] Configure Direct Connect to on-premise systems
- [ ] Set up CI/CD pipeline (CDK Pipelines)
- [ ] Initialize project structure:
  ```bash
  mkdir -p infrastructure backend frontend
  cd infrastructure && npx aws-cdk init app --language typescript
  cd ../backend && npm init -y
  cd ../frontend && ng new viewdocs-frontend
  ```

#### Phase 7: Core Infrastructure (Weeks 5-8)
- [ ] Implement CDK stacks:
  - [ ] Foundation Stack (IAM roles, KMS keys)
  - [ ] Data Stack (DynamoDB Global Tables, S3 buckets)
  - [ ] Auth Stack (Cognito User Pool, SAML IdP integration)
  - [ ] API Stack (API Gateway, Lambda placeholders)
- [ ] Deploy to dev environment
- [ ] Test multi-region replication (DynamoDB, S3)

#### Phase 8: Backend Services (Weeks 9-14)
- [ ] Implement Lambda functions:
  - [ ] Document Service (view, download, ACL checks)
  - [ ] Search Service (index search, archive integration)
  - [ ] Admin Service (tenant onboarding, user management)
  - [ ] Auth Service (SAML callback handling)
  - [ ] Comment Service (CRUD operations)
  - [ ] Download Service (bulk download orchestration)
  - [ ] Event Service (EventBridge → FRS integration)
- [ ] Implement archive clients (IESC REST, IES SOAP, CMOD SOAP)
- [ ] Write unit tests (80% coverage target)
- [ ] Integration testing with DynamoDB

#### Phase 9: Frontend Development (Weeks 15-20)
- [ ] Angular application setup (Angular 17+)
- [ ] Core modules:
  - [ ] Authentication module (Cognito integration)
  - [ ] Document viewer module (PDF.js)
  - [ ] Search module (index search, full-text search)
  - [ ] Admin module (tenant management, user management)
  - [ ] Comments module (add, edit, view history)
  - [ ] Bulk download module (job submission, status tracking)
- [ ] UI/UX implementation (Angular Material)
- [ ] E2E tests (Cypress)

#### Phase 10: Advanced Features (Weeks 21-24)
- [ ] Step Functions workflow for bulk downloads
- [ ] EventBridge integration with FRS Proxy
- [ ] Email notifications (IDM Email Service)
- [ ] CloudWatch dashboards and alarms
- [ ] X-Ray tracing setup
- [ ] Load testing (Artillery - 500 concurrent users)

#### Phase 11: UAT & Testing (Weeks 25-28)
- [ ] Deploy to UAT environment
- [ ] Onboard 2 pilot tenants
- [ ] Security testing:
  - [ ] Penetration testing
  - [ ] Tenant isolation verification
  - [ ] ACL enforcement testing
- [ ] Performance testing (p95 latency < 500ms)
- [ ] DR failover testing (ap-southeast-2 → ap-southeast-4)

#### Phase 12: Production Launch (Weeks 29-30)
- [ ] Deploy to production (blue-green)
- [ ] Migrate first 5 tenants
- [ ] Monitor canary tenants (24-48 hours)
- [ ] Gradual rollout (10% → 25% → 50% → 100%)
- [ ] Decommission on-premise Viewdocs (3 months post-launch)

### Timeline Summary
- **Total Duration**: 30 weeks (~7 months)
- **Current Progress**: Phase 1-5 complete (architecture)
- **Next Milestone**: Foundation setup (4 weeks)
- **Production Launch Target**: Month 7

### Team Responsibilities
- **Cloud Architect**: Architecture review, ADR approvals ✅ COMPLETE
- **Tech Lead**: CDK implementation, code reviews 🔄 NEXT
- **Backend Developers**: Lambda functions, archive integration
- **Frontend Developers**: Angular application, UI/UX
- **DevOps Engineer**: CI/CD pipeline, monitoring
- **QA Engineer**: Testing strategy, automation
- **Security Engineer**: Security review, penetration testing

### Documentation Status
| Document | Status | Completeness |
|----------|--------|--------------|
| Business Requirements | ✅ Complete | 100% |
| Architecture Documents (9) | ✅ Complete | 100% |
| Architecture Decision Records (11) | ✅ Complete | 100% |
| Diagrams (Mermaid) | ✅ Complete | 100% |
| Draw.io AWS Diagrams | 📋 Instructions Ready | 0% (to be created) |
| CDK Infrastructure Code | ❌ Not Started | 0% |
| Backend Lambda Code | ❌ Not Started | 0% |
| Frontend Angular Code | ❌ Not Started | 0% |
| Test Suites | ❌ Not Started | 0% |

### Key Metrics Targets (Post-Launch)
- **API Latency (p95)**: < 500ms
- **Document View (p95)**: < 2s
- **Search Results (p95)**: < 1s
- **Uptime SLA**: 99.9% (43.8min downtime/month)
- **Monthly Cost**: $1,282 (500 tenants)
- **Cost per Tenant**: $2.96/month

### Repository Information
- **GitHub**: https://github.com/onesphereai/viewdocs-cloud
- **Commits**: 1 (initial architecture documentation)
- **Files**: 20 files, 8,765 lines of documentation
- **Size**: ~270KB

---

## Contact & Support

- **Project Team**: FBDMS ECM Team
- **GitHub Repository**: https://github.com/onesphereai/viewdocs-cloud
- **Architecture Lead**: Architecture Team (Phase 1-5 complete)
- **Next Phase Owner**: Tech Lead (Foundation setup)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/onesphereai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
