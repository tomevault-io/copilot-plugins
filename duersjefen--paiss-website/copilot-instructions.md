## paiss-website

> Static company website deployed via multi-tenant platform.

# CLAUDE.md - PAISS Project Instructions

Static company website deployed via multi-tenant platform.

---

## 🔗 DOCUMENTATION HIERARCHY

**Global principles (debugging, notifications, professional guidance):**
→ `/Users/martijn/Documents/Projects/CLAUDE.md`

**Platform infrastructure (nginx, SSL, deployment):**
→ `/Users/martijn/Documents/Projects/multi-tenant-platform/CLAUDE.md`

**This file:** PAISS specific configuration and deployment details only.

---

## ⚙️ PAISS CONFIGURATION

### Quick Reference
```bash
# Development
make dev                   # Start Vite dev server only (no AWS)
make dev-sst               # Start SST dev mode (AWS resources + Vite)
make build                 # Build project with Vite
make preview               # Preview production build locally

# Deployment
make deploy-staging        # Deploy to staging
make deploy-production     # Deploy to production
make remove-dev            # Remove dev stage AWS resources
make status                # Check deployment status

# Environment Stages
# Dev:        CloudFront URL (use make dev-sst)
# Staging:    https://staging.paiss.me
# Production: https://paiss.me

# Port Auto-Detection
# Vite automatically uses next available port (8002 → 5173 → 5174...)
# Multiple dev instances can run simultaneously
# No manual port killing needed

# NEVER use manual server commands - always use Makefile
# NOTE: Vite has hot-reloading (HMR) - no restart needed for code changes
# IMPORTANT: make dev is usually already running in a separate terminal
#            Do NOT run make dev unless explicitly requested by user
```

---

## 🏗️ PROJECT STRUCTURE

```
paiss-website/
├── index.html              # Main landing page
├── styles.css              # Styling
├── script.js               # JavaScript animations
├── vite.config.js          # Vite configuration
├── package.json            # Node dependencies
├── Dockerfile              # Container configuration (multi-stage build)
├── nginx.conf              # Nginx server config (in container)
├── docker-compose.yml      # Deployment orchestration
├── .env.production         # Production environment
├── .env.staging            # Staging environment
├── Makefile                # Development commands
├── CLAUDE.md               # This file
└── README.md               # Public documentation
```

---

## 🚢 INFRASTRUCTURE

### Local Development
- **Server:** Vite dev server (auto-detects port: 8002 → 5173 → 5174...)
- **Build Tool:** Vite (Hot Module Replacement, fast refresh)
- **Files:** HTML/CSS/JS with ES modules
- **Dev Mode:** `make dev` (Vite only) or `make dev-sst` (with AWS)

### Environment Stages
**Dev** (Local Development)
- Uses `--stage dev` for isolated AWS resources
- No custom domain (uses CloudFront URL)
- Multiple instances can run on different ports
- Clean up: `make remove-dev`

**Staging**
- Custom domain: https://staging.paiss.me
- Deploy: `make deploy-staging`
- Testing environment before production

**Production**
- Custom domain: https://paiss.me
- Deploy: `make deploy-production`
- Retention policy: AWS resources retained on `sst remove`

### Infrastructure (SST)
- **Build:** Vite (builds to `dist/`)
- **Static Hosting:** S3 + CloudFront (CDN)
- **Backend API:** API Gateway + Lambda (contact form)
- **Email:** AWS SES (Simple Email Service)
- **DNS:** Route53 (staging/production only)
- **SSL:** ACM certificates (automatic via SST)
- **Region:** eu-north-1

### SST Architecture
**Modern serverless deployment via [SST v3](https://sst.dev)**

**Infrastructure Components:**
```
SST Resources:
├── StaticSite (S3 + CloudFront)
│   ├── S3 bucket: Static assets (HTML/CSS/JS)
│   ├── CloudFront: Global CDN
│   └── Route53: DNS + SSL certificates
│
└── ApiGatewayV2 (API + Lambda)
    ├── POST /contact → Lambda function
    ├── SES integration for email
    └── CORS configured for paiss.me domains
```

**How Deployment Works:**
1. **Local Build** → Vite builds static assets to `dist/`
2. **SST Deploy** → `make deploy-production` or `make deploy-staging`
3. **Infrastructure** → SST creates/updates AWS resources:
   - S3 bucket for static files
   - CloudFront distribution (CDN)
   - Lambda function for contact form
   - API Gateway endpoint
   - Route53 DNS records
4. **Asset Upload** → Static files uploaded to S3
5. **DNS Propagation** → 1-5 minutes for CloudFront

**Key Files:**
- Website content: `index.html`, `styles.css`, `script.js`
- Infrastructure: `sst.config.ts`
- Lambda handler: `src/lambda/contact.ts`
- Deployment: `Makefile`

---

## 🎨 BRANDING

**PAISS** = **P**ragmatic **A**I **S**oftware **S**olutions

- **Primary Color:** Blue (#2563eb)
- **Secondary Color:** Purple (#8b5cf6)
- **Accent Color:** Cyan (#06b6d4)
- **Font:** Inter

---

## 📊 MONITORING

View application metrics at: https://monitoring.paiss.me

**Alerts configured:**
- Production down (1m duration) - Critical
- High error rate (5m, >5% errors) - Warning

---

## 🚀 DEPLOYMENT WORKFLOW

### Prerequisites
1. **AWS Account**: With appropriate permissions (S3, CloudFront, Lambda, Route53, SES)
2. **AWS CLI**: Configured with credentials (`aws configure`)
3. **SST CLI**: Installed via `npm install` (already in package.json)
4. **Node.js**: Version 18+ for SST and Vite

### Deployment Process
```bash
# 1. Make changes to HTML/CSS/JS

# 2. Test locally with Vite dev server
make dev  # Vite only (fast, no AWS)
# or
make dev-sst  # With AWS resources (tests full stack)

# 3. Commit changes
git add .
git commit -m "Update landing page"
git push origin main  # Recommended for version control

# 4. Deploy to staging via SST
make deploy-staging  # Deploys to https://staging.paiss.me

# 5. Test staging
curl https://staging.paiss.me

# 6. Deploy to production via SST
make deploy-production  # Deploys to https://paiss.me
```

### How It Works
1. **Build** → Vite builds static assets to `dist/`
2. **SST Deploy** → `npm run sst:deploy --stage production`
3. **Infrastructure Update** → SST creates/updates AWS resources:
   - CloudFormation stack
   - S3 bucket (if first deploy)
   - CloudFront distribution (if first deploy)
   - Lambda function (contact form handler)
   - API Gateway endpoint
4. **Asset Upload** → Static files uploaded to S3
5. **CloudFront Invalidation** → Cache cleared for new content
6. **DNS Update** → Route53 records updated (if needed)
7. **Done** → Site live in 2-5 minutes

**Caching Behavior:**
- **S3**: Immediate update
- **CloudFront**: 1-2 minutes (SST invalidates cache automatically)
- **Browser**: Vite content hashing ensures fresh assets
- **DNS**: 1-5 minutes for Route53 propagation

**No Docker** - Static files served directly from S3 via CloudFront
**No SSH/SSM** - Everything managed via AWS SDK
**Serverless** - No servers to maintain, scales automatically

---

**Generated:** 2025-10-09
**Platform:** https://github.com/duersjefen/multi-tenant-platform
**Repository:** https://github.com/duersjefen/paiss-website

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/duersjefen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/duersjefen)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
