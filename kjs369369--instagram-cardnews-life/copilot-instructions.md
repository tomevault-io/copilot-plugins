## instagram-cardnews-life

> > 메이TV(@AICLab_TV) 브랜드 카드뉴스를 클로드코드로 자동 생성합니다.

# 인스타그램 카드뉴스 자동 생성 프로젝트

> 메이TV(@AICLab_TV) 브랜드 카드뉴스를 클로드코드로 자동 생성합니다.

---

## 🎯 프로젝트 목표

`content/cards.json`의 텍스트만 수정하면, 1080×1350 PNG 카드뉴스 10장이 자동 생성된다.

---

## 📐 디자인 규격

| 항목 | 값 |
|------|---|
| 사이즈 | **1080 × 1350px** (인스타 4:5 세로) |
| 카드 수 | 10장 (표지 1 + 본문 8 + CTA 1) |
| 디스플레이 폰트 | Pretendard Variable (검정 900) |
| 본문 폰트 | Pretendard (Regular 400, Medium 500) |
| 메인 컬러 | `#1a2332` (딥네이비) |
| 포인트 컬러 | `#d4af37` (골드) |
| 배경 | `#f5f1e8` (아이보리) |
| 텍스트 | `#1a2332` (딥네이비) |

---

## 🗂️ 카드 구성 (10장)

| 번호 | 역할 | 디자인 패턴 |
|------|------|-------------|
| 1 | 표지 (후킹 헤드라인) | 큰 타이틀 + 골드 배지 |
| 2~9 | 본문 (한 카드 한 메시지) | 숫자 + 소제목 + 본문 |
| 10 | CTA (구독 유도) | 메이TV 로고 + 행동 유도 |

---

## ⚙️ 실행 방법

```bash
npm install
node scripts/generate.js
```

→ `output/` 폴더에 `card-01.png` ~ `card-10.png` 생성

---

## 📁 폴더 구조

```
instagram-cardnews/
├── CLAUDE.md                  # 이 파일
├── package.json               # 의존성 정의
├── templates/
│   ├── cover.html             # 표지 템플릿
│   ├── content.html           # 본문 템플릿
│   └── cta.html               # CTA 템플릿
├── content/
│   └── cards.json             # 카드별 텍스트
├── scripts/
│   └── generate.js            # Playwright 자동 캡처
└── output/                    # 결과 PNG 저장
```

---

## ✏️ 콘텐츠 수정 방법

`content/cards.json`만 수정하면 끝. 다른 파일은 건드릴 필요 없음.

---

## 🎨 디자인 수정 시

각 템플릿 HTML 내부의 `<style>` 블록만 수정.
컬러는 `:root` CSS 변수로 정의되어 있어 한 곳에서 관리 가능.

---

## 🔄 재사용 시나리오

1. **새 카드뉴스 제작**: cards.json 텍스트만 교체 → 실행
2. **유튜브 영상 연동**: `youtube-script-to-3pack` 스킬로 cards.json 자동 생성 → 실행
3. **시리즈 제작**: cards.json을 여러 개 만들고 `--input` 옵션으로 분기

---

## 👤 운영자 정보

- 운영자: 김진수(May 작가) - AICLab 소장 / 메이랜드AI비즈랩 CEO
- 브랜드: AI활용교육 & GEO마케팅전문가
- 유튜브: @AICLab_TV

---
> Source: [kjs369369/instagram-cardnews-life](https://github.com/kjs369369/instagram-cardnews-life) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-08 -->
