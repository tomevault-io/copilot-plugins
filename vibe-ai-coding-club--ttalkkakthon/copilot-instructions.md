## ttalkkakthon

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ttalkkakthon(Hackathon with AI) project.

- **Framework**: Next.js 16 (App Router)
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS 4
- **ORM**: Prisma 7 + PostgreSQL (Neon 어댑터)
- **Database**: Vercel Postgres (Neon 기반)
- **Package Manager**: pnpm
- **Deployment**: Vercel

## Build & Development Commands

```bash
pnpm install       # 의존성 설치
pnpm dev           # 개발 서버 (localhost:3000)
pnpm build         # 프로덕션 빌드
pnpm start         # 프로덕션 서버
pnpm lint          # ESLint 실행
pnpm prisma generate   # Prisma 클라이언트 생성
pnpm prisma migrate dev # DB 마이그레이션 (개발)
```

## Architecture

```
ttalkkakthon/
├── app/                    # Next.js App Router
│   ├── globals.css         # 글로벌 스타일 (Tailwind CSS 4)
│   ├── layout.tsx          # Root Layout
│   └── page.tsx            # 홈 페이지
├── docs/                   # 프로젝트 문서
│   └── YYYYMMDD-*.md       # 설계서/기획서
├── logs/                   # 개발 일지
│   └── YYYYMMDD-*.md       # 일자별 커밋 내용 정리
├── generated/prisma/       # Prisma 생성 클라이언트 (gitignored)
├── lib/                    # 공유 유틸리티
│   ├── prisma.ts           # Prisma 싱글톤 클라이언트
│   ├── utils.ts            # cn() 등 헬퍼 함수
│   └── validations/        # Zod 등 유효성 검사 스키마
├── prisma/
│   └── schema.prisma       # Prisma 스키마
├── prisma.config.ts        # Prisma 설정
├── public/                 # 정적 파일
├── .env                    # 환경 변수 (gitignored)
└── .env.example            # 환경 변수 템플릿
```

### 주요 패턴

- **Prisma Client**: `lib/prisma.ts`의 싱글톤 인스턴스 사용 (Neon 서버리스 어댑터, dev hot-reload 대응)
- **환경 변수**: Vercel Postgres 연결 시 `POSTGRES_URL`, `POSTGRES_PRISMA_URL`, `POSTGRES_URL_NON_POOLING` 자동 주입
- **클래스 병합**: `cn()` 함수 사용 (`lib/utils.ts`) — clsx + tailwind-merge
- **Server Components**: 기본값, 필요시 `"use client"` 명시
- **경로 별칭**: `@/*` → 프로젝트 루트

## 문서 관리

### 설계서/기획서 (`docs/`)

- 파일명: `YYYYMMDD-{제목}.md` (예: `20260224-team-registration.md`)
- 프로젝트 설계, 기획, 아키텍처 결정 등을 기록
- 한 주제당 하나의 파일로 관리

### 참가 신청 컴포넌트 명세 (`docs/feature/components/register/`)

- 참가 신청(`/register`) 관련 컴포넌트 명세를 컴포넌트별 개별 파일로 관리
- 컴포넌트의 Props/스타일/동작이 변경되면 **같은 작업 턴 내에** 해당 컴포넌트 문서를 반드시 갱신
- 신규 컴포넌트 추가 시 명세 파일 생성 + `register/README.md` 인덱스 등록
- 랜딩 페이지 컴포넌트(`docs/feature/components/`)와 분리하여 `register/` 하위 디렉토리에서 관리

### 개발 일지 (`logs/`)

- 파일명: `YYYYMMDD-{제목}.md` (예: `20260224-initial-setup.md`)
- 하루 작업 종료 시 해당 일자의 커밋 내용을 정리
- 커밋 타입별로 그룹화하여 작성
- 기존 파일이 있으면 내용 추가

## 디스코드 심사 규칙

### API 호출 규칙

- API 호출 시 `curl -s` 만 사용한다. `python3 -c`로 JSON을 파싱하지 않는다.
- curl 응답은 직접 읽고 필요한 값을 파악한다.

### 심사 명령어

디스코드에서 `@tkt`, `@ttalkkakthon`, `@삿` 중 아무 호출명으로 아래 명령을 사용할 수 있다.

### 다중 프로젝트 처리

팀당 프로젝트가 2개 이상일 수 있으므로, 팀 이름으로 조회 시 프로젝트 수를 먼저 확인한다.

- API 응답에 `projectCount`가 있으면 → 프로젝트 목록을 디스코드에 안내하고 선택을 요청
- API 응답에 `data`가 있으면 → 프로젝트가 1개이므로 바로 진행
- `?team={팀이름}&project={프로젝트순번}`으로 특정 프로젝트 지정 가능

### 심사 요청

`@ttalkkakthon 심사 {순번}` 또는 `@ttalkkakthon 심사 {팀이름}` 또는 `@ttalkkakthon 심사 {팀이름} {프로젝트순번}`

1. `GET http://localhost:3000/api/evaluate?index={순번}` 또는 `?team={팀이름}` (프로젝트순번이 있으면 `&project={프로젝트순번}`)으로 프로젝트 데이터 조회
2. 프로젝트가 여러 개면(`projectCount` 존재) 목록을 디스코드에 안내하고 선택 요청
3. 단일 프로젝트면 응답의 `evaluationPrompt`(평가 프롬프트)를 기반으로 심사 수행
4. 프로젝트의 GitHub/데모/영상 URL이 있으면 접근하여 실제 내용 확인
5. 평가 결과를 `POST http://localhost:3000/api/evaluate`로 DB 저장 (`projectId` + `promptFeedback`)
6. 디스코드에 심사 결과 회신

### 심사 결과 조회

`@ttalkkakthon 심사결과 {순번}` 또는 `@ttalkkakthon 심사결과 {팀이름}` 또는 `@ttalkkakthon 심사결과 {팀이름} {프로젝트순번}`

1. `GET http://localhost:3000/api/evaluate?index={순번}` 또는 `?team={팀이름}` (프로젝트순번이 있으면 `&project={프로젝트순번}`)으로 조회
2. 프로젝트가 여러 개면 목록을 디스코드에 안내하고 선택 요청
3. 단일 프로젝트면 `promptFeedback` 값을 디스코드에 회신 (없으면 "아직 심사 결과 없음" 안내)

### 냥심사 요청

`@ttalkkakthon 냥심사 {순번}` 또는 `@ttalkkakthon 냥심사 {팀이름}` 또는 `@ttalkkakthon 냥심사 {팀이름} {프로젝트순번}`

고양이 나비의 예능 보너스 심사(15점)만 수행한다. 기본 심사(promptFeedback)와는 별개로 독립 운영된다.

1. `GET http://localhost:3000/api/evaluate?index={순번}` 또는 `?team={팀이름}` (프로젝트순번이 있으면 `&project={프로젝트순번}`)으로 프로젝트 데이터 조회
2. 프로젝트가 여러 개면(`projectCount` 존재) 목록을 디스코드에 안내하고 선택 요청
3. 단일 프로젝트면 응답의 `fullEvaluationPrompt`(고양이 심사 프롬프트)를 기반으로 심사 수행
4. 프로젝트의 GitHub/데모/영상 URL이 있으면 접근하여 실제 내용 확인
5. 평가 결과를 `POST http://localhost:3000/api/evaluate`로 DB 저장 (`projectId` + `catFeedback`)
6. 디스코드에 심사 결과 회신 (고양이 나비의 예능 심사 결과)

### 냥심사 결과 조회

`@ttalkkakthon 냥심사결과 {순번}` 또는 `@ttalkkakthon 냥심사결과 {팀이름}` 또는 `@ttalkkakthon 냥심사결과 {팀이름} {프로젝트순번}`

1. `GET http://localhost:3000/api/evaluate?index={순번}` 또는 `?team={팀이름}` (프로젝트순번이 있으면 `&project={프로젝트순번}`)으로 조회
2. 프로젝트가 여러 개면 목록을 디스코드에 안내하고 선택 요청
3. 단일 프로젝트면 `catFeedback` 값을 디스코드에 회신 (없으면 "아직 냥심사 결과 없음" 안내)

### 프로젝트 분석

`@ttalkkakthon 분석 {순번}` 또는 `@ttalkkakthon 분석 {팀이름}` 또는 `@ttalkkakthon 분석 {팀이름} {프로젝트순번}`

공개 GitHub 레포를 등록한 프로젝트에 한해, `gh` CLI로 소스 코드에 직접 접근하여 분석을 수행한다.

1. `GET http://localhost:3000/api/evaluate?index={순번}` 또는 `?team={팀이름}` (프로젝트순번이 있으면 `&project={프로젝트순번}`)으로 프로젝트 데이터 조회
2. 프로젝트가 여러 개면 목록을 디스코드에 안내하고 선택 요청
3. 단일 프로젝트면 `githubUrl`이 공개 레포인지 확인
4. `gh api`로 레포 구조, 주요 파일, 코드 내용에 직접 접근
5. 기술 스택, 프로젝트 구조, 코드 품질, 주요 기능 등을 분석
6. 분석 결과를 디스코드에 회신

> **제한사항**: 비공개(private) 레포는 접근 불가. 공개 레포 URL이 등록되지 않은 프로젝트는 분석할 수 없음.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vibe-ai-coding-club) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
