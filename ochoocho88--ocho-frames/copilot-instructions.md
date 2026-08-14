## ocho-frames

> 1. `memory.md`, starting with the CURRENT STATE block at the very top, then the top session entries (the running log: what we did, learned, decided, and what is still open).

# Workspace: ocho-frames

## Read these first, every session, before doing anything else

1. `memory.md`, starting with the CURRENT STATE block at the very top, then the top session entries (the running log: what we did, learned, decided, and what is still open).
2. The active client's `brand.md`, currently `clients/sportif/brand.md`, and `clients/sportif/voice-guidelines.md` before writing any content.
3. Any status or resume note in the active client's folder, e.g. `clients/sportif/intake/RESEARCH-RUN-STATUS.md`.

Doing this is what gives a new session continuity. Do not start work cold.

## What this workspace is

Hugo's (Ocho's) personal creative and marketing workspace. Two tracks:

1. A creative-strategy pipeline: competitor analysis, then a synthesis brief, then AI-generated production media. See `docs/pipeline-architecture.md` and `docs/marketing-fundamentals.md`.
2. Client work under `clients/`.

It is also a HyperFrames video workspace (video as code). See `README.md`.

## Active work

- Client: **Sportif** (founder Lucy Wayne). Affordable-luxury fitnesswear accessories, Australian, launching September 2026. All context lives in `clients/sportif/`. Launch products are accessories (booty bands + vegan ankle strap), not apparel.

## Two environments, one workspace (sync protocol)

Hugo works on this folder from TWO places, often alternating within the same day:

1. **Claude Code** (terminal / VS Code on his Mac): full shell, background processes survive, all local fonts, direct file deletion.
2. **Cowork** (Claude desktop app): sandboxed Linux shell, see the Cowork-specific gotchas below.

Which one am I? If shell paths look like `/sessions/<name>/mnt/hyperframes/`, this is Cowork. If they look like `/Users/hugobrizuela/...`, this is Claude Code.

**The handoff protocol (both environments, no exceptions):**

1. **Session start:** read the CURRENT STATE block, then `git log --oneline -5` to see what the other environment did last. If the working tree is dirty with changes you did not make, a session somewhere did not close out; commit or flag before working. To orient fast: `python3 scripts/memory_tools.py open --client <client>` (open loops) and `... search "<term>"` (find past context).
2. **Session end (the close-out ritual):** (a) update the CURRENT STATE block (including the handoff line); (b) add a session entry to the top of `memory.md` tagged with the environment AND with `Client:` + `Tags:` lines, e.g. `## Session 026 (2026-07-24, Claude Code): ...`; (c) record any new settled decisions in `DECISIONS.md` and any new/resolved open loops in `OPEN-QUESTIONS.md` (schemas are in those files); (d) run `python3 scripts/archive_memory.py` (no-op unless memory.md crossed 90KB) and `python3 scripts/memory_tools.py index` (regenerates `memory-index.md`); (e) run `python3 scripts/memory_tools.py check` (must print OK) and optionally `... reconcile`; (f) `git add -A && git commit`. The commit is what makes the work visible to the other side. (A pre-push hook runs `check` as a warn-only reminder; `MEMORY_ENFORCE=1` makes it block.)
3. **Session numbers are continuous across both environments.** Check the top of memory.md for the last number before starting a new entry.
4. **Do not run both environments on the same files at the same time.** If both are open, one is the builder and the other is the sounding board; the sounding board reads but does not write.
5. **Environment-specific learnings** go in "Tools and gotchas" below, labelled with which environment they apply to.

## Conventions (follow these)

- **Log every session.** At the end of any session with real work, add an entry to the TOP of `memory.md` (date, what we did, learned, decided, open). Match the existing format, and include a `Client:` line (client name, or `Ochoproductions` for workspace-wide) and a `Tags:` line. Mirror settled decisions into `DECISIONS.md` and open loops into `OPEN-QUESTIONS.md` (extractable registries, filterable by client, queried via `scripts/memory_tools.py`). The full memory system is documented in `docs/memory-system.md`.
- **Memory scales by client.** One unified chronological log (keeps cross-client learnings), but everything is tagged and filterable per client. When a second client goes active, split the CURRENT STATE block into per-client mini-blocks under a shared workspace header.
- **Voice rule: no em dashes and no en dashes** anywhere in written output or files. Use commas, periods, or parentheses instead.
- **Secrets:** API keys go in `.env` only (gitignored). Never put a real key in `.env.example`, never commit secrets.
- **Work with Hugo:** ask for project context before assuming, and flag trade-offs rather than defaulting to one approach. Background on how he works is in the `hugo-working-style` skill.
- **Generated media** goes in `clients/<client>/generated/images/` or `generated/videos/`, and every keeper's prompt is saved to the client's `image-prompts.md` (the prompt is the source of truth, binaries are gitignored). Iterate at quality low in Cowork (45s cap), render finals in Claude Code.

## Tools and gotchas

- **Research (Perplexity), Cowork version.** In Cowork, background shell processes do NOT survive across tool calls (the sandbox reaps them; each call caps at ~45s), so nohup-and-poll does not work here. Use `scripts/pplx_async.py` instead: it submits async deep-research jobs that run server-side at Perplexity, persists request ids to a registry file on disk, and polls them in later short calls. Perplexity rate-limits async submissions (HTTP 429), so stagger submits (12 to 18s) and resubmit failures. Quick synchronous queries with sonar-pro can still use `python3 scripts/perplexity_search.py "..." --model sonar-pro`. (The old nohup workflow only works on Hugo's Mac via Claude Code, a different environment.)
- **Images:** the proven engine for our warm-neutral look is **gpt-image-2** (OpenAI API key in `.env`, or run the prompts in ChatGPT). Pixa is a different engine; if you use it, flag the mismatch to Hugo. gpt-image-2 prompt format is in `docs/platform-prompt-formats.md`.
- **Network egress:** scripts that call the internet need the sandbox Domain allowlist open (Settings, Capabilities, Network Egress). A change only applies to a freshly booted sandbox, so a NEW chat is required after changing it.
- **PDF generation:** weasyprint must be pip-installed per fresh sandbox (`pip install weasyprint --break-system-packages`). System sandbox fonts are limited to Lora (serif) and Poppins (geometric sans), BUT brand font files now live in `brand/fonts/` and load by path in both environments. **Glacial Indifference (Sportif's real font, all three weights) is at `brand/fonts/glacial-indifference/`**: use it via @font-face (weasyprint) or ImageFont.truetype (Pillow) for wordmarks and overlays instead of the old Poppins stand-in. The two PDF generators still use Poppins; switch them to the real font on their next edit.
- **Showing PDFs to Hugo:** present_files does not reliably preview a PDF in chat. Render pages to PNG and show a combined montage instead.
- **File locations:** keep all deliverables in the mounted hyperframes folder (stable path). Treat the sandbox outputs dir as throwaway; it gets wiped when the sandbox reboots mid-session.
- **File deletion in the mount** requires the `allow_cowork_file_delete` tool first; plain `rm` fails with Operation not permitted.
- **Client PDF set (Sportif):** exactly two Lucy-facing PDFs are current, `Sportif-Brand-Value-Plan.pdf` (strategy) and `Sportif-Launch-Plan.pdf` (operations), regenerated via `build-brand-value-plan.py` and `build-launch-plan.py` from the `-client.md` sources. Everything else lives in `clients/sportif/_archive/`. Do not resurrect archived PDFs.
- **Two-doc drift rule:** internal source docs (e.g. `brand-value-plan.md`) drive condensed client cuts (e.g. `brand-value-plan-client.md`). Any change to an internal doc must be reflected in its client cut and the PDF re-exported. Each client cut carries a "Source of truth" header; update its synced date when you sync.

---
> Source: [OchoOcho88/ocho-frames](https://github.com/OchoOcho88/ocho-frames) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
