## s1000d-rag-android

> S1000D 기술 교범 DM(Data Module) 데이터를 기반으로 한 RAG(Retrieval-Augmented Generation) Q&A 앱.

# S1000D-RAG-Android — 네이티브 Android S1000D 기술 교범 Q&A 앱

## 프로젝트 개요
S1000D 기술 교범 DM(Data Module) 데이터를 기반으로 한 RAG(Retrieval-Augmented Generation) Q&A 앱.
갤럭시탭 S11 Ultra에서 완전 온디바이스로 실행 (서버/네트워크 불필요).

## 원본 프로젝트
- **Python RAG 파이프라인**: `D:\S1000D-RAG`
- 인제스천(XML 파싱 → 청킹 → 벡터 인덱싱)은 PC에서 수행
- `export_index.py`로 내보낸 인덱스 파일을 이 앱에서 사용
- 계획 문서: Obsidian `01-Projects/Plans/2026-03-16-galaxy-tab-s9-migration.md`

## 기술 스택
- **언어**: Kotlin
- **UI**: Jetpack Compose + Material 3
- **LLM**: llama.cpp (JNI, C++ NDK 빌드)
- **LLM 모델**: `Konan-LLM-OND-Q4_K_M.gguf` (2.4GB, GGUF Q4_K_M 양자화)
- **STT**: whisper.cpp (JNI, C++ NDK 빌드) — 배치 처리 방식
- **STT 모델**: `ggml-large-v3-turbo-korean-q5_0.bin` (548MB, whisper large-v3-turbo 한국어 fine-tuned, Q5_0 양자화)
- **Embedding**: ONNX Runtime Mobile (`BGE-m3-ko-onnx-int8`, 2.7GB)
- **벡터 검색**: 브루트포스 코사인 유사도 (171 chunks, 별도 DB 불필요)
- **Reranker**: 비활성화 (메모리 절약)
- **타겟 디바이스**: Galaxy Tab S11 Ultra (Snapdragon 8 Elite, 16GB RAM, 14.6" AMOLED)
- **minSdk**: 28 (Android 9)
- **targetSdk**: 35

## 프로젝트 구조
```
D:\S1000D-RAG-Android\
├── app/
│   ├── src/main/
│   │   ├── java/com/s1000d/rag/
│   │   │   ├── MainActivity.kt
│   │   │   ├── S1000DRagApp.kt          # Application 클래스
│   │   │   ├── ui/
│   │   │   │   ├── theme/Theme.kt
│   │   │   │   ├── screens/ChatScreen.kt
│   │   │   │   ├── components/
│   │   │   │   │   ├── MessageBubble.kt
│   │   │   │   │   ├── EvidenceCard.kt
│   │   │   │   │   └── InputBar.kt
│   │   │   │   └── viewmodel/ChatViewModel.kt
│   │   │   ├── rag/
│   │   │   │   ├── RagPipeline.kt        # 메인 RAG 흐름
│   │   │   │   ├── VectorRetriever.kt    # 코사인 유사도 검색
│   │   │   │   ├── ContextBuilder.kt     # 컨텍스트 문자열 구성
│   │   │   │   └── PromptTemplate.kt     # LLM 프롬프트
│   │   │   ├── speech/
│   │   │   │   ├── WhisperModel.kt       # whisper.cpp JNI 래퍼
│   │   │   │   ├── WhisperStt.kt         # 배치 STT (녹음→변환)
│   │   │   │   └── AudioRecorder.kt      # PCM 오디오 캡처
│   │   │   ├── llm/
│   │   │   │   └── LlamaModel.kt         # llama.cpp JNI 래퍼
│   │   │   ├── embedding/
│   │   │   │   └── OnnxEmbedding.kt      # ONNX Runtime 임베딩
│   │   │   └── data/
│   │   │       ├── ChunkIndex.kt         # 인덱스 파일 로더
│   │   │       └── Models.kt             # 데이터 클래스
│   │   ├── cpp/
│   │   │   ├── CMakeLists.txt            # llama.cpp + whisper.cpp NDK 빌드
│   │   │   ├── llama-jni.cpp             # LLM JNI 브릿지
│   │   │   └── whisper-jni.cpp           # STT JNI 브릿지
│   │   ├── res/
│   │   └── AndroidManifest.xml
│   └── build.gradle.kts
├── llama.cpp/                             # git submodule (LLM)
├── whisper.cpp/                           # git submodule (STT)
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
├── CLAUDE.md
└── scripts/
    └── deploy.sh                          # 모델+인덱스 배포 스크립트
```

## 아키텍처

### RAG 파이프라인 흐름 (Kotlin 포팅)
```
사용자 질문
  → OnnxEmbedding.embed(question)        # 쿼리 벡터화 (~2-3초)
  → VectorRetriever.search(queryVec, k)  # 코사인 유사도 (<1ms)
  → threshold 필터링 (score >= 0.3)
  → ContextBuilder.build(chunks)          # 컨텍스트 문자열
  → LlamaModel.generate(prompt)          # LLM 추론 (~15-30초)
  → RagResult(answer, evidences)
```

### 프롬프트 템플릿 (Python 원본과 동일)
```
당신은 S1000D 기술 교범 기반 질의응답 어시스턴트입니다.

## 규칙
1. 반드시 아래 참고 문서(Context)의 내용만 사용하여 답변하세요.
2. 참고 문서에 없는 내용은 절대로 추측하거나 일반 지식으로 답변하지 마세요.
3. 질문이 참고 문서의 범위를 벗어나면 반드시 "제공된 문서에서 해당 정보를 찾을 수 없습니다."라고만 답하세요.
4. 답변 시 근거가 되는 문서의 DMC를 언급하세요.

## Context
[DMC: {dmc} | Type: {dm_type}]
{chunk_text}

---

[DMC: {dmc2} | Type: {dm_type2}]
{chunk_text2}

## Question
{question}

## Answer
```

## 인덱스 파일 포맷

PC의 `D:\S1000D-RAG\export_index.py`로 생성:

### embeddings.bin
- 헤더: `[n_chunks: int32_le, n_dims: int32_le]`
- 바디: `float32_le[n_chunks * n_dims]` (row-major)
- 현재: 171 chunks × 1024 dims = 700KB

### metadata.json
```json
[
  {
    "id": "chunk_id",
    "text": "chunk 원문 텍스트",
    "dmc": "S1000DBIKE-AAA-D00-00-00-00AA-018A-A",
    "chunk_id": "S1000DBIKE-..._0",
    "dm_type": "descriptive",
    "security": "01",
    "applicability": "",
    "structure_path_range": "...",
    "title": "자전거 개요"
  }
]
```

### manifest.json
```json
{
  "version": 1,
  "n_chunks": 171,
  "n_dims": 1024,
  "n_dmcs": 57,
  "collection_name": "s1000d_chunks",
  "embedding_model": "BGE-m3-ko"
}
```

## 모델 파일 (앱 외부 저장소)

앱 설치 후 아래 파일을 `/sdcard/S1000D-RAG/`에 배치:
```
/sdcard/S1000D-RAG/
├── models/
│   ├── Konan-LLM-OND-Q4_K_M.gguf   (2.4GB)
│   ├── ggml-large-v3-turbo-korean-q5_0.bin (548MB, whisper STT)
│   └── BGE-m3-ko-onnx-int8/        (2.7GB)
│       ├── model_int8.onnx
│       ├── model.onnx.data
│       ├── tokenizer.json
│       ├── tokenizer_config.json
│       ├── config.json
│       ├── special_tokens_map.json
│       └── 1_Pooling/
└── index/
    ├── embeddings.bin               (700KB)
    ├── metadata.json                (132KB)
    └── manifest.json                (240B)
```

배포 스크립트: `scripts/deploy.sh` (adb push)

## RAG 파라미터 기본값
- `top_k`: 10 (벡터 검색 후보)
- `relevance_threshold`: 0.3 (최소 스코어)
- `max_context_chars`: 10000
- `max_tokens`: 512 (LLM 응답 길이)
- `n_ctx`: 4096 (LLM 컨텍스트 윈도우, 메모리 절약)
- `temperature`: 0.1
- `repeat_penalty`: 1.15

## 메모리 예산 (Galaxy Tab S11 Ultra, 16GB)
| 컴포넌트 | 예상 |
|----------|------|
| Android OS | ~4GB |
| LLM (Q4_K_M, mmap) | ~2.5GB |
| Embedding (ONNX int8) | ~1.5GB |
| STT (whisper Q5_0) | ~548MB |
| 앱 + 인덱스 | ~200MB |
| **합계** | **~8.7GB (여유 ~7.3GB)** |

## 주요 의존성
```kotlin
// Gradle
implementation("androidx.compose.material3:material3")
implementation("androidx.lifecycle:lifecycle-viewmodel-compose")
implementation("com.microsoft.onnxruntime:onnxruntime-android:1.19.+")
implementation("ai.djl.huggingface:tokenizers:0.30.+")  // HF 토크나이저

// Native (CMake)
// llama.cpp — git submodule, NDK arm64-v8a 빌드
// whisper.cpp — git submodule, NDK arm64-v8a 빌드 (STT)
```

## 구현 순서
1. Android 프로젝트 셋업 (Gradle + Compose + llama.cpp submodule)
2. VectorRetriever (embeddings.bin 로드 + 코사인 유사도)
3. OnnxEmbedding (ONNX Runtime + tokenizer.json)
4. LlamaModel (JNI 래퍼 + CMake NDK)
5. RagPipeline (검색 → 컨텍스트 → LLM)
6. ChatScreen (Compose UI)
7. 배포 + 테스트

## 참고: Python 원본 코드 위치
| Kotlin 클래스 | Python 원본 |
|--------------|------------|
| RagPipeline | `D:\S1000D-RAG\src\rag\pipeline.py` |
| VectorRetriever | `D:\S1000D-RAG\src\rag\retriever.py` |
| ContextBuilder | `D:\S1000D-RAG\src\rag\pipeline.py:_build_context()` |
| PromptTemplate | `D:\S1000D-RAG\src\rag\pipeline.py:_RAG_PROMPT` |
| Models (데이터 클래스) | `D:\S1000D-RAG\src\types\rag.py` |

---
> Source: [Han43seong/S1000D-RAG-Android](https://github.com/Han43seong/S1000D-RAG-Android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
