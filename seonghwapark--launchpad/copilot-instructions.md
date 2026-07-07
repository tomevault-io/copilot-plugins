## launchpad

> Star Wars 테마의 할일 관리 웹 애플리케이션

# GALACTIC COMMAND

Star Wars 테마의 할일 관리 웹 애플리케이션

## 기술 스택
- **Frontend**: Vanilla HTML/CSS/JS (단일 파일)
- **Backend**: Firebase
  - Authentication (Google 로그인)
  - Realtime Database
- **폰트**: Orbitron, Exo 2

## 파일 구조
```
index.html  - 전체 앱 (HTML + CSS + JS 포함)
```

## 주요 기능
- 캘린더 기반 할일 관리
- Google 로그인 (특정 이메일만 허용)
- 실시간 클라우드 동기화
- 한국 공휴일 표시
- 일일 통계
- 검색 기능
- 알림/우선순위

## Firebase 구조
- Database URL: `galactic-todo-default-rtdb.firebaseio.com`
- 데이터 경로: `users/{userId}/todos`

## 스타일 특징
- 다크 우주 배경
- 홀로그램 스캔라인 효과
- 네온 컬러 (cyan #4fd1ff, yellow #ffe81f)
- 제국 로고 워터마크

## 개발 노트
- 단일 HTML 파일로 구성 (빌드 도구 없음)
- 로컬에서 바로 열어서 테스트 가능
- Firebase SDK는 CDN으로 로드

---
> Source: [SeonghwaPark/launchpad](https://github.com/SeonghwaPark/launchpad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
