## hexo-theme-cosmos

> Cosmos is an editorial Hexo theme for personal blogs, with a cover or article-list homepage, archives, taxonomy pages, and a configurable About page. The source repository is [cweiai/hexo-theme-cosmos](https://github.com/cweiai/hexo-theme-cosmos). It uses EJS templates, CommonJS helpers, plain CSS, and browser JavaScript. There is no frontend framework or asset compilation step.

# Agent Guide for Cosmos

## Overview

Cosmos is an editorial Hexo theme for personal blogs, with a cover or article-list homepage, archives, taxonomy pages, and a configurable About page. The source repository is [cweiai/hexo-theme-cosmos](https://github.com/cweiai/hexo-theme-cosmos). It uses EJS templates, CommonJS helpers, plain CSS, and browser JavaScript. There is no frontend framework or asset compilation step.

This guide applies throughout the repository. **MUST** means required, **SHOULD** means the default unless there is a concrete reason to depart, and **MAY** means optional. Follow explicit task instructions and applicable higher-priority instructions; surface a conflict instead of silently ignoring it.

## Read before working

Agents **MUST** read the root [README](README.md) and [contribution guide](CONTRIBUTING.md) before changing files. Read the relevant module guide and the sources listed below before editing that area. The contribution guide is the shared source for [code conventions](CONTRIBUTING.md#code-conventions), [commit conventions](CONTRIBUTING.md#commit-conventions), and [packaging](CONTRIBUTING.md#package-the-theme).

## Main Interfaces / Implementations

| Work area | Required reading and source of truth |
| --- | --- |
| Theme settings | [Configuration guide](docs/configuration.md), [_config.yml](_config.yml), [settings helpers](lib/README.md), and [reference generator](tools/reference.cjs) |
| Templates and page generation | [Layout guide](layout/README.md), [Hexo integration guide](scripts/README.md), and the relevant EJS templates/helpers |
| Articles, About, and custom pages | [Content guide](docs/content.md), [examples](examples/README.md), and [presets](presets/README.md) |
| Appearance and browser behavior | [Source guide](source/README.md), [style.css](source/css/style.css), and the affected templates/scripts |
| Translation and assets | [Language guide](languages/README.md), [icon guide](assets/social-icons/README.md), and [third-party notices](THIRD_PARTY_NOTICES.md) |
| Documentation | [Documentation guide](docs/README.md) and the corresponding English/Chinese document pair |
| Tests, installation, and distribution | [Test guide](test/README.md), [tool guide](tools/README.md), [automation guide](.github/AUTOMATION.md), [usage guide](docs/usage.md), and [package.json](package.json) |

## Art direction

Cosmos should feel like a warm printed journal: quiet, spacious, typographic, and focused on reading. Its character comes from asymmetric composition, large headings, thin rules, and generous whitespace. The About sidebar supports this editorial layout.

| Element | Default treatment |
| --- | --- |
| Paper and ink | Warm paper `#f4eedf`, dark brown ink `#342c24`, muted text `#6c6052` |
| Accents and rules | Terracotta `#a6402c`, honey `#eccc77`, fine rules `#d4c7b1` |
| Body and interface | Locally bundled Hanken Grotesk, with Chinese system-font fallbacks |
| Display accents | Locally bundled Petrona serif and italic, with Chinese serif fallbacks |
| Cover | Typography by default; optional static artwork supplied by the blog owner |

- Agents **MUST** preserve the default visual identity unless a task explicitly changes it. Extend the existing tokens and configuration before adding independent visual systems.
- Agents **SHOULD** favor readable text, deliberate spacing, and restrained transitions. Avoid unsolicited dashboard cards, glass effects, neon palettes, decorative animation, or remote font dependencies.
- Visual changes **MUST** retain responsive layouts, visible keyboard focus, semantic controls, and reduced-motion behavior. Primary navigation and article reading **MUST** remain usable without JavaScript; search and other enhancements may require it.
- New examples **MUST** use fictional or neutral content. Personal identities, live blog settings, and account details do not belong in theme defaults.

## Code and behavior contracts

- Agents **MUST** follow [.editorconfig](.editorconfig) and the contribution guide's code conventions. Keep changes focused and preserve neighboring style.
- Configuration **MUST** preserve the documented precedence: theme defaults, blog `_config.cosmos.yml`, then inline `theme_config`; About data and page overrides apply afterward. Objects merge recursively; arrays replace, including `[]`. Preserve meaningful `false` and empty-string overrides.
- New or changed settings **MUST** update `_config.yml`, both guides, and relevant examples/tests. Run `npm run docs:reference` to regenerate the marked default-table sections and [JSON schema](docs/theme.schema.json); do not hand-edit generated content.
- EJS **MUST** escape plain text and attributes with `<%=`. Use `<%-` only for intentionally rendered content, trusted HTML extension slots, or helpers that return safe markup. Reuse the existing URL, path, JSON, and CSS handling in [lib/settings.cjs](lib/settings.cjs) and [scripts/theme.js](scripts/theme.js).
- Routes and assets **MUST** work under a Hexo subdirectory root. Preserve source-page precedence and explicit route-conflict errors rather than silently replacing user content.
- About custom HTML **MUST** retain its documented precedence: a nonempty `sidebar_html` replaces the configured profile even when that profile is disabled.
- UI text **MUST** keep matching English/Chinese locale keys and honor configured label overrides. Do not translate or rewrite blog-authored content.
- Browser code **SHOULD** remain small and progressively enhance the static output. A new framework, build step, dependency, or external service requires a demonstrated task need.

### Hexo-specific traps

- `scripts/README.md` and its Chinese edition **MUST** retain their outer JavaScript block-comment wrappers. Hexo's script loader evaluates files in that directory; ordinary unwrapped Markdown breaks startup.
- Theme-owned `source/README*.md` files **MUST** stay excluded from generated pages. Do not suppress a blog owner's own README pages when changing that filter.
- `examples/source/` contains content fixtures, not project documentation. Files under `_content/` are fragments; ordinary source pages support Hexo tags, while configured content fragments are rendered directly.
- `post.comments` controls the article-end HTML slot; it does not install a comment provider. Documentation **MUST NOT** claim built-in integrations that the runtime does not supply.

## Documentation boundaries

- User and contributor Markdown documents **MUST** have English-default and Simplified Chinese editions (`name.md` / `name.zh-CN.md`), direct language switches, and matching structure. This explicitly English-only `AGENTS.md` is exempt. Original third-party licenses, the canonical JSON schema, and sample blog posts are not translation pairs.
- The root README **MUST** stay focused on the theme, screenshots, quick installation, and direct links. Configuration details belong in [docs/configuration.md](docs/configuration.md). Keep related instructions together and link directly to the relevant document or section.
- Documentation **MUST** remain Markdown files. Do not introduce a documentation site, extra navigation layers, or personal preview/push shortcuts without an explicit request. Shared tests, CI, and packaging tools belong in contributor documentation.
- Agents **MUST** update both affected module READMEs when interfaces or behavior change, retaining their overview and interface table. Agents **MAY** omit README edits for internal changes that leave the documented contract intact.
- The `.github` module **MUST** use `AUTOMATION.md` and `AUTOMATION.zh-CN.md` instead of README files: GitHub gives `.github/README` precedence over the root README on the repository homepage.
- Do not maintain or recreate a CHANGELOG file. Describe user-visible changes and migration steps in the pull request and the relevant user guides.
- Claims and commands **MUST** match the actual package, scripts, defaults, and workflows. Repository URLs alone do not establish that a release, hosted demo, or npm publication exists.

## Verification and delivery

Use Node.js 24 and `npm ci` for repository work. The declared runtime supports Node.js 20+ and Hexo 7–8; CI explicitly exercises Node 20/22/24 and clean installations with Hexo 7.3.0/8.1.2.

- Agents **MUST** run `npm run check` after code or documentation changes. Dependency, installation, packaging, or generator changes also require `npm run check:package`; use `COSMOS_HEXO_VERSION=7.3.0 npm run check:package` when checking the other supported Hexo major on a POSIX shell.
- Tests **SHOULD** exercise meaningful behavior and regressions. Visual changes require desktop/mobile inspection and appropriate screenshots, as described in the contribution guide. Report any check that could not be completed.
- Packaging changes **MUST** keep agent/contributor instructions, module READMEs, tests, examples, and tooling out of the installation archive while retaining public guides and all required licenses. Review [tools/package.cjs](tools/package.cjs) and [tools/check-package.cjs](tools/check-package.cjs).
- Workflows perform validation, and `npm run theme:pack` creates local archives. Agents **MUST NOT** add release automation without an explicit request. Agents **MUST NOT** publish, push, or create commits unless the task authorizes those actions; when committing, follow the required conventional English format in the contribution guide.
- Agents **MUST** inspect the actual working tree and preserve unrelated changes. Do not assume that a local directory has Git metadata or the expected remote.
- Final reports **SHOULD** state the resulting behavior, relevant verification, and remaining limitations. Do not present local checks as hosted CI results or generated artifacts as published releases.

---
> Source: [cweiai/hexo-theme-cosmos](https://github.com/cweiai/hexo-theme-cosmos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
