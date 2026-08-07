## yan-atelier-site

> Project-level instructions Claude reads automatically before acting.

# YÀN Atelier · Claude Operating Notes

Project-level instructions Claude reads automatically before acting.

## Session continuity protocol (READ FIRST)

This project's memory is shared via directory junctions across 4 CWDs: `D:\YAN_Atelier 品牌`, `D:\YAN_Atelier_Site`, `D:\wechat-claude`, `D:\YAN_AutoEdit`. All four resolve to the same `MEMORY.md` + `project_session_state_*` files. Anything you write to memory in one CWD is visible from the others.

**On session start** (first user message in this project):
1. The auto-memory MEMORY.md is already in your context. Scan for the LATEST `project_session_state_YYYY-MM-DD*` entry — that file is the canonical resume anchor.
2. If user's first message is `继续 YÀN` / `继续上次` / `接 X` / `恢复` / `restart` / `where were we`: read that latest session_state file in full BEFORE replying. Summarize in ≤3 lines: (a) last commit + state, (b) what was in progress, (c) any pending user actions / blockers. Then ask "继续 X 还是新任务?"
3. If user's first message is a fresh task: do NOT volunteer resume context — just work. Only surface a 1-line `(resuming from: <state-file-name>)` hint if the new task touches the same lane as the last session_state.

**Mid-session snapshot triggers** — write/update today's `project_session_state_YYYY-MM-DD.md` memory file when ANY of these fire:
- User says 收工 / 明天继续 / 我先去X / 拜 / bye / 睡了 / 先停一下
- You're about to ship a load-bearing change (commit + push) involving 3+ files OR Hero/H1/pricing/section-structure
- A multi-agent review (CEO/Investor/persona panel) just completed
- User explicitly says 存一下 / 记一下 / save state
- Heap risk: this session has run > ~6 hours of dialog (Claude Code Bun heap caps at ~1.4GB → OOM. See [[user-yan-atelier]] crash 2026-06-23)

**Snapshot format** — body structure: 3 short paragraphs in this order:
1. **Last commit** (SHA + 1-line) + **live URL state** (deployed yes/no)
2. **What was just done** this session (3-6 bullets, factual)
3. **Pending** (user-action items + Claude-action items, separated)

Title slug: `project_session_state_YYYY-MM-DD[_eod|_late|_early].md`. Always add a `MEMORY.md` index line. If today's state file already exists, EDIT it (don't create _v2 / _2 / etc — overwrite the body).

**Stale state supersession** — when a new session_state supersedes an older one (e.g., morning anchor → EOD anchor), add `**SUPERSEDED BY** [[new-slug]]` at the top of the old file's body. Don't delete.

**Daily restart hygiene** — Claude Code's Bun runtime OOMs around 1.4GB heap. Long sessions accumulate transcript in RAM. Tell the user once per restart cycle: "建议每天滚一次 claude.exe; 重开用 `claude --continue --dangerously-skip-permissions` 接上一段对话(用户标准开机口令,见 [[user-daily-boot-command]]); 长上下文用 [[project-session-state-YYYY-MM-DD]] 恢复战略上下文."

## Live URLs

- **Production (LIVE)**: `https://yan-atelier-site.pages.dev/`
  - This is the URL to give the user for any "test the site" / "mobile preview" / "send to customer" request.
  - Mobile + desktop both responsive. Works on any network (no LAN required).
  - Latest commit auto-deploys via Cloudflare Pages ~30-60s after `git push origin main`.
- **Custom domain (NOT yet wired)**: `https://yan-atelier.com/` — DNS not resolving. Do NOT give this URL to user; it returns nothing.

## Mobile testing

When user asks "给我手机测试链接" or "mobile link" or similar:
- **DO**: reply with `https://yan-atelier-site.pages.dev/`
- **DO NOT**: start a local dev server (no dev server needed; site is single-file static HTML).
- **DO NOT**: invent LAN IP addresses like `http://192.168.x.x:3000` — these almost never work and confuse the user.

## Deploy flow

This IS a git repo (despite environment occasionally reporting otherwise). Repo on GitHub → Cloudflare Pages auto-builds on push to `main`.

To ship a change:
1. Edit files in `D:\YAN_Atelier_Site\`
2. `git add <file>` (be specific — avoid `git add -A`; there are untracked WeChat-export images that should NOT be committed)
3. `git commit -m "..."` (include `Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>`)
4. `git push origin main`
5. Wait 30-60s, Cloudflare deploys
6. Hard-refresh on `https://yan-atelier-site.pages.dev/` to verify (browser cache can mask new deploys)

## Operating rules (load-bearing — see BRAND.md for full)

- **R1**: Section titles name the JOB, never enumerate contents/inventory/taxonomy
- **R2 (ZONE)**: Voice by zone — Cold (Home above-fold) strict commerce, Warm (Home below-fold) atelier-register, Deep (/craft, /about, /consult) atelier-encouraged
- **R3**: PDP trust info inline near CTA, never as top promo bar

Read `BRAND.md` for full rule details + worked examples + banned phrasings before any voice/title/PDP changes.

## Substance protection (do NOT touch)

- plique-à-jour technique name + Renaissance lineage (in Plique section)
- Hu Yan name + Founder status (biographical references in About / Maker)
- Yunnan workshop / 滇韵珐琅手工坊 / Yunnan Studio
- 800°C high-fire fact
- Craft page Schema.org Article markup + 10 academic citations
- Hero copy "Wear it. People will ask where you found it. / 戴出去, 别人会问你哪儿淘的。"
- CN view full readability (cascade flips must preserve CN view)

## What NOT to do

- Do not start local dev servers for "mobile testing" — use prod URL
- Do not invent LAN URLs
- Do not push to remote without explicit user confirmation when working via WeChat bridge (user can't quickly intervene from mobile)
- Do not commit WeChat export files (`images/_cgi-bin_*`, `images/00 (*).png`, `images/output*.png`, Chinese-filename reference images, `video/`, `docs/`)
- Do not give user the custom domain `yan-atelier.com` — not live

## WeChat bridge context

If this Claude session is reached via WeChat ClawBot (user typing from mobile):
- Be more conservative with destructive ops (no git push without "yes do it")
- Show diffs / changes summary BEFORE applying
- Replies should be tight (mobile screen reads shorter)
- Long-running agents are fine (user reads at next WeChat check)
- Never start local servers (user can't see localhost from phone unless same WiFi + firewall open + correct port)

## Image-gen prompt research (evidence-led scene library)

**Report**: `D:/YAN_AutoEdit/docs/jewelry_ecom_scene_research_2026-06-21.md`

Contains real-world scene templates harvested from indie fine jewelry listings on Etsy / Amazon Handmade / Shopify (SBB / WWAKE / Alighieri / Pippa) / 小红书 / Anna Hu, with subject-preservation language patterns specifically tested on Nano Banana Pro + gpt-image-2 /edits. If user (especially via WeChat) asks "找一个 X 风格 prompt" / "查那个调研报告" / "给我场景模板" — read this file first, don't reinvent.

Gap-analyzed against `D:/YAN_AutoEdit/config/gen_presets.yaml` (18 existing presets). Recommended scene additions are at end of report.

**Subject preservation language patterns** (the user's #1 pain point — they want 1:1 identity preservation when changing scenes): documented in Part D of the report. Use these patterns in `modifier` when writing curl calls, not free-form "preserve identity" mumbo.

## Image-gen flow (AutoEdit v2 · platform-led · 2026-06-22)

> **PROTOCOL · READ FIRST when user asks to generate / 出图 / 出套图 / sends product photo + edit request**: `D:/YAN_AutoEdit/config/prompts/image_gen_intent.md` — 9-section 5-step gate (intent → assets → mid-clarify → summary → execute). Refuses to POST `/api/generate/image` until step-4 summary is confirmed by user (`1`=ship, `2`=edit, `3`=drop). Handles 7 modes (single / 风格转移 / 场景植入 / 配件合成 / B1 库匹配 / B2 新设计 / 套图战略). The reference data below (curl format / keyword map / SKU table / wearing rules) is for use AFTER the protocol gate confirms. **Direct curl without running the 5-step gate is a hard-law violation** (§ 0.1) — every gen call costs ¥0.x.

**Endpoint**: `POST http://127.0.0.1:8765/api/generate/image` (multipart form)

**Flow**: user sends a request via WeChat → Claude parses platform / image_type / SKU → POSTs → waits 60-120s → response includes `public_url` (auto-CDN-deployed) → Claude sends URL text to user → WeChat renders preview.

**Required form fields** (all optional individually but valid combinations apply):
| Field | Values | Notes |
|---|---|---|
| `platform` | `xiaohongshu` / `tiktok` / `facebook` / `instagram` | sets aspect: xhs 3:4, tt 9:16, fb 1:1, ig 4:5 |
| `image_type` | `商业图` / `产品图` / `场景图` / `品宣图` / `佩戴特写` | maps to preset via combinations matrix |
| `sku_id` | `0001`-`0012` (from SKU library) | OR upload `sku_ref` file |
| `sku_ref` | file upload | use when user supplies a custom SKU photo not in library |
| `style_ref` | file upload (optional) | inspiration image — model sees this for composition/mood, NOT for product |
| `remix` | `lock` / `vary` (default `lock`) | `vary` = model face/identity remixed each gen (anti-likeness-infringement) |
| `modifier` | free text | appended to prompt; use for one-off tweaks ("偏暖色 / 不要文字") |
| `auto_publish` | `true` (default) / `false` | `true` returns `public_url` from CDN; keep ON for WeChat flow |

**SKU library** (auto-fills photo + standard description):
- `0001` 雏菊·胸针 · `0002` 蝴蝶·胸针 · `0003` 银杏叶·胸针 · `0004` 银杏·发饰
- `0005` Iris·镯·紫 · `0006` Iris·镯·Azure · `0007` Iris·镯·Dusk
- `0008` 莲鱼·吊坠 · `0009` 云纹·镯(福) · `0010` 蝴蝶·项链 · `0011` 云纹·镯(素) · `0012` 青莲·编绳
- Full library: `GET http://127.0.0.1:8765/api/skus/` returns photo URLs + standard descriptions

**Sample WeChat call** (user: "用蝴蝶项链做小红书佩戴特写, 模特换脸"):
```bash
curl --noproxy '*' -X POST http://127.0.0.1:8765/api/generate/image \
  -F platform=xiaohongshu \
  -F image_type=佩戴特写 \
  -F sku_id=0010 \
  -F remix=vary \
  -F auto_publish=true \
  --max-time 200
# returns: {assets:[{path, url, public_url}], elapsed_s, cdn_warnings}
# public_url is the Cloudflare Pages URL — send THAT to WeChat
```

**Parsing user intent** from WeChat text:
- Platform keywords: 小红书/xhs → `xiaohongshu`; TikTok/TT/抖音/竖版9:16 → `tiktok`; Facebook/FB → `facebook`; Instagram/IG/INS → `instagram`
- Type keywords: 主图/广告/首页 → `商业图`; 白底/catalog/搜索图 → `产品图`; 场景/氛围 → `场景图`; 品宣/editorial → `品宣图`; 佩戴/上身/on-model → `佩戴特写`
- SKU disambiguation (default-ambiguous-motif rules, unchanged):
  - 「蝴蝶」 → **0010 Butterfly Necklace** (0002 brooch is 已售/sold)
  - 「云纹」/「云」 → **0011 Cloud Bangle · Plain** (default plain; 0009 is 福 variant)
  - 「鸢尾」/Iris → **0005** (others 0006/0007 are color variants)
  - 「银杏」 → **0003 Ginkgo Leaf** (0004 is hairpin pair)
- 模特换脸/防侵权/换脸 → `remix=vary`. **Default to `vary` if user didn't specify** (safer for commercial use).

**Reply template to WeChat**:
```
匹配到: <SKU> · <平台> · <图类型> · <remix mode>
生成耗时: <elapsed_s>s

<public_url>

(Cloudflare CDN 任意网络可开. ~30s 后未渲染就刷新)
```

**Hard rules**:
- NEVER pass user's WeChat ref image as `sku_ref` — use `style_ref` instead (it'll inform mood/composition, not product shape).
- NEVER send the local `path` (D:\...) to WeChat — send only `public_url`.
- If `cdn_warnings` is non-null in response, surface it to user (CDN failed, only local file available — won't render in WeChat).
- If 502 from API: check `D:\YAN_AutoEdit` logs — usually aidraw365 channel flake. Retry is automatic (3 attempts, 5s/15s backoff) but a 3rd failure means upstream is down. Wait 2 min and retry, OR tell user to try again later.
- If `gpt-image-2` channel down explicitly (503 "no available channel"): config switches to `aidraw365_nanobanana_pro` will work for SKU-only (no style_ref) — but tell user "降级到 nano-banana, 不能用 style_ref" if you do that swap.

## Jewelry wearing position rules (CRITICAL for image gen)

**手链/手镯 (Bracelet/Bangle)** — MUST encircle the wrist:
- ✅ Correct: bracelet worn AROUND the wrist, forming a closed loop encircling the wrist bone
- ❌ Wrong: bracelet laid flat on top of hand, displayed as a flat chain (NOT worn)
- Modifier to add: `bracelet worn around the wrist in closed circular position, encircling wrist bone, NOT laid flat on hand`
- 中文 modifier: `手链环绕手腕佩戴，围绕手腕骨形成闭环，不是平铺展示`

**项链 (Necklace)** — MUST hang around the neck:
- ✅ Correct: necklace draped around neck, pendant resting on chest/collarbone area
- ❌ Wrong: necklace laid flat on surface or held in hand
- Modifier to add: `necklace worn around neck, pendant hanging naturally on chest`

**胸针 (Brooch)** — MUST be pinned to clothing:
- ✅ Correct: brooch pinned to lapel, collar, scarf, or clothing fabric
- ❌ Wrong: brooch held in hand or placed on flat surface
- Modifier to add: `brooch pinned to clothing fabric, worn on lapel or collar`

**发饰/发簪 (Hair accessory/Hair pin)** — MUST be inserted in hair:
- ✅ Correct: hair pin inserted through hair bun or styled hair
- ❌ Wrong: hair pin held in hand or placed on surface
- Modifier to add: `hair pin inserted in styled hair or bun`

**吊坠 (Pendant)** — same as necklace, must hang on chain around neck.

**戒指 (Ring)** — MUST encircle finger:
- ✅ Correct: ring worn on finger, encircling the finger
- ❌ Wrong: ring held or placed on palm
- Modifier to add: `ring worn on finger, encircling the finger`

**ALWAYS add the appropriate wearing position modifier when generating 佩戴特写 (worn close-up) images.**

## Legacy: vision-analyze for "make similar to this ref" (when no SKU library match)

If user sends a ref image but no SKU/platform/type:
1. **Vision-analyze the ref image** — extract style WORDS: lighting / background / composition / palette / mood. The ref image's TEXT description goes into `modifier`. NEVER pass user's ref image as `sku_ref` — that uses it as the shape to preserve, which gives the wrong output (a "product image" of the user's ref instead of a YÀN piece styled like the ref).
2. **Pick YÀN SKU** — two paths:
   - **(A) Auto-match**: read `PRODUCTS` array (line ~7159 in `index.html`). Match by form (brooch / bracelet / necklace / pendant / hair pin) + dominant motif if user's ref hints at one (e.g., shows a delicate bracelet → match Iris Bracelet).
   - **(B) User-specified**: if user said "用蝴蝶胸针" / "做个鸢尾镯款" / mentions a SKU number → use that. User-specified always overrides auto-match.
   - **Default-ambiguous-motif rules** (when user names a motif but not the form):
     - 「蝴蝶」 → **0010 Butterfly Necklace** (0002 brooch is 已售/sold, prefer in-stock 0010)
     - 「云纹」/「云」 → **0011 Cloud Bangle · Plain** (0009 Cloud Bangle · 福 is the variant with character, prefer the plain one as default)
     - 「鸢尾」 → **0005 Iris Bracelet** (original, in-stock; 0006 0007 are color variants)
     - 「银杏」 → **0003 Ginkgo Leaf Brooch** (0004 is hairpin pair — more specific)
3. **Pick preset** — `white_bg_main` for Amazon/Temu/Etsy clinical main image; `macro_detail` for close-up texture/material; `flat_lay_collection` for top-down editorial. If user's ref is lifestyle / model-worn / mirror selfie → still use `white_bg_main` for clean catalog output OR pass strong modifier to override. (Default to `white_bg_main` unless user specifies.)
4. **Build curl call** — `sku_ref` is the YÀN catalog file, `modifier` carries the ref-image style words:
   ```bash
   curl --noproxy '*' -X POST http://127.0.0.1:8765/api/generate/image \
     -F preset_id=white_bg_main \
     -F modifier="soft natural daylight, warm cream linen surface, top-down composition, editorial lookbook feel" \
     -F sku_ref=@D:/YAN_Atelier_Site/images/p05-bracelet-purple.jpg
   # DO NOT pass -F style_ref=@... — that field is currently a noop in AutoEdit.
   # DO NOT pass user's WeChat ref image as sku_ref — wrong direction.
   ```
4. **Parse response** for `assets[0].path` (local file path on PC).
5. **Push to public CDN** so user can open from any network. yan-gen-cdn project is **NOT** connected to GitHub — git push does NOT auto-deploy. **Use wrangler:**
   ```bash
   # Copy generated image into yan-gen-cdn (use distinctive filename — never overwrite)
   cp "<assets[0].path>" "D:/yan-gen-cdn/images/<timestamp>-<sku-id>.png"
   # Deploy via wrangler (CLOUDFLARE_API_TOKEN already in env; uses --commit-dirty=true since no git tracking required)
   cd /d D:/yan-gen-cdn
   CLOUDFLARE_API_TOKEN="$CLOUDFLARE_API_TOKEN" npx --yes wrangler@latest pages deploy . --project-name=yan-gen-cdn --branch=main --commit-dirty=true
   # Wrangler outputs the preview URL + main URL takes ~30s extra to propagate.
   # Public URL pattern:
   #   https://yan-gen-cdn.pages.dev/images/<filename>
   ```
   **DO NOT** use `git add + git commit + git push` for yan-gen-cdn — that pushes to GitHub but Cloudflare doesn't watch GitHub for this project. The actual deploy mechanism is `wrangler pages deploy`.
6. **VERIFY** before sending URL to user: `curl --noproxy '*' -o /dev/null -w "%{http_code} %{size_download}" <full-url>` should return 200 AND size matching the generated file (typically > 200kb). Wait 30s after deploy + retry if first attempt 404s OR returns ~574b (that's the index.html placeholder = file not propagated yet).
7. **Always append `?v=<timestamp>` cache-bust to URL sent to user** (because WeChat / mobile browser may have cached a previous 404 → user sees placeholder index.html instead of new image). Example: `https://yan-gen-cdn.pages.dev/images/X.png?v=1781949000`
8. **Then send the public URL via WeChat**. Don't send the URL before step 6 verify passes — user gets index.html / 404 page when file isn't live yet, and screenshots it as "still broken".

**Presets to pick from** (depends on use case):
- `white_bg_main` — Amazon-grade pure white bg main image
- `macro_detail` — close-up texture/material shot
- `flat_lay_collection` — top-down flat-lay with 3-5 similar pieces
- (More via `curl http://127.0.0.1:8765/api/generate/presets`)

**SKU → image path map** (current local images for sku_ref):

⚠️ **Photo reality vs SKU name mismatch** — some SKUs are named as brooch/etc but current photo shows a different form (necklace). For image gen purposes ALWAYS go by the **PHOTO ACTUAL** column, not the SKU name. New form-specific photos are pending user shoots.

| SKU | SKU name (catalog) | Image path | **PHOTO ACTUAL** |
|---|---|---|---|
| YA-2026-0001 | Daisy Brooch | `D:/YAN_Atelier_Site/images/p01-daisy-green.jpg` | **necklace form** (brooch photo pending) |
| YA-2026-0002 | Butterfly Brooch | `D:/YAN_Atelier_Site/images/p02-butterfly-blue.jpg` | **necklace form** (brooch photo pending) |
| YA-2026-0003 | Ginkgo Leaf Brooch | `D:/YAN_Atelier_Site/images/p03-ginkgo-ring.jpg` | brooch (verify before gen) |
| YA-2026-0004 | Ginkgo Hair Pin · Pair | `D:/YAN_Atelier_Site/images/p04-ginkgo-hairpin.jpg` | hairpin |
| YA-2026-0005 | Iris Bracelet | `D:/YAN_Atelier_Site/images/p05-bracelet-purple.jpg` | bracelet ✓ |
| YA-2026-0006 | Iris Bracelet · Azure | `D:/YAN_Atelier_Site/images/p06-bracelet-blue.jpg` | bracelet ✓ |
| YA-2026-0007 | Iris Bracelet · Dusk | `D:/YAN_Atelier_Site/images/p07-bracelet-purple2.jpg` | bracelet ✓ |
| YA-2026-0008 | Fish & Lotus Pendant | `D:/YAN_Atelier_Site/images/p08-lock.jpg` | pendant ✓ |
| YA-2026-0009 | Cloud Bangle · 福 | `D:/YAN_Atelier_Site/images/p09-bangle-fu.jpg` | bangle (verify · suspected 0009↔0011 secondary swap pending 2026-06-23) |
| YA-2026-0010 | Butterfly Necklace | `D:/YAN_Atelier_Site/images/p10-butterfly-tassel.jpg` | butterfly necklace ✓ (corrected 2026-06-23: file was content-swapped with p09; restored to match name) |
| YA-2026-0011 | Cloud Bangle · Plain | `D:/YAN_Atelier_Site/images/p11-bangle-cloud.jpg` | bangle (verify · suspected 0009↔0011 secondary swap pending 2026-06-23) |
| YA-2026-0012 | Lotus Cord Bracelet | `D:/YAN_Atelier_Site/images/p12-bracelet-cord.jpg` | cord bracelet ✓ |

**Implication for generation**: when user says 「蝴蝶」 or 「雏菊」, the only available photos show **necklace forms**. Don't promise a "brooch" generation — output will look like necklace. If user explicitly wants brooch generation for these motifs, tell them: "需先有胸针款实物照才能保形生成 — 现在只有项链款照片" (real brooch photo needed first — only necklace photos exist currently).

**Reply template to WeChat** (Markdown OK):
```
匹配到: <SKU name>
风格分析: <brief 1-2 lines>
预设: <preset_id>

生成图: https://yan-gen-cdn.pages.dev/images/<filename>
(任何网络都能开 · 推 Cloudflare Pages 约 30-60s 后生效)
```

**Notes / caveats** (updated 2026-06-22):
- API server is on `127.0.0.1:8765` (Vite dev :5173 proxies /api to it). If down, kill any orphan + relaunch via `start_api.bat` (env keys `YAN_GEN_API_KEY` + `SILICONFLOW_API_KEY` come from User-scope env vars; bridge bash must source them).
- **Active image provider** (config/gen.yaml): `apinebula_gpt_image_2` — same `gpt-image-2` model, swapped from aidraw365 on 2026-06-28 after upstream 502s on aidraw365's auto-channel. Each call ~$0.04. Fallbacks preserved in gen.yaml: `aidraw365_gpt_image_2` / `_pro` / `_nanobanana_pro` (last one is single-image only — no style_ref) plus `siliconflow_qwen_edit` / `siliconflow_kolors`. To fall back: edit `image.active` in `D:/YAN_AutoEdit/config/gen.yaml` AND verify `YAN_GEN_API_KEY` (aidraw365) or `SILICONFLOW_API_KEY` (SF) matches the provider's key.
- **Generation takes 60-120s** for gpt-image-2 (longer than Kolors). +15-30s for CDN deploy when `auto_publish=true`.
- Output saved to `D:\YAN_AutoEdit\assets\generated\<date>\<job-id>_0.png` (LEGACY: `D:\YAN_AutoEdit\out\<date>\` is the OLD path, no longer used).
- CDN copy lands in `D:\yan-gen-cdn\images\<job-id>_0.png` then wrangler-deploys.
- When user sends a ref image to WeChat, the bridge saves it to `~/.wechat-claude-code/inbox-images/<ts>.jpeg` (path prepended to prompt).
- Photo reality: 12 SKU images in `D:/YAN_Atelier_Site/images/p*.jpg` are currently real but some show different forms than the SKU name (e.g., 0001/0002 named "brooch" but photo is necklace). Workflow: pass via `sku_id` and trust the photo; do not promise forms not shown.

---
> Source: [jianyouren/yan-atelier-site](https://github.com/jianyouren/yan-atelier-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
