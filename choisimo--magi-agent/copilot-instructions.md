## magi-agent

> **루트 디렉토리(/)에 다음 파일들을 생성하는 것을 엄격히 금지:**


# 프로젝트 파일 구조 정리 및 루트 디렉토리 보호 규칙

## 1. 루트 디렉토리 보호 규칙 (CRITICAL)

### 1.1 절대 금지 사항
**루트 디렉토리(/)에 다음 파일들을 생성하는 것을 엄격히 금지:**

❌ **금지된 파일 타입들**
```
- *.md (문서 파일)
- *.log (로그 파일)  
- *.txt (텍스트 파일)
- *.sh (쉘 스크립트)
- *.py (파이썬 스크립트)
- *.js (자바스크립트 파일)
- *.json (설정 파일 - package.json 등 제외)
- *.yml, *.yaml (설정 파일 - docker-compose.yml 등 제외)
- *.csv, *.xlsx (데이터 파일)
- *.sql (SQL 파일)
- *.conf (설정 파일)
- 임시 파일 (temp_*, tmp_*, test_*)
```

### 1.2 루트 디렉토리 허용 파일 (최소한으로 제한)
✅ **허용된 파일들 (필수 프로젝트 파일만)**
```
README.md                 # 프로젝트 메인 설명
package.json              # Node.js 프로젝트 설정
package-lock.json         # 의존성 잠금
Dockerfile               # 도커 이미지 빌드
docker-compose.yml       # 도커 컴포즈 설정
.gitignore              # Git 무시 규칙
.env.example            # 환경변수 예시
requirements.txt        # Python 의존성 (Python 프로젝트)
pom.xml                 # Maven 설정 (Java 프로젝트)
gradle.build            # Gradle 설정 (Java 프로젝트)
LICENSE                 # 라이센스 파일
```

---

## 2. 표준 프로젝트 구조

### 2.1 필수 디렉토리 구조
```
project-root/
├── docs/                    # 📚 문서 전용 폴더
│   ├── api/                # API 문서
│   ├── architecture/       # 아키텍처 문서
│   ├── deployment/         # 배포 관련 문서
│   ├── user-guide/         # 사용자 가이드
│   └── technical/          # 기술 문서
├── scripts/                # 🔧 스크립트 전용 폴더
│   ├── build/              # 빌드 스크립트
│   ├── deploy/             # 배포 스크립트
│   ├── test/               # 테스트 스크립트
│   ├── setup/              # 설치/설정 스크립트
│   └── utils/              # 유틸리티 스크립트
├── logs/                   # 📋 로그 전용 폴더
│   ├── application/        # 애플리케이션 로그
│   ├── access/             # 접근 로그
│   ├── error/              # 에러 로그
│   └── audit/              # 감사 로그
├── config/                 # ⚙️ 설정 파일 전용 폴더
│   ├── environments/       # 환경별 설정
│   │   ├── dev.yml
│   │   ├── staging.yml
│   │   └── prod.yml
│   ├── database/           # DB 설정
│   ├── nginx/              # 웹서버 설정
│   └── monitoring/         # 모니터링 설정
├── data/                   # 📊 데이터 전용 폴더
│   ├── fixtures/           # 테스트 데이터
│   ├── migrations/         # DB 마이그레이션
│   ├── seeds/              # 시드 데이터
│   └── samples/            # 샘플 데이터
├── tests/                  # 🧪 테스트 전용 폴더
│   ├── unit/               # 단위 테스트
│   ├── integration/        # 통합 테스트
│   ├── e2e/                # E2E 테스트
│   ├── performance/        # 성능 테스트
│   └── fixtures/           # 테스트 픽스처
├── tools/                  # 🛠️ 개발 도구
│   ├── generators/         # 코드 생성기
│   ├── validators/         # 검증 도구
│   └── analyzers/          # 분석 도구
└── temp/                   # 🗂️ 임시 파일 (gitignore 처리)
```

---

## 3. 파일 타입별 저장 위치 규칙

### 3.1 문서 파일 (.md, .txt, .pdf)
**저장 위치**: `docs/` 폴더 내 적절한 서브 폴더

```bash
# ✅ 올바른 위치
docs/api/user-api.md
docs/architecture/msa-design.md
docs/deployment/kubernetes-guide.md
docs/technical/database-schema.md

# ❌ 잘못된 위치
README-additional.md          # 루트 금지
user-guide.md                # 루트 금지
api-spec.md                  # 루트 금지
```

### 3.2 스크립트 파일 (.sh, .py, .js)
**저장 위치**: `scripts/` 폴더 내 용도별 서브 폴더

```bash
# ✅ 올바른 위치
scripts/build/build-docker.sh
scripts/deploy/deploy-prod.sh
scripts/test/run-integration-tests.sh
scripts/setup/install-dependencies.py

# ❌ 잘못된 위치
build.sh                     # 루트 금지
deploy.sh                    # 루트 금지
test.py                      # 루트 금지
```

### 3.3 로그 파일 (.log, .out)
**저장 위치**: `logs/` 폴더 내 타입별 서브 폴더

```bash
# ✅ 올바른 위치
logs/application/app.log
logs/error/error-2024-01.log
logs/access/nginx-access.log
logs/audit/security-audit.log

# ❌ 잘못된 위치
app.log                      # 루트 금지
error.log                    # 루트 금지
debug.log                    # 루트 금지
```

### 3.4 설정 파일 (.yml, .json, .conf)
**저장 위치**: `config/` 폴더 내 서비스별 서브 폴더

```bash
# ✅ 올바른 위치
config/environments/dev.yml
config/database/postgres.conf
config/nginx/default.conf
config/monitoring/prometheus.yml

# ❌ 잘못된 위치 (필수 파일 제외)
app-config.yml               # 루트 금지
database.conf                # 루트 금지
```

### 3.5 테스트 파일
**저장 위치**: `tests/` 폴더 내 테스트 타입별 서브 폴더

```bash
# ✅ 올바른 위치
tests/unit/user.test.js
tests/integration/api.test.py
tests/e2e/checkout-flow.spec.js
tests/performance/load-test.sh

# ❌ 잘못된 위치
user.test.js                 # 루트 금지
integration-test.py          # 루트 금지
```

### 3.6 데이터 파일 (.csv, .json, .sql)
**저장 위치**: `data/` 폴더 내 용도별 서브 폴더

```bash
# ✅ 올바른 위치
data/fixtures/users.json
data/migrations/001_create_tables.sql
data/seeds/initial-data.csv
data/samples/test-dataset.json

# ❌ 잘못된 위치
users.json                   # 루트 금지
schema.sql                   # 루트 금지
test-data.csv               # 루트 금지
```

---

## 4. 네이밍 컨벤션

### 4.1 폴더명 규칙
- **소문자 + 하이픈 사용**: `user-service`, `api-gateway`
- **복수형 사용**: `docs`, `scripts`, `tests`, `logs`
- **명확한 의미**: `temp` (X) → `temporary-files` (O)

### 4.2 파일명 규칙
```bash
# 문서 파일
api-specification.md         # kebab-case
user-authentication-guide.md
deployment-procedures.md

# 스크립트 파일
build-docker-images.sh       # kebab-case + 확장자
run-integration-tests.py
deploy-to-production.sh

# 로그 파일
application-YYYY-MM-DD.log   # 날짜 포함
error-service-name.log       # 서비스명 포함
access-nginx-YYYYMMDD.log    # 타임스탬프 포함

# 설정 파일
database-production.yml      # 환경명 포함
nginx-ssl.conf              # 기능명 포함
monitoring-alerts.json       # 용도 명시
```

---

## 5. 자동화된 검증 규칙

### 5.1 Pre-commit Hook 설정
```bash
#!/bin/bash
# .git/hooks/pre-commit

# 루트 디렉토리 파일 검증
ROOT_FILES=$(git diff --cached --name-only | grep -E '^[^/]+\.(md|log|txt|sh|py|js|conf|yml|yaml|csv|sql)$' | grep -v -E '^(README\.md|package\.json|Dockerfile|docker-compose\.yml|\.gitignore|\.env\.example|requirements\.txt|pom\.xml|gradle\.build|LICENSE)$')

if [ ! -z "$ROOT_FILES" ]; then
    echo "❌ ERROR: 루트 디렉토리에 금지된 파일들이 발견되었습니다:"
    echo "$ROOT_FILES"
    echo ""
    echo "다음 위치로 파일을 이동하세요:"
    echo "- 문서 파일 → docs/"
    echo "- 스크립트 → scripts/"
    echo "- 로그 파일 → logs/"
    echo "- 설정 파일 → config/"
    echo "- 테스트 파일 → tests/"
    echo "- 데이터 파일 → data/"
    exit 1
fi
```

### 5.2 CI/CD 파이프라인 검증
```yaml
# .github/workflows/file-structure-check.yml
name: File Structure Validation

on: [push, pull_request]

jobs:
  validate-structure:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Check Root Directory
        run: |
          FORBIDDEN_FILES=$(find . -maxdepth 1 -type f \
            -name "*.md" -o -name "*.log" -o -name "*.txt" \
            -o -name "*.sh" -o -name "*.py" -o -name "*.js" \
            -o -name "*.conf" -o -name "*.yml" -o -name "*.yaml" \
            -o -name "*.csv" -o -name "*.sql" \
            | grep -v -E '^\./(README\.md|package\.json|Dockerfile|docker-compose\.yml|\.gitignore|\.env\.example|requirements\.txt|pom\.xml|gradle\.build|LICENSE)$' || true)
          
          if [ ! -z "$FORBIDDEN_FILES" ]; then
            echo "❌ 루트 디렉토리에 금지된 파일들:"
            echo "$FORBIDDEN_FILES"
            exit 1
          fi
          
      - name: Validate Directory Structure
        run: |
          # 필수 디렉토리 존재 확인
          for dir in docs scripts logs config tests data; do
            if [ ! -d "$dir" ]; then
              echo "❌ 필수 디렉토리 누락: $dir"
              exit 1
            fi
          done
```

---

## 6. 프로젝트 초기화 스크립트

### 6.1 자동 폴더 구조 생성
```bash
#!/bin/bash
# scripts/setup/initialize-project-structure.sh

echo "🚀 프로젝트 구조 초기화 중..."

# 필수 디렉토리 생성
mkdir -p {docs/{api,architecture,deployment,user-guide,technical},scripts/{build,deploy,test,setup,utils},logs/{application,access,error,audit},config/{environments,database,nginx,monitoring},data/{fixtures,migrations,seeds,samples},tests/{unit,integration,e2e,performance,fixtures},tools/{generators,validators,analyzers},temp}

# .gitkeep 파일 생성 (빈 폴더 유지)
find . -type d -empty -exec touch {}/.gitkeep \;

# .gitignore 업데이트
cat >> .gitignore << EOF

# Logs directory
logs/*.log
logs/**/*.log

# Temporary files
temp/*
!temp/.gitkeep

# OS generated files
.DS_Store
Thumbs.db
EOF

echo "✅ 프로젝트 구조 초기화 완료!"
echo "📁 생성된 폴더 구조:"
tree -d -L 2
```

---

## 7. 위반 시 조치사항

### 7.1 즉시 조치 필요
**루트 디렉토리에 금지된 파일 발견 시:**
1. **즉시 커밋 차단** (pre-commit hook)
2. **CI/CD 파이프라인 실패**
3. **PR 병합 차단**
4. **개발자에게 즉시 알림**

### 7.2 수정 절차
```bash
# 1. 잘못 위치한 파일 식별
find . -maxdepth 1 -name "*.md" -o -name "*.log" -o -name "*.sh" | grep -v README.md

# 2. 적절한 위치로 이동
mv *.md docs/
mv *.log logs/application/
mv *.sh scripts/utils/

# 3. 커밋 및 푸시
git add .
git commit -m "refactor: 파일 구조 정리 - 규칙 준수"
git push
```

---

## 8. 모니터링 및 감사

### 8.1 주기적 검증
- **매일**: 자동화된 구조 검증 스크립트 실행
- **주간**: 팀 코드 리뷰 시 구조 준수 확인
- **월간**: 전체 프로젝트 구조 감사

### 8.2 메트릭 수집
```bash
# 구조 준수율 측정 스크립트
#!/bin/bash
# scripts/utils/measure-structure-compliance.sh

TOTAL_FILES=$(find . -type f | wc -l)
MISPLACED_FILES=$(find . -maxdepth 1 -type f -name "*.md" -o -name "*.log" -o -name "*.sh" | grep -v -E "(README\.md|package\.json)" | wc -l)
COMPLIANCE_RATE=$(echo "scale=2; (($TOTAL_FILES - $MISPLACED_FILES) / $TOTAL_FILES) * 100" | bc)

echo "📊 파일 구조 준수율: $COMPLIANCE_RATE%"
echo "📁 전체 파일 수: $TOTAL_FILES"
echo "⚠️ 잘못 위치한 파일 수: $MISPLACED_FILES"
```

---

## 9. 팀 교육 및 가이드라인

### 9.1 신규 팀원 온보딩
1. **구조 규칙 설명서 제공**
2. **실습을 통한 규칙 체험**
3. **IDE 설정 가이드 제공**
4. **자동화 도구 설치 지원**

### 9.2 정기 교육
- **월 1회**: 파일 구조 베스트 프랙티스 세션
- **분기 1회**: 프로젝트 구조 개선 워크샵
- **연 2회**: 전사 개발 표준 교육

---

**중요**: 이 규칙은 프로젝트의 유지보수성과 가독성을 위한 필수 사항입니다. 모든 팀원은 예외 없이 준수해야 하며, 위반 시 즉시 수정 조치를 취해야 합니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/choisimo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
