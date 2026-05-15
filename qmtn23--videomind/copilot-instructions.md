## videomind

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This project is an enterprise-grade AI knowledge management system built with RAG (Retrieval-Augmented Generation) technology. It provides intelligent document processing and retrieval capabilities using a modern tech stack including Spring Boot, Vue 3, Elasticsearch, and AI services.

## Development Environment Setup

### Prerequisites
- Java 17
- Maven 3.8.6+
- Node.js 18.20.0+
- pnpm 8.7.0+
- Docker Desktop (用于运行所有基础服务)

---

## 已验证的本机启动流程（Windows）

> 本节记录了在此机器上实际调试通过的启动步骤，遇到问题请优先参考这里。

### 环境特殊说明

- **本机已有 MySQL 服务（MySQL80）运行在 3306 端口**，无管理员权限无法停止
- 因此 Docker MySQL 端口映射已改为 **13306:3306**，`application.yml` 已对应修改
- `application.yml` 中所有服务密码已与 `docs/docker-compose.yaml` 对齐（密码均为 `PaiSmart2025`）
- AI API 使用**阿里 DashScope**，key 已配置（`deepseek` 配置块复用，URL 指向 DashScope 兼容接口）
- **视频解析为必选能力**：须配置 **`asr.api.key`**、本机 **FFmpeg**，以及 **`minio.publicUrl` 公网可达**（本地常用 ngrok 暴露 MinIO **19000**）；否则视频无法上传或解析

### 第一步：启动 Docker 基础服务

在 PowerShell 中执行：

```powershell
cd "D:\idea-workspace\VideoMind\docs"
docker compose up -d
```

验证所有容器正常：

```powershell
docker ps
```

应看到 5 个容器：`mysql`、`redis`、`kafka`、`es`、`minio` 均为 `Up` 状态。

**首次启动需创建数据库（只需执行一次）：**

```powershell
docker exec mysql mysql -uroot -pPaiSmart2025 -e "CREATE DATABASE IF NOT EXISTS PaiSmart CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

**首次启动需创建 MinIO bucket（只需操作一次）：**

打开浏览器访问 `http://localhost:19001`，用 `admin` / `PaiSmart2025` 登录，创建名为 `uploads` 的 bucket。

### 第二步：启动后端

```powershell
cd "D:\idea-workspace\VideoMind"
mvn spring-boot:run -Dspring-boot.run.profiles=local
```

启动成功标志：日志出现 `Started VideoMindApplication`，端口 `8081`。

> `local` profile 会加载 `application-local.yml`（含真实密钥和本机端口 13306），该文件已加入 `.gitignore` 不会提交到 GitHub。

### 第三步：启动前端

新开终端窗口：

```powershell
cd "D:\idea-workspace\VideoMind\frontend"
pnpm dev
```

启动成功后访问：`http://localhost:9527`（若端口被占用会自动使用 9528）

默认管理员账号：`admin` / `admin123`

### 常见问题

| 问题 | 原因 | 解决方法 |
|------|------|----------|
| 后端报 `Access denied for user 'root'` | 连接到了本机 MySQL 而非 Docker | 确认 `application.yml` 中端口为 `13306` |
| MinIO 上传报 `bucket does not exist` | bucket 未创建 | 访问 `http://localhost:19001` 创建 `uploads` bucket |
| 前端 `pnpm` 命令未找到 | pnpm 未安装 | 执行 `npm install -g pnpm` |
| Docker MySQL 启动失败 `port in use` | 本机 MySQL 占用 3306 | docker-compose.yaml 已改为 13306，无需处理 |
| 视频上传被拒 / ASR 未就绪 | 未配 `asr.api.key`、无 FFmpeg 或 `minio.publicUrl` 非公网 | 配置 `application-local.yml` 中 asr、ffmpeg、`minio.publicUrl`（ngrok）；`asr.enabled` 默认 true |

---

### Quick Start with Docker（原始说明）
```bash
# Start all services
cd docs && docker-compose up -d

# Backend
mvn spring-boot:run

# Frontend
cd frontend && pnpm install && pnpm dev
```

## Common Development Commands

### Backend (Spring Boot)
```bash
# Run application
mvn spring-boot:run

# Build
mvn clean package

# Test
mvn test

# Run with specific profile
mvn spring-boot:run -Dspring-boot.run.profiles=dev
```

### Frontend (Vue 3 + TypeScript)
```bash
# Install dependencies
cd frontend && pnpm install

# Development server
pnpm dev

# Build for production
pnpm build

# Type checking
pnpm typecheck

# Linting
pnpm lint

# Preview build
pnpm preview
```

## Architecture Overview

### Backend Structure
```
src/main/java/com/qmtn/videomind/
├── VideoMindApplication.java     # Main application entry
├── client/                       # External API clients (DeepSeek, Embedding)
├── config/                       # Configuration classes (Security, JWT, etc.)
├── consumer/                     # Kafka consumers for async processing
├── controller/                   # REST API endpoints
├── entity/                       # JPA entities
├── exception/                    # Custom exceptions
├── handler/                      # WebSocket handlers
├── model/                        # Domain models
├── repository/                   # Data access layer
├── service/                      # Business logic layer
└── utils/                        # Utility classes
```

### Frontend Structure
```
frontend/src/
├── assets/                       # Static assets (SVG, images)
├── components/                   # Reusable Vue components
├── layouts/                      # Page layouts
├── router/                       # Vue Router configuration
├── service/                      # API integration
├── store/                        # Pinia state management
├── views/                        # Page components
└── utils/                        # Utility functions
```

## Key Components

### Core Services
- **DocumentService**: Handles document upload, parsing, and management
- **ElasticsearchService**: Manages document indexing and search
- **VectorizationService**: Converts text to embeddings using AI models
- **ChatHandler**: Processes AI chat interactions with RAG
- **UserService**: User authentication and management
- **ConversationService**: Chat history and session management

### AI Integration
- **DeepSeek API**: Primary LLM for chat responses
- **Embedding API**: Text-embedding-v4 for document vectorization
- **RAG Pipeline**: Document → Chunk → Embedding → Search → Response

### Multi-tenant Architecture
- **Organization Tags**: Supports multi-tenant isolation
- **Permission System**: Public/private document access control
- **User-Organization Mapping**: Flexible user-to-org relationships

## Configuration Files

### Backend Configuration
- `application.yml`: Main configuration with database, Redis, Kafka, AI services
- `application-dev.yml`: Development-specific settings
- `application-docker.yml`: Docker deployment settings

### Frontend Configuration
- `vite.config.ts`: Vite build configuration
- `tsconfig.json`: TypeScript configuration
- `pnpm-workspace.yaml`: Workspace configuration for monorepo

## Database Schema

The application uses MySQL as the primary database with JPA/Hibernate for ORM. Key entities include:
- `User`: User accounts and authentication
- `FileUpload`: Document metadata and storage info
- `Conversation`: Chat sessions and history
- `OrganizationTag`: Multi-tenant organization structure
- `ChunkInfo`: Document chunks for vector search

## External Dependencies

### Services
- **Elasticsearch 8.10.0**: Document search and vector storage
- **Kafka 3.2.1**: Message queue for async file processing
- **Redis 7.0.11**: Caching and session management
- **MinIO 8.5.12**: File storage service
- **MySQL 8.0**: Primary database

### AI Services
- **DeepSeek API**: LLM for generating responses
- **DashScope Embedding**: Text-embedding-v4 for document vectorization

## Development Workflow

### Adding New Features
1. Backend: Create entity → repository → service → controller
2. Frontend: Create API service → store module → Vue component → router configuration
3. Update database schema if needed (JPA auto-generates DDL)
4. Test with both unit and integration tests

### API Development
- Follow RESTful conventions
- Use proper HTTP status codes
- Implement authentication/authorization via JWT
- Add request validation and error handling

### Frontend Development
- Use Vue 3 Composition API with TypeScript
- Follow established component patterns in `/src/components/`
- Use Pinia for state management
- Implement proper TypeScript types for API responses

## Testing

### Backend Testing
```bash
# Run all tests
mvn test

# Run specific test class
mvn test -Dtest=UserServiceTest

# Run with coverage
mvn clean verify
```

### Frontend Testing
```bash
# Type checking
pnpm typecheck

# Linting
pnpm lint

# Build verification
pnpm build
```

## Deployment

### Docker Deployment
```bash
# Build backend
mvn clean package

# Build frontend
cd frontend && pnpm build

# Start services
cd docs && docker-compose up -d
```

### Environment Variables
Key configuration variables that should be set in production:
- `JWT_SECRET_KEY`: JWT signing secret
- `DEEPSEEK_API_KEY`: DeepSeek API credentials
- `EMBEDDING_API_KEY`: Embedding service API key
- Database credentials and service URLs

## Security Considerations

- JWT-based authentication with Spring Security
- Role-based access control (admin/user)
- Organization-level data isolation
- File upload validation and size limits
- CORS configuration for frontend integration
- Input validation and sanitization

## Performance Considerations

- Elasticsearch for efficient document search
- Redis caching for frequently accessed data
- Kafka for asynchronous file processing
- File chunking for large document processing
- Vector embeddings for semantic search
- Connection pooling for database and external services

---
> Source: [qmtn23/VideoMind](https://github.com/qmtn23/VideoMind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
