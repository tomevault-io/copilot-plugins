## prohibitions

> - ❌ Firebase 데이터를 로컬스토리지에 중복 저장

# 절대 금지사항

## 데이터 관리 금지
- ❌ Firebase 데이터를 로컬스토리지에 중복 저장
- ❌ 큰 이미지 파일을 압축 없이 업로드
- ❌ 민감한 정보를 코드에 하드코딩
- ❌ API 키를 소스코드에 직접 포함

## 코드 구조 금지
- ❌ 백엔드와 프론트엔드 코드 혼재
- ❌ 이미 있는 기능의 중복 구현
- ❌ 반응형을 고려하지 않은 고정 레이아웃
- ❌ TypeScript 타입 정의 없이 개발

## 파일 관리 금지
- ❌ `.env` 파일을 Git에 커밋
- ❌ 백업 폴더나 압축 파일 업로드
- ❌ 불필요한 주석이나 설명 코드
- ❌ 테스트 파일을 프로덕션에 포함

## 성능 관련 금지
- ❌ 큰 번들 크기의 라이브러리 사용
- ❌ 불필요한 리렌더링 발생
- ❌ 네트워크 요청 최적화 없이 구현
- ❌ 이미지 최적화 없이 사용

## 보안 관련 금지
- ❌ 사용자 입력 데이터 검증 없이 사용
- ❌ XSS 공격 방어 코드 누락
- ❌ CORS 설정 없이 API 호출
- ❌ 인증/인가 로직 누락
description:
globs:
alwaysApply: false
---

---
> Source: [jang-seung-hee/rental-news](https://github.com/jang-seung-hee/rental-news) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
