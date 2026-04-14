## mislgom-image-auto

> AI 기반 스토리보드 생성 웹 애플리케이션.

# 미슬곰(Miseulgom) 프로젝트 가이드

## 프로젝트 개요
AI 기반 스토리보드 생성 웹 애플리케이션.
사용자가 대본을 입력하면 AI가 장면을 분할하고 각 장면의 이미지를 Gemini + Whisk로 생성한다.
Google Cloud Run에 배포, Node.js 서버.

## 기술 스택
- 프론트엔드: Vanilla JS, CSS (프레임워크 없음)
- 백엔드: Node.js (Express)
- AI: Google Gemini API (Vertex AI), Whisk (Chrome 확장 프로그램 연동)
- 저장소: IndexedDB (이미지, 프로젝트 상태), localStorage (경량 상태)
- 배포: Google Cloud Run

## 핵심 파일 구조
- server.js: API 엔드포인트, Gemini 호출, 프롬프트 빌더
- app.js: 앱 초기화, 프로젝트 저장/로드, 전역 이벤트
- scriptManager.js: 대본 관리, 분석 결과(analysisResult) 관리
- characterManager.js: 캐릭터 생성/수정/라이브러리
- storyboardManager.js: 스토리보드 상태, 이미지 생성, 장면 프롬프트
- storyboardModal.js: 스토리보드 모달 UI, 카드 표시/상호작용
- grokPromptGenerator.js: GROK 프롬프트 생성 (메인 화면 버튼용)
- api.js: 서버 API 호출, 타임아웃, 재시도
- referenceImageStore.js: 참조 이미지 저장소
- projectStateStore.js: IndexedDB 프로젝트 상태 저장

## 데이터 저장소 맵

### IndexedDB
- miseulgom_image_store / images: 장면/캐릭터 이미지 base64
- miseulgom_image_store / projects: 프로젝트 전체 상태
- miseulgom_image_store / character_library: 캐릭터 라이브러리
- miseulgom_reference_store / reference_images: 참조 이미지 (의상, 머리 등)
- miseulgom_style_store / style_images: 스타일 참조 이미지
- miseulgom_vrew_cache / vrew_cache: 브루 내보내기 캐시

### localStorage
- miseulgom_scene_status: 장면별 성공/실패 상태
- miseulgom_brief_partial: brief 생성 중간 저장
- miseulgom_custom_styles: 사용자 커스텀 스타일
- miseulgom_reference_images: 참조 이미지 메타데이터
- miseulgom_reference_roles: 참조 이미지 역할
- miseulgom_reference_matches: 참조 이미지 매칭
- miseulgom_reference_custom_groups: 참조 이미지 커스텀 그룹
- mislgom_current_step: 현재 진행 단계
- project:latest:id: 마지막 프로젝트 ID

## 수정 시 필수 규칙

### 1. 저장 안전성
- saveProject()는 loadProject() 완료 후에만 호출 가능 (_isRestoringProject 플래그 확인)
- 장면 이미지 생성 성공 시 즉시 IndexedDB + localStorage에 저장
- localStorage.clear() 사용 금지 → clearProjectStorage()로 프로젝트 키만 삭제
- 참조 이미지 키(miseulgom_reference_*)는 새 프로젝트/초기화에서 보존

### 2. IndexedDB 비동기 처리
- IndexedDB 저장은 반드시 await로 완료 보장
- fire-and-forget 방식 금지
- 이미지 저장 시 imageBase64가 null이면 imageUrl에서 base64 추출 후 저장

### 3. Gemini API 호출
- 모든 Gemini API 호출에 _retryApiCall() 적용 (apiFn, sceneId — 2개 파라미터, MAX_RETRY=2 하드코딩, RETRY_DELAY=10000ms)
- 429 Rate Limit 대비 필수
- catch에서 에러를 삼키지 말고 원본 에러 메시지 포함하여 throw
- en(영문 프롬프트)이 빈 문자열이면 에러로 처리
- /api/generate-scene-prompt 응답 형식: { en, ko, negative } — grokPrompt 없음
- ko는 Gemini가 직접 한국어 번역 반환 (ENGLISH:/KOREAN: 레이블 포맷)

### 4. 이미지 생성 시 grokPrompt 금지
- 장면 이미지 생성 경로에서 grokPrompt 생성/저장/파싱 코드 사용 금지
- grokPrompt는 메인 화면 GROK 버튼(grokPromptGenerator.js)에서만 생성
- 서버 프롬프트 지시문에 GROK_VIDEO 관련 규칙 포함 금지

### 5. 캐릭터 재설계
- mode='redesign' + type='character': 외모 보존 지시문 사용
- mode='redesign' + type='scene': 기존 장면 지시문 유지
- type 미지정 시 기존 로직(분기 B/C) 유지

### 6. UI 일관성
- 추가된 카드는 기존 카드와 100% 동일한 데이터 구조와 기능
- 모든 카드는 이미지 클릭 → 수정 모달, 삭제 버튼, PICK 체크박스 동일 동작
- 확정 후 메인 화면 버튼(이미지 다운로드, GROK, 브루)에서 추가 카드 포함

## 수정 작업 진행 방법
- docs/tasks/ 폴더의 md 파일을 Phase 순서대로 진행
- 각 파일의 수정 지시를 정확히 따르고 임의 변경 금지
- 수정 완료 후 해당 md 파일의 체크리스트를 업데이트
- Phase 완료 후 사용자 테스트 → 통과 시 다음 Phase 진행

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mislgom487-hash)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mislgom487-hash)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
