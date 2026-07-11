## new-issue

> 요청을 scripts/new-issue.sh로 규칙에 맞춰 GitHub Issue 등록


# 이슈 생성 규칙

사용자가 "이슈 만들어줘" 류의 요청을 하면:

- 제목·ID·본문·라벨 규칙은 `scripts/new-issue.sh`·`.github/ISSUE_TEMPLATE/`가 강제한다(제목 `[ID] 명사형`). 팀원 안내는 `docs/issue-open-guide.md`.
- **직접 `gh issue create`를 손으로 만들지 말고 `scripts/new-issue.sh`를 사용**한다.
- 요청에서 유형(feat/setup/prototype/infra/deploy/fix/chore)·ID·제목(명사형)·순위를 뽑아 인자로 넘긴다.
  - 카탈로그(`docs/proposal/14`, `docs/issues-list.md`)에 있으면 그 내용을 재사용한다.
- **본문**은 `.github/ISSUE_TEMPLATE/` 템플릿을 그대로 사용한다(feat·prototype·fix·chore). `setup`·`infra`·`deploy`만 `--page/--refs/--done/--setup/--verify`로 채운다.
- 먼저 `--dry-run`으로 조립 결과를 보여주고, 확인되면 실제 등록한다.
- 등록 후 이슈 번호·제목·라벨을 보고한다.
- gh CLI 인증(`gh auth login`)이 선행 필요.

예: `scripts/new-issue.sh --type feat --id F-06 --priority 1 --title "산업 파급 경로 연결지도 시각화" --dry-run`

---
> Source: [siuuhaeju-com/suhyeju-com](https://github.com/siuuhaeju-com/suhyeju-com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
