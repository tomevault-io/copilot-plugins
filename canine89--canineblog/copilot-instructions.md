## canineblog

> **이 프로젝트의 디자인은 완성된 상태입니다. 어떠한 디자인 관련 변경도 금지됩니다.**

# Claude Code 프로젝트 설정

## 🚨 중요: 디자인 변경 금지

**이 프로젝트의 디자인은 완성된 상태입니다. 어떠한 디자인 관련 변경도 금지됩니다.**

### 절대 변경하지 말아야 할 파일들:

#### 스타일 관련 파일
- `src/app/globals.css` - 전체 스타일시트
- `tailwind.config.js` - Tailwind CSS 설정
- `postcss.config.mjs` - PostCSS 설정

#### 레이아웃 및 컴포넌트 파일
- `src/app/layout.tsx` - 메인 레이아웃
- `src/app/page.tsx` - 홈페이지 레이아웃
- `src/components/CategoryDropdown.tsx` - 카테고리 드롭다운
- `src/components/MobileNav.tsx` - 모바일 네비게이션
- `src/components/Navigation.tsx` - 네비게이션 컴포넌트

#### 페이지 레이아웃 파일
- `src/app/about/page.tsx`
- `src/app/books/page.tsx`
- `src/app/posts/[...slug]/page.tsx`
- `src/app/category/[category]/page.tsx`
- `src/app/tags/page.tsx`

### 허용되는 작업:

✅ **기능적 변경만 허용**
- 새로운 기능 추가 (단, 기존 디자인 유지)
- 버그 수정
- 콘텐츠 추가/수정
- 성능 최적화
- SEO 관련 메타데이터 수정
- Google AdSense 관련 기능적 수정

✅ **새로운 파일 생성**
- 새로운 유틸리티 함수
- 새로운 데이터 처리 로직
- 새로운 마크다운 콘텐츠

❌ **금지되는 작업**
- CSS 클래스명 변경
- 레이아웃 구조 변경
- 색상, 폰트, 간격 등 시각적 요소 변경
- 컴포넌트의 HTML 구조 변경
- Tailwind 설정 변경
- 기존 스타일 파일 수정

### 디자인 변경 요청 시 대응방법:

사용자가 디자인 변경을 요청하면:
1. "현재 이 프로젝트는 디자인 변경이 금지되어 있습니다"라고 알림
2. 대신 기능적 개선사항을 제안
3. 정말 디자인을 변경하려면 `.claudeignore`와 `CLAUDE.md`를 수정하도록 안내

### 프로젝트 정보:
- **타입**: Next.js 15.4.4 블로그
- **스타일링**: Tailwind CSS
- **주요 기능**: 마크다운 블로그, Google AdSense
- **디자인 상태**: 완성됨 - 변경 금지

### 작업 우선순위:
1. Google AdSense 관련 기능적 문제 해결
2. 블로그 콘텐츠 관리 기능 개선
3. 성능 최적화
4. SEO 개선
5. 새로운 기능 추가 (디자인 변경 없이)

---
**⚠️ 주의: 이 지시사항을 무시하고 디자인을 변경하는 것은 금지됩니다. 모든 변경사항은 기존 디자인을 유지하면서 기능적 개선만 수행해야 합니다.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Canine89) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
