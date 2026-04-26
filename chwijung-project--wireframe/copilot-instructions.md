## wireframe

> AI-X 교육과정의 통합 운영 관리 시스템 - 운영진, 코치, 멘토를 위한 교육과정 관리 플랫폼

# AI-X 통합 운영 시스템 아키텍처 가이드

## 프로젝트 개요
AI-X 교육과정의 통합 운영 관리 시스템 - 운영진, 코치, 멘토를 위한 교육과정 관리 플랫폼

## 핵심 모듈 구조

### 1. 진입점 및 네비게이션
- [index.html](mdc:index.html) - 메인 대시보드 및 시스템 진입점
- [main.html](mdc:main.html) - 전체 기수 관리 페이지
- [dashboard.html](mdc:dashboard.html) - 교육과정 소개 및 온보딩

### 2. 기수 및 학습자 관리
- [cohort-management.html](mdc:cohort-management.html) - 기수 생성, 진행률 모니터링, 아카이브
- [team-management.html](mdc:team-management.html) - 조 편성, 출석 관리, 수강생 현황
- [student-onboarding.html](mdc:student-onboarding.html) - 수강생 설문 및 팀 편성용 정보 수집

### 3. 과제 및 프로젝트 시스템
- [project-system.html](mdc:project-system.html) - 과제 등록, 제출 현황, 채점 관리
- [meeting-scrum.html](mdc:meeting-scrum.html) - 데일리 체크인, 운영 회의, 진행 점검

### 4. 멘토링 시스템
- [mentoring-system.html](mdc:mentoring-system.html) - 멘토 배정, 일지 관리, 학습자 추적
- [mentoring-log.html](mdc:mentoring-log.html) - 조별 멘토링 기록, 진행 상황 관리
- [smart-mentoring.html](mdc:smart-mentoring.html) - AI 자동 분석 기반 멘토링 추적
- [counseling-schedule.html](mdc:counseling-schedule.html) - 취업 상담, 개별 멘토링 스케줄 관리

### 5. 포트폴리오 및 산출물 관리
- [portfolio-generator.html](mdc:portfolio-generator.html) - 스마트 포트폴리오 자동 생성기
- [file-storage.html](mdc:file-storage.html) - 교육자료, 과제, 제출물 중앙 관리

### 6. 소통 및 지원
- [student-inquiry.html](mdc:student-inquiry.html) - 학습 문의, 기술 지원, 상담 요청 관리

### 7. 시스템 관리
- [admin.html](mdc:admin.html) - 시스템 모니터링, 백업, 사용자 권한 관리

## 기술 스택
- Frontend: HTML5, Tailwind CSS, Vanilla JavaScript
- Charting: Chart.js
- Icons: Heroicons (SVG)
- Responsive Design: Mobile-first approach

## 주요 기능 특징
- 실시간 알림 시스템 (긴급 알림, 과제 마감, 멘토링 일지)
- AI 기반 자동 포트폴리오 생성
- 스마트 멘토링 패턴 분석 및 위험 학습자 감지
- 통합 파일 관리 및 Google Drive 동기화
- 조별 자동 편성 알고리즘

## 사용자 권한 체계
- 운영자(권회은): 전체 시스템 관리 권한
- 코치/멘토: 멘토링 및 과제 관리 권한
- 수강생: 제한된 조회 및 제출 권한

## 데이터 플로우
1. 학습자 정보 수집 (student-onboarding)
2. 자동 조 편성 (team-management)
3. 과제 관리 및 진행 추적 (project-system)
4. 멘토링 기록 및 분석 (mentoring-system, smart-mentoring)
5. 자동 포트폴리오 생성 (portfolio-generator)
6. 성과 분석 및 아카이브 (cohort-management)

## 파일 명명 규칙
- 기능별 모듈: `{기능명}-{서브기능}.html`
- 메인 페이지: `index.html`, `main.html`, `dashboard.html`
- 관리 도구: `admin.html`
- 테스트 파일: `test.html`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chwijung-project) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
