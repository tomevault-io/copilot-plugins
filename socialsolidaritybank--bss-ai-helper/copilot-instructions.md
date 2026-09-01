## bss-ai-helper

> 이 저장소에서 사용자가 `BSS AI Helper 실행해줘`, `AI 세팅 이어서 해줘`, `개발환경 설치 도와줘`라고 말하면 먼저 현재 상태를 확인한다.

# BSS AI Helper Agent Notes

이 저장소에서 사용자가 `BSS AI Helper 실행해줘`, `AI 세팅 이어서 해줘`, `개발환경 설치 도와줘`라고 말하면 먼저 현재 상태를 확인한다.

기본 흐름은 `git clone https://github.com/socialsolidaritybank/bss-ai-helper.git ~/bss-ai-helper`, `cd ~/bss-ai-helper`, `codex`입니다. Codex가 열리면 사용자는 `BSS AI Helper 실행해줘`라고 말합니다.

작업을 이어갈 때는 `./install.sh --status`로 상태를 먼저 보여준다. Windows에서는 `.\windows\install.ps1 -Status`를 쓴다.

비밀번호, 토큰, OAuth 코드는 저장하지 않는다. `~/.codex/skills`, `~/.claude/skills`, `~/.agents/skills`에는 직접 쓰지 않는다. 스킬 설치는 `skill-add <dir> --mine`가 있을 때만 SSOT 경로로 처리한다.

GitHub 게시 준비는 읽기 전용으로만 확인한다. 커밋, 푸시, 레포 생성, 원격 변경은 사용자가 명시적으로 요청한 뒤에만 한다.

---
> Source: [SocialSolidarityBank/bss-ai-helper](https://github.com/SocialSolidarityBank/bss-ai-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
