## culture-walk

> Culture Walk(`문화산책`)은 한국 문화행사를 찾는 모바일 우선 지도 서비스입니다. Next.js 16, React 19, Tailwind CSS, OpenNext 기반 Cloudflare Workers, Cloudflare D1, Cloudflare KV를 사용합니다.

# 문화산책 에이전트 가이드

## 프로젝트

Culture Walk(`문화산책`)은 한국 문화행사를 찾는 모바일 우선 지도 서비스입니다. Next.js 16, React 19, Tailwind CSS, OpenNext 기반 Cloudflare Workers, Cloudflare D1, Cloudflare KV를 사용합니다.

## 저장소 구성

- `src/app`: App Router 페이지와 API 라우트
- `src/components`: 기능별 UI 컴포넌트(`Map`, `Header`, `BottomSheet`, 공용 `Common`)
- `src/services`: TourAPI 접근, 정규화, 스냅샷 동기화, 캐시, 도메인 로직
- `src/db/schema.ts`: Drizzle 스키마
- `db/migrations`: 추가 전용 D1 마이그레이션. 적용된 마이그레이션은 수정하지 않습니다.
- `tests`: Node 테스트 러너 테스트. 특히 데이터 정규화와 동기화 동작을 검증합니다.
- `worker.js`: Cloudflare Worker 스케줄 이벤트 진입점
- `wrangler.jsonc`: Worker 바인딩과 크론 일정

## 개발 명령어

```bash
npm test
npm run typecheck
npm run lint
npm run build
npm run db:migrate:local
npm run db:migrate:remote
npm run deploy
```

작은 변경에는 필요한 범위의 검증만 실행합니다. 배포 전 또는 공유 데이터, API, UI 동작을 변경한 뒤에는 `npm test`, `npm run typecheck`, `npm run lint`를 실행합니다.

## 구현 규칙

- TypeScript와 `@/*` import 별칭을 사용하고 엄격한 타입을 유지합니다.
- TourAPI와 D1 서버 전용 코드는 API 라우트, Worker 코드, 서비스에 둡니다. 클라이언트 컴포넌트에 비밀 정보를 노출하지 않습니다.
- 기능 중심 컴포넌트 구조를 유지합니다. 새 기본 요소를 만들기 전 `Button`, `IconButton`, `CultureCategoryBadge`, 공용 surface 클래스를 재사용합니다.
- 새 컨트롤에는 Lucide 아이콘을 사용합니다. 아이콘 전용 컨트롤에는 접근 가능한 이름 또는 툴팁이 필요합니다.
- 한국어 UI 문구는 짧고 행동 중심으로 작성합니다.
- `.next`, `.open-next` 같은 생성 디렉터리는 수정하지 않습니다.
- `.env*`, API 키, 동기화 토큰, 데이터베이스 내보내기, 생성된 Worker 산출물은 커밋하지 않습니다.

## 데이터 동기화 규칙

- 목록 스냅샷은 안정적인 `tourapi:{contentId}` source key, 스테이징 행, 소유 잠금을 사용합니다. 삭제 후 재삽입 방식으로 바꾸지 않습니다.
- 목록 동기화는 목록 필드만 소유합니다. TourAPI 상세 엔드포인트가 채운 상세 필드를 덮어쓰지 않습니다.
- 누락 횟수를 바꾸거나 비활성화하려면 성공적으로 검증된 스냅샷이 필요합니다. 첫 번째 누락은 행사를 활성 상태로 유지하고, 연속 두 번째 누락에서 비활성화합니다.
- 상세 조회는 stale-while-revalidate를 사용합니다. 먼저 D1/KV 데이터를 반환하고, 오래된 데이터는 새로고침 요청을 기록한 뒤 스케줄 Worker가 TourAPI를 호출합니다.
- 외부 API 실패는 파괴적이지 않아야 합니다. 기존 상세 데이터는 유지하고 현재의 재시도 및 백오프 필드를 사용합니다.
- 동기화 필드를 변경하는 마이그레이션은 `db/migrations`와 `src/db/schema.ts`를 함께 갱신하고, 테스트를 추가하거나 수정합니다.

## 배포 및 운영

- 변경한 코드가 의존하는 원격 D1 마이그레이션은 배포 전에 적용합니다.
- `SYNC_TOKEN`, `TOUR_API_KEY`는 Worker 비밀 값입니다. `wrangler secret put`으로 설정하고 `wrangler.jsonc`에 추가하지 않습니다.
- 배포 후 `GET /api/health`를 확인합니다. 활성 건수, 좌표·날짜 품질, 캐시 상태, 최근 스냅샷 결과를 제공합니다.
- 크론 일정은 `wrangler.jsonc`에 정의합니다. TourAPI 할당량, Worker 서브리퀘스트 제한, D1 쓰기량을 고려하지 않은 일정 변경은 하지 않습니다.
- Windows에서는 프로덕션 빌드 전에 `.open-next`를 점유하는 로컬 `wrangler dev` 프로세스를 종료합니다. 그렇지 않으면 OpenNext가 빌드 출력을 정리하지 못할 수 있습니다.

## 변경 체크리스트

1. 변경 범위를 좁게 유지하고 동작 변경에 맞는 테스트를 갱신합니다.
2. UI 변경은 모바일과 데스크톱 너비에서 라이트·다크 테마를 확인합니다.
3. 변경 범위에 맞는 검증 명령을 실행합니다.
4. 커밋 전에 `git diff`, `git status`를 확인합니다.
5. 배포 후 `/api/health`와 변경된 공개 경로를 확인합니다.

---
> Source: [rkdals0301/culture-walk](https://github.com/rkdals0301/culture-walk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
