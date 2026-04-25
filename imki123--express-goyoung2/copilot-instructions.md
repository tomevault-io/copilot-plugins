## express-goyoung2

> Express + TypeScript 기반 백엔드 서버. MongoDB를 DB로, Render를 배포 플랫폼으로 사용합니다.

# CLAUDE.md

## 프로젝트 개요

Express + TypeScript 기반 백엔드 서버. MongoDB를 DB로, Render를 배포 플랫폼으로 사용합니다.

## 주요 도메인

| 경로 | 설명 |
| ---- | ---- |
| `/memo` | 메모 앱 (사용자 인증, 메모 CRUD, 잠금 기능) |
| `/accountBook` | 가계부 앱 (사용자 인증, 시트/타입 관리) |
| `/catbook` | 고양이 관련 기능 |

## 인증 방식

- Google OAuth 로그인 후 JWT 발급 (Bearer 토큰)
- 각 도메인별 미들웨어(`memoMiddleware`, `accountBookMiddleware`)에서 토큰 검증
- 메모는 추가로 잠금 비밀번호(bcrypt) 기능 보유

## 기술 스택

- **Runtime**: Node.js, TypeScript
- **Framework**: Express
- **DB**: MongoDB (Mongoose)
- **Auth**: JWT (`jsonwebtoken`), bcrypt
- **Deploy**: Render (production sleep 방지 로직 포함)
- **Package manager**: pnpm

## 개발 명령어

```bash
pnpm dev        # 개발 서버 (nodemon + ts-node)
pnpm build      # TypeScript 컴파일
pnpm start      # 프로덕션 실행
```

## 환경변수

`.env copy` 파일을 참고해 `.env` 파일을 생성합니다.

## 문서

[docs/index.md](./docs/index.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/imki123) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
