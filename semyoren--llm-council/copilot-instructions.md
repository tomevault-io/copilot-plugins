## llm-council

> Forked from karpathy/llm-council and rebuilt into a council of models

# CLAUDE.md - LLM Council

Forked from karpathy/llm-council and rebuilt into a council of models
for design decisions under constraints. The upstream remote is kept as
`upstream`; this project does not contribute back.

## What it does

A question goes to every council member in parallel. They answer
independently, then review each other's answers anonymously, then a
chairman synthesizes a verdict. Every model call is persisted, priced and
retryable on its own.

## The council

The roster is a user setting, not a constant, and nothing in the code
depends on its size: three members or ten, the same code runs. The floor
is two, because peer review of a single answer is not review.

The default is one model per vendor, so no two members share a training
lineage and agreement between them carries more signal than agreement
inside a family would. That is a default worth defending, not a
constraint the code enforces - two models from one vendor are a
legitimate council if the difference between them is the measurement.

The council, the chairman, the endpoint, the key and the title model are
edited in Settings and stored in the database. `backend/config.py` holds
only what a fresh install starts from.

## Stages

- **solve** - all members answer in parallel, with web search enabled
- **review** - members rank the anonymized answers
- **verdict** - the chairman writes the final answer

Anonymous labels go only to models that actually answered, so four
answers means A-D with no gap and a reviewer never speculates about a
missing one. They run A-Z and then AA, AB like spreadsheet columns:
`chr(65 + index)` walks into punctuation past the 26th member, and a
"Response [" is something no reviewer can cite and no parser can match.

## Runs are background objects

A run is an asyncio task owned by `backend/runner.py`, not something
living inside the request that started it. The HTTP handler only
subscribes. Closing that response - a reload, switching conversation,
a dropped connection - does not stop work that is already being paid
for, and several conversations can run at once.

Every event a run emits is buffered, so a client attaching late gets a
replay and sees the stages it missed rather than an empty screen.

Runs are in memory, so a restart orphans them: on startup anything left
`running` in the database is closed out as interrupted. Without that it
would spin in the sidebar forever.

## Decisions that are not obvious from the code

**The council is configuration, not code.** It used to live in
`config.py`, which meant a Python module read once at import: editing the
file changed nothing until the process was restarted, and the UI kept
showing the council the running process had loaded. Settings live in the
database and are read per run, so a change applies to the next run with
no restart.

**A key is not accepted on the user's word.** Setup and Settings will
not commit until the key has answered the endpoint it was typed next to.
The check runs against `/key`, not `/models`: OpenRouter's model list is
public and returns 200 with no key at all, so a check against it would
pass for any string. What is remembered is the (url, key) pair - editing
either half asks for proof again, because a key is only ever valid for
one endpoint.

**Nothing falls back to the environment.** An absent settings row means
"never set up" and sends the user to the Setup page; it does not mean
"use the defaults". A council nobody chose is worse than no council,
because it spends money under someone else's assumptions.

**A run pins one settings snapshot.** The configuration is read once when
the run starts, never per stage. Otherwise an edit made while solving
would change who reviews, and the anonymous labels would stop describing
the answers they were handed out for.

**A retried call uses the settings of now, not of the run.** A key
rotated after a failure is exactly why the retry is being asked for.

**Effort is chosen per role**, not per run or per model: solving is
research, reviewing is checking, and they should not cost the same. The
four levels are ours; vendors map them onto their own scales.

**Requested and actual effort are separate fields.** Providers do not
reliably report the level they applied. When nothing comes back the UI
says "not confirmed" rather than echoing the request as a fact.

**Search is pinned to one engine for every model.** Left to defaults,
OpenRouter sends each vendor to its own native search, and the
comparison would measure "model + its search engine" instead of the
model.

**The council's size is bounded by cost, not by code.** Every reviewer
is shown every answer, so prompt volume grows with the square of the
roster - ten members is ten reviews of ten answers. That is the real
ceiling, and it is a billing one.

**The ranking parser is built from the labels actually handed out**,
longest first, so "AA" is never read as "A" and a word that merely
starts with a letter is never read as a label. Matching a bare `[A-Z]`
after "Response" counted "Response ABOVE" as a vote for A.

**Each reviewer sees its own random ordering** of the answers, and the
permutation is stored. Upstream showed one shared order to everyone, so
position bias pushed every ranking the same way instead of averaging
out.

**The aggregate ranking is computed from stored votes, never stored.**
A derived value kept beside its inputs eventually disagrees with them.
Ties share a rank and are broken by first-place counts.

**Cost comes from the provider's usage accounting**, not from a local
price table, so it cannot drift from real billing on cache or promos.

**The chairman never sees model names, and is not a council member.**
It writes the verdict at the one step nobody else reviews, so a member
in that seat would be grading its own paper. Settings refuses the
overlap rather than resolving it silently: whichever way it were
resolved, the stored roster would stop describing what actually ran.

**Anonymity does not remove self-preference, and this is measured.**
Across the runs in this database, a model places its own answer higher
than the rest of the council placed the same answer by +0.12 on a 0-1
rank scale, paired per answer so it is not a reflection of who writes
better. n=84, t=4.3. It varies sharply by model - one shows +0.30,
another +0.00 - so it is a property of the model rather than a law.
`repository.judge_stats` computes it from stored votes and Settings
shows it beside the roster, where it is acted on.

**The chairman's answers are shuffled too.** Reviewers each got their
own permutation from the start, but the chairman used to receive the
roster order every run, leaving position bias unchecked at the most
consequential step.

**Attachments become text server-side.** The transport accepts no file
uploads. Extraction is lossy in ways that are invisible downstream, so
the extracted text is stored and shown before an expensive run starts.

**Conversation history carries only questions and verdicts**, never
full stage transcripts, which would blow the context open on the second
turn. It is folded into the question as text rather than replayed as
user/assistant turns - see the gotcha below; the shape is forced by the
search plugin, and it has the side benefit that every vendor receives a
byte-identical message list.

**The completed-in-background mark is client-side and deliberately not
persisted.** It means "this finished while you were watching", so a
reload must not resurrect it for every unread run in the database. The
server does track `seen_at`, but the green mark in the sidebar comes
from what the open page observed.

**Interactive things are the elements they behave like.** A conversation
row is a `Link`, so it has an address that middle-click and
open-in-new-tab can use and the keyboard can reach; the row's menu
button sits beside the link rather than inside it, because a button
inside an anchor is invalid markup and swallows the click. A stage
header is a `button` with `aria-expanded`, not a div that toggles - a
section that only opens for a mouse hides its contents from the keyboard
entirely. Both suppress the browser's focus ring and draw the palette's.

**Stage sections start collapsed** - answers, peer review and stats.
A run is several screens of material and the verdict is what the user
came for, so only the verdict opens by itself. Header badges still show
progress without expanding.

**A closed run has no live calls.** Enforced on every startup, not just
when that startup orphaned something: a call whose run is over can never
be written again, so left `running` it renders as a stage forever about
to finish - a verdict eternally being synthesized - that no reload
clears. Rows stranded by an earlier path are exactly the ones nobody
notices, which is why the sweep is unconditional.

**Stop aborts the task, not a flag.** Cancellation used to be checked
between stages, so on Ultra the button did nothing visible for minutes.
Now the run task is cancelled outright; calls already returned are kept
and unfinished ones are marked cancelled rather than left running.

**A finished run exports to Markdown, HTML or PDF.** The verdict leads
and the answers, reviews and standing follow as its evidence. A council
run is expensive; a result that cannot leave the app is hard to act on.

## Addresses

The open conversation is the URL - `/c/<id>` - not state sitting beside
it. A reload lands on the same run, back and forward work, and the
address is worth copying somewhere. Both routes render the same element
so switching conversations does not remount the app and tear down the
run subscriptions with it. An id that no longer exists replaces itself
with `/`, because a link that can never resolve is worse than no link.

## Layout

- `backend/config.py` - defaults for a fresh install, effort, timeouts
- `backend/settings.py` - the stored council, its validation and defaults
- `backend/openrouter.py` - one call, with retries, usage and provider
- `backend/council.py` - the three stages and the aggregate
- `backend/prompts.py` - prompt text and ranking parsing
- `backend/repository.py` - queries and API serialization
- `backend/runner.py` - background runs, event fan-out, replay
- `backend/export.py` - Markdown, HTML and PDF rendering
- `frontend/src/components/RunRail.jsx` - run navigation rail
- `frontend/src/lib/download.js` - client-side verdict export
- `backend/db/` - SQLAlchemy models and session factory
- `backend/Dockerfile`, `frontend/Dockerfile`, `frontend/nginx.conf`,
  `docker-compose.yaml` - the containerized stack
- `docs/screens/` - README screenshots, taken against synthetic data
- `frontend/src/components/` - sidebar, composer, run view
- `frontend/src/components/SettingsFields.jsx` - the settings form, shared
  by the modal and the first-run page
- `frontend/src/lib/settings.js` - theme, validation, defaults
- `frontend/src/lib/useKeyCheck.js` - live proof that a key works
- `frontend/src/components/JudgeStats.jsx` - measured neutrality per model

## Storage

SQLite through SQLAlchemy, migrated with Alembic. WAL is required: a run
writes from many concurrent tasks and SQLite allows one writer. Moving
to Postgres later is a URL change.

## Running

Backend on port 8001, frontend on 5173. Start the backend as a module
from the project root, not from inside `backend/`.

Under Docker it is one origin: nginx serves the built app and forwards
`/api` to the backend, so nothing has to be told where the backend is
and there is no cross-origin request to configure. The frontend calls
`/api` on its own origin everywhere - the Vite dev server proxies the
same path, so development and production differ in who forwards, not in
what the app requests.

`data/` is bind-mounted rather than a named volume: the database is the
council's history and its cost, and it belongs in the project rather
than in Docker's storage. The backend port is not published - the app
does not use it, and 8001 collides with whatever else the host runs.

## Theme

Three modes - light, dark, auto - chosen in Settings, and the only part
of the configuration that stays client-side: it is a property of the
browser looking at the app, not of the council. `auto` is a standing
subscription to `prefers-color-scheme`, not a reading taken once at save
time. It is applied on commit only, so editing the control in an open
form does not repaint the app behind it, and painted in `main.jsx`
before React mounts, so a reload does not flash white.

**A call's prompt is stored before it is sent**, not with its result. A
call that never returns is exactly the one worth retrying, and it has no
result to be written alongside - so writing the prompt at the end left
interrupted calls with an empty `prompt` and a Retry button that could
do nothing.

**A call's timeout is a wall-clock cap, not the transport's.** httpx
counts its timeout per read and restarts the clock on every byte, so a
provider that keeps the socket warm holds a call open for as long as it
likes: one review call ran 63 minutes against a 300 second limit, still
on its first attempt, with nothing in the code able to stop it. Each
attempt is wrapped in `asyncio.wait_for`, which makes the numbers in
`config.py` mean what they look like and bounds a stage at
`attempts x timeout` plus backoff.

## Gotchas

- Every call goes out as exactly one `user` message. Search is on, and
  OpenRouter's web plugin injects its results as a `system` message;
  Anthropic rejects one that lands after an assistant turn, so real
  multi-turn roles made the second question of every conversation fail
  with HTTP 400 for Anthropic members while every other vendor answered
  - a council that silently shrank on each follow-up. History is folded
  into the question instead. `normalize_messages` folds prompts stored
  before that too, so retrying an old failure does not reproduce it.
- SSE frames split across chunks. The client buffers between reads;
  decoding each chunk alone silently drops whole stages.
- A run keeps a cancel flag. Cancellation takes effect between stages -
  calls already in flight are paid for either way, so their results are
  kept.
- Only one run per conversation at a time; starting a second is a 409.
- HTTP headers are latin-1, and conversation titles routinely are not.
  Export filenames need the RFC 5987 form plus an ASCII fallback.
- Error bodies are stored clipped, so the JSON inside them is often
  incomplete. The UI recovers the message without parsing it.
- `DialogContent` is a grid by default. With a `max-h` on it the body
  sizes to its content and the footer lands below the dialog's own
  bottom edge - Save included, unreachable. It has to be a flex column
  with `min-h-0` on the scrolling body.
- `localhost` resolves to `::1` first here. Any other service holding
  that address on the backend's port answers instead of the council's -
  a 404 from a stranger, with nothing to say it came from elsewhere. The
  dev proxy targets `127.0.0.1` explicitly.
- nginx buffers proxied responses by default and times reads out at 60s.
  A run streams for minutes with silences between stages, so the SSE
  location needs `proxy_buffering off` and a long read timeout, or the
  UI shows nothing and then the stream dies mid-run.
- Never open `data/council.db` from the host while the stack is up. The
  database is WAL and bind-mounted, and WAL coordinates writers through
  shared memory that a macOS host and the container's Linux VM do not
  share coherently. Querying it from the host during a run produced
  `sqlite3.OperationalError: disk I/O error` inside the container, which
  killed the run task and left its row `running` with nobody left to
  finish it. Read it with `docker compose exec backend python` instead.
- SQLite stores no UTC offset, so timestamps come back naive even
  though every one was written in UTC. Serialized bare they are read as
  local time by the browser: the displayed clock is wrong by the
  viewer's offset and an elapsed counter is wrong by the same amount in
  the other direction. `_iso` stamps the zone back on.
- An empty API key must not become an empty `Authorization` header:
  httpx rejects a bare `Bearer ` as an illegal header value, and the run
  dies in the transport with a message that never mentions the key.
- Model ids and URLs are ASCII by construction, and a Cyrillic character
  that renders identically to its Latin twin surfaces as a provider
  error minutes into an expensive run. Both the form and the API reject
  non-ASCII rather than passing it through.
- Every floating surface shares one frame through `.overlay-border` in
  `index.css`, not a repeated Tailwind utility. Two components carrying
  the same opacity by coincidence stop matching the first time one of
  them is edited, and a third is added without either.
- shadcn components reference more theme tokens than a hand-written
  palette defines. A missing one fails silently - a popover with no
  background rather than an error.
- `transition: all` on anything that re-renders often never settles: the
  size change restarts on every render. Transition named properties.
- Tailwind does not resolve `var()` inside an arbitrary `shadow-[...]`,
  and arbitrary sizes lost the cascade here. Rail styling lives in
  plain CSS in `index.css` for that reason.
- The async clipboard refuses when the document is not focused, so the
  copy buttons fall back to `execCommand`.
- Long text in a textarea scrolls straight through `padding-bottom`;
  controls have to sit outside the text box, not float over it.

---
> Source: [semyoren/llm-council](https://github.com/semyoren/llm-council) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
