## pindoc

> - Load PINDOC.md first.

@PINDOC.md

## Pindoc Session Start

- Load PINDOC.md first.
- At session start, run pindoc.workspace.detect, then run
  pindoc.task.queue with across_projects=true and compact=true before
  implementation. Review projects[slug].items for assigned Tasks across
  visible projects, then pin the concrete project_slug for follow-up tools.
- If a scoped task.queue call returns MULTI_PROJECT_WORKSPACE, rerun the
  sweep with across_projects=true or retry with an explicit project_slug.
- Before replacing PINDOC.md or this agent settings file after a harness
  change, call pindoc.harness.install with current_pindoc_md and
  current_agent_settings_body, then apply drifted_sections and
  suggested_write_targets yourself. Pindoc MCP never writes local files.

<!-- pindoc:register-separation:v1 BEGIN -->
### Register 분리 - 사용자 대화와 Pindoc artifact

Pindoc artifact 본문(Context / Decision / Rationale / Alternatives / Consequences 같은 섹션)은 구조화된 register에 속해서 표, bullet, ADR 스타일 축약이 자연스럽다. 반면 사용자 대면 응답은 추론 흐름이 연결된 산문 register에 속한다. "Alt A(...) · B(...)" 같은 축약, 괄호 안 한 단어 기각 이유, 중점으로 나열한 짧은 구절이 artifact 본문에서 대화로 역류하지 않게 한다. 애매할 때는 응답을 두세 문장 산문으로 다시 쓰며 추론을 압축이 아닌 서술로 노출한다.
<!-- pindoc:register-separation:v1 END -->

---
> Source: [var-gg/pindoc](https://github.com/var-gg/pindoc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
