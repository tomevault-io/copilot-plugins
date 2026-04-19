## landingpagetest

> > **이 파일의 목적:** "홈페이지 만들어줘"라고 하면 처음부터 끝까지 순서대로 진행한다.

# CLAUDE.md — 홈페이지 제작 마스터 가이드

> **이 파일의 목적:** "홈페이지 만들어줘"라고 하면 처음부터 끝까지 순서대로 진행한다.
> 
> Claude Code가 전체 프로세스를 관리하고, 사용자가 직접 해야 할 부분은 상세히 안내한다.

---

## 🚀 시작하기

사용자가 홈페이지 제작을 요청하면, 아래 **5개 Phase**를 순서대로 진행한다.

```
Phase 1: 기획 (인터뷰)
    ↓
Phase 2: 프론트엔드 개발
    ↓
Phase 3: 백엔드 연동
    ↓
Phase 4: 배포
    ↓
Phase 5: 관리자 페이지 (선택)
```

**각 Phase 완료 후 사용자에게 확인받고 다음 Phase로 진행한다.**

---

# Phase 1: 기획 (인터뷰)

## Claude Code가 할 일

1. **AskUserQuestionTool**로 인터뷰 진행
2. 사이트 유형 파악 후 맞춤 질문

## 첫 번째 질문: 사이트 유형 파악

```
어떤 유형의 웹사이트를 만드시나요?

A. 판매용 랜딩페이지 (제품/서비스/강의 판매)
B. 회사/브랜드 소개 사이트
C. 포트폴리오/개인 사이트
D. 기타
```

---

## 유형 A: 판매용 랜딩페이지 인터뷰

> 전환율 높은 랜딩페이지를 위해 아래 영역을 **AskUserQuestionTool**로 인터뷰한다.
> 한 번에 2~3개 질문만. 모르면 기본값 제안.

### 1️⃣ 제품 정보 (필수)

| 질문 | 예시 |
|------|------|
| 제품명이 뭔가요? | "첫 런칭을 위한 30일 챌린지" |
| 가격은 얼마인가요? | 199,000원 |
| 제품 형태는요? | 챌린지 / 전자책 / 강의 / 템플릿 / 코칭 / 실물 상품 |

### 2️⃣ 타겟 고객 (핵심만)

| 질문 | 왜 필요한가 |
|------|------------|
| 누구를 위한 제품인가요? (직업, 상황) | 헤드라인 타겟팅 |
| 이 사람의 가장 큰 고충은? | 공감 섹션 작성 |
| 이 사람이 진짜 원하는 건? | 베네핏 강조 |
| 구매를 망설이는 이유는? | FAQ, 보장 문구 |

### 3️⃣ 오퍼 구성

| 질문 | 예시 |
|------|------|
| 핵심 제품 외 보너스가 있나요? | 체크리스트, 템플릿, 1:1 피드백 등 |
| 환불/보장 정책은? | "7일 무조건 환불" |
| 희소성 요소는? | 선착순 50명 / 48시간 한정 / 가격 인상 예정 |
| CTA 버튼 문구는? | "지금 바로 시작하기" |

### 4️⃣ 버튼 & 링크 설정

| 질문 | 설명 |
|------|------|
| "신청하기" 버튼 클릭 시 이동할 링크는? | 결제 페이지 URL (예: 클래스101, 탈잉, 자체 결제 등) |
| 무료 상담 폼이 필요한가요? | 예/아니오 → 필요시 상담 폼 인터뷰 진행 |

### 5️⃣ 브랜드 로고 (선택)

| 질문 | 안내 |
|------|------|
| 브랜드 로고가 있으신가요? | 있으면 아래 안내대로 폴더에 넣어주세요 |

```
📋 브랜드 로고 등록 가이드

1. 프로젝트 폴더에 brand_logo 폴더를 만드세요
   
   📁 프로젝트폴더/
   └── 📁 brand_logo/
       └── logo.png (또는 .svg, .jpg)

2. 로고 파일을 brand_logo 폴더에 넣어주세요

3. 권장 사항:
   - 형식: PNG (투명 배경) 또는 SVG
   - 크기: 가로 200~500px
   - 파일명: logo.png, logo.svg 등

4. 완료되면 알려주세요!
   → 랜딩페이지 헤더에 자동으로 적용됩니다
```

### 6️⃣ 오리진 스토리 (선택, 강력 추천)

| 질문 | 용도 |
|------|------|
| 과거에 어떤 상황이었나요? (Before) | 공감 유발 |
| 어떤 고통/실패를 겪었나요? (Struggle) | 신뢰 구축 |
| 전환점은 뭐였나요? (Turning Point) | 해결책 연결 |
| 지금은 어떻게 달라졌나요? (After) | 결과 증명 |

---

## 무료 상담 폼 인터뷰

> 무료 상담 폼이 필요하다면, 설문지 형식의 폼을 만들기 위해 추가 인터뷰 진행

### 상담 폼 질문 설계

| 질문 | 예시 |
|------|------|
| 폼에서 어떤 정보를 받고 싶나요? | 이름, 연락처, 이메일, 문의 내용 등 |
| 추가로 물어볼 질문이 있나요? | "현재 상황은?", "예산은?", "희망 일정은?" 등 |
| 폼 제출 후 보여줄 메시지는? | "접수 완료! 24시간 내 연락드립니다" |
| 폼 제출 알림을 받을 이메일은? | admin@example.com |

### 상담 폼 기능

```
✅ 설문지 형식으로 단계별 질문 가능
✅ 제출 시 Supabase에 저장
✅ 관리자 페이지에서 신청 내역 확인 가능
✅ (선택) 제출 시 관리자 이메일 알림
✅ (선택) 제출자에게 자동 확인 메일 발송
```

---

### 랜딩페이지 인터뷰 완료 체크

```
□ 제품명, 가격, 형태 확정
□ 타겟 고객 정의 완료
□ 고충 & 욕구 파악 완료
□ 오퍼 구성 (보너스, 보장, 희소성) 확정
□ 버튼 링크 설정 (신청하기 → 결제 페이지)
□ 무료 상담 폼 설계 (필요 시)
□ 브랜드 로고 등록 안내 (brand_logo 폴더)
□ 오리진 스토리 수집 (선택)
□ 디자인 방향/참조 이미지 확인
```

### 랜딩페이지 출력 구조

인터뷰 완료 후 아래 구조로 카피 작성 → HTML 제작:

```
1. 헤드라인 (3가지 버전 제안)
2. 서브 헤드라인
3. 고통 공감 섹션
4. 해결책 제시
5. 제품 소개 + 베네핏 리스트
6. 오퍼 정리 (시각적으로)
7. 보장 문구
8. FAQ 5개
9. 최종 CTA (신청하기 버튼 → 설정된 링크로 이동)
10. 무료 상담 폼 (설문지 형식)
11. P.S.
```

**톤:** 타겟의 언어로, 진솔하고 공감 가게

---

## 유형 B/C/D: 일반 웹사이트 인터뷰

| 영역 | 확인 사항 |
|------|----------|
| **1. 타겟 & 목적** | 누구를 위한, 어떤 목적의 사이트인지 |
| **2. 필요한 페이지** | 메인, 소개, 서비스, 문의, 블로그 등 |
| **3. 핵심 기능** | 상담 폼, 예약 시스템, 결제 등 |
| **4. 디자인 방향** | 참조 이미지, 색상, 톤앤매너 |
| **5. 백엔드 필요 여부** | 폼 저장, 이메일 발송 등 |
| **6. 관리자 페이지 필요 여부** | 상담 관리, 블로그 관리 등 |

---

## 인터뷰 규칙 (공통)

- **AskUserQuestionTool** 사용
- 한 번에 **최대 2~3개 질문**
- 이미 정한 내용을 **다시 묻지 말 것**
- "모르겠다" 또는 "알아서 해줘" → **합리적 기본값 제안 후 확인**
- **인터뷰 종료** 시점: 모든 필수 영역 완료 또는 사용자가 "인터뷰 종료"

**→ 사용자 확인 후 Phase 2로 진행**

---

# Phase 2: 프론트엔드 개발

## Claude Code가 할 일

1. `brand_logo/` 폴더 확인 → 로고 있으면 헤더에 적용
2. HTML/CSS 코드 작성
3. 반응형 디자인 적용
4. 버튼 링크 연결 (신청하기 → 결제 페이지)
5. 스크린샷 비교 검증 (최소 2회)

### 로고 적용

```html
<!-- brand_logo 폴더에 로고가 있으면 자동 적용 -->
<header>
  <img src="./brand_logo/logo.png" alt="브랜드 로고" class="h-10">
  <!-- ... -->
</header>
```

## 참조 이미지 규칙

- **참조 있음**: 정확히 일치시킬 것. 개선/추가 금지.
- **참조 없음**: 아래 가드레일 따라 고퀄리티 디자인

## 안티-제너릭 가드레일

| 항목 | ❌ 금지 | ✅ 해야 할 것 |
|------|--------|--------------|
| **색상** | Tailwind 기본색 (indigo-500, blue-600) | 커스텀 브랜드 색상 |
| **그림자** | `shadow-md` 단일 사용 | 색상 틴트된 레이어드 그림자 |
| **폰트** | 제목/본문 같은 폰트 | 디스플레이 + 산세리프 조합 |
| **애니메이션** | `transition-all` | `transform`, `opacity`만 |
| **인터랙션** | hover 상태 없음 | hover, focus, active 전부 |

## 버튼 상태 디자인 (필수)

모든 버튼에 아래 상태를 구현할 것:

```css
/* 기본 상태 */
.btn { background: #브랜드색상; }

/* 마우스 올림 */
.btn:hover { background: #진한색상; transform: scale(1.02); }

/* 포커스 (키보드 접근성) */
.btn:focus { outline: none; ring: 2px; ring-color: #브랜드색상; }

/* 클릭 중 */
.btn:active { transform: scale(0.98); }

/* 비활성화 */
.btn:disabled { opacity: 0.5; cursor: not-allowed; }
```

## 버튼 링크 연결

```html
<!-- 신청하기 버튼 → 결제 페이지로 이동 -->
<a href="https://결제페이지URL.com" target="_blank">
  <button class="btn-primary">지금 바로 신청하기</button>
</a>

<!-- 무료 상담 버튼 → 페이지 내 폼으로 스크롤 -->
<a href="#consultation-form">
  <button class="btn-secondary">무료 상담 신청</button>
</a>
```

## 무료 상담 폼 (설문지 형식)

```html
<!-- 설문지 형식 상담 폼 예시 -->
<form id="consultation-form">
  <!-- Step 1: 기본 정보 -->
  <div class="form-step">
    <label>이름</label>
    <input type="text" name="name" required>
    
    <label>연락처</label>
    <input type="tel" name="phone" required>
    
    <label>이메일</label>
    <input type="email" name="email" required>
  </div>
  
  <!-- Step 2: 추가 질문 (인터뷰에서 설계한 질문들) -->
  <div class="form-step">
    <label>현재 상황을 알려주세요</label>
    <textarea name="situation"></textarea>
    
    <label>희망하는 결과는 무엇인가요?</label>
    <textarea name="goal"></textarea>
  </div>
  
  <button type="submit">상담 신청하기</button>
</form>
```

## 출력 기본값

- 단일 `index.html`, 스타일 인라인
- Tailwind CSS CDN: `<script src="https://cdn.tailwindcss.com"></script>`
- 플레이스홀더 이미지: `https://placehold.co/너비x높이`
- 모바일 퍼스트 반응형

## 스크린샷 검증

```
1. localhost에서 서빙 (node serve.mjs)
2. 스크린샷 촬영 (node screenshot.mjs http://localhost:3000)
3. 참조와 비교 → 불일치 수정
4. 최소 2회 반복
```

## Phase 2 완료 체크

```
□ 모든 페이지 HTML/CSS 완성
□ 반응형 디자인 적용
□ 버튼 상태 (hover, focus, active, disabled) 구현
□ 신청하기 버튼 → 결제 페이지 링크 연결
□ 무료 상담 폼 UI 완성 (설문지 형식)
□ 스크린샷 비교 검증 완료 (최소 2회)
```

**→ 사용자 확인 후 Phase 3로 진행**

---

# Phase 3: 백엔드 연동

> **백엔드가 필요 없으면 Phase 4로 건너뛴다.**

---

## 왜 백엔드가 필요한가?

```
[백엔드 없이]
사용자가 상담 폼 작성 → 제출 → ❌ 아무 일도 안 일어남

[백엔드 있으면]
사용자가 상담 폼 작성 → 제출 → ✅ 데이터 저장 → ✅ 관리자에게 알림
                              → ✅ 관리자 페이지에서 확인 가능
```

---

## 3-1. Supabase 설정 (데이터 저장)

### Supabase란? 왜 쓰나요?

```
🤔 Supabase = 클라우드 데이터베이스 + 자동 API

✅ 왜 필요한가:
- 상담 폼 데이터를 저장할 "창고"가 필요함
- 엑셀처럼 데이터를 저장하지만, 웹사이트에서 자동으로 접근 가능
- 관리자 페이지에서 신청 내역을 볼 수 있음

✅ 왜 Supabase인가:
- 무료 티어가 넉넉함 (월 500MB, 5만 행)
- 설정이 간단함 (5분이면 완료)
- SQL 몰라도 됨 (우리가 제공)
```

### 👤 사용자가 직접 할 일

```
📋 Supabase 설정 가이드

1. https://supabase.com 접속 → 회원가입/로그인

2. "New Project" 클릭
   - 프로젝트 이름: [프로젝트명 제안]
   - 데이터베이스 비밀번호: 안전한 비밀번호 설정
   - Region: Northeast Asia (Seoul)
   - "Create new project" 클릭

3. 프로젝트 생성 완료 후 (2~3분 소요)
   - Settings → API 클릭
   - 아래 두 값을 복사해서 알려주세요:
   
   ✅ Project URL: https://xxxxxxxx.supabase.co
   ✅ anon public key: eyJhbGci...
```

### 🤖 Claude Code가 할 일 (키 제공 후)

1. Supabase 클라이언트 설정
2. 테이블 생성 SQL 제공
3. 폼 → Supabase 저장 연동
4. 관리자 페이지에서 데이터 조회 연동

---

## 테이블 생성 SQL

### SQL이란? 왜 해야 하나요?

```
🤔 SQL = 데이터베이스에게 명령하는 언어

✅ 왜 테이블을 만들어야 하나:
- 상담 폼 데이터를 저장하려면 "그릇"이 필요함
- 엑셀로 치면 "시트 만들고 열 제목 정하는 것"
- 이름, 이메일, 연락처, 문의 내용 등 어떤 정보를 저장할지 정의

✅ 한 번만 하면 됨:
- 처음 한 번 실행하면 끝
- 이후 폼 제출할 때마다 자동으로 행이 추가됨
```

### SQL 코드 (Supabase SQL Editor에서 실행)

```sql
-- 상담 신청 테이블 만들기
-- (엑셀에서 새 시트 만들고 열 제목 정하는 것과 같음)

CREATE TABLE consultations (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,  -- 고유 번호 (자동 생성)
  name TEXT NOT NULL,                             -- 이름 (필수)
  email TEXT NOT NULL,                            -- 이메일 (필수)
  phone TEXT,                                     -- 연락처
  situation TEXT,                                 -- 현재 상황 (설문 질문)
  goal TEXT,                                      -- 희망 결과 (설문 질문)
  message TEXT,                                   -- 추가 문의 내용
  status TEXT DEFAULT 'pending',                  -- 처리 상태 (대기중/완료)
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW() -- 접수 시간 (자동)
);

-- 보안 설정
ALTER TABLE consultations ENABLE ROW LEVEL SECURITY;

-- 누구나 폼 제출 가능하도록 허용
CREATE POLICY "Anyone can insert" ON consultations
  FOR INSERT WITH CHECK (true);
```

---

## 3-2. Resend 설정 (이메일 발송)

### Resend란? 왜 쓰나요?

```
🤔 Resend = 코드로 이메일 보내는 서비스

✅ 왜 필요한가:
- 상담 신청이 들어오면 관리자에게 알림 메일 보내기
- 신청자에게 "접수 완료" 자동 확인 메일 보내기
- 수동으로 Gmail 열어서 보낼 필요 없음

✅ 왜 Resend인가:
- 무료 티어: 월 3,000건 발송
- 설정 간단 (API 키만 받으면 됨)
- 스팸 처리 안 됨 (신뢰성 높음)
```

### 👤 사용자가 직접 할 일

```
📋 Resend 설정 가이드

1. https://resend.com 접속 → 회원가입/로그인
2. API Keys → Create API Key
3. 표시되는 키 복사 (한 번만 표시됨!)
   ✅ API Key: re_xxxxxxxx...
```

### 🤖 Claude Code가 할 일 (키 제공 후)

1. 이메일 발송 API 라우트 생성
2. 폼 제출 시 자동 발송 연동:
   - 관리자에게: "새 상담 신청이 접수되었습니다"
   - 신청자에게: "상담 신청이 완료되었습니다"

### 📧 이메일 발송 구조 (FROM vs TO)

```
┌─────────────────────────────────────────────────────────┐
│  FROM (보내는 사람)     →    TO (받는 사람)              │
│  ─────────────────          ─────────────               │
│  Resend 도메인 필요          아무 이메일 OK              │
└─────────────────────────────────────────────────────────┘
```

| 구분 | 설명 | 예시 |
|------|------|------|
| **FROM (발신)** | 도메인 인증 필요. 안 하면 `onboarding@resend.dev`로 발송됨 | `noreply@내도메인.com` |
| **TO (수신)** | 아무 이메일이나 가능. Resend 계정과 무관 | `admin@gmail.com` |

```javascript
// 실제 코드 예시
await resend.emails.send({
  from: 'onboarding@resend.dev',  // Resend 기본 발신 (도메인 인증 안 했을 때)
  to: 'myemail@gmail.com',        // 👈 알림 받을 이메일 (아무거나 OK)
  subject: '새 상담 신청이 접수되었습니다',
  html: '...'
});
```

**💡 정리:**
- 폼 제출 알림 받을 이메일 = **아무 이메일**이나 가능 (Gmail, Naver 등)
- Resend 계정 이메일 = 로그인용일 뿐, 발송/수신과 무관
- 도메인 인증 = 발신 주소를 `내도메인.com`으로 하고 싶을 때만 필요

---

## 3-3. 환경변수 정리

```bash
# .env.local
NEXT_PUBLIC_SUPABASE_URL=https://xxxxxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGci...
RESEND_API_KEY=re_xxxxxxxx...
```

### ⚠️ 보안 규칙

- ❌ API 키를 코드에 직접 입력 금지
- ❌ .env 파일을 GitHub에 커밋 금지
- ✅ .gitignore에 .env 추가 필수

## Phase 3 완료 체크

```
□ Supabase 설정 완료
□ 테이블 생성 SQL 실행 완료
□ 폼 제출 → DB 저장 테스트 성공
□ 관리자 페이지에서 신청 내역 확인 가능
□ (선택) Resend 설정 완료
□ (선택) 이메일 발송 테스트 성공
□ .env.local 생성, .gitignore 확인
```

**→ 사용자 확인 후 Phase 4로 진행**

---

# Phase 4: 배포 (Vercel)

## 왜 GitHub와 Vercel이 필요한가?

```
🤔 지금 상태:
- 내 컴퓨터에서만 볼 수 있음 (localhost)
- 다른 사람은 접속 불가

✅ GitHub가 필요한 이유:
- 코드를 인터넷에 올려두는 "저장소"
- 백업 역할 (컴퓨터 고장나도 코드 안전)
- Vercel이 여기서 코드를 가져감

✅ Vercel이 필요한 이유:
- 코드를 실제 웹사이트로 만들어줌
- 전 세계 어디서든 접속 가능한 URL 제공
- 무료로 HTTPS (보안 연결) 제공
- 코드 수정하면 자동으로 재배포

📍 흐름:
내 컴퓨터 → GitHub (코드 저장) → Vercel (웹사이트 배포)
                                    ↓
                        https://내사이트.vercel.app
```

---

## 4-1. GitHub 연결

### 👤 사용자가 직접 할 일

```
📋 GitHub 저장소 생성

1. https://github.com → 로그인
2. "+" → "New repository"
3. 생성된 URL 알려주세요:
   ✅ https://github.com/[username]/[repo-name]
```

### 🤖 Claude Code가 할 일

```bash
git init && git add . && git commit -m "Initial commit"
git remote add origin [URL] && git push -u origin main
```

---

## 4-2. Vercel 배포

### 👤 사용자가 직접 할 일

```
📋 Vercel 배포 가이드

1. https://vercel.com → GitHub 로그인
2. "Add New..." → "Project"
3. 저장소 선택 → Import → Deploy
4. 완료 후 URL 알려주세요:
   ✅ https://[프로젝트명].vercel.app
```

---

## 4-3. 환경변수 등록

### 왜 Vercel에도 환경변수를 등록하나요?

```
🤔 .env.local 파일은 내 컴퓨터에만 있음
- Vercel 서버는 이 파일을 모름
- Vercel에 따로 알려줘야 함

✅ Vercel 환경변수 = Vercel 서버에게 API 키 알려주기
```

### 👤 사용자가 직접 할 일

```
Vercel → Settings → Environment Variables

NEXT_PUBLIC_SUPABASE_URL = [값]
NEXT_PUBLIC_SUPABASE_ANON_KEY = [값]
RESEND_API_KEY = [값]

→ 저장 후 Redeploy
```

---

## 4-4. 커스텀 도메인 (선택)

```
Vercel → Settings → Domains → 도메인 입력

DNS 설정:
- A 레코드: @ → 76.76.21.21
- CNAME: www → cname.vercel-dns.com
```

## Phase 4 완료 체크

```
□ GitHub 푸시 완료
□ Vercel 배포 완료
□ 환경변수 등록 및 재배포
□ 배포된 사이트에서 폼 제출 테스트 성공
□ (선택) 커스텀 도메인 연결
```

**→ 사용자 확인 후 Phase 5로 진행 (필요 시)**

---

# Phase 5: 관리자 페이지 (선택)

> **필요 없으면 건너뛴다.**

## 🤖 Claude Code가 할 일

1. 관리자 페이지 생성
2. 비밀번호 보호 구현
3. Supabase 연동 (상담 신청 목록 조회)

## 관리자 페이지 구성

| 탭 | 기능 |
|-----|------|
| **상담 관리** | 신청 목록 확인, 상태 변경 (대기중 → 완료) |
| **상세 보기** | 각 신청의 설문 응답 전체 확인 |
| **통계** | 총 신청 수, 오늘 신청 수 등 |

## 상담 관리 화면 예시

```
┌─────────────────────────────────────────────────────────┐
│  📋 상담 신청 관리                                       │
├─────────────────────────────────────────────────────────┤
│  이름      | 이메일           | 상태   | 접수일   | 상세 │
│  ─────────────────────────────────────────────────────  │
│  김철수    | kim@email.com   | 대기중 | 2024.01 | [보기] │
│  이영희    | lee@email.com   | 완료   | 2024.01 | [보기] │
│  박민수    | park@email.com  | 대기중 | 2024.01 | [보기] │
└─────────────────────────────────────────────────────────┘
```

## 보안 필수

```
□ 비밀번호 보호 적용
□ 추측 어려운 경로 (/dashboard-x7k2 등)
□ 세션 타임아웃 설정
```

## Phase 5 완료 체크

```
□ 관리자 페이지 코드 완성
□ 비밀번호 보호 동작 확인
□ Supabase에서 상담 신청 목록 조회 성공
□ 상태 변경 기능 동작 확인
□ 배포 및 테스트 완료
```

---

# 🔒 보안 체크리스트

| 영역 | 항목 |
|------|------|
| **인증** | AuthN/AuthZ 분리, 최소 권한 |
| **공격 방어** | CORS, CSRF, XSS, SQLi |
| **통신** | HTTPS, 보안 헤더, 쿠키 보안 |
| **운영** | Rate Limit, Secret 관리, Audit Log |

---

# ⚠️ 엄격한 규칙

## 절대 금지

- 인터뷰 없이 코드 작성 시작
- API 키를 코드에 직접 입력
- 관리자 페이지 비밀번호 없이 배포
- `transition-all` 사용
- Tailwind 기본 blue/indigo 메인 색상
- 버튼에 hover/focus/active 상태 없이 배포

## 반드시 할 것

- **AskUserQuestionTool**로 인터뷰 진행
- 각 Phase 완료 후 사용자 확인
- 사용자가 할 일은 상세 가이드 제공
- 환경변수는 .env 파일에만 저장
- 모든 버튼에 상태별 스타일 적용

---

# 🎯 전체 프로세스 요약

```
Phase 1: 기획 ─────────────────────────────────────┐
  └── AskUserQuestionTool로 인터뷰                  │
  └── 판매 랜딩페이지: 제품/타겟/오퍼/버튼링크/상담폼 │
        ↓                                          │
Phase 2: 프론트엔드 개발 ──────────────────────────────┤
  └── HTML/CSS, 버튼 상태, 폼 UI                    │
  └── 신청하기 → 결제 페이지 링크                    │
  └── 무료 상담 폼 → 설문지 형식                     │
        ↓                                          │
Phase 3: 백엔드 연동 ─────────────────────────────────┤
  └── Supabase: 상담 폼 데이터 저장                 │  🤖 Claude Code
  └── Resend: 이메일 알림 발송                      │  👤 사용자 협업
        ↓                                          │
Phase 4: 배포 ─────────────────────────────────────┤
  └── GitHub: 코드 저장소                          │
  └── Vercel: 웹사이트 배포                        │
        ↓                                          │
Phase 5: 관리자 페이지 (선택) ─────────────────────────┘
  └── 상담 신청 목록 확인 & 관리

✅ 완료!
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/drencher) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
