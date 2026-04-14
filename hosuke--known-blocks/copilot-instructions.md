## known-blocks

> An LLMBase instance that autonomously learns blockchain/DeFi knowledge.

# Known-Blocks Project Guidelines

## What is this?
An LLMBase instance that autonomously learns blockchain/DeFi knowledge.
Fork of [llmbase](https://github.com/Hosuke/llmbase) with blockchain-specific data source plugins and a learning orchestrator.

## Architecture
- Base: llmbase framework (raw/ → LLM compile → wiki/concepts/ trilingual articles)
- New plugins in tools/: ethdocs, eips, defillama, l2beat, rekt, spellbook_browse, web_research
- Orchestrator (tools/orchestrator.py): priority queue that auto-decides what to learn next
- Worker (tools/worker.py): background loop that runs orchestrator on schedule

## Data Source Plugins (tools/)
Each plugin implements `learn(batch_size, base_dir) -> list[str]` and progress tracking in `wiki/_meta/`.

| Plugin | Source | What it learns |
|--------|--------|---------------|
| ethdocs.py | ethereum.org GitHub | EVM, gas, consensus, scaling, smart contracts |
| eips.py | ethereum/EIPs GitHub | ERC-20, ERC-721, EIP-1559, etc. |
| defillama.py | DefiLlama API | Protocol metadata, TVL, chains, categories |
| l2beat.py | L2Beat API | L2 architecture, risk assessment, stages |
| rekt.py | rekt.news | Security incidents, exploit analysis |
| spellbook_browse.py | Local spellbook clone | On-chain event schemas, data models |
| web_research.py | DuckDuckGo search | Gap-filling for missing concepts |
| orchestrator.py | All of the above | Priority-based auto-learning |

## Orchestrator Priority Queue
- P0: Foundation concepts (blockchain, EVM, gas, DeFi, AMM...)
- P1: Broken wiki-links
- P2: Trending protocols (DefiLlama top-N)
- P3: Structured traversal (EIPs, ethereum.org docs)
- P4: Deepen stub articles
- P5: Curiosity browsing (random spellbook/rekt/l2beat)

## Key Config (config.yaml)
- `worker.learn_source: orchestrator` — uses the orchestrator for auto-learning
- `spellbook.path` — local path to Dune spellbook clone
- `defillama.min_tvl` / `defillama.top_n` — protocol filtering
- `orchestrator.curiosity_ratio` — probability of random exploration

## Code Patterns
- LLM calls through tools/llm.py:chat() — never call OpenAI directly
- All wiki-links use [[target]] syntax; resolved via tools/resolve.py
- Article slugs are kebab-case; domain-agnostic compile pipeline
- Reference plugins in tools/refs/ provide get_source_url() for citations
- Alias resolution via tools/resolve.py — always use resolve_link() for wiki-link targets
- Taxonomy is LLM-generated (not hardcoded) — tools/taxonomy.py
- Never expose specific LLM provider names in public code or commits

## Auto-Fix Pipeline (tools/lint.py:auto_fix)
1. clean_garbage() — remove template stubs
2. fix metadata — LLM generates missing summary/tags
3. fix_broken_links() — alias-aware, only stubs for truly missing concepts
4. merge_duplicates() — LLM confirms, content 叠加进化
5. fix_uncategorized() — regenerate taxonomy

## Key Files
- tools/resolve.py — alias system (central to all link resolution)
- tools/xici.py — guided introduction generation
- tools/taxonomy.py — emergent LLM-generated categories
- wiki/_meta/ — aliases.json, taxonomy.json, health.json, backlinks.json

## Commit Process (MANDATORY)
Before EVERY git commit, you MUST:
1. Run `cd frontend && npx tsc --noEmit` — TypeScript check
2. Run `python -c "from tools.lint import lint; print('OK')"` — Python import check
3. Run Codex review on staged changes and WAIT for the result:
   ```
   codex exec --sandbox read-only -C . \
     --output-last-message /tmp/codex-review-result.txt \
     "Review the staged git diff for bugs, security, edge cases. file:line format. Say LGTM if clean."
   ```
4. Read the Codex review output. If there are HIGH issues, fix them BEFORE committing.
5. Only then run `git commit`

Do NOT skip Codex review. Do NOT commit while Codex is still running.

## CI Process
- Python import check: `python -c "from tools.orchestrator import learn; print('OK')"`
- CLI check: `python -c "from tools.cli import main; print('OK')"`
- TypeScript check: `cd frontend && npx tsc --noEmit`
- Python lint check: `python -c "from tools.lint import lint; print('OK')"`
- Lint check: `python llmbase.py lint check`
- Build: `cd frontend && npx vite build`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hosuke)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Hosuke)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
