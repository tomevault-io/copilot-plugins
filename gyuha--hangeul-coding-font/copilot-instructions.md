## hangeul-coding-font

> [src/App.tsx](mdc:src/App.tsx)는 애플리케이션의 메인 컴포넌트로 다음 기능들을 관리합니다:

# 컴포넌트 아키텍처

## 메인 앱 구조
[src/App.tsx](mdc:src/App.tsx)는 애플리케이션의 메인 컴포넌트로 다음 기능들을 관리합니다:
- 전체 상태 관리 (폰트 상태, 병합 옵션, 폰트 이름)
- 컴포넌트 간 데이터 흐름 조정
- 사용자 인터페이스 레이아웃 구성

## 핵심 컴포넌트

### FontUploader
**파일**: [src/components/FontUploader.tsx](mdc:src/components/FontUploader.tsx)
**역할**: 폰트 파일 업로드 및 미리보기
**주요 기능**:
- 드래그 앤 드롭 파일 업로드
- 지원 형식 검증 (TTF, OTF, WOFF, WOFF2)
- 업로드된 폰트 정보 표시
- 파일 크기 및 폰트 이름 표시

### MergeOptions
**파일**: [src/components/MergeOptions.tsx](mdc:src/components/MergeOptions.tsx)
**역할**: 폰트 병합 옵션 설정
**주요 기능**:
- 문자 범위별 선택 옵션 (한글, 영문, 숫자, 특수문자)
- 폰트 이름 설정
- 옵션 변경 시 실시간 업데이트

### FontPreview
**파일**: [src/components/FontPreview.tsx](mdc:src/components/FontPreview.tsx)
**역할**: 병합된 폰트 미리보기
**주요 기능**:
- 사용자 정의 텍스트 입력
- 실시간 폰트 미리보기
- 다양한 문자 범위 테스트

## 커스텀 훅

### useFontMerger
**파일**: [src/hooks/useFontMerger.ts](mdc:src/hooks/useFontMerger.ts)
**역할**: 폰트 병합 로직 및 상태 관리
**주요 기능**:
- 폰트 파일 로딩 및 파싱
- 폰트 병합 알고리즘
- 진행률 추적
- 에러 처리 및 성공 메시지
- 병합된 폰트 다운로드

## 상태 관리
- **FontState**: 폰트 로딩, 병합, 에러 상태 관리
- **MergeOptions**: 병합 옵션 설정
- **로컬 상태**: 폰트 이름, 미리보기 텍스트 등

## 데이터 흐름
1. FontUploader → useFontMerger (폰트 로딩)
2. MergeOptions → App (옵션 변경)
3. App → useFontMerger (병합 실행)
4. useFontMerger → FontPreview (미리보기 업데이트)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gyuha)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gyuha)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
