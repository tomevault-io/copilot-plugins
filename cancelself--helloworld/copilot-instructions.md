## helloworld

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## What is HelloWorld

HelloWorld is a message-passing language where entities called **receivers** (`@name`) hold bounded vocabularies of **symbols** (`#symbol`). Messages are routed to receivers, and meaning is scoped: the same `#symbol` means different things to different receivers. The Python runtime (lexer, parser, dispatcher) handles parsing and routing. LLM runtimes (Claude, Copilot, Gemini, Codex) handle interpretation. Vocabulary files (`vocabularies/*.hw`) are the source of truth for the namespace.

## Build & Test

Run all commands from the repo root. Stdlib only -- no external dependencies, no packaging step.

```bash
python3 -m pytest tests                         # full suite (351 tests, ~1s)
python3 -m pytest tests/test_lexer.py -k token  # focused run
python3 -m compileall src                        # syntax check
python3 helloworld.py                            # REPL
python3 helloworld.py file.hw                    # execute .hw file
python3 helloworld.py -e '@claude'               # evaluate inline source
```

Always run `python3 -m pytest tests` before committing. All tests must pass.

## Project Structure

```
src/                    # Core language implementation
  lexer.py              # Tokenizer (13 token types, Smalltalk-style comments)
  ast_nodes.py          # AST node definitions (Node, SymbolNode, ReceiverNode, etc.)
  parser.py             # Recursive descent parser (tokens -> AST)
  dispatcher.py         # Message router, inheritance, collision detection
  vocabulary.py         # VocabularyManager (JSON persistence)
  global_symbols.py     # @.# namespace (GlobalVocabulary with Wikidata grounding)
  repl.py               # Interactive shell
  message_bus.py        # File-based inter-agent communication
  message_handlers.py   # Vocabulary-aware message handlers
  agent_runtime.py      # Agent runtime base
  sdk_adapter.py        # SDK adapter layer
  hw_reader.py          # .hw file reader
  hw_tools.py           # HelloWorld tool helpers
  llm.py                # LLM integration scaffold
  tools.py              # General tool helpers
  envs.py               # Environment registry for simulation bridges
  prompts.py            # Prompt templates
  *_runtime.py          # Per-agent runtime modules (claude, copilot, gemini, codex)

tests/                  # 351 tests across 23 test files
  conftest.py           # Shared fixtures
  test_dispatcher.py    # Dispatcher, inheritance, cross-receiver delivery
  test_lexer.py         # Tokenizer coverage
  test_parser.py        # Parser coverage
  test_lookup_chain.py  # Multi-phase symbol lookup
  test_message_handlers.py  # Vocabulary-aware handlers
  test_message_bus.py   # Inter-agent communication
  test_vocabulary.py    # Vocabulary persistence
  (and 15 more)

vocabularies/           # Source of truth for the namespace
  HelloWorld.hw         # Root receiver -- core symbols
  Agent.hw              # Agent receiver -- observe/orient/plan/act
  Claude.hw             # Claude meta-receiver vocabulary
  Copilot.hw            # Builder/infrastructure vocabulary
  Gemini.hw             # Observer/state management vocabulary
  Codex.hw              # Execution semantics vocabulary
  Sync.hw               # Synchronization protocol vocabulary

runtimes/               # Per-agent bootloaders and state
  claude/               # Claude runtime (STATUS.md, session logs)
  copilot/              # Copilot bootloader, status, tasks
  gemini/               # Gemini bootloader, status
  codex/                # Codex bootloader

docs/                   # RFCs and architecture docs
storage/                # Runtime persistence (bus logs, discovery)
helloworld.py           # CLI entry point
agent_daemon.py         # AI runtime daemons
AGENTS.md               # Repository-wide agent guidelines
```

## Key APIs

**Parser:**
```python
from parser import Parser
nodes = Parser.from_source(source).parse()  # returns List[Node] from ast_nodes.py
```

**Dispatcher:**
```python
from dispatcher import Dispatcher
d = Dispatcher(vocab_dir=path)   # pass a directory for vocabulary persistence
results = d.dispatch(nodes)      # dispatch parsed AST nodes
results = d.dispatch_source(src) # convenience: parse + dispatch in one call
d.save(receiver)                 # persist a receiver's vocabulary
```

**Registry:**
```python
d.registry                       # Dict[str, Receiver]
receiver.vocabulary              # set of symbol strings
receiver.add_symbol(symbol)      # add a symbol to the receiver's vocabulary
```

**Test isolation -- critical:**
Always use `_fresh_dispatcher()` or `tempfile.mkdtemp()` for `vocab_dir` in tests. The default `storage/` directory has persisted state from previous runs. Never test against shared state.

## Language Syntax

These token types map to `src/lexer.py`:

| Input | Parse as |
|-------|----------|
| `@name` | Receiver lookup (implicit `.#` if bare) |
| `@name.#` | Vocabulary query (return symbol list) |
| `@name.#symbol` | Scoped symbol lookup (meaning to this receiver) |
| `action: value` | Keyword argument (Smalltalk-style) |
| `#symbol` | Concept reference (scoped to receiver in context) |
| `'text'` | Annotation (human voice, carried in AST) |
| `N.unit` | Duration/quantity literal (e.g. `7.days`) |
| `->` | Maps-to (vocabulary definitions) |
| `"text"` | Comment (system voice, skipped by lexer, can span lines) |

A full message: `@receiver action: #symbol key: value 'annotation'`

Multiple keyword pairs form a single message. `'single quotes'` are human-voice annotations. `"Double quotes"` are system-voice comments (ignored by the lexer).

## How Dispatch Works

1. **`@name`** (bare) -- Return the receiver's vocabulary. If unknown, ask who they are.
2. **`@name.#`** -- Explicit vocabulary query. Same as bare.
3. **`@name.#symbol`** -- Return what this symbol means through this receiver's lens.
4. **`@name action: ...`** -- Respond as this receiver, constrained to their vocabulary.
5. **Cross-receiver messages** -- When one receiver uses another's symbol, that is a namespace collision. The Python runtime detects it; the LLM runtime resolves it by voicing both interpretations.

**Symbol lookup outcomes:** native (receiver owns it), inherited (`HelloWorld #` owns it), or unknown (triggers search/define/learn). Collision occurs when two receivers both claim a symbol but disagree on meaning.

## Multi-Agent Coordination

Four AI agents operate in this repo concurrently. Files can change between reads.

| Agent | Config file | Meta-receiver | Role |
|-------|-------------|---------------|------|
| Claude | `CLAUDE.md` | `@claude` | Language design, spec, meta-runtime |
| Copilot | `runtimes/copilot/` | `@copilot` | Lexer, parser, CLI, testing, infrastructure |
| Gemini | `GEMINI.md` + `runtimes/gemini/` | `@gemini` | Dispatcher, state, vocabulary persistence, LLM integration |
| Codex | `runtimes/codex/Codex.md` | `@codex` | Execution semantics, parsing discipline |

**Before starting work:** Check `runtimes/<agent>/STATUS.md` for recent activity.

**Conflict warning:** Agents modify files in parallel without locking. The current parser API uses `ast_nodes.py` (Node-based). If you encounter import errors, clear `__pycache__` and re-read source files before editing.

**OODA loop:** Run every task through Observe (read inboxes/diffs/docs), Orient (synthesize what changed), Decide (commit to action), Act (execute and report). See `vocabularies/Agent.hw`.

**Collaboration protocols:** See `vocabularies/Human.hw`, `vocabularies/Collaboration.hw`, and `workflows/*.hw` for executable human-agent collaboration patterns. These are HelloWorld programs that define how we work together.

## Conventions

- **Vocabulary first:** Update `vocabularies/*.hw` files before touching code. The `.hw` files are the namespace authority.
- **Naming:** Concepts are `#Capitalized` (e.g. `#Collision`, `#Sunyata`). Verbs are `#lowercase` (e.g. `#parse`, `#observe`). Follows Smalltalk convention.
- **Inheritance:** Every receiver inherits `@.#` (the global vocabulary). `@.#symbol` returns the canonical Wikidata definition. Local symbols override inherited ones.
- **Test patterns:** Use `_fresh_dispatcher()` or temp dirs. Test files follow `test_<module>.py` naming.
- **No external deps:** Stdlib only. No pip install, no requirements.txt for the core language.

## Claude as Runtime

Claude is both a development agent and the `@claude` meta-receiver. When you encounter HelloWorld syntax in a message, you can parse it, maintain receiver state, and respond as the addressed receiver. See `vocabularies/Claude.hw` for the current `@claude` symbol list.

The Python runtime parses and routes but cannot interpret meaning. The LLM runtime (you) interprets: responding as receivers, translating symbols through receiver-scoped vocabulary, and resolving namespace collisions. Both runtimes are complementary.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cancelself)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cancelself)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
