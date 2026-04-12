## wikibot-kakao

> KakaoTalk 챗봇 서버 - LOD(어둠의전설) 게임 거래 시세, 파티 모집, 공지사항, 게시판 검색 기능을 제공하는 Node.js/Express 백엔드.

# CLAUDE.md

## 프로젝트 개요
KakaoTalk 챗봇 서버 - LOD(어둠의전설) 게임 거래 시세, 파티 모집, 공지사항, 게시판 검색 기능을 제공하는 Node.js/Express 백엔드.
RAG 서버(lod-rag-server)와 연동하여 AI 기반 게시판/카페 검색도 지원.
iris-kakao-bot(Python, 카카오톡 메시지 중계)과 연동하여 카카오톡 오픈채팅방에서 동작.

## 기술 스택
- **런타임**: Node.js 20 (Docker: node:20-slim)
- **프레임워크**: Express 4
- **DB**: sql.js (SQLite in-memory, 파일 persist) - nickname.db, notice.db, trade.db, party.db
- **스크래핑**: cheerio, axios
- **테스트**: Jest
- **배포**: Docker + GitHub Actions (self-hosted runner)

## 명령어

```bash
# 개발
npm run dev          # nodemon 개발 서버 (포트 3000)
npm start            # 프로덕션 서버

# 테스트
npx jest                           # 전체 테스트
npx jest src/services/partyService.test.js  # 단일 파일

# 배포 (서버에서 자동 실행)
./deploy.sh --force  # Docker 빌드 + 재시작
```

## 프로젝트 구조

```
src/
├── index.js                  # Express 앱 + 전체 라우트 정의 (메인 엔트리)
├── controllers/
│   ├── webhookController.js  # KakaoTalk 웹훅 (!현자, !공지 등 명령어 처리)
│   └── nicknameController.js # 닉네임 관리 API
├── services/
│   ├── communityService.js   # LOD 게시판 스크래핑
│   ├── noticeService.js      # 공지사항/업데이트 크롤링 + 자동 알림
│   ├── nicknameService.js    # 닉네임 DB (sql.js)
│   ├── searchRagService.js   # RAG 서버 연동 검색
│   ├── tradeService.js       # 거래 시세 수집/조회 (sql.js)
│   ├── partyService.js       # 파티 모집 수집/조회 (sql.js)
│   └── partyService.test.js  # 파티 서비스 테스트
├── middleware/
│   ├── index.js              # 미들웨어 re-export
│   ├── rateLimiter.js        # 유저별 요청 제한
│   └── errorHandler.js       # 글로벌 에러 핸들러
└── utils/
    ├── messageParser.js      # 카카오 메시지 파싱 (!명령어 추출)
    └── responseFormatter.js  # 응답 포맷터

public/
├── index.html               # 위키 검색 웹 UI
├── admin.html                # 파티 DB 관리 페이지
└── party-template.txt        # 파티 메시지 파싱 템플릿

lod-rag-server/               # RAG 서버 (Python FastAPI, 게시판/카페 크롤링 + AI 검색)
iris-kakao-bot/               # Python 카카오톡 봇 (git submodule)
```

## API 엔드포인트 구조

- `POST /ask/community` - 게시판 검색
- `POST /ask/notice` - 공지사항 조회
- `POST /ask/update` - 업데이트 내역
- `GET /ask/check-new` - 새 공지/업데이트 체크
- `POST /api/trade/*` - 거래 시세 API
- `GET/POST /api/party/*` - 파티 모집 API
- `GET/PUT/DELETE /api/party/admin/*` - 파티 관리 API
- `POST /api/nickname/*` - 닉네임 API
- `GET/POST /api/wiki/*` - Notion 위키 API
- `POST /webhook/kakao` - 카카오톡 웹훅
- `GET /health` - 헬스체크

## 배포 구조

- GitHub push → GitHub Actions (self-hosted runner) → `deploy.sh --force`
- Docker: `wikibot-server` 컨테이너, 포트 `8214:3000`
- DB 파일: `$HOME/wikibot-data/` (Docker 볼륨 마운트)
- 환경변수: `$HOME/wikibot-data/.env` (--env-file)
- Cloudflare Tunnel: `party.milddok.cc` → wikibot
- iris-bot: `deploy.sh`가 `iris-kakao-bot/app.py`를 서버에 복사 후 컨테이너 재시작

## 코딩 컨벤션

- 한국어 주석/로그 사용
- CommonJS (`require/module.exports`)
- 서비스 클래스 패턴: `class XxxService { constructor() { ... } initialize() { ... } close() { ... } }`
- DB 서비스는 `sql.js` 사용, `initialize()`에서 DB 로드, `close()`에서 파일 저장
- 에러 응답: `{ success: false, message: '...' }`

## 주의사항

- `src/index.js`에 라우트가 직접 정의되어 있음 (대부분의 API가 여기에 있음)
- DB 파일(*.db)은 `.gitignore`에 포함, Docker 볼륨으로 관리
- `.env`는 서버의 `$HOME/wikibot-data/.env`에만 존재 (repo에 없음)
- `iris-kakao-bot/`은 git submodule
- `lod-rag-server/`는 RAG 서버 (Docker Compose로 함께 배포)
- 공지/업데이트 자동 체크 스케줄러가 `src/index.js`에 내장
- DB 자동 정리: 매일 04:00 (거래 14일, 파티 7일)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/minho83)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/minho83)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
