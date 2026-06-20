## revise-skill

> Perform common code quality revisions on AI-drafted code. Use when reviewing recently drafted code for readability and maintainability improvements. Use when asked to learn new revise patterns from a commit or diff.


# Revise

Perform common code quality improvement revisions on code that was recently drafted by AI.

## Why this skill exists

AI-written code is usually drafted in a way that is easy for an AI to *write* at the time but not necessarily easy to *read* later (by human or AI). For example:

- AIs frequently use local imports in Python code -- contrary to standard Python convention, but easy to write in isolation without considering the rest of the file.
- AIs tend to write low-level functions before high-level functions -- easy to write a high-level function when you already know the names of the low-level functions it will call, but less efficient for readers who always want to start reading at high-level entrypoints.

The goal of revising code is to optimize its organization for *readers* of the code.

## Input

`$ARGUMENTS` specifies what to review. Supported forms:

- **`uncommitted`** -- Review uncommitted changes, as reported by `git diff` and `git diff --staged`.
- **`last-commit`** -- Review the most recent git commit (`git diff HEAD~1`).
- **A list of files** -- Review the specified files in their entirety (for recently created or heavily modified files).
- **A list of functions or classes** -- Review the specified symbols in context.

If no argument is given, default to reviewing uncommitted changes.
If there are no uncommitted changes, default to reviewing the last commit.

## Procedure

1. **Identify files to review.** Collect all files from the diff or argument list that need review.

2. **Order files for review.** Review higher-level files first, before lower-level implementation details:
   1. **Documentation files** (README, RELEASE_NOTES, doc/**) -- Often explain new features/systems at the highest level, for users or developers.
   2. **Test files** (test/**) -- Define acceptance criteria for a feature and exercise new API surfaces.
   3. **Product code files** -- Among these, try to identify which are higher-level entrypoints and review those first.

3. **Pass 1 -- Immediate fixes.** Review each file in order. For each code smell found:
   - **Apply immediately** if the fix is small and localized (rename, inline, add comment, extract helper, etc.).
   - **Defer** if the fix would cause large diff traffic or is more like a new feature than a small revision. Record deferred fixes per file. Common reasons to defer:
     - **High diff traffic:** Reordering top-level sections/functions, changing indentation style. These touch many lines, easily conflict with smaller changes happening in parallel.
     - **Non-trivial new work:** Changing build systems, configuring new tooling (linters, typecheckers), adding assets. These are features, not revisions.

4. **Review point 1.** Present the immediate fixes for review before proceeding. Also present the list of unapplied deferred fixes for confirmation.

5. **Pass 2 -- Deferred fixes.** Apply the recorded deferred fixes, one file at a time.

6. **Review point 2.** Present the deferred fixes for review.

## Code smells

### Organization (file-level)

- **[Local imports in non-entrypoint modules](patterns/local_imports.md)** -- Project imports placed inside functions instead of at the top of the file.
  ```python
  def _open_window(title, diff_bytes, api):
      from gvc.renderer import render  # should be at top of file
  ```

- **[Functions ordered bottom-up](patterns/functions_ordered_bottom_up.md)** -- Callees defined above callers; reader hits details before the big picture. **Always check this whenever new functions have been added to an existing file** — the cursor-position drop-in is almost never the right place in the call tree.
  ```python
  def _build_title(args): ...    # helper defined first
  def main(): ...                # entry point buried below
  ```

- **[Unnecessarily public functions/methods](patterns/privatize_by_default.md)** -- Default functions/methods to `_`-prefixed private; make public only when externally needed. Small public APIs focus attention; private methods are easier to reason about and refactor.
  ```python
  class TestClient:
      def call(self, method: str): ...          # only used by ping/list_windows below
      def ping(self): return self.call("ping")  # -> _call
  ```

- **[Many functions with no grouping section](patterns/missing_sections.md)** -- A file has many top-level definitions (>7-10) with no visual grouping.

- **[Sections grouped by kind instead of feature/concern](patterns/sections_by_kind.md)** -- Headings like "Constants", "Data Model", "Public API" lump items by what they are, not what feature they serve.
  ```python
  # --- Constants ---       # serves 3 different features
  # --- Public API ---      # public vs. private is rarely useful
  ```

- **[Symmetric operations split across modules](patterns/symmetric_operations_colocated.md)** -- Send/receive, read/write, encode/decode pairs live in different modules instead of adjacent in the same one.
  ```python
  # _ipc.py has try_send(), but _gui.py has the receive logic inline
  ```

- **[Class-specific helpers not on the using class](patterns/helpers_on_class.md)** -- Module-level `_helper()` used only by one class in the same module. Hoist onto the class as `@staticmethod` to match the real scope.
  ```python
  class TestClient:
      def _call(self, ...):
          gvc_log = _read_gvc_log(...)   # -> self._read_gvc_log(...)
  ```

- **[Definitions/parameters not in visual/logical order](patterns/visual_logical_ordering.md)** -- Parameters (or declarations) listed in a different order than their visual/logical order.
  ```python
  def _open_window(raw: bytes, title: str, ...):  # title appears first in UI
  ```

### Organization (within-function)

- **[Single concern divided by blank line](patterns/split_paragraph.md)** -- A blank line separates two tightly coupled blocks (e.g., sequential error checks on the same operation).

- **[Multiple concerns not separated by blank lines](patterns/missing_paragraph_breaks.md)** -- Distinct phases of a function (validation, work, result) sit adjacent without a blank line between them, so the reader can't see phase boundaries.

- **[Multiple related paragraphs not grouped with anonymous block](patterns/grouped_paragraphs.md)** -- Multi-paragraph sections inside a function are labeled by a leading comment but have no visible end. Wrap in `if True:` (Python) or `{ ... }` (JS/TS/Java/C/C++) to delimit both ends.
  ```python
  # File menu
  if True:
      file_menu = AppKit.NSMenu.alloc().init()
      ...
      main_menu.insertItem_atIndex_(file_menu_item, 1)
  ```

- **[Long then-block with missing/minimal else-block](patterns/long_then_block.md)** -- A long block sits indented under `if <match>:` with the surrounding scope (function, loop, branch) having nothing meaningful after it. Invert to `if not <match>: <exit>` (`return`/`continue`/`raise`) so the body un-indents.
  ```python
  for i in range(menu.numberOfItems()):
      submenu = menu.itemAtIndex_(i).submenu()
      if submenu is not None and str(submenu.title()) == "Edit":
          ...  # 15 indented lines
          break
  ```

- **[Guard clause followed by peer alternative](patterns/symmetric_branches.md)** -- An `if/return` followed by the alternative case at the same level, when both branches are peer alternatives.
  ```python
  if try_send(sock_path, tmp_path):
      return
  subprocess.Popen(...)  # looks like main path, but is actually the alternative
  ```

### Good Names

An `mcp__revise__rename_symbol` tool is available for quickly renaming functions, variables, modules, and other symbols. Details: <reference/rename_symbol.md>

- **[Vague or generic names](patterns/vague_generic_names.md)** -- Name describes mechanism (`tmp_file`, `data`, `result`) rather than domain concept. Also applies to function verbs like `make_X`, `do_X`, `handle_X` that hide the mechanism and overclaim scope.
  ```python
  def write_tmp_file(raw: bytes, title: str) -> Path: ...
  def _make_editable() -> None: ...   # actually just symlinks one dir
  ```

- **[Name implies wrong type](patterns/name_implies_wrong_type.md)** -- Name suggests a collection or different type than what it actually holds.
  ```python
  LARGE_BYTES = 1_048_576   # sounds like a list of bytes, actually a count
  ```

- **[Abbreviations in API surfaces](patterns/abbreviations_in_api.md)** -- Abbreviated names in function parameters, class fields, or other API-like surfaces.
  ```python
  def render(fd: list[FileDiff], ld_info: LargeDiffInfo | None = None): ...
  ```

- **[Failable operation not named `try_X`](patterns/try_x_naming.md)** -- A failable operation whose bare name reads as fire-and-forget or raise-on-failure (`send`, `insert`, `parse`) lacks a `try_` prefix to signal that failure is returned as a value. Most load-bearing for `bool`-returning effect methods, where the type alone can't force callers to check.
  ```python
  def send(sock_path, request_filepath) -> bool: ...   # -> try_send; callers may treat as fire-and-forget
  ```

- **[Lifecycle teardown not named `close`](patterns/lifecycle_close_naming.md)** -- Prefer `close()` over `cleanup`/`teardown`/`dispose` for resource-releasing methods. Matches stdlib convention (`file.close`, `socket.close`) and interops with `contextlib.closing`.
  ```python
  class GvcSandbox:
      def cleanup(self) -> None: ...   # -> close()
  ```

- **[Function name references an uncommon implementation technique](patterns/implementation_technique_in_name.md)** -- A function/section name references a named technique that the file's likely readers won't recognize (e.g. FLIP for the animation technique). Rename to describe the domain action; move the technique reference (with link) into an implementation comment. Does NOT apply to commonly-recognized techniques (`debounce`, `memoize`).
  ```javascript
  function _flipAnimate(elements, mutate) { ... }   // -> _reorderOutlineRowsWithAnimation
  // section header "// Flip Animation"             // -> "// Reorder Animation"
  ```

- **[Short/generic/unscoped name in a shared namespace](patterns/unscoped_name_in_shared_namespace.md)** -- When the language/tooling doesn't supply per-file namespacing (JS without modules, CSS selectors, C top-level symbols), generic top-level names risk silent collision and read ambiguously at call sites. Prefix by subsystem/element/feature. For JS event handlers, name after the element the listener is attached to.
  ```javascript
  outline.addEventListener("keydown", _onHandleKeydown);   // -> _onOutlineKeydown
  ```
  ```css
  .outline-row { ... }                                     /* -> .gvc-outline-row */
  ```

### Clarity / Anti-Obscurity

- **[Magic numbers](patterns/magic_numbers.md)** -- Numeric literals used directly in logic where the meaning isn't obvious.
  ```python
  size = max(8, min(32, int(size)))
  ```

- **[Short CLI flags in subprocess calls](patterns/short_cli_flags.md)** -- Short flags like `-M`, `-w` in subprocess calls instead of self-documenting long flags.
  ```python
  cmd = ["git", "diff", "-M"] + args
  ```

- **[Docstring with implementation details](patterns/docstring_with_impl_details.md)** -- Docstring contains implementation details (why a workaround, why a specific library). Docstrings are for API guarantees; implementation rationale goes in `# NOTE:` comments.
  ```python
  def _is_dark_mode() -> bool:
      """... Uses NSUserDefaults rather than ..."""  # implementation detail, not API
  ```

- **[Obvious/redundant docstrings](patterns/obvious_docstrings.md)** -- Docstring restates what the function/test/module name already says. Especially common in AI-drafted test functions.
  ```python
  def test_user_can_log_in():
      """Test that a user can log in."""  # adds nothing
  ```

- **[Unprefixed comment is general commentary](patterns/unprefixed_comment_scope.md)** -- Unprefixed comments label *what* the immediately following code does. General commentary belongs under `NOTE:` or in documentation.
  ```python
  # Internal dispatch: the bundled executable is dual-purpose. When another
  # gvc process invokes it with `--gui-server <request_file>` (see below),
  # act as the persistent GUI server instead of the CLI.
  if args and args[0] == "--gui-server":
  ```

- **[Branch-scoped comment placed above the conditional](patterns/branch_scoped_comment.md)** -- When a comment explains *why one branch* of a conditional is taken, place it inside that branch — not above the whole `if`/`match`/ternary. Applies to unprefixed and prefixed comments alike.
  ```python
  # Unix socket paths have a 104-char limit on macOS...     # only applies to one branch
  return root / "runtime" if root is not None else default
  ```

- **[Missing clarifying comments on non-obvious code](patterns/clarifying_comments.md)** -- A code block responds to a situation non-obviously (e.g., silently swallowing errors), or a paragraph is 5-7+ lines with no label.
  ```python
  except (json.JSONDecodeError, TypeError):
      return cls()  # why? intentional? looks like a bug
  ```

- **[Truthy check for Optional values instead of `is not None`](patterns/is_not_none_over_truthy.md)** -- Truthy checks on Optional values silently misbehave on falsy-valid states (`Path("")`, `0`, empty collections). Use explicit `is not None`.
  ```python
  return root / "runtime" if root else default   # -> `if root is not None`
  ```

### Correctness / Safety

- **[Silent early return on failure](patterns/silent_early_return.md)** -- `return` on an unexpected condition without logging or raising leaves field debugging blind. Prefer `raise`; when log-and-return is right, format the message as `[subsystem] what happened. impact. remediation.`
  ```python
  if main_menu is None:
      return   # silent — future debug session has no starting point
  ```

- **[Manual resource cleanup](patterns/manual_resource_cleanup.md)** -- Explicit `close()`/`unlink()` instead of context managers or `try/finally`. Cleanup is skipped on exceptions.
  ```python
  server_sock = socket.socket(...)
  ...
  server_sock.close()  # skipped on exception
  ```

- **[Overscoped try-block](patterns/minimize_try_scope.md)** -- A `try:` block should contain *only* the failable statements. Move success-result assignments into an `else:` clause attached to the `try`.
  ```python
  try:
      _set_window_appearance(window, appearance)
      result = {"ok": None}      # not failable; belongs in `else:`
  except Exception as e:
      result = {"error": str(e)}
  ```

- **[Counter-based wait loops](patterns/deadline_over_counter_loop.md)** -- Time-based waits should use `time.monotonic()` deadlines, not counter-based loops. `time.sleep(0.1) * 30` can take far more than 3s under load; `time.time()` is not monotonic.
  ```python
  for _ in range(30):           # claims "3 seconds", drifts under load
      time.sleep(0.1)
  ```

- **[Unmarked reassignment → `# reinterpret`](patterns/unmarked_reassignment.md)** -- In Python and other non-final-by-default languages, mark intentional variable rebindings so reorders/edits treat them as anomalous.
  ```python
  text = _strip_frontmatter(text)   # -> # reinterpret
  ```

- **[Unmarked value-preserving rename → `# rename`](patterns/unmarked_rename.md)** -- Mark rebindings that change the *name* (same value) to match a narrower meaning that's just become known — otherwise the duplicate assignment reads as waste.
  ```python
  if item.action() == "orderFrontStandardAboutPanel:":
      about_item = item   # -> # rename
  ```

- **[Unmarked ordering-sensitive assignment → `# capture`](patterns/unmarked_capture.md)** -- Mark snapshots whose correctness depends on *when* the RHS is evaluated, so maintainers don't inline or reorder them.
  ```python
  deadline = time.monotonic() + timeout   # -> # capture
  ```

- **[Unmarked intentional copy → `# clone`](patterns/unmarked_clone.md)** -- Mark defensive copies (for iteration safety, mutation isolation) so they aren't "cleaned up" as wasteful.
  ```python
  for listener in list(self.listeners):   # -> # clone
  ```

- **[`let`/`var` used instead of `const` (JS/TS)](patterns/prefer_const_over_let.md)** -- In languages with concise final-by-default variables, default to `const`; let non-`const` itself be the mutability marker.
  ```typescript
  let result = computeThing();   // never reassigned — should be const
  ```

### Formatting & Style

- **[Em/en dashes in comments and text](patterns/em_en_dashes.md)** -- Dashes used to attach trailing fragments to sentences. Signature AI writing style.
  ```python
  # Stale socket file — remove it so the next launch starts fresh
  ```

- **[British English spelling](patterns/british_spelling.md)** -- British spelling in comments/docs (e.g., `behaviour`, `initialise`).

- **[Old-style string formatting instead of f-strings](patterns/prefer_fstrings.md)** -- `template % args` or `template.format(args)` where an f-string reads more directly. Watch the argparse exception: `%(default)s`/`%(prog)s` in `help=` are argparse's own interpolation, not the `%` operator.
  ```python
  help="use the launch code (%s) as the secret" % DEMO_PASSWORD   # -> f"...({DEMO_PASSWORD})..."
  ```

- **[Docstring uses imperative verb](patterns/docstring_verb_form.md)** -- Docstring starts with an imperative verb. Prefer third-person indicative.
  ```python
  def main() -> None:
      """Build ./dist/gvc.app via PyInstaller."""  # -> "Builds ..."
  ```

- **[Single-line if-statement](patterns/single_line_if_statement.md)** -- An `if` whose condition and non-trivial body share one line is easy to miss when scanning, and usually overflows line-length anyway. Break the body onto its own line. Short one-token guards (`return`/`break`/`continue`/`throw`) are an idiomatic exception.
  ```javascript
  if (e.dataTransfer) e.dataTransfer.dropEffect = "move";    // -> break body to next line
  if (rows.length === 0) break;                              // OK, idiomatic guard
  ```

- **[Multi-line if/loop body without braces (optional-brace languages)](patterns/optional_brace_omitted_on_multiline.md)** -- In JS/TS/C/C++/Java/Perl, omitting braces on a multi-line `if`/`else`/`for`/`while` invites goto-fail-style edit bugs (a second indented statement reads as part of the block but isn't). Always brace. Does not apply to Python (indentation is the scope) or Go/Rust (braces mandatory).
  ```javascript
  if (rows.length === 0)
      break;            // -> wrap break in braces
  ```

### Concision

- **[Duplicate code](patterns/duplicate_code.md)** -- Two+ blocks doing substantially the same thing, either >3 lines or far apart. Copies will eventually diverge. Either extract, OR when dedup is impossible/deferred, mark every copy with `NOTE: Duplicated in X and Y` (bidirectional, identical text at both sites).
  ```python
  # _render_file: icon default "⬜"    — already diverged from:
  # _render_outline: icon default "📄"
  ```

- **[Temporary variable with no explanatory value](patterns/temporary_variable.md)** -- Variable assigned once, used once, name adds nothing the expression doesn't already say.
  ```python
  prefs = Prefs.load()
  create_window(html_doc, title, prefs, api)
  ```

- **[if/else statement assigns to same variable](patterns/conditional_expression.md)** -- Both branches of an if/else assign to the same variable; expressions are short.
  ```python
  if large_diff_info is not None:
      html_doc = render(large_diff_info)
  else:
      html_doc = render(parse(diff_bytes))
  ```

- **[Dead code](patterns/dead_code.md)** -- Unused constants, unreachable branches, unused literal members, commented-out blocks. Misleads readers into thinking it matters.
  ```python
  _OLD_FILE = re.compile(r"^--- (?:a/)?(.+)$")   # never referenced
  ```

- **[Unnecessarily quoted type annotations](patterns/quoted_annotations.md)** -- Quoted annotation (`"Path | None"`) in a Python 3.14+ project where all annotations are already deferred.
  ```python
  def _enclosing_app_executable() -> "Path | None":  # quotes no longer needed
  ```

- **[Hand-rolled caching](patterns/hand_rolled_caching.md)** -- Module-level sentinels (`_FOO: str | None = None`) with `if _FOO is None` guard. Replace with `@cache`.
  ```python
  _CSS: str | None = None
  def _assets():
      global _CSS
      if _CSS is None: ...
  ```

### Type Design

- **[Data clump](patterns/data_clump.md)** -- Multiple functions share the same parameter bundle.
  ```python
  def write_gui_request_file(raw: bytes, title: str) -> Path: ...
  def read_gui_request_file(path: Path) -> tuple[bytes, str]: ...
  ```

- **[Conditionally-meaningful fields or parameters](patterns/conditionally_meaningful_fields.md)** -- Fields/params only meaningful when some condition holds (boolean flag, status literal).
  ```python
  @dataclass
  class FileDiff:
      status: Literal["added", "deleted", "modified", "large"]
      raw_size: int = 0    # only meaningful when status == "large"
      raw_lines: int = 0
  ```

- **[Ignored None parameter](patterns/ignored_none_parameter.md)** -- A function accepts `T | None` and silently no-ops on `None`. Tighten the parameter to `T` and push the `None` check to each call site, where the surrounding context can decide what to do.
  ```python
  def _run_js_in_window_titled(api: AppApi, title: str | None, js: str) -> None:
      if title is None:
          return   # silently refuses to do what the name says
      ...
  ```

### Type Safety

- **[Variant dispatch missing exhaustive fallback](patterns/variant_dispatch_nonexhaustive.md)** -- Switch-like dispatch (if/else chain, ternary, match) on a set of known variants without an exhaustive fallback. New variants silently fall through or get swallowed by the final `else`.
  ```python
  if isinstance(shape, Triangle): ...
  elif isinstance(shape, Square): ...
  else:  # Circle -- unsafe: silently swallows new variants
  ```
  ```typescript
  // ternary assumes "if not verb, must be keyword" -- unsafe
  const cls = (type === 'verb') ? 'verb-class' : 'keyword-class';
  ```

- **[`type: ignore` and `cast(...)` in typed code](patterns/type_ignore_cast.md)** -- Type-safety escape hatches that suppress real errors.

- **[Missing type annotations](patterns/missing_type_annotations.md)** -- Untyped parameters, especially non-obvious types like callbacks or API objects.
  ```python
  def _open_window(title: str, diff_bytes: bytes, api: AppApi, prefs_loader):
  ```

- **[Bare `dict` at serialization boundaries](patterns/bare_dict_at_boundary.md)** -- Function returns `dict` where data crosses systems (Python→JS, module→module). Use `TypedDict` to make the contract explicit.
  ```python
  def get_prefs(self) -> dict:    # shape is implicit
      return {"font_size": ...}
  ```

- **[Catching overbroad exception types](patterns/catch_specific_exceptions.md)** -- `except Exception` in a targeted recovery path, or tuple-excepts lumping unrelated failures, both hide real bugs. Define a named exception for each expected-recoverable condition.
  ```python
  try: self._client.list_windows()
  except Exception: pass   # server not ready? or a real bug? -> GvcGuiNotDoneStarting
  ```

### Uncategorized (elaborated)

- (none remaining)

### Loose patterns (not yet elaborated)

- **Speculative generality / unnecessary indirection:** Abstraction layers with only one concrete usage and no tests. Examples: callback parameters always passed the same callable. Inline the value and remove the indirection.
    - Category: "Concision"
- **Underscore-prefixed module names in applications:** `_foo.py` in applications where there is no public API to distinguish from. Rename to `foo.py`.
    - Category: "Good Names"
- **Verbose construction where simpler equivalent exists:** `{f for f in x}` → `set(x)`, `[x for x in items]` → `list(items)`, `{k: v for k, v in d.items()}` → `dict(d)`. Use the built-in constructor when the comprehension adds no filtering or transformation.
    - Category: "Concision"
- **Redundant safety mechanisms:** When two mechanisms provide the same guarantee, remove the more complex one. Example: `fcntl.flock` + `os.replace` -- the atomic replace already ensures last-writer-wins, making the lock unnecessary.
    - Category: "Concision"
- **`sys.exit(main())` wrapper when `main` returns `None`:** `sys.exit(None)` is equivalent to exiting normally, so `sys.exit(main())` adds no behavior beyond calling `main()`. The wrapper only earns its keep when `main` returns an exit code. Drop it (and the `sys` import if it becomes unused) when `main` is `-> None`.
    - Special case. Consider deletion of pattern.

### Loose categories

Categories that may emerge as more patterns are discovered:

- **Define errors out of existence** -- similar to Type Design
- **Cohesion**

---

## How to learn new patterns from a commit

Invoked when the user says something like "Learn any new revise patterns from commit `<SHA>`" or "Review the revisions I made to AI-drafted code in commit `<SHA>`".

The user has manually revised code that an AI previously drafted, and captured those revisions in a single commit. The goal is to extract *generalizable* patterns from that commit and codify them into this skill, so future `/revise` runs catch the same issues automatically.

### Procedure

1. **Examine the commit's diff.**
   - Run `git show -U10 -w <SHA>`.
     - `-U10` gives enough context to see what's going on without being too noisy.
     - `-w` ignores whitespace changes, which makes indentation-only revisions far less noisy.
     - `show` (vs. `diff`) includes the commit *message*, which often summarizes the user's intent.
   - If `<SHA>` is omitted, default to `HEAD`.

2. **Interview the user.** Use the `AskUserQuestion` tool. Ask about:
   - **Why specific revisions were made**, when the rationale isn't obvious from the diff alone. Surface-level changes often hide a deeper principle worth codifying.
   - **Why expected revisions were NOT made.** Scan the pre-revision code for patterns already in this skill -- if a pattern *would* have triggered but the user left the code alone, that's a boundary condition worth capturing. Don't only learn from changes; learn from the user's deliberate non-changes too.
   - **Where a revision fits** among existing patterns -- is it a new pattern, a refinement of an existing one, or an inverse/complementary case?

   Keep questions focused and batched. Prefer a few well-chosen questions over an exhaustive interrogation.

3. **Draft an update to the skill.** For each generalizable pattern learned:
   - If the pattern is genuinely new: add a bullet under the appropriate category in SKILL.md, and create a detail file in `patterns/<name>.md`.
   - If the pattern refines or qualifies an existing one: update the existing detail file (add a "When NOT to revise" entry, an inverse case, or a clarifying example) rather than creating a new one.
   - Follow the structure of existing pattern files: **Quick trigger**, **Why revise**, **When NOT to revise**, **Fix**, **Example (Before/After)**.
   - Capture the user's *rationale* in the pattern file, not just the mechanical rule. The "why" lets future-me judge edge cases.

4. **Reevaluate skill organization.** After drafting updates, check whether the skill still reads well at a high level. Reorganize if needed -- see [organization-guidelines.md](organization-guidelines.md) for the principles (7-10 items per category, progressive disclosure, categories close to recognizable situations).

### What counts as a generalizable pattern

- **Generalizable:** applies across files, languages, or projects -- e.g., "docstrings should state API guarantees, not implementation details."
- **Not generalizable:** one-off cleanups tied to this specific codebase's history, tooling, or content -- e.g., "switched the build system from Hatch to Poetry," "moved the paused PyInstaller packaging files to a side branch," or "added a screenshot to README." These are project decisions, not code-quality patterns. They belong in the commit message, not the skill.

When in doubt, ask: *would this same revision make sense in a different codebase written by a different team?* If yes, it's a pattern.

---

*See [organization-guidelines.md](organization-guidelines.md) for the principles governing how this skill is structured.*

---
> Source: [davidfstr/revise-skill](https://github.com/davidfstr/revise-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
