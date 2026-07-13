## subtitle-pro

> **목표:** `http://localhost:3001/watch?v=VIDEO_ID`로 들어온 YouTube 영상에 한글 자막을 생성(AssemblyAI 음성인식 + Claude 번역)해 영상 위에 오버레이하는 웹 서비스를 구축·유지한다.

# CLAUDE.md

## 하네스: 유튜브 프리미엄 한글자막 서비스 (subtitle-pro)

**목표:** `http://localhost:3001/watch?v=VIDEO_ID`로 들어온 YouTube 영상에 한글 자막을 생성(AssemblyAI 음성인식 + Claude 번역)해 영상 위에 오버레이하는 웹 서비스를 구축·유지한다.

**트리거:** subtitle-pro 관련 개발 작업(스캐폴딩, 자막 생성 백엔드, watch 페이지·오버레이 UI, 통합 검증, 기능 추가·수정·재실행) 요청 시 `subtitle-service-orchestrator` 스킬을 사용하라. 단순 질문은 직접 응답 가능.

**스택:** Next.js(App Router, port 3001) · TailwindCSS · shadcn/ui · TypeScript. 백엔드는 AssemblyAI SDK + @anthropic-ai/sdk. 자막 생성 로직의 검증된 원형은 `~/Documents/subtitle/`(Python)에 있으며 TypeScript로 이식한다.

**핵심 계약:** `types/subtitle.ts`가 백엔드/프론트 경계의 단일 진실 원천. 시간은 초 단위 실수. `SubtitleCue { index, startSec, endSec, text }`, `JobStatus`(pending/extracting/transcribing/translating/done/error).

**환경변수:** `ANTHROPIC_API_KEY`, `ASSEMBLYAI_API_KEY`. 시스템 도구: `yt-dlp`, `ffmpeg`.

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-07-07 | 초기 하네스 구성 (4개 에이전트 + 4개 스킬 + 오케스트레이터) | 전체 | - |
| 2026-07-07 | 하네스 실행 — 서비스 초판 구축 완료(스캐폴딩→백엔드/프론트 병렬→통합 QA). 경계면 버그 0, build 통과. | app/·lib/·hooks/·components/ | 초기 구축 |
| 2026-07-07 | 런타임 버그 2건 부분 재실행 수정: (1) 외부 정리로 유실된 route.ts 복구(POST 404 해소), (2) Button nativeButton 경고 제거 | app/api/subtitles/route.ts, components/watch-view.tsx | 런타임 에러 피드백 |
| 2026-07-07 | 번역 모델 claude-sonnet-5 전환 + 영어 자막 버그 부분 재실행 수정: 번역 실패 은폐 버그 제거(실패 배치 20% 초과 시 job error·캐시 미저장, 400/401 등 결정적 에러 즉시 실패), yt-dlp 일시 403 재시도(3회+player_client 폴백). 근본 원인은 API 크레딧 부족으로 확정(코드·모델 무관) — 오염 캐시 삭제, 충전 후 재실행 필요 | lib/config.ts, lib/translate.ts, lib/extract.ts | 영어 자막 버그 피드백 |
| 2026-07-08 | 오픈소스 공개 준비: git 저장소 초기화 후 GitHub private(revfactory/subtitle-pro) 생성·AGPL-3.0 라이선스 지정·커밋 히스토리 단일화. README 전면 재작성(서비스 설명·아키텍처·설치), CONTRIBUTING·CODE_OF_CONDUCT·.github 이슈/PR 템플릿 추가. 비밀 유출 검사 통과(.env 미추적). | README.md, CONTRIBUTING.md, CODE_OF_CONDUCT.md, .github/, LICENSE | 오픈소스 공개 요청 |
| 2026-07-08 | 번역 속도 최적화(품질 로직 불변): 병렬성 상수 기본값 상향(BATCH_SIZE 5→10, MAX_WORKERS 5→12) + `SUBTITLE_BATCH_SIZE`/`SUBTITLE_MAX_WORKERS` env 노출(안전 파싱·[1,max] 클램프). 파도 수 12→3(300세그먼트 기준) ≈4배 단축. FAILURE_THRESHOLD·지수 백오프·매핑·프롬프트 캐시 전부 미변경, MAX_TOKENS 8000 유지. build 통과. 후속: TPM 429 관측 시 env 하향. | lib/config.ts, lib/translate.ts, .env.example | 번역 속도 개선 피드백 |
| 2026-07-12 | (1) 번역 완료 시 캐시 JSON에 사용 토큰·비용(USD) 기록: translate.ts usage 수집·합산 + 순수함수 `computeTranslationCost`(Sonnet $2/$10 MTok, 프롬프트 캐시 read 0.1×·write 1.25× 반영), pipeline→writeCache(cost) 전달, `CachedResult.cost` 선택필드(옛 캐시 하위호환). 프론트 계약·품질 로직 불변. 캐시 히트 영상은 소급 미기록. (2) watch-view 준비 문구에서 자막 개수 제거("한글 자막 N개가 준비"→"한글 자막이 준비"). build 통과. | lib/config.ts, lib/translate.ts, lib/pipeline.ts, lib/job-store.ts, types/subtitle.ts, components/watch-view.tsx | 비용 기록·UI 문구 피드백 |
| 2026-07-12 | 전체 화면 자막 오버레이 버그 부분 재실행 수정: 근본 원인은 YouTube 네이티브 fullscreen(fs=1)이 iframe만 top layer로 승격해 형제 오버레이가 소실되는 구조 — `fs:0`으로 차단하고 iframe+오버레이를 감싸는 컨테이너 단위 커스텀 Fullscreen API 전환(신규 `hooks/use-fullscreen.ts`, webkit 폴백·fullscreenchange 동기화·Escape 대응). 전체 화면 폰트 vh 기반 clamp 스케일(사용자 설정이 하한), `f` 단축키(입력 포커스 시 무시), 토글 버튼은 컨테이너 내부 배치. 계약·폴링·백엔드·cue 동기화 불변. verify.sh 13통과·build 통과·QA PASS. 알려진 한계: 전체 화면 중 자막 설정 팝오버 접근 불가(해제 후 조정), iframe 포커스 시 `f` 미동작 | hooks/use-fullscreen.ts(신규), components/watch-view.tsx, components/subtitle-overlay.tsx, components/youtube-player.tsx, types/youtube.d.ts, app/globals.css | 전체 화면 자막 미표시 버그 피드백 |

---
> Source: [revfactory/subtitle-pro](https://github.com/revfactory/subtitle-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
