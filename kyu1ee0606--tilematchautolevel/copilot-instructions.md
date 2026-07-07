## tilematchautolevel

> 타일 매칭 퍼즐 게임의 레벨 자동 생성 및 난이도 분석 도구

# TileMatchAutoLevel - Claude Code Guidelines

## Project Overview
타일 매칭 퍼즐 게임의 레벨 자동 생성 및 난이도 분석 도구

## 🎮 게임측 레벨 포맷 정본 (필요 시에만 참조 — 포인터)

이 에디터가 생성한 레벨은 **GameBoost → sp_meowsgarden 게임**이 소비한다. 게임이 실제로 먹는 포맷의 **단일 진실 공급원(정본)** 은 게임 프로젝트 위키에 live 로 유지된다 (복사본 아님 — 항상 최신):

```
/Users/casualdev/sp_meowsgarden/Assets/99.Docs/claudedocs/nodes/DESIGN_LEVEL_MAP_SCHEMA.md
```

> 비주얼 타일 시드(visualTileSeed) 등 t0 분배/렌더 동기화 작업은 위 §4-1 + `DESIGN_TILE_COLOR_BALANCE.md §4-1`(에디터 포팅) 참조. 에디터측 구현: `claudedocs/t0_tile_distribution.md`, `claudedocs/CHANGELOG_20260630_VISUAL_TILE_SEED.md`.

**언제 읽나 (조건부 — 아래 작업일 때만 위 파일 Read)**:
- 레벨/맵 **JSON 스키마**(필드명·구조) 변경/검증
- **타일 ID**(t1~t16/stack_*/craft_*) 또는 **효과/기믹**(ice/chain/grass/bomb_N/curtain/teleporter/frog) enum 추가·수정
- **좌표계**(col_row→world)·**홀짝 레이어 규칙**·**num%3 검증** 관련 작업
- 게임 배포 포맷이 "게임이 안 먹는다"는 이슈 디버깅

**왜**: 이 에디터는 sp_template(타운팝) 기반이라, 게임이 **신규 기믹/필드 추가**하면 자동 반영 안 됨. 위 정본이 게임측 최신 계약 → 드리프트 점검 기준.

> 일반 레벨 생성·난이도 분석 등 평상 작업엔 읽을 필요 없음. **포맷/enum 작업일 때만.**

## Testing Guidelines

### Playwright 활용 (적극 권장)
UI 관련 테스트나 기능 검증 시 Playwright MCP를 적극 활용하세요:

1. **레벨 생성 테스트**
   - 레벨 세트 생성 시 UI에서 진행 상황 확인
   - 생성된 레벨의 검증 결과 확인

2. **난이도 검증 테스트**
   - AutoPlay 패널에서 봇별 클리어율 확인
   - 목표 클리어율과 실제 클리어율 비교

3. **일반적인 테스트 흐름**
   ```
   1. browser_navigate → http://localhost:5173
   2. browser_snapshot → 현재 상태 확인
   3. browser_click → UI 조작
   4. browser_wait_for → 결과 대기
   5. browser_snapshot → 결과 확인
   ```

### API 테스트
백엔드 API 테스트 시:
- `/api/generate/validated` - 검증 기반 레벨 생성
- `/api/analyze/autoplay` - 봇 시뮬레이션 기반 난이도 분석

## Key Endpoints

### Level Generation
- `POST /api/generate` - 기본 레벨 생성
- `POST /api/generate/validated` - 검증 기반 레벨 생성 (재시도 포함)

### Analysis
- `POST /api/analyze` - 정적 난이도 분석
- `POST /api/analyze/autoplay` - 봇 시뮬레이션 분석

## Development Notes

### 동적 목표 클리어율
- 정적 분석 점수 기반으로 봇별 목표 클리어율 동적 계산
- 쉬운 레벨(S/A등급): 높은 목표 클리어율
- 어려운 레벨(C/D등급): 낮은 목표 클리어율

### 레벨 생성 재시도 로직
- 프론트엔드: validation_passed가 false면 최대 10회 재시도
- 백엔드: max_retries 파라미터로 내부 재시도 횟수 설정

## 필수 검증 규칙

### 백엔드 수정 시 프론트엔드 동작 검증 (필수)
백엔드 API 스키마, 엔드포인트, 응답 형식 등을 수정할 때 반드시:
1. **프론트엔드 타입 동기화**: 백엔드 스키마 변경 시 `frontend/src/api/generate.ts` 등 관련 TypeScript 인터페이스도 함께 수정
2. **TypeScript 빌드 검증**: `npx tsc --noEmit`으로 타입 에러 없는지 확인
3. **웹뷰 동작 테스트**: Playwright로 실제 UI에서 해당 기능이 정상 동작하는지 확인
4. **API 호출 테스트**: curl 또는 Playwright로 변경된 API가 정상 응답하는지 확인

수정 → 타입 동기화 → 빌드 검증 → 웹뷰 테스트 순서를 반드시 따를 것.

---
> Source: [kyu1ee0606/TileMatchAutoLevel](https://github.com/kyu1ee0606/TileMatchAutoLevel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
