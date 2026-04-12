## snack-storyteller

> 이 파일은 ZETA AI 캐릭터 채팅 앱 클론 개발을 위한 종합적인 가이드라인입니다.

# ZETA Clone Development - Project Rules & Guidelines

이 파일은 ZETA AI 캐릭터 채팅 앱 클론 개발을 위한 종합적인 가이드라인입니다.

## 🎯 프로젝트 개요

**목표**: AI 개발자 전향용 포트폴리오 - AI 단편 소설 생성 플랫폼

**핵심 가치 제안**:

- 🤖 **AI 통합 숙련도**: OpenAI GPT-4 API 활용 및 Prompt Engineering
- 🎨 **스타일 커스터마이제이션**: 작가 페르소나 + 장르/분위기/결말 조합
- ⚡ **빠른 생성**: 1,500-2,000 단어 완성작을 30초 내 생성
- 📚 **라이브러리 관리**: 생성된 소설 저장, 검색, 북마크 시스템

**포트폴리오 강점**:

- ✅ 완성도 높은 풀스택 프로젝트 (1-2주 완성 가능)
- ✅ 최신 AI 기술 실전 활용 (OpenAI API, Prompt Engineering)
- ✅ 현대적 기술 스택 (React, NestJS, Prisma, PostgreSQL)
- ✅ 실제 배포 가능한 MVP
- ✅ 확장 가능한 아키텍처

---

## 🤖 AI 개발자 실행 모드

> **중요**: 당신은 "명세를 읽고 계획하는" AI가 아니라, **"작동하는 코드를 만드는"** AI 개발자입니다.

### 핵심 실행 원칙

#### 1. 완성된 코드만 작성 (No TODOs, No Stubs)

```typescript
// ❌ 절대 금지: 미완성 코드
export class AuthService {
  // TODO: implement login
  async login(email: string, password: string) {
    throw new Error('Not implemented');
  }
}

// ✅ 필수: 완전히 작동하는 코드
export class AuthService {
  constructor(
    private readonly userService: UserService,
    private readonly jwtService: JwtService,
  ) {}

  async login(email: string, password: string): Promise<LoginResponse> {
    const user = await this.userService.findByEmail(email);
    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    return {
      access_token: this.jwtService.sign({ sub: user.id, email: user.email }),
      user: { id: user.id, email: user.email, name: user.name },
    };
  }
}
```

#### 2. 모를 때는 공식 문서 확인 (Context7 활용)

```yaml
situation_to_mcp_mapping:
  'React hooks 사용법?': /context7 react hooks
  'NestJS guards 구현?': /context7 nestjs guards
  'Prisma migration 에러?': /context7 prisma migrations
  'OpenAI streaming?': /context7 openai streaming api
  'pgvector 설정?': /context7 pgvector postgres
  'shadcn 컴포넌트?': /magic (UI는 무조건 Magic MCP)
```

**절대 원칙**: 추측하지 말고, Context7로 공식 문서를 확인하라.

#### 3. 복잡한 설계는 Sequential Thinking 사용

다음 상황에서는 반드시 Sequential Thinking MCP를 활용:

- pgvector 설정 방법 결정
- AI context building 알고리즘 설계
- 아키텍처 트레이드오프 분석
- 복잡한 데이터 구조 설계
- 성능 최적화 전략 수립

#### 4. 각 단계마다 즉시 검증

```bash
# 코드 작성 후 즉시 실행
pnpm type-check              # TypeScript 에러 확인
pnpm lint                    # 코드 스타일 검증
pnpm test                    # 테스트 실행
pnpm dev                     # 실제 실행 확인

# API 엔드포인트는 curl로 테스트
curl -X POST http://localhost:3001/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"test123"}'
```

### Phase 실행 체크리스트

각 Phase 명령(`/phase1-init`, `/phase4-chat` 등) 실행 시:

#### 🟢 Phase 시작 전 (Pre-flight)

```yaml
- [ ] 이 Phase의 명확한 성공 기준을 이해했는가?
- [ ] 필요한 dependencies가 무엇인지 파악했는가?
- [ ] 모르는 부분은 Context7로 공식 문서 확인했는가?
- [ ] 복잡한 부분은 Sequential Thinking으로 계획했는가?
- [ ] 이전 Phase가 완료되었는가? (verify-phase 실행)
```

#### 🟡 Phase 진행 중 (In-flight)

```yaml
- [ ] 각 파일 작성 전 기존 프로젝트 패턴 확인
- [ ] 코드 작성 후 즉시 컴파일/타입체크 실행
- [ ] 각 기능 완성 시 간단한 수동 테스트 실행
- [ ] 에러 발생 시 Context7로 해결 방법 검색
- [ ] 진행 상황을 TodoWrite로 추적
```

#### 🔴 Phase 완료 시 (Post-flight)

```yaml
- [ ] Success criteria 각 항목 검증 완료
- [ ] pnpm build가 성공하는가?
- [ ] 핵심 기능이 실제로 작동하는가? (curl 테스트)
- [ ] 다음 Phase에 필요한 것들을 명시
- [ ] 미완성 부분이 있다면 명확히 문서화
```

### MCP 서버 활용 결정 트리

```
┌─ 질문: 무엇을 만들어야 하는가?
│
├─ UI 컴포넌트?
│  └─ ✅ Magic MCP 사용
│     예: "shadcn dialog with login form"
│
├─ 라이브러리/프레임워크 사용법?
│  └─ ✅ Context7 MCP 사용
│     예: /context7 nestjs authentication
│
├─ 복잡한 알고리즘 설계?
│  └─ ✅ Sequential Thinking 사용
│     예: context building 전략 수립
│
├─ 브라우저 테스트?
│  └─ ✅ Playwright MCP 사용
│     예: E2E 채팅 플로우 테스트
│
└─ 그 외 코드 작성?
   └─ ✅ Native Claude Code
      단, Context7로 패턴 확인 후 작성
```

### 실패 대응 프로토콜

#### 컴파일 에러 발생 시

```yaml
1. 에러 메시지 정확히 읽기
2. /context7 [관련 라이브러리] 로 공식 해결책 검색
3. Sequential Thinking으로 디버깅 계획 수립
4. 수정 후 반드시 재검증
5. 같은 에러 재발 방지를 위한 패턴 문서화
```

#### 기능이 너무 복잡할 때

```yaml
1. TodoWrite로 세부 작업 분해
2. 가장 간단한 버전부터 구현 (MVP)
3. 그 버전을 테스트하고 검증
4. 점진적으로 복잡도 추가
5. 각 단계마다 검증
```

#### 구현 방법을 모를 때

```yaml
1. 절대 추측하지 말 것
2. /context7 [기술명] 으로 공식 문서 확인
3. 공식 예시를 프로젝트에 맞게 조정
4. 조정한 내용을 주석으로 문서화
5. 작동 확인 후 다음 단계 진행
```

### 성공 기준의 재정의

**"완료"의 의미**:

- ✅ 코드가 컴파일된다 (최소 조건)
- ✅ 타입 에러가 없다
- ✅ **실제로 작동한다** (필수)
- ✅ 수동 테스트로 검증 완료
- ✅ 다음 Phase를 진행할 수 있는 상태

**"거의 다 됐다"는 완료가 아닙니다.**

### 코드 품질 자동 검증

매 커밋 전 실행할 검증 스크립트:

```bash
#!/bin/bash
# scripts/verify-quality.sh

echo "🔍 Quality Check Starting..."

# 1. TypeScript 검증
echo "📘 TypeScript check..."
pnpm -r type-check || exit 1

# 2. Linting
echo "✨ ESLint check..."
pnpm -r lint || exit 1

# 3. Build 검증
echo "🔨 Build check..."
pnpm -r build || exit 1

# 4. 테스트 실행
echo "🧪 Running tests..."
pnpm -r test || exit 1

echo "✅ All quality checks passed!"
```

### Phase별 핵심 검증 포인트

#### Phase 1 (Init)

```bash
✅ pnpm install 성공
✅ apps/web 컴파일 성공
✅ apps/server 컴파일 성공
✅ 브라우저에서 React 앱 로드
✅ NestJS 서버 /health 응답
```

#### Phase 2 (Auth)

```bash
✅ Prisma migration 성공
✅ User 생성 API 작동
✅ JWT 토큰 발급 확인
✅ Protected route가 401 반환
✅ 유효한 토큰으로 인증 성공
```

#### Phase 4 (Chat + AI)

```bash
✅ OpenAI API 연결 성공
✅ SSE 스트리밍 엔드포인트 작동
✅ Frontend에서 실시간 토큰 출력 확인
✅ 대화 내역 DB 저장 확인
✅ curl로 streaming 테스트 통과
```

#### Phase 5 (Memory)

```bash
✅ pgvector extension 설치 완료
✅ Vector column migration 성공
✅ 임베딩 생성 파이프라인 작동
✅ 유사도 검색 쿼리 실행 성공
✅ Context에 관련 메모리 포함 확인
```

---

## 📐 Spec-Driven Development (SDD) 실행 규칙

> **핵심**: Specification이 Source of Truth다. 코드보다 명세가 먼저다.

### Phase Commands 실행 시 필수 절차

**모든 Phase Commands (`/phase1-init`, `/phase2-auth`, `/phase3-writers`, 등) 실행 시 다음을 반드시 따른다:**

#### STEP 0: SDD 문서 읽기 (MANDATORY)

```bash
# 1. Constitution 읽기 (프로젝트 불변 원칙)
Read specs/constitution.md

# 2. 해당 Phase의 Spec 읽기 (요구사항)
Read specs/phase{N}-{name}/spec.md

# 3. 해당 Phase의 Plan 읽기 (기술 설계)
Read specs/phase{N}-{name}/plan.md

# 4. 해당 Phase의 Tasks 읽기 (작업 분해)
Read specs/phase{N}-{name}/tasks.md
```

**예시**: `/phase3-writers` 실행 시

```bash
Read specs/constitution.md           # 프로젝트 원칙
Read specs/phase3-writers/spec.md    # Phase 3 요구사항
Read specs/phase3-writers/plan.md    # Phase 3 기술 설계
Read specs/phase3-writers/tasks.md   # Phase 3 작업 목록
```

#### 문서별 역할

| 문서              | 역할               | 읽는 이유                                    |
| ----------------- | ------------------ | -------------------------------------------- |
| `constitution.md` | 프로젝트 불변 원칙 | 절대 위반하면 안 되는 규칙 확인              |
| `spec.md`         | 요구사항 명세      | 무엇을 만들어야 하는지 정확히 파악           |
| `plan.md`         | 기술 설계          | 어떻게 만들 것인지 아키텍처 결정 확인        |
| `tasks.md`        | 작업 분해          | 구체적으로 무엇을 구현해야 하는지 체크리스트 |

#### 읽은 후 확인사항

SDD 문서를 읽은 후 다음을 명시적으로 확인:

1. **Constitution 위반 여부**: 불변 원칙을 지킬 수 있는가?
2. **Spec 이해도**: 핵심 요구사항 3가지를 말할 수 있는가?
3. **Plan 숙지도**: 주요 아키텍처 결정 사항을 이해했는가?
4. **Tasks 파악**: 총 몇 개의 작업이 있고, 우선순위는?

### SDD 문서가 없을 때

만약 해당 Phase의 SDD 문서(spec/plan/tasks)가 아직 없다면:

1. **절대 추측하지 말 것** - 임의로 구현하지 않음
2. **SDD 문서 먼저 작성** - `.claude/docs/methodology/SDD-Deep-Research.md` 참고
3. **사용자 확인 받기** - spec.md 초안을 사용자에게 검토 요청
4. **승인 후 구현 시작** - 승인된 명세를 기반으로만 코드 작성

### SDD → 구현 워크플로우

```
┌─────────────────────────────────────────────────────────┐
│ Phase Command 실행 (/phase3-writers)                    │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ STEP 0: SDD 문서 읽기                                   │
│ • constitution.md (프로젝트 원칙)                       │
│ • spec.md (요구사항)                                    │
│ • plan.md (기술 설계)                                   │
│ • tasks.md (작업 목록)                                  │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ STEP 1: Context7 공식 문서 학습                         │
│ • NestJS CRUD patterns                                  │
│ • Prisma relations                                      │
│ • File upload validation                                │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ STEP 2: Sequential Thinking 설계                        │
│ • SystemPrompt 구조 설계                                │
│ • 아키텍처 트레이드오프 분석                            │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ STEP 3: TodoWrite 작업 추적                             │
│ • tasks.md 기반으로 작업 리스트 생성                    │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ STEP 4: 순차 구현                                       │
│ • Backend → Frontend                                    │
│ • 각 단계마다 type-check, lint, test                   │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ STEP 5: 검증                                            │
│ • spec.md 성공 기준 확인                                │
│ • curl 테스트                                           │
│ • 브라우저 수동 테스트                                  │
└─────────────────────────────────────────────────────────┘
```

### SDD 문서 업데이트 규칙

**코드 구현 중 설계 변경 필요 시**:

1. **즉시 멈추기** - 코드 작성 중단
2. **변경 사유 기록** - 왜 변경이 필요한지 명시
3. **plan.md 업데이트** - 설계 문서 먼저 수정
4. **사용자 확인** - 변경사항 승인 요청
5. **승인 후 재개** - 업데이트된 plan.md 기반으로 구현

**잘못된 예**:

```
❌ "spec.md에는 X라고 되어 있지만, Y가 더 나을 것 같아서 Y로 구현했습니다"
```

**올바른 예**:

```
✅ "spec.md에 X라고 되어 있는데, 다음 이유로 Y가 더 적합합니다:
    [이유 설명]
    plan.md를 업데이트하겠습니다. 승인 부탁드립니다."
```

### SDD Hooks 연동

`.claude/settings.json`의 hooks가 SDD 문서 수정을 감지:

- `spec.md` 수정 시 → Context7/Sequential로 검증 권장
- `plan.md` 수정 시 → 관련 문서 업데이트 필요성 알림
- `tasks.md` 수정 시 → TodoWrite 동기화 권장

**SDD 문서를 수정했다면 반드시**:

1. 다른 SDD 문서와 일관성 확인
2. 이미 작성된 코드와의 정합성 점검
3. 필요시 코드 리팩토링

---

## 🏗️ 기술 스택 & 아키텍처

### Frontend Stack

```yaml
framework: React 18 + Vite + TypeScript
ui_library: shadcn/ui + TailwindCSS
state_management:
  - TanStack Query (서버 상태)
  - Zustand (클라이언트 상태)
routing: React Router v6
build_tool: Vite
package_manager: pnpm
```

### Backend Stack

```yaml
framework: NestJS + TypeScript
database: PostgreSQL + Prisma ORM
vector_db: pgvector (의미적 메모리)
auth: Passport JWT + bcrypt
validation: class-validator + class-transformer
api_style: RESTful + SSE (스트리밍)
```

### AI & External Services

```yaml
ai_providers:
  - OpenAI GPT-4 (주요)
  - Anthropic Claude (보조)
  - OpenRouter (폴백)
streaming: Server-Sent Events (SSE)
moderation: OpenAI Moderation API
payments: Stripe
storage: AWS S3 (이미지)
```

### Infrastructure

```yaml
containerization: Docker + docker-compose
monorepo: pnpm workspaces
deployment: Vercel (frontend) + Railway/Render (backend)
monitoring: Sentry + PostHog
cdn: Cloudflare
```

## 📁 프로젝트 구조

```
snack-storyteller/
├── apps/
│   ├── web/                 # React frontend
│   └── server/             # NestJS backend
├── packages/
│   ├── ui/                 # shadcn 컴포넌트
│   ├── types/              # 공유 타입
│   └── utils/              # 유틸리티 함수
├── .claude/
│   └── commands/           # 개발 명령어
├── docs/                   # 문서
├── docker-compose.yml      # 개발 환경
├── pnpm-workspace.yaml     # 모노레포 설정
└── CLAUDE.md               # 이 파일
```

## 🎨 UI/UX 디자인 시스템 (ZETA 기반)

### 컬러 팔레트

```css
/* Primary Colors */
--background: 0 0% 3.9%; /* 다크 배경 */
--foreground: 0 0% 98%; /* 화이트 텍스트 */
--primary: 262.1 83.3% 57.8%; /* 브랜드 퍼플 */
--primary-foreground: 210 20% 98%;

/* UI Colors */
--muted: 0 0% 14.9%; /* 회색 배경 */
--muted-foreground: 0 0% 63.9%; /* 회색 텍스트 */
--accent: 0 0% 14.9%; /* 액센트 */
--destructive: 0 84.2% 60.2%; /* 에러/삭제 */
```

### 타이포그래피

```css
font-family:
  'Pretendard Variable',
  -apple-system,
  BlinkMacSystemFont,
  system-ui,
  sans-serif;

/* Font Sizes */
--text-xs: 0.75rem; /* 12px */
--text-sm: 0.875rem; /* 14px */
--text-base: 1rem; /* 16px */
--text-lg: 1.125rem; /* 18px */
--text-xl: 1.25rem; /* 20px */
--text-2xl: 1.5rem; /* 24px */
```

### 레이아웃 패턴

- **모바일 우선 설계**: 스마트폰에서 2시간+ 사용 최적화
- **다크 테마 기본**: 몰입감 있는 어두운 인터페이스
- **하단 탭 내비게이션**: 홈, 채팅, 생성, 마이페이지
- **모달 중심 인터랙션**: 로그인, 설정, 캐릭터 생성
- **카드 기반 레이아웃**: 캐릭터, 스토리, 랭킹 표시

### 컴포넌트 가이드라인

```typescript
// 항상 shadcn/ui 컴포넌트 사용
import { Button, Card, Input, Dialog } from '@/components/ui';

// 다크 테마 최적화
className = 'bg-background text-foreground border-border';

// 모바일 터치 친화적
className = 'min-h-[44px] touch-manipulation';

// 애니메이션 사용
className = 'transition-all duration-200 hover:scale-105';
```

## 🤖 AI 프롬프트 엔지니어링 Best Practices

### 시스템 프롬프트 구조

```typescript
const systemPrompt = `
당신은 ${character.name}입니다.

## 캐릭터 설정
- 이름: ${character.name}
- 성격: ${character.personality}
- 배경: ${character.background}

## 대화 규칙
1. 캐릭터의 성격을 일관되게 유지하세요
2. 한국어로 자연스럽게 대화하세요
3. 스토리 진행에 도움이 되는 선택지를 제공하세요
4. 부적절한 내용은 자연스럽게 회피하세요

## 현재 상황
${storyContext}

이전 대화:
${recentMessages}

사용자의 메시지에 캐릭터로서 응답하세요.
`;
```

### 컨텍스트 빌딩 전략

```typescript
// 우선순위 기반 컨텍스트 구성
const context = {
  priority: 1, // 캐릭터 시스템 프롬프트
  priority: 2, // 글로벌 안전 가이드라인
  priority: 3, // 장기 메모리 (pgvector Top-K)
  priority: 4, // 최근 N턴 대화
  priority: 5, // 현재 사용자 입력
};
```

### 스트리밍 최적화

```typescript
// 토큰 단위로 스트리밍, 문장 단위로 표시
const streamResponse = async function* (prompt) {
  for await (const chunk of aiProvider.stream(prompt)) {
    yield chunk.content;
  }
};
```

## 🧠 SuperClaude 활용 가이드

### MCP 서버 실전 활용 패턴

#### Context7: 공식 문서 기반 구현

**사용 시점**:

- 새로운 라이브러리 사용 전 (필수)
- 에러 메시지를 해결할 때
- API 사용법이 불확실할 때
- 공식 권장 패턴을 따라야 할 때

**실전 예시**:

```bash
# Phase 2: Auth 구현 시
/context7 nestjs jwt authentication
/context7 prisma user model relations
/context7 bcrypt password hashing

# Phase 4: Chat 구현 시
/context7 openai streaming api
/context7 nestjs sse server-sent-events
/context7 react eventSource streaming

# Phase 5: Memory 구현 시
/context7 pgvector postgres setup
/context7 prisma vector datatype
/context7 openai embeddings api
```

**출력 활용 방법**:

1. Context7 결과를 읽고 공식 패턴 파악
2. 프로젝트 구조에 맞게 조정
3. 코드 작성 시 주석으로 출처 명시
4. 작동 확인 후 다음 단계 진행

#### Sequential Thinking: 복잡한 설계 결정

**필수 사용 시점**:

- 여러 구현 방법 중 선택해야 할 때
- 트레이드오프 분석이 필요할 때
- 단계별 계획이 필요한 복잡한 기능
- 디버깅 전략을 수립할 때

**실전 예시**:

```
상황 1: pgvector 설정 방법 결정
→ Sequential Thinking으로:
  - Docker extension vs 수동 설치 비교
  - Prisma 마이그레이션 전략 수립
  - 테스트 데이터 준비 계획

상황 2: AI Context Building 전략
→ Sequential Thinking으로:
  - 토큰 제한 내에서 최적 컨텍스트 구성
  - Recency vs Relevance 균형 분석
  - 성능 vs 품질 트레이드오프
```

**사용 패턴**:

```typescript
// 코드 작성 전에 먼저 생각
'Sequential Thinking을 사용하여 [문제]를 분석하고 구현 계획을 수립해줘';

// 결과를 바탕으로 코드 작성
// 각 단계마다 Sequential Thinking 결과의 단계 번호를 주석으로 명시
```

#### Magic MCP: UI 컴포넌트 생성

**절대 원칙**: 모든 UI는 Magic으로 생성, 수동 작성 금지

**사용 예시**:

```bash
# Phase 2: Auth UI
"shadcn dialog with email/password login form, dark theme"
"user profile dropdown with avatar and logout button"

# Phase 3: Characters
"character card grid with image, name, description, responsive"
"character creation form with image upload, dark theme"

# Phase 4: Chat
"chat message bubble with avatar, timestamp, streaming animation"
"message input with send button and character counter"

# Phase 11: Storytelling
"visual novel style dialogue box with typewriter effect"
"story choice buttons with hover effects, bottom aligned"
```

**통합 방법**:

1. Magic으로 컴포넌트 코드 생성
2. `packages/ui/` 또는 `apps/web/src/components/`에 저장
3. Tailwind class를 프로젝트 컬러 시스템에 맞게 조정
4. Storybook이나 브라우저에서 확인

#### Playwright: E2E 테스트 자동화

**사용 시점**:

- Phase 완료 시 검증 자동화
- 주요 사용자 플로우 테스트
- 회귀 테스트 필요 시

**테스트 시나리오**:

```typescript
// Phase 2 검증: 로그인 플로우
test('complete login flow', async ({ page }) => {
  await page.goto('http://localhost:3000/login');
  await page.fill('[name="email"]', 'test@test.com');
  await page.fill('[name="password"]', 'test123');
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL('/');
});

// Phase 4 검증: 채팅 스트리밍
test('chat streaming works', async ({ page }) => {
  // 로그인
  // 캐릭터 선택
  // 메시지 입력
  // 스트리밍 응답 확인
});
```

### 통합 워크플로우 예시

#### Phase 4 구현 전체 흐름

```bash
# 1. 설계 단계
Sequential Thinking: "OpenAI streaming 구현 전략 수립"
→ 출력: 5단계 구현 계획

# 2. 학습 단계
/context7 openai streaming api
/context7 nestjs sse implementation
→ 공식 패턴 파악

# 3. Backend 구현
# Sequential Thinking 결과 1단계 구현
# Context7 패턴 적용하여 코드 작성

# 4. Frontend UI 생성
Magic: "chat interface with streaming message display"
→ 컴포넌트 코드 받아서 통합

# 5. 통합 테스트
Playwright: E2E 채팅 플로우 테스트
→ 자동 검증

# 6. 검증
curl로 API 테스트
브라우저에서 실제 사용 확인
```

### MCP 조합 패턴

#### 패턴 1: 학습 → 설계 → 구현

```
Context7 (공식 문서)
  → Sequential Thinking (전략 수립)
    → Native Claude Code (구현)
      → Playwright (검증)
```

#### 패턴 2: UI 중심 개발

```
Magic (컴포넌트 생성)
  → Context7 (React 패턴 확인)
    → Native Claude Code (통합)
      → Playwright (시각적 테스트)
```

#### 패턴 3: 복잡한 기능 개발

```
Sequential Thinking (문제 분해)
  → Context7 (각 부분의 공식 방법)
    → TodoWrite (작업 추적)
      → Native Claude Code (순차 구현)
        → Bash (중간 검증)
```

### 페르소나별 MCP 활용

#### 풀스택 개발자 페르소나 (기본)

```yaml
primary_mcp:
  - Context7: 모든 새 기술 사용 전
  - Sequential Thinking: 아키텍처 결정
  - Magic: 모든 UI 컴포넌트

workflow: 1. Phase 명령 받음
  2. Context7로 관련 문서 확인
  3. Sequential Thinking으로 계획
  4. 구현 (Context7 참고 + Magic)
  5. Bash로 검증
  6. Playwright로 E2E 테스트
```

#### 디버깅 페르소나 (문제 해결 시)

```yaml
primary_mcp:
  - Sequential Thinking: 원인 분석
  - Context7: 공식 해결책 검색
  - Bash: 검증 스크립트

workflow: 1. 에러 메시지 분석
  2. Sequential Thinking으로 가능한 원인 나열
  3. Context7로 공식 해결 방법 확인
  4. 수정 적용
  5. Bash로 재검증
```

#### 최적화 페르소나 (성능 개선 시)

```yaml
primary_mcp:
  - Sequential Thinking: 병목 분석
  - Context7: 최적화 패턴 확인
  - Playwright: 성능 측정

workflow: 1. Sequential Thinking으로 성능 문제 분석
  2. Context7로 권장 최적화 기법 검색
  3. 최적화 적용
  4. Playwright로 before/after 측정
```

## 📋 개발 워크플로우

### Phase 기반 개발 (MVP 5단계)

```bash
# Phase 1-5 (순차 진행)
/phase1-init        # 프로젝트 초기화 → 검증: 앱 실행
/phase2-auth        # 인증 시스템 → 검증: 로그인 성공
/phase3-writers     # 작가 페르소나 → 검증: CRUD + systemPrompt
/phase4-story-gen   # AI 소설 생성 ⭐ → 검증: GPT-4 소설 생성
/phase5-library     # 소설 라이브러리 → 검증: 저장/검색/북마크

# 유틸리티 Commands
/debug-check        # 현재 상태 진단
/help               # Commands 목록 확인
```

**개발 예상 시간**: 1-2주 (집중 개발 시)

### 각 Phase의 실행 → 검증 사이클

#### Phase 실행 워크플로우

```yaml
step_1_planning:
  action: 'Phase 명령 실행'
  tools: [TodoWrite, Sequential Thinking]
  output: '구현 계획 및 작업 리스트'

step_2_learning:
  action: 'Context7로 공식 문서 확인'
  tools: [Context7]
  output: '공식 패턴 및 예제 코드'

step_3_implementation:
  action: '코드 작성 및 통합'
  tools: [Native Claude Code, Magic]
  output: '작동하는 코드'
  validation: 'pnpm type-check && pnpm build'

step_4_manual_testing:
  action: '수동 기능 테스트'
  tools: [Bash, Playwright]
  output: '기능 작동 확인'
  validation: 'curl 테스트 또는 브라우저 확인'

step_5_verification:
  action: '/verify-phase [N] 실행'
  output: '완성도 리포트'
  decision: '80% 이상이면 다음 Phase, 아니면 수정'
```

#### 실전 예시: Phase 2 (Auth) 워크플로우

```bash
# 1. Phase 시작
사용자: /phase2-auth

# 2. AI 자동 실행
Context7: /context7 nestjs jwt authentication
Context7: /context7 prisma user model
Sequential Thinking: "JWT 구현 전략 수립"
TodoWrite: 5개 작업 생성

# 3. 순차 구현
[1/5] Prisma User model 작성
      → pnpm prisma migrate dev
      → 검증: DB에서 users 테이블 확인

[2/5] AuthService 구현
      → pnpm type-check
      → 검증: 컴파일 성공

[3/5] AuthController 작성
      → pnpm dev
      → 검증: http://localhost:3001/auth/login 404 아님

[4/5] Frontend 로그인 폼
      → Magic: "login form with email/password"
      → 검증: 브라우저에서 폼 렌더링 확인

[5/5] 통합 테스트
      → curl로 실제 로그인 시도
      → 검증: JWT 토큰 반환 확인

# 4. Phase 검증
/verify-phase 2
→ 출력: "Phase 2 완성도 90% ✅"

# 5. 다음 Phase 진행
/phase3-characters
```

### 품질 관리 (자동화 중심)

#### 코드 품질 체크포인트

```yaml
every_file_write:
  - [ ] TypeScript 컴파일 확인
  - [ ] ESLint 규칙 준수
  - [ ] 의미 있는 변수/함수명
  - [ ] JSDoc 주석 (복잡한 로직)

every_feature_complete:
  - [ ] 수동 테스트 통과
  - [ ] 에러 핸들링 추가
  - [ ] 로딩/성공/실패 상태 처리
  - [ ] 타입 안정성 확보

every_phase_complete:
  - [ ] /verify-phase 80% 이상
  - [ ] 핵심 기능 curl 테스트 통과
  - [ ] 다음 Phase 의존성 충족
  - [ ] 미완성 부분 문서화
```

#### 자동 검증 스크립트

**scripts/verify-build.sh**:

```bash
#!/bin/bash
set -e

echo "🔨 Building all packages..."
pnpm -r build

echo "✅ Build successful!"
```

**scripts/verify-types.sh**:

```bash
#!/bin/bash
set -e

echo "📘 Type checking..."
pnpm -r type-check

echo "✅ No type errors!"
```

**scripts/verify-phase.sh** (예시: Phase 2):

```bash
#!/bin/bash

echo "🔍 Verifying Phase 2: Authentication"

# 1. 서버 실행 확인
if ! curl -f http://localhost:3001/health > /dev/null 2>&1; then
  echo "❌ Server not running"
  exit 1
fi

# 2. 회원가입 테스트
SIGNUP=$(curl -s -X POST http://localhost:3001/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"test123","name":"Test"}')

if [[ $SIGNUP == *"id"* ]]; then
  echo "✅ Signup works"
else
  echo "❌ Signup failed"
  exit 1
fi

# 3. 로그인 테스트
LOGIN=$(curl -s -X POST http://localhost:3001/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"test123"}')

if [[ $LOGIN == *"access_token"* ]]; then
  echo "✅ Login works"
  TOKEN=$(echo $LOGIN | jq -r '.access_token')
else
  echo "❌ Login failed"
  exit 1
fi

# 4. 인증된 요청 테스트
ME=$(curl -s http://localhost:3001/auth/me \
  -H "Authorization: Bearer $TOKEN")

if [[ $ME == *"email"* ]]; then
  echo "✅ Protected route works"
else
  echo "❌ Protected route failed"
  exit 1
fi

echo "🎉 Phase 2 verification: 100%"
```

### 디버깅 워크플로우

#### 에러 발생 시 체계적 대응

```yaml
step_1_identify:
  action: '에러 메시지 정확히 읽기'
  questions:
    - 컴파일 에러? → TypeScript/ESLint
    - 런타임 에러? → 로그 확인
    - API 에러? → curl로 재현
    - UI 에러? → 브라우저 DevTools

step_2_research:
  action: 'Context7로 공식 해결책 검색'
  example: '/context7 prisma migration error'

step_3_analyze:
  action: 'Sequential Thinking으로 원인 분석'
  output: '가능한 원인 3-5가지 리스트'

step_4_fix:
  action: '각 원인 하나씩 테스트'
  validation: '수정 후 즉시 재검증'

step_5_prevent:
  action: '재발 방지 패턴 문서화'
  output: 'CLAUDE.md 또는 주석에 추가'
```

#### 일반적인 에러 대응 매뉴얼

**Prisma Migration 실패**:

```bash
# 1. 현재 상태 확인
pnpm prisma migrate status

# 2. 해결 시도
pnpm prisma migrate reset  # 개발 환경만
pnpm prisma migrate dev

# 3. 여전히 실패 시
/context7 prisma migration troubleshooting
```

**TypeScript 에러**:

```bash
# 1. 정확한 에러 위치 확인
pnpm type-check

# 2. Context7로 공식 해결책
/context7 typescript [에러 타입]

# 3. 타입 정의 추가/수정
```

**API 연결 실패**:

```bash
# 1. 서버 실행 확인
curl http://localhost:3001/health

# 2. CORS 문제인지 확인
# DevTools Network 탭 확인

# 3. 환경 변수 확인
cat apps/web/.env | grep API_URL
```

### 성능 기준 (측정 가능)

```yaml
phase_2_auth:
  - JWT 토큰 발급: < 200ms
  - 비밀번호 검증: < 300ms (bcrypt)
  - 보호된 라우트: < 100ms

phase_4_chat:
  - 첫 토큰 응답: < 2000ms
  - 스트리밍 토큰: < 100ms/token
  - 대화 저장: < 500ms

phase_5_memory:
  - 임베딩 생성: < 1000ms
  - 벡터 검색: < 500ms
  - Context 빌딩: < 1000ms
```

### Git 커밋 전 체크리스트

```bash
# 자동 실행 스크립트
scripts/pre-commit.sh:

#!/bin/bash
echo "🔍 Pre-commit checks..."

# TypeScript
pnpm -r type-check || exit 1

# Linting
pnpm -r lint || exit 1

# Build
pnpm -r build || exit 1

# Tests (if exist)
if [ -d "apps/server/test" ]; then
  pnpm test || exit 1
fi

echo "✅ All checks passed! Safe to commit."
```

## 🔒 보안 & 컴플라이언스

### 데이터 보호

```yaml
encryption:
  - JWT 토큰 24시간 만료
  - bcrypt 솔트 라운드 12
  - HTTPS 강제 적용
  - 환경변수 암호화

content_safety:
  - OpenAI Moderation API 적용
  - 한국어 키워드 필터링
  - 사용자 신고 시스템
  - 14세 미만 접근 차단
```

### 개인정보 처리

```yaml
data_retention:
  - 사용자 계정: 탈퇴 후 즉시 삭제
  - 대화 기록: 1년 보관 후 자동 삭제
  - 로그 데이터: 30일 보관

privacy:
  - 최소한의 개인정보 수집
  - 쿠키 동의 구현
  - 데이터 내보내기 기능
  - GDPR/개인정보보호법 준수
```

## 📊 성능 & 모니터링

### 핵심 지표

```yaml
user_engagement:
  - 일일 활성 사용자 (DAU)
  - 평균 세션 시간 (목표: 2시간+)
  - 캐릭터당 대화 수
  - 스토리 완성률

technical_metrics:
  - API 응답 시간
  - 에러율 < 0.1%
  - 서버 가동률 > 99.9%
  - 동시 접속자 처리 능력
```

### 모니터링 도구

```yaml
application:
  - Sentry (에러 추적)
  - PostHog (사용자 분석)
  - Vercel Analytics (성능)

infrastructure:
  - Railway Metrics (서버)
  - Cloudflare Analytics (CDN)
  - PostgreSQL 쿼리 성능
```

## 🚀 배포 & 운영

### 환경 설정

```yaml
development:
  - 로컬 Docker 환경
  - 핫 리로드 지원
  - 개발용 AI API 키

staging:
  - 프로덕션 동일 환경
  - 실제 데이터 테스트
  - 성능 벤치마크

production:
  - Auto-scaling 적용
  - CDN 최적화
  - 백업 자동화
```

### CI/CD 파이프라인

```yaml
workflow: 1. Pull Request → 자동 테스트
  2. Merge → Staging 배포
  3. 검증 완료 → Production 배포
  4. 모니터링 & 알림

rollback:
  - 원클릭 롤백 시스템
  - 데이터베이스 백업 복원
  - 트래픽 단계적 전환
```

## 💡 개발 시 주의사항

### 공통 원칙

1. **사용자 경험 우선**: 모든 기술적 결정은 사용자 경험을 기준으로
2. **점진적 개선**: 완벽한 기능보다 빠른 검증과 반복 개선
3. **확장성 고려**: 100만 사용자를 염두에 둔 아키텍처 설계
4. **안전성 최우선**: 부적절한 콘텐츠 차단 시스템 필수

### 금지사항

```yaml
avoid:
  - console.log (프로덕션)
  - any 타입 남용
  - 하드코딩된 URL/키
  - 동기식 DB 쿼리
  - 무한 스크롤 없는 대량 데이터
  - 보안 검증 없는 사용자 입력
```

## 🎓 학습 리소스 & 학술적 배경

### 필수 문서

- [React 18 공식 문서](https://react.dev)
- [NestJS 공식 문서](https://nestjs.com)
- [Prisma 가이드](https://prisma.io/docs)
- [shadcn/ui 컴포넌트](https://ui.shadcn.com)

### AI/ML 리소스

- [OpenAI API 문서](https://platform.openai.com/docs)
- [pgvector 가이드](https://github.com/pgvector/pgvector)
- [RAG 구현 패턴](https://docs.anthropic.com/claude/docs)

### 📚 학술적 참고문헌

```yaml
software_engineering:
  - "Clean Architecture" by Robert C. Martin (2017)
  - "Designing Data-Intensive Applications" by Martin Kleppmann (2017)
  - "Building Microservices" by Sam Newman (2021)

ai_systems:
  - "Attention Is All You Need" (Vaswani et al., 2017)
  - "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (Lewis et al., 2020)
  - "Constitutional AI: Harmlessness from AI Feedback" (Bai et al., 2022)

real_time_systems:
  - "Real-Time Systems" by Jane W.S. Liu (2000)
  - "Event Sourcing" by Martin Fowler
  - "Reactive Manifesto" (Bonér et al., 2014)
```

### 🏆 과제 평가 포인트

```yaml
기술적_깊이:
  - 고급 데이터 구조 활용 (pgvector, B-tree 인덱싱)
  - 복잡한 알고리즘 구현 (의미적 검색, 스트리밍)
  - 시스템 설계 패턴 적용 (Strategy, Observer, Factory)
  - 성능 최적화 기법 (연결 풀링, 캐싱, 지연 로딩)

소프트웨어_공학:
  - SOLID 원칙 준수 및 설계 패턴 구현
  - 종합적 테스트 전략 (단위/통합/E2E/성능)
  - 코드 품질 지표 (커버리지 90%+, 복잡도 <10)
  - CI/CD 파이프라인 및 자동화

혁신성_연구:
  - 최신 AI 기술 통합 (GPT-4, Claude, 벡터 DB)
  - 실시간 스트리밍 아키텍처 구현
  - 멀티모달 상호작용 설계
  - 학술 논문 기반 구현

문서화_발표:
  - 학술적 수준의 기술 문서
  - UML 다이어그램 및 시스템 모델링
  - 성능 벤치마크 및 분석
  - 코드 주석 및 API 문서
```

---

## 📊 상세 평가 기준 및 고득점 전략

### 평가 배점 구조 (총 100점)

```yaml
백엔드_개발_역량: 40점
  - 아키텍처 설계: 10점
  - 데이터 모델링: 10점
  - API 설계: 10점
  - 에러 처리 및 보안: 10점

AI_활용: 40점 ⭐ 가장 중요!
  - 학습 과정에서의 AI 활용: 10점
  - 문제 해결 과정에서의 AI 활용: 10점
  - 프롬프트 엔지니어링: 10점
  - AI 도구 다양성 및 전략: 10점

완성도: 20점
  - 계획한 주요 기능 작동: 10점
  - 구현의 완성도: 10점
```

### 1️⃣ 백엔드 개발 역량 (40점) - 목표: 36-38점

#### 아키텍처 설계 (10점)

**평가 요소**:

- NestJS 모듈 구조의 명확성
- 레이어 분리 (Controller → Service → Repository)
- Dependency Injection 활용
- 확장 가능한 설계

**고득점 전략**:

```typescript
// ✅ 명확한 모듈 구조
/apps/server/src
  /auth       → 인증 관련 (AuthModule, AuthService, AuthController)
  /user       → 사용자 관리
  /writer     → 작가 페르소나
  /story      → 소설 생성
  /ai         → AI 서비스 레이어 (핵심 분리!)
  /common     → 공통 유틸리티
    /filters  → Exception Filters
    /guards   → Auth Guards
    /pipes    → Validation Pipes

// ❌ 피해야 할 것
- 모든 로직을 controller에 작성
- Service layer 없이 직접 DB 접근
- 하드코딩된 의존성
```

#### 데이터 모델링 (10점)

**평가 요소**:

- Prisma schema의 정규화
- 관계 설정의 적절성
- 인덱싱 전략
- 데이터 무결성 제약

**고득점 전략**:

```prisma
// ✅ 완벽한 데이터 모델링
model Story {
  id          String   @id @default(cuid())
  title       String
  content     String   @db.Text
  tags        String[]
  wordCount   Int
  readTime    Int

  writerId    String
  userId      String
  isPublic    Boolean  @default(false)

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // 관계 설정
  writer      Writer   @relation(fields: [writerId], references: [id], onDelete: Cascade)
  user        User     @relation(fields: [userId], references: [id])
  bookmarks   Bookmark[]

  // 성능 최적화
  @@index([userId, createdAt])
  @@index([isPublic, createdAt])
  @@index([writerId])
}
```

#### API 설계 (10점)

**평가 요소**:

- RESTful 원칙 준수
- 일관된 응답 형식
- DTO 검증
- 에러 응답 표준화

**고득점 전략**:

```typescript
// ✅ 완벽한 API 설계
@Controller('stories')
@UseGuards(JwtAuthGuard)
export class StoryController {
  @Post('generate')
  @UseGuards(ThrottlerGuard)
  @Throttle(5, 60) // Rate limiting
  async generateStory(
    @Body() dto: GenerateStoryDto, // DTO 검증
    @CurrentUser() user: User,
  ): Promise<StoryResponseDto> {
    return this.storyService.generateStory(dto, user.id);
  }

  @Get()
  async getStories(
    @Query() query: PaginationDto,
    @CurrentUser() user: User,
  ): Promise<PaginatedResponse<StoryDto>> {
    return this.storyService.getUserStories(user.id, query);
  }
}

// DTO 검증
export class GenerateStoryDto {
  @IsString()
  @IsNotEmpty()
  writerId: string;

  @IsArray()
  @ArrayMinSize(1)
  @ArrayMaxSize(3)
  tags: string[];
}
```

#### 에러 처리 및 보안 (10점)

**평가 요소**:

- 전역 Exception Filter
- 입력 검증
- Rate Limiting
- 보안 헤더

**고득점 전략**:

```typescript
// ✅ 전역 에러 처리
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    const status = exception.getStatus();

    const errorResponse = {
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception.message,
    };

    logger.error(errorResponse);
    response.status(status).json(errorResponse);
  }
}

// ✅ 보안 설정
app.use(helmet());
app.enableCors({ origin: process.env.FRONTEND_URL });
app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
app.useGlobalFilters(new HttpExceptionFilter());
```

### 2️⃣ AI 활용 (40점) ⭐ 최우선 집중!

#### 학습 과정에서의 AI 활용 (10점)

**평가 요소**:

- Context7을 통한 공식 문서 학습 과정 증명
- 학습 내용의 프로젝트 적용
- 학습 과정 문서화

**고득점 전략**:

**Phase Commands에 AI 학습 전략 명시**:

````markdown
# phase4-story-gen.md에 추가

## 🤖 AI 활용 학습 전략

### Step 1: OpenAI API 공식 문서 학습 (Context7)

```bash
# GPT-4 Chat Completions API
/context7 openai gpt-4 chat completions

학습 목표:
- messages 구조 (system, user, assistant)
- temperature, max_tokens, presence_penalty, frequency_penalty 파라미터
- Response 형식 및 에러 코드
- Best practices

# Streaming API
/context7 openai streaming responses server-sent-events

학습 목표:
- Stream 방식 (AsyncGenerator)
- SSE (Server-Sent Events) 프로토콜
- 에러 처리 및 재연결 로직
- 프론트엔드 EventSource 통합

# Prompt Engineering Guide
/context7 openai prompt engineering best practices

학습 목표:
- Few-shot learning 기법
- System message 최적 활용
- 토큰 최적화 전략
- Safety & moderation
```
````

### Step 2: 학습 내용 코드 적용

```typescript
// Context7에서 학습한 패턴 적용
// 출처: OpenAI Official Documentation - Chat Completions API

const response = await this.openai.chat.completions.create({
  model: 'gpt-4-turbo-preview',
  messages: [
    {
      role: 'system',
      content: writerSystemPrompt, // 학습한 system message 패턴
    },
    {
      role: 'user',
      content: storyRequest,
    },
  ],
  temperature: 0.9, // 창의성 최적화 (공식 문서 권장)
  max_tokens: 3000,
  presence_penalty: 0.6, // 다양성 증진
  frequency_penalty: 0.3, // 반복 감소
});
```

### Step 3: 학습 과정 문서화

README.md나 개발 일지에 작성:

```markdown
## AI 학습 과정

### Phase 4 - AI 소설 생성 구현

**학습 도구**: Context7 (공식 문서 조회)

**학습 내용**:

1. OpenAI GPT-4 API 기본 구조
   - Context7 검색: "openai gpt-4 chat completions"
   - 핵심 학습: messages 배열 구조, role 타입 (system/user/assistant)

2. 파라미터 튜닝 전략
   - Context7 검색: "openai temperature parameter guide"
   - 적용: 창의적 글쓰기에는 temperature 0.8-1.0 권장

3. 스트리밍 구현
   - Context7 검색: "openai streaming api examples"
   - 핵심 학습: AsyncGenerator 패턴, SSE 프로토콜

**적용 결과**:

- [코드 파일 링크]
- [실제 작동 스크린샷]
```

````

**증빙 자료 준비**:
- Context7 검색 스크린샷
- 학습 내용 요약 문서
- 코드에 출처 주석

#### 문제 해결 과정에서의 AI 활용 (10점)

**평가 요소**:
- Sequential Thinking을 통한 체계적 문제 분석
- AI 도구를 활용한 디버깅
- 해결 과정 문서화

**고득점 전략**:

```markdown
## 디버깅 사례: AI 생성 소설이 너무 짧은 문제

### 문제 상황
- 요구사항: 1500-2000 단어
- 실제 결과: 800-1000 단어만 생성됨

### Sequential Thinking 분석 과정

````

Sequential Thinking 질문:
"GPT-4가 요청한 단어 수보다 적게 생성하는 원인을 분석하고 해결 방법을 제시하라"

사고 과정:

1. 가능한 원인 나열
   - max_tokens 설정이 부족?
   - 프롬프트 명확성 문제?
   - temperature/penalty 영향?
   - 한국어 토큰 계산 오차?

2. 각 원인 검증
   - max_tokens: 3000으로 설정됨 → 충분함
   - 프롬프트: "1500-2000단어" 명시 → 모호할 수 있음
   - 한국어: 영어 대비 토큰 소비 차이 존재

3. 해결책 도출
   - 프롬프트 개선: 절대적 지시 추가
   - Few-shot examples: 긴 소설 예시 제공
   - 토큰 여유: max_tokens 4000으로 증가

````

### Context7로 해결책 검색

```bash
/context7 openai output length control techniques

학습 결과:
- System message에 "반드시 N단어 이상 작성" 명시
- Few-shot examples로 기대 길이 학습
- finish_reason 모니터링으로 조기 종료 감지
````

### 해결 코드

```typescript
// 개선 전
const systemPrompt = `
길이: 1,500-2,000단어로 작성하세요.
`;

// 개선 후 (Context7 + Sequential 결과 적용)
const systemPrompt = `
당신은 단편 소설 작가입니다.

중요: 이 소설은 정확히 1,500단어 이상이어야 합니다.
1,500단어 미만으로 끝내지 마세요. 반드시 완전한 이야기를 작성하세요.

예시 길이 참고:
[1,800단어 분량의 예시 소설]

지금부터 1,500단어 이상의 소설을 작성하세요.
`;

// 검증 로직 추가
const wordCount = content.split(/\s+/).length;
if (wordCount < 1500) {
  logger.warn(`Generated ${wordCount} words, retrying...`);
  // 재생성 로직
}
```

### 결과

- Before: 800-1000 단어
- After: 1,600-2,000 단어 ✅
- 해결 시간: 2시간 (Sequential로 체계적 접근)

````

#### 프롬프트 엔지니어링 (10점)

**평가 요소**:
- 구조화된 프롬프트 설계
- Few-shot learning 활용
- 파라미터 최적화
- A/B 테스트 전략

**고득점 전략**:

```typescript
// ✅ 고급 프롬프트 엔지니어링

const PROMPT_VERSION = 'v2.1.0';

// 1. Few-shot Examples 활용
const fewShotExamples = [
  {
    style: "하드보일드 + 느와르 + 반전",
    story: `비는 도시를 적시고, 내 사무실 창문을 두드렸다.
수화기 너머 여자의 목소리는 떨리고 있었다. "그를 찾아주세요. 제발."
나는 담배에 불을 붙이며 대답했다. "주소를 대시오."

[... 1,800 단어 완성 소설 ...]

그가 살아있었다. 하지만 찾던 사람은 내가 아니었다.
그녀가 찾던 건, 죽은 남자가 아니라 살아있는 거짓말이었다.`
  },
  {
    style: "로맨스 + 경쾌한 + 해피엔딩",
    story: `그가 카페 문을 열고 들어온 순간, 시간이 멈췄다.
[... 1,600 단어 ...]
우리는 웃으며 서로의 손을 잡았다. 이것이 시작이었다.`
  }
];

const systemPrompt = `
당신은 뛰어난 한국어 단편 소설 작가입니다.

# 작가 스타일 학습 예시:

${fewShotExamples.map(ex => `
## ${ex.style}
${ex.story}
`).join('\n\n')}

# 이제 당신의 차례입니다.
위 예시들과 같은 수준의 한국어 단편 소설을 작성하세요.
`;

// 2. 파라미터 최적화
const optimizedParams = {
  temperature: 0.9,         // 창의성 최대
  max_tokens: 4000,         // 충분한 여유
  presence_penalty: 0.6,    // 주제 다양성
  frequency_penalty: 0.3,   // 반복 방지
  top_p: 0.95,              // Nucleus sampling
};

// 3. A/B 테스트 프레임워크
const promptVariants = {
  'control': buildPromptV1(),
  'few_shot': buildPromptV2WithExamples(),
  'enhanced': buildPromptV3WithExamplesAndConstraints(),
};

const variant = selectVariant(userId);  // A/B 테스트 배정
const prompt = promptVariants[variant];

// 성능 추적
await prisma.promptMetrics.create({
  data: {
    version: variant,
    storyId: story.id,
    generationTime: elapsed,
    wordCount: story.wordCount,
    userRating: null,  // 나중에 업데이트
  }
});

// 4. 프롬프트 버전 관리
await prisma.promptVersion.create({
  data: {
    version: PROMPT_VERSION,
    systemPrompt: systemPrompt,
    parameters: optimizedParams,
    performance: {
      avgQuality: 4.2,
      avgWordCount: 1750,
      successRate: 0.95,
    }
  }
});
````

#### AI 도구 다양성 및 전략 (10점)

**평가 요소**:

- 여러 AI 도구의 조합 활용
- 각 도구의 적재적소 사용
- 도구 활용 전략 문서화

**고득점 전략**:

```markdown
## AI 도구 조합 전략 (Phase별)

### Phase 1: 프로젝트 초기화
```

도구 조합:

1. Context7: pnpm workspaces, NestJS 초기 설정 학습
2. Sequential: 모노레포 구조 설계 의사결정
3. Magic: 불필요 (백엔드 설정)
4. Playwright: 불필요 (아직 UI 없음)

워크플로우:
Context7 → Sequential → Native Implementation

```

### Phase 2: 인증 시스템
```

도구 조합:

1. Context7: NestJS JWT, Prisma 관계 학습
2. Sequential: JWT vs Session 트레이드오프 분석
3. Magic: 로그인/회원가입 UI 컴포넌트 생성
4. Playwright: 로그인 플로우 E2E 테스트

워크플로우:
Context7 (학습) → Sequential (설계) →
Native (Backend) → Magic (Frontend) →
Playwright (검증)

```

### Phase 4: AI 소설 생성 ⭐
```

도구 조합:

1. Context7: OpenAI API, Streaming 학습
2. Sequential:
   - 프롬프트 구조 설계
   - Few-shot vs Fine-tuning 결정
   - 토큰 최적화 전략
3. Magic:
   - 생성 진행 UI (progress bar, streaming display)
   - 로딩 애니메이션
4. Playwright:
   - 소설 생성 플로우 E2E
   - 스트리밍 동작 검증

워크플로우:
Sequential (프롬프트 설계) →
Context7 (API 패턴 학습) →
Native (AIService 구현) →
Magic (UI 생성) →
Playwright (E2E 검증) →
Sequential (성능 분석 및 개선)

```

## AI 도구 의사결정 매트릭스

| 상황 | Context7 | Sequential | Magic | Playwright |
|------|----------|------------|-------|------------|
| 새 라이브러리 사용 | ✅ 필수 | - | - | - |
| 복잡한 설계 결정 | ✅ 참고 | ✅ 필수 | - | - |
| UI 컴포넌트 필요 | - | - | ✅ 필수 | - |
| 알고리즘 최적화 | ✅ 패턴 | ✅ 분석 | - | - |
| 버그 디버깅 | ✅ 해결책 | ✅ 원인분석 | - | - |
| E2E 테스트 | - | - | - | ✅ 필수 |
| 프롬프트 설계 | ✅ 가이드 | ✅ 필수 | - | - |

## 도구 활용 증빙 자료

### 1. Sequential Thinking 사용 증빙
- 스크린샷: 프롬프트 설계 사고 과정
- 문서: 의사결정 트레이드오프 분석
- 코드 주석: "Sequential 결과 Step 3 적용"

### 2. Context7 사용 증빙
- 학습 일지: 검색한 문서 목록
- 코드 주석: "// From OpenAI Docs (Context7)"
- README: 학습 과정 섹션

### 3. Magic 사용 증빙
- 컴포넌트 파일 히스토리
- 주석: "// Generated with Magic MCP"
- UI 스크린샷

### 4. Playwright 사용 증빙
- 테스트 코드 파일
- 테스트 실행 스크린샷
- CI/CD 통합
```

### 3️⃣ 완성도 (20점) - 목표: 17-19점

#### 계획한 주요 기능 작동 (10점)

**평가 요소**:

- Phase 1-5의 핵심 기능 모두 작동
- 에러 없이 안정적 실행
- 실제 사용 시나리오 커버

**고득점 전략**:

```yaml
검증 체크리스트:

Phase 1 (Infrastructure):
  - ✅ pnpm install 성공
  - ✅ Docker 컨테이너 실행
  - ✅ Frontend dev server 구동
  - ✅ Backend /health 엔드포인트 응답

Phase 2 (Authentication):
  - ✅ 회원가입 성공 (DB에 저장 확인)
  - ✅ 로그인 시 JWT 토큰 발급
  - ✅ Protected route 401 응답
  - ✅ 유효한 토큰으로 인증 통과

Phase 3 (Writers):
  - ✅ 작가 생성 (systemPrompt 포함)
  - ✅ 작가 목록 조회
  - ✅ 작가 이미지 업로드
  - ✅ PUBLIC/PRIVATE 권한 작동

Phase 4 (Story Generation): ⭐ 핵심
  - ✅ OpenAI API 연결 성공
  - ✅ 소설 생성 완료 (1500+ 단어)
  - ✅ 제목 자동 생성
  - ✅ DB에 저장 및 메타데이터
  - ✅ 에러 시 적절한 메시지

Phase 5 (Library):
  - ✅ 소설 목록 pagination
  - ✅ 태그 필터링
  - ✅ 북마크 추가/제거
  - ✅ 소설 읽기 페이지
```

#### 구현의 완성도 (10점)

**평가 요소**:

- 코드 품질
- 에러 핸들링
- Production 준비도
- 테스트 커버리지

**고득점 전략**:

```typescript
// ✅ Production-Grade 코드

// 1. 완벽한 에러 핸들링
async generateStory(dto: GenerateStoryDto, userId: string) {
  try {
    // 1단계: 입력 검증
    const writer = await this.findWriter(dto.writerId);
    if (!writer) {
      throw new NotFoundException(`Writer ${dto.writerId} not found`);
    }

    // 2단계: AI 생성 (재시도 로직)
    const content = await retry(
      () => this.aiService.generateStory(writer.systemPrompt, dto.tags),
      {
        retries: 3,
        factor: 2,
        onRetry: (error, attempt) => {
          this.logger.warn(`Retry ${attempt}/3: ${error.message}`);
        }
      }
    );

    // 3단계: 후처리 및 저장
    const title = await this.aiService.generateTitle(content);
    const wordCount = this.countWords(content);

    // 4단계: DB 저장 (트랜잭션)
    const story = await this.prisma.$transaction(async (tx) => {
      return tx.story.create({
        data: { title, content, wordCount, /* ... */ }
      });
    });

    // 5단계: 성능 모니터링
    await this.metrics.record({
      operation: 'story_generation',
      duration: Date.now() - startTime,
      success: true,
    });

    return story;

  } catch (error) {
    // 에러 로깅
    this.logger.error({
      operation: 'generateStory',
      userId,
      error: error.message,
      stack: error.stack,
    });

    // Sentry 리포트
    Sentry.captureException(error);

    // 적절한 HTTP 예외
    if (error instanceof OpenAIError) {
      throw new ServiceUnavailableException('AI service temporarily unavailable');
    }
    throw error;
  }
}

// 2. 테스트 커버리지
describe('StoryService', () => {
  describe('generateStory', () => {
    it('should generate story successfully', async () => {
      const story = await service.generateStory(dto, userId);
      expect(story.wordCount).toBeGreaterThanOrEqual(1500);
    });

    it('should throw NotFoundException for invalid writer', async () => {
      await expect(
        service.generateStory({ writerId: 'invalid', tags: [] }, userId)
      ).rejects.toThrow(NotFoundException);
    });

    it('should retry on AI service failure', async () => {
      aiService.generateStory.mockRejectedValueOnce(new Error('Timeout'));
      const story = await service.generateStory(dto, userId);
      expect(story).toBeDefined();
    });
  });
});

// 3. Production 설정
// .env.production
NODE_ENV=production
DATABASE_URL=${RAILWAY_DATABASE_URL}
OPENAI_API_KEY=${SECRET_OPENAI_KEY}

// Rate limiting
@Throttle(5, 60)  // AI 엔드포인트 보호

// Monitoring
Sentry.init({ dsn: process.env.SENTRY_DSN });

// Caching
@CacheKey('stories-list')
@CacheTTL(300)

// Logging
logger.info('Story generated', { storyId, userId, duration });
```

### 점수 예측 및 개선 로드맵

#### 현재 Commands만으로 (개선 전)

```
백엔드: 30-32/40 (75-80%)
AI 활용: 21-24/40 (52-60%) 🚨
완성도: 12-14/20 (60-70%)
─────────────────────────────
총점: 63-70/100 (D~C 수준)
```

#### 권장사항 적용 후

```
백엔드: 36-38/40 (90-95%)
  → Phase 6 추가로 에러 처리, 보안, 테스트 완비

AI 활용: 35-38/40 (87-95%) ⭐
  → Commands에 AI 도구 활용 전략 명시
  → Sequential Thinking 설계 과정 문서화
  → Context7 학습 과정 증빙
  → Few-shot + A/B test 프롬프트 엔지니어링

완성도: 17-19/20 (85-95%)
  → 배포 + 모니터링 완비
  → E2E 테스트 자동화
─────────────────────────────
총점: 88-95/100 (A 수준) 🎉
```

---

## 📌 Quick Start for AI Developers

### 첫 Phase 시작하기

```bash
# 1. 프로젝트 초기화
/phase1-init

# 2. Context7로 공식 문서 확인 (자동)
# 3. Sequential Thinking으로 계획 수립 (자동)
# 4. 코드 작성 및 검증 (단계별)
# 5. /verify-phase 1 실행

# ✅ 80% 이상이면 다음 Phase 진행
/phase2-auth
```

### 기억해야 할 3가지

1. **완성된 코드만 작성** - TODO나 stub 코드 금지
2. **모르면 Context7** - 추측하지 말고 공식 문서 확인
3. **매 단계 검증** - 작성 → 컴파일 → 테스트 → 다음 단계

---

**이 가이드라인을 기반으로 일관성 있고 고품질의 ZETA 클론을 개발하세요!**

_마지막 업데이트: 2025년 1월 (AI 개발자 실행 모드 추가)_

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YuuRiLee)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/YuuRiLee)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
