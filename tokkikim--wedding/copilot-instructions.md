## wedding

> - 모든 개발 내용은 문서화를 해야 함


- 모든 개발 내용은 문서화를 해야 함

1. 폴더 구조 규칙

App Router(app/) 기반을 기본으로 사용한다. (pages/는 신규 프로젝트에서 사용 금지)

라우팅 = 폴더 구조 = URL (폴더명 = 경로명)

소문자-케밥케이스만 사용 (예: /user-profile, not /UserProfile).

공용 컴포넌트는 app/\_components/ 또는 src/components/에 분리.

서버 전용 로직(db.ts, auth.ts)은 절대 client component에 import하지 않는다.

2. 서버/클라이언트 컴포넌트 구분

기본은 서버 컴포넌트 (SSR, 데이터 접근 안전).

클라이언트 state/useEffect/useState/useRef가 필요한 경우만

"use client";

선언 후 사용.

서버 컴포넌트에서만 DB·API 호출, env 변수를 직접 사용한다.

3. 데이터 패칭 규칙

서버 컴포넌트 → 직접 fetch (async 함수 사용).

동일 데이터 반복 호출 방지를 위해 React cache 또는 SWR 활용.

CSR fetch 금지 (필요하면 React Query/SWR로 명확히 분리).

API 라우트(app/api/\*)는 외부에 노출될 때만 사용. 내부에서 쓸 데이터는 서버 컴포넌트에서 직접 접근.

4. 환경 변수 규칙

.env는 절대 Git에 포함 금지.

서버 전용 = NEXT*PUBLIC* 없이 사용.

클라이언트 전용 = NEXT*PUBLIC* prefix 반드시 붙여야 함.

잘못된 prefix 사용은 보안 사고로 직결되므로 코드 리뷰 필수 항목.

5. 스타일링 규칙

TailwindCSS를 기본으로 사용.

전역 스타일은 app/globals.css 하나에만.

컴포넌트 단위 스타일은 className으로만 적용.

CSS Module은 레거시 코드 유지용으로만 허용.

6. 상태 관리 규칙

서버 우선, 클라이언트 최소화 원칙.

전역 상태는 React Context → 불필요한 경우 금지.

API 캐싱은 React Query or RSC fetch 캐싱을 활용.

Redux, MobX 등 무거운 상태 관리 툴은 불가피한 경우만.

7. 성능/최적화 규칙

Image 컴포넌트 사용은 필수 (img 금지).

Link 컴포넌트 사용은 필수 (a 태그 직접 사용 금지, 필요 시 passHref).

dynamic import()로 큰 컴포넌트 분리 로딩.

모든 페이지는 Lighthouse 90점 이상 목표.

외부 JS 라이브러리는 지연 로딩(동적 import, "use client" 최소화).

8. 테스트/품질 규칙

ESLint + Prettier는 CI에서 강제.

TypeScript strict 모드 필수.

API route, DB access에는 Jest/e2e 테스트 케이스 반드시 포함.

PR 머지 전 빌드 통과 + 테스트 통과 확인 필수.

9. 보안 규칙

API route에서만 cookies, headers 직접 접근.

XSS 방지 위해 dangerouslySetInnerHTML 사용 금지 (필요 시 sanitizer 필수).

인증 로직은 서버 컴포넌트에서만 처리.

민감한 데이터는 절대 클라이언트 컴포넌트로 내려보내지 않는다.

10. 배포 규칙

Vercel/Node 서버 환경은 동일하게 맞춘다.

next build 실패 시 절대 강제 배포 금지.

Static Export(next export) 금지 (App Router와 충돌).

CDN 캐싱, ISR(Incremental Static Regeneration) 전략을 반드시 설계 단계에서 정의.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tokkikim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tokkikim)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
