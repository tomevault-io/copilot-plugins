## shoppingmall

> | 프론트엔드 | HTML / CSS / Vanilla JS |

# GOODS SHOP — 프로젝트 개요

## 기술 스택

| 역할 | 기술 |
|---|---|
| 프론트엔드 | HTML / CSS / Vanilla JS |
| 호스팅 | GitHub Pages |
| 인증 & DB | Supabase (Auth + PostgreSQL) |
| 결제 | Toss Payments (테스트 모드) |
| 결제 승인 서버 | Supabase Edge Functions (Deno) |

## 페이지 구조

- `index.html` — 상품 목록, 장바구니, 결제 진입
- `auth.html` — 로그인 / 회원가입 (이메일 인증 없음)
- `checkout.html` — Toss 결제 위젯
- `success.html` — 결제 완료 처리
- `fail.html` — 결제 실패 안내
- `orders.html` — 내 주문 내역
- `admin.html` — 관리자 전용 (주문 조회 + 상품 관리)

## 환경변수 / 설정값 위치

`js/config.js` 파일에 아래 값을 직접 입력:

```js
const SUPABASE_URL     = 'https://xxxx.supabase.co';
const SUPABASE_ANON_KEY = 'eyJ...';
const TOSS_CLIENT_KEY  = 'test_ck_D5GePWvyJnrK0W0k6q8gLzN97Eo0'; // 기본값 유지 가능
```

## 관리자 계정

- 이메일: `admin@admin.com`
- 비밀번호: `superadmin`
- `auth.html`에서 회원가입하면 트리거가 자동으로 `role = 'admin'` 부여

## 배포 순서 (빠른 참조)

1. Supabase 프로젝트 생성
2. Authentication > Email > **Confirm email OFF**
3. SQL 에디터에서 `sql/schema.sql` 실행
4. SQL 에디터에서 `sql/seed.sql` 실행
5. `js/config.js`에 Supabase URL + anon key 입력
6. Edge Function 배포:
   ```
   supabase functions deploy confirm-payment --project-ref <ref>
   supabase secrets set TOSS_SECRET_KEY=test_sk_zXLkKEypNArWmo50nX3lmeaxYG5R --project-ref <ref>
   ```
7. GitHub에 push → Settings > Pages > `main` 브랜치 루트 선택
8. `admin@admin.com / superadmin`으로 회원가입

## Toss Payments 테스트 카드

| 카드번호 | 유효기간 | CVC |
|---|---|---|
| 4242 4242 4242 4242 | 임의 미래 날짜 | 임의 3자리 |

---
> Source: [everyonegenie/shoppingmall](https://github.com/everyonegenie/shoppingmall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
