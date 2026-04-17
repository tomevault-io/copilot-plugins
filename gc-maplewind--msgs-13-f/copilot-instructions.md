## msgs-13-f

> - 모든 작업은 하나의 명확한 Prompt로 시작하고 완료합니다

# AI 코딩 컨벤션 - 단풍바람 프로젝트

## 🎯 기본 원칙

### 1. Prompt 단위 작업
- 모든 작업은 하나의 명확한 Prompt로 시작하고 완료합니다
- 한 Prompt는 하나의 기능 또는 수정사항만 다룹니다
- 작업 완료 = 개발 + 브라우저 테스트 + Git commit
- **중요: 코드 작성 후 반드시 브라우저로 자동 테스트합니다**

### 2. PR 기반 협업
- 모든 변경사항은 PR(Pull Request)을 통해 리뷰받습니다
- main 브랜치에 직접 push 금지
- PR은 Prompt 단위 또는 Feature 단위로 생성합니다

---

## 📝 Commit Message 규칙

다음 형식을 엄격히 준수:

```
<type>: <subject>

[optional body]
[optional footer]
```

### Type 종류
- `Add`: 새로운 기능 추가
- `Update`: 기존 기능 개선/수정
- `Fix`: 버그 수정
- `Refactor`: 코드 리팩토링 (기능 변경 없음)
- `Style`: 코드 포맷팅, 세미콜론 누락 등
- `Docs`: 문서 수정
- `Test`: 테스트 코드 추가/수정
- `Chore`: 빌드 설정, 패키지 매니저 등
- `Remove`: 파일/코드 삭제

### 예시
```
Add: Character profile image upload feature

- Upload component implementation
- Image optimization with sharp
- S3 storage integration

Closes #42
```

---

## 🌿 Branch Naming 규칙

```
<type>/<issue-number>-<short-description>
```

### 예시
- `feature/13-character-search`
- `fix/42-image-save-bug`
- `refactor/99-api-service-layer`
- `docs/25-api-documentation`

---

## 🌐 브라우저 기반 테스트 (필수)

### 자동 테스트 원칙
모든 코드 변경 후 브라우저로 자동 테스트를 실행합니다.

### MCP Browser 사용
- `browser_navigate`: 페이지 이동
- `browser_snapshot`: 화면 캡처 및 DOM 확인
- `browser_click`: 요소 클릭
- `browser_type`: 텍스트 입력
- `browser_evaluate`: JavaScript 실행

### 테스트 시나리오 예시
```
새 기능 개발 시:
1. 코드 작성
2. 브라우저로 localhost:5173 접속
3. 기능 동작 확인
4. 모바일 크기(375px) 확인
5. 콘솔 에러 확인
6. 스크린샷 첨부
7. 문제 있으면 자동 수정 후 재테스트
```

### 자동 테스트가 필요한 경우
- ✅ 새 컴포넌트 추가
- ✅ UI/UX 변경
- ✅ 사용자 인터랙션 구현
- ✅ 반응형 레이아웃 수정
- ✅ 버그 수정

### 테스트 보고 형식
```markdown
## 테스트 결과
- [x] 기능 동작 확인
- [x] 모바일 반응형 확인
- [x] 콘솔 에러 없음
- [x] 스크린샷: [첨부]

문제: 없음
```

**상세 가이드:** BROWSER_TESTING.md 참조

---

## 🎨 코드 작성 규칙

### TypeScript
- 타입은 명시적으로 작성 (`any` 사용 금지)
- Interface는 `I` 접두사 없이 작성 (예: `Character`, not `ICharacter`)
- Enum은 PascalCase (예: `UserRole`)

### Svelte
- 컴포넌트는 PascalCase (예: `CharacterCard.svelte`)
- Props는 명시적 타입 지정
- Svelte 5 문법 사용 ($state, $derived 등)

### Styling
- Tailwind CSS 유틸리티 클래스 우선 사용
- 커스텀 CSS는 최소화
- 색상, 간격 등은 Tailwind 디자인 토큰 사용

---

## 📂 파일 구조 규칙

### 컴포넌트
```
src/lib/components/
├── [ComponentName].svelte       # 컴포넌트
├── [ComponentName].test.ts      # 테스트 (선택)
└── index.ts                     # Export 모음
```

### 페이지
```
src/routes/
├── [route]/
│   ├── +page.svelte            # 페이지 컴포넌트
│   ├── +page.ts                # 데이터 로딩 (선택)
│   └── +page.server.ts         # 서버 데이터 (선택)
```

---

## 🤖 AI Prompt 작성 가이드

### 좋은 Prompt 예시
```
"캐릭터 검색 기능을 추가해줘:
- 이름, 닉네임으로 검색 가능
- 실시간 필터링
- 대소문자 구분 없음
- 검색어 하이라이트
- lib/components/CharacterSearch.svelte로 생성"
```

### 나쁜 Prompt 예시
```
"검색 만들어줘"  ❌ (너무 모호함)
"캐릭터도 검색하고 메생결산도 검색하고 댓글도 검색..."  ❌ (한 번에 너무 많음)
```

### Prompt 체크리스트
- [ ] 무엇을 만들지 명확한가?
- [ ] 어디에 만들지 명시했는가?
- [ ] 필요한 기능을 구체적으로 나열했는가?
- [ ] 한 Prompt에 하나의 기능만 포함되어 있는가?

---

## 🔍 Code Review 체크리스트

### 기능
- [ ] Prompt의 요구사항을 모두 충족하는가?
- [ ] Edge case를 고려했는가?
- [ ] 에러 처리가 적절한가?

### 코드 품질
- [ ] 타입이 명시적으로 선언되어 있는가?
- [ ] 중복 코드가 없는가?
- [ ] 네이밍이 명확한가?
- [ ] 주석이 필요한 곳에 있는가?

### 성능
- [ ] 불필요한 리렌더링이 없는가?
- [ ] 큰 파일/이미지는 최적화되어 있는가?

### 테스트
- [ ] 로컬에서 테스트했는가?
- [ ] 모바일에서 확인했는가?
- [ ] 다양한 데이터로 테스트했는가?

---

## 📦 PR 작성 규칙

### PR 제목
```
[Type] Short description (#issue-number)
```

예시:
```
[Feature] Add character search functionality (#13)
```

### PR Description 템플릿
```markdown
## 📋 작업 내용
무엇을 했는지 간단히 설명

## 🎯 관련 이슈
Closes #13

## 🤖 사용한 Prompt
- "캐릭터 검색 기능 추가해줘..."
- "검색어 하이라이트 기능 추가해줘..."

## ✅ 체크리스트
- [ ] 로컬에서 테스트 완료
- [ ] 타입 에러 없음
- [ ] Linter 통과
- [ ] 모바일 반응형 확인

## 📸 스크린샷 (선택)
(있다면 추가)

## 💬 특이사항
(있다면 추가)
```

---

## 🚀 배포 규칙

### main 브랜치
- 항상 배포 가능한 상태 유지
- PR 승인 후에만 merge
- Squash merge 사용 (커밋 히스토리 정리)

### 버전 태깅
- Semantic Versioning 사용: `v1.2.3`
- `v[major].[minor].[patch]`
- 태그 생성 시 자동 배포

---

## 📚 문서화 규칙

### 코드 주석
```typescript
/**
 * 캐릭터를 ID로 조회합니다
 * 
 * @param id - 캐릭터 고유 ID
 * @returns 캐릭터 객체 또는 undefined
 * 
 * @example
 * const char = getCharacterById('char-1');
 */
export function getCharacterById(id: string): Character | undefined {
  return characters.find((c) => c.id === id);
}
```

### 컴포넌트 주석
```svelte
<!--
  CharacterCard 컴포넌트
  
  @component
  캐릭터 정보를 카드 형태로 표시합니다
  
  @props
  - character: Character - 표시할 캐릭터 객체
  - onClick: () => void - 클릭 이벤트 핸들러 (선택)
  
  @example
  <CharacterCard {character} onClick={() => console.log('clicked')} />
-->
```

---

## 🛠️ 개발 도구 설정

### VS Code / Cursor 확장
- Svelte for VS Code
- Tailwind CSS IntelliSense
- ESLint
- Prettier
- GitLens

### 추천 설정 (.vscode/settings.json)
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "tailwindCSS.experimental.classRegex": [
    ["class:\\s*['\"]([^'\"]*)['\"]"]
  ]
}
```

---

## 🔐 보안 규칙

- `.env` 파일은 절대 커밋하지 않음
- API 키, 비밀번호 등 민감 정보는 환경 변수 사용
- 사용자 입력은 항상 검증/sanitize
- XSS 공격 방지를 위해 `{@html}`은 신중하게 사용

---

## ⚡ 성능 최적화 가이드

### 이미지
- WebP 포맷 사용
- Lazy loading 적용
- 적절한 크기로 리사이징

### 번들 크기
- 사용하지 않는 의존성 제거
- Dynamic import 활용
- Tree shaking 확인

### 렌더링
- `$derived`로 계산된 값 캐싱
- 불필요한 반응성 제거
- Virtual scroll for 긴 리스트

---

## 🧪 테스트 규칙

### 단위 테스트
```typescript
// CharacterCard.test.ts
import { render } from '@testing-library/svelte';
import CharacterCard from './CharacterCard.svelte';

test('renders character name', () => {
  const { getByText } = render(CharacterCard, {
    props: { character: mockCharacter }
  });
  expect(getByText('담뫄')).toBeInTheDocument();
});
```

### E2E 테스트 (선택)
- Playwright 사용
- 주요 사용자 플로우 테스트
- CI/CD에 통합

---

## 📞 도움 요청하기

### AI에게 질문할 때
```
좋은 예:
"CharacterCard 컴포넌트에서 호버 시 그림자 효과를 추가하려고 하는데,
Tailwind CSS로 어떻게 구현할 수 있어?"

나쁜 예:
"안 돼요" ❌
"에러 나요" ❌
```

### 에러 보고 시 포함할 내용
1. 무엇을 하려고 했는지
2. 전체 에러 메시지
3. 관련 코드 조각
4. 이미 시도해본 해결책

---

## 🎓 학습 자료

### 읽어야 할 문서
- [SvelteKit 공식 문서](https://kit.svelte.dev/docs)
- [Svelte 5 튜토리얼](https://learn.svelte.dev/)
- [Tailwind CSS 문서](https://tailwindcss.com/docs)
- [TypeScript 핸드북](https://www.typescriptlang.org/docs/handbook/intro.html)

### 추천 블로그/영상
- [Josh Collinsworth - Svelte 시리즈](https://joshcollinsworth.com/)
- [Huntabyte - YouTube](https://www.youtube.com/@Huntabyte)

---

## 📅 정기 작업

### 매주
- [ ] 의존성 업데이트 확인
- [ ] 미해결 이슈 정리
- [ ] PR 리뷰

### 매월
- [ ] 성능 모니터링
- [ ] 번들 크기 체크
- [ ] 보안 취약점 점검

---

**이 규칙은 AI가 자동으로 읽고 따릅니다.**
**팀원도 반드시 숙지하고 준수해야 합니다.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GC-MapleWind) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
