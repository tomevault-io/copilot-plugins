## cursor-rules

> 1. do not use .get if we know the key should be there. let it fail if a key/value combo is incorrectly not there


1. do not use .get if we know the key should be there. let it fail if a key/value combo is incorrectly not there
- example of bad code:
```py
CONFIG.get('supplier_normalization', {}).get('replacements', {})
```
- reasoning: hard to read, causes issues when we accidentally are missing an important value and are now getting incorrect default values

2. do not create fake data under any circumstances. if for some reason yyou think it would be beneficial, you need to ask me before coding it.
- this is dangerous
- example: you can't connect to an API so you hard-code fake data, or posts/tweets, etc.

3. do not use logic like this for a python script to add the path to src. 
- example:
```py
sys.path.append(str(Path(__file__).parent.parent / 'src'))
```
- better: using os, finding the root of the dir (use config if you can, otherwise find a .gitignore file), 
- reasoning: if you are to drag/drop into a notebook, this causes an error

4. do no check for existence of an item that should be there
- better: check it exists, if not, make code fail

```py
# exmaple 1
if df and df['important_column_name'] and df in globals():
    ...

# example 2
if 'kelly_info' in reasoning and reasoning['kelly_info']:
    ...
```
- reasoning:
    - super hard to read, tons of indenting that is not necessary
    - it is okay to write code that breaks, doing checks such as if a necessary column exists would just be prolonging the failure of the script later on 

5. do NOT search for columns that might work, read the file in, print column names and values, and figure it out for yourself (or ask me!)

```py
# bad
possible_names = ["store_number", "store_nbr", "StoreNumber", "storeNum", "StoNum"]
store_col = next((g for g in possible_names if g in reader.fieldnames), None)

if not store_col:
    print("Couldn't find a store column — maybe it's named something else?")
else:
    print(f"Found store column: {store_col}")
    for row in rows[:5]:
        print(row[store_col])

```

6. when creating Python files from my direction, put the context into the docstring so it is easy for both me AND you to remember what i've asked for
- bad: python file with no docstring + separate .md file
- good: all in 1 python file

7. if you are going to use emojis, have an emoji map in the config or something similar
- good:

```py

# =============================================================================
# EMOJI MAP
# =============================================================================

EMOJI = {
    # Status
    'success': '✅',
    'error': '❌',
    'warning': '⚠️',
    'info': 'ℹ️',
    'refresh': '🔄',
    'save': '💾',
    
    # Analysis
    'chart': '📊',
    'target': '🎯',
    'calendar': '📅',
    'star': '⭐',
    'money': '💰',
    'up': '📈',
    'down': '📉',
    
    # Luck categories
    'lucky': '🍀',
    'unlucky': '💔',
    'neutral': '😐',
    
    # Sports
    'nba': '🏀',
    'nfl': '🏈',
    
    # Test/Debug
    'test': '🧪',
}

```

- bad: hard-coding

8. do not do a git commit / push until you know we have fixed the issue / iterated on the task successfully

9. if you are going to do git commits, ask me for permission first, and use the methodology from this link: 
- Jiawei's article on submitting Git Patches / commit messages
- link: https://git.kernel.org/pub/scm/git/git.git/tree/Documentation/SubmittingPatches

advice on how to make the message:
- Keep it ~50 chars, no period at the end.
- Start with a scope (cli, docs, parser, build, a file/dir).
- Use imperative mood: “add”, “fix”, “refactor”, “revert”, “docs”.
- Say what changed and hint at why/impact (crash avoided, speed up, UX).
- Avoid fluff: no “This commit…”, no emojis, no trailing punctuation.
- Prefer specific nouns over vague ones (“nil ptr on empty input” > “issue”).
- If reverting, start with revert: …. If following up, consider follow-up:.
- Reference an issue only if your team’s convention requires it (short form).
- If it’s a sweeping change, split commits instead; one logical change per commit.

Examples you can copy and tweak:
- git commit --message "parser: fix nil ptr on empty input"
- git commit --message "cli: add --dry-run flag for safety"
- git commit --message "docs: clarify DCO vs PGP signing"
- git commit --message "build: cache deps to cut CI time"

10. don't stage large data files that won't make it past the github file size limit of 50mb (hard limit 100mb)
- stop me and lmk that i need to .gitignore or do something else

11. if i ask for a git commit/add series, provide the proposed commands with clear commit messages
- show the `git add` command(s) for specific files
- show each `git commit -m` command following the Jiawei methodology (rule #9)
- explain what each commit adds/changes and why it's grouped that way
- wait for my approval before executing any git commands
- example format:
```bash
git add scripts/kalshi_order_book_tracker.py
git commit -m "kalshi: add health check tracking with CLI flag"
```

12. organize helper functions in the order they are used (execution flow order)
- helper functions should appear in the same order they're called in the main function
- if a helper function uses another helper, put the dependency first
- reasoning: improves readability - readers can follow the code flow top-to-bottom
- example (good order):

```py
# Main function calls: get_yesterday_et() → clone_repo() → send_sns()
# clone_repo() internally uses run_cmd()

def get_yesterday_et():
    """First thing called in main."""
    ...

def run_cmd(cmd, cwd=None):
    """Helper used by clone_repo."""
    ...

def clone_repo(token):
    """Uses run_cmd, called after get_yesterday_et."""
    result = run_cmd(['git', 'clone', ...])
    ...

def send_sns(subject, message):
    """Called at end for notifications."""
    ...

def main():
    yesterday = get_yesterday_et()
    repo = clone_repo(token)
    send_sns("Done", "Success")
```

- bad: functions organized alphabetically or randomly - forces reader to jump around

13. if giving me bash/code that does not materially change data (ie., it is safe for you to test before giving to me) - please test it so i don't need to debug small errors

14. in notebooks, use `display(df)` for pandas DataFrames instead of `print(df.to_string(...))`
- prefer readable notebook-native table output when showing DataFrames
- use print for scalar/log lines, not for rendering DataFrame tables

15. when printing or displaying numeric summaries (notebooks, logs, small tables), **round floats** to a fixed precision (e.g. `.round(3)` / `.round(4)` on DataFrames, or `f"{x:.4f}"` for scalars) so values don’t spam meaningless digits and stay easy to scan
- bad: raw `0.04188599999999999` or 12-digit floats in `display(df)` / `print` summaries
- good: `display(df.round(4))` for exploration tables; keep full precision only in **stored artifacts** (parquet, CSV exports) or **downstream calcs** where rounding would change decisions
- reasoning: readability and diff-friendliness; rounding for display should not silently replace exact math inside core pricing / settlement logic unless you intend to

16. use readable variable names and types when needed ie. df_abcdef or lst_abcdef

---
> Source: [MylesThomas/betting](https://github.com/MylesThomas/betting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
