## saveonedropone

> Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

## 5. Design System (UI 작업 필수 참조)

이 프로젝트는 `docs/design/`에 완성된 디자인 시스템이 있습니다. **UI가 포함된 모든 작업에서 반드시 먼저 참조하세요.**

| 파일 | 용도 |
|------|------|
| `docs/design/colors_and_type.css` | 색상·타이포그래피 토큰 (단일 진실 출처) |
| `docs/design/ui_kits/streamer-native/` | 구현 가능한 React 컴포넌트 (HomeScreen, MatchupScreen, ResultScreen, Sidebar, TopNav, BracketCard, ChatPanel) |
| `docs/design/README.md` | 전체 디자인 시스템 설명 (색상값, 버튼 스펙, 레이아웃 치수 등) |

**단계별 적용 규칙:**
- **아키텍처 설계 시**: UI 레이어는 `docs/design/README.md`의 Visual Foundations 섹션을 기준으로 정의
- **스토리 구현 시**: 해당 화면의 컴포넌트 파일(`ui_kits/streamer-native/*.jsx`)을 읽고 pixel-perfect 구현
- **코드 리뷰 시**: `colors_and_type.css` 토큰을 직접 쓰고 있는지, 하드코딩된 값이 없는지 확인

**픽셀 퍼펙트 기준:** 디자인 시스템의 토큰값(색상, 간격, 반경)을 임의로 바꾸지 말 것. 변경이 필요하면 먼저 사용자에게 확인.

---
> Source: [gyeongmin-lee/saveonedropone](https://github.com/gyeongmin-lee/saveonedropone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
