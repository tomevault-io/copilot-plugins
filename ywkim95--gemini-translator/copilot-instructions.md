## gemini-translator

> Gemini 1.5 Flash 모델을 사용하여 현재 웹 페이지의 콘텐츠를 요약하고 한국어로 번역하는 크롬 확장 프로그램을 개발합니다. 이 프로그램은 개인 사용을 목적으로 하며, 효율적인 API 사용과 직관적인 사용자 경험에 최우선 순위를 둡니다.

## **[프로젝트 목표] Gemini 기반 '사이드킥 요약/번역' 크롬 확장 프로그램 개발**

Gemini 1.5 Flash 모델을 사용하여 현재 웹 페이지의 콘텐츠를 요약하고 한국어로 번역하는 크롬 확장 프로그램을 개발합니다. 이 프로그램은 개인 사용을 목적으로 하며, 효율적인 API 사용과 직관적인 사용자 경험에 최우선 순위를 둡니다.

### **핵심 기술 스택**

*   **프론트엔드:** Chrome Extension APIs (Manifest V3), JavaScript, CSS
*   **AI 모델:** Google Gemini 1.5 Flash
*   **외부 라이브러리:** Mozilla Readability.js, Showdown.js

### **Agent 작업 원칙**

*   메모리에 답변하기 이전에 로컬에서 테스트 및 검증 과정을 거친다.

---

### **개발 단계별 지침**

#### **Phase 1: 프로젝트 구조 및 파일 생성**

*   **폴더 구조:**
    ```
    /Sidekick-Translator
    |-- manifest.json               # 확장 프로그램의 청사진
    |-- background.js               # 백그라운드 로직 담당
    |-- options.html                # API 키 입력 설정 페이지 UI
    |-- options.js                  # API 키 입력 설정 페이지 로직
    |
    |-- /scripts
    |   |-- content_script.js       # 웹페이지에 직접 주입될 스크립트
    |   |-- readability.js          # 본문 추출을 위한 외부 라이브러리
    |   |-- showdown.js             # 마크다운 렌더링을 위한 외부 라이브러리
    |
    |-- /ui
    |   |-- sidebar.css             # 사이드바의 모든 스타일 정의
    |
    |-- /icons
        |-- icon16.png
        |-- icon48.png
        |-- icon128.png
    ```
*   **작업:**
    1.  `Readability.js` 라이브러리를 다운로드하여 `/scripts`에 저장합니다.
    2.  `Showdown.js` 라이브러리를 다운로드하여 `/scripts`에 저장합니다.
    3.  임시 아이콘 파일 (16x16, 48x48, 128x128)을 생성하여 `/icons`에 저장합니다.

#### **Phase 2: 프론트엔드 개발 (Chrome Extension)**

*   **`manifest.json`:**
    *   `manifest_version`: 3
    *   `name`, `version`, `description` 설정
    *   `permissions`: `activeTab`, `scripting`, `storage`
    *   `host_permissions`: `<all_urls>`
    *   `action`: `default_title` 설정
    *   `background`: `background.js`를 service worker로 지정
    *   `icons` 설정
    *   `web_accessible_resources`: `readability.js`, `showdown.js`, `ui/sidebar.css` 접근 허용
    *   **`options_page`**: `options.html`로 설정

*   **`background.js` (총괄):**
    1.  **아이콘 클릭 리스너:** `chrome.action.onClicked.addListener`를 사용하여 툴바 아이콘 클릭 이벤트를 감지합니다.
    2.  **상태 관리:** `chrome.storage.local`을 사용하여 탭별로 사이드바의 열림/닫힘 상태(`isSidebarOpen`)를 관리합니다.
    3.  **스크립트 주입/실행:**
        *   클릭 시, 해당 탭의 `isSidebarOpen` 상태를 확인합니다.
        *   만약 `false`이거나 없으면, `chrome.scripting.executeScript`를 사용하여 `readability.js`, `showdown.js`, `content_script.js`를 순서대로 주입하고 상태를 `true`로 변경합니다.
        *   만약 `true`이면, `chrome.tabs.sendMessage`를 통해 `content_script.js`에 `{ type: "TOGGLE_SIDEBAR" }` 메시지를 보내 사이드바를 닫게 하고, 상태를 `false`로 변경합니다.
    4.  **API 통신 중개 (직접 Gemini API 호출):**
        *   `chrome.runtime.onMessage.addListener`를 통해 `content_script.js`로부터 `{ type: "ANALYZE_PAGE", text: "..." }` 메시지를 수신합니다.
        *   메시지 수신 시, `chrome.storage.sync`에서 저장된 `GEMINI_API_KEY`를 가져옵니다.
        *   API 키가 없으면, 사용자에게 설정 페이지로 이동하도록 안내하는 에러 메시지를 `content_script.js`에 전달합니다.
        *   API 키가 있으면, Google Gemini API (`https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=YOUR_API_KEY`)로 `POST` 요청을 보냅니다.
            *   **Request Header:** `Content-Type: 'application/json'`
            *   **Request Body:** 마스터 AI 프롬프트와 `message.text`를 결합한 JSON
        *   **성공 시:** Gemini API로부터 받은 `data`를 `{ type: "DISPLAY_RESULTS", payload: data }` 형태로 `content_script.js`에 전달합니다.
        *   **실패 시:** 에러를 콘솔에 기록하고, `{ type: "DISPLAY_ERROR", payload: { message: "API 호출에 실패했습니다." } }` 형태로 `content_script.js`에 전달합니다.

*   **`content_script.js` (현장):**
    1.  **중복 실행 방지:** `if (window.hasRun)` 플래그 사용.
    2.  **사이드바 생성/제거 함수 (`toggleSidebar`) 구현:**
        *   페이지에 `id="sidekick-translator-root"` 요소가 있는지 확인합니다.
        *   없으면, 페이지 오른쪽에 사이드바 UI 전체를 생성하는 `createSidebar` 함수를 호출합니다.
        *   있으면, 해당 요소를 DOM에서 완전히 제거합니다.
    3.  **`createSidebar` 함수 상세 로직:**
        *   최상위 컨테이너 `<div id="sidekick-translator-root">`를 생성하고 `fixed` 포지션으로 화면 오른쪽 전체 높이에 고정합니다.
        *   내부에 `<iframe>`을 생성하여 CSS 충돌을 원천적으로 방지하고, `iframe`의 너비와 높이를 100%로 설정합니다.
        *   사이드바 상단에 너비 조절 버튼(`Small`, `Medium`, `Large`)을 추가하고 클릭 이벤트 리스너를 등록하여 `sidebarRoot.style.width`를 조절합니다. (최소 300px, 최대 1000px)
        *   생성된 `div`를 `document.body`에 추가하고, iframe 내부에 사이드바 HTML 구조를 동적으로 삽입합니다.
    4.  **본문 추출 및 API 요청:**
        *   사이드바가 생성된 직후, `Readability.js`를 사용하여 `document.cloneNode(true)`로부터 본문을 추출합니다.
        *   추출된 본문의 `textContent`를 `background.js`로 `{ type: "ANALYZE_PAGE", text: ... }` 메시지와 함께 전송합니다.
    5.  **결과/에러 수신 및 표시 로직:**
        *   `chrome.runtime.onMessage.addListener`를 통해 `background.js`의 메시지를 수신합니다.
        *   **`DISPLAY_RESULTS` 타입 수신 시:** 로딩 상태를 숨기고 결과 상태를 표시합니다. `Showdown.js`를 사용하여 `payload.summary`와 `payload.translated_text`를 마크다운에서 HTML로 변환하여 삽입합니다.
        *   **`DISPLAY_ERROR` 타입 수신 시:** 로딩 상태를 숨기고 에러 상태를 표시합니다. `payload.message`를 삽입합니다.
        *   **`TOGGLE_SIDEBAR` 타입 수신 시:** `toggleSidebar` 함수를 호출하여 자신(사이드바)을 닫습니다.

*   **`ui/sidebar.css`:**
    *   사이드바(`iframe` root)를 화면 오른쪽에 고정하고, `min-width: 300px;`, `max-width: 1000px;`를 적용합니다.
    *   가독성 높은 폰트와 깔끔한 UI/UX 디자인.
    *   CSS 애니메이션으로 로딩 스피너 구현.
    *   요약 및 번역 내용(`st-summary`, `st-translation`)은 `text-align: left;`로 설정합니다.
    *   너비 조절 버튼들의 스타일을 정의합니다.

#### **Phase 3: API 키 관리 및 직접 API 호출**

*   **`options.html` (API 키 설정 페이지 UI):**
    *   사용자가 Gemini API 키를 입력하고 저장할 수 있는 간단한 HTML 폼을 제공합니다.
    *   입력 필드와 저장 버튼, 저장 성공/실패 메시지 표시 영역을 포함합니다.

*   **`options.js` (API 키 설정 페이지 로직):**
    *   `options.html`의 폼 요소를 제어합니다.
    *   페이지 로드 시 `chrome.storage.sync`에서 기존에 저장된 API 키를 불러와 입력 필드에 표시합니다.
    *   저장 버튼 클릭 시, 입력된 API 키를 `chrome.storage.sync`에 저장합니다.
    *   저장 성공/실패 피드백을 사용자에게 제공합니다.

*   **마스터 AI 프롬프트 (background.js 내부에 포함):**
    ```text
    # 페르소나 (Persona)
    당신은 고도로 숙련된 정보 분석가이자 전문 번역가입니다. 당신의 임무는 사용자가 제공한 영문 텍스트의 핵심을 빠르고 정확하게 파악하여 명료한 한국어 요약문을 생성하고, 원문의 뉘앙스를 최대한 살리면서 자연스러운 한국어로 전체를 번역하는 것입니다.

    # 지시사항 (Instruction)
    아래 "처리할 텍스트" 부분에 제공된 내용을 분석하여, 다음 두 가지 과업을 수행해주십시오.
    당신의 응답은 반드시 지정된 JSON 형식만을 포함해야 합니다. JSON 객체 외의 다른 설명, 인사, 추가 텍스트를 절대로 포함해서는 안 됩니다.

    ## 최종 출력 JSON 형식
    {
      "summary": "이곳에는 텍스트의 핵심 주제와 결론을 담은 3~5개의 간결한 한국어 문장으로 구성된 **마크다운 형식의 글머리 기호 목록**을 넣어주세요.",
      "translated_text": "이곳에는 '처리할 텍스트'의 전체 내용을 문단 구조를 유지하며 자연스러운 한국어로 번역한 결과를 **마크다운 형식**으로 넣어주세요. 원본의 제목, 부제목, 목록 등도 마크다운 문법으로 표현해주세요."
    }
    ```

    # 예외 처리 규정 (Edge Case Rules)
    1.  만약 "처리할 텍스트"의 내용이 3문장 미만으로 너무 짧아 유의미한 요약이 불가능할 경우, "summary" 키의 값으로 "요약하기에는 텍스트가 너무 짧습니다."를 반환하세요.
    2.  만약 "처리할 텍스트"의 내용이 분석 불가능한 문자(예: 깨진 인코딩, 무작위 문자열)로 판단될 경우, "summary"와 "translated_text" 키의 값 모두에 "분석할 수 없는 콘텐츠입니다."를 반환하세요.

    # 처리할 텍스트 (Text to Process)

    {{사용자가 보고 있는 웹페이지에서 추출된 본문 텍스트가 이곳에 삽입됩니다.}}

---

### **최종 테스트 및 개인용 배포**

1.  **API 키 설정:** 확장 프로그램 설치 후, 확장 프로그램 아이콘을 우클릭하여 '옵션' 페이지로 이동한 뒤 Gemini API 키를 입력하고 저장합니다.
2.  **통합 테스트:**
    *   다양한 실제 웹사이트(뉴스, 블로그 등)를 방문하여 확장 프로그램의 전체 기능(요약, 번역, UI)을 테스트하고 오류를 확인합니다.
3.  **개인용 배포:**
    *   `chrome://extensions` 페이지에서 '개발자 모드'를 활성화합니다.
    *   '압축 해제된 확장 프로그램을 로드합니다'를 클릭하여 `Sidekick-Translator` 폴더를 선택하고 설치합니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ywkim95)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ywkim95)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
