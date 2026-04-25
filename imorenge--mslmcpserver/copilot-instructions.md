## mslmcpserver

> - **필수**: `YYYY-MM-DD` 형식 사용

# VoiceMacro Pro 개발 규칙 및 가이드라인

## 📝 Git 커밋 규칙

### 🏷️ 커밋 이름 형식
```
날짜-기능분류: 간략한 변경사항
```

#### 📅 날짜 형식
- **필수**: `YYYY-MM-DD` 형식 사용
- **예시**: `2025-mm-dd`, `2025-12-25`

#### 🏭 기능분류 태그
| 태그 | 설명 | 예시 |
|------|------|------|
| `feat` | 새로운 기능 추가 | `2025-mm-dd-feat: 음성인식 시스템 추가` |
| `fix` | 버그 수정 | `2025-mm-dd-fix: 매크로 실행 오류 수정` |
| `update` | 기존 기능 개선/수정 | `2025-mm-dd-update: UI 레이아웃 개선` |
| `refactor` | 코드 리팩토링 | `2025-mm-dd-refactor: 데이터베이스 모듈 분리` |
| `docs` | 문서 수정 | `2025-mm-dd-docs: API 문서 업데이트` |
| `style` | 코드 스타일 변경 | `2025-mm-dd-style: 코드 포맷팅 적용` |
| `test` | 테스트 추가/수정 | `2025-mm-dd-test: 단위테스트 추가` |
| `init` | 프로젝트 초기화 | `2025-mm-dd-init: 프로젝트 구조 생성` |

### 📋 커밋 메시지 작성 규칙

#### ✅ 올바른 커밋 예시
```bash
# 커밋 이름: 간략하게
git commit -m "2025-mm-dd-feat: 매크로 관리 UI 추가" -m "
📍 변경된 파일들:
- VoiceMacroPro/Views/MacroManagementWindow.xaml 생성
- VoiceMacroPro/Views/MacroManagementWindow.xaml.cs 생성
- VoiceMacroPro/Models/MacroModel.cs 수정

🔧 주요 변경사항:
- 매크로 목록 표시를 위한 DataGrid 구현
- 매크로 CRUD 기능 버튼 추가 (추가/수정/삭제/복사)
- 매크로 검색 및 필터링 기능 구현
- 매크로 정렬 기능 (이름순/생성일순/사용빈도순)

🎯 구현된 기능:
- 매크로 목록 실시간 업데이트
- 매크로 선택 시 상세정보 표시
- 우클릭 컨텍스트 메뉴 제공

🧪 테스트 상태:
- UI 레이아웃 테스트 완료
- 기본 CRUD 동작 테스트 완료
- 데이터바인딩 동작 확인 완료

⚠️ 알려진 이슈:
- 매크로 삭제 시 확인 대화상자 미구현
- 대량 데이터 로딩 시 성능 최적화 필요
"
```

```bash
# 버그 수정 예시
git commit -m "2025-mm-dd-fix: 데이터베이스 연결 실패 문제 해결" -m "
🐛 문제 상황:
- 앱 시작 시 데이터베이스 파일이 없을 때 크래시 발생
- SQLite 데이터베이스 경로 문제로 connection 실패

🔧 수정 내용:
- database.py에 데이터베이스 파일 존재 여부 확인 로직 추가
- 데이터베이스 파일이 없을 경우 자동 생성 기능 구현
- 데이터베이스 초기화 시 스키마 자동 생성

📍 변경된 파일들:
- database.py: init_database() 메서드 수정
- database.py: _create_initial_schema() 메서드 추가

🧪 테스트 결과:
- 새로운 환경에서 앱 시작 테스트 완료
- 기존 데이터베이스 연결 테스트 완료
- 스키마 생성 및 기본 데이터 삽입 테스트 완료
"
```

#### ❌ 잘못된 커밋 예시
```bash
# ❌ 날짜 누락
git commit -m "feat: UI 추가"

# ❌ 너무 간략한 메시지
git commit -m "2025-mm-dd-fix: 버그수정"

# ❌ 한글 타입 미사용 (가독성 저하)
git commit -m "2025-mm-dd-feat: add macro management" -m "
- Added MacroManagementWindow.xaml
- Modified MacroModel.cs
"

# ❌ 변경사항 설명 부족
git commit -m "2025-mm-dd-update: 코드 수정" -m "파일들 업데이트함"
```

### 📂 커밋 단위 규칙

#### ✅ 올바른 커밋 단위
- **하나의 기능 완성 단위로 커밋**
- **관련된 파일들을 한번에 커밋**
- **동작하는 상태에서 커밋**

```bash
# ✅ 좋은 예: 매크로 추가 기능 완성
git add VoiceMacroPro/Views/AddMacroWindow.xaml
git add VoiceMacroPro/Views/AddMacroWindow.xaml.cs  
git add VoiceMacroPro/Models/MacroModel.cs
git commit -m "2025-mm-dd-feat: 매크로 추가 기능 구현"

# ✅ 좋은 예: 버그 수정 단위
git add database.py
git commit -m "2025-mm-dd-fix: 데이터베이스 연결 오류 수정"
```

#### ❌ 잘못된 커밋 단위  
```bash
# ❌ 나쁜 예: 파일별로 개별 커밋
git add VoiceMacroPro/Views/AddMacroWindow.xaml
git commit -m "2025-mm-dd-feat: XAML 파일 추가"

git add VoiceMacroPro/Views/AddMacroWindow.xaml.cs
git commit -m "2025-mm-dd-feat: C# 파일 추가"

# ❌ 나쁜 예: 동작하지 않는 상태에서 커밋
git add . 
git commit -m "2025-mm-dd-feat: 개발 중인 기능 저장"
```

### 🔄 커밋 히스토리 관리

#### 📜 브랜치 전략
```bash
# 메인 개발 브랜치
main (또는 master)

# 기능별 브랜치 생성 시 (선택사항)
feature/2025-mm-dd-voice-recognition
feature/2025-mm-dd-macro-management
```

#### 🏃‍♂️ 커밋 전 체크리스트
- [ ] 코드가 정상 동작하는가?
- [ ] 컴파일 오류가 없는가?
- [ ] 주석이 적절히 작성되었는가?
- [ ] 커밋 메시지가 규칙에 맞는가?
- [ ] 관련 파일들이 모두 포함되었는가?


---

## 📁 디렉토리 구조 규칙

### 🚫 절대 금지 사항
- **기존 디렉토리 구조 변경 금지**
- **중복 파일 생성 금지** (동일한 기능의 파일을 여러 위치에 생성)
- **무분별한 새 폴더 생성 금지**

### ✅ 디렉토리 사용 규칙

```
📁 Gamesuport/ (프로젝트 루트)
├── 🐍 Python 백엔드 파일들 (루트에 직접 배치)
│   ├── database.py           # 데이터베이스 전용
│   ├── *_service.py          # 비즈니스 로직 전용
│   ├── api_server.py         # API 엔드포인트 전용
│   └── *_utils.py            # 공통 유틸리티 (향후 추가시)
│
└── 📁 VoiceMacroPro/ (C# WPF 전용)
    ├── Models/               # 데이터 모델만
    ├── Services/             # API 통신 및 비즈니스 로직만
    ├── Views/                # UI 윈도우만
    ├── Utils/                # 공통 유틸리티 (향후 추가시)
    └── Resources/            # 리소스 파일 (향후 추가시)
```

---

## 🔄 코드 중복 방지 규칙

### 1. 🐍 Python 백엔드 중복 방지

#### ✅ DO (권장사항)
```python
# ✅ 올바른 예: 공통 기능은 별도 파일로 분리
# common_utils.py (새로 생성할 경우)
def format_datetime(dt):
    """공통 날짜 포맷팅 함수"""
    return dt.strftime("%Y-%m-%d %H:%M:%S")

def validate_required_fields(data, required_fields):
    """공통 필드 검증 함수"""
    for field in required_fields:
        if not data.get(field):
            raise ValueError(f"필수 필드 누락: {field}")

# database.py에서 사용
from common_utils import format_datetime

# macro_service.py에서 사용  
from common_utils import validate_required_fields
```

#### ❌ DON'T (금지사항)
```python
# ❌ 금지: 동일한 함수를 여러 파일에 복사
# database.py
def format_datetime(dt):  # 중복!
    return dt.strftime("%Y-%m-%d %H:%M:%S")

# macro_service.py  
def format_datetime(dt):  # 중복!
    return dt.strftime("%Y-%m-%d %H:%M:%S")
```

### 2. 🖥️ C# WPF 중복 방지

#### ✅ DO (권장사항)
```csharp
// ✅ 올바른 예: 공통 UI 로직은 Utils 클래스로 분리
// Utils/UIHelper.cs (새로 생성할 경우)
public static class UIHelper
{
    /// <summary>
    /// 공통 메시지 박스 표시
    /// </summary>
    public static void ShowError(string message)
    {
        MessageBox.Show(message, "오류", MessageBoxButton.OK, MessageBoxImage.Error);
    }
    
    /// <summary>
    /// 공통 확인 대화상자
    /// </summary>
    public static bool ShowConfirm(string message)
    {
        return MessageBox.Show(message, "확인", MessageBoxButton.YesNo, 
                              MessageBoxImage.Question) == MessageBoxResult.Yes;
    }
}

// MainWindow.xaml.cs에서 사용
UIHelper.ShowError("매크로 로드 실패");

// MacroEditWindow.xaml.cs에서 사용
if (UIHelper.ShowConfirm("변경사항을 저장하시겠습니까?"))
```

#### ❌ DON'T (금지사항)
```csharp
// ❌ 금지: 동일한 UI 로직을 여러 파일에 복사
// MainWindow.xaml.cs
private void ShowError(string message)  // 중복!
{
    MessageBox.Show(message, "오류", MessageBoxButton.OK, MessageBoxImage.Error);
}

// MacroEditWindow.xaml.cs
private void ShowError(string message)  // 중복!
{
    MessageBox.Show(message, "오류", MessageBoxButton.OK, MessageBoxImage.Error);
}
```

---

## 🗄️ 데이터베이스 스키마 규정

### 📋 스키마 설계 원칙

#### 1. 🔑 명명 규칙
```sql
-- ✅ 테이블명: 소문자, 복수형, 언더스코어 사용
CREATE TABLE macros (...)        -- ✅ 올바름
CREATE TABLE presets (...)       -- ✅ 올바름
CREATE TABLE user_settings (...)  -- ✅ 올바름

-- ❌ 금지사항
CREATE TABLE Macro (...)         -- ❌ 대문자 사용
CREATE TABLE macro (...)         -- ❌ 단수형 사용
CREATE TABLE MacroData (...)     -- ❌ CamelCase 사용
```

#### 2. 🏗️ 컬럼 규칙
```sql
-- ✅ 올바른 컬럼 설계
CREATE TABLE macros (
    id INTEGER PRIMARY KEY AUTOINCREMENT,    -- 필수: 기본키
    name TEXT NOT NULL,                      -- 필수: 이름
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,  -- 필수: 생성일
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,  -- 필수: 수정일
    is_active BOOLEAN DEFAULT TRUE,          -- 선택: 상태 플래그
    settings TEXT,                           -- 선택: JSON 데이터
    version INTEGER DEFAULT 1               -- 선택: 버전 관리
);
```

### 📐 스키마 버전 관리

#### ✅ 스키마 변경 규칙
```python
# database.py - 스키마 버전 관리 예시
class DatabaseManager:
    SCHEMA_VERSION = 1  # 현재 스키마 버전
    
    def init_database(self):
        """데이터베이스 초기화 및 마이그레이션"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        # 현재 버전 확인
        current_version = self._get_schema_version(cursor)
        
        if current_version < self.SCHEMA_VERSION:
            self._migrate_schema(cursor, current_version)
        
        conn.commit()
        conn.close()
    
    def _migrate_schema(self, cursor, from_version):
        """스키마 마이그레이션 실행"""
        if from_version == 0:
            # 초기 스키마 생성
            self._create_initial_schema(cursor)
        
        # 향후 버전 업그레이드 로직 추가
        # if from_version == 1:
        #     self._migrate_v1_to_v2(cursor)
```

### 🔒 데이터 무결성 규칙

#### 1. 필수 제약조건
```sql
-- ✅ 모든 테이블에 필수로 포함해야 할 컬럼들
CREATE TABLE [테이블명] (
    id INTEGER PRIMARY KEY AUTOINCREMENT,           -- 기본키 (필수)
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,  -- 생성일 (필수)
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP   -- 수정일 (필수)
);

-- ✅ 외래키 제약조건 설정 (관계가 있는 경우)
CREATE TABLE macro_executions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    macro_id INTEGER NOT NULL,
    executed_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (macro_id) REFERENCES macros(id) ON DELETE CASCADE
);
```

#### 2. 인덱스 규칙
```sql
-- ✅ 검색 성능을 위한 인덱스 생성
CREATE INDEX idx_macros_name ON macros(name);              -- 이름 검색용
CREATE INDEX idx_macros_voice_command ON macros(voice_command);  -- 음성 명령 검색용
CREATE INDEX idx_macros_created_at ON macros(created_at);        -- 날짜 정렬용
CREATE INDEX idx_macros_usage_count ON macros(usage_count);      -- 사용빈도 정렬용
```

---

## 🔧 API 설계 규칙

### 📡 RESTful API 규칙

#### ✅ URL 패턴
```python
# ✅ 올바른 REST API 설계
@app.route('/api/macros', methods=['GET'])           # 목록 조회
@app.route('/api/macros/<int:id>', methods=['GET'])  # 단일 조회
@app.route('/api/macros', methods=['POST'])          # 생성
@app.route('/api/macros/<int:id>', methods=['PUT'])  # 수정
@app.route('/api/macros/<int:id>', methods=['DELETE']) # 삭제

# 특별한 동작은 별도 엔드포인트
@app.route('/api/macros/<int:id>/copy', methods=['POST'])     # 복사
@app.route('/api/macros/<int:id>/execute', methods=['POST'])  # 실행
```

#### ✅ 응답 형식 통일
```python
# ✅ 모든 API 응답은 동일한 형식 사용
def create_api_response(success=True, data=None, message="", error=None):
    """통일된 API 응답 형식"""
    return {
        'success': success,
        'data': data,
        'message': message,
        'error': error,
        'timestamp': datetime.now().isoformat()
    }

# 사용 예시
return jsonify(create_api_response(
    success=True,
    data=macros,
    message="매크로 목록 조회 성공"
)), 200
```

---

## 📝 코드 문서화 규칙

### 🐍 Python 문서화
```python
def create_macro(self, name: str, voice_command: str, action_type: str, 
                key_sequence: str, settings: Dict = None) -> int:
    """
    새로운 매크로를 생성하는 함수
    
    Args:
        name (str): 매크로 이름 - 고유해야 함
        voice_command (str): 음성 명령어 - 실제 인식될 단어
        action_type (str): 동작 타입 - combo, rapid, hold, toggle, repeat 중 하나
        key_sequence (str): 키 시퀀스 - 실행될 키보드 입력
        settings (Dict, optional): 추가 설정 정보. Defaults to None.
    
    Returns:
        int: 생성된 매크로의 ID
        
    Raises:
        ValueError: 필수 매개변수가 누락된 경우
        DatabaseError: 데이터베이스 저장 실패시
        
    Example:
        >>> macro_id = service.create_macro(
        ...     name="공격 콤보",
        ...     voice_command="공격",
        ...     action_type="combo",
        ...     key_sequence="Q,W,E",
        ...     settings={"delay": 100}
        ... )
        >>> print(macro_id)  # 1
    """
```

### 🖥️ C# 문서화
```csharp
/// <summary>
/// 매크로 목록을 서버에서 불러와 DataGrid에 표시하는 함수
/// UI 스레드에서 실행되며 오류 발생 시 사용자에게 메시지를 표시합니다.
/// </summary>
/// <param name="showLoading">로딩 메시지 표시 여부 (기본값: true)</param>
/// <returns>비동기 작업을 나타내는 Task</returns>
/// <exception cref="HttpRequestException">서버 통신 실패 시 발생</exception>
/// <exception cref="JsonException">JSON 파싱 실패 시 발생</exception>
/// <example>
/// <code>
/// // 기본 사용법
/// await LoadMacros();
/// 
/// // 로딩 메시지 없이 실행
/// await LoadMacros(showLoading: false);
/// </code>
/// </example>
private async Task LoadMacros(bool showLoading = true)
```

---

## 🧪 테스트 규칙

### 📂 테스트 파일 구조
```
📁 Tests/ 
├── 📁 Python/
│   ├── test_database.py      # 데이터베이스 테스트
│   ├── test_macro_service.py # 비즈니스 로직 테스트
│   └── test_api_server.py    # API 테스트
└── 📁 CSharp/
    ├── ModelTests/           # 모델 테스트
    ├── ServiceTests/         # 서비스 테스트
    └── UITests/              # UI 테스트
```

---

## 🔍 코드 리뷰 체크리스트

### ✅ 필수 확인 사항
1. **중복 코드 없음** - 동일한 기능이 여러 곳에 구현되지 않았는가?
2. **명명 규칙 준수** - 함수, 변수, 클래스명이 일관성 있게 작성되었는가?
3. **문서화 완료** - 모든 public 함수에 적절한 주석이 있는가?
4. **오류 처리** - 예외 상황에 대한 적절한 처리가 있는가?
5. **리소스 관리** - 데이터베이스 연결, 파일 핸들 등이 올바르게 해제되는가?

### ⚠️ 주의 사항
1. **하드코딩 금지** - 설정값은 별도 파일로 분리
2. **SQL 인젝션 방지** - 매개변수화된 쿼리 사용 필수
3. **동시성 고려** - 다중 사용자 환경에서의 데이터 일관성
4. **성능 최적화** - 불필요한 데이터베이스 쿼리 최소화

---

## 📋 체크리스트 템플릿

### 새 기능 추가시 확인사항
- [ ] 기존 디렉토리 구조 유지
- [ ] 중복 코드 없음
- [ ] 데이터베이스 스키마 규칙 준수
- [ ] API 응답 형식 통일
- [ ] 적절한 문서화 완료
- [ ] 오류 처리 구현
- [ ] 테스트 케이스 작성 (권장)

### 앱 실행시 커맨드 준수 
- python 실행시 커맨드는 "py example.py" 사용 do not "python example.py"
- 애플리케이션 실행 및 테스트 시 우선 "cd voicemacropro" 커맨드로 디렉토리 선택후 "dotnet run" 커맨드 실행 
- 모든 커맨드 실행시 "&&"기호 사용금지 각각의 커맨드를 분리해서 사용할것 


이 규칙들을 준수하면 코드 품질과 유지보수성이 크게 향상되며, 팀 협업 시 일관성을 유지할 수 있습니다! 😊

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ImOrenge) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
