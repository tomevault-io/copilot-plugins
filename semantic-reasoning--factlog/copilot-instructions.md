## factlog

> 커밋 메시지, PR 제목, 브랜치 이름, 릴리스 노트처럼 저장소 이력과 공개 협업 기록에

# AGENTS.md

## 협업 표기 원칙

커밋 메시지, PR 제목, 브랜치 이름, 릴리스 노트처럼 저장소 이력과 공개 협업 기록에
남는 메타데이터에는 어떤 에이전트와 협업했는지 등록하거나 표기하지 않는다.

코드 변경에 대한 책임은 커밋 작성자, PR 작성자, 리뷰어, 머지 권한자에게 있다. 어떤
도구나 에이전트를 사용했는지는 책임 주체를 대체하지 않으며, 권한 있는 사람이 보낸
PR과 커밋 사용자 정보만으로 변경의 소유권과 책임을 추적하기에 충분하다.

따라서 다음과 같은 표기는 사용하지 않는다.

- 커밋 메시지의 `Generated with ...`, `Co-authored-by: ... agent`
- PR 제목이나 본문에 특정 에이전트명으로 작성 사실을 강조하는 문구
- 브랜치 이름, 태그, 변경 로그에 에이전트명을 포함하는 관례

필요한 경우에는 구현 의도, 검증 방법, 남은 위험처럼 코드 리뷰에 직접 도움이 되는
정보를 기록한다.

---
> Source: [semantic-reasoning/factlog](https://github.com/semantic-reasoning/factlog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
