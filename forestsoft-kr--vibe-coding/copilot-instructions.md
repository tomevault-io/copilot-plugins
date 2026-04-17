## vibe-coding

> 본 프로젝트는 SSAFY 브랜드의 AI 챗봇 서비스를 구축하는 것을 목표로 합니다. 사용자는 직관적인 UI/UX 환경에서 로그인, 회원가입을 진행하고, AI 챗봇과의 실시간 채팅을 통해 질문을 주고받을 수 있습니다. 챗봇은 OpenAI의 `gpt-5-nano` 모델을 활용하여 응답을 생성합니다.

# 프로젝트: SSAFY AI 챗봇 서비스 개발

---

## 개요

본 프로젝트는 SSAFY 브랜드의 AI 챗봇 서비스를 구축하는 것을 목표로 합니다. 사용자는 직관적인 UI/UX 환경에서 로그인, 회원가입을 진행하고, AI 챗봇과의 실시간 채팅을 통해 질문을 주고받을 수 있습니다. 챗봇은 OpenAI의 `gpt-5-nano` 모델을 활용하여 응답을 생성합니다.

---

## 기술 스택

* **프런트엔드:** React, TypeScript
* **스타일링:** Bootstrap 5.3, 커스텀 포인트 컬러 `#00EEFF`
* **상태관리:** Redux
* **백엔드:** Node.js (Vanilla JS), Express
* **AI 모델:** OpenAI `gpt-5-nano` API

---

## 공통 UI/UX 설계

### 네비게이션 바 (Navbar)

* **구성요소:**

  * SSAFY 브랜드 로고 (좌측)
  * 채팅, 로그인, 회원가입 버튼 (우측)
* **동작:**

  * SSAFY 로고 클릭 시 랜딩 페이지(`/`)로 이동
  * 채팅 버튼 클릭 시 채팅 페이지(`/chat`)로 이동
  * 로그인 버튼 클릭 시 로그인 페이지(`/login`)로 이동
  * 회원가입 버튼 클릭 시 회원가입 페이지(`/register`)로 이동
* **디자인:**

  * 모던하고 미니멀한 스타일 적용
  * 포인트 컬러 `#00EEFF` 적용

---

## 화면별 상세 요구 사항

### 1. 랜딩 페이지 (`/`)

* SSAFY 챗봇 서비스 소개 내용 포함
* 브랜드 이미지, 슬로건, 기능 소개 섹션 포함
* 반응형 구성

---

### 2. 로그인 페이지 (`/login`)

* 이메일/비밀번호 입력 필드
* 로그인 버튼 및 비밀번호 찾기 링크
* "회원이 아니신가요?" → 회원가입 페이지 링크

---

### 3. 회원가입 페이지 (`/register`)

* 이름, 이메일, 비밀번호, 비밀번호 확인 필드
* 약관 동의 체크박스
* 회원가입 버튼
* "이미 회원이신가요?" → 로그인 페이지 링크

---

### 4. 채팅 페이지 (`/chat`)

#### 4.1 챗봇 기능

* OpenAI의 `gpt-5-nano` 모델 API 연동
* 사용자의 질문에 대해 AI가 실시간으로 응답
* API 호출 예시:

```javascript
import OpenAI from "openai";

const openai = new OpenAI();

const response = await openai.responses.create({
    model: "gpt-4.1-mini",
    input: [{
        role: "user",
        content: [
            { type: "input_text", text: "what's in this image?" },
            {
                type: "input_image",
                image_url: "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",
            },
        ],
    }],
});

console.log(response.output_text);
```

#### 4.2 UI 구성

* **입력창 (화면 하단 고정)**

  * 왼쪽부터: 이미지 업로드 버튼, 프롬프트 입력창, 전송 버튼
  * 화면 하단 고정

* **이미지 업로드 기능**

  * 이미지 선택 시, 업로드 버튼 위에 미리보기 이미지 표시
  * 미리보기 위에 마우스를 올리면 ‘취소’ 버튼 노출
  * 취소 버튼 클릭 시 이미지 제거

* **채팅 내역 UI**

  * 채팅 내역은 시간순으로 정렬
  * 사용자 질문은 우측, AI 응답은 좌측에 출력
  * 텍스트와 이미지 모두 표시 가능
  * 각 메시지는 말풍선 형태로 렌더링

* **디자인 요소**

  * Bootstrap 5.3 활용하여 반응형 UI 구현
  * 포인트 컬러 `#00EEFF` 버튼, 강조 색상 등에 활용

---

## 상태관리 (Redux)

* **스토어 구성**

  * 채팅 내역: `chatHistory`
  * 현재 입력 중인 텍스트: `currentPrompt`
  * 업로드된 이미지: `uploadedImage`
  * 로딩 상태: `isLoading`

* **액션 예시**

  * `ADD_MESSAGE`, `SET_PROMPT`, `UPLOAD_IMAGE`, `CLEAR_IMAGE`, `SET_LOADING`

* **미들웨어 사용**

  * API 요청 시 비동기 처리를 위한 `redux-thunk` 또는 `redux-saga` 적용 가능

---

## 백엔드 API 설계 (Node.js)

* **엔드포인트 예시**

  * `POST /api/chat`: 사용자 질문 전송 및 gpt-5-nano 응답 반환
  * `POST /api/upload`: 이미지 업로드 처리
  * `POST /api/login`, `POST /api/register`: 사용자 인증

* **보안**

  * CORS 설정
  * JSON validation
  * 기본적인 에러 핸들링

---

## 디자인 및 반응형 가이드

* **화면 크기에 따른 UI 대응**

  * 모바일: 하단 고정 입력창, 말풍선 최소화
  * 태블릿/PC: 전체 레이아웃 확장

* **접근성 고려**

  * 키보드 네비게이션, 스크린리더 대응 태그 사용

---

## 테스트 및 검수

* **테스트 항목**

  * 입력창 동작 (텍스트, 이미지)
  * API 통신 및 응답 처리
  * 반응형 레이아웃 확인
  * 이미지 업로드 및 미리보기/취소 기능

* **브라우저 호환성**

  * Chrome, Edge, Firefox, Safari에서 검수

---

## 기타 유의사항

* 전역 스타일은 Bootstrap을 기반으로 커스터마이징
* 디자인 시스템 일관성 유지
* 외부 라이브러리 최소화

---

## 향후 확장 고려 사항

* 채팅 내역 저장 기능 (DB 연동)
* 로그인 사용자 기반 퍼스널 채팅 히스토리 제공
* 음성 입력 및 응답 기능 추가

---

## 폴더 구조

```bash
ssafy-ai-chatbot/
├─ client/                  # React + TS + Redux + Bootstrap
├─ server/                  # Node.js (Vanilla JS) + Express
├─ .gitignore
└─ README.md             
```

## .env 예시

```bash
# server/.env
PORT=3001
OPENAI_API_KEY=sk-...
CORS_ORIGIN=http://localhost:5173
```

```bash
# client/.env
VITE_API_BASE_URL=http://localhost:3001
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/forestsoft-kr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
