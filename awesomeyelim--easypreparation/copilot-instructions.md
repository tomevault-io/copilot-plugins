## easypreparation

> 이 프로젝트는 Go 기반 보안 감사 에이전트입니다.

# GEMINI.md — gsac 프로젝트 가이드 (Gemini CLI)

이 프로젝트는 Go 기반 보안 감사 에이전트입니다.
**모든 도구는 `tools/run.sh` 를 통해 실행합니다.**

---

## 코드 분석

```bash
tools/run.sh analyze AD           # AD 모듈 전체 통계 + 함수목록
tools/run.sh analyze AD Ping      # Ping 함수 상세 (위치·호출·역방향)
```

자세한 모듈 목록과 플래그: `AGENTS.md` 참조

---

## 문서 생성 + Drive 업로드

```bash
# 1. 슬라이드 생성 스크립트 작성 후 실행
tools/.venv/Scripts/python.exe tools/output/_tmp.py   # Windows
tools/.venv/bin/python tools/output/_tmp.py           # Linux/macOS

# 2. Drive 업로드
tools/run.sh upload tools/output/result.pptx --folder <폴더ID> --key my_key

# 3. 임시 파일 삭제
rm tools/output/_tmp.py
```

슬라이드 프레임워크 API, 파일 경로 규칙 등: `AGENTS.md` 참조

---

## 토큰 관리

```bash
tools/run.sh token status   # 유효 여부 확인
tools/run.sh token refresh  # 강제 갱신
```

최초 인증: `python tools/auth.py`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AwesomeYelim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AwesomeYelim)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
