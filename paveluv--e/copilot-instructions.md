## e

> Always include a `Co-Authored-By` trailer for the AI model contributing to a commit.

# Repository rules

## Commit attribution

Always include a `Co-Authored-By` trailer for the AI model contributing to a commit.
Use the exact runtime model identifier from the active session, including its version or
snapshot suffix when exposed; never substitute a generic name such as Codex or GPT, infer
the model from the configured default, or invent a version. 

## Formatting

Before every commit, run `tools/scheme-format.sps -i` on all `*.sls`,
`*.ss`, `*.sps`, and `*.e` files, plus the extensionless loader `e`.

## Scripting

Temporary scripts (file surgery, probes, one-off drivers) are written in
Scheme, not Python or other languages including unix shell. Committed test
fixtures may shell out where a fixture genuinely needs another runtime,
but reach for Scheme first.

## Naming conventions

Every library but the command layer `edit` is imported with its own
prefix -- seams and apps alike (`store:`, `terminal:`, `git:`,
`sys:`) -- and that is how M-x sees them; only `edit`'s names are
bare, and apps import it as `(except (edit) init!)`.  Modules are
named in the singular (`style`, `file`, `mode`, `string`, `actor`,
`doc`), and exported names never repeat the module's stem:
`style:set!`, `log:add!`, `git:branches`, `terminal:send!` -- never
`styles:set-style!` or `git:git-branches`.  Rename in the export list
(`(rename (internal external))`) if the definition keeps a longer
name; a command that was the bare stem gets a verb (`terminal:open!!`,
`eval:run!`, `describe:show!`). Commands that interact with the user
(can block on input from the user) have double-bang suffix "!!". Commands
without a bang, or with a single bang "!" are supposed to finish without
the user's invervention.

## Docs 

`manual/` is the user's manual (nothing else goes there); 
development design docs, progress docs and other dev notes live in `dev/`.

---
> Source: [paveluv/e](https://github.com/paveluv/e) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
