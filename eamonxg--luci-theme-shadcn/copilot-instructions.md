## luci-theme-shadcn

> All dev commands run from `.dev/`:

# CLAUDE.md

## Commands

All dev commands run from `.dev/`:

```bash
cd .dev/
cp .env.example .env   # set VITE_OPENWRT_HOST / VITE_OPENWRT_SSH_HOST for proxy + .ut sync
pnpm dev      # Vite dev server (proxies LuCI to the router; optional scp of *.ut on save)
pnpm build    # Clean + build production assets to htdocs/luci-static/
pnpm clean    # Remove build output only
```

`VITE_OPENWRT_HOST` defaults to `http://192.168.1.1`. Set `VITE_OPENWRT_SSH_HOST` (e.g. `root@192.168.1.1`) to push `ucode/template/themes/shadcn/*.ut` to `/usr/share/ucode/luci/template/themes/shadcn/` when a template file changes; leave empty to disable. Optional `VITE_OPENWRT_SSH_KEY` for a dedicated private key.

No test suite or linter CLI. Prettier with format-on-save.

## Architecture

**Dual-layer build**: source in `.dev/` → OpenWrt-compatible output in `htdocs/luci-static/`.

- `.dev/src/media/main.css` → `htdocs/luci-static/shadcn/main.css`
- `.dev/src/media/login.css` → `htdocs/luci-static/shadcn/login.css`
- `.dev/src/resource/*.js` → `htdocs/luci-static/resources/*.js`
- `.dev/public/shadcn/` → `htdocs/luci-static/shadcn/`
- `ucode/template/themes/shadcn/*.ut` — server-side templates, not processed by Vite

## CSS

Single entry `main.css`. All styling via TailwindCSS v4 `@apply`. CSS Nesting for selectors. Tokens in `_tokens.css` as OKLCH custom properties. Dark mode via `@custom-variant dark` keyed on `[data-darkmode=true]`.

## Design Baseline

See root `CLAUDE.md` for full rules: shadcn style, Inter font (Linear's typeface via @fontsource-variable/inter), Lucide icons, no hardcoded colors.

## Sidebar Layout

- `header.ut`: minimal shell — empty `#sidebar` (like material `#mainmenu`); sidebar chrome + nav built in `menu-shadcn.js`
- `sidebar-shadcn.js`: state machine (collapse/expand, accordion, mobile drawer, theme toggle) — loads after `shadcn-sidebar-ready`
- `menu-shadcn.js`: resolve `admin` branch then two-level sidebar; `ICON_MAP` maps LuCI node `name` to `/shadcn/icons/*.svg`; deeper levels use `#tabmenu`

## Key References

- Vite config: `.dev/vite.config.ts`
- Design tokens: `.dev/src/media/_tokens.css`
- Version: `PKG_VERSION` in `Makefile`

---
> Source: [eamonxg/luci-theme-shadcn](https://github.com/eamonxg/luci-theme-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
