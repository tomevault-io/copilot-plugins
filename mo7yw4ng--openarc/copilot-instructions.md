## openarc

> For "run openarc patrol", "process the queue", or a scheduled OpenArc cycle, read

# OpenArc patrol (for Codex)

For "run openarc patrol", "process the queue", or a scheduled OpenArc cycle, read
`skills/openarc-patrol/SKILL.md` completely and follow it. That file is the canonical
workflow shared with Claude; do not maintain a second copy here.

---

# Building the source adapter (Codex twin of the `setup-source` skill)

If `openarc patrol` says no source adapter is configured, or the user asks to set up /
connect / rebuild the source, you build `openarc_adapter.py`. OpenArc ships no code
for talking to any source; it reaches its source through one seam
you implement locally, for the source the user is authorized to use, on the user's own
account/session. The user is responsible for complying with that source's terms — surface
that, don't work around it.

**Contract.** `openarc_adapter.py` exposes `create_source(config)` returning an object
with `search(keyword, since) -> list[Post]` and `publish_reply(post_id, text) -> None`
(the `PostSource` seam in `src/openarc/source.py`). `Post` is `openarc.source.Post`
(`id`, `author_id`, `author_username`, `text`, `permalink`, `posted_at` ISO string); `id`
and `author_id` must be stable. Read any source secrets from the environment inside the
adapter — OpenArc's config knows nothing about them. Copy `openarc_adapter.example.py`
to start.

**How.** The user supplies what only they have — which source, and how their own client
reaches it. Ask them to perform the action once in their own logged-in session (a search;
a reply) and give you the resulting request(s); reproduce those with `httpx` and map the
response into `Post`s. In DevTools, `SearchResultsQuery` and
`configure_text_only_post` are useful search for this Threads flow;
they are not stable contracts. Save captures as UTF-8 files, not PowerShell command-line
strings. Don't invent endpoints or guess a private interface. Verify with one real
`openarc patrol`, then let the user review in `openarc queue`.

When submitting non-ASCII draft text, write a UTF-8 file and use
`openarc draft <id> --text-file <path>`; do not inline Chinese in a PowerShell command.

**Keep it narrow.** This is the user's personal automation over their own access — search
and reply, at a human pace. Don't add bulk-scraping, multi-account, or evasion behavior;
if asked, decline and explain the account/legal risk.

---
> Source: [MO7YW4NG/OpenArc](https://github.com/MO7YW4NG/OpenArc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
