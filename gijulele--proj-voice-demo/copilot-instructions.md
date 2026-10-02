## proj-voice-demo

> > **프로젝트명**: vocalfit — 노래방 가기 전 30초 음역 판정 및 AI 가창 데모 서비스

# CLAUDE.md — vocalfit 프로젝트 개발 지침서 (PRD & System Prompt)

> **프로젝트명**: vocalfit — 노래방 가기 전 30초 음역 판정 및 AI 가창 데모 서비스  
> **개발 환경**: NVIDIA DGX Spark 로컬 환경 (On-Premise)  
> **핵심 목표**: 무반주 30초 가창으로 음역대를 정밀 진단하고, 로컬 환경에서 지연/다운 없이 2단 AI 가창 데모(피치시프트, SVC)를 제공한다.

---

## 1. 프로젝트 개요 및 핵심 솔루션
* **진단곡으로 십팔번(애창곡) 활용**: "노래방 가면 무조건 부르는 곡"을 최적의 진단 샘플로 사용.
* **원곡 멜로디 정답지 교차 검증**: Demucs로 원곡 보컬을 분리하여 멜로디 정답지를 구축하고, 사용자 음정과 대조.
* **가사 기준 구간 매칭 (Lyric Alignment)**: Whisper를 활용해 사용자-원곡 간 단어 단위 정렬을 수행하여 박자 오차 제거.
* **2단 소리 데모 생성**:
  1. 포먼트 보존 피치시프트 (본인 목소리 유지 + 키 이동)
  2. SoulX-Singer 기반 제로샷(Zero-shot) SVC 가창 데모 (AI 생성)

---

## 2. 기술 스택 (Tech Stack)
* **AI Engine**: `Demucs` (음원 분리), `Whisper large-v3-turbo` (STT 및 정렬), `SwiftF0` + `pYIN` + `Autocorrelation` (음정 추출), `SoulX-Singer` (SVC 음성 합성)
* **Backend**: Python 3.10+, FastAPI
* **Frontend**: React, Tailwind CSS, SVG 시각화
* **노래방 번호 API**: `Manana Karaoke API` (`https://api.manana.kr/karaoke/song/{title}.json`) 활용 (별도 API Key 없음)
---

## 3. 🚀 핵심 AI 최적화 지침 (Performance & VRAM Optimization)
NVIDIA DGX Spark 환경에서 GPU 메모리(VRAM) 초과 및 서버 다운을 막기 위해 **반드시 아래 4가지 최적화 원칙**을 코드로 구현해야 합니다.

**1. AI 모델 지연 로딩 및 해제 (Lazy Loading & Unloading)**
* 서버 시작 시 모델을 미리 로드하지 않습니다. 
* 해당 API(분리, 인식, 합성 등)가 호출될 때만 모델을 메모리에 올리고(Lazy Load), 작업이 끝나면 즉시 `del model`과 `torch.cuda.empty_cache()`를 호출하여 VRAM을 비워야 합니다.

**2. 오디오 데이터 조각화 (Chunking)**
* 60~90초 분량의 오디오를 한 번에 모델에 넣지 않습니다. 
* 10초~15초 단위로 오디오를 잘라(Chunking) 순차적으로 처리하여 메모리 피크를 낮춥니다.

**3. 정밀도 최적화 (FP16)**
* Whisper와 SoulX-Singer 추론 시 FP32 대신 **FP16 (반정밀도)** 모드를 강제로 사용하여 속도를 높이고 메모리를 아낍니다.

**4. 순차 처리 큐 (Request Queueing)**
* AI 가창 생성 등 무거운 작업은 동시에 여러 개가 실행되지 않도록, 대기열(Queue)을 만들어 한 번에 하나씩(Single Worker) 처리합니다.

---

## 4. 백엔드 개발 규칙 (Backend Rules)
* **음정 3중 교차검증**: SwiftF0, pYIN, Autocorrelation 3가지 값을 조합하여 "안전 최고음"과 "무너지는 지점"을 산출합니다.
* **박자 밀림 방지**: 전체 시간축을 맞추지 않고, Whisper로 뽑아낸 '가사 단어(Word)'를 기준으로 구간별 앵커 정렬(Anchor Alignment)을 수행합니다.
* **에러 처리**: 사용자의 목소리가 너무 작거나 잡음이 심한 경우를 대비한 꼼꼼한 예외 처리(Exception Handling) 로직을 포함합니다.
* **노래방 번호 조회 기능**: * `httpx` 또는 `requests` 라이브러리를 사용하여 마나나 API에 `GET` 요청을 보냅니다. * 검색 결과 중 `brand == 'tj'`와 `brand == 'kumyoung'` 항목을 추출하여 프론트엔드 결과 화면으로 전달합니다.
### 💡 코드 작성 예시 (Lazy Loading & Chunking 패턴)
```python
class AudioPipeline:
    @staticmethod
    def process_stt_chunked(audio_path: str):
        # 1. Lazy Loading & FP16
        model = load_whisper_model(precision="fp16") 
        try:
            # 2. Chunking (10~15초 단위)
            chunks = split_audio(audio_path, chunk_sec=15)
            results = [model.transcribe(chunk) for chunk in chunks]
            return merge_results(results)
        finally:
            # 3. 강제 메모리 해제
            del model
            torch.cuda.empty_cache()
\`\`\`

---
> Source: [gijulele/proj_voice-demo-](https://github.com/gijulele/proj_voice-demo-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
