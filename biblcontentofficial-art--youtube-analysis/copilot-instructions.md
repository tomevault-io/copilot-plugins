## youtube-analysis

> - **작업 중 절대 `git push` 하지 않는다.**

# Claude 작업 규칙

## 커밋 & 배포 정책 (Vercel 빌드 비용 절감)

### 핵심 규칙
- **작업 중 절대 `git push` 하지 않는다.**
- `git add` + `git commit`은 로컬에서 자유롭게 해도 된다. (push 없이)
- **push는 반드시 사용자가 "배포해줘", "push해줘", "올려줘" 라고 명시적으로 말할 때만 한다.**
- 여러 기능이 완성된 후 한 번에 push → Vercel 빌드 횟수 최소화

### 작업 흐름
1. 코드 수정 → 로컬 테스트
2. 기능 완성 → `git commit` (로컬만)
3. 사용자가 배포 요청 → 그때 `git push`

### 잘못된 예시 (하지 말 것)
- 파일 하나 수정할 때마다 push
- 텍스트 변경 직후 바로 push
- 작은 버그 수정마다 push

### 올바른 예시
- 여러 기능 작업 완료 → 커밋 묶어서 → 사용자 확인 후 push

---
> Source: [biblcontentofficial-art/youtube-analysis](https://github.com/biblcontentofficial-art/youtube-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
