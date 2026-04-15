## test

> 이 파일은 Claude 등 AI 어시스턴트가 이 프로젝트를 이해하고 올바르게 기여하기 위한 가이드입니다.

# CLAUDE.md — AI 어시스턴트 가이드

이 파일은 Claude 등 AI 어시스턴트가 이 프로젝트를 이해하고 올바르게 기여하기 위한 가이드입니다.
각 영역별 상세 개발 규칙은 하위 `Skills.md`를 참조하세요.

---

## 프로젝트 한 줄 요약

> 이미지(PNG/JPG/WebP) 또는 Figma 디자인을 입력받아 **HTML + CSS + JavaScript** 코드를 자동 생성하는 변환 도구

---

## 영역별 상세 가이드 (Skills.md)

각 폴더에 위치한 `Skills.md`에 해당 영역의 구체적인 개발 패턴, 컴포넌트 명세, 서비스 로직, 테스트 규칙이 기술되어 있습니다.

| 영역 | 파일 | 주요 내용 |
|---|---|---|
| 클라이언트 (React UI) | [`client/Skills.md`](./client/Skills.md) | 컴포넌트 작성 규칙, API 호출 패턴, 스타일링, 다운로드/포맷 유틸 |
| 서버 (Node.js API) | [`server/Skills.md`](./server/Skills.md) | 라우트 구조, IR 데이터 포맷, 서비스 명세, Figma 클라이언트, 미들웨어 |

> 새로운 기능을 추가하거나 버그를 수정하기 전에 **반드시 해당 영역의 Skills.md를 먼저 확인**하세요.

---

## 레포지토리 구조

```
/
├── client/                        # React 기반 변환 도구 UI
│   ├── Skills.md                  # 클라이언트 개발 가이드 ★
│   └── src/
│       ├── components/            # 재사용 가능한 UI 컴포넌트
│       │   ├── Uploader/          # 이미지 업로드 드래그앤드롭
│       │   ├── FigmaInput/        # Figma URL / 토큰 입력 폼
│       │   ├── Preview/           # iframe 기반 변환 결과 미리보기
│       │   └── CodeEditor/        # CodeMirror 기반 코드 편집기
│       ├── pages/
│       │   ├── Home.jsx           # 변환 입력 페이지
│       │   └── Result.jsx         # 변환 결과 페이지
│       ├── services/
│       │   ├── figmaService.js    # Figma API 호출 함수 모음
│       │   └── convertService.js  # 이미지 변환 API 호출 함수 모음
│       └── utils/
│           ├── download.js        # ZIP / 단일 파일 다운로드
│           └── format.js          # 생성된 코드 prettier 포맷
│
├── server/                        # Node.js + Express API 서버
│   ├── Skills.md                  # 서버 개발 가이드 ★
│   ├── routes/
│   │   ├── convert.js             # POST /api/convert
│   │   ├── figma.js               # POST /api/figma
│   │   └── export.js              # GET  /api/export/:id
│   ├── services/
│   │   ├── imageAnalyzer.js       # Vision API 연동 → 레이아웃 분석
│   │   ├── figmaParser.js         # Figma JSON → 중간 표현(IR) 변환
│   │   ├── codeGenerator.js       # IR → HTML / CSS / JS 문자열 생성
│   │   └── cssExtractor.js        # 스타일 추출 및 CSS 변수화
│   ├── middlewares/
│   │   ├── upload.js              # Multer 설정 (10MB 제한)
│   │   └── errorHandler.js        # 공통 에러 응답 처리
│   ├── utils/
│   │   └── figmaClient.js         # Figma REST API Axios 클라이언트
│   └── app.js                     # Express 앱 진입점
│
├── output/                        # 변환된 코드 임시 저장 (git 제외)
├── .env.example                   # 환경 변수 템플릿
├── README.md                      # 사용자용 프로젝트 문서
└── CLAUDE.md                      # AI 어시스턴트용 가이드 (현재 파일)
```

---

## 실행 및 개발 명령어

```bash
# 서버 개발 서버 실행 (포트 4000, nodemon)
cd server && npm run dev

# 클라이언트 개발 서버 실행 (포트 3000)
cd client && npm start

# 전체 테스트 실행
cd server && npm test
cd client && npm test

# 클라이언트 프로덕션 빌드
cd client && npm run build
```

---

## 핵심 변환 파이프라인

AI가 코드를 수정할 때 이 흐름을 반드시 이해해야 합니다.

### 이미지 → 코드

```
[사용자 이미지 업로드]
        ↓
upload.js (Multer) → 파일 저장
        ↓
routes/convert.js → POST /api/convert
        ↓
services/imageAnalyzer.js → OpenAI Vision API 호출
        ↓
중간 표현(IR) JSON 생성
        ↓
services/codeGenerator.js → IR를 HTML/CSS/JS 문자열로 변환
        ↓
output/{id}/ 에 index.html, style.css, script.js 저장
        ↓
클라이언트에 변환 ID 응답
```

> IR 데이터 구조 상세 → [`server/Skills.md`](./server/Skills.md#중간-표현ir-데이터-구조)

### Figma → 코드

```
[사용자 Figma URL + Token 입력]
        ↓
utils/figmaClient.js → GET https://api.figma.com/v1/files/{fileKey}
        ↓
Figma 노드 트리(JSON) 수신
        ↓
services/figmaParser.js → Figma 노드 → 중간 표현(IR) 변환
        ↓
services/codeGenerator.js → IR → HTML/CSS/JS 생성
        ↓
output/{id}/ 저장 후 ID 응답
```

> Figma 노드 타입 매핑 상세 → [`server/Skills.md`](./server/Skills.md#figmaparserjsservicesfigmaparserjs)

---

## API 엔드포인트 명세

| 메서드 | 경로 | 설명 | 요청 | 응답 |
|---|---|---|---|---|
| POST | `/api/convert` | 이미지 → 코드 변환 | `multipart/form-data` (image 필드) | `{ id, files: ['index.html', 'style.css', 'script.js'] }` |
| POST | `/api/figma` | Figma → 코드 변환 | `{ fileKey, token, nodeId? }` | `{ id, files: [...] }` |
| GET | `/api/export/:id` | 변환 결과 ZIP 다운로드 | — | ZIP 파일 스트림 |
| GET | `/api/preview/:id` | 미리보기 HTML 반환 | — | HTML 문자열 |

---

## 환경 변수

| 변수명 | 필수 | 설명 |
|---|---|---|
| `PORT` | 선택 | 서버 포트 (기본값: 4000) |
| `OPENAI_API_KEY` | 필수 | OpenAI Vision API 키 |
| `FIGMA_ACCESS_TOKEN` | 조건부 필수 | Figma Personal Access Token |
| `UPLOAD_DIR` | 선택 | 업로드 임시 경로 (기본값: `./uploads`) |
| `OUTPUT_DIR` | 선택 | 변환 결과 저장 경로 (기본값: `./output`) |
| `CLIENT_ORIGIN` | 선택 | CORS 허용 Origin (기본값: `http://localhost:3000`) |

> `.env` 파일은 절대 커밋하지 마세요. `.env.example`만 커밋 대상입니다.

---

## 파일 네이밍 컨벤션

| 대상 | 컨벤션 | 예시 |
|---|---|---|
| React 컴포넌트 | PascalCase | `CodeEditor.jsx` |
| 서버 서비스 / 유틸 | camelCase | `figmaParser.js` |
| 서버 라우트 | camelCase | `convert.js` |
| CSS 모듈 | camelCase | `codeEditor.module.css` |

---

## 코드 스타일

- **들여쓰기**: 공백 2칸
- **따옴표**: 싱글 쿼트 (`'`)
- **세미콜론**: 항상 사용
- **변수 선언**: `const` 우선, 재할당 필요 시 `let`, `var` 금지
- **함수 선언**: 서비스/유틸은 `export function` 명명 함수, 컴포넌트는 화살표 함수 + `export default`

---

## 커밋 메시지 규칙

```
feat:     새로운 기능 추가
fix:      버그 수정
refactor: 기능 변경 없이 코드 구조 개선
style:    포맷, 세미콜론 등 동작 무관한 변경
docs:     문서 수정 (README, CLAUDE.md, Skills.md 등)
test:     테스트 추가 또는 수정
chore:    빌드 설정, 패키지 변경
```

---

## AI 어시스턴트를 위한 작업 지침

### 작업 전 체크리스트

1. 수정할 코드가 `client/`에 속하면 → [`client/Skills.md`](./client/Skills.md) 확인
2. 수정할 코드가 `server/`에 속하면 → [`server/Skills.md`](./server/Skills.md) 확인
3. 변환 파이프라인에 영향을 주는 변경이라면 → 위 파이프라인 흐름도 재확인

### 새 변환 기능 추가 시

1. `server/services/figmaParser.js` 또는 `imageAnalyzer.js`에서 IR 생성 로직 수정
2. `server/services/codeGenerator.js`에서 새 IR 타입에 대한 HTML/CSS 생성 로직 추가
3. 라우트(`routes/convert.js`, `routes/figma.js`)는 가급적 수정하지 않음
4. `server/__tests__/`에 유닛 테스트 추가 → [`server/Skills.md 테스트 섹션`](./server/Skills.md#테스트-작성-규칙) 참고

### 클라이언트 컴포넌트 수정 시

- `client/services/` 함수를 통해서만 API 호출 (컴포넌트에서 직접 Axios 호출 금지)
- 상태 관리는 React 훅(`useState`, `useEffect`) 사용
- 컴포넌트 명세 → [`client/Skills.md 컴포넌트별 핵심 명세`](./client/Skills.md#컴포넌트별-핵심-명세) 참고

### 절대 하지 말아야 할 것

- `.env` 파일에 실제 API 키 하드코딩 후 커밋
- `output/` 디렉토리 내 파일 커밋
- `codeGenerator.js`에서 특정 프레임워크(React, Vue 등) 의존 코드 생성
- 생성된 HTML에 인라인 `style=""` 속성 사용
- `figmaClient.js` 대신 직접 `axios.get` 으로 Figma API 호출
- 컴포넌트에서 직접 Axios 호출 (`client/services/` 우회)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dheo0)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dheo0)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
