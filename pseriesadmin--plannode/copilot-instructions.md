## plannode-web

> Vanilla 파일럿(index.html·plannode.js) 및 Svelte/Vite 편집 시 스택 규칙


# 프론트엔드 스택 (Plannode)

## 공통

- 기존 클래스명·DOM ID·인라인 스타일 패턴을 따른다. 새 의존성(npm 패키지 번들) 추가는 사용자와 합의 후에만 한다.
- **구조·UI 토큰 상세**: `plannode-architecture.mdc`, `plannode-ui-identity.mdc`.

## Vanilla 파일럿 (`index.html`, `plannode.js`)

- CDN 스크립트(Supabase 등)는 `index.html`에 명시적으로 추가하고, 순수 정적 호환성(`file://` 제외 시 로컬 서버)을 깨지 않도록 한다.
- 브라우저 전역과 이벤트 리스너 정리: 새 리스너 추가 시 중복 등록·메모리 누수를 피한다.

## SvelteKit + Vite (`src/**`)

- `$lib` 별칭·`$app/environment`의 `browser` 가드로 SSR/클라이언트 분기 — `window`/`localStorage`/Supabase 세션은 브라우저에서만.
- 환경 변수는 `VITE_*`만 클라이언트에 노출; 값 미설정 시에도 모듈이 로드되도록 한 뒤, 호출부에서 `isSupabaseCloudConfigured()` 등으로 차단하는 패턴을 유지한다 (`client.ts` 참고).
- 파일럿과 공유하는 **고정 DOM id**는 `plannode-architecture.mdc`의 와이어 싱크 절을 따른다.

---
> Source: [pseriesadmin/plannode](https://github.com/pseriesadmin/plannode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
