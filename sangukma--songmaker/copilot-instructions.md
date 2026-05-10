## songmaker

> Song Maker는 6개 Stage로 구성된 파이프라인이다.

# Song Maker — Agent Definitions

## 1. 개요

Song Maker는 6개 Stage로 구성된 파이프라인이다.
각 Stage는 독립 모듈로 동작하며, CLI(`cli.py`)가 오케스트레이션한다.

---

## 2. 모듈(Agent) 역할 정의

### trend/ — 트렌드 조사 에이전트

```
역할: YouTube 인기 음악 데이터를 수집하고 분석하여 사용자에게 보고
입력: region(지역코드), max_results(조회 수)
출력: TrendReport (인기곡 목록 + 장르/키워드 분석)
외부 의존: YouTube Data API v3
```

| 파일 | 책임 |
|------|------|
| `youtube.py` | API 호출, 원시 데이터 수신 |
| `analyzer.py` | 데이터 분석, 포맷팅, 출력 |

### creator/ — 곡 생성 에이전트

```
역할: 사용자 입력을 Suno 프롬프트로 변환하고 곡을 생성
입력: SongRequest (장르, 분위기, 주제, 키워드)
출력: audio.mp3 + lyrics.txt
외부 의존: Suno API (서드파티 래퍼)
```

| 파일 | 책임 |
|------|------|
| `prompt.py` | 사용자 입력 → Suno 프롬프트 변환 |
| `suno.py` | API 호출, 폴링, 다운로드 |

### imager/ — 이미지 생성 에이전트

```
역할: 곡 메타데이터를 기반으로 배경 이미지와 썸네일을 AI 생성
입력: Song (장르, 분위기, 주제)
출력: background.png + thumbnail.png
외부 의존: Gemini Image API (google-genai)
```

| 파일 | 책임 |
|------|------|
| `prompt.py` | 곡 정보 → 이미지 프롬프트 변환 |
| `gemini.py` | API 호출, 이미지 저장 |

### renderer/ — 렌더링 에이전트

```
역할: 오디오 + 이미지를 합쳐 MP4 영상 생성
입력: audio.mp3 + background.png + (선택) subtitles.srt
출력: video.mp4
외부 의존: FFmpeg (로컬 설치)
```

| 파일 | 책임 |
|------|------|
| `ffmpeg.py` | FFmpeg 명령어 조합, 실행, 결과 검증 |

### uploader/ — 업로드 에이전트

```
역할: 영상을 YouTube에 업로드하고 메타데이터 설정
입력: video.mp4 + thumbnail.png + 곡 메타데이터
출력: YouTube URL
외부 의존: YouTube Data API v3 (OAuth 2.0)
```

| 파일 | 책임 |
|------|------|
| `youtube.py` | OAuth 인증, 영상 업로드, 메타데이터 설정 |

---

## 3. 공유 모듈

| 모듈 | 역할 |
|------|------|
| `cli.py` | 오케스트레이터. 명령어 라우팅, Stage 순차 실행, 사용자 입력 수집 |
| `config.py` | 설정 관리. TOML 로드/저장, API 키 제공, 환경변수 오버라이드 |
| `models.py` | 데이터 모델 (Pydantic BaseModel). TrendItem, SongRequest, Song, Project 등 |
| `gates.py` | 검증 게이트. Gate 1~6 검증 함수, Check/GateResult 모델, 실패 분기 처리 |

---

## 4. 에이전트 간 통신

에이전트끼리 직접 호출하지 않는다. `cli.py`가 중재한다.

```
cli.py가 Stage 1 호출 → TrendReport 수신
cli.py가 사용자 입력 수집 → SongRequest 생성
cli.py가 Stage 3 호출(SongRequest) → Song 수신
cli.py가 Stage 4 호출(Song) → 이미지 파일 경로 수신
cli.py가 Stage 5 호출(Song) → 비디오 파일 경로 수신
cli.py가 Stage 6 호출(Song) → YouTube URL 수신
```

데이터는 `models.py`의 Pydantic 모델과 파일시스템(`output/`)을 통해 전달된다.
각 Stage 호출 후 `gates.py`의 검증 함수를 실행하여 Gate를 통과해야 다음 Stage로 진행한다.

---
> Source: [SANGUKMA/songmaker](https://github.com/SANGUKMA/songmaker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
