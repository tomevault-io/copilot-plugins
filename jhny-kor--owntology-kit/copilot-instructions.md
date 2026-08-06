## owntology-kit

> 이 파일은 **자동화 에이전트**(Codex, Claude Code 등)가 OWNtology-Kit을 사용자 대신

# AGENTS.md — Codex·Claude Code 등 에이전트 실행 규칙

이 파일은 **자동화 에이전트**(Codex, Claude Code 등)가 OWNtology-Kit을 사용자 대신
설치·구동할 때 따르는 지침입니다. 핵심 원칙: **코드/CLI는 에이전트가 처리하되, macOS
권한·앱 로그인·개인 입력값은 사람만 할 수 있으므로 그 지점에서 반드시 멈추고 요청한다.**

## 대원칙

1. **doctor 먼저, 항상.** 어떤 수집·실행 전에도 `python3 kit.py doctor` 를 돌려 무엇이
   빠졌는지 확인하고, `[FAIL]`/`[WARN]` 을 사람에게 그대로 보고한다.
2. **권한·로그인·개인정보는 대신 하지 않는다.** 아래 "사람에게 넘길 것"에 해당하면
   진행을 멈추고 사람에게 명확한 안내와 함께 요청한다. 추측으로 값을 채우지 않는다.
3. **기본은 안전.** 수집 소스는 기본 전부 꺼져 있다. 사람이 켜라고 지정한 소스만 켠다.
4. **파괴적 동작 확인.** `kit.py purge` 는 데이터를 지운다 — 항상 dry-run(기본)을 먼저
   보여주고, 사람이 승인하면 `--apply`.
5. **개인정보 유출 금지.** 볼트에는 평문 개인정보가 있다. 볼트 내용을 커밋·업로드·외부
   전송하지 않는다. `config.json`·볼트 경로는 절대 공개 저장소에 넣지 않는다.

## 표준 순서

```bash
# 1. 준비
git clone https://github.com/jhny-kor/OWNtology-Kit.git && cd OWNtology-Kit
python3 kit.py init                 # config.json + 볼트 스캐폴드 (개인정보 경고 출력)

# 2. 설정 — 사람 입력 필요 (아래 "사람에게 넘길 것" 참고)
#    카카오 닉네임/켤 소스 등을 사람에게 물어 config.json 반영하거나 kit.py web 안내

# 3. 사전 점검
python3 kit.py doctor               # FAIL 이 있으면 여기서 멈추고 사람에게 해결 요청

# 4. 수집·온톨로지화 (doctor 통과분만)
python3 kit.py run                  # 실패/건너뜀 요약을 사람에게 보고

# 5. (선택) MCP 서버
python3 -m pip install -r requirements-mcp.txt
python3 kit.py mcp                  # 로컬 stdio — MCP 클라이언트 설정에 등록 (README 참고)
```

## 에이전트가 해도 되는 것 ✅

- 레포 클론, `kit.py init/doctor/run/ontologize/mcp` 실행
- 서드파티 CLI 설치 시도: `katok`([NomaDamas/katok](https://github.com/NomaDamas/katok)),
  `imsg`([openclaw/imsg](https://github.com/openclaw/imsg)) — 설치 명령은 실행하되, 실패하면 사람에게 보고
- `pip install -r requirements-mcp.txt`
- 사람이 알려준 값으로 `config.json` 편집(닉네임·켤 소스·볼트 경로·GitHub 아이디)
- MCP를 Claude Code/Codex/Desktop 설정 파일에 등록
- `kit.py doctor` 로 상태 점검·보고

## 사람에게 넘길 것 (여기서 멈추고 요청) ⚠️

에이전트는 이것들을 대신 수행할 수 없다. 발견 즉시 멈추고, 무엇을·왜·어떻게 해야 하는지
안내하며 사람에게 넘긴다.

| 항목 | 왜 사람만 가능한가 | 사람이 할 일 |
|------|-------------------|-------------|
| **Full Disk Access** | 시스템 설정의 GUI 토글, 부여 후 앱 재시작 필요 | 시스템 설정 > 개인정보 보호 및 보안 > 전체 디스크 접근 권한 → 에이전트 실행 앱(Terminal/iTerm/Claude Code 등) 추가 후 재시작 |
| **Mail·메모 자동화 팝업** | 첫 실행 시 뜨는 macOS 허용 팝업 클릭 | 팝업에서 "허용" (또는 시스템 설정 > 자동화) |
| **카카오톡 로그인** | 카카오톡 앱 설치·본인 로그인 상태여야 katok이 읽음 | 카카오톡 데스크톱 설치·로그인 |
| **카카오 닉네임 등 개인값** | 본인만 아는 값(닉네임·볼트 경로·GitHub 아이디·켤 소스) | 값 제공, 또는 `kit.py web` 에서 직접 입력 |
| **클라우드 이관** | 서버·도메인·토큰 준비 | 서버 접속정보·도메인·`OWNTOLOGY_TOKEN` 제공 (deploy/README.md) |
| **purge 실삭제** | 되돌릴 수 없는 데이터 삭제 | dry-run 확인 후 `--apply` 승인 |

## 실패 시 행동

- 수집기 실패는 정상일 수 있다(권한/미설치) — `run` 은 **성공/실패/건너뜀 요약**을 내므로
  그대로 사람에게 전달한다. 핵심 수집기(kakao·sms·mail)가 전부 실패하면 종료코드가 0이 아니다.
- 원인 진단은 `kit.py doctor` 와 README 의 **문제 해결(Troubleshooting)** 표를 먼저 참조한다.
- 권한 문제로 막히면 임의로 우회하지 말고 위 표대로 사람에게 넘긴다.

---
> Source: [jhny-kor/OWNtology-Kit](https://github.com/jhny-kor/OWNtology-Kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
