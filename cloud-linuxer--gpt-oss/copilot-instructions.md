## gpt-oss

> GPT/LLM deployment project with ArgoCD/GitOps setup for AI model serving.

# CLAUDE.md - Project Configuration for Claude

## Project Overview
GPT/LLM deployment project with ArgoCD/GitOps setup for AI model serving.

## System Specifications
- **CPU**: AMD Ryzen 9 9950X3D 16-Core (32 threads, up to 5.75 GHz)
- **RAM**: 60GB
- **GPU**: NVIDIA GeForce RTX 5090 (32GB VRAM, CUDA 12.9)
- **Storage**: 1.9TB (1.5TB available)
- **OS**: Fedora Linux 42 Server Edition
- **Kernel**: 6.15.10

## Project Structure
```
/home/gpt-oss/
├── backend/       # Backend services
├── frontend/      # Frontend application
├── vllm/         # VLLM configuration
├── models/       # Model storage
├── docker-compose.yml
├── Makefile
└── build-vllm-local.sh
```

## Development Commands
```bash
# Build VLLM locally
./build-vllm-local.sh

# Docker compose operations
docker-compose up -d
docker-compose down

# Make commands
make build
make run
make clean
```

## Linting and Type Checking
```bash
# Backend (if Python)
cd backend && ruff check .
cd backend && mypy .

# Frontend (if Node.js)
cd frontend && npm run lint
cd frontend && npm run typecheck
```

## Testing
```bash
# Backend tests
cd backend && pytest

# Frontend tests  
cd frontend && npm test
```

## Git Workflow
- Main branch: `main`
- Commit message format: Descriptive message focusing on "why"
- Always run linting and tests before committing

## Important Notes
- This is an ArgoCD/GitOps managed project
- Models directory contains AI model files
- VLLM is configured for vector LLM operations
- System has high-end GPU suitable for AI/ML workloads

## Docker Build Optimization 작업 내역

### 성공 사례
1. **의존성 사전 다운로드 스크립트 생성** (download-deps.sh)
   - PyTorch 및 관련 패키지 wheel 파일 사전 다운로드 구현
   - 캐시 디렉토리 구조 설정 (deps/wheels/)

2. **빠른 빌드 스크립트 생성** (build-fast.sh) 
   - 자동으로 의존성 확인 및 다운로드
   - Docker BuildKit 캐시 활용 설정

3. **Python 버전 호환성 문제 해결**
   - Python 3.13 → 3.11 → 3.10 순차적 해결
   - Docker 이미지의 Python 버전 확인 후 맞춤형 wheel 다운로드

4. **PyTorch 버전 충돌 해결**
   - torch 2.5.1과 호환되는 torchaudio 2.5.1, torchvision 0.20.1 버전 매칭
   - CUDA 12.4 버전용 wheel 파일 사용

5. **여러 Dockerfile 변형 생성**
   - Dockerfile.fast: 사전 다운로드 wheel 활용
   - Dockerfile.offline: 간단한 접근 방식
   - Dockerfile.simple: 최소 Python 기반 이미지

### 실패 사례
1. **Python 3.13 wheel 호환성 문제**
   - 원인: 시스템 Python 3.13과 Docker 이미지 Python 버전 불일치
   - 해결: Docker 이미지에 맞는 Python 3.10 wheel 다운로드

2. **PyTorch 컴포넌트 버전 충돌**
   - 원인: torchaudio 2.6.0이 torch 2.6.0 요구하나 2.5.1 설치됨
   - 해결: 모든 컴포넌트를 2.5.1 버전으로 통일

3. **멀티스테이지 빌드 시도**
   - 원인: 사용자가 멀티스테이지 빌드 대신 직접 wheel 사용 요청
   - 해결: 단순한 COPY 방식으로 변경

4. **Docker 이미지 다운로드 속도 문제**
   - NVIDIA CUDA 베이스 이미지 크기: 1.44GB + 2.53GB
   - 여전히 진행 중인 문제, 로컬 캐싱으로 부분 해결

### 추가 발견 사항
5. **vLLM 의존성 충돌**
   - 문제: vLLM이 torch 2.7.1을 요구하여 기존 2.5.1과 충돌
   - 해결: vLLM 설치 시 자동으로 torch 2.7.1로 업그레이드됨
   - 결과: 빌드 성공

### 최종 결과
✅ **빌드 성공!**
- 이미지 이름: `vllm-rtx5090:fast`
- 이미지 크기: 18.9GB
- 빌드 시간: 약 9분 (550.4초)
- 주요 패키지 버전:
  - torch: 2.7.1
  - vllm: 0.10.1.1
  - CUDA: 12.6
  - Python: 3.10

### 빌드 최적화 효과
- 사전 다운로드된 wheel 파일 활용으로 초기 PyTorch 설치 시간 단축
- Docker 레이어 캐싱으로 재빌드 시 속도 향상
- 최종 이미지 크기: 18.9GB (기존 34.4GB 대비 약 45% 감소)

### RTX 5090 호환성 문제
⚠️ **중요한 제한사항**
- 문제: RTX 5090 (Blackwell, sm_120)과 PyTorch 2.7.1 비호환
- 오류 메시지: "NVIDIA GeForce RTX 5090 with CUDA capability sm_120 is not compatible"
- 원인: PyTorch 2.7.1은 현재 sm_50~sm_90까지만 지원

### RTX 5090 해결 시도 현황

#### 1. 환경 변수 우회 시도 (Dockerfile.rtx5090-fix) 🔄
- RTX 5090을 RTX 4090 (sm_89)로 에뮬레이션
- 환경 변수 설정:
  - `TORCH_CUDA_ARCH_LIST="8.0;8.6;8.9;9.0+PTX"`
  - `PYTORCH_NO_CUDA_MEMORY_CACHING=1`
  - `VLLM_WORKER_MULTIPROC_METHOD=spawn`
- vLLM 소스 빌드로 최신 지원 확보
- 상태: 빌드 진행 중

#### 2. PyTorch Nightly 빌드 (Dockerfile.nightly) ❌
- 최신 개발 버전 사용 시도
- 결과: 여전히 sm_120 미지원으로 실패

#### 3. PyTorch 소스 빌드 (Dockerfile.source-build) 🔄
- PyTorch를 소스에서 직접 컴파일
- `TORCH_CUDA_ARCH_LIST="8.0;8.6;8.9;9.0;10.0;12.0+PTX"` 설정
- 가장 포괄적인 해결책이나 빌드 시간 장시간 소요
- 상태: 빌드 진행 중

#### 4. CPU 모드 사용 ❌
- 사용자가 명시적으로 거부 ("cpu 모드는 필요없어")

### 다음 단계
- 진행 중인 빌드 완료 대기 및 테스트
- PyTorch 공식 Blackwell 지원 업데이트 모니터링
- NVIDIA 및 PyTorch 레포지토리 이슈 트래킹

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Cloud-Linuxer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Cloud-Linuxer)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
