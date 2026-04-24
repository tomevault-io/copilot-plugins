## vibe-proxy-practice

> **VibeChatbot** — CLIProxyAPI + Oracle VPS 기반 개인용 AI 챗봇

# PRD: AI Chatbot with VibeProxy (Oracle VPS Edition)

## 1. 프로젝트 개요

### 1.1 프로젝트명

**VibeChatbot** — CLIProxyAPI + Oracle VPS 기반 개인용 AI 챗봇

### 1.2 목적

Oracle Cloud Always Free VPS에 CLIProxyAPI를 배포하고, Vercel에 배포된 Next.js 챗봇에서 이를 호출하여 구독 중인 AI 모델(Claude, GPT 등)을 API 비용 없이 사용한다.

### 1.3 배경

- VibeProxy/CLIProxyAPI는 구독형 AI 서비스의 OAuth 토큰을 재활용하여 OpenAI 호환 API를 제공하는 프록시이다.
- 로컬에서만 동작하는 VibeProxy의 한계를 극복하기 위해, 24시간 가동되는 VPS에 프록시를 배포한다.
- Oracle Cloud Always Free Tier는 ARM 4코어/24GB RAM VM을 무료로 제공하므로, 프록시 서버로 충분하다.
- Next.js의 API Route가 VPS의 프록시를 호출하므로, 클라이언트에 프록시 URL이 노출되지 않는다.

### 1.4 대상 사용자

- 개발자 본인 (개인용)
- AI 구독을 보유하고 있으며, 배포된 환경에서도 AI를 활용하고 싶은 사용자

---

## 2. 시스템 아키텍처

### 2.1 전체 흐름

```
┌──────────────────────────────────────────────────────┐
│                     Browser                           │
│  ┌────────────────────────────────────────────────┐  │
│  │              React Client (Vercel)               │  │
│  │  - ChatInput / ChatMessages / ModelSelector      │  │
│  └─────────────────┬──────────────────────────────┘  │
│                    │ POST /api/chat                   │
└────────────────────┼─────────────────────────────────┘
                     │
┌────────────────────┼─────────────────────────────────┐
│          Vercel Serverless Function                    │
│  ┌─────────────────┴──────────────────────────────┐  │
│  │           API Route: /api/chat                    │  │
│  │  - 요청 검증                                      │  │
│  │  - OpenAI SDK로 VPS 프록시에 요청                  │  │
│  │  - 스트리밍 응답 파이프                            │  │
│  └─────────────────┬──────────────────────────────┘  │
└────────────────────┼─────────────────────────────────┘
                     │ POST /v1/chat/completions
                     │ Authorization: Bearer <proxy-api-key>
                     │
┌────────────────────┼─────────────────────────────────┐
│       Oracle Cloud VPS (Always Free ARM)              │
│  ┌─────────────────┴──────────────────────────────┐  │
│  │         CLIProxyAPI (Docker, :8317)               │  │
│  │  - API Key 인증 확인                              │  │
│  │  - OAuth Token으로 교체                           │  │
│  │  - 실제 AI 서버로 프록시                          │  │
│  └─────────────────┬──────────────────────────────┘  │
│                    │                                  │
│  ┌─────────────────┴──────────────────────────────┐  │
│  │  Cloudflare Tunnel (HTTPS 노출)                   │  │
│  │  proxy.mydomain.com → localhost:8317              │  │
│  └────────────────────────────────────────────────┘  │
│                                                       │
│  ~/.cli-proxy-api/                                    │
│  ├── claude-*.json    (OAuth Token)                   │
│  ├── codex-*.json     (OAuth Token)                   │
│  └── config.yaml      (프록시 설정)                   │
└────────────────────┼─────────────────────────────────┘
                     │
              ┌──────┴──────┐
              │ AI Provider │
              │ (Anthropic, │
              │  OpenAI 등) │
              └─────────────┘
```

### 2.2 인프라 구성 요약

| 구성 요소 | 서비스 | 비용 |
|-----------|--------|------|
| 프론트엔드 + API Route | Vercel (Hobby Plan) | 무료 |
| AI 프록시 서버 | Oracle Cloud Always Free (ARM 4C/24GB) | 무료 |
| HTTPS 터널 | Cloudflare Tunnel | 무료 |
| 도메인 (선택) | Cloudflare DNS | 무료 (기존 도메인) 또는 약 $10/년 |
| AI 구독 | Claude Max / ChatGPT Plus | 기존 구독 비용만 |

---

## 3. 기능 요구사항

### 3.1 핵심 기능 (MVP)

| # | 기능 | 설명 | 우선순위 |
|---|------|------|---------|
| F-01 | 채팅 메시지 전송 | 텍스트 입력 후 전송하면 AI 모델에 요청 | Must |
| F-02 | AI 응답 수신 | AI의 응답을 스트리밍(SSE)으로 실시간 표시 | Must |
| F-03 | 대화 히스토리 | 현재 세션의 대화 내역을 화면에 유지 | Must |
| F-04 | 마크다운 렌더링 | AI 응답의 마크다운(코드블록, 볼드 등)을 정상 렌더링 | Must |
| F-05 | 모델 선택 | 사용 가능한 AI 모델을 선택할 수 있는 드롭다운 | Should |
| F-06 | 새 대화 시작 | 대화 히스토리를 초기화하고 새 세션을 시작 | Should |
| F-07 | 로딩 인디케이터 | AI 응답 대기 중 타이핑 인디케이터 표시 | Should |

### 3.2 비기능 요구사항

| # | 항목 | 설명 |
|---|------|------|
| NF-01 | 반응형 | 모바일/데스크톱 모두 사용 가능한 레이아웃 |
| NF-02 | 성능 | 스트리밍 응답 시 UI 버벅임 없이 렌더링 |
| NF-03 | 접근성 | Enter로 전송, Shift+Enter로 줄바꿈 |
| NF-04 | 에러 처리 | 프록시 연결 실패, 모델 오류 시 사용자 친화적 에러 메시지 |
| NF-05 | 보안 | 프록시 API Key를 서버 사이드에서만 사용, 클라이언트 노출 금지 |

---

## 4. 기술 스택

| 분류 | 기술 | 비고 |
|------|------|------|
| 프레임워크 | Next.js 15 | App Router |
| 언어 | TypeScript | strict mode |
| 스타일링 | Tailwind CSS v4 | `@import "tailwindcss"` |
| AI 통신 | Vercel AI SDK (`ai`, `@ai-sdk/openai`) | OpenAI 호환 스트리밍 |
| 마크다운 | react-markdown + remark-gfm | 코드 하이라이팅 포함 |
| 프록시 서버 | CLIProxyAPI (Docker) | Oracle VPS |
| HTTPS 터널 | Cloudflare Tunnel (`cloudflared`) | 무료 HTTPS |
| 배포 | Vercel | Hobby Plan |

---

## 5. API 설계

### 5.1 POST /api/chat

**Request Body:**

```typescript
interface ChatRequest {
  messages: Array<{
    role: 'user' | 'assistant' | 'system';
    content: string;
  }>;
  model?: string; // 기본값: "claude-sonnet-4"
}
```

**Response:** Vercel AI SDK Data Stream (SSE 기반)

### 5.2 서버 → 프록시 통신

API Route 내부에서 CLIProxyAPI로 보내는 요청:

```
POST https://proxy.mydomain.com/v1/chat/completions
Headers:
  Content-Type: application/json
  Authorization: Bearer <PROXY_API_KEY>
Body:
  { "model": "claude-sonnet-4", "messages": [...], "stream": true }
```

---

## 6. 페이지 & 컴포넌트 구조

```
src/
├── app/
│   ├── layout.tsx              # 루트 레이아웃 (서버 컴포넌트)
│   ├── page.tsx                # 메인 채팅 페이지 (서버 컴포넌트)
│   ├── globals.css             # Tailwind v4 임포트
│   └── api/
│       └── chat/
│           └── route.ts        # AI 프록시 API Route (POST)
├── components/
│   ├── chat-container.tsx      # 채팅 전체 래퍼 (use client)
│   ├── chat-messages.tsx       # 메시지 리스트
│   ├── chat-input.tsx          # 입력 폼
│   ├── chat-bubble.tsx         # 개별 메시지 버블
│   ├── model-selector.tsx      # 모델 선택 드롭다운
│   └── markdown-renderer.tsx   # 마크다운 렌더링
├── lib/
│   ├── constants.ts            # 모델 목록, 설정값
│   └── utils.ts                # 유틸리티 함수
└── types/
    └── chat.ts                 # 타입 정의
```

---

## 7. 환경변수

### 7.1 Next.js (Vercel Dashboard에 설정)

```env
# CLIProxyAPI 연결 (Oracle VPS)
PROXY_BASE_URL=https://proxy.mydomain.com/v1
PROXY_API_KEY=sk-your-secure-proxy-key

# 기본 모델
DEFAULT_MODEL=claude-sonnet-4
```

### 7.2 CLIProxyAPI (Oracle VPS config.yaml)

```yaml
port: 8317

remote-management:
  allow-remote: false
  secret-key: ""

auth-dir: "~/.cli-proxy-api"
debug: false
logging-to-file: true
request-retry: 3

quota-exceeded:
  switch-project: true
  switch-preview-model: true

auth:
  providers:
    - type: "api-key"
      key: "sk-your-secure-proxy-key"
```

---

## 8. Oracle VPS 세팅 절차 (요약)

### Phase 0: Oracle Cloud VM 프로비저닝

1. Oracle Cloud 가입 (신용카드 인증, 과금 없음)
2. Home Region → 서울 또는 도쿄 선택
3. Ampere A1 VM 생성 (2 OCPU / 12GB RAM, Ubuntu 22.04)
4. VCN + Public Subnet + Security List에서 SSH(22)만 오픈

### Phase 1: CLIProxyAPI Docker 배포

```bash
sudo apt update && sudo apt install -y docker.io docker-compose
sudo systemctl enable docker
mkdir -p ~/cli-proxy && cd ~/cli-proxy
# docker-compose.yml + config.yaml 작성 후
docker compose up -d
```

### Phase 2: OAuth 로그인

```bash
docker exec -it cli-proxy-api ./CLIProxyAPI --claude-login
```

### Phase 3: Cloudflare Tunnel HTTPS

```bash
# cloudflared 설치 → 터널 생성 → DNS 연결 → 서비스 등록
cloudflared tunnel create cli-proxy
cloudflared tunnel route dns cli-proxy proxy.mydomain.com
```

### Phase 4: Vercel 환경변수 설정 & 배포

---

## 9. 보안 체크리스트

- [ ] CLIProxyAPI `auth.providers`에 강력한 API Key 설정
- [ ] VPS 8317 포트 외부 차단, Cloudflare Tunnel로만 접근
- [ ] Vercel 환경변수에 API Key 저장, 클라이언트 노출 금지
- [ ] `remote-management.allow-remote: false`
- [ ] Security List: SSH(22)만 오픈

---

## 10. 성공 기준

- [ ] Vercel 배포 앱에서 AI 스트리밍 응답 수신 가능
- [ ] Multi-turn 대화 컨텍스트 유지
- [ ] 마크다운 정상 렌더링
- [ ] 에러 시 사용자 친화적 메시지
- [ ] API 과금 없이 구독 크레딧만으로 동작
- [ ] 모바일/데스크톱 반응형

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yyj0917) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
