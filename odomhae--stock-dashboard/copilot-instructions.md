## stock-dashboard

> 한국 및 미국 주식(개별 주식 및 ETF) 학습을 위한 대시보드 프로젝트.

# 주식 공부 프로젝트

한국 및 미국 주식(개별 주식 및 ETF) 학습을 위한 대시보드 프로젝트.

## 기술 스택

- **HTML5, CSS3, Vanilla JavaScript** (ES modules)
- **Vite 5** — 빌드 도구
- **Supabase JS** — 데이터베이스 클라이언트
- **Chart.js 4** — 차트 렌더링
- **배포** — Vercel

## 파일 구조

```
dashboard-project/
├── index.html
├── styles/
│   └── main.css
├── scripts/
│   ├── app.js          # 앱 진입점
│   ├── supabase.js     # Supabase 클라이언트 초기화
│   ├── db.js           # DB 쿼리 함수
│   └── chart.js        # Chart.js 래퍼
├── data/
│   └── sample.json
├── .env                # Git 제외 — 절대 커밋하지 말 것
└── vite.config.js
```

## 코딩 컨벤션

- **함수명**: camelCase — `loadDashboardData`, `fetchStockPrice`
- **파일명**: kebab-case — `dashboard-chart.js`, `stock-list.js`
- **들여쓰기**: 스페이스 2칸
- **주석**: 한국어

## 환경변수 규칙

- Supabase 관련 환경변수는 반드시 `VITE_` 접두사 사용
  - `VITE_SUPABASE_URL`
  - `VITE_SUPABASE_ANON_KEY`
- `.env` 파일은 절대 수정하거나 Git에 추가하지 말 것
- `.gitignore`에 `.env` 포함 여부를 항상 확인할 것

## 작업 규칙

1. **환경변수**: `.env` 파일 직접 수정 금지. 환경변수 이름만 참조할 것.
2. **동작 확인**: 기능 완성 후 반드시 브라우저에서 동작 확인.
3. **커밋 메시지**: Conventional Commit 형식 사용.
   ```
   feat: 미국 ETF 수익률 차트 추가
   fix: 차트 날짜 포맷 오류 수정
   chore: 의존성 업데이트
   ```
4. **ES modules**: `import/export` 사용. `require()` 금지.
5. **Supabase 쿼리**: `db.js`에 집중 관리. 다른 파일에서 직접 쿼리 작성 금지.

## 개발 명령어

```bash
npm run dev      # 개발 서버 실행 (Vite)
npm run build    # 프로덕션 빌드
npm run preview  # 빌드 결과 미리보기
```

## 배포

Vercel에 자동 배포. `main` 브랜치 푸시 시 트리거됨.
Vercel 대시보드에서 `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY` 환경변수 설정 필요.

---
> Source: [Odomhae/Stock_dashboard](https://github.com/Odomhae/Stock_dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
