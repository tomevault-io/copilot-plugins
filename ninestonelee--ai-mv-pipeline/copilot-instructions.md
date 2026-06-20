## ai-mv-pipeline

> name: mv-clip-factory

---
name: mv-clip-factory
version: 3.0.0
description: "음악 + 레퍼런스 영상 → 가사 작성 → 스토리보드 → 웹툰 이미지 생성 → AI 영상 클립 생성 → 최종 MV 편집까지의 전체 뮤직비디오 제작 파이프라인. 실행 스크립트 포함 (Flux 2 Pro, Qwen, Wan, Veo 3). 뮤직비디오, MV, 영상제작, AI영상, 클립생성, 스토리보드, Veo, 이미지투비디오 시 자동 활성화."
preamble-tier: 2
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - WebSearch
  - WebFetch
hooks:
  post-phase4: "이미지 생성 완료 후 /qa로 품질 검수 실행 가능"
  post-phase7: "최종 MV 완성 후 /ship으로 배포 실행 가능"
shared-modules:
  # 점진적 분리 예정 (하이브리드 아키텍처):
  # /ai-image    — Imagen 4.0 + fal.ai Flux Schnell 통합 인터페이스
  # /ai-video    — Veo 3.0 + fal.ai Wan2.2 폴백 체인
  # /ffmpeg-tools — 프레임 추출, 클립 concat, 자막 삽입
  # /cost-tracker — 프로젝트별 비용 대시보드
scripts:
  images:
    - scripts/images/generate_flux2_pro_images.py    # Flux 2 Pro (fal.ai)
    - scripts/images/generate_qwen_images.py         # Qwen Image
    - scripts/images/generate_wan_images.py          # Wan 2.2 Image
  clips:
    - scripts/clips/generate_wan_clips.py            # Wan 2.2 (폴백용)
    - scripts/clips/veo3/veo_generator.py            # Veo 3 단일 호출
    - scripts/clips/veo3/generate_clip.py            # Veo 3 클립 생성
    - scripts/clips/veo3/batch_generate.py           # Veo 3 배치 실행
  build:
    - scripts/build_mv.sh                            # 최종 MV 컴포지션
changelog:
  v3.0.0: "실행 스크립트 통합 (Flux 2 Pro, Qwen, Wan, Veo 3). Ragnarok2 MV 실전 코드 흡수. GitHub 공개 배포 (2026-04-25)."
  v2.0.0: "gstack 통합 포인트 추가, 공유 모듈 마커 삽입, 허브 서버 위자드 문서 보강"
  v1.0.0: "초기 릴리즈 (왕사님 MV 시행착오 기반)"
---

# MV Clip Factory — AI 뮤직비디오 제작 파이프라인

## 핵심 원칙 3가지 (왕사님 MV 시행착오에서 도출)

> 이 원칙을 지키면 **72시간 → 12시간**으로 단축, 비용 **$34 → $20**으로 절감 가능.

### 1. 음원 먼저, 씬 나중
- 음원의 에너지 프로파일(RMS, 비트)이 씬 분할의 근거
- 음원 없이 씬을 나누면 반드시 재조정이 발생 (v4→v5에서 11씬→13씬 재작업)
- **Phase 1에서 음원 확보 → Phase 3에서 librosa 분석 후 씬 분할**

### 2. 한 번에 확정, 한 번에 생성
- 스토리보드 확정 → 필요 이미지 목록 확정 → prompts.json 완성 → 한 번에 일괄 생성
- 분할 생성(57장→9장→3장)은 3일에 걸쳐 비효율 발생
- **`--dry-run`으로 목록 확인 → 한 번에 전량 생성 → `--missing`으로 실패분만 재생성**

### 3. 1장 테스트, N장 실행
- 도구 선택 전 대표 이미지 1장으로 Veo/Kling/fal.ai 비교 (30분)
- Kling AI 5건 생성 후 포기 = 2시간 낭비 (왕사님 실제 사례)
- **동일 이미지 1장 → 3종 비교 → 최적 도구 결정 → 본 작업 배치 실행**

---

## 전체 워크플로우

```
Phase 0: 프로젝트 초기화 & 허브 등록         (10분)
           ↓
Phase 1: 레퍼런스 수집 & ⚡음원 확보⚡       (2시간)
           ↓
Phase 2: 가사 작성 & 런닝타임 검증           (1시간)
           ↓
Phase 3: 음원 분석 → 스토리보드 설계          (2시간)
           ↓
Phase 3.5: ⚡1장 도구 비교 테스트⚡           (30분)
           ↓
Phase 4: 이미지 ⚡일괄⚡ 생성                 (1시간)
           ↓
Phase 5: AI 영상 클립 ⚡배치⚡ 생성           (3시간)
           ↓
Phase 6: 나레이션/자막 (선택)                 (1시간)
           ↓
Phase 7: ffmpeg 편집 & 최종 출력              (1시간)
           ↓
Phase 8: 검수 & 버전 태깅 & 기록             (30분)
```

**⚡ = 왕사님 시행착오에서 추가/변경된 단계**

### 두 가지 시작 방법

| 방법 | 적합한 경우 | 절차 |
|------|-----------|------|
| **웹 위자드** | YouTube URL에서 바로 시작 | `http://localhost:8787/new/` → 5단계 자동 |
| **수동 초기화** | 세밀한 제어가 필요할 때 | Phase 0 원커맨드 실행 → Phase 1부터 순차 |

---

## Phase 0: 프로젝트 초기화 & 허브 등록

새 MV 제작 시 반드시 이 단계를 먼저 수행한다. 허브 서버(`hub_server.py`)가 자동으로 인식하는 구조로 폴더를 생성한다.

### 0-A: 프로젝트 폴더 생성

허브 서버의 루트 디렉토리(`video_creator/`) 아래에 프로젝트 폴더를 만든다.

```bash
PROJECT_ROOT="/path/to/your/video_creator"
PROJECT_NAME="my-new-mv"  # 영문 슬러그 (URL에 사용됨)

mkdir -p "${PROJECT_ROOT}/${PROJECT_NAME}"/{images,clips,downloads,storyboard,output}
```

### 0-B: project.json 생성 (허브 등록 필수 파일)

이 파일이 있어야 허브 서버가 프로젝트를 자동 인식한다.

```json
{
  "name": "프로젝트 표시 이름",
  "slug": "my-new-mv",
  "subtitle": "프로젝트 한줄 설명 — MV",
  "description": "프로젝트 상세 설명 (허브 카드에 표시)",
  "color": "#D4A853",
  "image_dir": "images",
  "clip_dir": "clips",
  "prompts_file": "prompts.json"
}
```

**필드 설명:**

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| `name` | O | 허브에 표시되는 프로젝트명 | `"Journey to the Cross"` |
| `slug` | O | URL 경로 (영문, 하이픈 가능) | `"journey"` → `localhost:8787/journey/` |
| `subtitle` | O | 카드 부제목 | `"Pixar Style MV"` |
| `description` | O | 카드 설명 텍스트 | `"픽사 3D 스타일 워십 MV"` |
| `color` | O | 테마 색상 (hex) | `"#D4A853"` (골드), `"#4A90D9"` (블루) |
| `image_dir` | O | 이미지 폴더 경로 (상대) | `"images"` |
| `clip_dir` | O | 클립 저장 폴더 경로 (상대) | `"clips"` |
| `prompts_file` | O | 영상 생성 프롬프트 JSON 경로 | `"prompts.json"` |

### 0-C: prompts.json 생성 (빈 파일로 시작)

이미지별 영상 생성 프롬프트를 담는 파일. 초기에는 빈 객체로 생성하고, Phase 4~5에서 채운다.

```json
{}
```

**나중에 이미지가 추가되면 다음 형식으로 채운다:**

```json
{
  "01_scene_name.jpg": {
    "en": "Animate this illustration. [영문 프롬프트]. 5 seconds.",
    "ko": "일러스트를 애니메이트. [한글 프롬프트]. 5초."
  }
}
```

### 0-D: 최종 폴더 구조 확인

```
video_creator/
├── hub_server.py              ← 허브 서버
├── my-new-mv/                 ← 새 프로젝트 (자동 인식됨)
│   ├── project.json           ← 허브 등록 (필수)
│   ├── prompts.json           ← 영상 프롬프트 (필수)
│   ├── images/                ← 웹툰/픽사 이미지
│   ├── clips/                 ← AI 영상 클립 (Veo 생성물)
│   ├── downloads/             ← 레퍼런스 영상
│   ├── storyboard/            ← 스토리보드
│   └── output/                ← 최종 MV
├── 0320_왕사님/               ← 기존 프로젝트
├── 0321_신이랑법률사무소/      ← 기존 프로젝트
└── journey-to-cross/          ← 기존 프로젝트
```

### 0-E: 허브 서버 반영

프로젝트 폴더 생성 후 허브 서버를 재시작하면 자동 인식된다.

```bash
# 방법 1: 서버 재시작
lsof -i :8787 -t | xargs kill; sleep 1
cd ${PROJECT_ROOT} && python3 hub_server.py &

# 방법 2: 위자드 사용 (웹 UI에서 전 과정 자동)
# http://localhost:8787/new/ 에서 YouTube URL 입력 → 자동 생성
```

**확인:** `http://localhost:8787/` 에서 새 프로젝트 카드가 보이면 성공.

### 0-F: 초기화 스크립트 (권장)

스킬의 `resources/init_project.sh`를 허브 루트에 복사 후 실행한다.

```bash
# 허브 서버 루트에서 실행
cd /path/to/your/video_creator
bash init_project.sh my-new-mv
```

스크립트가 자동 생성하는 파일:
- `project.json` — 허브 서버 자동 등록
- `prompts.json` — 이미지→영상 프롬프트 DB (빈 상태)
- `constitution.md` — 스타일/예산 불변 규칙 (직접 편집)
- `progress.md` — Phase 진행 추적
- `output/build_mv.sh` — ffmpeg 최종 편집 스크립트

### 0-G: 수동 초기화 (원커맨드)

```bash
# === MV 프로젝트 수동 초기화 ===

SLUG="my-new-mv"
NAME="나의 새 MV"
SUBTITLE="프로젝트 부제 — MV"
DESC="프로젝트 설명"
COLOR="#D4A853"
ROOT="/path/to/your/video_creator"

# 폴더 생성
mkdir -p "${ROOT}/${SLUG}"/{images,clips,downloads,storyboard,output}

# project.json
cat > "${ROOT}/${SLUG}/project.json" << EOF
{
  "name": "${NAME}",
  "slug": "${SLUG}",
  "subtitle": "${SUBTITLE}",
  "description": "${DESC}",
  "color": "${COLOR}",
  "image_dir": "images",
  "clip_dir": "clips",
  "prompts_file": "prompts.json"
}
EOF

# prompts.json (빈 파일)
echo '{}' > "${ROOT}/${SLUG}/prompts.json"

echo "✓ 프로젝트 생성 완료: ${ROOT}/${SLUG}"
echo "  허브 서버 재시작 후 http://localhost:8787/${SLUG}/ 에서 확인"
```

---

## Phase 1: 레퍼런스 수집 & ⚡음원 확보⚡

> **원칙 #1 적용**: 음원이 확보되어야 Phase 3(씬 분할)을 정확히 할 수 있다.
> 음원 없이 씬을 나누면 반드시 재조정 필요. (왕사님: 11씬→13씬 재작업)

### 1-A: 레퍼런스 영상 다운로드

```bash
# youtube-downloader 스킬 사용
python3 ~/.claude/skills/youtube-downloader/scripts/download_video.py "{URL}" -q best -o "{프로젝트}/downloads"
```

### 1-B: 씬 프레임 자동 추출

ffmpeg scene detection으로 주요 장면 전환 프레임을 추출한다.

```bash
# 출력 폴더 생성
mkdir -p {프로젝트}/{VIDEO_ID}_extract/frames

# scene detection (threshold 0.3 권장, 너무 많으면 0.4로 올림)
ffmpeg -i "{영상파일}" \
  -vf "select=gt(scene\,0.3),showinfo" \
  -fps_mode vfr -q:v 2 \
  "frames/scene_%04d.jpg" \
  2>ffmpeg_scene.log
```

**주의**: zsh에서 `2>` 리다이렉트가 ffmpeg 출력으로 해석될 수 있음. `/bin/bash -c '...'`로 감싸서 실행할 것.

### 1-C: 타임스탬프 인덱스 생성

```bash
grep "pts_time" ffmpeg_scene.log | \
  sed -n 's/.*n: *\([0-9]*\) .*pts_time:\([0-9.]*\).*/\1,\2/p' \
  > scene_index.csv
```

### 1-D: 콘택트 시트 생성

```python
from PIL import Image, ImageDraw, ImageFont
import os, math

frames_dir = "frames"
files = sorted([f for f in os.listdir(frames_dir) if f.endswith(".jpg")])

cols = 5
rows = math.ceil(len(files) / cols)
thumb_w, thumb_h = 256, 144  # 16:9
label_h = 22

sheet = Image.new("RGB", (cols*(thumb_w+3)+3, rows*(thumb_h+label_h+3)+3), (0,0,0))
draw = ImageDraw.Draw(sheet)
font = ImageFont.truetype("/System/Library/Fonts/AppleSDGothicNeo.ttc", 11)

for i, fname in enumerate(files):
    col, row = i % cols, i // cols
    x = 3 + col * (thumb_w + 3)
    y = 3 + row * (thumb_h + label_h + 3)
    img = Image.open(os.path.join(frames_dir, fname)).resize((thumb_w, thumb_h), Image.LANCZOS)
    sheet.paste(img, (x, y))
    draw.text((x+2, y+thumb_h+3), fname.replace(".jpg",""), fill=(180,180,180), font=font)

sheet.save("contact_sheet.jpg", quality=90)
```

### 1-E: 댓글/반응 분석 (선택)

YouTube 영상 댓글을 수집하여 관객 감정 패턴, 명장면, 명대사를 추출한다.

**분석 프레임워크:**
- 핵심 감정 키워드 빈도
- 명장면/명대사 언급 순위
- 재관람 의향 표현
- 관객 연령대/성별 추정

### 1-F: ⚡레퍼런스 과잉 추출 방지 (중요)

> **교훈**: 왕사님에서 500+장 추출 → 실사용 30장 (6% 활용률)

- 콘택트 시트를 먼저 생성하여 전체 조감
- **씬당 3~5장만 타겟팅** → 나머지는 추출하지 않음
- `Last_selected_img/` 같은 큐레이션 폴더로 선별 관리

### 1-G: ⚡음원 생성 & 확보 (Phase 1에서 완료 필수)

음원이 있어야 에너지 분석 → 씬 분할이 정확해진다. **Phase 3 전에 반드시 확보**.

```bash
# Suno AI 음원 생성 후 다운로드
# 스타일 3종 시도 → 최종 1곡 선택 → output/music/ 에 저장
```

**산출물**: `output/music/{곡명}.mp3` — 이후 모든 타이밍의 기준

---

## Phase 2: 가사 작성 & 런닝타임 검증

### 2-A: 서사 구조 → 가사 변환

원작의 서사 아크를 음악 구조로 매핑한다.

```
서사 아크                    → 음악 구조
───────────────────────────────────────
설정/발단 (Setup)           → Intro + Verse 1
갈등/전개 (Rising Action)   → Pre-Chorus + Chorus 1
심화/전개 (Development)     → Verse 2
위기/절정 (Climax)          → Pre-Chorus 2 + Chorus 2
결말/해소 (Resolution)      → Bridge + Final Chorus
여운 (Denouement)           → Outro
```

### 2-B: 가사 작성 원칙

1. **핵심 모티프 3~4개 설정** — 가사 전체를 관통하는 상징 (예: 강, 줄, 밥상, 이름)
2. **모티프 의미 전환** — 시작과 끝에서 같은 단어가 다른 의미를 가짐
3. **Spoken Word 삽입** — 클라이맥스 직전에 대사 형태의 가사로 감정 폭발
4. **Outro는 역설** — 가장 비극적인 순간에 가장 따뜻한 말로 마무리

### 2-B2: 런닝타임 기반 가사 분량 검증 (필수)

가사 작성 후 **반드시** 목표 런닝타임에 맞는지 검증한다. 템포에 따른 행당 소요 시간이 크게 다르므로, 감으로 작성하면 30~60% 초과하는 경우가 흔하다.

**행당 소요 시간 기준표:**

| BPM 범위 | 장르 특성 | 행당 소요 시간 | 비고 |
|---------|----------|--------------|------|
| 60~70 | 느린 발라드, 워십 | **5~6초** | 음 끌기, 호흡 길음 |
| 70~85 | 미드 발라드, CCM | **4~5초** | 표준 |
| 85~100 | 팝 발라드, 포크 | **3~4초** | 비교적 빠른 전달 |
| 100~120 | 업템포, 댄스 팝 | **2~3초** | 빠른 가사 전달 |

**검증 공식:**

```
가용 시간 = 목표 런닝타임 - Intro(8~12초) - 간주(4~8초) - Outro 여운(10~15초)
적정 행 수 = 가용 시간 ÷ 행당 소요 시간
```

**검증 예시 (2분 40초, 72 BPM 발라드):**

```
가용 시간 = 160초 - 8초(인트로) - 6초(간주) - 14초(아웃트로 여운) = 132초
행당 소요 = 5초 (72 BPM 느린 발라드)
적정 행 수 = 132 ÷ 5 = 약 26행

→ 가사가 26행을 넘으면 런닝타임 초과!
```

**섹션별 행 수 가이드 (느린 발라드 기준):**

| 구간 | 권장 행 수 | 소요 시간 |
|------|----------|----------|
| Verse | 4행 | ~20초 |
| Pre-Chorus | 2행 | ~10초 |
| Chorus | 4행 | ~22초 |
| Bridge | 3행 | ~15초 |
| Outro (보컬) | 3~4행 | ~15초 |

**흔한 실수:**
- 김이나 스타일 작사 시 서사적 표현에 몰입하여 행 수가 2배로 늘어남
- Verse를 8행으로 쓰면 Verse 하나에 40초 → 2분 40초 곡에 Verse 2개도 못 넣음
- Chorus도 8행이면 Chorus 자체가 40초 → 곡의 1/4을 차지

**검증 템플릿:**

```markdown
### 분량 검증 ({목표시간}, {BPM} BPM)

| 구간 | 행 수 | 소요 시간 | 누적 |
|------|-------|----------|------|
| Intro | — | {N}초 | 0:{NN} |
| Verse 1 | {N}행 | {N}초 | {누적} |
| Pre-Chorus | {N}행 | {N}초 | {누적} |
| Chorus | {N}행 | {N}초 | {누적} |
| (간주) | — | {N}초 | {누적} |
| Verse 2 | {N}행 | {N}초 | {누적} |
| Chorus 2 | {N}행 | {N}초 | {누적} |
| Bridge | {N}행 | {N}초 | {누적} |
| Outro | {N}행 | {N}초 | {누적} |
| 여운 | — | {N}초 | **{최종}** |
| **합계** | **{총행}행** | | **{목표}** ✓/✗ |
```

### 2-C: Suno AI 음원 생성

**스타일 프롬프트 템플릿:**

```
Style of Music: {장르}, {보컬 스타일}, {악기 구성}, {BPM}, {박자},
{키/조성}, {분위기 키워드}, {믹싱 특성}, {엔딩 스타일}
```

**예시 (한국 사극 발라드):**
```
Style of Music: Korean cinematic ballad, solo male vocal with emotional vibrato,
traditional Korean instruments gayageum and daegeum and haegeum,
gentle piano arpeggios, slow orchestral strings building gradually,
68 BPM, 6/8 time, dark minor key, intimate whisper-like verse,
powerful belting chorus, haegeum solo in bridge,
full orchestra and choir climax in final chorus,
film soundtrack quality, reverb-heavy atmospheric mix,
bittersweet melancholy, fadeout ending with gayageum and water ambience
```

**가사 입력 형식:**

```
[Intro]
가사 텍스트

[Verse 1]
가사 텍스트

[Pre-Chorus]
...
```

---

## Phase 3: 음원 분석 → 스토리보드 설계

> **원칙 #1 적용**: 음원 에너지 프로파일이 씬 분할의 근거.
> 이 단계에서 Phase 1-G에서 확보한 음원을 분석하여 자동으로 씬 경계를 도출한다.

### 3-A: 음원 에너지 프로파일 분석

실제 음원 파일을 분석하여 에너지 곡선을 도출한다.

```python
import librosa
import numpy as np

y, sr = librosa.load("음원.mp3")
# RMS 에너지
rms = librosa.feature.rms(y=y)[0]
# 10초 단위 세그먼트 평균
segment_len = int(sr * 10 / 512)
energy_profile = [np.mean(rms[i:i+segment_len]) for i in range(0, len(rms), segment_len)]
```

### 3-B: 씬 분할 기준

| 기준 | 설명 |
|------|------|
| 가사 섹션 경계 | [Intro], [Verse], [Chorus] 등 구간 전환 |
| 에너지 변화점 | 급격한 볼륨/템포 변화 지점 |
| 서사 전환점 | 감정 아크의 전환 지점 |
| 영상 호흡 | 4~8초 단위 클립을 3~5개 묶어 1씬 |

### 3-C: 씬 명세서 템플릿

각 씬을 다음 형식으로 상세 기술한다.

```markdown
### Scene {번호}: {제목}

| 항목 | 내용 |
|------|------|
| **구간** | {시작}~{끝} |
| **길이** | {초}초 |
| **에너지** | {1~10} |
| **가사 섹션** | {Intro/Verse/Chorus...} |
| **설정** | {장소, 시간, 날씨, 분위기} |
| **액션** | {카메라 움직임, 캐릭터 행동, 감정 비트} |
| **카메라** | {Wide/Medium/Close-up, 움직임} |
| **조명** | {광원, 비율, 색온도} |
| **컬러** | {hex 컬러 3~5개} |
| **감정** | {감정 아크 — 시작→끝} |
| **레퍼런스 이미지** | {파일명 목록} |
```

### 3-D: 컬러 팔레트 설정

MV 전체를 관통하는 5색 팔레트를 정의한다.

```
예시:
#1B2838 — 야반 남색 (밤, 슬픔, 왕궁의 어둠)
#8B6914 — 금박 (왕관, 촛불, 따뜻한 기억)
#C4463A — 사약 붉은색 (위기, 배신, 피)
#D4C5A9 — 창호지 미색 (일상, 평화, 밥상)
#4A6741 — 영월 산록 초록 (자연, 유배지, 희망)
```

---

## Phase 3.5: ⚡1장 도구 비교 테스트 (신규)

> **원칙 #3 적용**: 도구 선택 전 대표 이미지 1장으로 비교. Kling 5건 생성 후 포기 = 2시간 낭비 방지.

### 테스트 방법

1. Phase 4에서 사용할 대표 이미지 **1장만** Imagen 4.0으로 먼저 생성
2. 같은 이미지로 영상 도구 **3종 비교** (각 1클립씩):

```python
# 동일 이미지, 동일 프롬프트로 도구 비교
tools = {
    "veo-3.0":  {"model": "veo-3.0-generate-001",      "cost": "$0.50", "len": "8s"},
    "veo-fast": {"model": "veo-3.0-fast-generate-001",  "cost": "$0.25", "len": "8s"},
    "wan-2.2":  {"model": "fal-ai/wan-i2v",             "cost": "$0.10", "len": "5s"},
}
# 이미지 생성 도구도 비교 가능
image_tools = {
    "imagen-4":  {"model": "imagen-4.0-generate-001", "cost": "$0.04"},
    "flux2-pro": {"model": "fal-ai/flux-pro/v1.1",    "cost": "$0.10"},
}
```

3. **비교 기준**: 스타일 일관성, 움직임 자연스러움, 캐릭터 변형 정도
4. 최적 도구 결정 → Phase 5 전체 적용

**소요**: 30분, **비용**: ~$0.85 (3클립)
**절약**: 부적합한 도구로 N클립 생성 후 폐기하는 비용 방지

---

## Phase 4: 웹툰 이미지 ⚡일괄⚡ 생성

<!-- [MODULE: ai-image] Imagen 4.0 / fal.ai Flux Schnell 호출 집중 구간
     추출 예정: /ai-image 모듈 (Imagen + Flux 통합 인터페이스, 재시도 로직, 비용 계산)
     현재: 이 SKILL.md 내 인라인 구현 -->

> **원칙 #2 적용**: prompts.json 완성 후 한 번에 전량 생성.
> 분할 생성(왕사님: 57→9→3장, 3일)은 비효율의 원인.

### 4-A: 캐릭터 디자인 잠금

모든 이미지 생성 시 캐릭터 외형을 일관되게 유지한다.

**기본형 (단일 타임라인):**
```python
CHARACTER_LOCK = {
    "주인공": "17-year-old Korean boy, black silk headband (manggeon), "
              "white dopo robe, melancholic dark eyes, slim build, pale skin",
    "조연": "40s Korean man, broad shoulders, short dark beard, "
            "brown rough peasant clothes, weathered warm face, strong hands",
}
```

**타임라인별 정의 (파리대사관 실전 패턴, 권장):**

캐릭터가 시간대/상황별로 다른 모습을 보이는 경우, 타임라인별로 분리 정의한다.
이 패턴은 파리대사관 프로젝트에서 검증되었으며, 캐릭터 일관성 보정률을 50% → 0%로 줄였다.

```python
CHARACTER_LOCK = {
    "서연": {
        "present": "Late 50s Korean woman, short salt-and-pepper bob, "
                   "oval face, quiet inner strength, dark navy work vest, "
                   "grey long-sleeve, company badge, no makeup",
        "boardroom": "Same woman but in maroon blazer, white silk blouse, "
                     "pearl earrings, commanding authoritative posture",
        "flashback": "Same woman at age 45, smoother skin, black bob with "
                     "silver only at temples, navy diplomat suit, "
                     "noble posture, warm gold lighting",
        "ending": "Same 50s woman but with victorious smile, cream suit, "
                  "confident stride, warm golden lighting",
    }
}

# prompts.json에서 타임라인 지정
# "characters": ["서연.present"]  → present 묘사가 프롬프트에 자동 주입
# "characters": ["서연.flashback"] → flashback 묘사가 주입
```

**앵커 이미지 선정 (중요):**
- 각 타임라인에서 가장 잘 나온 이미지 1장을 **앵커**로 선정
- 나머지 이미지는 앵커와 비교하여 일관성 검증
- 불일치 시 해당 이미지만 재생성 (전체 재생성 금지)

### 4-A2: 이미지 재생성 시 v1 백업 (파리대사관 교훈)

캐릭터 불일치로 이미지를 재생성할 때, 원본을 반드시 백업한다.

```bash
# v1 이미지 백업 후 v2 재생성
mkdir -p images_v1_backup
cp images/02_*.jpg images/03_*.jpg images_v1_backup/

# v2 재생성 (불일치 이미지만)
python3 scripts/generate_images.py --only 02,03,04,05,07,09,12,16,18
```

**검증 기준 (3가지):**
1. 머리 스타일/색상 일치
2. 얼굴형/연령대 일치
3. 의상/액세서리 일치

**파리대사관 실적:** v1 18장 → 9장 유지 + 9장 재생성 = 총 $1.28 (최적화 $0.76)

### 4-B: 스타일 프리픽스 (모든 프롬프트에 공통 적용)

```python
STYLE_PREFIX = (
    "Korean historical sageuk webtoon/manhwa illustration style, "
    "detailed ink linework with digital coloring, warm earth tones, "
    "muted color palette, cinematic 16:9 widescreen composition, "
    "dramatic lighting, film-quality atmosphere. "
)
```

### 4-C: 이미지 생성 API (Google Imagen 4.0)

```python
from google import genai
from google.genai import types

client = genai.Client(api_key=API_KEY)

result = client.models.generate_images(
    model='imagen-4.0-generate-001',
    prompt=STYLE_PREFIX + CHARACTER_LOCK["주인공"] + scene_prompt,
    config=types.GenerateImagesConfig(
        number_of_images=1,
    )
)

# 이미지 저장
img_data = result.generated_images[0].image.image_bytes
with open(f"{output_dir}/{filename}.jpg", "wb") as f:
    f.write(img_data)
```

### 4-D: 중복 제거

대량 생성 후 MD5 해시 기반으로 동일 이미지를 탐지하여 제거한다.

```python
import hashlib, os

hashes = {}
for f in sorted(os.listdir(img_dir)):
    with open(os.path.join(img_dir, f), "rb") as fp:
        h = hashlib.md5(fp.read()).hexdigest()
    if h in hashes:
        os.rename(os.path.join(img_dir, f), os.path.join("_duplicates", f))
    else:
        hashes[h] = f
```

### 4-E: 내용 기반 파일명 리네이밍

이미지 내용을 분석하여 다음 형식으로 파일명을 변경한다.

```
{번호}_{장소}_{캐릭터}_{액션}_{감정}.jpg

예시:
01_palace_throne_young_king_audience.jpg
24_tiger_eom_encounter_misty_forest.jpg
44_farewell_eom_crying_holding_hands.jpg
66_silhouette_carrying_body_sunrise_river.jpg
```

### 4-F: 13씬 폴더 분류

생성된 이미지를 스토리보드 씬별 폴더로 분류한다.

```
Last_selected_img/
├── 01_전주_어둠속대금소리/
├── 02_인트로_나는이제어디로갑니까/
├── 03_1절전반_이산골에왕은없다/
│   ├── 09_arrival_overlooking_riverside_village.jpg
│   ├── 11_village_people_group_mountain_path.jpg
│   └── ...
├── ...
└── 13_아웃트로_따뜻한데로/
```

---

## Phase 5: AI 영상 클립 생성

<!-- [MODULE: ai-video] Veo 3.0 / fal.ai Wan2.2 호출 집중 구간
     추출 예정: /ai-video 모듈 (Veo + Wan 폴백 체인, 배치 생성, 상태 폴링, 비용 계산)
     현재: 이 SKILL.md 내 인라인 구현 -->

### 5-A: 프롬프트 설계 원칙

각 이미지별 영상 프롬프트를 **영문 + 한글** 이중으로 작성한다.

**프롬프트 구조:**
```
[스타일 지시] + [원본 이미지 참조] + [구체적 움직임] + [카메라 워크] + [감정/분위기] + [시간]

예시:
"Animate this Korean historical webtoon illustration.
 The young king in white robe sits motionless on the throne,
 then slowly stands up. His robe flows as he turns his back to camera.
 Candlelight flickers. Melancholic dreamlike movement. 5 seconds."
```

**핵심 지침:**
- 반드시 `"Animate this Korean historical webtoon illustration."` 으로 시작
- 원본 이미지의 구도와 캐릭터를 유지하도록 명시
- 움직임은 3~5개 구체적 동작으로 기술
- 카메라 움직임 명시 (pan, zoom, orbit, static)
- 감정 키워드 포함
- 초 단위 길이 명시

### 5-B: Google Veo 3.0 API 영상 생성

```python
from google import genai
from google.genai import types
import time

client = genai.Client(api_key=API_KEY)

# 이미지 로드
with open(image_path, 'rb') as f:
    img_data = f.read()

# 비동기 생성 요청
op = client.models.generate_videos(
    model='veo-3.0-generate-001',
    prompt=prompt_text,
    image=types.Image(image_bytes=img_data, mime_type='image/jpeg'),
    config=types.GenerateVideosConfig(
        aspect_ratio='16:9',
        number_of_videos=1,
    )
)

# 완료 대기 (10초 간격 폴링)
while not op.done:
    time.sleep(10)
    op = client.operations.get(op)

# 영상 다운로드
if op.result and op.result.generated_videos:
    vid = op.result.generated_videos[0]
    video_bytes = client.files.download(file=vid.video)
    with open(output_path, 'wb') as f:
        f.write(video_bytes)
```

### 5-C: 모델 폴백 전략

Rate limit 발생 시 자동으로 대체 모델로 전환한다.

```python
MODELS = [
    "veo-3.1-generate-preview",   # 최신 프리뷰 (2025.03~)
    "veo-3.0-generate-001",       # 안정 고품질
    "veo-3.0-fast-generate-001",  # 빠른 생성
    "veo-2.0-generate-001",       # 최후 폴백
]

MAX_RETRIES = 3
RETRY_DELAYS = [60, 90, 120]  # 지수 백오프 (초) — 30초는 너무 짧음
```

**폴백 로직 (실전 검증 2026-03-29):**
```
veo-3.0 시도 → 429 에러 → 90초 대기 → 재시도
    → 다시 429 → 90초 대기 → 재시도
    → 다시 429 → veo-3.0-fast로 전환
    → 성공 시 완료 / 실패 시 fal.ai Wan2.2로 전환
```

**fal.ai 폴백 (Veo 3회 이상 실패 시):**
```python
import fal_client, base64

with open(img_path, "rb") as f:
    data_url = f"data:image/png;base64,{base64.b64encode(f.read()).decode()}"

result = fal_client.subscribe("fal-ai/wan-i2v", arguments={
    "prompt": prompt_text,
    "image_url": data_url,
    "num_frames": 81,
    "resolution": "480p",
})
# result["video"]["url"]로 다운로드 (curl 사용 — Python urllib은 SSL 오류 발생)
```

**fal.ai 주의사항:**
- 해상도가 Veo(720x1280)와 다름 (480x832) → ffmpeg `pad` 필터로 통일 필수
- SSL 인증서 오류 시 `curl -sL` 또는 `subprocess` 사용
- `.env`의 `FAL_KEY` 환경변수 필요

### 5-D: 비용 추적

```python
VEO3_COST_PER_CLIP = 0.50  # USD

gen_cost = {
    "clips_generated": 0,
    "session_clips": 0,
    "session_cost_usd": 0.0,
    "total_estimated_usd": 0.0,
    "remaining_clips": 66,
    "remaining_cost_usd": 33.0,
    "per_clip_usd": 0.50,
}
```

### 5-E: 배치 생성 전략 (Rate Limit 대응)

**실전 검증 결과 (2026-03-29):** Veo 3.0은 동시 3~4개 작업까지 허용. 순차 1개씩은 비효율적.

**권장 배치 전략:**

```python
def generate_videos_batch(filenames, batch_size=3, wait_between=90):
    """배치 제출 → 완료 대기 → 다음 배치"""
    pending = [f for f in filenames if not os.path.exists(clip_path(f))]

    for i in range(0, len(pending), batch_size):
        batch = pending[i:i+batch_size]

        # Phase 1: 배치 제출 (3초 간격)
        operations = []
        for fname in batch:
            try:
                op = submit_veo_job(fname)
                operations.append((fname, op))
                time.sleep(3)
            except Exception as e:
                if "429" in str(e):
                    break  # 제한 도달, 이 배치까지만

        # Phase 2: 완료 대기 (15초 폴링)
        for fname, op in operations:
            wait_and_download(op, clip_path(fname))

        # Phase 3: 다음 배치 전 쿨다운
        if i + batch_size < len(pending):
            time.sleep(wait_between)
```

**스크립트 재실행 패턴:**
- 기존 파일 자동 SKIP → 실패분만 재시도
- 90초 대기 후 재실행 → 점진적 완성

---

## Phase 6: 대시보드 & 관리 웹앱

### 6-A: 서버 아키텍처

```python
# Python HTTP 서버 (Port 8787)
from http.server import HTTPServer, SimpleHTTPRequestHandler

class MVHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/api/images':    # 이미지 목록 + 프롬프트
        elif self.path == '/api/status':  # 생성 상태 + 비용
        elif self.path.startswith('/img/'): # 원본 이미지 서빙
        elif self.path.startswith('/clip/'): # 생성 클립 서빙

    def do_POST(self):
        if self.path == '/api/generate':  # 선택 이미지 생성 시작

HTTPServer(("", 8787), MVHandler).serve_forever()
```

### 6-B: API 엔드포인트

| 엔드포인트 | 메서드 | 응답 |
|------------|--------|------|
| `/api/images` | GET | `[{filename, prompt_en, prompt_ko, has_clip, status}]` |
| `/api/status` | GET | `{status: {파일별상태}, cost: {비용정보}}` |
| `/api/generate` | POST | `{status: "started", count: N}` |
| `/img/{name}` | GET | 원본 이미지 파일 |
| `/clip/{name}` | GET | 생성된 영상 클립 |

### 6-C: 프론트엔드 UI 요구사항

- 10개씩 페이지네이션
- 이미지 + 영/한 프롬프트 표시
- 체크박스 선택 → "영상 생성" 버튼
- 실시간 상태 뱃지 (idle / generating / done / error)
- 비용 대시보드 (생성완료 / 누적비용 / 잔여예상 / 클립단가)
- 완료된 클립 미리보기 (인라인 비디오 플레이어)

---

## Phase 7: 최종 MV 편집

<!-- [MODULE: ffmpeg-tools] ffmpeg 편집 파이프라인 집중 구간
     추출 예정: /ffmpeg-tools 모듈 (프레임 추출, 클립 concat, 자막 burn-in, 속도 조절)
     현재: 이 SKILL.md 내 인라인 구현 -->

### 7-A: 클립 시퀀싱

스토리보드 타임코드에 맞춰 클립을 배치한다.

```python
# ffmpeg concat 방식
# filelist.txt:
# file 'clip_01.mp4'
# file 'clip_02.mp4'
# ...

ffmpeg -f concat -safe 0 -i filelist.txt \
  -c copy mv_visual_only.mp4
```

### 7-B: 음원 합성

> **경고: `-shortest` 함정 (왕사님 실전 교훈)**
> 클립 합계 > 음원 길이 시 뒤쪽 클립이 잘림.
> 블랙 프레임 삽입 시 전체 길이 증가 → 엔딩 클립이 잘릴 수 있음.
> **반드시 합성 전에 누적 타임라인을 계산하고 핵심 클립 위치를 확인할 것.**

```bash
# 1. 사전 길이 검증 (필수!)
VIDEO_DUR=$(ffprobe -v error -show_entries format=duration -of csv=p=0 mv_visual_only.mp4)
MUSIC_DUR=$(ffprobe -v error -show_entries format=duration -of csv=p=0 music.mp3)
echo "영상: ${VIDEO_DUR}s / 음원: ${MUSIC_DUR}s"
# 영상 > 음원이면 뒤쪽 클립 잘림 주의!

# 2. 음원 합성
ffmpeg -i mv_visual_only.mp4 -i music.mp3 \
  -c:v copy -c:a aac -shortest \
  mv_final.mp4
```

### 7-C: TTS 나레이션 (선택)

컴필레이션/코미디 포맷에서 검증된 나레이션 파이프라인 (funny_cats 실전 사례):

```python
# Edge TTS (무료 한국어 음성)
import edge_tts, asyncio

async def generate_tts(text, output_path, voice="ko-KR-HyunsuNeural"):
    """남성: ko-KR-HyunsuNeural / 여성: ko-KR-SunHiNeural"""
    communicate = edge_tts.Communicate(text, voice)
    await communicate.save(output_path)

# TTS 넘침 검증 (중요!)
# audio_dur + 0.3 <= clip_dur 여야 함
# 넘치면 나레이션 텍스트를 줄이거나 TTS 속도 조정
```

```bash
# 나레이션을 클립에 합성
ffmpeg -i clip.mp4 -i narration.wav \
  -c:v copy -c:a aac -b:a 192k \
  -filter_complex "[1:a]adelay=500|500[a1];[0:a][a1]amix=inputs=2" \
  clip_with_narration.mp4
```

### 7-D: 가사 자막 (ASS 형식)

```ass
[Script Info]
Title: 강을 건너다
ScriptType: v4.00+

[V4+ Styles]
Style: Lyrics,Pretendard,28,&H00FFFFFF,&H000000FF,0,0,0,0,100,100,0,0,1,2,0,2,30,30,40,1

[Events]
Dialogue: 0,0:00:10.00,0:00:15.00,Lyrics,,0,0,0,,나는 이제 어디로 갑니까
```

```bash
ffmpeg -i mv_final.mp4 -vf "ass=subtitles.ass" mv_with_subs.mp4
```

---

## Phase 8: ⚡검수 & 버전 태깅 & 기록 (신규)

> **교훈**: 왕사님에서 306MB와 97MB 두 파일이 공존, 어느 것이 최종인지 불명확.

### 8-A: 최종 파일 명명 규칙

```
output/FINAL_{프로젝트명}_v{N}.mp4
```

예: `output/FINAL_강을건너다_v1.mp4`

- 수정할 때마다 v{N+1}로 새 파일 생성 (덮어쓰기 금지)
- 중간 산출물(merged, concat)은 `output/_work/`에 보관

### 8-B: 검수 체크리스트

```markdown
- [ ] 영상 전체 재생 (끊김, 검은 화면 없음)
- [ ] 음원 싱크 확인 (가사와 씬 전환 일치)
- [ ] 해상도/비율 확인 (16:9 또는 9:16)
- [ ] 자막 가독성 (있는 경우)
- [ ] 최종 파일명 FINAL_v{N} 태깅
```

### 8-C: progress.md 업데이트

```markdown
## 현재 상태
- Phase: 8 (완료)
- 최종 파일: output/FINAL_{프로젝트명}_v1.mp4
- 비용: ${총액}
```

### 8-D: 작업 로그 기록

`work-logger` 스킬 실행하여 `/logs/`에 기록.

---

## 빠른 실행 가이드

### 최소 요구사항

- Python 3.10+
- Google API Key (Imagen 4.0 + Veo 3.0 접근 권한)
- ffmpeg 설치
- Pillow (PIL) 라이브러리
- 음원 파일 (MP3)

### 12시간 퀵스타트 (핵심 원칙 3가지 적용)

```bash
# Phase 0: 프로젝트 초기화 (10분)
bash scripts/init-video-project.sh MMDD_프로젝트명

# Phase 1: 레퍼런스 + ⚡음원 확보⚡ (2시간)
yt-dlp -o "downloads/{ID}.mp4" "URL"          # 레퍼런스 다운로드
# Suno AI로 음원 생성 → output/music/에 저장  ← 반드시 이 단계에서!

# Phase 2: 가사 + 런닝타임 검증 (1시간)
# Claude에게: "이 음원(5:09)에 맞는 가사를 작성하고 런닝타임 검증해줘"

# Phase 3: 음원 분석 → 스토리보드 (2시간)
# Claude에게: "음원 에너지 분석 후 씬 분할하고 스토리보드 만들어줘"

# Phase 3.5: ⚡1장 도구 비교⚡ (30분)
# Claude에게: "이 이미지 1장으로 Veo/fal.ai 비교 테스트해줘"

# Phase 4: 이미지 ⚡일괄⚡ 생성 (1시간)
python3 scripts/generate_images.py {프로젝트} --dry-run  # 목록 확인
python3 scripts/generate_images.py {프로젝트}             # 전량 생성

# Phase 5: 영상 ⚡배치⚡ 생성 (3시간)
# 3~4개씩 배치 → 90초 간격 → fal.ai 폴백

# Phase 6~7: 나레이션 + 편집 (2시간)
# Edge TTS + ffmpeg concat + 음원 합성

# Phase 8: 검수 → output/FINAL_{이름}_v1.mp4
```

---

## 프로젝트 파일 구조 (권장)

```
video_creator/                     # 허브 루트
├── hub_server.py                  # 통합 서버 (python3 hub_server.py)
├── _wizard_temp/                  # 위자드 임시 파일
│
├── {프로젝트 슬러그}/              # ← 각 MV 프로젝트
│   ├── project.json               # ★ 허브 등록 (필수)
│   ├── prompts.json               # ★ 영상 프롬프트 (필수)
│   ├── images/                    # 웹툰/픽사 이미지 (최종)
│   │   ├── 01_scene_name.jpg
│   │   └── ...
│   ├── clips/                     # AI 영상 클립 (Veo 생성물)
│   │   ├── 01_scene_name.mp4
│   │   └── ...
│   ├── downloads/                 # 레퍼런스 영상
│   ├── storyboard/
│   │   └── mv_storyboard_v{N}.md
│   ├── lyrics.md                  # 최종 가사
│   ├── suno_package.md            # Suno 입력 패키지
│   ├── music.mp3                  # 음원
│   └── output/
│       └── mv_final.mp4           # 최종 결과물
│
├── 0320_왕사님/                   # 프로젝트 예시 1
├── journey-to-cross/              # 프로젝트 예시 2
└── ...
```

**허브 자동 인식 조건**: 폴더 안에 `project.json`이 있으면 자동 등록.
**웹 접근**: `http://localhost:8787/{slug}/` 에서 클립 생성기 UI 사용.

---

## 체크리스트

### Phase 0: 프로젝트 초기화
- [ ] 프로젝트 폴더 생성 (`video_creator/{slug}/`)
- [ ] `project.json` 작성 (name, slug, color 등)
- [ ] `prompts.json` 생성 (빈 `{}`)
- [ ] 하위 폴더 생성 (images, clips, downloads, storyboard, output)
- [ ] 허브 서버 재시작 → 카드 확인

### Phase 1: 레퍼런스
- [ ] 레퍼런스 영상 다운로드
- [ ] 씬 프레임 추출 (scene detection 0.3)
- [ ] 콘택트 시트 생성
- [ ] 핵심 감정/명장면 분석

### Phase 2: 가사 & 음원
- [ ] 서사 구조 분석
- [ ] 핵심 모티프 3~4개 설정
- [ ] 가사 작성 (섹션 구분)
- [ ] **런닝타임 분량 검증** (행 수 × 행당 시간 ≤ 목표 시간)
- [ ] Suno AI 스타일 프롬프트 작성
- [ ] 음원 생성 & 최종 선정

### Phase 3: 스토리보드
- [ ] 음원 에너지 프로파일 분석
- [ ] 씬 분할 (10~15씬)
- [ ] 씬별 상세 명세서 작성
- [ ] 컬러 팔레트 정의
- [ ] 캐릭터 디자인 잠금

### Phase 4: 이미지
- [ ] 스타일 프리픽스 확정
- [ ] 캐릭터 디자인 잠금 프롬프트
- [ ] 이미지 생성 (씬당 4~8장)
- [ ] 중복 제거 (MD5 해시)
- [ ] 내용 기반 파일명 리네이밍
- [ ] 씬별 폴더 분류

### Phase 5: 영상 생성
- [ ] 영/한 이중 프롬프트 작성
- [ ] 대시보드 서버 구축
- [ ] 클립 생성 (Veo 3.0)
- [ ] 품질 검수 & 리테이크
- [ ] 비용 추적

### Phase 6: 최종 편집
- [ ] 클립 시퀀싱 (타임라인)
- [ ] 음원 합성
- [ ] 가사 자막 추가
- [ ] 컬러 그레이딩
- [ ] 최종 렌더링

---

## 사용 가능한 모델 & 비용 (2026-04 기준)

### 이미지 생성

| 모델 | 단가 | 특성 | 검증 상태 |
|------|------|------|----------|
| Google Imagen 4.0 | ~$0.04/장 | 안정적, 고품질, 캐릭터 일관성 우수 | 실전 검증 (왕사님, 파리대사관, demo) |
| Google Imagen 4.0 Ultra | ~$0.08/장 | 최고 품질, 복잡한 장면에 적합 | 부분 검증 |
| FLUX.2 Pro (fal.ai) | ~$0.10/장 | 고성능, 다양한 스타일, 정책 유연 | 실전 검증 (Ragnarok2) |

### 영상 생성

| 모델 | 단가 | 클립 길이 | 특성 | 검증 상태 |
|------|------|----------|------|----------|
| Google Veo 3.0 | ~$0.50/클립 | ~8초 | 시네마틱, 고품질, Rate Limit 주의 | 실전 검증 |
| Google Veo 3.0 Fast | ~$0.25/클립 | ~8초 | 빠른 생성, 약간 낮은 품질 | 실전 검증 |
| Google Veo 2.0 | ~$0.25/클립 | ~8초 | 폴백용, 안정적 | 검증됨 |
| fal.ai Wan 2.2 Turbo | ~$0.10/클립 | ~5초 | 저비용, 이미지→영상 전문, 480~720p | 실전 검증 (Ragnarok2) |

### 음원/음성

| 도구 | 비용 | 용도 |
|------|------|------|
| Suno AI | 무료~$10/곡 | AI 음악 생성 |
| Edge TTS | 무료 | 한국어 나레이션 (ko-KR-HyunsuNeural, ko-KR-SunHiNeural) |

### 비용 예측 (66클립 MV 기준)

| 항목 | 단가 | 수량 | 합계 |
|------|------|------|------|
| Suno AI 음원 | 무료~$10 | 1곡 | ~$10 |
| Imagen 4.0 이미지 | ~$0.04 | 80장 | ~$3 |
| Veo 3.0 영상 클립 | ~$0.50 | 66개 | ~$33 |
| **총 예상 비용 (Veo)** | | | **~$46** |
| **Wan 2.2 대안** | ~$0.10 | 66개 | **~$20** |

---

## 트러블슈팅

### Veo 3.0 Rate Limit (429 에러)
- 원인: 일일/분당 API 할당량 초과
- 해결: 자동 폴백 로직 (v3.0 → v3.0-fast → v2.0)
- 대기: 30초 → 60초 → 120초 지수 백오프

### ffmpeg scene detection 에러
- zsh에서 `2>` 리다이렉트 충돌 → `/bin/bash -c '...'`로 감쌈
- 콤마 이스케이프: `select=gt(scene\,0.3)` (백슬래시 필수)

### 이미지 중복 생성
- Claude 대화형 생성 시 같은 이미지가 여러 번 다운로드됨
- MD5 해시 기반 중복 탐지 → `_duplicates/` 폴더로 분리

### 캐릭터 일관성 깨짐
- 모든 프롬프트에 CHARACTER_LOCK 프리픽스 강제 적용
- 타임라인별 character_lock 정의 사용 (Phase 4-A 참조)
- 앵커 이미지 선정 → 나머지와 비교 → 불일치분만 재생성
- 재생성 시 반드시 `images_v1_backup/` 백업 (Phase 4-A2 참조)
- 레퍼런스 이미지를 함께 입력 (image-to-image 모드)

### 가사 분량이 런닝타임 초과
- 원인: 72 BPM 발라드에서 한 줄에 5초 소요되는데, 감으로 쓰면 44행까지 늘어남
- 해결: 작성 후 반드시 **2-B2 검증 공식** 적용. 행 수 × 행당 시간 + 인트로/간주/여운 = 목표 시간
- 팁: 서사적 가사(김이나 스타일)는 절제가 핵심. Verse 4행, Chorus 4행이 적정. 8행은 무조건 초과

### 영상 품질 불일치
- 프롬프트에 `"preserving the exact composition and art style"` 추가
- 움직임 최소화: `"very slow, subtle movement"` 지시
- 리테이크 시 다른 모델(Kling, Grok 등) 병행 테스트

---

## 실전 비용 실적 (프로젝트별)

<!-- [MODULE: cost-tracker] 비용 추적 데이터 집중 구간
     추출 예정: /cost-tracker 모듈 (프로젝트별 비용 집계, 예산 경고, 대시보드 HTML)
     현재: 이 SKILL.md 내 정적 테이블로 기록 -->

실제 제작에서 발생한 비용 데이터. 예산 계획 시 참고.

### 왕사님 MV (13씬, 41클립, 5분 9초)
| 항목 | 수량 | 비용 | 비고 |
|------|------|------|------|
| Imagen 4.0 이미지 | 69장 | $2.76 | 3회 분할 (57+9+3) |
| 테스트 이미지 | ~15장 | $0.60 | 스타일 탐색 |
| Kling AI 영상 (폐기) | 5클립 | $2.50 | 도구 비교 실패 → 폐기 |
| Veo 3.0 영상 | 42클립 | $21.00 | 메인 생성 |
| **실제 합계** | | **~$34** | 중복/테스트 포함 |
| **최적화 예상** | | **~$20** | 원칙 3가지 적용 시 |

**교훈:** Kling 2시간 낭비($2.50) + 이미지 3회 분할 → 원칙 적용 시 40% 절감

### 파리대사관 서연 (10씬, 18클립, 진행 중)
| 항목 | 수량 | 비용 | 비고 |
|------|------|------|------|
| Imagen 4.0 v1 | 18장 | $0.76 | 초기 생성 |
| Imagen 4.0 v2 | 9장 | $0.36 | 캐릭터 보정 재생성 |
| 테스트 | 4장 | $0.16 | character_lock 검증 |
| **이미지 합계** | | **$1.28** | |
| Veo 3.0 (예정) | 18클립 | $9.00 | 0.50/클립 |
| **총 예상** | | **~$10** | |

**교훈:** character_lock 타임라인 정의로 재생성 9장만 (50%) → 사전 정의 시 0%

---

## 허브 서버 — 멀티 프로젝트 관리

### 개요

`hub_server_template.py`는 여러 MV 프로젝트를 하나의 웹 대시보드로 통합 관리하는 서버다.
하위 폴더의 `project.json`을 자동 탐색하여 프로젝트를 등록한다.

### 설치 & 실행

```bash
# 1. 허브 서버 파일 복사 (스킬 resources에서)
cp resources/hub_server_template.py /path/to/your/video_creator/hub_server.py
cp resources/init_project.sh /path/to/your/video_creator/

# 2. API 키 설정
export GOOGLE_API_KEY='your-google-api-key'

# 3. 서버 실행
cd /path/to/your/video_creator
python3 hub_server.py
# → http://localhost:8787
```

### 허브 페이지 구조

| 페이지 | URL | 기능 |
|--------|-----|------|
| **허브 홈** | `/` | 전체 프로젝트 카드 + 통계 (이미지 수, 클립 수, 비용) |
| **프로젝트 상세** | `/{slug}/` | 이미지 그리드 + 클립 생성 + 실시간 상태 모니터링 |
| **새 MV 위자드** | `/new` | YouTube URL → 프레임 추출 → 웹툰 변환 → 프로젝트 생성 |

### 프로젝트 상세 페이지 (스토리보드 뷰)

각 프로젝트 페이지에서 할 수 있는 것:
- **이미지 카드 그리드**: 프롬프트와 함께 이미지 미리보기 (10개/페이지)
- **실시간 상태 뱃지**: idle → queued → generating → done / error
- **선택 생성**: 원하는 이미지만 선택 → "생성 시작" → Veo API 호출
- **미완성 전체생성**: 클립이 없는 이미지 모두 한번에 생성
- **비용 대시보드**: 완성 수, 전체 수, 누적 비용, 잔여 클립
- **폴링**: 생성 중 3초 간격, 대기 시 15초 간격 자동 업데이트

### API 엔드포인트

| Method | Path | 설명 |
|--------|------|------|
| GET | `/` | 허브 홈 HTML |
| GET | `/api/projects` | 전체 프로젝트 JSON |
| GET | `/{slug}/` | 프로젝트 상세 HTML |
| GET | `/{slug}/api/images` | 이미지 메타데이터 JSON |
| GET | `/{slug}/api/status` | 생성 상태 + 비용 JSON |
| POST | `/{slug}/api/generate` | 선택 이미지로 클립 생성 시작 |
| GET | `/{slug}/img/{name}` | 원본 이미지 서빙 |
| GET | `/{slug}/clip/{name}` | 생성된 클립 서빙 |

### 위자드 API (새 MV 만들기)

| Method | Path | 설명 |
|--------|------|------|
| POST | `/wizard/api/download` | YouTube 영상 다운로드 |
| POST | `/wizard/api/extract` | 프레임 추출 (씬 감지 / 일정 간격) |
| POST | `/wizard/api/convert` | Imagen 4.0으로 웹툰 스타일 변환 |
| POST | `/wizard/api/convert-status` | 변환 진행 상태 |
| POST | `/wizard/api/create-project` | 프로젝트 폴더 생성 + 허브 등록 |

### 환경 변수

| 변수 | 기본값 | 설명 |
|------|--------|------|
| `GOOGLE_API_KEY` | (없음) | Google AI API 키 (Veo, Imagen) |
| `MV_PORT` | `8787` | 서버 포트 |

### 의존성

```bash
pip install google-genai Pillow
# 시스템 도구
brew install ffmpeg yt-dlp  # macOS
```

### 주의사항

- **반드시 hub_server.py만 실행** — 개별 프로젝트의 server.py 실행 금지 (포트 충돌)
- 서버 재시작 시 프로젝트 자동 재탐색
- `_wizard_temp/` 폴더는 위자드 임시 파일 — 정리 가능

---

## 리소스 파일 목록

| 파일 | 용도 |
|------|------|
| `resources/hub_server_template.py` | 멀티 프로젝트 허브 서버 (위자드 포함) |
| `resources/server_template.py` | 단일 프로젝트 서버 (레거시, 허브 사용 권장) |
| `resources/init_project.sh` | 프로젝트 초기화 스크립트 |
| `resources/project_template.json` | project.json 템플릿 |
| `resources/prompts_template.json` | prompts.json 템플릿 |

---

## gstack 통합 포인트 (v2 신규)

이 섹션은 gstack 파이프라인과의 연결 지점을 정의한다.

### /qa 연결 (Phase 4 완료 후)

이미지 생성 완료 시 다음 체크리스트를 실행한다:

```markdown
## MV 이미지 QA 체크리스트

### 캐릭터 일관성
- [ ] 앵커 이미지 선정 완료
- [ ] 모든 씬에서 CHARACTER_LOCK 프리픽스 적용 확인
- [ ] 5장 이상 비교 — 헤어/의상/표정 일치 여부

### 스타일 일관성
- [ ] 전체 이미지에 동일 스타일 프리픽스 적용
- [ ] 해상도/종횡비 통일 (16:9 or 9:16)
- [ ] 저품질/아티팩트 이미지 식별 → 재생성 목록 작성

### 스토리보드 매핑
- [ ] 씬 번호 ↔ 이미지 파일명 대응 확인
- [ ] 누락 씬 없음 확인
- [ ] prompts.json에 모든 이미지 엔트리 존재

### 산출물
- `images/` 폴더 이미지 수: {N}장
- 재생성 필요: {N}장
- QA 통과: YES / NO
```

### /ship 연결 (Phase 7 완료 후)

최종 MV 완성 시 배포/공유 워크플로우:

```markdown
## MV 배포 체크리스트

### 최종 파일 확인
- [ ] `output/final_mv_v{N}.mp4` 존재
- [ ] 해상도 확인: 1920x1080 (16:9) 또는 1080x1920 (9:16 쇼츠)
- [ ] 런닝타임 확인: 목표 시간 ±5초 이내
- [ ] 음원 싱크 확인: 클립 경계에서 박자 맞음

### 메타데이터
- [ ] 버전 태그: `git tag mv-v{N}` 또는 파일명에 버전 포함
- [ ] PRODUCTION-LOG.md에 배포 기록 추가
- [ ] 비용 합계 기록: ${총액}

### 배포 채널
- [ ] YouTube 업로드 (설명, 태그, 썸네일)
- [ ] Instagram Reels / TikTok (쇼츠 버전)
- [ ] 클라이언트 전달 (WeTransfer / Google Drive 링크)

### work-logger 실행
- [ ] 작업 로그 기록: 프로젝트의 `logs/` 또는 사용자 지정 위치
```

### 공유 모듈 분리 로드맵

현재 이 스킬 내 인라인으로 구현된 코드를 단계적으로 공유 모듈로 분리한다:

| 우선순위 | 모듈 | 분리 조건 | 예상 위치 |
|---------|------|---------|---------|
| 1 | `/ai-image` | cinematic-sites도 Flux를 쓰기 시작하면 | `skills/shared/ai-image/` |
| 2 | `/ai-video` | cinematic-sites도 Wan/Veo를 병행하면 | `skills/shared/ai-video/` |
| 3 | `/ffmpeg-tools` | 3번째 스킬이 ffmpeg를 쓰기 시작하면 | `skills/shared/ffmpeg-tools/` |
| 4 | `/cost-tracker` | 월 비용 $50 초과 시 관리 필요성 발생 | `skills/shared/cost-tracker/` |

> **원칙**: 두 군데에서 쓰이면 분리를 검토, 세 군데면 반드시 분리한다.

---
> Source: [ninestonelee/ai-mv-pipeline](https://github.com/ninestonelee/ai-mv-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
