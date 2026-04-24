## linux-iac-practice

> 이 파일은 Claude가 이 저장소의 코드를 작업할 때 참고할 지침을 제공합니다.

# CLAUDE.md - 개발 가이드

이 파일은 Claude가 이 저장소의 코드를 작업할 때 참고할 지침을 제공합니다.

---

## 📋 저장소 개요

이는 Linux 기초, 클라우드 인프라, 컨테이너화에 대한 학습 저장소입니다. 주제별로 체계적으로 정리된 실습 자료를 포함합니다.

---

## 📁 저장소 구조

### `apache/`
Azure Virtual Machine에서 Apache 웹 서버 설정 및 배포 예제

**파일:**
- `CREATE_AZURE_VM.md` - Azure VM 생성 및 Apache 설치 가이드
- `DOMAIN.md` - 도메인 설정 및 DNS 학습
- `RSYNC_SSH.md` - rsync 및 SSH를 이용한 파일 동기화
- `etc-apache2/` - Apache 설정 파일 (ports.conf, 가상 호스트)
- `site1/`, `var-www-html/` - 웹 콘텐츠 예제

### `docker/`
두 가지 Docker 학습 경로 제공:

**dockerfile-basic/** - 단일 이미지 빌드 및 Dockerfile 작성
- `springboot-demo/` - Spring Boot (Java 17, Gradle) 멀티스테이지 빌드
  - 특징: linux/amd64 플랫폼 강제 지정 (Apple Silicon 호환성)
- `nestjs-demo/` - NestJS (Node 18) 멀티스테이지 빌드
  - 특징: ESLint, Jest 테스트 포함
- `nginx-demo/` - Nginx로 정적 웹사이트 서빙

**docker-compose/** - 멀티 컨테이너 오케스트레이션
- `nestjs-mysql-demo/` - NestJS + MySQL 데이터베이스 연동
  - 서비스: NestJS API (port 3000), MySQL (port 3306)
- `nginx-demo/` - Nginx 역프록시 설정

### `linux/`
리눅스 명령어, 쉘 스크립트, 시스템 관리 실습

| 디렉토리 | 학습 내용 |
|---------|---------|
| `section3-basic-and-io/` | 기본 명령어, 파일 I/O, 리다이렉션, 파이프 |
| `section4-shell-practice/` | 쉘 스크립트 작성, 변수, 제어 흐름, 백업 스크립트 |
| `section6-process/` | 프로세스 관리, 백그라운드 작업 |
| `section7-user/` | 사용자 및 그룹 관리 |
| `section8-permission/` | 파일 권한 관리 (chmod, chown) |

### `terraform/`
AWS 인프라 코드 (Infrastructure as Code) - ap-northeast-2 리전

| 경로 | 설명 |
|------|------|
| `terraform-basic/` | S3 버킷 생성 기본 예제 |
| `vpc-practice/` | VPC, 퍼블릭/프라이빗 서브넷, IGW, NAT Gateway, 라우트 테이블 |
| `iam-practice/` | IAM 역할 및 정책 관리 |
| `s3-pratice/` | S3 버킷 및 객체 관리 |

**특징:**
- AWS provider는 Seoul 리전(ap-northeast-2) 사용
- VPC는 표준 AWS 패턴 구현: 퍼블릭 서브넷(IGW), 프라이빗 서브넷(NAT Gateway)
- Terraform 상태는 로컬에서 관리됨

---

## 🔧 주요 명령어

### Terraform (AWS ap-northeast-2)
```bash
# VPC 실습
cd terraform/vpc-practice
terraform init          # 초기화
terraform plan          # 실행 계획 확인
terraform apply         # 인프라 생성
terraform destroy       # 인프라 삭제

# 기본 S3 예제
cd terraform/terraform-basic
terraform plan
terraform apply
```

### Docker - Dockerfile 기초

**Spring Boot 빌드**
```bash
cd docker/dockerfile-basic/springboot-demo

# 빌드 및 실행
docker build -t springboot-demo .
docker run -p 8080:8080 springboot-demo

# Gradle 명령어 (로컬)
./gradlew bootJar      # JAR 빌드
./gradlew test         # 단위 테스트
./gradlew bootRun      # 개발 모드 실행
```

**NestJS 빌드**
```bash
cd docker/dockerfile-basic/nestjs-demo

# 빌드 및 실행
docker build -t nestjs-demo .
docker run -p 3000:3000 nestjs-demo

# npm 명령어 (로컬)
npm install
npm run start:dev      # 핫 리로드와 함께 개발 모드
npm run build          # 프로덕션 빌드
npm run lint           # ESLint 실행
npm run test           # Jest 단위 테스트
npm run test:e2e       # E2E 테스트
```

**Nginx 정적 사이트**
```bash
cd docker/dockerfile-basic/nginx-demo

# 빌드 및 실행
docker build -t nginx-demo .
docker run -p 80:80 nginx-demo
```

### Docker Compose (멀티 컨테이너)

**NestJS + MySQL**
```bash
cd docker/docker-compose/nestjs-mysql-demo

# 시작
docker-compose up -d

# 중지
docker-compose down

# 로그 확인
docker-compose logs -f nestjs-api
docker-compose logs -f mysql
```

**Nginx**
```bash
cd docker/docker-compose/nginx-demo

# 시작
docker-compose up -d

# 상태 확인
docker-compose ps

# 중지
docker-compose down
```

### Linux 실습

**쉘 스크립트 실행**
```bash
cd linux/section4-shell-practice

# 백업 스크립트 실행
bash backup.sh

# 권한 부여
chmod +x backup.sh
./backup.sh
```

**프로세스 관리**
```bash
cd linux/section6-process

# 장시간 실행되는 작업 시뮬레이션
bash long_task.sh &

# 프로세스 상태 확인
ps aux | grep long_task
jobs
```

---

## 🏗️ 아키텍처 및 설계 패턴

### Docker 멀티스테이지 빌드
- **목표**: 최종 이미지 크기 최소화
- **구현**: 빌드 단계와 런타임 단계 분리
- **예제**: Spring Boot (Java 17), NestJS (Node 18)

### Terraform VPC 아키텍처
```
┌─────────────────────────────────────────────┐
│              VPC (172.31.0.0/16)            │
├─────────────────────────────────────────────┤
│ Public Subnet    │ Private Subnet           │
│ (IGW attached)   │ (NAT attached)           │
│                  │                         │
│ [EC2 Instance]   │ [RDS/DB]                │
│ [Bastion Host]   │ [Private App]           │
└─────────────────────────────────────────────┘
```

### Docker Compose 네트워킹
- 서비스는 자동으로 내부 DNS 네트워크에 연결됨
- 서비스명으로 접근 가능: `mysql:3306`, `nestjs-api:3000`

---

## 📌 주의사항 및 고려사항

### 플랫폼 호환성
- Spring Boot Dockerfile은 `linux/amd64` 플랫폼 강제 지정
  - Apple Silicon(M1/M2/M3) 사용자를 위해 크로스 플랫폼 빌드 지원
  - 실행 시: `docker build --platform linux/amd64 -t springboot-demo .`

### Terraform
- `terraform-basic/`과 `vpc-practice/`의 상태 파일은 분리됨
- 각 디렉토리에서 독립적으로 `terraform init` 필요
- AWS 자격 증명 설정 필수 (`~/.aws/credentials`)

### Docker Compose
- MySQL 데이터는 named volume에 저장됨
- 초기 데이터베이스 생성은 환경 변수로 설정
- 포트는 호스트 머신에서 예약되므로 충돌 주의

### Linux 실습
- 스크립트는 bash 또는 zsh에서 실행 가능
- `section8-permission/`은 파일 권한 변경을 포함하므로 주의

---

## 📚 주요 학습 목표

### Cloud & Infrastructure
- [ ] Azure Virtual Machine 프로비저닝 및 설정
- [ ] AWS VPC 네트워크 아키텍처 이해
- [ ] Infrastructure as Code (Terraform) 실습
- [ ] 웹 서버 설정 및 도메인 관리

### Containerization
- [ ] Dockerfile 작성 및 멀티스테이지 빌드
- [ ] Docker Compose로 멀티 컨테이너 환경 구성
- [ ] 프레임워크별 최적화 (Spring Boot, NestJS)
- [ ] 데이터베이스 연동 (MySQL)

### Linux & System Administration
- [ ] 기본 명령어 및 쉘 스크립트 작성
- [ ] 파일 시스템 및 권한 관리
- [ ] 프로세스 및 리소스 모니터링
- [ ] 사용자 및 그룹 관리

---

## 🔗 참고 자료

- [AWS Terraform Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Docker Documentation](https://docs.docker.com/)
- [NestJS Documentation](https://docs.nestjs.com/)
- [Linux Man Pages](https://man7.org/linux/man-pages/)

---

## 📝 라이선스

이 저장소는 학습 목적으로 작성되었습니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/krsy0411) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
