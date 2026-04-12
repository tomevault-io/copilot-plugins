## workflow-kit

> - 저장소: djheksa/workflow-kit

# Workflow Kit

## GitHub
- 저장소: djheksa/workflow-kit
- 계정: 개인 (`djheksa`) — push 전 `gh auth switch --user djheksa` 확인

## 함께 업데이트할 파일
- 새 SwiftBar 플러그인 추가 시: `README.md`의 "SwiftBar 메뉴바 플러그인" 섹션 + 프로젝트 구조 트리
- 새 기능(feature) 추가 시: `features/README.md` 카탈로그 + `README.md` 기능 테이블 + `features/{기능명}/README.md`
- 새 훅 스크립트 추가 시: `hooks/README.md` 훅 목록 + `features/README.md` 카탈로그
- 새 기능을 `/kit-setup`에 추가 시: `.claude/commands/kit-setup.md`의 기능 선택지 + 설정 절차 + 완료 요약 테이블
- 워크플로우 변경 시: 이 파일(`CLAUDE.md`)의 워크플로우 섹션 반영

Claude Code에서 사용하는 워크플로우 도구. 이 프로젝트를 클론하고 해당 경로에서 Claude Code를 실행하면 정의된 워크플로우를 사용할 수 있음.

## 작업 원칙

- 모르거나 불확실한 사항은 AskUserQuestion을 적극 활용하여 사용자에게 선택지를 제공할 것 (토큰 절약 및 올바른 플로우 실행 목적)
- 추측하지 말고, 확인이 필요한 사항은 반드시 질문할 것
- **`.claude.local.md`에 정의된 값(프로젝트 키, 스페이스 키, 경로 등)은 절대 사용자에게 다시 묻지 말 것** — 파일을 먼저 읽고 그 값을 그대로 사용
- 워크플로우 실행 전 반드시 `.claude.local.md`를 읽어 Jira 프로젝트 키, Confluence 스페이스 키, 프로젝트 경로를 확인할 것

## 세션 시작 동작

첫 번째 메시지가 `[headless 모드 - 봇 자동 실행]`으로 시작하면 아래 세션 시작 로직을 **완전히 건너뛰고** 즉시 워크플로우를 실행한다.

세션의 **첫 번째 메시지**에서 아래 순서로 실행한다:

1. `/tmp/claude-session-workflow-kit.md` 존재 시 읽어서 이전 세션 컨텍스트 파악
   - 어떤 파일을 수정했는지, 마지막으로 무엇을 작업 중이었는지 확인
   - 사용자에게 별도로 보고하지 않고 조용히 내부 컨텍스트로만 활용
2. `.claude.local.md` 존재 여부를 확인하고 동작을 분기한다.

### `.claude.local.md` 없을 때 → 기능 소개 출력

아래 형식으로 기능 카탈로그를 출력한 후 초기 설정을 안내한다:

```
## workflow-kit 사용 가능한 기능

### 즉시 사용 가능 (설정 불필요)
[✅] Mac 답변 알림       — hooks/notify-on-stop.sh → hooks/README.md 참조
[✅] Claude 사용량 모니터 — SwiftBar 심볼릭 링크 설정 → features/claude-usage-monitor/ 참조
[✅] 코드 작성 표준       — 자동 적용 중
[✅] Confluence 규칙     — 자동 적용 중

### 설정 필요
[⬜] Jira/Confluence 워크플로우 — .claude.local.md + Atlassian MCP 필요
[⬜] Slack 워크플로우 봇        — bot/.env (Slack 토큰) 필요
[⬜] AWS 인프라 모니터          — AWS CLI + SwiftBar 필요

전체 기능 목록: features/README.md
```

이어서 초기 설정 안내를 출력한다:

```
설정 마법사: /kit-setup
(기능을 선택하면 단계별로 안내합니다)

설정 상태가 궁금하면 "도움" 이라고 입력하세요.
```

### `.claude.local.md` 있을 때 → 조용히 시작

별도 출력 없이 바로 작업 대기.

### 특정 단어 입력 시 → 기능 목록 재출력

사용자가 아래 단어 중 하나를 입력하면 기능 카탈로그를 다시 출력한다:
- `기능 목록`, `feature list`, `features`, `기능 소개`, `도움말`

### "도움" 입력 시 → 환경 검증

1. `.claude.local.md` 존재 여부 확인
2. 백엔드/프론트엔드 경로 디렉토리 존재 여부 검증
3. Atlassian MCP 도구 사용 가능 여부 확인
4. Slack MCP 도구 사용 가능 여부 확인
5. 실패 항목별 해결 방법 안내

## 사전 요구사항

### 1. Claude Code 설치
- https://claude.ai/code 에서 설치

### 2. Atlassian MCP 설정

Claude Code 설정 파일에 Atlassian MCP를 추가해야 합니다.

**설정 파일 위치:**
- macOS: `~/.claude/mcp_settings.json`
- Windows: `%APPDATA%\claude\mcp_settings.json`

**설정 내용:**
```json
{
  "mcpServers": {
    "atlassian": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-atlassian"],
      "env": {
        "ATLASSIAN_SITE": "your-company.atlassian.net",
        "ATLASSIAN_USER_EMAIL": "your-email@your-company.com",
        "ATLASSIAN_API_TOKEN": "your-api-token"
      }
    }
  }
}
```

**API 토큰 발급 방법:**
1. https://id.atlassian.com/manage-profile/security/api-tokens 접속
2. "Create API token" 클릭
3. 라벨 입력 (예: "Claude Code MCP")
4. 생성된 토큰 복사하여 설정 파일에 입력

### 3. Claude Code 재시작
설정 후 Claude Code를 재시작해야 MCP가 적용됩니다.

### 4. Slack Bot 설정 (자동 트리거 사용 시)

봇을 통한 자동 워크플로우(Slack 메시지 감지 → 티켓 생성, 이모지 → 분석)를 사용하려면 `bot/.env` 설정이 필요합니다.

```bash
cd bot
cp .env.example .env
```

**필수 환경변수:**

| 환경변수 | 설명 |
|---------|------|
| `SLACK_BOT_TOKEN` | Slack Bot Token (`xoxb-...`) |
| `SLACK_APP_TOKEN` | Slack App-Level Token (`xapp-...`, Socket Mode용) |
| `ATLASSIAN_SITE` | Atlassian 사이트 URL (예: `your-company.atlassian.net`) |
| `JIRA_PROJECT_KEY` | Jira 프로젝트 키 (예: `MYPROJECT`) |

> 필수 환경변수가 누락되면 봇이 시작되지 않습니다. `bot/.env.example`에 전체 목록이 있습니다.

---

## 사용 가능한 워크플로우

### 워크플로우 1: 티켓 생성 (자동 / 수동)

Slack 메시지 또는 수동 입력으로 Jira 티켓을 생성하고, 요청자에게 DM 알림을 발송합니다.

**트리거 방식:**
- **자동**: Slack Workflow Form → 특정 채널에 메시지 게시 → Bot이 감지 → `claude -p`로 실행
- **수동**: Claude Code에서 직접 입력

**Slack Workflow Form 형식 (자동 트리거 시 Bot이 파싱하는 원본):**
```
📋 요청사항
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
유형 :  기능 추가
제목 : 멀티 수납취소 기능 개발 요청
우선순위 : 🟡 보통
관련 화면 : 수납관리
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
상세 설명

[이슈 내용 / 요청 내용]
시스템 개선건 입니다

[발생 현황 / 요청 배경]
시스템 개선건 입니다

[AS-IS]
현재는 한달 한건 씩만 취소가능

[TO-BE]
체크박스가 생성되어 멀티 수납취소 기능 개발

[기대 효과]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
요청자 : @홍길동 (Gildong Hong)
담당자 : @이한상(Hansang Lee)
```

**우선순위 이모지 매핑:**
- 🔴 긴급
- 🟠 높음
- 🟡 보통
- 🟢 낮음

**필드 참고:**
- 담당자: 선택 입력 (비어있을 수 있음)
- 상세 설명의 하위 섹션([AS-IS], [TO-BE] 등): 비어있을 수 있음

**수동 사용 방법:**
수동으로 실행할 때도 위 Form 형식을 그대로 붙여넣거나, 아래 간략 형식을 사용할 수 있음:
```
티켓 생성해줘

유형: 버그 (또는: 기능 추가 / 개선 사항 / 기타)
제목: 계약 상태 변경 시 오류 발생
우선순위: 높음 (또는: 긴급 / 보통 / 낮음)
관련 화면: 계약관리
상세 설명: 계약 목록에서 일괄 변경 시 "변경하지 않음" 선택해도 값이 변경됨
요청자: 홍길동
담당자: 이한상 (선택)
```

**실행 결과:**
1. Jira 티켓 생성 (`.claude.local.md`에 설정된 프로젝트, 상태: Backlog)
2. 원본 메시지에 스레드 댓글:
```
티켓 생성 완료: <https://{ATLASSIAN_SITE}/browse/{PROJECT_KEY}-XXX|{PROJECT_KEY}-XXX>
```
3. Slack DM 알림 발송:
   - 요청자 DM:
```
:clipboard: Jira 티켓 생성 완료

• 티켓: <https://{ATLASSIAN_SITE}/browse/{PROJECT_KEY}-XXX|{PROJECT_KEY}-XXX>
• 제목: [기능] XXX
• 우선순위: 보통
• 담당자: OOO (없으면 미지정)
• 상태: Backlog
```
   - 담당자 DM (담당자가 있고, **요청자와 다른 사람인 경우만**):
```
:clipboard: Jira 티켓 배정 알림

• 티켓: <https://{ATLASSIAN_SITE}/browse/{PROJECT_KEY}-XXX|{PROJECT_KEY}-XXX>
• 제목: [기능] XXX
• 요청자: OOO
• 우선순위: 보통
• 상태: Backlog
```

> **중복 DM 방지**: 요청자와 담당자가 동일 인물이면 요청자 DM만 발송하고 담당자 DM은 생략할 것.
> **링크 형식**: Slack 메시지 내 URL은 반드시 `<URL|표시텍스트>` 형식으로 하이퍼링크 처리할 것 (plain URL 금지)

### 워크플로우 2: AI 분석 + 문서 생성 (이모지 트리거 / 수동)

생성된 티켓에 대해 코드베이스를 분석하고 Confluence 분석 문서를 생성합니다.

**트리거 방식:**
- **이모지 트리거**: 워크플로우 1로 생성된 Slack 메시지에 🤖 이모지를 추가하면 Bot이 감지 → `claude -p`로 실행
- **수동**: Claude Code에서 직접 입력

**수동 사용 방법:**
```
분석해줘

티켓: {PROJECT_KEY}-123
분석 범위: 백엔드 (또는: 프론트엔드 / 전체)
```

**실행 결과:**
1. 코드베이스 분석 (`.claude.local.md`에 정의된 프로젝트 경로 참조)
2. Confluence 분석 문서 생성 (`.claude.local.md`에 설정된 스페이스 > 분석 폴더)
3. Jira 티켓에 Confluence 분석 문서 링크 연결
4. Jira 티켓 설명에 AI 분석 요약 추가
5. Jira 티켓 상태 전환: `Backlog` → `AI 분석 시작` → `분석 완료` (AI Reviewed)
6. 원본 메시지에 스레드 댓글:
```
*<https://{ATLASSIAN_SITE}/browse/{PROJECT_KEY}-XXX|{PROJECT_KEY}-XXX>* 분석이 완료되었습니다.
분석 문서: <{Confluence URL}|🤖 {Confluence 문서 제목}>
```
7. Slack DM 알림 발송:
   - 트리거한 사람 DM:
```
:mag: AI 분석 완료

• 티켓: <https://{ATLASSIAN_SITE}/browse/{PROJECT_KEY}-XXX|{PROJECT_KEY}-XXX>
• 제목: XXX
• 담당자: OOO (없으면 미지정)
• 상태: AI Reviewed
• 분석 문서: <{Confluence URL}|🤖 {Confluence 문서 제목}>
```
   - 담당자 DM (있는 경우):
```
:mag: Jira 티켓 분석 완료 — 담당자 배정

• 티켓: <https://{ATLASSIAN_SITE}/browse/{PROJECT_KEY}-XXX|{PROJECT_KEY}-XXX>
• 제목: XXX
• 요청자: OOO
• 우선순위: 보통
• 상태: AI Reviewed
• 분석 문서: <{Confluence URL}|🤖 {Confluence 문서 제목}>
```

> **링크 형식**: Slack 메시지 내 URL은 반드시 `<URL|표시텍스트>` 형식으로 하이퍼링크 처리할 것 (plain URL 금지)

### 워크플로우 3: 코드베이스 분석 (단독)

특정 기능이나 버그에 대해 티켓 없이 코드만 분석합니다. (수동 전용)

**사용 방법:**
```
코드 분석해줘

분석 대상: 계약 상태 일괄 변경 기능
분석 범위: 백엔드 (또는: 프론트엔드 / 전체)
```

---

## 연동 대상

연동 대상은 `.claude.local.md`에서 설정합니다:

| 항목 | 설정 키 | 설명 |
|------|---------|------|
| Jira 프로젝트 | `{PROJECT_KEY}` | Jira 프로젝트 키 (예: MYPROJECT) |
| Confluence 스페이스 | `{SPACE_KEY}` | Confluence 스페이스 키 (예: TEAM) |
| 분석 문서 위치 | `{ANALYSIS_FOLDER}` | 분석 문서 저장 폴더 (예: 🤖 AI Analysis) |
| Atlassian 사이트 | `{ATLASSIAN_SITE}` | Atlassian 사이트 URL (예: your-company.atlassian.net) |

---

## 관련 프로젝트 경로

분석 시 참조하는 코드베이스 경로는 `.claude.local.md`에 정의됩니다.
워크플로우 실행 시 `.claude.local.md`의 "프로젝트 경로" 테이블에서 백엔드/프론트엔드 경로를 읽어 사용할 것.

`.claude.local.md`가 없거나 경로가 정의되지 않은 경우, AskUserQuestion으로 사용자에게 경로를 질문할 것.

---

## 템플릿

### Jira 티켓 템플릿

**제목 형식:**
- 버그: `[긴급] {제목}` 또는 `{제목}`
- 기능: `[기능] {제목}`
- 개선: `[개선] {제목}`

**설명 형식 (워크플로우 1 - 티켓 생성 시):**
```
## 요청사항

| 항목 | 내용 |
|------|------|
| 유형 | {유형} |
| 제목 | {제목} |
| 우선순위 | {우선순위} |
| 관련 화면 | {화면} |
| 요청자 | @{요청자} |
| 담당자 | @{담당자} |

## 상세 설명

### 이슈 내용 / 요청 내용
{내용}

### 발생 현황 / 요청 배경
{내용}

### AS-IS
{내용}

### TO-BE
{내용}

### 기대 효과
{내용}
```

> 상세 설명의 하위 섹션은 원본 Form에 내용이 있는 항목만 포함할 것. 비어있는 섹션은 생략.

**워크플로우 2 실행 후 추가되는 내용:**
```
## AI 분석 요약

{1-2문장 핵심 요약}

분석 문서: [🤖 {Confluence 문서 제목}|{Confluence URL}]
```
> Jira 설명 내 링크: `[표시텍스트|URL]` Jira 위키 마크업 형식 (plain URL 금지)
> Slack 메시지 내 링크: `<URL|표시텍스트>` 형식 (plain URL 금지)

### Confluence 분석 문서 템플릿

**제목:** `🤖 [{유형 prefix}] {요청 제목} 분석`

유형별 prefix 및 하위 폴더 매핑:

| 티켓 유형 | 제목 prefix | 저장 폴더 |
|----------|------------|---------|
| 버그 수정 | `[버그]` | `🐛 버그 분석` |
| 기능 추가 | `[기능]` | `✨ 기능 분석` |
| 개선 사항 | `[개선]` | `💡 개선 분석` |
| 기타 | `[기타]` | `💡 개선 분석` |

**위치:** `.claude.local.md`에 설정된 `{SPACE_KEY}` > `{ANALYSIS_FOLDER}` > 위 매핑표의 하위 폴더

**내용 구조:**
```
🤖 AI 생성 문서 | 분석일: {날짜}

## 요청 정보
| 항목 | 내용 |
|------|------|
| 요청 유형 | {유형} |
| 요청자 | {이름} |
| 우선순위 | {우선순위} |
| 관련 화면 | {화면} |
| Jira 티켓 | {PROJECT_KEY}-{번호} |

## 요청 내용
> {원본 요청}

## 분석 결과

### 요약
{1-2문장 요약}

### 관련 파일
| 파일 경로 | 관련도 | 설명 |
|----------|--------|------|
| {경로} | 높음/중간/낮음 | {설명} |

### 영향 범위
| 항목 | 내용 |
|------|------|
| 예상 수정 파일 수 | {N}개 |
| 복잡도 | 단순/보통/복잡 |
| API 변경 | 있음/없음 |
| DB 마이그레이션 | 필요/불필요 |

## 제안 구현 방안

### 접근 방식
{설명}

### 단계별 작업
1. {단계 1}
2. {단계 2}

## 리스크 및 고려사항
- ⚠️ {리스크}

## 미해결 질문
- ❓ {질문}

---
🤖 이 문서는 Claude Code를 사용하여 생성되었습니다.
```

---

## 워크플로우 실행 규칙

아래 규칙 파일들은 워크플로우 실행 시 반드시 참조해야 합니다:

### Confluence 문서 작업 규칙
- 워크플로우 2에서 Confluence 분석 문서 생성/수정 시 적용
- 규칙 파일: `.claude/rules/confluence.md` 를 읽을 것

### 에이전트 모델 전략
- 워크플로우 2, 3에서 코드베이스 분석 시 서브 에이전트를 활용할 때 적용
- 규칙 파일: `.claude/rules/agent-strategy.md` 를 읽을 것

---

## SwiftBar 메뉴바 관리

봇 프로세스를 macOS 메뉴바에서 시작/중지/상태 확인할 수 있도록 SwiftBar 플러그인을 사용한다.

### 구조
- `bot/scripts/start-bot.sh` — PID 파일 기반 봇 시작
- `bot/scripts/stop-bot.sh` — 봇 중지 (PID 기반 + pkill fallback)
- `bot/scripts/restart-bot.sh` — 중지 → 재시작
- `swiftbar/*.5s.sh` — SwiftBar 플러그인 (5초 폴링)

### 새 봇 추가 시 SwiftBar 플러그인 작성 절차
1. `bot/scripts/`에 start/stop/restart 스크립트 작성 (PID 파일: `/tmp/{봇이름}.pid`, 로그: `/tmp/{봇이름}.log`)
2. `swiftbar/{봇이름}.5s.sh` 플러그인 스크립트 작성 (`workflow-bot.5s.sh`를 템플릿으로 참고)
3. `chmod +x` 실행 권한 부여
4. `ln -s $(pwd)/swiftbar/{봇이름}.5s.sh ~/SwiftBar/{봇이름}.5s.sh` 심볼릭 링크 생성

### 설치 (최초 1회)
```bash
brew install --cask swiftbar
mkdir -p ~/SwiftBar
# SwiftBar 실행 후 플러그인 폴더로 ~/SwiftBar 선택
open -a SwiftBar
# 각 플러그인 심볼릭 링크
ln -s $(pwd)/swiftbar/workflow-bot.5s.sh ~/SwiftBar/workflow-bot.5s.sh
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/djheksa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/djheksa)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
