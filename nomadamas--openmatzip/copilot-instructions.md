## openmatzip

> 코딩 에이전트가 이 저장소에서 작업할 때 지켜야 할 규칙.

# AGENTS.md

코딩 에이전트가 이 저장소에서 작업할 때 지켜야 할 규칙.

## 이 저장소는 무엇인가

맛잘알이 보증한 대한민국 맛집 오픈 데이터베이스. `data/restaurants/<id>.yaml` 파일 하나가 맛집 하나다.

## 셋업

```bash
bun install
```

## 명령어

| 명령 | 용도 |
|---|---|
| `bun run validate` | 데이터 전체 검증. **PR 전 반드시 실행** |
| `bun run build` | `schema/restaurant.schema.json` 과 `dist/openmatzip.*` 생성 |
| `bun run query -- --city 서울 --cuisine korean` | 검색 |
| `bun run new -- --id <id> --name <이름>` | 새 데이터 파일 초안 |
| `bun run import -- --file <목록> --handle <id> --yes` | 카카오맵/네이버지도 저장 목록 가져오기 |
| `bun run dedupe -- [--fix]` | 중복 맛집 검사 / 자동 병합 |
| `bun run checkup -- [--days 180]` | 영업 여부 재확인이 필요한 맛집 목록 |
| `bun run checkup -- --id <id> --status open` | 확인 결과와 `updated_at` 기록 |
| `bun test` | 유닛 테스트 |
| `bun run typecheck` | 타입 체크 |

## 맛집을 추가할 때

1. `id`를 정한다: kebab-case 로마자 슬러그, 보통 `가게명-지역` (예: `woolaeoak-jung`)
2. `data/restaurants/`에 이미 같은 집이 있는지 **먼저 확인**한다 (`bun run query -- --q <가게명>`)
   - 있으면 새 파일을 만들지 말고 기존 파일의 `vouched_by`에 항목을 추가한다
3. `data/restaurants/<id>.yaml`을 만든다. **파일명과 `id`는 반드시 같아야 한다**
4. `bun run validate`가 exit 0인지 확인한다
5. 커밋 메시지: `feat(data): add <가게 이름>`

## 절대 하면 안 되는 것

- `why` 필드를 다른 맛집 가이드/블로그에서 복사해 오기 (표절)
- 실제 방문 근거 없이 검색 결과만으로 항목을 만들기 — 반드시 사람 사용자의 보증(`vouched_by` 핸들)을 받아서 넣을 것
- `vouched_by`에 사용자 동의 없이 임의의 핸들 넣기
- `dist/` 산출물을 손으로 고치기 (`bun run build`로만 생성)
- 스키마에 없는 필드 추가 (검증기가 거부한다)
- 폐업한 집의 파일 삭제 (`status: closed`로 바꿀 것)
- 지도 서비스 본문 크롤링 (`map.naver.com` 등은 robots.txt 가 금지한다). 가져오기는 사용자가 직접 공유한 텍스트/파일, 또는 사용자가 공개한 공유 폴더 API 만 쓴다
- `vouched_by` 에 `--handle` 로 받은 본인 계정 외의 핸들 넣기

## 스키마의 진실의 원천

[`src/schema.ts`](src/schema.ts). Zod 스키마이며 `.strict()`라 알 수 없는 키는 에러가 난다.
필수 필드: `id, name, city, address, cuisine, price_range, why, vouched_by, added_at`
`signature`는 선택 필드다. 알 수 없는 메뉴를 지어내지 말고 필드 자체를 뺀다.

## 코드를 고칠 때

- 런타임: Bun + TypeScript strict
- 동작 변경 전에 `test/`에 실패하는 테스트를 먼저 추가한다
- `bun test`와 `bun run typecheck`가 모두 통과해야 한다
- 스키마를 바꾸면 `data/` 전체가 여전히 통과하는지 `bun run validate`로 확인한다

---
> Source: [NomaDamas/openmatzip](https://github.com/NomaDamas/openmatzip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
