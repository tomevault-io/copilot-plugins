## claude-code-chat

> **목표:** AI 교육에 대한 5명 전문가의 독립 의견(claude -p)을 생성하고, 팀 토론을 거쳐 전략 보고서를 작성한다.

# claude-code-chat

## 하네스: AI 교육 다관점 토론

**목표:** AI 교육에 대한 5명 전문가의 독립 의견(claude -p)을 생성하고, 팀 토론을 거쳐 전략 보고서를 작성한다.

**트리거:** "AI 교육 토론/보고서/전략", "전문가 토론", "다관점 의견 수렴" 관련 요청 시 `ai-edu-debate` 스킬을 사용하라. 단순 질문은 직접 응답 가능.

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-04-09 | 초기 구성 (전문가 5 + 진행자 + 전략분석가 + ai-edu-debate 스킬) | 전체 | 하네스 최초 구축 |
| 2026-04-09 | 실시간 라이브 뷰어 추가 (SSE 서버 + index.html + emit.sh) | skills/ai-edu-debate, agents/debate-moderator, strategy-analyst | "토론을 실시간으로 보고 싶다" 요청 반영 |

---
> Source: [revfactory/claude-code-chat](https://github.com/revfactory/claude-code-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
