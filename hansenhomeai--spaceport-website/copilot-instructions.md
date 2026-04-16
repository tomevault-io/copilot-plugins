## spaceport-website

> This is a comprehensive web application with an integrated Gaussian Splatting ML pipeline for 3D reconstruction from images.

# Spaceport Website & ML Pipeline - Cursor Rules

## 🚀 Project Overview
This is a comprehensive web application with an integrated Gaussian Splatting ML pipeline for 3D reconstruction from images.

### Architecture Components:
1. **Frontend**: React-based website with drone path visualization and ML processing interface
2. **Backend**: AWS CDK infrastructure with Lambda functions and API Gateway
3. **ML Pipeline**: Step Functions orchestrating SageMaker jobs for 3D Gaussian Splatting
4. **Infrastructure**: Production-grade AWS services with monitoring and security

## 🏗️ Infrastructure Stack

### AWS CDK Stacks:
- `SpaceportStack`: Main website infrastructure (S3, CloudFront, Lambda, API Gateway)
- `MLPipelineStack`: ML processing infrastructure (Step Functions, SageMaker, ECR)

### Key AWS Services:
- **S3 Buckets**: Website hosting, ML data storage with organized prefixes
- **CloudFront**: Global CDN for website delivery
- **API Gateway**: RESTful API endpoints
- **Lambda**: Serverless functions for backend logic
- **Step Functions**: ML workflow orchestration
- **SageMaker**: ML model training and processing
- **ECR**: Container registry for ML algorithms
- **CloudWatch**: Monitoring, logging, and alerting
- **SES**: Email notifications for ML job completion

## 🤖 ML Pipeline - Gaussian Splatting

### Approved AWS SageMaker Quotas (PRODUCTION READY):
✅ **ml.g5.xlarge for training job usage**: 1 instance
   - **Usage**: 3D Gaussian Splatting Training step
   - **Specs**: 4 vCPUs, 16 GB RAM, 1x NVIDIA A10G GPU
   - **Purpose**: GPU-accelerated neural rendering training (supports gsplat labeled_partition)

✅ **ml.c6i.2xlarge for processing job usage**: 1 instance  
   - **Usage**: SfM Processing (COLMAP) step
   - **Specs**: 8 vCPUs, 16 GB RAM
   - **Purpose**: Structure-from-Motion reconstruction

✅ **ml.c6i.4xlarge for processing job usage**: 2 instances
   - **Usage**: Compression (SOGS) step  
   - **Specs**: 16 vCPUs, 32 GB RAM
   - **Purpose**: Gaussian splat optimization and compression

### ML Pipeline Workflow:
1. **SfM Processing** (COLMAP on ml.c6i.2xlarge)
   - Feature extraction and matching
   - Sparse reconstruction
   - Dense reconstruction
   - Point cloud generation

2. **3DGS Training** (Gaussian Splatting on ml.g4dn.xlarge)
   - Neural rendering training
   - Gaussian splat optimization
   - Model convergence

3. **SOGS Compression** (Self-Organizing Gaussian Splats on ml.g5.xlarge)
   - PlayCanvas SOGS compression algorithm
   - WebP images + metadata.json output
   - GPU-accelerated optimization for web delivery

4. **Notification** (Lambda)
   - Email notifications via SES
   - Job status updates

### Container Images (ECR) - PRODUCTION READY:
- `spaceport/sfm:latest`: COLMAP 3.11.1 Structure-from-Motion ✅
- `spaceport/3dgs:latest`: Optimized 3D Gaussian Splatting training ✅  
- `spaceport/compressor:latest`: PlayCanvas SOGS Self-Organizing Gaussian Splats ✅

### Container Organization Rules:
- **GOLDEN RULE**: One Dockerfile per container directory
- **Build Process**: Use `scripts/deployment/deploy.sh` or GitHub Actions
- **Platform**: Always `--platform linux/amd64` for SageMaker compatibility
- **NO DUPLICATES**: Never create multiple Dockerfiles in same directory

## 📁 Directory Structure

```
/
├── infrastructure/           # AWS CDK infrastructure code
│   ├── spaceport_cdk/       # CDK stack definitions
│   ├── lambda/              # Lambda function code
│   └── containers/          # PRODUCTION ML containers (1 Dockerfile each)
│       ├── sfm/             # COLMAP Structure-from-Motion
│       ├── 3dgs/            # 3D Gaussian Splatting training
│       └── compressor/      # SOGS compression
├── scripts/                 # Build and deployment scripts
│   └── deployment/          # Production deployment scripts
├── docs/                    # ALL documentation files
├── tests/                   # Organized test files
├── assets/                  # Static images and logos
├── [frontend files]         # HTML, CSS, JS for website
└── [config files]           # .cursorrules, .gitignore, buildspec.yml
```

## 🔧 Development Guidelines

### Code Style:
- Use TypeScript for frontend development
- Follow AWS CDK best practices for infrastructure
- Implement proper error handling and logging
- Use least-privilege IAM policies

### Security:
- All S3 buckets have encryption enabled
- IAM roles follow least-privilege principle
- API Gateway has CORS configured
- CloudWatch monitoring for all services

### Deployment:
- GitHub Actions CI/CD automatically deploys CDK on push
- ECR containers must be built and pushed manually after infrastructure deployment
- Use `cdk deploy --all` for full stack deployment

## 🚨 Important Notes

### ML Pipeline Considerations:
- **Cost Optimization**: Use Spot instances where possible for training jobs
- **Monitoring**: CloudWatch alarms configured for job failures and cost thresholds
- **Data Management**: S3 lifecycle policies for automatic cleanup of intermediate data
- **Security**: All ML data encrypted at rest and in transit

### Frontend Features:
- Drone path visualization with 3D trajectory display
- ML processing interface with S3 URL input
- Real-time job status updates
- Responsive design for mobile and desktop

### API Endpoints:
- `/start-job`: Initiates ML pipeline processing
- `/drone-path`: Handles drone trajectory calculations
- All endpoints have proper validation and error handling

## 🎯 Future Development

### Planned Enhancements:
- Real-time progress tracking for ML jobs
- Advanced 3D visualization of Gaussian splats
- Batch processing capabilities
- Cost optimization features

### Performance Targets:
- SfM Processing: < 30 minutes for typical image sets
- 3DGS Training: < 2 hours for convergence
- Compression: < 15 minutes for optimization
- Total pipeline: < 3 hours end-to-end

## 🔍 Debugging & Troubleshooting

### Common Issues:
- Check CloudWatch logs for Lambda and Step Functions
- Verify S3 permissions for cross-service access
- Monitor SageMaker job logs for ML pipeline issues
- Use CloudWatch metrics for performance monitoring

### Key Metrics to Monitor:
- Step Function execution success rate
- SageMaker job duration and cost
- Lambda function cold starts
- S3 data transfer costs

---

**Last Updated**: After AWS quota approvals for production ML pipeline
**Status**: Production Ready 🚀
**Next Steps**: Deploy and test full ML pipeline with approved instance types 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HansenHomeAI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
