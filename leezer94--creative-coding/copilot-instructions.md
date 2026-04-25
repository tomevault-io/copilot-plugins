## creative-coding

> 새로 추가·리네임하는 파일과 폴더 이름은 케밥케이스(kebab-case)로 통일한다.


# 파일·폴더 이름: 케밥케이스

- **규칙**: 이 레포에서 **새로 만들거나 이름을 바꿀 때** 파일·디렉터리 베이스 이름은 **케밥케이스**만 사용한다 (`word-word.ext`). 카멜케이스·파스칼케이스·스네이크케이스 파일명은 쓰지 않는다.
- **적용 범위**: 소스, 스크립트, 스타일, 문서(`*.md` 본문 파일명), 앱/패키지 내부 자산 등 **프로젝트가 짓는 이름** 전부.

## 예외(도구·관례상 고정)

다음은 생태계 관례에 맞춰 **케밥케이스로 바꾸지 않는다**.

- `package.json`, `pnpm-lock.yaml`, `turbo.json` 등 패키지 매니저/빌드 도구가 기대하는 이름
- `README.md`, `LICENSE`, `CHANGELOG.md` 등 관용 대문자 문서명
- `index.html`, `index.ts` 등 번들/엔트리 관례의 `index.*`
- 숨김·도구 설정: `.gitignore`, `.env.example`, `eslint.config.js` 등 (이미 존재하는 패턴 유지)
- Monorepo 루트·앱 폴더명은 기존 구조 유지; **그 안에서 새 파일**은 케밥케이스

## 예시

| ❌ 피하기          | ✅ 사용              |
|--------------------|----------------------|
| `useGuestWebRtc.ts` | `use-guest-webrtc.ts` |
| `HostPage.tsx`      | `host-page.tsx`       |
| `face_stats.ts`     | `face-stats.ts`       |
| `MirrorRoom/` (새 폴더) | `mirror-room/`   |

- **코드 내부** 식별자(변수, 함수, 컴포넌트, 타입 이름)는 해당 언어/프레임워크 관례(예: TS/React의 PascalCase 컴포넌트)를 따른다. 이 규칙은 **파일·폴더 경로명**만 다룬다.
- 기존 카멜/파스칼 이름 파일은 **다음에 손댈 때** 또는 별도 리팩터에서 케밥케이스로 맞춘다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leezer94) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
