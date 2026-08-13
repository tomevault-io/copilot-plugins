## careerwiki

> **다음 작업은 반드시 사용자의 명시적 동의 없이 절대 실행하지 마세요:**

# Cursor AI Rules for CareerWiki Project

## ⛔ 절대 금지 규칙 (CRITICAL - NEVER DO THIS)

**다음 작업은 반드시 사용자의 명시적 동의 없이 절대 실행하지 마세요:**

1. **`git stash`** - 커밋되지 않은 변경 사항이 사라짐 (2026-01-08 사고 발생)
2. **`git reset --hard`** - 모든 변경 사항 영구 삭제
3. **`git checkout -- .`** - 모든 변경 사항 삭제
4. **`DROP TABLE`** - 테이블 삭제
5. **`DELETE FROM` (WHERE 없이)** - 전체 데이터 삭제
6. **`rm -rf`** - 파일/폴더 영구 삭제
7. **데이터베이스 파일 직접 삭제**

**대신 해야 할 것:**
- 코드 변경 전: `git commit -m "백업: 작업 전 상태"` 로 먼저 커밋
- 데이터 작업 전: 백업 파일 생성 확인
- 위험한 작업 전: 사용자에게 먼저 경고하고 동의 받기

**2026-01-08 사고 기록:**
> `git stash`로 인해 20,000줄 이상의 작업이 숨겨지고 코드가 이전 상태로 롤백됨.
> 사용자가 작업한 "소개 탭", "직업 리스트 이미지", "HowTo" 등 모든 변경 사항이 사라짐.
> `git stash pop`으로 복구했지만, 사용자에게 큰 불안감을 줌.

---

## Project Context
This is a lightweight web application built with:
- **Backend**: Hono framework (TypeScript)
- **Frontend**: Hono JSX SSR + Tailwind CSS via CDN
- **Deployment**: Cloudflare Pages + Workers
- **Database**: Cloudflare D1 (SQLite)
- **Build**: Vite + esbuild
- **Process Manager**: PM2

## Architecture
- **ISR (Incremental Static Regeneration)**: Wikipedia-style cached dynamic pages
- **Template Version System**: Automatic cache invalidation on template updates
- **API Integration**: CareerNet + Goyong24 Korean job/major data
- **Sample Registry**: Fallback data for development

## ISR Cache Management (CRITICAL)
**⚠️ ALWAYS increment template version when modifying template code or data processing logic!**

- Template versions: `src/constants/template-versions.ts`
  - `MAJOR`: Major detail page template version
  - `JOB`: Job detail page template version
  - `HOWTO`: How-to guide template version

**Cache Behavior**:
- Cache key = `slug` + `template version`
- If version unchanged → cached HTML is served (new code won't execute)
- If version changed → cache miss → new HTML generated

**When to increment**:
- ✅ Template HTML structure changes
- ✅ Data processing/normalization logic changes
- ✅ Styling changes (classes, inline styles)
- ✅ Client-side JavaScript changes
- ❌ Only backend logic changes (not affecting HTML output)

**Example**:
```typescript
// BAD: Changed code but forgot to increment version
// Result: Cached HTML still shows old data
export const TEMPLATE_VERSIONS = {
  MAJOR: 47,  // ❌ Should be 48!
}

// GOOD: Always increment when changing template logic
export const TEMPLATE_VERSIONS = {
  MAJOR: 48,  // ✅ Version bumped
}

## Current Status (2026-01-08)
**✅ Production 마이그레이션 완료!**

### 완료된 작업:
1. **D1 데이터베이스 마이그레이션** - 모든 핵심 테이블 Production에 업로드 완료
2. **R2 이미지 업로드** - ~7,500개 이미지 Production R2에 업로드 완료
3. **개발환경 연동** - `npm run dev`로 Production D1/R2 직접 연결

### 개발 명령어:
```bash
npm run dev              # Production D1/R2 연결 (wrangler dev --remote)
npm run deploy:prod      # Cloudflare Pages 배포
```

### 환경 설정:
- `.dev.vars` 파일에 `CLOUDFLARE_API_TOKEN` 필요
- `wrangler.jsonc`에 R2/D1 바인딩 설정됨

### 알려진 이슈:
- HowTo 데이터가 Production D1에 없음 (원본 데이터 손실됨)
- 로컬 D1에는 HowTo 데이터 있음 (백업에서 복구됨)

## Code Style
- Use TypeScript strict mode
- Prefer functional programming
- Use Hono JSX for templates (NOT React)
- Korean comments for business logic
- English comments for technical implementation

## Important Files
- `src/index.tsx` - Main application entry
- `src/templates/` - SSR templates (Hono JSX)
- `src/data/sampleRegistry.ts` - Sample data
- `src/utils/page-cache.ts` - ISR cache utilities
- `public/static/api-client.js` - Client-side scripts

## Testing URLs
- Home: http://localhost:3000
- Job detail: http://localhost:3000/job/software-developer
- Major detail: http://localhost:3000/major/digital-marketing-major
- Debug mode: http://localhost:3000/major/digital-marketing-major?debug=true

## Build & Run
```bash
npm run build           # Vite build (works on local, fails on GenSpark)
pm2 start ecosystem.config.cjs  # Start server
npm run dev:d1          # Alternative: Vite dev with D1
```

## Deployment
```bash
npm run deploy:prod     # Deploy to Cloudflare Pages
```

## Critical Constraints
- **NO Node.js APIs** in production code (Cloudflare Workers runtime)
- **NO file system access** at runtime
- Static files MUST be in `public/` and served via `serveStatic`
- Use Web APIs only (Fetch, Web Crypto, etc.)

## Communication
- User's name: 정우 (Jeong-woo)
- Respond in Korean (ko-KR)
- Be concise and action-oriented

## Next Steps
1. HowTo 데이터 Production D1에 업로드 (로컬 D1에서 추출 필요)
2. 추가 기능 개발 진행

## Reference Docs
- Quick Start: `CURSOR_QUICKSTART.md`
- Full Migration Guide: `MIGRATION_TO_CURSOR.md`
- Project README: `README.md`

---
> Source: [Tok2coder/Careerwiki](https://github.com/Tok2coder/Careerwiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
