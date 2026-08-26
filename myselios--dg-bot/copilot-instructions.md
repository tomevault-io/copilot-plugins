## dg-bot

> **중요**: 모든 문서는 `docs/` 디렉토리 하위에 생성해야 합니다.

# Cursor 개발 규칙

## 📁 파일 및 문서 생성 규칙

### 문서 파일 위치

**중요**: 모든 문서는 `docs/` 디렉토리 하위에 생성해야 합니다.

#### 문서 생성 규칙

1. **README.md 예외**
   - 프로젝트 루트의 `README.md`만 예외
   - 프로젝트 개요 및 빠른 시작 가이드 포함

2. **모든 마크다운 문서**
   - 가이드, 튜토리얼, 설명서: `docs/` 하위에 생성
   - 예시: `docs/DOCKER_SETUP_GUIDE.md`, `docs/API_GUIDE.md`
   
3. **리포트 및 분석 문서**
   - 분석 리포트, 성능 리포트: `docs/reports/` 하위에 생성
   - 예시: `docs/reports/PERFORMANCE_REPORT.md`

4. **아키텍처 및 기술 문서**
   - 시스템 설계 문서: `docs/` 하위에 생성
   - 예시: `docs/ARCHITECTURE.md`, `docs/DATABASE_SCHEMA.md`

5. **사용자 가이드 및 매뉴얼**
   - 사용자 대상 문서: `docs/` 하위에 생성
   - 예시: `docs/USER_GUIDE.md`, `docs/TROUBLESHOOTING.md`

#### 스크립트 파일 위치

1. **개발/배포 스크립트**
   - 프로젝트 루트에 위치 가능
   - 예시: `verify-docker.ps1`, `verify-and-run.bat`, `deploy.sh`
   - 단, 스크립트 설명 문서는 `docs/` 하위에 생성

2. **백테스팅/데이터 수집 스크립트**
   - `scripts/` 디렉토리 하위에 위치
   - 각 스크립트 설명은 해당 디렉토리의 `README.md`에 작성

#### 금지 사항

- ❌ 프로젝트 루트에 가이드 문서 생성 금지 (README.md 제외)
- ❌ 프로젝트 루트에 분석/리포트 문서 생성 금지
- ✅ 모든 문서는 반드시 `docs/` 하위에 생성
- ✅ 문서 생성 시 적절한 하위 디렉토리 사용 (예: reports, guides, api)

#### 디렉토리 구조 예시

```
bitcoin/
├── README.md                    # ✅ 허용 (프로젝트 개요)
├── .cursorrules                 # 개발 규칙
├── verify-docker.ps1            # ✅ 허용 (실행 스크립트)
├── verify-and-run.bat           # ✅ 허용 (실행 스크립트)
│
├── docs/                        # 모든 문서는 여기에
│   ├── ARCHITECTURE.md          # 시스템 아키텍처
│   ├── USER_GUIDE.md            # 사용자 가이드
│   ├── DOCKER_SETUP_GUIDE.md    # 도커 설정 가이드
│   ├── API_REFERENCE.md         # API 레퍼런스
│   ├── TROUBLESHOOTING.md       # 문제 해결 가이드
│   │
│   ├── reports/                 # 분석 리포트
│   │   ├── PERFORMANCE_REPORT.md
│   │   ├── BACKTEST_ANALYSIS.md
│   │   └── TDD_COMPLIANCE_REPORT.md
│   │
│   └── guides/                  # 상세 가이드 (선택적)
│       ├── DEPLOYMENT_GUIDE.md
│       └── DEVELOPMENT_GUIDE.md
│
└── scripts/                     # 스크립트
    └── backtesting/
        └── README.md            # 스크립트 설명
```

## 🔧 스크립트 실행 규칙 (Windows 환경)

### 인코딩 문제 해결

**중요**: Windows 환경에서 한글 경로 및 한글 출력 시 인코딩 문제가 발생할 수 있습니다.

#### PowerShell 스크립트 실행 시 필수 규칙

AI 에이전트가 PowerShell 스크립트를 실행하거나 생성할 때는 **반드시** 다음 규칙을 따릅니다:

1. **스크립트 파일 시작 부분에 인코딩 설정 추가**
   ```powershell
   # UTF-8 인코딩 설정 (필수)
   [Console]::OutputEncoding = [System.Text.Encoding]::UTF8
   [Console]::InputEncoding = [System.Text.Encoding]::UTF8
   $OutputEncoding = [System.Text.Encoding]::UTF8
   $PSDefaultParameterValues['*:Encoding'] = 'utf8'
   chcp 65001 | Out-Null
   ```

2. **기존 스크립트 실행 전 인코딩 설정**
   - 스크립트 실행 전에 `fix-encoding.ps1` 먼저 실행
   - 또는 배치 파일(.bat) 사용을 우선 권장

3. **배치 파일(.bat) 사용 권장**
   - 한글 경로 처리에 더 안정적
   - 파일 시작 부분에 `chcp 65001 >nul` 필수

#### 스크립트 실행 우선순위

AI 에이전트가 스크립트를 실행할 때는 다음 우선순위를 따릅니다:

1. **1순위: 배치 파일 (.bat) 사용**
   ```batch
   @echo off
   chcp 65001 >nul
   
   REM 스크립트 내용...
   ```
   - 예시: `verify-and-run.bat`, `venv\Scripts\activate.bat`

2. **2순위: PowerShell 스크립트 (.ps1) - 인코딩 설정 포함**
   ```powershell
   # UTF-8 인코딩 설정 (필수)
   [Console]::OutputEncoding = [System.Text.Encoding]::UTF8
   $OutputEncoding = [System.Text.Encoding]::UTF8
   chcp 65001 | Out-Null
   
   # 스크립트 내용...
   ```

3. **3순위: Python 스크립트 - venv 사용**
   ```bash
   # Windows (배치 파일 사용)
   .\venv\Scripts\python.exe script.py
   
   # 또는 venv 활성화 후 실행
   .\venv\Scripts\activate.bat
   python script.py
   ```

#### 명령 실행 가이드라인

**❌ 피해야 할 실행 방식:**
```powershell
# PowerShell에서 인코딩 설정 없이 직접 실행 (인코딩 깨질 수 있음)
powershell -Command "Set-Location 'C:\Users\user\OneDrive\문서\git\bitcoin'"
```

**✅ 권장 실행 방식:**
```batch
# 배치 파일 사용
verify-and-run.bat

# 또는 인코딩 수정 후 PowerShell 스크립트 실행
.\fix-encoding.ps1
.\verify-docker.ps1
```

#### 새 스크립트 생성 시 템플릿

**PowerShell 스크립트 템플릿:**
```powershell
<# 
.SYNOPSIS
스크립트 설명

.DESCRIPTION
상세 설명
#>

# UTF-8 인코딩 설정 (필수)
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
[Console]::InputEncoding = [System.Text.Encoding]::UTF8
$OutputEncoding = [System.Text.Encoding]::UTF8
$PSDefaultParameterValues['*:Encoding'] = 'utf8'
chcp 65001 | Out-Null

# 프로젝트 루트 디렉토리 확인
$ProjectRoot = $PSScriptRoot
if (-not $ProjectRoot) {
    $ProjectRoot = Get-Location
}

# 스크립트 로직...
```

**배치 파일 템플릿:**
```batch
@echo off
chcp 65001 >nul
setlocal enabledelayedexpansion

echo ========================================
echo   스크립트 제목
echo ========================================
echo.

REM 스크립트 로직...
```

#### Docker 관련 명령

Docker 명령 실행 시에도 인코딩 고려:

```batch
# 배치 파일 사용 (권장)
verify-and-run.bat

# 또는 인코딩 설정 후 PowerShell 사용
.\fix-encoding.ps1
docker-compose up -d
docker-compose logs -f trading-bot
```

#### 주의사항

- ❌ 인코딩 설정 없이 PowerShell 명령 직접 실행 금지
- ❌ 한글 경로가 포함된 명령을 임시 스크립트 없이 실행 금지
- ✅ 가능한 배치 파일(.bat) 사용 우선
- ✅ PowerShell 스크립트는 반드시 인코딩 설정 포함
- ✅ Python 명령은 venv 환경에서 실행

## 🧪 TDD (Test-Driven Development) 원칙

### 핵심 규칙: 테스트 먼저, 구현 나중

새로운 기능이나 버그 수정을 개발할 때는 반드시 **TDD 사이클**을 따릅니다:

1. **Red (빨강)**: 실패하는 테스트를 먼저 작성
2. **Green (초록)**: 테스트를 통과하는 최소한의 구현 코드 작성
3. **Refactor (리팩토링)**: 코드를 개선하고 최적화

### 테스트 작성 가이드라인

#### 테스트 파일 위치 및 명명 규칙
- 모든 테스트는 `tests/` 디렉토리에 위치
- 테스트 파일명: `test_*.py` 형식
- 테스트 함수명: `test_*` 형식으로 시작
- 테스트 클래스명: `Test*` 형식으로 시작

#### 테스트 구조 예시
```python
"""
모듈 설명
TDD 원칙: 테스트 케이스를 먼저 작성하고 구현을 검증합니다.
"""
import pytest
from src.module_name import ClassName


class TestClassName:
    """ClassName 클래스 테스트"""
    
    @pytest.mark.unit
    def test_method_name(self):
        """메서드 설명"""
        # Given (준비)
        # When (실행)
        # Then (검증)
        assert condition
```

#### 테스트 작성 순서

1. **새 기능 개발 시**:
   - 먼저 `tests/test_module_name.py` 파일에 실패하는 테스트 작성
   - 테스트 실행하여 실패 확인 (Red 단계)
   - `src/module_name.py`에 최소한의 구현 코드 작성
   - 테스트 실행하여 통과 확인 (Green 단계)
   - 코드 리팩토링 (Refactor 단계)

2. **버그 수정 시**:
   - 버그를 재현하는 테스트 케이스 먼저 작성 (실패해야 함)
   - 버그 수정 코드 작성
   - 테스트 통과 확인
   - 기존 테스트도 모두 통과하는지 확인

3. **리팩토링 시**:
   - 기존 테스트가 모두 통과하는 상태에서 시작
   - 코드 개선 후 모든 테스트가 여전히 통과하는지 확인

#### 테스트 작성 우선순위

1. **Unit Test (단위 테스트)** - 최우선
   - 개별 함수/메서드/클래스 단위 테스트
   - `@pytest.mark.unit` 데코레이터 사용

2. **Integration Test (통합 테스트)**
   - 여러 모듈 간 상호작용 테스트
   - `tests/integration/` 디렉토리에 위치
   - `@pytest.mark.integration` 데코레이터 사용

3. **Fixture 사용**
   - 공통 테스트 데이터는 `tests/conftest.py`에 fixture로 정의
   - 재사용 가능한 mock 객체도 fixture로 관리

#### 가상환경 (venv) 사용

**중요**: 이 프로젝트는 Python 가상환경(venv)을 사용합니다.

- 가상환경 위치: `venv/` (프로젝트 루트)
- Windows PowerShell에서 활성화: `.\venv\Scripts\Activate.ps1`
- Windows CMD에서 활성화: `venv\Scripts\activate.bat`
- Linux/Mac에서 활성화: `source venv/bin/activate`

**모든 Python 명령은 반드시 venv 환경에서 실행해야 합니다.**

#### 테스트 실행

```bash
# Windows PowerShell (가상환경 활성화 후)
.\venv\Scripts\Activate.ps1
pytest

# 또는 직접 venv의 pytest 실행
.\venv\Scripts\python.exe -m pytest

# 전체 테스트 실행
.\venv\Scripts\python.exe -m pytest

# 특정 파일 테스트
.\venv\Scripts\python.exe -m pytest tests/test_module_name.py

# 특정 테스트 실행
.\venv\Scripts\python.exe -m pytest tests/test_module_name.py::TestClassName::test_method_name

# 커버리지 포함 실행
.\venv\Scripts\python.exe -m pytest --cov=src --cov-report=html

# 빠른 실행 (상세 출력)
.\venv\Scripts\python.exe -m pytest -v
```

### 개발 워크플로우

1. **기능 요구사항 분석**
2. **테스트 케이스 설계 및 작성** (실패하는 테스트)
3. **최소 구현 코드 작성** (테스트 통과)
4. **코드 리팩토링 및 최적화**
5. **모든 테스트 통과 확인**
6. **커밋 전 전체 테스트 실행**

### 주의사항

- ❌ 구현 코드를 먼저 작성하지 않음
- ❌ 테스트 없이 기능 구현 완료하지 않음
- ✅ 테스트가 실패하는 것을 두려워하지 않음 (예상된 동작)
- ✅ 하나의 테스트 케이스가 하나의 동작만 검증하도록 작성
- ✅ 테스트 이름은 명확하고 구체적으로 작성
- ✅ Given-When-Then 패턴 사용 권장

### 백테스팅 전략 테스트

백테스팅 관련 코드는 특히 TDD가 중요합니다:
- 전략 로직 변경 시 기존 테스트가 실패하면 회귀(Regression) 감지
- `tests/test_rule_based_strategy.py` 참고
- 샘플 데이터는 fixture로 관리 (`sample_chart_data_*`)
- Mock 객체 활용으로 외부 의존성 제거

## 🗑️ 임시 파일 관리 규칙

### 임시 스크립트 및 테스트 파일 삭제

**중요**: 작업 완료 후 임시로 생성한 파일들은 반드시 삭제해야 합니다.

#### 삭제 대상 임시 파일

1. **임시 테스트 스크립트**
   - 테스트 실행용으로 생성한 `.bat` 파일
   - 예시: `run-test.bat`, `test-temp.bat`, `check-env.bat`
   - **예외**: 프로젝트에서 지속적으로 사용하는 스크립트는 유지
     - `verify-docker.ps1`
     - `verify-and-run.bat`
     - `start-scheduler.ps1` / `start-scheduler.sh`
     - `rebuild-scheduler.bat`

2. **임시 설정 파일**
   - 디버깅용 설정 파일
   - 임시 환경변수 파일 (`.env.test`, `.env.temp` 등)

3. **임시 데이터 파일**
   - 테스트 중 생성된 CSV, JSON 파일
   - 캐시 파일 (단, `.gitignore`에 포함된 경우 제외)

#### 삭제 시점

1. **테스트 완료 직후**
   - 단위 테스트 실행 후
   - 통합 테스트 실행 후
   - 작업 검증 완료 후

2. **작업 종료 전**
   - 사용자에게 최종 결과 보고하기 전
   - 다음 작업으로 넘어가기 전

#### 유지해야 할 파일

다음 파일들은 삭제하지 않습니다:

1. **프로덕션 스크립트**
   - 배포 및 실행에 필요한 스크립트
   - 사용자가 직접 사용하는 스크립트

2. **문서 파일**
   - `docs/` 디렉토리의 모든 문서
   - `README.md`

3. **설정 파일**
   - `.cursorrules`
   - `docker-compose.yml`
   - `Dockerfile`
   - `.env.example`

4. **테스트 파일**
   - `tests/` 디렉토리의 pytest 테스트 파일
   - `conftest.py`

#### 삭제 명령 예시

```bash
# 임시 .bat 파일 삭제
delete_file("temp-test.bat")
delete_file("run-quick-test.bat")
delete_file("check-setup.bat")

# 임시 설정 파일 삭제
delete_file(".env.temp")
delete_file("test-config.json")
```

#### 체크리스트

작업 완료 전 확인 사항:

- [ ] 테스트용으로 생성한 임시 `.bat` 파일 삭제 완료
- [ ] 디버깅용 임시 스크립트 삭제 완료
- [ ] 임시 데이터 파일 정리 완료
- [ ] 프로덕션에 필요한 파일은 유지 확인
- [ ] `.gitignore`에 임시 파일 패턴 추가 (필요시)

### 주의사항

- ❌ 사용자가 직접 사용할 스크립트를 삭제하지 않음
- ❌ 문서 파일을 삭제하지 않음
- ❌ 테스트 파일(`tests/` 디렉토리)을 삭제하지 않음
- ✅ 작업 완료 후 반드시 임시 파일 정리
- ✅ 삭제 전 파일 용도 재확인
- ✅ 유지가 필요한 파일은 명확히 구분

---
> Source: [myselios/dg_bot](https://github.com/myselios/dg_bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
