## biztalk-gemini-cli

> 이 `GEMINI.md` 파일은 일상적인 언어를 다양한 대상에 맞춰 전문적인 비즈니스 커뮤니케이션으로 변환하는 AI 기반 웹 솔루션인 BizTone Converter 프로젝트에 대한 포괄적인 개요를 제공합니다.

# BizTone Converter 프로젝트 개요

이 `GEMINI.md` 파일은 일상적인 언어를 다양한 대상에 맞춰 전문적인 비즈니스 커뮤니케이션으로 변환하는 AI 기반 웹 솔루션인 BizTone Converter 프로젝트에 대한 포괄적인 개요를 제공합니다.

## 프로젝트 목적
BizTone Converter는 신입사원이나 비즈니스 커뮤니케이션에 익숙하지 않은 사람들을 포함한 사용자들이 상사, 동료, 고객 등 다양한 대상에 맞춰 일상적인 언어를 전문적인 비즈니스 언어로 빠르고 쉽게 변환할 수 있도록 돕는 것을 목표로 합니다. 이는 커뮤니케이션 효율성을 향상시키고, 오해를 줄이며, 개인과 조직의 전문적인 이미지를 강화합니다.

## 주요 기술

### 프론트엔드
-   **HTML5**: 웹 콘텐츠 구조화를 위한 시맨틱 마크업.
-   **CSS3**: 스타일링 및 반응형 레이아웃을 위한 기본 프레임워크로 **Tailwind CSS**를 사용합니다. `tailwind.config.js` 파일에는 커스텀 색상 팔레트와 폰트 패밀리(`Pretendard`, `Noto Sans KR`)가 설정되어 있습니다.
-   **JavaScript (ES6+)**: DOM 조작, 이벤트 처리, `Fetch API`를 사용한 API 호출, 글자 수 계산, UI 피드백(예: 로딩 스피너, 에러 메시지) 관리 등 클라이언트 측 로직을 담당합니다.

### 백엔드
-   **Python 3.11**: 서버의 핵심 프로그래밍 언어.
-   **Flask**: RESTful API 엔드포인트를 제공하는 경량 웹 프레임워크.
-   **Flask-CORS**: 프론트엔드-백엔드 간의 교차 출처 리소스 공유(CORS)를 처리합니다.
-   **python-dotenv**: 환경 변수를 관리하여 API 키와 같은 민감한 정보가 노출되지 않도록 합니다.

### AI 통합
-   **Groq AI API**: 자연어 변환 처리에 활용됩니다. 애플리케이션은 대상별 프롬프트 엔지니어링과 함께 Groq의 채팅 완성 모델(예: "meta-llama/llama-4-scout-17b-16e-instruct" 또는 "mixtral-8x7b-32768")을 활용합니다.

### 개발 도구 및 환경
-   **npm**: 프론트엔드 개발 종속성(Tailwind CSS, PostCSS, Autoprefixer) 관리에 사용됩니다.
-   **pip**: Python 백엔드 종속성 관리에 사용됩니다.
-   **Git/GitHub**: 버전 관리 및 협업 개발에 사용됩니다.
-   **Vercel**: 정적 프론트엔드 파일 및 서버리스 함수 배포를 위한 플랫폼으로 예정되어 있습니다.

## 아키텍처
프로젝트는 프론트엔드와 백엔드 컴포넌트를 명확히 분리하는 모듈형 아키텍처를 따릅니다.
-   **프론트엔드**: 사용자 인터페이스를 제공하는 정적 웹 파일(HTML, CSS, JS).
-   **백엔드**: 비즈니스 로직을 처리하고 Groq AI API와 통신하는 Flask 기반 API 서버.
-   **데이터 흐름**: 프론트엔드의 사용자 입력은 Flask 백엔드의 `/api/convert` 엔드포인트로 POST 요청을 통해 전송됩니다. 백엔드는 이 요청을 처리하고, Groq AI API를 위한 프롬프트를 구성하며, 변환된 텍스트를 받아 프론트엔드로 반환합니다.

## 빌드 및 실행

### 필수 사항
-   Python 3.11 이상
-   Node.js 및 npm
-   프로젝트 루트에 `GROQ_API_KEY`가 설정된 `.env` 파일

### 백엔드 설정 및 실행
1.  `backend` 디렉토리로 이동합니다: `cd backend`
2.  Python 종속성을 설치합니다: `pip install -r requirements.txt`
3.  프로젝트 루트로 돌아갑니다: `cd ..`
4.  Flask 서버를 실행합니다: `python backend/app.py`
    *(참고: PowerShell에서 지속적인 백그라운드 작업을 위해 `Start-Process python -ArgumentList "backend/app.py"`를 사용할 수 있습니다.)*

### 프론트엔드 설정 및 빌드
1.  Node.js 및 npm이 설치되어 있는지 확인합니다.
2.  Node.js 종속성(Tailwind CSS용)을 설치합니다: `npm install`
3.  Tailwind CSS 출력을 빌드합니다 (이 명령은 `frontend/css/style.css`를 생성합니다):
    `npx tailwindcss -i ./frontend/input.css -o ./frontend/css/style.css`
    *(이 명령은 Tailwind 설정 또는 HTML/JS의 유틸리티 클래스가 변경될 때마다 실행해야 합니다.)*

### 전체 애플리케이션 실행
백엔드와 프론트엔드 설정을 완료한 후, Flask 서버가 실행 중인지 확인하고 웹 브라우저에서 `http://127.0.0.1:5000`에 접속하십시오.

## 개발 규칙
-   **문서화**: `PRD.md` 및 `프로그램 개요서.md`는 주요 요구사항 및 개요 문서로 사용됩니다.
-   **환경 변수**: 민감한 API 키(예: `GROQ_API_KEY`)는 `.env` 파일을 통해 관리되며 서버 측에서만 접근 가능하도록 합니다.
-   **버전 관리**: GitHub에서 Pull Request를 통한 코드 리뷰를 필수로 하는 `main`, `develop`, `feature` 브랜치 전략을 따릅니다.
-   **스타일링**: 프론트엔드 스타일링은 `tailwind.config.js`를 통해 커스텀 팔레트 및 폰트가 구성된 Tailwind CSS로 관리됩니다.
-   **오류 처리**: 백엔드는 Groq API 호출 및 일반적인 예외에 대한 강력한 오류 처리 로직과 상세한 로깅을 포함합니다.
-   **UI 피드백**: 프론트엔드는 로딩 상태(스피너), 복사 성공, 오류 메시지에 대한 시각적 피드백을 제공합니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yoojisoo99)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yoojisoo99)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
