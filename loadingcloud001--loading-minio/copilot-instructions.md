## loading-minio

> MinIO is a high-performance S3-compatible object storage service that provides file storage capabilities for the CMP Platform.

# MinIO (S3 Object Storage)

## Service Purpose

MinIO is a high-performance S3-compatible object storage service that provides file storage capabilities for the CMP Platform.

### Main Configuration

| Item | Default |
|------|---------|
| Storage | 10Gi (ReadWriteOnce) |
| StorageClass | do-block-storage |
| CPU Request | 100m |
| Memory Request | 256Mi |
| CPU Limit | 500m |
| Memory Limit | 512Mi |

### Health Checks

- Readiness: `GET /minio/health/ready`
- Liveness: `GET /minio/health/live`

## Dependencies

### Consumed By

- **Speckle** - BIM file storage
- **Budibase** - Application attachment storage

### Does Not Depend On

MinIO is an infrastructure service with no dependencies on other CMP services.

## Kubernetes Resources

- Deployment: `minio` (1 replica)
- Service: `minio` (ClusterIP: 9000, 9001)
- PVC: `minio-data` (10Gi)

## Development Standards

This project follows LoadingCloud development standards:

- CI template from [dev-standards](https://github.com/loadingcloud001/dev-standards)
- Commit format: [Conventional Commits](https://github.com/loadingcloud001/dev-standards/blob/main/COMMIT_CONVENTION.md)
- PR template: [PR_TEMPLATE.md](https://github.com/loadingcloud001/dev-standards/blob/main/PR_TEMPLATE.md)
- Secrets encrypted with `age` in [dev-standards](https://github.com/loadingcloud001/dev-standards/blob/main/RESOURCES.md)

## External Resources

- [MinIO Documentation](https://min.io/docs/minio/kubernetes/upstream/index.html)
- [MinIO Kubernetes Operator](https://min.io/docs/minio/kubernetes/upstream/index.html)
- [S3 API Reference](https://docs.min.io/docs/python-client-api-reference.html)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/loadingcloud001)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/loadingcloud001)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
