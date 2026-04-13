## ai-builders-meetup

> 커밋 메시지 작성 시 다음 footer는 **절대 추가하지 않습니다**:

# AI Builders Meetup

## Git 커밋 규칙

커밋 메시지 작성 시 다음 footer는 **절대 추가하지 않습니다**:
- `🤖 Generated with [Claude Code]` 또는 유사한 자동 생성 표시
- `Co-Authored-By: Claude` 또는 유사한 공동 저자 표시

커밋 메시지는 conventional commit 형식을 따르며, 본문만 작성합니다.

---

## 현재 작업 계획: 영상 자막 생성

### 요구사항
- **작업**: 2-5개 영상에서 한국어 자막 생성
- **방식**: Whisper (openai-whisper large-v3) - WSL2 환경
- **교정**: 발표자료(PDF) 기반 전문용어 교정
- **출력**: SRT 형식
- **언어**: 한국어 (추후 영어 번역 고려)
- **진행**: 구체화 필요한 부분은 유저에게 질문

### 진행 단계

| 단계 | 작업 | 상태 |
|------|------|------|
| 1 | 영상 파일 준비 (`2-echo-delta/videos/`에 배치) | 완료 |
| 2 | 각 영상별 자막 생성 | 진행중 |
| 3 | 결과 확인 및 수동 교정 | 대기 |
| 4 | (향후) 영어 번역 | 미정 |

### 자막 생성 현황

| 영상 | 최종 SRT 파일 | 상태 |
|------|--------------|------|
| 건호님 | `subtitles/corrected/meetup_02_건호님_corrected.srt` | 완료 |
| 동훈님 | `subtitles/corrected/meetup_02_동훈님_corrected.srt` | 완료 |
| 동운님 | `subtitles/corrected/meetup_02_동운님_corrected.srt` | 완료 |
| 서진님 | `subtitles/corrected/meetup_02_서진님_corrected.srt` | 완료 |
| 패널 | `subtitles/corrected/meetup_02_패널_corrected.srt` | 완료 |

### 영상-PDF 매칭

| 영상 | 발표자료 |
|------|----------|
| 서진님 발표 | `slides/1-김서진-[251230] AI-Native Company.pdf` |
| 동훈님 발표 | `slides/2-이동훈-myrealtrip-donghoon-ailab.pdf` |
| 동운님 발표 | `slides/3-최동운-계산기 압수 당해서...pdf` |
| 건호님 발표 | `slides/4-신건호-AI Builder Meetup...pdf` |
| 키노트 | `slides/0-정구봉-echo-delta-keynote.pdf` |
| 패널 토론 | `terms/glossary.yaml` (통합 용어집) |

### 자막 생성 명령어

```bash
# WSL2 환경에서 venv 활성화 후 실행
source .venv/bin/activate

# 자막 생성 (openai-whisper 사용)
python scripts/subtitle/generate.py "2-echo-delta/videos/raw/{영상파일}.mov"


### 환경 설정

- **환경**: WSL2 (Ubuntu on Windows)
- **Python**: .venv 가상환경
- **패키지**:
  - `openai-whisper` (Whisper AI)
  - `torch` (PyTorch)
  - `ffmpeg` (오디오 처리)

### 주의사항

1. **한글 파일명**: macOS/WSL에서 NFD 형식으로 저장됨. 반드시 NFC 정규화 후 비교:
   ```python
   import unicodedata
   filename = unicodedata.normalize('NFC', filename)
   ```
2. **처리 시간**: 10분 영상당 약 15-20분 소요 (large-v3 모델)
3. **메모리**: large-v3 모델은 약 10GB VRAM 필요
4. **다음 단계**: `/video-subtitle` 스킬 사용
   - `--srt`로 기존 자막 파일 지정하면 정리/교정부터 시작
   - `--reference`로 발표자료 지정하면 전문용어 교정 및 검증

---

## 프로젝트 구조

```
ai-builders-meetup/
├── 2-echo-delta/           # 2회차 밋업
│   ├── slides/             # 발표자료 PDF
│   ├── terms/              # 용어집
│   │   └── glossary.yaml   # 통합 용어집 (패널 토론용)
│   ├── videos/             # 영상 및 자막
│   │   ├── raw/            # 원본 영상 (.mov)
│   │   ├── cropped/        # 크롭된 영상
│   │   ├── subtitles/      # 자막 파일들
│   │   │   ├── raw/        # 원본 SRT
│   │   │   ├── corrected/  # 교정된 SRT
│   │   │   ├── en/         # 영어 번역
│   │   │   └── validation/ # 검증 보고서
│   │   └── burnin_output/  # 자막 합성 영상
│   └── speakers/           # 스피커 정보
├── scripts/
│   ├── subtitle/           # 자막 관련 스크립트
│   │   ├── generate.py         # Whisper로 자막 생성
│   │   ├── generate_api.py     # OpenAI API로 자막 생성
│   │   ├── burnin.py           # ffmpeg burn-in
│   │   └── STYLE_GUIDE.md      # 자막 하드코딩 스타일 가이드
│   └── youtube/            # YouTube 업로드
│       ├── auth.py             # OAuth 인증
│       ├── upload.py           # 영상 업로드
│       └── captions.py         # 자막 업로드
└── .claude/
    ├── hooks/              # Claude Code Hooks
    │   ├── codex-review.sh     # 커밋 시 코드 리뷰
    │   └── suggest-commit.sh   # 커밋 메시지 제안
    ├── skills/             # Claude Code Skills (진입점)
    │   ├── video-subtitle/ # 자막 생성 스킬
    │   ├── finalize/       # 업로드 준비물 완성 (번역+번인)
    │   ├── speaker-guide/  # 스피커 가이드 생성
    │   ├── clarify/        # 요구사항 구체화
    │   └── panel-talk-questions/
    └── agents/             # 스킬에서 호출하는 에이전트
        └── video-subtitle/
            ├── subtitle-generator.md   # Whisper로 자막 생성
            ├── subtitle-cleaner.md     # 중복/hallucination 제거
            ├── subtitle-corrector.md   # 전문용어 교정
            ├── subtitle-validator.md   # 품질 검증 보고서 생성
            ├── subtitle-qa.md          # Claude + Codex 이중 검증
            ├── subtitle-translator.md  # 한국어 → 영어 번역
            └── subtitle-burnin.md      # 영상에 자막 하드코딩
```

## Skills 사용법

- `/video-subtitle` - 영상 자막 생성 및 교정
  - `--video`: 영상 파일 경로 (새로 생성시)
  - `--srt`: 기존 자막 파일 경로 (정리/교정부터 시작시)
  - `--reference`: 발표자료 PDF (교정/검증용)
  - 워크플로우: generator → cleaner → corrector → validator → corrector(2차) → qa → corrector(3차)
- `/finalize` - 업로드 준비물 완성 (영어 번역 + 자막 번인 병렬 실행)
  - `--srt`: 교정된 SRT 파일 경로 (미지정 시 corrected/ 목록에서 선택)
  - 워크플로우: translator + burnin (병렬)
  - 출력: `en/*_en.srt` + `burnin_output/*_burnin.mp4`
- `/speaker-guide` - 스피커 가이드 문서 생성
- `/clarify` - 요구사항 구체화 (질문을 통해 정리)

---

## Codex 코드 리뷰 시스템

커밋 시 자동으로 OpenAI Codex CLI를 통해 코드 리뷰가 실행됩니다.

### 설치

```bash
npm i -g @openai/codex
```

### 동작 방식

1. `git commit` 명령 감지 (PreToolUse Hook)
2. staged 파일 중 코드 파일만 필터링
3. Codex CLI로 리뷰 실행
4. `.reviews/YYYY-MM-DD/{hash}_{timestamp}.md`에 보고서 저장
5. 결과 요약 출력 후 커밋 진행 (차단 없음)

### 리뷰 파일 위치

```
.reviews/
├── 2026-01-06/
│   ├── abc1234_143022.md
│   └── def5678_152145.md
└── ...
```

### 최근 리뷰 확인

```bash
ls -lt .reviews/$(date +%Y-%m-%d)/ | head -5
```

### 리뷰 스킵

```bash
export SKIP_CODEX_REVIEW=1
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gamma4638)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gamma4638)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
