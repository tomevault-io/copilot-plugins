## jtd-rmw

> This is an extensively modified version of Patrick Marsceill's [Just the Docs](https://github.com/just-the-docs/just-the-docs) Jekyll theme, used as a custom GitHub Pages remote theme. Consuming sites reference it via `remote_theme: RMWinslow/JTD-RMW` in their `_config.yml`.

# CLAUDE.md — JTD-RMW

## Project Overview

This is an extensively modified version of Patrick Marsceill's [Just the Docs](https://github.com/just-the-docs/just-the-docs) Jekyll theme, used as a custom GitHub Pages remote theme. Consuming sites reference it via `remote_theme: RMWinslow/JTD-RMW` in their `_config.yml`.

## Repo Structure

### Layouts (`_layouts/`)

The layout inheritance chain is: `vendor/compress` → `table_wrappers` → `default` → content layouts.

- **`vendor/compress.html`** — The outermost wrapper. A third-party HTML compression layout that minifies the final output, controlled by the `compress_html` setting in the consuming site's `_config.yml`.
- **`table_wrappers.html`** — Wraps `<table>` elements in `<div>` wrappers for responsive scrolling. Behavior is controlled by the `table_wrappers` frontmatter variable (see below). This is the parent of `default.html`.
- **`default.html`** — The main structural layout. Renders the sidebar navigation, search bar, breadcrumbs, heading anchors, the parent-child TOC, and the page footer (including "last edited" timestamps and "edit on GitHub" links). Almost all other layouts inherit from this.
- **`post.html`** — Extends `default`. Adds a title/subtitle header group, a date line (posted and last modified dates, author), and an optional collapsible table of contents. This is the layout to use for most content pages.
- **`page.html`** — Extends `default`. A bare passthrough — just renders `{{ content }}` with no additional chrome. Identical to `home.html` and `about.html`.
- **`home.html`** — Extends `default`. Identical to `page.html`.
- **`about.html`** — Extends `default`. Identical to `page.html`.
- **`remark_slides.html`** — A standalone layout (does **not** inherit from `default`). Renders the page content as a [Remark.js](https://remarkjs.com/) slideshow with KaTeX math support. Includes its own inline CSS for slide styling. Note: this layout replaces `\\` with `\\\\\\\\` in the content to make LaTeX escapes survive the Remark markdown engine, which can break filenames containing underscores.
- **`null.html`** — Renders raw content with no wrapping at all. Intended for including pre-built HTML pages directly.

### Includes (`_includes/`)

- **`head.html`** — The `<head>` element. Loads the compiled JTD stylesheet, sets the page `<title>`, includes Google Analytics (if configured), loads a math renderer if one is requested (no math loaded by default), loads the Lunr search script, and includes the SEO and custom head partials.
- **`head_seo.html`** — Outputs Open Graph meta tags (`og:title`, `og:description`, `og:image`, `og:url`, canonical URL, and author). Uses `page.description`, `page.imagepreview`, `site.title`, `site.author`, `site.url`, and `site.baseurl`.
- **`head_custom.html`** — Empty by default. Consuming sites can override this to inject custom `<head>` content (additional stylesheets, scripts, etc.).
- **`header_custom.html`** — Empty by default. Injected into the main content area header, after the search bar.
- **`nav_details.html`** — The primary sidebar navigation. Uses `<details>`/`<summary>` elements for collapsible sections. Supports up to four levels of nesting (top-level → children → grandchildren → great-grandchildren via `parent`, `grand_parent`, and `great_grand_parent`). Respects `nav_order`, `nav_exclude`, and `has_children`. Also renders `site.nav_external_links`.
- **`nav.html`** — An older navigation partial using JavaScript-driven expand/collapse (chevron icons) instead of `<details>` elements. Supports only three levels of nesting. **This file is not currently included anywhere** — `default.html` uses `nav_details.html` instead.
- **`parent_child_toc.html`** — Automatically generates a "Pages in this group" list at the bottom of any parent page. Shows each child's title, dates, subtitle, and description. Respects `nav_exclude`. Controlled by the `has_toc` frontmatter variable.
- **`title.html`** — Renders either the site logo (if `site.logo` is set) or the site title text.
- **`toc.html`** — The [jekyll-toc](https://github.com/allejo/jekyll-toc) include (v1.2.0). Generates a table of contents from heading elements in the rendered HTML. Used by `post.html` when `toc: true` is set.
- **`katex.html`** — Loads KaTeX v0.16.22 from the jsDelivr CDN: the stylesheet, the main script (deferred), auto-render extension, and copy-tex extension. Configures delimiters (`$$...$$` for display, `$...$` for inline, plus `\(...\)` and `\[...\]`), enables `globalGroup` (for `\gdef`), and sets `throwOnError: false` to prevent one bad equation from breaking the entire page. Includes a small style block to make display math horizontally scrollable with a max width of 800px.
- **`mathjax.html`** — Loads MathJax v4.0.0-beta.4 from jsDelivr. Configures `$...$` for inline math. Note: this is a beta version of MathJax 4.
- **`fix_linenos.html`** — A workaround for a Jekyll/Rouge bug where highlighted code blocks with line numbers produce invalid nested `<pre>` tags. Used via `{% include fix_linenos.html code=some_var %}`.
- **`footer_custom.html`** — Renders `site.footer_content` if present.
- **`nav_header_custom.html`** — Empty. Can be overridden by consuming sites to add content above the navigation.
- **`nav_footer_custom.html`** — Empty. Can be overridden by consuming sites to add content below the navigation.

### CSS Includes (`_includes/css/`)

- **`just-the-docs.scss.liquid`** — The SCSS entry point. Imports support modules, the selected color scheme, all JTD modules, and the custom SCSS override file. This is how the theme's Sass gets compiled.
- **`custom.scss.liquid`** — Imports `custom/custom`, providing a hook for consuming sites to add their own Sass.

### Assets (`assets/`)

- **`css/compiled-jtd-style.css`** — The pre-compiled main stylesheet for JTD. Contains all the core layout, typography, navigation, and search styles. This is a compiled artifact, not a source file to edit directly.
- **`css/colorscheme.css`** — Defines CSS custom properties for the color scheme (solarized-inspired light theme with a dark mode `prefers-color-scheme` media query). Variables include `--basecolor`, `--boxcolor`, `--feedbackcolor`, `--accentcolor`, `--bordercolor`, `--textcolor`, and `--titlecolor`.
- **`css/extrabits.css`** — Empty placeholder for consuming sites to add custom CSS.
- **`css/kineticgraphs.css`** — Styles for [kgjs](https://github.com/cmakler/kgjs) interactive graph containers: text selection prevention on SVG text, KaTeX font sizing, game theory matrix tables, slider/checkbox styling. Contains a commented-out scrollbar-hiding trick (`width: 0px` on `::-webkit-scrollbar`).
- **`js/just-the-docs.js`** — The main JavaScript file handling search (Lunr integration), navigation toggle, and scroll behavior.
- **`js/zzzz-search-data.json`** — A Liquid template that generates the search index JSON. Iterates over all site pages (respecting `search_exclude`), splits content by headings, and creates searchable entries with titles, content snippets, and URLs. The `zzzz-` prefix ensures it sorts last in directory listings.
- **`js/vendor/lunr.min.js`** — The Lunr.js search library.

### Other Files

- **`_config.yml`** — The theme's default Jekyll configuration. The `url` and `baseurl` values here are vestiges of the original JTD and get overridden by each consuming site. Contains defaults for search, navigation sorting, footer, "edit on GitHub" links, Google Analytics, and kramdown settings.
- **`Gemfile`** — Points to the `just-the-docs` gemspec.
- **`just-the-docs.gemspec`** — The gem specification (v0.3.3). Lists Jekyll ≥ 3.8.5 as a runtime dependency.
- **`lib/tasks/search.rake`** — A Rake task (likely for building the search index).

## Page Frontmatter Reference

These are all the YAML frontmatter variables that JTD-RMW recognizes on individual pages. They can be set in the frontmatter block at the top of any page or post.

### Layout Selection

**`layout`** — Which layout template to use for this page.
- `post` — The recommended layout for most content pages. Displays the title, subtitle, dates, author, and an optional table of contents above the content.
- `default` — The base layout with sidebar navigation, search, breadcrumbs, and footer. No title header or date line — just the raw content.
- `page` / `home` / `about` — Identical passthrough layouts that extend `default`. They exist mainly for semantic naming and backward compatibility.
- `remark_slides` — Renders the page as a Remark.js slideshow instead of a normal page. Does not include the sidebar or any JTD chrome.
- `null` — Outputs raw content with absolutely no wrapping HTML. Useful for embedding pre-built HTML files.

### Title and Metadata

**`title`** — The page title. Used in the browser tab (`<title>` tag), the sidebar navigation, breadcrumbs, the parent-child TOC, the search index, and Open Graph meta tags. If `layout: post` is used, it is also rendered as an `<h1>` heading at the top of the page.

**`subtitle`** — A subtitle displayed beneath the title in the `post` layout's `<hgroup>` header. Also shown in the parent-child TOC listing for this page.

**`description`** — A short description of the page. Used in the HTML `<meta name="description">` tag and the Open Graph `og:description` property for SEO and social sharing. Also displayed in the parent-child TOC listing beneath the subtitle.

**`author`** — The author of the page. Displayed in the date header line of the `post` layout (e.g., "by Robert Winslow").

**`imagepreview`** — A URL to an image for social media previews. Sets the `og:image` Open Graph meta property. Should be a full URL.

### Dates

**`date`** — The original publication date of the page. Displayed in the `post` layout's date header and in the parent-child TOC. Use `YYYY-MM-DD` format.

**`last_modified_date`** — The date the page was last modified. Displayed in the `post` layout's date header, in the parent-child TOC, and in the footer's "Page last modified" timestamp (the footer uses the `site.last_edit_time_format` Ruby time format string). Use `YYYY-MM-DD` format.

**`modified`** — An alias for `last_modified_date`, added as a convenience because the full parameter name is easy to forget. The `post` layout checks for `last_modified_date` first and falls back to `modified`. Note: the footer in `default.html` only checks `page.last_modified_date`, not `modified`, so using `modified` alone will show the date in the post header but not in the footer.

### Navigation

**`parent`** — The title of the parent page in the navigation hierarchy. Setting this nests the page under the parent in the sidebar and generates a breadcrumb trail. The value must exactly match the `title` of the intended parent page.

**`grand_parent`** — The title of the grandparent page. Used for three-level nesting in the sidebar navigation and breadcrumbs. Must match the `title` of the grandparent page.

**`great_grand_parent`** — The title of the great-grandparent page. Only used in `nav_details.html` to determine whether to expand the navigation tree to show the current page. Does not appear in breadcrumbs (which only go two levels deep).

**`has_children`** — Set to `true` to indicate that this page has child pages. This controls whether the navigation renders an expandable `<details>` group for this page's children and whether the parent-child TOC appears at the bottom of the page.

**`has_toc`** — Set to `false` to suppress the automatically generated list of child pages at the bottom of a parent page. Only relevant when `has_children` is `true`. Defaults to `true`.

**`nav_order`** — Controls the sort position of this page in the sidebar navigation. Can be a number or a string. Numeric values are sorted first (by value), followed by string values (sorted alphabetically). If omitted, the page's `title` is used as the sort key.

**`nav_exclude`** — Set to `true` to hide this page from the sidebar navigation and from parent-child TOC listings. The page is still rendered and accessible by direct URL; it is just invisible in the navigation menus.

### Search

**`search_exclude`** — Set to `true` to exclude this page from the Lunr search index. The page will not appear in search results, though it remains accessible by URL and visible in the navigation (unless also `nav_exclude`d).

### Math Rendering

**`math`** — Controls which math rendering engine is loaded for this page. The resolution order is: page frontmatter → layout frontmatter → `site.math` in `_config.yml` → `"none"` (the hardcoded default).
- `"katex"` — Loads KaTeX with auto-render. Supports `$...$` (inline), `$$...$$` (display), `\(...\)` (inline), and `\[...\]` (display).
- `"mathjax"` — Loads MathJax 4 beta. Supports `$...$` (inline) and `\(...\)` (inline).
- `"none"` (default, or any other unrecognized value) — No math renderer is loaded. Use this (or simply omit the setting) on pages that don't need math.

### Table Wrapping

**`table_wrappers`** — Controls how `<table>` elements on the page are wrapped for responsive display. Processed by `table_wrappers.html`.
- *(default / omitted)* — Tables are wrapped in `<div class="table-wrapper">`, which provides horizontal scrolling on narrow screens and limits table width.
- `"wide"` — Tables are wrapped in `<div class="table-wrapper-wide">`, which spans the full page width without constraining the table.
- `false` — Tables are not wrapped at all. Bare `<table>` elements.

### Table of Contents (Post Layout)

**`toc`** — Set to `true` to display a collapsible table of contents at the top of the page in the `post` layout. The TOC is generated from heading elements in the rendered HTML using the `toc.html` include (jekyll-toc). Only works with `layout: post`.

## Site Configuration Reference (`_config.yml`)

These are the `_config.yml` settings that JTD-RMW's templates read. Consuming sites set these in their own `_config.yml`.

| Setting | Description |
|---------|-------------|
| `title` | The site title. Displayed in the sidebar header and browser tab. |
| `description` | Site description (not currently used in templates, but available to plugins). |
| `author` | Site author. Used in the `<meta property="author">` SEO tag. |
| `url` | The base URL of the site (e.g., `https://www.rmwinslow.com`). Used for canonical URLs and Open Graph. |
| `baseurl` | The subpath of the site (e.g., `/blog`). Used in URL construction. |
| `logo` | Path to a logo image. If set, the sidebar header displays this image instead of the site title text. |
| `search_enabled` | Set to `true` to enable the Lunr search bar. Defaults to `true`. |
| `search.heading_level` | The heading depth to split pages into searchable sections (1–6, default 2). |
| `search.previews` | Number of preview snippets per search result (default 3). |
| `search.preview_words_before` | Words to show before a search match in the preview (default 5). |
| `search.preview_words_after` | Words to show after a search match in the preview (default 10). |
| `search.tokenizer_separator` | Regex for splitting search tokens (default `/[\s\-/]+/`). |
| `search.rel_url` | Whether to display relative URLs in search results (default `true`). |
| `search.button` | If `true`, shows a floating search button in the bottom-right corner. |
| `heading_anchors` | If `true` (default), headings get clickable `¶` anchor links. |
| `nav_sort` | `"case_sensitive"` (default) or `"case_insensitive"` for sidebar nav sorting. |
| `nav_external_links` | A list of `{title, url}` objects for external links in the sidebar. |
| `back_to_top` | If `true`, shows a "Back to top" link in the page footer. |
| `back_to_top_text` | The text for the back-to-top link (default `"Back to top"`). |
| `footer_content` | HTML string displayed at the bottom of every page. |
| `last_edit_timestamp` | If `true`, shows "Page last modified: ..." in the footer. |
| `last_edit_time_format` | Ruby time format string for the last-modified timestamp. |
| `gh_edit_link` | If `true`, shows an "Edit this page on GitHub" link in the footer. |
| `gh_edit_link_text` | The text for the edit link. |
| `gh_edit_repository` | The GitHub repository URL for edit links. |
| `gh_edit_branch` | The branch name for edit links. |
| `gh_edit_source` | An optional subdirectory for the source files. |
| `gh_edit_view_mode` | `"tree"` or `"edit"` — whether the GitHub link opens the file viewer or the editor. |
| `color_scheme` | The color scheme name. Used by the SCSS pipeline to select a color scheme file. |
| `ga_tracking` | Google Analytics tracking ID (e.g., `UA-XXXXXXX-XX`). If set, includes the GA script. |
| `ga_tracking_anonymize_ip` | If `true`, enables GDPR-compliant IP anonymization in GA. |
| `math` | The default math renderer for the site: `"katex"`, `"mathjax"`, or `"none"`. Can be overridden per-page. |
| `webfontdirectory` | A URL to a directory of web fonts (used by some consuming sites, not directly by the theme templates). |
| `compress_html.ignore.envs` | Controls the HTML compression layout. Set to `all` to disable compression. |

## Potential Issues

These are things noticed during the code review. Tagged to distinguish them from user-created TODOs.

- **[claude's suggestion]** The original JTD Sass pipeline was manually removed, but some remnants survive: `_includes/css/just-the-docs.scss.liquid` (the SCSS entry point — references `_sass/` modules that no longer exist), `_includes/css/custom.scss.liquid` (the custom SCSS hook), and the `just-the-docs.gemspec` (still references `_sass` in its file glob on line 13). The `_sass/` directory itself is gone, so the pipeline would fail if triggered. The ~30 `!important` flags on `font-size` declarations throughout `compiled-jtd-style.css` are also vestigial — they were likely needed to win specificity battles against the SCSS output, but since the SCSS pipeline is dead, they serve no purpose and make it harder for consuming sites to override font sizes.
- **[claude's suggestion]** `nav.html` is never included anywhere — `default.html` uses `nav_details.html` instead. The file could be removed, or kept as an alternate navigation style that consuming sites could swap in via an override.
- **[claude's suggestion]** The `modified` alias only works in the `post` layout's date header. The footer timestamp in `default.html` (line 136) only checks `page.last_modified_date`, so a page using `modified:` alone will not show a "Page last modified" footer. Consider adding a fallback in the footer as well, or documenting this caveat prominently.
- **[claude's suggestion]** The `remark_slides.html` layout loads KaTeX v0.16.7, while `katex.html` uses the newer v0.16.22. These should probably be kept in sync.
- **[claude's suggestion]** The `mathjax.html` include loads MathJax v4.0.0-beta.4, which is a beta release. MathJax 4 has since been released. Consider updating to a stable version.
- **[claude's suggestion]** `_config.yml` still has the original JTD author's `url`, `baseurl`, `ga_tracking`, `footer_content`, and `gh_edit_repository` values. While consuming sites override these, having stale defaults could cause confusion if someone builds the theme repo directly (e.g., for testing).
- **[claude's suggestion]** The `great_grand_parent` frontmatter variable is checked in `nav_details.html` (line 63) to auto-expand the sidebar, but it is not used in the breadcrumb logic in `default.html`, which only handles two levels (`parent` and `grand_parent`). If a four-level hierarchy is intentional, breadcrumbs could be extended to match.
- **[claude's suggestion]** `head.html` sets the `<title>` to `{{ page.title }} - {{ site.title }}`. If a page has no `title`, this produces ` - Site Title` with a leading space and dash. A conditional fallback might be worth adding.

## Writing Style Guidelines

- **Use natural language in all commit messages, PR descriptions, comments, and documentation.** Write in complete sentences. Do not drop articles ("a", "the", "an") or use telegraphic shorthand. For example, write "Add the missing KaTeX stylesheet to the post layout" rather than "add missing KaTeX stylesheet post layout."
- Keep commit messages concise but grammatically complete.

## Theme TODOs

- [x] Remove the dead Sass pipeline remnants: deleted `_includes/css/just-the-docs.scss.liquid` and `_includes/css/custom.scss.liquid`, and removed the `_sass` reference from `just-the-docs.gemspec`. The `$logo` Sass variable that `just-the-docs.scss.liquid` set was replaced with a direct `<img>` tag in `title.html`.
- [ ] Audit the ~30 `!important` flags on `font-size` declarations in `compiled-jtd-style.css` — they may be fighting internal specificity battles within the file, not just the dead Sass pipeline. Each one needs to be checked individually before removal.
- [ ] Examine whether the `width: 0px` scrollbar trick (commented out in `kineticgraphs.css:14-17`) is used anywhere in the consuming sites, and whether it should be removed or restored.
- [ ] Index the differences between JTD-RMW and `just-the-docs-tweaked` to understand what would be involved in migrating the sites that still use the older fork (`3102old`, `tones`).
- [ ] Deprecate and remove the `remark_slides` layout. The Remark.js slide engine required too many workarounds (backslash escaping, underscore conflicts with filenames, separate KaTeX version). The two pages that use it in `RMWinslow.github.io` (`3102/measurement-money-slides` and `slides/remark_formattest`) need to be either deleted or migrated to the standalone `slides` repo.
- [ ] Remove `great_grand_parent` support from `nav_details.html`. It is never used by any page across all five sites, and the breadcrumbs in `default.html` don't support it anyway. The deepest hierarchy in practice is three levels.
- [ ] Remove or repurpose `nav.html`. It's unused — `default.html` uses `nav_details.html`. Either delete it or document it as an alternative that consuming sites can swap in.
- [ ] Consolidate identical layouts. `page.html`, `home.html`, and `about.html` are byte-for-byte identical. Consider whether they should all just be symlinks or whether the semantic distinction is worth maintaining.
- [ ] Sync KaTeX versions. `katex.html` uses v0.16.22 but `remark_slides.html` has v0.16.7 hardcoded inline.
- [ ] Update MathJax. `mathjax.html` loads a beta (v4.0.0-beta.4). MathJax 4 has since had stable releases. Note: v4 beta was chosen because MathJax v3 doesn't support line breaks. Before upgrading, verify that stable v4 releases still support line breaks.
- [x] Extend `modified` fallback to the footer. The `post` layout handles `modified` as an alias for `last_modified_date`, but the footer in `default.html` didn't. Fixed with a `{% assign footer_modified = page.last_modified_date | default: page.modified %}` in the footer block.
- [ ] Consider deprecating `last_modified_date` in favor of the shorter `modified`. Investigate whether we can reassign the variable early in the processing chain (e.g., in `default.html` or a Liquid `assign`) so downstream code only needs to check one name. Currently the footer requires `page.last_modified_date` specifically. If we settle on `modified`, Claude could do a batch replacement of `last_modified_date` across all five consuming sites' frontmatter.
- [x] Guard the `<title>` tag. Add a conditional so pages without a `title` get just the site title instead of ` - Site Title`.
- [ ] Clean up `_config.yml` defaults. The theme's own config still has the original JTD author's URLs, GA tracking ID, and footer content. These are overridden by consuming sites but are misleading if someone builds the theme repo directly for testing.
- [ ] Consider making `extrabits.css` and `kineticgraphs.css` opt-in via config rather than always-on. `extrabits.css` is loaded but empty; `kineticgraphs.css` is loaded on every site but only relevant to sites using kgjs.
- [ ] Verify color contrast for accessibility. The light mode's `--textcolor: #55525B` on `--basecolor: #fdf6e3` should be checked against WCAG AA standards.
- [x] Changed the default math renderer to `"none"`. KaTeX is now only loaded on pages that explicitly set `math: katex` or on sites with `math: katex` in `_config.yml`. Added `math: katex` to all posts pages that use math, and set `math: katex` site-wide on `bib`.
- [ ] Fix the double KaTeX conflict. A few pages in `RMWinslow.github.io` use kgjs (which bundles its own KaTeX) and stutter when the theme also loads KaTeX. The default-off change reduces the risk, but those specific pages should either set `math: none` or the theme's KaTeX loading should detect an existing KaTeX instance before initializing.
- [ ] Consider adding `page_excerpts` support to the theme, or document that consuming sites should remove the setting. It is set on some sites but the theme never reads it.
- [ ] Consider adding a unified `hidden` frontmatter variable that sets both `nav_exclude` and `search_exclude` in one go. Currently, hiding a page from both navigation and search requires remembering to set two separate flags, which is easy to forget.
- [ ] Consider deprecating the dark mode CSS theme or overhauling the color scheme in `colorscheme.css`.
- [ ] Consider whether to replace the ¶ heading anchors with the `svg-link` icon (original JTD approach). Pro: SVG icons aren't included when users copy-paste page text, while the ¶ character is. Con: ¶ is pure text, degrades gracefully without CSS, and is lighter. If we keep ¶, the `svg-link` symbol definition in `default.html` can be removed.
- [ ] Make the mobile hamburger menu work without JavaScript. Currently the sidebar is hidden at narrow viewport widths and only revealed by JS adding `.nav-open` on click. A CSS-only approach (e.g., a `<details>`/`<summary>` wrapper or a checkbox hack) would let the site degrade gracefully to pure HTML+CSS. The navigation tree itself already uses `<details>` elements and doesn't need JS. The only other thing `just-the-docs.js` does besides the hamburger toggle is power the Lunr search UI, which degrades silently (the search input just does nothing).
- [ ] Replace the `scaleX(-1)` double-flip hack for the nav disclosure triangles with a proper `::after` approach. The current hack (`compiled-jtd-style.css:1005-1020`) flips the entire `<summary>` horizontally to move the marker to the right, then flips the child link back. This is fragile and unintuitive. The replacement should use `::after` with absolute positioning. Needs a cross-browser test page first (see below).
- [ ] Set up Playwright MCP for browser testing. Install via `claude mcp add playwright npx @playwright/mcp@latest --headless`. Would allow Claude to navigate to locally-served Jekyll pages, take screenshots, and read console errors. Ruby/Jekyll are not currently installed on this machine and would also need to be set up.
- [ ] Make nav menu titles truncate with a fade-out instead of wrapping to a new line. This will make the nav sidebar visually cleaner and more compact, especially as sections with longer page titles are added.
- [ ] Investigate adding a custom kramdown parser plugin that recognizes `$...$`
  as inline math (protecting content from markdown parsing). Kramdown only
  recognizes `$$` natively. A working example exists in kramdown issue #672.
  This would require switching consuming sites from GitHub Pages' default build
  to a GitHub Actions workflow, since GitHub Pages runs in safe mode and blocks
  custom plugins.
- [ ] Build a `<details>`/`<summary>` CSS test page to nail down the disclosure triangle approach before changing the nav. The test page should cover: (1) native marker behavior across browsers, (2) removing the marker with `list-style: none` + `::-webkit-details-marker { display: none }`, (3) `::after` replacement with absolute positioning, (4) `::after` with `transform: rotate()` animation on open, (5) `::marker` with `content` override (works in Chrome/Firefox, not Safari). This would be a standalone HTML file for manual testing, not part of the theme.

## Extended Notes

The cross-site audit results (navigation sitemaps, broken links, orphaned files, consuming sites inventory, and site-specific TODOs) are in `claude_audits.md`. That file is intended to be moved to the `RMWinslow.github.io` repo since most of its content concerns the consuming sites rather than the theme itself.

### Details/Summary Disclosure Triangle Research (2026-03-17)

The nav menu's disclosure triangles currently use a `scaleX(-1)` double-flip hack to put the marker on the right side. The user previously tried `::after` pseudo-elements but they didn't work correctly. Research findings:

**How browsers implement the disclosure triangle:**
- **Firefox:** Uses `::marker` with `list-style-type: disclosure-closed/disclosure-open`. Removing it: `list-style: none`.
- **Chrome:** Migrated from `::-webkit-details-marker` to `::marker` in Chrome 89. Both removal methods work.
- **Safari:** Uses internal shadow DOM, only addressable via `::-webkit-details-marker`. Does **not** support `::marker` on `<summary>`. `list-style: none` alone does **not** remove the triangle.

**Cross-browser marker removal requires both rules:**
```css
summary { list-style: none; }
summary::-webkit-details-marker { display: none; }
```

**Key gotchas for `::after` replacement:**
- Safari does not support `display: flex` on `<summary>` itself — flexbox layout is ignored. Workaround: wrap summary text in a child `<span>` and flex that, or use absolute positioning on the `::after` instead.
- If you only use `list-style: none` without the `::-webkit-details-marker` rule, Safari shows both the native triangle and the `::after` — a double indicator.
- `::marker` only allows a very restricted set of CSS properties: `content`, `color`, `font-*`, `white-space`, `direction`, `unicode-bidi`. No `background`, `width`, `height`, `transform`, or `position`.
- Removing the native marker breaks screen reader state announcements in Firefox + VoiceOver (the triangle direction is the only state indicator).

**Recommended replacement approach:** Remove marker with both rules, use `::after` with `position: absolute`. Use `details[open] > summary::after` to swap the character or apply `transform: rotate(90deg)` for animation. Avoid flexbox directly on `<summary>`.

### Theme JavaScript Inventory (2026-03-17)

`just-the-docs.js` does exactly three things:
1. **Hamburger toggle** (`initNav`) — toggles `.nav-open` class on click. Only JS-dependent feature that affects usability.
2. **Lunr search** (`initSearch`) — fetches `search-data.json` via XHR, builds the Lunr index, renders results, handles keyboard navigation. Degrades silently (search input does nothing without JS).
3. **Theme switcher API** (`jtd.getTheme`/`jtd.setTheme`) — global functions for swapping CSS files at runtime. Never called by the theme itself; just a hook for consuming sites that want a theme picker.

`_includes/js/custom.js` is empty. The nav tree collapsing is pure `<details>`/`<summary>` — no JS involved.

**External resources loaded by the `default` layout:** All CSS and JS is self-hosted. The only external resources are conditional: Google Analytics (if `site.ga_tracking` is set), KaTeX from jsDelivr (if `math: katex`), and MathJax from jsDelivr (if `math: mathjax`). No external fonts.

## Session Log

Keep this section updated with what was accomplished in each Claude session.

### 2026-03-18

- Completed three theme fixes in a single commit. First, removed the dead Sass pipeline by deleting `just-the-docs.scss.liquid` and `custom.scss.liquid`, and by removing the `_sass` reference from the gemspec. Second, fixed the broken site logo feature: the old approach set a Sass `$logo` variable that was supposed to be compiled into a CSS `background-image`, but since the Sass pipeline was dead, the `<div class="site-logo">` in `title.html` was always empty. Replaced it with a straightforward `<img>` tag. Third, extended the `modified` frontmatter fallback to the footer in `default.html`, so that pages using the shorter `modified:` key get a "Page last modified" timestamp in the footer, matching what `post.html` already does.
- Added `width="24" height="24"` attributes to all of the inline SVG icon references in `default.html` and `just-the-docs.js`. Without these, the SVGs expand to fill the entire viewport when the stylesheet isn't loaded — the magnifying glass icon was the giant black circle visible in the screenshot.
- Discovered that the `svg-link` symbol is defined in `default.html` but is never actually referenced anywhere in the theme. The original Just the Docs theme used it for heading anchor links, but this fork uses the ¶ paragraph character instead. Added a TODO weighing the tradeoffs: SVG icons don't appear when users copy-paste page text, but the ¶ character degrades gracefully without CSS and is simpler.
- Corrected the `!important` audit TODO. The earlier assumption was that the ~30 `!important` flags on `font-size` declarations in `compiled-jtd-style.css` were vestigial leftovers from the dead Sass pipeline. But that reasoning was wrong — the compiled CSS file *is* the output of the Sass pipeline, so those flags may be fighting internal specificity battles within the same file. Each declaration needs to be checked individually against competing rules before it can safely be removed.
- Added context to the MathJax update TODO. The commit message from `a90ea63` explains that the v4 beta was chosen because MathJax v3 doesn't support line breaks. Before upgrading to a stable v4 release, we need to verify that line break support is still present in the stable version.
- Marked the `<title>` guard TODO as complete — it was already implemented in commit `ee1eeb7`.

**Local development pipeline:**

- Installed Playwright for Python (`pip install playwright && playwright install chromium`). This gives us a headless Chromium browser that Claude can use to take screenshots of web pages without any interactive browser session. The command is `python -m playwright screenshot --full-page <url> <output.png>`, and Claude can read the resulting image files natively. The Chromium binary is cached in `AppData\Local\ms-playwright\` — no project folder structure is needed.
- Installed Ruby and Jekyll inside WSL2 Ubuntu. Created a shared `Gemfile` in the parent `GitHub/` directory that uses the `github-pages` gem (v232, which pins Jekyll to version 3.10 to match GitHub's actual build servers). Also created a `_config_dev.yml` overlay in the same directory that sets `url: http://localhost:4000`. The gems are installed into `GitHub/.bundles_cache/`.
- Successfully served the circe site locally by running `bundle exec jekyll serve --config _config.yaml,../_config_dev.yml` from within the circe directory in WSL. Had to add `jekyll-remote-theme` to circe's `_config.yaml` plugins list — GitHub Pages injects this plugin automatically, but local Jekyll requires it to be listed explicitly. Adding it to the config is harmless on GitHub's side since the plugin is already in their whitelist.
- Tested the full pipeline end-to-end: Jekyll serves a consuming site in WSL, Playwright takes a headless screenshot of `localhost:4000` from Windows, and Claude reads the resulting image. This works well for verifying how the sites look after theme changes.
- **Key limitation:** Claude cannot independently start or restart the Jekyll server. WSL commands that require `sudo` or that run as long-lived processes don't work from Claude's non-interactive bash shell. The practical workflow is: Claude edits files, the user starts or restarts the Jekyll server in their own terminal, and then Claude can screenshot `localhost:4000` to verify the results. For standalone HTML/CSS prototypes (like the planned disclosure triangle test page), Claude can serve them independently using `python -m http.server` on the Windows side and take screenshots without needing the user's help.

### 2026-03-17

- Fixed the image div in `circe/index.md` to use `class="aside-like" style="max-width: 280px;"` instead of the old inline float, so the title page image stays small on mobile.
- Changed the theme's hardcoded math default from `"katex"` to `"none"` in `head.html`. Updated the README and CLAUDE.md to reflect this.
- Added `math: katex` to `bib/_config.yml` (site-wide, since bib is math-heavy).
- Added `math: katex` to all nine `posts` pages that use KaTeX: `econ/timeusechange.md`, `markdown.md`, `numbers/horizon.md`, the five `nature/birdup/` scoring pages, and the `_drafts/special-relativity.md` draft.
- Noted that kgjs is only used on a few pages in `RMWinslow.github.io`, not on the games site as previously assumed.
- Verified that all 9 math-using pages in the `posts` repo have `math: katex` in their frontmatter. No files were missed.
- Audited all JavaScript and external assets loaded by the theme. Documented the full inventory in Extended Notes.
- Researched `<details>`/`<summary>` disclosure triangle styling across browsers (Safari's `::-webkit-details-marker` vs. Firefox/Chrome's `::marker`, flexbox-on-summary bugs, accessibility implications of marker removal). Documented findings in Extended Notes.
- Identified that the `scaleX(-1)` double-flip hack in the nav CSS should be replaced with a `::after` + absolute positioning approach. Added TODOs for building a standalone HTML test page and doing the replacement.
- Added TODO for making the mobile hamburger menu work without JS.
- Next step: build a standalone HTML test page with several disclosure triangle CSS approaches for manual browser testing, then use results to replace the scaleX hack in the nav.

### 2026-03-16 (session 2)

- Scanned all five active JTD-RMW sites for files not in the navigation trees. Found 0 orphans in circe, 1 in bib, 8 in games, 35 in posts, and 110 in RMWinslow.github.io (mostly legacy pre-Jekyll HTML).
- Split theme-specific TODOs and audit ideas into CLAUDE.md; left cross-site audit results in `claude_audits.md` for the user to move to the github.io repo.
- Added `nav_exclude`/`search_exclude` frontmatter to the audit file.
- Added a TODO to consider a unified `hidden` frontmatter variable.

### 2026-03-16 (session 1)

- Created this CLAUDE.md file to establish project conventions and provide context for future sessions.
- Began indexing the repo structure and understanding the theme's custom features.
- Indexed all public repos at https://github.com/RMWinslow for GitHub Pages usage and theme references.
- Thoroughly documented the entire repo: every layout, include, CSS file, JS file, and asset, with natural language descriptions.
- Documented all page frontmatter variables and site configuration settings with full descriptions and behavior details.
- Identified several potential issues (KaTeX version mismatch between layouts, unused `nav.html`, `modified` alias inconsistency, stale `_config.yml` defaults, etc.).
- Audited layout usage across all five active JTD-RMW sites.
- Built full navigation sitemaps for all five sites and identified 11 hierarchy issues (orphaned parents, phantom parents, duplicate titles, missing HC flags, missing grand_parents).
- Scanned all five sites for broken internal links. Found 0 in circe, 4 in bib, 3 in games, 11 in posts, and 31 in RMWinslow.github.io (mostly a systematic sakura.css relative path issue in older HTML files).

## Notes and Preferences

- Track all project context, session history, and preferences here in CLAUDE.md rather than in hidden memory files. This file should be the single source of truth for cross-session continuity.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RMWinslow)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RMWinslow)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
