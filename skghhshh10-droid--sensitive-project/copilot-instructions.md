## sensitive-project

> HTML, CSS, JavaScript, Node.js, OpenAI API, Supabase, Vercel을 사용해 **텍스트 감성 분석 서비스**를 구현한다.

# AGENTS.md — Antigravity AI 작업 규칙

## 1. 프로젝트 목표
HTML, CSS, JavaScript, Node.js, OpenAI API, Supabase, Vercel을 사용해 **텍스트 감성 분석 서비스**를 구현한다.

사용자는 텍스트를 입력하고, 서버는 OpenAI API에 감성 분석을 요청한다. 결과는 `긍정`, `부정`, `중립` 중 하나로 표시하며, 신뢰도 백분율, 분석 이유, 오류 메시지를 함께 제공한다.

## 2. 절대 작업 규칙
AI 코딩 에이전트는 아래 규칙을 반드시 지킨다.

1. 사용자가 명시하지 않은 기능을 임의로 추가하지 않는다.
2. OpenAI API Key와 Supabase Service Role Key는 절대 프론트엔드 코드에 노출하지 않는다.
3. 감성 분석 요청은 반드시 Node.js 백엔드 API를 통해 처리한다.
4. 프론트엔드는 HTML, CSS, JavaScript만 사용한다.
5. React, Next.js, TypeScript, Tailwind CSS는 사용하지 않는다.
6. DB 저장 기능은 Supabase 연결이 완료된 뒤에만 구현한다.
7. API 오류, 빈 입력, 네트워크 오류, 응답 파싱 오류를 모두 사용자에게 보여준다.
8. 각 작업 완료 후 실행 방법과 검증 방법을 함께 설명한다.

## 3. 권장 폴더 구조
```txt
sentiment-analysis-service/
├─ AGENTS.md
├─ PRD.md
├─ README.md
├─ package.json
├─ .env.example
├─ server/
│  └─ index.js
├─ public/
│  ├─ index.html
│  ├─ styles.css
│  └─ app.js
└─ docs/
   ├─ 01_project_overview.md
   ├─ 02_ui_design_guide.md
   ├─ 03_frontend_implementation.md
   ├─ 04_backend_api.md
   ├─ 05_supabase_schema.md
   └─ 06_deployment_verification.md
```

## 4. 환경변수 규칙
`.env`에는 아래 값을 사용한다.

```env
OPENAI_API_KEY=
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
PORT=3000
```

`.env.example`만 Git에 포함하고, 실제 `.env`는 커밋하지 않는다.

## 5. 완료 기준
- `npm install` 후 `npm run dev`로 실행된다.
- 브라우저에서 메인 화면이 표시된다.
- 텍스트 입력 후 분석 버튼을 누르면 감성 결과가 표시된다.
- 빈 입력 시 오류 메시지가 표시된다.
- API Key가 없거나 잘못되면 친절한 오류 메시지가 표시된다.
- Supabase 연결이 있으면 분석 기록이 저장된다.

## 6. 검증 기준
- Chrome 브라우저 기준으로 동작 확인한다.
- 모바일 화면에서도 입력창과 결과 카드가 깨지지 않는다.
- 서버 콘솔에 치명적 오류가 없어야 한다.
- 브라우저 개발자도구 Console에 불필요한 오류가 없어야 한다.

---
> Source: [skghhshh10-droid/sensitive-project](https://github.com/skghhshh10-droid/sensitive-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
