## job-assistant

> - 기능 단위로 구현 후 자동으로 커밋할 것

# Job Assistant - Claude Code 작업 지침

## 작업 방식

- 기능 단위로 구현 후 자동으로 커밋할 것
- 커밋 메시지는 conventional commits 형식 사용 (feat:, fix:, chore: 등)
- 구현 전 계획을 먼저 설명하고 승인 후 진행할 것
- 임의로 구조 변경 시 반드시 먼저 물어볼 것
- 설계 문서(docs/design.md) 기준으로 구현할 것

## 코드 스타일

- TypeScript strict 모드
- 변수명, 함수명은 영어로 작성
- 주석은 한글로 작성
- 함수/클래스마다 JSDoc 형식으로 한글 주석 작성
- 복잡한 로직에는 인라인 한글 주석 추가

## AI 클라이언트 주석 규칙

AI API를 호출하는 모든 함수에는 아래 형식의 주석을 반드시 포함할 것.
현재는 Gemini API를 사용하지만, 추후 다른 모델로 교체할 수 있도록
교체 시 변경이 필요한 지점을 명확히 표시한다.

```typescript
/**
 * 채용 공고와 내 프로필의 매칭 점수를 계산합니다.
 *
 * @ai-provider Gemini API (gemini-2.0-flash)
 * @ai-change-point 다른 AI로 교체 시: packages/shared/gemini.ts의 클라이언트와
 *   이 함수의 모델명, 응답 파싱 로직을 함께 수정할 것
 */
async function calculateMatchScore(jd: string): Promise<MatchResult> { ... }
```

## 시작 프롬프트 (첫 작업 시 Claude Code에 입력)

```
docs/design.md를 읽고 개발 우선순위 순서대로 구현해줘.
각 기능 구현 후 커밋하고, 다음 단계 진행 전에 나에게 확인을 받아줘.
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hungerbk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
