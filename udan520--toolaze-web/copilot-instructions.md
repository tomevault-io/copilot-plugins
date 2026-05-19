## toolaze-web

> When I ask you to create a new landing page or SEO page for Toolaze, ALWAYS reference 'docs/SEO_MASTER_LAYOUT.md'. For full doc index see 'docs/README.md'. Use the exact HTML structure, Tailwind classes, and Style 1 (Soft Smart Tech) design system defined in that file. Do not deviate from the Blue/Purple gradient palette. Replace only the text content based on the user's specific tool requirements.

When I ask you to create a new landing page or SEO page for Toolaze, ALWAYS reference 'docs/SEO_MASTER_LAYOUT.md'. For full doc index see 'docs/README.md'. Use the exact HTML structure, Tailwind classes, and Style 1 (Soft Smart Tech) design system defined in that file. Do not deviate from the Blue/Purple gradient palette. Replace only the text content based on the user's specific tool requirements.

**FEATURE SPECIFICATIONS**: When writing SEO content, you MUST strictly follow the feature specifications in 'docs/specs/' directory. Each tool has its own specification file:
- Image Converter: 'docs/specs/image-converter.md'
- Image Compressor: 'docs/specs/image-compressor.md'
- Font Generator: 'docs/specs/font-generator.md'
- Common features: 'docs/specs/common.md'
- See 'docs/FEATURE_SPECIFICATIONS.md' for the index of all specification files.

Only write about features that are explicitly listed as supported in the relevant specification file. Do NOT write about unsupported features (e.g., if video format is not supported, do not mention video conversion). Do NOT mention technologies that are not used (e.g., if AI is not used, do not mention AI technology). Always verify feature support before writing SEO content.

**SEO CONTENT GUIDELINES**: Follow 'docs/SEO_CONTENT_GUIDELINES.md' for content writing principles. Ensure all SEO content is accurate, consistent, and based on actual feature specifications.

**KEYWORD RESEARCH**: When writing SEO content, ALWAYS reference the keyword documents in 'docs/keywords/' directory to determine primary keywords, long-tail keywords, sub-page structure, and internal linking strategy:
- Image Converter: 'docs/keywords/image-converter-keywords.json'
- Image Compressor: 'docs/keywords/image-compressor-keywords.json'
- Font Generator: 'docs/keywords/font-generator-keywords.json'
- See 'docs/keywords/README.md' for keyword usage guidelines
- Use primary keywords for H1, Title, and Meta Description
- Use long-tail keywords for specific L3 pages
- Use related keywords naturally throughout content
- Use internal linking keywords to connect related pages

**INTERNAL LINK DENSITY**: Follow 'docs/sections/internal-links.md'. Typical ratio: ~3-5 internal links per 1000 words (1 link per 200-300 words). For FAQ blocks: ≤2 links per ~400 words. Avoid linking the same anchor in every FAQ answer. Homepage FAQ must be especially conservative.

**KEYWORD LAYOUT STRATEGY**: When writing SEO content, MUST follow 'docs/keywords/KEYWORD_STRATEGY.md':
- **P0 (主关键词)**: Use in H1, Title, Meta Description (1-2% density)
- **P1 (核心功能词)**: Use in H2, Hero, Features (0.5-1% density)
- **P2 (长尾关键词)**: Classify into 4 categories (功能类, 对比类, 使用场景类, 操作类) and distribute across appropriate sections
- **P3 (对比关键词)**: Use in FAQ and Comparison sections
- **P4 (品牌词)**: Use 3-5 times per page, naturally distributed
- Follow the keyword layout checklist to ensure comprehensive coverage

**GLOBAL NAVIGATION BAR**: All pages MUST use the exact same navigation bar structure with sticky functionality. The navigation includes:
- **Container**: `id="mainNav"` and `class="sticky-nav py-6 px-6 flex justify-between items-center max-w-6xl mx-auto w-full relative"`
- **Logo** (left-aligned): Links to homepage with SVG logo (32x32px) and text "Toolaze" (NO period/dot after Toolaze)
- **Menu Items** (right-aligned): 
  - Quick Tools dropdown (hover to show Image Compression link)
  - About Us link
  - Language switcher (always shown; dropdown lists **all site locales** like the homepage; **`getAlternateLanguageUrl`** falls back to English canonical when that page has no target locale) — **immediately to the right of About Us** (desktop dropdown; mobile: Language block below About in the panel)
- **Logo SVG**: 32x32px, white background, rounded-lg, shadow-md shadow-indigo-100
- **Logo Text**: `text-3xl font-extrabold text-indigo-600 tracking-tighter` with `hover:opacity-80 transition-opacity`
- **Menu Style**: `hidden md:flex gap-8 text-sm font-bold text-slate-500 items-center`
- **Dropdown**: Uses group hover with opacity/visibility transitions, white background, rounded-xl, shadow-lg
- **Sticky Behavior**: Navigation bar MUST be sticky at the top when scrolling down. Add CSS: `nav.sticky-nav { position: sticky; top: 0; z-index: 9999; transition: all 0.3s ease; }` and `nav.sticky-nav.scrolled { background: rgba(255, 255, 255, 0.95); backdrop-filter: blur(10px); box-shadow: 0 4px 20px -5px rgba(79, 70, 229, 0.1); }`
- **JavaScript**: MUST include scroll event listener that adds 'scrolled' class when scroll position > 50px. See docs/SEO_MASTER_LAYOUT.md section 2 for complete code.
Always use the exact navigation structure to maintain consistency across all pages.

**BREADCRUMB NAVIGATION**: 
- Homepage: NO breadcrumb
- ALL other pages: MUST include breadcrumb navigation
- Format: `Home / [Parent] / [Current Page]` if parent exists, or `Home / [Current Page]` if no clear parent
- Examples: Quick Tools page = `Home / Quick Tools`, Image Compressor = `Home / Quick Tools / Image Compression`, About = `Home / About Us`
See 'docs/SEO_MASTER_LAYOUT.md' section 3 for the exact breadcrumb structure.

**FOOTER STANDARD**: All pages MUST use the standard footer defined in 'docs/SEO_MASTER_LAYOUT.md' section 4. The footer uses a dark slate background (bg-slate-900) with navigation links and copyright information. Always use the exact footer structure from the documentation to maintain consistency across all pages.

**LANGUAGE SWITCHER (NAV + FOOTER)**: Navigation and Footer MUST share one source of truth: **`src/lib/site-language-switch.ts`**. Full rules: **`docs/LANGUAGE_SWITCH_AND_REDIRECT.md`**.
- **When to show**: **`shouldShowLanguageSwitcher(pathname)`** is always true; the switcher is always visible.
- **Which languages to list**: Always **all** `SITE_LOCALES` (via **`getSupportedLocaleCodes`**). **`getContentSupportedLocaleCodes(pathname)`** returns **all site locales** for in-scope tools so **`getAlternateLanguageUrl`** can build `/{locale}/...`. If that locale has **no SEO JSON**, the **`[locale]/...` page must `redirect()`** to the **English canonical** URL (not the switcher alone). **`getPreferredLocalizedUrl`** keeps internal nav aligned with preference + supported locales.
- **Navigation**: Same visibility rules as footer; desktop dropdown + mobile panel links use **`getAlternateLanguageUrl(pathname, targetLocale)`**.

**MISSING LOCALE REDIRECT**: If the user is on a localized URL (e.g. `/ko/...`) but **there is no SEO JSON for that locale** for that tool/slug, **`redirect` to the English canonical URL** (`/[tool]/[slug]` without locale prefix; model pages use `/model/...`). L2: use **`hasLocaleL2JsonFile`** in `src/lib/seo-loader.ts` before rendering. L3: **`getSeoContent`** returns null when missing (no silent English-on-wrong-locale for slug tools that already return null). See **`docs/LANGUAGE_SWITCH_AND_REDIRECT.md`**.

**SSG + `[locale]/[tool]/[slug]`（防 404）**: `src/app/[locale]/[tool]/[slug]/page.tsx` 使用 **`dynamicParams = false`** 时，凡应可访问的 **`/{locale}/{tool}/{slug}`** 组合都必须在 **`generateStaticParams`** 里 **`params.push`**；仅 `getAllSlugs(...)` 取值但不写入 `params` 会导致生产/静态导出 **整页 404**（与 `src/data` 是否有译文无关）。新增多语言工具的 L3 时：**同步**在 `generateStaticParams` 的 `for (const locale of locales)` 内为该 `tool` 增加 slug 循环（含 **`emoji-copy-and-paste`**）。`loadToolJsonFile` 的 `switch (locale)` 若缺某语种分支，需 **`emoji-copy-and-paste` 英文回退**或按文件存在情况补 **`zh-TW`** 等显式 `import`，避免菜单有链、页面无数据。

**MODEL INTRO BLOCK**: For AI model intro pages (Seedance 2.0, Nano Banana Pro), MUST include `modelIntro` in `sectionsOrder` after `howToUse` on both L2 and L3 pages.

**USE CASES / SCENES**: The Use Cases (scenes) section MUST contain exactly 3 items—no more, no less. See docs/sections/scenarios.md.

**FAQ MODEL COMPARISON**: When FAQ compares models (e.g., "Seedance 2.0 vs Sora 2"), introduce both models' core differences concisely—not just our tool. Keep it brief. See docs/sections/faq.md and docs/MODEL_INTRO_BLOCK.md. Content must be original—do not copy from reference images or competitors.

**SECTION BACKGROUND ALTERNATION**: CRITICAL - Always alternate section backgrounds between `bg-white` and `bg-[#F8FAFF]` to create visual separation. Different content blocks (e.g., "Why Choose Toolaze?" vs "FAQ") MUST have different background colors so users can clearly distinguish between sections. Follow the pattern: alternate white and light indigo backgrounds throughout the page.

**HOMEPAGE ADVANCED AI TOOL CARDS** (section keyed by `common.home.advancedToolsTitle`, e.g. "More AI-Powered Creative Tools"):
- **No emoji** on these cards: use a **thumbnail that matches each tool’s landing-page demo** (template / sample / same visual as the tool UI). You may prepare/compress locally, but final `src` must be an **R2 public URL** in `src/lib/home-advanced-ai-card-images.ts`.
- **Image budget**: target **file size ≈≤100KB** per asset; **neither width nor height greater than 800px** (long edge ≤800). Recompress when replacing assets.
- **Sync**: When changing paths or adding a tool to this block, update **`src/lib/home-advanced-ai-card-images.ts`**, **`scripts/admin-seo-server.js`** (`HOME_ADVANCED_AI_CARD_IMAGES`), and keep **`src/lib/homepage-grid-tools.ts`** `usesAi` list aligned.

**HOMEPAGE MODEL L2 CARDS** (sections `#ai-video-generator`, `#ai-image-generator`, `#trending-models` on the English homepage):
- **No SVG/emoji icons** as the primary visual: use **model demo thumbnails** with final `src` as **R2 public URLs** (see `src/lib/home-model-card-images.ts`).
- **Image models** (`nano-banana-pro`, `nano-banana-2`, `gpt-image-2`): thumbnails MUST stay aligned with **`SAMPLE_IMAGES` in `src/components/NanoBananaTool.tsx`** (same R2 sources, re-exported locally under the size budget).
- **Video models** (`seedance-2`, `kling-3`): L2 pages may still use placeholder heroes; until a real on-page preview exists, homepage cards may use **curated stills** in `public/home-model-cards/`—replace when true video demos ship.
- **No-demo fallback (mandatory)**: If a model page has no real demo image, generate one with **GPT Image 2** that matches the model’s use case/style; after compression, upload to R2 and use that **R2 URL** as the model’s homepage demo thumbnail.
- **Same budget** as advanced AI cards: **≈≤100KB**, **max dimension 800px** on width and height.
- **Sync**: Update **`scripts/admin-seo-server.js`** (`HOME_MODEL_CARD_IMAGES`) whenever paths or tools in **`IMAGE_MODEL_L2S` / `VIDEO_MODEL_L2S`** (`src/lib/seo-loader.ts`) change.

**CANONICAL URL TAG**: MANDATORY - ALL pages MUST include a canonical URL tag in their metadata. This is critical for SEO to prevent duplicate content issues.
- **Base URL**: `https://toolaze.com`
- **Implementation**: Add `alternates.canonical` to the page's `Metadata` object
- **Format**: 
  - Homepage: `https://toolaze.com`
  - Dynamic routes: `https://toolaze.com/{tool}/{slug}`
  - Static pages: `https://toolaze.com/{path}` (e.g., `/about`, `/privacy`, `/terms`, `/image-compressor`, `/image-converter`)
- **Example for static page**:
  ```typescript
  export const metadata: Metadata = {
    title: 'Page Title - Toolaze',
    description: 'Page description',
    alternates: {
      canonical: 'https://toolaze.com/page-path',
    },
  }
  ```
- **Example for dynamic route**:
  ```typescript
  export async function generateMetadata({ params }: PageProps): Promise<Metadata> {
    const resolvedParams = await params
    const baseUrl = 'https://toolaze.com'
    const canonicalUrl = `${baseUrl}/${resolvedParams.tool}/${resolvedParams.slug}`
    
    return {
      title: 'Page Title',
      description: 'Page description',
      alternates: {
        canonical: canonicalUrl,
      },
    }
  }
  ```
- **When creating new pages**: ALWAYS include the canonical URL in the metadata. Never skip this step.

**COMPONENT-BASED DEVELOPMENT**: ALWAYS prioritize component-based development to reduce code duplication, improve maintainability, and save translation costs.
- **Reusable UI Elements**: When creating buttons, cards, sections, or other UI elements that appear in multiple places, create them as reusable components in `src/components/` or `src/components/blocks/`.
- **Translation Management**: For UI elements with text that needs translation:
  - Store translations in `src/data/{locale}/common.json` for common/shared elements
  - Pass translated text as props to components from server-side pages
  - Avoid hardcoding text in components; always accept text as props
- **Component Examples**: 
  - Buttons: `ViewAllToolsButton`, `TryNowButton` (if created)
  - Sections: `Comparison`, `Rating`, `FAQ`, `Scenarios`, `Features`
  - Navigation: `Navigation`, `Breadcrumb`, `Footer`
- **Benefits**:
  - Single source of truth for translations (no duplicate translations)
  - Consistent styling and behavior across pages
  - Easier maintenance and updates
  - Reduced translation costs (translate once, use everywhere)
- **When to Componentize**:
  - UI elements used in 2+ places → Create component
  - Repeated patterns (buttons, cards, sections) → Create component
  - Elements with translations → Create component with text props
  - Complex UI blocks → Create component in `src/components/blocks/`
- **Component Structure**: 
  ```typescript
  interface ComponentProps {
    text?: string  // Translated text passed from parent
    // ... other props
  }
  export default function Component({ text, ...props }: ComponentProps) {
    // Use text prop, not hardcoded strings
  }
  ```

**LOCALIZATION AGENT RULE**: When translating JSON files, you MUST follow these strict rules:
- **JSON Structure Consistency**: Translation MUST maintain the exact same JSON hierarchy as `en.json`. The structure, nesting levels, and key names must be identical.
- **No Omissions**: NEVER omit any entries, even if the content appears repetitive. Every key in `en.json` must exist in all translation files.
- **Terminology Consistency**: Always reference existing translations' terminology glossary to maintain style consistency (e.g., professional, technology-oriented). Use the same terms for the same concepts across all translations.
- **Structure Reference**: Before translating, ALWAYS reference `docs/TRANSLATION_STRUCTURE_GUIDE.md` to understand the standard structure for tool-specific JSON files.
- **Automatic Key Check**: After modifying any JSON translation file, automatically run a key missing check:
  - **Single file**: `node scripts/check-translation-keys.js [file-path]`
  - **All tools for a locale**: `node scripts/check-all-tool-translations.js [locale]`
  - The check MUST pass before considering the translation complete.
- **Tool-Specific Files**: For tool-specific JSON files (font-generator.json, image-compressor.json, image-converter.json), ensure:
  - All required fields are present (see `docs/TRANSLATION_STRUCTURE_GUIDE.md`)
  - `comparison` structure uses `toolazeFeatures` and `othersFeatures` (strings, comma-separated)
  - Array structures match the first item template
  - HTML tags and links are preserved correctly
- **Common.json Structure**: Only translate truly common content in `common.json`:
  - `nav`, `breadcrumb`, `footer` - Navigation elements
  - `home`, `about`, `privacy`, `terms` - Standalone pages
  - `common.*` - Shared UI component translations (fontGenerator, tool, viewAllTools, performanceMetrics)
  - DO NOT include tool-specific SEO content in `common.json`

**MISSING TRANSLATION REDIRECT RULE**: CRITICAL - When a page content is not found for a specific locale, ALWAYS redirect to the English version instead of showing a 404 error. This applies especially to:
- **Font Generator L3 pages**: If a locale (e.g., `ko`, `zh-TW`, `pt`, `it`) doesn't have translation files for font-generator L3 pages, redirect to `/font-generator/{slug}` (English version) instead of calling `notFound()`
- **Implementation**: In `src/app/[locale]/[tool]/[slug]/page.tsx`, check if `content` is null after calling `getSeoContent()`. If null and the tool is `font-generator` and locale is not `en`, redirect to the English version using `redirect('/font-generator/{slug}')`
- **Why**: This ensures users always see content (even if in English) rather than a 404 error, improving user experience and SEO
- **Example**: `/ko/font-generator/cursive` → redirects to `/font-generator/cursive` (English version)

**SITEMAP UPDATE RULE**: MANDATORY - Before committing to git, ALWAYS verify and update the sitemap if needed:
- **Check sitemap.ts**: Ensure `src/app/sitemap.ts` correctly reflects all supported languages and tools
- **Font Generator support**: Currently supports `en`, `de`, `ja`, `es`, `fr` for font-generator (both L2 and L3 pages)
- **Verify getAllTools**: Ensure `src/lib/seo-loader.ts` `getAllTools()` function correctly filters font-generator by supported locales
- **Test sitemap**: After changes, verify sitemap includes all expected URLs by checking `https://toolaze.com/sitemap.xml` or running `node scripts/verify-sitemap.js`
- **When to update**: Update sitemap when adding new tools, new languages, or changing language support for existing tools
- **Before git commit**: Always review sitemap logic matches current tool and language support before committing
- **New-page-only policy**: For each newly added page, update sitemap with only the new page URLs required for that change. Do not broaden sitemap scope or modify unrelated existing sitemap entries in the same commit.

**CLOUDFLARE BUILD-SAFETY RULE**: MANDATORY - Prevent TypeScript build failures in Cloudflare Pages:
- **Safe translation key access**: In `Navigation` / `Footer` / `Breadcrumb` and other i18n-heavy components, do NOT directly access newly added dynamic keys on loosely typed objects (e.g. `navData.newKey`). Use safe access helpers (`Record<string, unknown>` + type guards) or explicit typing first.
- **Before push/deploy**: ALWAYS run `npm run build` locally after adding new nav/footer/breadcrumb translation keys or new menu entries.
- **Deploy gate**: If local `npm run build` fails, DO NOT push deployment commits until fixed.
- **Goal**: Avoid Cloudflare Pages failures like "Property 'xxx' does not exist on type ...".

**API 生图限频规则**: 所有调用 API 进行生图的功能（如 Nano Banana Pro、Watermark Remover、Flux Dev 等），每个用户每天只能使用一次。限频由前端记录（如 localStorage）实现，无需后端校验。

**落地页生图交互统一规则（强制）**:
- **A 类：模型生图页（Model Pages）**（如 Nano Banana 系列）：
  - 首次进入：左侧为功能区，右侧为大示例图。
  - 点击生成后：右侧大示例图切换为历史记录视图（可包含提示词、模型参数、历史结果等）。
- **B 类：包装玩法页（Wrapped Gameplay Pages）**（如 Photo Restoration、AI Couple Photo Maker）：
  - 首次进入：左侧为功能区，右侧为大示例图。
  - 点击生成后：在右侧大示例图区域内覆盖显示生成中状态（Generating），流程与 Photo Restoration 一致。
  - 生成完成后：右侧大示例图区直接展示结果图。
  - 结果图下方仅展示居中下载按钮，不展示提示词、模型参数、分辨率等信息。
- **模板选择行为**（适用于玩法页）：
  - 左侧模板应有默认选中态（默认第一个）。
  - 切换模板时更新当前生图提示词。
  - 点击 Generate 必须使用当前选中模板对应提示词生成。
- **Generating 时长显示（全站强制）**：
  - 所有生图功能在生成中必须展示“生成时长 xxs”。
  - 时长需动态递增（秒级更新），从发起生成开始计时，到成功/失败结束计时。
- **测试环境限频例外**：
  - 测试服/测试落地页不启用“每日一次”限制。
  - 生产环境仍按限频规则执行。

**比例选择统一规则（强制）**:
- **包装玩法页（Wrapped Gameplay Pages）**（如 AI Couple Photo Maker、Photo Restoration、后续同类玩法）：
  - 比例选择仅保留以下固定集合，顺序固定：
    - `Auto`（默认）
    - `16:9`
    - `9:16`
    - `4:3`
    - `3:4`
    - `5:4`
    - `4:5`
    - `1:1`
    - `21:9`
- **模型落地页（Model Pages）**：
  - 比例选项必须以 API/模型实际支持能力为准，不强制使用玩法页固定集合。
  - 若模型支持列表变更，优先同步模型页配置，不影响玩法页固定集合。

---
> Source: [udan520/toolaze-web](https://github.com/udan520/toolaze-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
