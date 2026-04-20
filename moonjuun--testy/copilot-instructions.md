## testy

> Testy 서비스의 글로벌 검색 노출과 소셜 공유 최적화를 위해 아래 규칙을 엄격히 준수한다.


# SEO & Metadata Rules (Next.js 16 Standard)

Testy 서비스의 글로벌 검색 노출과 소셜 공유 최적화를 위해 아래 규칙을 엄격히 준수한다.
**반드시 Next.js 16 (App Router)의 비동기 Metadata API 표준을 따른다.**

## 1. 메타데이터 (Metadata) 작성 원칙

### 1.1 Next.js 16 Breaking Changes 준수 (필수)

- **Async Metadata:** `generateMetadata` 함수는 반드시 `async`여야 한다.
- **Await Params:** `params`, `searchParams`, `parent`는 **Promise** 객체이므로, 데이터 접근 전 반드시 `await` 키워드를 사용해야 한다.

  ```typescript
  // ❌ Bad (Next.js 14 이하 스타일)
  export function generateMetadata({ params }: Props) {
    const id = params.id; // Error in Next.js 16
  }

  // ✅ Good (Next.js 16 스타일)
  export async function generateMetadata({ params }: Props) {
    const { locale, testId } = await params; // Must await
  }
  ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Moonjuun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
