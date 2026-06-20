## llm-wiki-skill

> |


# /wiki

LLM Wiki maintainer for `~/vaults/`. Implements the Karpathy LLM Wiki pattern:

- **raw/** — immutable source materials
- **wiki/** — LLM-owned curated knowledge (concepts, entities, source summaries, comparisons)
- **CLAUDE.md** — schema and workflow rules

The vault is **consumer-only**. `graphify` is never run inside the vault.
Run `graphify` in each project; this skill brings the output into `raw/repos/` and synthesizes a wiki summary.

Reference: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

## Usage

```
/wiki                                       # show this help
/wiki init [path]                           # create vault structure (default ~/vaults)
/wiki ingest-project <project-path> [name]  # graphify project → raw/repos → wiki/sources
/wiki ingest-url <url>                      # fetch URL into raw/articles, then summarize
/wiki ingest-file <path>                    # copy local file into raw/, then summarize
/wiki query "<question>"                    # answer using wiki, file synthesis back
/wiki lint                                  # check orphans, frontmatter, sources, staleness
/wiki overview                              # rewrite wiki/overview.md from current pages
```

## Vault resolution

- Default root: `~/vaults`. Override with `WIKI_VAULT` env var or `init [path]`.
- For every non-`init` command: verify `$VAULT/CLAUDE.md` exists. If not, tell the user to run `/wiki init` first and stop.
- Conventions live in `$VAULT/CLAUDE.md`. Read it before any non-trivial operation — it is the source of truth for page types, frontmatter, and absolute rules.

## What you must do when invoked

If the user invoked `/wiki` or `/wiki -h` with no subcommand, print the Usage block verbatim and stop.

Otherwise, bind:

```bash
VAULT="${WIKI_VAULT:-$HOME/vaults}"
TODAY=$(date +%Y-%m-%d)
```

Then dispatch on the subcommand below.

## Common helpers

**Append a log entry** (always at the end of the operation, never mid-flight):

```bash
printf '\n## [%s] %s | %s\n' "$TODAY" "$OP" "$MSG" >> "$VAULT/wiki/log.md"
```

`$OP` is one of `init | sync | ingest | query | lint | synthesis`.
**Never edit existing log lines. Append only.**

**Read vault schema before non-init operations:** read `$VAULT/CLAUDE.md`. Defer to its rules over anything embedded in this skill.

---

### /wiki init [path]

Bootstrap a fresh LLM Wiki vault. Default path is `~/vaults`; if `[path]` is given, use it.

1. Verify state:
   ```bash
   TARGET="${1:-$HOME/vaults}"
   if [ -f "$TARGET/CLAUDE.md" ]; then
       echo "$TARGET/CLAUDE.md already exists. Refusing to overwrite."
       echo "Remove it first or pick another path."
       exit 1
   fi
   mkdir -p "$TARGET"
   ```

2. Create directory structure:
   ```bash
   mkdir -p "$TARGET"/raw/{articles,papers,repos,data,images,assets}
   mkdir -p "$TARGET"/wiki/{concepts,entities,sources,comparisons}
   ```

3. Write `$TARGET/CLAUDE.md` with the schema below (use the Write tool, do NOT cat-heredoc — the content has many backticks):

   ```markdown
   # LLM Wiki Schema (Karpathy Pattern)

   이 vault는 Andrej Karpathy의 LLM Wiki 패턴을 따른다.
   참조: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

   ## 3-Layer 아키텍처

   1. **Raw Sources (`raw/`)** — immutable. 외부에서 가져온 원본 자료. 절대 수정하지 말 것
   2. **Wiki (`wiki/`)** — LLM이 소유. raw/를 컴파일한 결과물
   3. **Schema (`CLAUDE.md`)** — 이 파일. 에이전트를 wiki maintainer로 변환하는 규칙

   ## 디렉토리 레이아웃

   - `raw/articles/` — 블로그·뉴스·클리핑 (`YYYY-MM-DD-slug.md`)
   - `raw/papers/` — 논문 PDF
   - `raw/repos/` — 외부 레포 README, graphify export 산출물
   - `raw/data/` — 벤치마크, CSV, JSON
   - `raw/images/` — 다이어그램, 스크린샷
   - `raw/assets/` — Obsidian 첨부파일 기본 경로
   - `wiki/index.md` — 전체 카탈로그 (카테고리별)
   - `wiki/log.md` — append-only 활동 로그
   - `wiki/overview.md` — 지식 베이스 전체 합성 요약
   - `wiki/concepts/` — 개념·이론·패턴
   - `wiki/entities/` — 인물·조직·제품
   - `wiki/sources/` — raw/ 항목별 요약 (1:1 매핑)
   - `wiki/comparisons/` — 비교·대조 페이지

   ## 페이지 프론트매터

   모든 wiki 페이지는 YAML frontmatter 필수:

   - `title` — Human-readable 제목
   - `type` — `concept | entity | source-summary | comparison`
   - `sources` — 참조한 raw/ 파일 경로 배열
   - `related` — 링크된 다른 wiki 페이지
   - `created` — `YYYY-MM-DD`
   - `updated` — `YYYY-MM-DD`
   - `confidence` — `high | medium | low`

   ## 페이지 유형별 규칙

   - **concept (`wiki/concepts/`)** — 개념·알고리즘·패턴. "무엇이고 왜 중요한가" 중심
   - **entity (`wiki/entities/`)** — 인물·조직·제품·모델. 핵심 사실 + 관련 작업 + 관계
   - **source-summary (`wiki/sources/`)** — raw/ 단일 항목의 압축 요약 (1:1 매핑)
   - **comparison (`wiki/comparisons/`)** — N개 항목을 같은 축에서 비교. 표 권장

   ## 특수 파일

   - `wiki/index.md` — 카테고리별 카탈로그. 신규·삭제 시 반드시 갱신
   - `wiki/log.md` — append-only. 절대 과거 항목 수정 금지. 포맷: `## [YYYY-MM-DD] <operation> | <description>` (operation: `init | sync | ingest | query | lint | synthesis`)
   - `wiki/overview.md` — 전체 high-level 합성. 주기적으로 다시 쓰기

   ## 워크플로

   주요 워크플로는 `/wiki` skill을 통해 실행:

   - `/wiki ingest-project <path>` — graphify 산출물을 raw/repos/ 로 가져와 wiki/sources/ 로 요약
   - `/wiki ingest-url <url>` — URL을 raw/articles/ 로 저장하고 요약
   - `/wiki ingest-file <path>` — 로컬 파일을 raw/ 적절한 위치에 저장하고 요약
   - `/wiki query "<question>"` — wiki를 통해 답변. 새 합성은 페이지로 저장
   - `/wiki lint` — 고아·frontmatter·sources 경로·stale 점검
   - `/wiki overview` — overview.md 재합성

   ## graphify 연동

   이 vault는 **consumer-only**다. graphify는 vault 안에서 절대 실행하지 않는다.

   - graphify는 각 프로젝트에서 실행 (`/graphify .`)
   - `/wiki ingest-project <프로젝트경로>` 가:
     1. 프로젝트에서 `graphify . --update --wiki --no-viz` 실행
     2. `graphify-out/wiki/` → `raw/repos/<name>/` 로 rsync (snapshot)
     3. Claude가 raw/repos/<name>/ 을 읽어 `wiki/sources/summary-<name>.md` 작성
     4. 새 개념·인물은 wiki/concepts/, wiki/entities/ 에 페이지 생성
     5. wiki/index.md, wiki/log.md 갱신

   ## Obsidian 설정

   - Attachment folder path → `raw/assets/`
   - Default view mode: preview
   - `alwaysUpdateLinks: true`

   ## 절대 규칙

   - `raw/` 파일은 절대 수정하지 말 것
   - `wiki/log.md` 는 append-only. 과거 항목 절대 편집 금지
   - frontmatter 없는 wiki 페이지 생성 금지
   - `wiki/index.md` 를 갱신하지 않고 새 wiki 페이지 생성 금지
   - **vault 안에서 graphify 직접 실행 금지** (consumer-only)
   ```

4. Write the three special files (use the Write tool):

   `$TARGET/wiki/index.md`:
   ```markdown
   ---
   title: Index
   type: index
   created: <TODAY>
   updated: <TODAY>
   ---

   # Wiki Index

   전체 wiki 페이지 카탈로그. 새 페이지를 만들 때 반드시 이 파일에 등록한다.

   ## Special

   - [overview](overview.md) — 지식 베이스 전체 high-level 합성
   - [log](log.md) — append-only 활동 로그

   ## Concepts

   _아직 항목 없음._

   ## Entities

   _아직 항목 없음._

   ## Sources

   _아직 항목 없음._

   ## Comparisons

   _아직 항목 없음._
   ```

   `$TARGET/wiki/log.md`:
   ```markdown
   ---
   title: Activity Log
   type: log
   created: <TODAY>
   updated: <TODAY>
   ---

   # Log

   Append-only. 과거 항목 절대 수정 금지.
   포맷: `## [YYYY-MM-DD] <operation> | <description>`

   ## [<TODAY>] init | vault initialized at <TARGET>
   ```

   `$TARGET/wiki/overview.md`:
   ```markdown
   ---
   title: Overview
   type: overview
   sources: []
   related: []
   created: <TODAY>
   updated: <TODAY>
   confidence: low
   ---

   # Overview

   지식 베이스 전체에 대한 high-level 합성.
   충분한 페이지가 쌓이면 다시 작성한다.

   ## 주요 테마

   _아직 ingest된 자료가 없음._

   ## 미해결 질문

   _TBD._

   ## 큰 그림

   _TBD._
   ```

5. Suggest Obsidian config (don't write — just tell user):

   ```
   Obsidian을 사용한다면 .obsidian/app.json 에 다음을 추가:
     "attachmentFolderPath": "raw/assets"
   ```

6. Report:
   ```
   LLM Wiki initialized at <TARGET>.
   Schema: <TARGET>/CLAUDE.md
   Next: /wiki ingest-project <project-path>  또는  /wiki ingest-url <url>
   ```

---

### /wiki ingest-project <project-path> [name]

Pipeline: project graphify → vault raw/repos snapshot → curated wiki summary.

**Args:**
- `<project-path>` — absolute or `~`-relative project directory
- `[name]` — optional. Defaults to `basename "$project-path"`

**Steps:**

1. **Resolve and validate:**
   ```bash
   PROJECT="$(cd "${1/#\~/$HOME}" 2>/dev/null && pwd)" || { echo "project path not found: $1"; exit 1; }
   NAME="${2:-$(basename "$PROJECT")}"
   DEST="$VAULT/raw/repos/$NAME"
   ```

2. **Pre-flight check:** If `$PROJECT/graphify-out/graph.json` is missing, stop with:
   > graphify hasn't been run in $PROJECT yet. Run `/graphify .` in the project first, then re-invoke /wiki ingest-project.

3. **Refresh graphify output in the project** (incremental — fast if no source changes).

   `graphify`는 community 라벨링을 단일 명령으로 처리하지 않는다 (그쪽 SKILL의 Step 5는 외부 LLM이 채워야 하는 수동 단계). 만약 `.graphify_labels.json`이 없거나 community 수와 어긋난 상태에서 `--wiki`를 같이 돌리면 `graphify/wiki.py`가 `Community {cid}` placeholder를 파일명으로 굳혀버리고 (`_safe_filename(label)` → `Community_0.md` 등), 매 export마다 기존 `wiki/*.md`를 모두 삭제 후 재생성하므로 이전의 사람-라벨까지 함께 날아간다.

   그래서 3단계로 분리해 실행한다:

   **3a. graph만 먼저 갱신 (wiki export 제외):**
   ```bash
   (cd "$PROJECT" && graphify . --update --no-viz)
   ```

   **3b. community 라벨 검증 및 보강:**
   ```bash
   ANALYSIS="$PROJECT/graphify-out/.graphify_analysis.json"
   LABELS="$PROJECT/graphify-out/.graphify_labels.json"
   [ -f "$ANALYSIS" ] || { echo "analysis missing — graphify did not produce communities. abort."; exit 1; }
   ```

   - `$ANALYSIS`의 `communities` 키 목록과 `$LABELS`(있다면)의 키 목록을 비교한다.
   - `$LABELS`가 없거나, community 키와 라벨 키 집합이 다르거나, 어떤 라벨이 `^Community \d+$` 패턴이면 → **누락된 cid 각각에 대해** `$ANALYSIS.communities[cid]` 안의 node label들을 읽고 2~5단어의 사람이 읽을 수 있는 이름을 만들어 라벨 dict에 채운다.
   - 채운 결과로 `$LABELS`를 덮어쓴다 (JSON, key는 문자열, `ensure_ascii=False` 상응).
   - 라벨이 이미 완전하고 placeholder가 없으면 이 단계는 no-op.

   **3c. 검증된 라벨로 wiki를 export:**
   ```bash
   (cd "$PROJECT" && graphify export wiki)
   ```

   이 시점에 `$PROJECT/graphify-out/wiki/` 의 파일명은 모두 사람-라벨 기반이다. `Community_N.md` / `Cluster_N.md` 파일이 하나라도 보이면 3b가 실패한 것이므로 중단하고 사용자에게 보고한다.

4. **Snapshot to vault raw layer:**
   ```bash
   mkdir -p "$DEST"
   rsync -a --delete "$PROJECT/graphify-out/wiki/" "$DEST/"
   COMMIT=$(cd "$PROJECT" && git rev-parse --short HEAD 2>/dev/null || echo n/a)
   FILES=$(find "$DEST" -type f -name '*.md' | wc -l | tr -d ' ')
   OP=sync MSG="raw/repos/$NAME ($COMMIT, $FILES files)"
   printf '\n## [%s] %s | %s\n' "$TODAY" "$OP" "$MSG" >> "$VAULT/wiki/log.md"
   ```

5. **Synthesize `wiki/sources/summary-$NAME.md`:**

   Read `$DEST/index.md` (graphify-generated wiki index) and the top community pages by node count. Then write `$VAULT/wiki/sources/summary-$NAME.md` with frontmatter:

   ```yaml
   ---
   title: "Summary: <NAME>"
   type: source-summary
   sources:
     - raw/repos/<NAME>/
   related: []
   created: <TODAY>
   updated: <TODAY>
   confidence: high
   ---
   ```

   Body sections (in order):
   - **TL;DR** — 1-2 sentences. What is this project?
   - **주요 모듈** — list of god nodes / large communities with one-line purpose each
   - **진입점** — entry-point files or APIs (if identifiable from the graph)
   - **외부 의존성** — key external libs/services
   - **인용 가능한 발췌** — short snippets that capture key claims
   - **Open questions** — ambiguities or gaps

6. **Create / update concept and entity pages — sparingly:**

   For named concepts that appear in multiple communities AND are reusable across other sources, create `$VAULT/wiki/concepts/<kebab-case>.md`. Add the new source to its `sources:`.

   For named organizations, products, or people, create `$VAULT/wiki/entities/<kebab-case>.md`.

   **Do NOT create a page for every god node.** Only create when the concept/entity is genuinely cross-cutting. Otherwise just mention in the summary. Page proliferation hurts retrieval.

7. **Update `wiki/index.md`** — add new pages under their categories. Preserve existing entries.

8. **Append to log:**
   ```bash
   OP=ingest MSG="$NAME → wiki/sources/summary-$NAME.md (+$NEW_C concepts, +$NEW_E entities)"
   printf '\n## [%s] %s | %s\n' "$TODAY" "$OP" "$MSG" >> "$VAULT/wiki/log.md"
   ```

9. **Report:**
   ```
   Ingested $NAME.
     raw/repos/$NAME/                ($FILES files synced, commit $COMMIT)
     wiki/sources/summary-$NAME.md   (created/updated)
     wiki/concepts/...               ($NEW_C new, $UPD_C updated)
     wiki/entities/...               ($NEW_E new, $UPD_E updated)
     wiki/index.md                   (updated)
   ```

---

### /wiki ingest-url <url>

1. Use the WebFetch tool to fetch and convert to markdown.
2. Derive slug from URL or title. Filename: `$TODAY-<slug>.md`.
3. Save raw to `$VAULT/raw/articles/$TODAY-<slug>.md` with a "Source:" line near the top.
4. Synthesize `$VAULT/wiki/sources/summary-<slug>.md` — same shape as ingest-project (TL;DR, claims, quotes, open questions). frontmatter `sources: [raw/articles/$TODAY-<slug>.md]`.
5. Create/update concept and entity pages only if genuinely reusable.
6. Update `wiki/index.md`.
7. Append log: `OP=ingest MSG="<url> → summary-<slug>"`

---

### /wiki ingest-file <path>

1. Detect kind by extension:
   - `.pdf` → `raw/papers/`
   - `.md`, `.txt` → `raw/articles/`
   - `.csv`, `.json`, `.tsv` → `raw/data/`
   - `.png`, `.jpg`, `.jpeg`, `.webp`, `.svg` → `raw/images/`
   - other → ask user where it goes
2. `cp "$PATH" "$VAULT/raw/<kind>/$(basename "$PATH")"`. If collision, ask user.
3. Same summarize + concepts/entities + index + log flow as ingest-url.

---

### /wiki query "<question>"

1. Read `$VAULT/wiki/index.md`.
2. Pick 3-5 candidate pages by keyword relevance to the question.
3. Read those pages; follow `related:` and `sources:` as needed.
4. Compose answer **with citations to wiki page paths** (e.g. `wiki/concepts/foo.md`).
5. If the wiki lacks enough material, say so explicitly — do not hallucinate.
6. If your synthesis is genuinely new and reusable (not already in any page), save it:
   - Concept-level → new/updated `wiki/concepts/...md`
   - Cross-page comparison → new `wiki/comparisons/...md`
   - Update `wiki/index.md` to register
7. Append log: `OP=query MSG="<question>"` (truncate question to ~80 chars)

---

### /wiki lint

Run these checks across `$VAULT/wiki/**/*.md` (skip `index.md`, `log.md`, `overview.md`):

1. **Orphans** — pages with zero inbound links from other wiki pages. Use grep to count `[text](path)` and `[[wikilink]]` references.
2. **Frontmatter** — missing required fields: `title`, `type`, `sources`, `created`, `updated`. Type-specific fields too.
3. **Broken sources** — for each page's `sources:` array, verify each path exists relative to `$VAULT`.
4. **Stale** — `updated:` more than 90 days before `$TODAY`.

Report findings as a checklist. Auto-fix only safe items:
- Missing `updated:` → add with current date
- Missing `confidence:` on non-index pages → add `medium`

For everything else, ask the user.

Append log: `OP=lint MSG="$ORPHANS orphan / $FM frontmatter / $SRC sources / $STALE stale"`

---

### /wiki overview

Rewrite `$VAULT/wiki/overview.md` from current wiki state.

1. Read `wiki/index.md`, then every `wiki/concepts/*.md` and `wiki/entities/*.md`.
2. Identify:
   - **주요 테마** — clusters of related concepts (3-5 themes)
   - **미해결 질문** — collected from "Open questions" sections of source summaries
   - **큰 그림** — what shape is the knowledge taking?
3. Overwrite `wiki/overview.md`:
   ```yaml
   ---
   title: Overview
   type: overview
   sources: []
   related: [<top 5 concept paths>]
   created: <preserve from existing overview>
   updated: <TODAY>
   confidence: medium
   ---
   ```
   Body: 주요 테마 / 미해결 질문 / 큰 그림 sections.
4. Append log: `OP=synthesis MSG="overview rewrite — N themes, M open questions"`

---

## Absolute rules

- Never edit files under `$VAULT/raw/` — that directory is immutable
- Never edit past entries in `$VAULT/wiki/log.md` — append-only
- Never create wiki pages without YAML frontmatter
- Never create new wiki pages without registering in `$VAULT/wiki/index.md`
- **Never run `graphify` inside the vault** — graphify always runs in the source project; the vault is consumer-only
- When in doubt about a new concept page vs. embedding in a summary, prefer embedding — page proliferation hurts retrieval
- For non-init commands, if `$VAULT/CLAUDE.md` is missing, tell the user to run `/wiki init` first and stop

## Honesty rules

- If a source summary doesn't have enough material for a section, write `_TBD_` — don't fabricate
- Mark `confidence: low` when synthesis is speculative
- Cite source paths in answers — never present synthesis without provenance

---
> Source: [dbsdud/llm-wiki-skill](https://github.com/dbsdud/llm-wiki-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
