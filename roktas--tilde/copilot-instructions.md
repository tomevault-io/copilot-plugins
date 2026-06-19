## tilde

> Use for Tilde deployment, provisioning, and home-management work. Trigger for Tilde prompt markers such as `/tilde ...` or `$tilde ...`, `~ ...`, messages whose first word is `tilde`, or requests to create, initialize, deploy, update, diagnose managed state, adopt app/config files, customize data-layer policy, or work with Tilde home behavior.


# Tilde

Tilde is a standalone agent skill and control plane. The canonical contract is
[Specification](references/specification.md). Read that file before changing behavior or executing nontrivial Tilde
work. The `.agents/specs/tilde.md` path is only a symlink to the same canonical file.

The provisioning model is:

- Desired state comes from the current committed public/private home repositories.
- The only persistent Tilde state file is `~/.local/state/tilde/state.yml`.
- `state.yml` stores repository bindings and the last fully converged commit anchors.
- Planning evaluates current manifests and targeted live facts on every run.
- Every run compares current desired manifests with targeted live checks and idempotent actions.
- Failed, conflicted, or deferred runs do not advance `applied` anchors.
- The host-local lock is process coordination, not cleanup state.

When existing docs, code, habits, or memory conflict with `references/specification.md`, the specification wins.

## Agent Quickstart

When the user invokes a Tilde prompt such as `/tilde <command> [target] [qualifiers...]`:

1. Treat `/tilde`, `$tilde`, and similar Tilde prompt markers as agent prompt contracts. They are not shell commands.
2. Resolve the controller-side runtime entrypoint before shell execution. Use the loaded skill directory's `bin/tilde`;
   if that path is not available, use `~/.agents/skills/tilde/bin/tilde`. Do not rely on bare `tilde` being on `PATH`.
3. Identify the target: current host, the current host name, `host`, `ssh:host`, or an all-caps host group such as
   `ALL`, `HOME`, or `WORK`.
4. Classify the command from the Command Reference below.
5. Map agent-orchestrated commands to the Workflow Matrix before running helpers.
6. For remote targets, run the Remote Freshness Preflight before reading target state, generating plans, or applying
   results.
7. Use the resolved runtime entrypoint for script delivery. Generate plans, run live checks, and apply results on the
   target host, not on the controller.
8. Keep proposal-first behavior for destructive, preference-sensitive, privilege-requiring, or remote mutations.
9. After a successful mutating remote apply, run a cheap final status read on the target before closeout.
10. After the run, summarize successful, changed, deferred, conflicted, and failed work.

If a direct runtime call says a command is agent-orchestrated, stop trying shell variants of that command. Load this
skill, classify the command, and run the appropriate agent workflow.

Do not execute `/tilde ...` or `$tilde ...` in a terminal. `/tilde` is a prompt marker, and `$tilde` may be a Codex skill
trigger or shell variable expansion depending on the environment. Shell execution uses the resolved runtime entrypoint,
written as `"$TILDE"` in examples:

```bash
TILDE=${TILDE:-"$HOME/.agents/skills/tilde/bin/tilde"}
"$TILDE" help
"$TILDE" ssh spinoza
```

If the skill was loaded from another directory, set `TILDE` to that directory's `bin/tilde` instead. If a controller-side
`tilde` command is not found, do not search the filesystem for it; switch to the resolved runtime entrypoint.

For remote targets, the first target-state read must happen on the target through Tilde SSH transport. Do not
inspect the controller's `~/.local/state/tilde/state.yml` to discover a remote host's repository bindings, applied
anchors, level, platform, or bootstrap state. That file belongs only to the controller host.

## Target Grammar

Host-aware prompt commands accept these target forms:

- no target: current host, also called localhost.
- `host`: remote host shorthand for `ssh:host`, except when it names the current host.
- `ssh:host`: explicit remote host target.
- `GROUP`: a bare all-caps host group defined by the active home policy, such as `ALL`, `HOME`, or `WORK`.

If a bare host token equals the current host name, such as `hostname -s`, treat it as the current host and run the local
workflow. Use explicit `ssh:host` only when the user really wants SSH transport to that host, including self-SSH.

The host-aware prompt commands are `deploy`, `update`, `repair`, `upgrade`, `align`, `status`, and `doctor`.
Commands with their own subject syntax, such as `adopt APP_OR_PATH`, `clean SUBJECT`, `organize SUBJECT`, `create`, and
`init`, do not treat a bare argument as a host unless the command's own policy says so.

Bare all-caps targets are not hostnames. Expand them from the active `~/AGENTS.md` home policy before running any remote
work. Group expansion applies only to unprefixed target tokens; `ssh:host` is always an explicit host target. If the
policy does not define the requested group, ask the user for the host list. For explicitly requested configured groups,
report the expanded host list and continue; the prompt itself is consent to run the requested workflow. Ask only when
the group is undefined, ambiguous, unexpectedly expands outside active policy, or a later step requires separate
explicit confirmation. Run each host as a separate target workflow and report per-host results.

When traversing a host group, perform a bounded noninteractive reachability check before each host workflow. Skip
unreachable hosts and continue with the remaining hosts. Report skipped hosts separately; an unreachable host inside a
group is not a failure for reachable hosts. If every expanded host is unreachable, stop with a clear `deferred` result.
Use Tilde SSH transport for the reachability check itself. Do not use raw `ssh`, even for this cheap probe.

```bash
TILDE=${TILDE:-"$HOME/.agents/skills/tilde/bin/tilde"}
host=spinoza
"$TILDE" ssh -o BatchMode=yes -o ConnectTimeout=5 "$host" << 'SCRIPT'
printf 'ok\n'
SCRIPT
```

## Runtime

The PATH-visible runtime surface is `bin/tilde`, `bin/sudo`, and helper commands intentionally exposed in the Tilde
runtime PATH. Normal command implementations live in `libexec/` and are dispatched through `bin/tilde`.

Controller-side runtime calls must go through the resolved entrypoint. Bare `tilde` is valid only when the Tilde runtime
PATH is already active, such as inside a remote script delivered by Tilde SSH transport.

`bin/tilde` has direct runtime routes for helper and diagnostic commands. It intentionally refuses agent-orchestrated
prompt commands; those commands are interpreted and orchestrated by the loaded Tilde skill.

Implementation routes such as `plan` and `apply` are stable primitives for agent workflows. They are not user-facing
prompt commands. Use them only inside the workflow patterns in this skill and the specification.

## Command Reference

Commands are grouped by execution model.

### Agent-Orchestrated Prompt Commands

The agent interprets these high-level commands and orchestrates the workflow. They have no direct runtime route.

- `deploy`: prepare a local or remote host and apply desired state.
- `update`: run explicit update behavior from the current desired state.
- `repair`: apply the current desired state again; it does not read a repair queue or module-level state.
- `upgrade`: run the widest explicitly requested upgrade path.
- `adopt`: inspect a requested app, config, package, or path and propose public/private repository placement.
- `create`, `init`, `clean`, and `organize`: follow the specification and user policy; keep destructive or
  preference-sensitive work proposal-first.

Treat `dry-run` and `plan-only` as qualifiers. Do not invent a separate stateful planning model for them.

### Direct Runtime Commands

These commands have direct `bin/tilde` runtime routes. For remote targets, deliver the command through
Tilde SSH transport so the target host supplies live facts.

- `help`: show public commands or one command's usage.
- `doctor`: diagnose state, repository, target, and managedness problems without executing module code or mutating
  targets.
- `handoff`: copy and print the privilege handoff command for the local or remote host.
- `status`: show compact repository bindings and last fully converged anchors.

### Dual Command

- `align`: local link, copy, seed, and reset reconciliation can run directly through the resolved runtime entrypoint.
  Remote align remains agent-orchestrated: use `/tilde align spinoza` prompt semantics and deliver the target-side
  workflow through Tilde SSH transport. Do not run `tilde align --format json` on a remote target; `align` has no
  `--format` option.

### Implementation Routes

- `ssh`: deliver scripts to a remote host with the Tilde runtime environment active.
- `plan`: produce a plan from current committed repository content and target-host facts.
- `apply`: apply one or more plan files.
- `sudo`: classify privilege needs and support handoff.
- `boot`, `checkout`, `preflight`, and `smoke`: specialized runtime helpers.

Do not present implementation routes as user-facing Tilde prompt commands. When using them on the controller, use the
resolved runtime entrypoint with undotted route names such as `"$TILDE" plan` and `"$TILDE" apply`.

## Workflow Matrix

Agent commands map to planning modes and module sections as follows.

| Prompt command | Plan mode | Module behavior |
| --- | --- | --- |
| `deploy` | `apply` | `Prerequisites`, `Install`, `Post Install` when `Install` changed, declared files and links, then `Configure` |
| `update` | `refresh`; `repair` for state recovery | `Prerequisites`, `Update`, then `Configure`; state recovery uses repair behavior |
| `repair` | `repair` | `Prerequisites`, `Install`, `Post Install` when `Install` changed, declared files and links, then `Configure` |
| `upgrade` | `upgrade` | broad refresh behavior, then package upgrades |
| `align` | `align` | directories, links, copies, seeds, and resets only; no bootstrap, packages, or module code |

`create`, `init`, `clean`, `organize`, and `adopt` are proposal-first operator workflows unless a direct helper is
explicitly documented for the requested step.

The prompt command is `update`; the ordinary plan mode is `refresh`. Do not call or invent `--mode update`. If target
status shows missing `state.yml` or no `applied` anchors, recover state with `plan --mode repair` for that run; do not
use refresh merely to recreate state, and do not tell the user to run deploy solely to initialize state. Missing
`state.yml` is not proof that the host was never deployed; describe it as missing state and state recovery unless
bounded evidence proves otherwise.

When an agent workflow materializes plan JSON files, create them under a per-run temporary directory and remove that
directory at exit. Do not write fixed plan paths such as `/tmp/opencode/HOST-public.json`, and do not leave plan files
behind after `tilde apply`:

```bash
tmpdir=$(mktemp -d)
trap 'rm -rf "$tmpdir"' EXIT

"$TILDE" plan --mode refresh --repo ~/Dropbox/home --host newton --format json > "$tmpdir/public.json"
"$TILDE" apply --plan "$tmpdir/public.json"
```

`refresh` mode does not advance `applied` anchors. After a successful `/tilde update`, final status may show target
repository HEAD ahead of the stored applied anchor; report that as expected refresh behavior, not as anchor advancement
and not as failed convergence.

## Remote Script Execution

### Remote Freshness Preflight

Remote Freshness Preflight is the first remote workflow step. Before any remote `status`, `plan`, `apply`, or
state-recovery decision, confirm that the target runtime is current. The controller-side loaded skill is the source for
the expected Tilde runtime commit.

For mutating remote prompt workflows such as `deploy`, `update`, `repair`, `upgrade`, and remote `align`:

- Compare the controller runtime commit with the target `~/.agents/skills/tilde` commit before the first target-state
  read.
- If the target runtime is a stale Git checkout, refresh it from the controller checkout with `"$TILDE" checkout remote`
  before running target `tilde status`, `tilde plan`, or `tilde apply`.
- If the target runtime is stale but cannot be refreshed safely because it is dirty, missing, not a Git checkout, or
  otherwise ambiguous, stop with `deferred`. Do not continue into state recovery with the stale runtime.
- If a clean target runtime cannot fast-forward because its history differs from the controller runtime, refresh it with
  `--replace-clean`. Do not use `--replace-clean` for public/private desired-state checkouts unless the operator has
  explicitly approved replacing that clean checkout.
- Do not patch, edit, or `sed` the target runtime checkout as part of the deploy/update/repair workflow or continue as
  if a target-side patch completed the workflow. If live debugging requires a temporary target edit, treat it as a
  separate explicitly approved recovery/debug operation, capture the diff, port accepted changes to the controller
  source repository, then refresh the target through checkout freshness routes before retrying.
- On Git-backed remote hosts, also refresh the target public/private desired-state checkouts from the controller
  repositories before planning. If a target checkout is dirty or cannot be fast-forwarded from the controller bundle,
  stop with `deferred`.
- On Dropbox-backed remote hosts, do not replace synced repositories with controller bundles; report stale or unsynced
  target checkouts as `deferred` unless the user asks for a Git-backed checkout refresh.

When using `checkout remote`, always pass the complete source and target binding. For target runtime freshness, the
controller repository is the loaded skill root and the target is `~/.agents/skills/tilde`:

```bash
TILDE=${TILDE:-"$HOME/.agents/skills/tilde/bin/tilde"}
controller_runtime=${TILDE%/bin/tilde}
"$TILDE" checkout remote --host spinoza --repo "$controller_runtime" --target '~/.agents/skills/tilde'
```

For Git-backed desired-state checkout freshness, pass the selected controller data repository and the target checkout:

```bash
"$TILDE" checkout remote --host vps --repo ~/Dropbox/home --target '~/.local/src/home'
```

Do not call `checkout remote` with only `--host`; the route cannot infer which controller repository maps to which
target checkout. Quote target paths that start with `~` so the controller shell does not expand them to the controller
home before the remote receives the path.

For read-only remote workflows such as `status` and `doctor`, detecting a stale target runtime is a reportable stale
runtime condition, not permission to mutate the remote host. Report it and stop unless the user requested repair or
update behavior.

A status output that mentions state paths outside this specification, such as `~/.local/state/tilde/config.yml` or
`~/.local/state/tilde/hosts/HOST/state.md` means the target runtime is stale. Stop and refresh or report the stale
runtime before any `plan --mode repair` step. Do not describe state writes outside the `state.yml` model as successful
state recovery.

Use the resolved runtime entrypoint for all remote script execution:

```bash
TILDE=${TILDE:-"$HOME/.agents/skills/tilde/bin/tilde"}
"$TILDE" ssh spinoza
```

It sets up the correct Tilde runtime environment (locale, non-secret `~/.config/environment.d/*.conf` values, PATH,
TILDE_ROOT, TILDE_SUDO) and delivers the script body through `ssh host sh -s --`.

Do not use raw `ssh`, `sh -c`, or `bash -lc` for multi-command remote work.
These bypass Tilde's PATH setup and sudo interceptor.

After the Remote Freshness Preflight, plan and execute on the target host. Platform detection, package inventory,
repository bindings, and live checks come from the target. A controller-side plan for a remote host is invalid.
After the target runtime is known current, read target status in JSON when a workflow needs repository bindings:

```bash
TILDE=${TILDE:-"$HOME/.agents/skills/tilde/bin/tilde"}
"$TILDE" ssh spinoza << 'SCRIPT'
tilde status --format json
SCRIPT
```

Use the returned `state.public` and `state.private` paths exactly when generating target-local plans. Do not hard-code
`~/Dropbox/home`, `~/Dropbox/home-`, `~/.local/src/home`, or `~/.local/src/home-` unless those paths came from target
status, explicit user arguments, or active home policy for a stale Git-backed target that must be refreshed before
status can be trusted.

For Git-backed targets whose status binds repositories outside Dropbox, controller-side `~/Dropbox/...` source paths are
not target paths. Do not create target-side `~/Dropbox` for repository binding, cleanup, shared app state, or convenience
paths unless active target policy explicitly says that host has Dropbox-backed storage.

When a remote workflow needs repository bindings in a target-side script, bind them from target status JSON before
planning. Do not retype host-convention paths in `tilde plan --repo ...` commands:

```bash
TILDE=${TILDE:-"$HOME/.agents/skills/tilde/bin/tilde"}
"$TILDE" ssh spinoza << 'SCRIPT'
tmpdir=$(mktemp -d)
trap 'rm -rf "$tmpdir"' EXIT
status_json=$tmpdir/status.json

tilde status --format json > "$status_json"
public=$(ruby -rjson -e 'data = JSON.parse(File.read(ARGV[0])); print data.dig("state", "public").to_s' "$status_json")
private=$(ruby -rjson -e 'data = JSON.parse(File.read(ARGV[0])); print data.dig("state", "private").to_s' "$status_json")
if [ -z "$public" ]; then
  printf '%s\n' 'STATUS_FAILED: missing state.public' >&2
  exit 1
fi
SCRIPT
```

Remote state is target-local. For `/tilde update spinoza`, `/tilde update ssh:spinoza`, `/tilde deploy spinoza`,
`/tilde deploy ssh:spinoza`, `/tilde doctor spinoza`, `/tilde status spinoza`, and `/tilde align spinoza`, do not read the controller's
`~/.local/state/tilde/state.yml`. If state or repository bindings are needed, read them on the target:

```bash
TILDE=${TILDE:-"$HOME/.agents/skills/tilde/bin/tilde"}
"$TILDE" ssh spinoza << 'SCRIPT'
tilde status --format markdown
SCRIPT
```

```bash
TILDE=${TILDE:-"$HOME/.agents/skills/tilde/bin/tilde"}
"$TILDE" ssh spinoza << 'SCRIPT'
tmpdir=$(mktemp -d)
trap 'rm -rf "$tmpdir"' EXIT
status_json=$tmpdir/status.json
mode=refresh
# Use mode=repair when target status reports missing state.yml or no applied anchors.

tilde status --format json > "$status_json"
public=$(ruby -rjson -e 'data = JSON.parse(File.read(ARGV[0])); print data.dig("state", "public").to_s' "$status_json")
private=$(ruby -rjson -e 'data = JSON.parse(File.read(ARGV[0])); print data.dig("state", "private").to_s' "$status_json")
if [ -z "$public" ]; then
  printf '%s\n' 'STATUS_FAILED: missing state.public' >&2
  exit 1
fi

tilde plan --mode "$mode" --repo "$public" --host spinoza --format json > "$tmpdir/public.json" || exit $?
if [ -n "$private" ]; then
  tilde plan --mode "$mode" --repo "$private" --host spinoza --format json > "$tmpdir/private.json" || exit $?
  tilde apply --plan "$tmpdir/public.json" --plan "$tmpdir/private.json"
else
  tilde apply --plan "$tmpdir/public.json"
fi
SCRIPT
```

Replace repository paths with the target host's configured bindings. Omit the private plan when the target has no
private repository.

For remote align, use the same target-local plan/apply shape with `mode=align`. Do not call the target's direct
`tilde align` route for a remote prompt workflow, and do not suppress diagnostics:

```bash
TILDE=${TILDE:-"$HOME/.agents/skills/tilde/bin/tilde"}
"$TILDE" ssh spinoza << 'SCRIPT'
tmpdir=$(mktemp -d)
trap 'rm -rf "$tmpdir"' EXIT
status_json=$tmpdir/status.json
mode=align

tilde status --format json > "$status_json"
public=$(ruby -rjson -e 'data = JSON.parse(File.read(ARGV[0])); print data.dig("state", "public").to_s' "$status_json")
private=$(ruby -rjson -e 'data = JSON.parse(File.read(ARGV[0])); print data.dig("state", "private").to_s' "$status_json")
if [ -z "$public" ]; then
  printf '%s\n' 'STATUS_FAILED: missing state.public' >&2
  exit 1
fi

tilde plan --mode "$mode" --repo "$public" --host spinoza --format json > "$tmpdir/public.json" || exit $?
if [ -n "$private" ]; then
  tilde plan --mode "$mode" --repo "$private" --host spinoza --format json > "$tmpdir/private.json" || exit $?
  tilde apply --plan "$tmpdir/public.json" --plan "$tmpdir/private.json"
else
  tilde apply --plan "$tmpdir/public.json"
fi
SCRIPT
```

Every remote Tilde step must treat a non-zero exit as failed, deferred, or conflicted, even when stdout is empty. Do not
redirect stderr to `/dev/null` for remote `status`, `doctor`, `checkout`, `plan`, `apply`, `align`, or verification
work. A later successful status read does not turn a failed plan, apply, checkout, or align step into success.

After `tilde apply`, inspect the JSON action results. `completed: true` only means the action list was processed; it is
not full success when any result is `deferred`, `conflict`, or `notok`. Treat such a run as incomplete and report the
exact action status and reason.

After a successful mutating remote apply, verify target convergence with a separate cheap status read so the apply JSON
stays easy to parse:

```bash
TILDE=${TILDE:-"$HOME/.agents/skills/tilde/bin/tilde"}
"$TILDE" ssh spinoza << 'SCRIPT'
tilde status --format markdown
SCRIPT
```

Use `target HEAD` or `target current commit` for the repository commit currently checked out on the remote host. Use
`applied anchor` for the commit recorded under `applied` in the target's `state.yml`. Do not call a remote repository
HEAD `local` in summaries; that word is ambiguous from the controller.

Inside the remote script body, bare `tilde` is valid because Tilde SSH transport places the target runtime directory at
the front of `PATH`.

Remote scripts must be `sh`-compatible by default. Use the plain `"$TILDE" ssh HOST << 'SCRIPT'` form for ordinary
status, plan, apply, and verification work. Do not pass `-- bash` for macOS targets or for scripts that can run under
`sh`. Passing an interpreter after `--` is reserved for a verified non-macOS target and a script that truly needs syntax
absent from `sh`.

In the default remote form, `sh`-compatible means POSIX `sh`, not "Bash without a Bash shebang." Do not use the Bash
skill prelude in these remote scripts. Avoid common Bashisms:

| Bashism | POSIX `sh` form |
| --- | --- |
| `set -o pipefail` | avoid critical pipelines; check each command separately |
| `[[ -n $x ]]` | `[ -n "$x" ]` |
| `[[ $x =~ re ]]` | `case $x in pattern) ... ;; esac` |
| `arr=(a b)` and `${arr[@]}` | newline-separated strings, positional parameters, or repeated commands |
| `source file` | `. file` |
| `function name { ...; }` | `name() { ...; }` |
| `local x=...`, `declare`, `mapfile`, `readarray`, `select` | simple variables and explicit loops |
| `< <(cmd)` and `<<< "$text"` | temp files, pipes, or here-documents |
| `{foo,bar}` brace expansion | spell out the words |
| `$'...\n...'` strings | `printf '%s\n' ...` |

Also remember that `dash` is a common `/bin/sh`; scripts that only work in Bash are not acceptable in the plain remote
form.

Use `"$TILDE" ssh --tty spinoza` for interactive remote sessions that require a pseudo-terminal (e.g., `sudo` password
prompts).

## Module Model

Module README code sections use this stateless contract:

- `Prerequisites`: external requirements and read-only checks only. Failed checks return `deferred`.
- `Install`: idempotently ensure packages, tools, directories, applications, repositories, or local resources are present.
- `Post Install`: runs only when `Install` changed something in the current run; correctness must not depend on it.
- `Configure`: idempotent desired configuration. In apply and repair modes, it runs after declared directories, links,
  copies, seeds, and resets are materialized.
- `Update`: explicit refresh or upgrade work.
- `Notes`: informational only; never executed.

Code blocks must be idempotent or guarded. If an effect cannot be detected from live target facts, represent it as a
prerequisite, note, or explicit proposal-first operator action.

Package managers are the source of truth for package presence. Package inventories are ephemeral, in-memory
optimizations only.

Module code blocks call Tilde helpers by their plain names, such as `line` and `span`. Privileged helper calls use the
same plain form with `sudo`, such as `sudo line ...`.

README frontmatter may declare `directories` along with `links`, `copies`, `seeds`, `resets`, and `packages`. Link
sources are repository-relative by default. A source that starts with `~/` or `/` is a target-home source and must stay
inside the target home. Use target-home sources for live home paths such as Dropbox-backed shared state directories and
XDG entrypoints; Dropbox-to-Dropbox symlink values must be relative to the target directory.

## Safety

Proposal-first behavior is required before replacing unmanaged files, removing stale links or managed spans, backing up
conflicting paths, uninstalling packages, running non-idempotent code, requiring privileges, mutating remote hosts, or
performing one-off operator workflows.

Managedness must be proven before destructive cleanup. No proof means no destructive mutation.

When apply reports `conflict`, stop after reporting the exact conflicted targets and wait for an explicit user response
before replacing files, forcing links, copying repository versions over targets, or rerunning apply. Do not infer consent
from silence, from the fact that a conflict exists, or from the agent's own question. Conflict resolution may reveal
additional conflicts in a later apply; repeat the same explicit-approval step for each newly revealed conflict.

If the user chooses repository replacement for a conflict, back up the exact target when practical, remove only the exact
conflicting target, and rerun apply so the normal declarative action recreates it. Do not rerun apply expecting a
default conflict result to overwrite the target. Broad cleanup, such as clearing an entire font directory after
package-manager file conflicts, requires a separate explicit approval that names the directory and effect.

If an apply command times out or reports `state lock busy`, do not delete the lock file as a first response. A lock held
with `flock` belongs to a process, and removing the pathname does not prove the process stopped. Treat this as
`deferred` or an active-run check: use bounded read-only process inspection on the target, wait when a Tilde/apply
process is still active, and remove a leftover lock only after proving no process holds it and getting explicit approval.

If sudo is required and noninteractive sudo fails, report `deferred` and present:

```bash
"$TILDE" handoff --host spinoza --copy
```

Run the helper on the controller, not inside `"$TILDE" ssh`. The helper prints one shell-neutral command line and
reports whether it copied the command to the controller clipboard. The command must work when pasted into bash, zsh, or
fish. Copy or present the `Handoff command:` line exactly; do not rewrite it, split it into a `TILDE=...` assignment
plus an `ssh` command, replace the remote path with a local variable, or add extra quoting. The printed command is also
valid during first-time bootstrap before the target runtime exists; do not replace it with an ad hoc sudoers one-liner.
Wait for the user to run it, then rerun the affected action and verify cleanup. Do not collect passwords in chat.

## Weak-Model Guardrails

For weaker or low-context agents:

- Plan from current committed repositories and targeted live facts.
- Treat `Prerequisites` as read-only checks.
- Advance `applied` anchors only after every required action succeeds.
- Keep `applied` anchors unchanged after `deferred`, `conflict`, or `notok`.
- On timeout or `state lock busy`, do not remove `~/.local/state/tilde/lock` blindly. Check whether an apply process is
  still active and treat uncertainty as `deferred`.
- Do not remove remote Tilde lock files as routine closeout cleanup after a failed or partial apply. Lock cleanup is a
  separate recovery step after bounded holder checks and, when any uncertainty remains, explicit user approval.
- Never plan a remote host from the controller.
- For mutating remote workflows, verify target Tilde runtime freshness and Git-backed desired-state checkout freshness
  before the first target `status`, `plan`, or `apply`.
- Do not run target `tilde status`, `tilde plan`, or `tilde apply` through a stale target runtime. Refresh the target
  runtime first, or stop with `deferred` if it cannot be safely refreshed.
- Do not patch target runtime or desired-state checkouts as part of deploy/update/repair to work around a bug and then
  continue as if the workflow succeeded. Save the diff, report it, and fix the controller source repository through
  normal review and commit flow. Temporary target edits are allowed only as a separate explicitly approved
  recovery/debug operation.
- Never read controller `~/.local/state/tilde/state.yml` for a remote target.
- Never execute `/tilde ...` or `$tilde ...` in a shell; they are prompt markers, not runtime commands.
- Before any controller-side runtime call, resolve `TILDE` to the loaded skill directory's `bin/tilde`, falling back to
  `~/.agents/skills/tilde/bin/tilde`.
- If bare `tilde` is not found on the controller, do not search for it; rerun through `"$TILDE"`.
- Do not run agent-orchestrated prompt routes through `"$TILDE"`: `deploy`, `update`, `repair`, `upgrade`, `adopt`,
  `create`, `init`, `clean`, and `organize`. `align` is a direct route only for current-host reconciliation; remote
  align is still a prompt workflow.
- For remote `/tilde align HOST`, do not run `tilde align --format json` on the target. Read target bindings on the
  target, then run `tilde plan --mode align --format json` for each target repository and `tilde apply` the plan files.
- Never redirect remote Tilde stderr to `/dev/null`. Non-zero remote exit status means the step failed, deferred, or
  conflicted, even when the tool output pane says `(no output)`.
- After `tilde apply`, inspect action results. Do not report success only because JSON was printed or `completed` is
  true; any `deferred`, `conflict`, or `notok` action means the run is incomplete.
- After a `notok` section or package result, do not run module snippets manually on the target for debugging or repair.
  Diagnose from structured output and source, fix desired state, then retry through `deploy` or `repair`. Any
  target-side manual mutation is a separate operator workflow and needs explicit approval.
- Inspect `ignored` and empty-output `ok` actions against the requested host. A guarded desktop install that returns
  `ok` with no output may have skipped the real package work; verify expected package state and condition diagnostics
  for modules such as terminals, browsers, Flatpak apps, and desktop tooling.
- Package declarations without a `type:` prefix use the platform default package manager: `brew` on Linux and macOS,
  and `scoop` on Windows. Do not rewrite unprefixed Linux packages as `brew:` just for clarity, and do not inspect
  `dpkg` or apt state to decide whether an unprefixed package is installed. Debian packages must be declared with an
  explicit `deb:` prefix.
- A final status read is verification only; it does not make an earlier failed remote `checkout`, `plan`, `apply`, or
  align step successful.
- Call `checkout remote` only with the complete mapping: `--host HOST --repo CONTROLLER_REPO --target TARGET_REPO`.
  Do not expect it to infer target paths from `--host`. Quote target paths that start with `~`.
- Use the `plan --mode refresh` implementation route for the `update` prompt command only when target status has
  existing `applied` anchors. If target status shows missing `state.yml` or no `applied` anchors, use
  `plan --mode repair` for state recovery.
- For host-aware prompt commands, omitted target means current host. Treat bare `host` as `ssh:host`, except when it
  names the current host; then run the local workflow. Use explicit `ssh:host` to force SSH transport.
- Missing `state.yml` or missing `applied` anchors means state recovery, not proof that the host was never deployed.
- Remote status paths outside the `state.yml` model, such as `config.yml` or `hosts/HOST/state.md`, mean stale target
  runtime, not successful current state recovery.
- Treat bare all-caps targets such as `ALL`, `HOME`, and `WORK` as home-policy host groups, not hostnames. For defined
  groups, report the expanded host list and continue; do not ask for permission to start the exact requested workflow.
  Ask only if the active home policy does not define the requested group, the expansion is ambiguous, or it unexpectedly
  leaves active policy.
- When traversing a host group, skip unreachable hosts after a bounded reachability check and continue with reachable
  hosts. Run this check through `"$TILDE" ssh`; do not use raw `ssh` for Tilde remote workflow probes.
- After remote runtime freshness is verified, read `tilde status --format json` on the target, bind the returned
  repository paths to shell variables, and use those variables for remote plan paths. Do not guess Dropbox or
  Git-backed checkout paths when status or explicit policy can supply them.
- Do not create `~/Dropbox` on a target just because controller source repositories live under Dropbox, an example uses
  `~/Dropbox/home`, or post-update cleanup mentions Dropbox. If target status binds Git-backed repositories under
  `~/.local/src`, use those paths and skip Dropbox-only maintenance when `~/Dropbox` is absent.
- Use `mktemp -d` and `trap` for all local and remote plan JSON files. Do not write fixed `/tmp/opencode/...` plan
  paths or leave generated plan files behind after apply.
- Do not say `applied` anchors advanced after a successful `refresh`/`update` run. Refresh mode does not write anchors;
  distinguish target HEAD from applied anchors in final status.
- Keep remote scripts `sh`-compatible unless Bash is explicitly required and the target is known to support it. Do not
  use Bash-only options such as `set -o pipefail` in the default `"$TILDE" ssh HOST << 'SCRIPT'` form.
- In default remote scripts, avoid Bashisms such as `[[ ]]`, arrays, `source`, `local`, `pipefail`, process
  substitution, here-strings, brace expansion, `mapfile`, and `$'...'` strings.
- Do not pass `-- bash` for macOS targets or ordinary remote status, plan, apply, and verification scripts.
- Treat package-manager metadata refresh warnings that report incomplete indexes, signature failures, or missing
  repository keys as `deferred`; do not summarize them as successful package refreshes.
- After a successful mutating remote apply, run a final target status read before the final answer.
- In remote summaries, distinguish `target HEAD` from `applied anchor`; do not call the target repository commit
  `local`.
- After an apply conflict, stop and wait for an explicit user answer before copying repo files over targets, forcing
  links, or rerunning apply. Newly revealed conflicts need a new explicit answer.
- After the user chooses repository replacement for a conflict, back up and remove only the exact conflicted target, then
  rerun apply. Broad directory cleanup needs a separate explicit approval.
- Run `"$TILDE" handoff --host HOST --copy` on the controller after sudo deferral. Do not run handoff through
  `"$TILDE" ssh`, do not invent a two-line `TILDE=...` command, do not rewrite the printed `Handoff command:`, and do
  not replace first-time bootstrap handoff with a handcrafted sudoers command.
- Remove packages, file materializations, files, or spans only with explicit managedness proof and proposal-first
  confirmation.
- Prefer a safe `deferred` or `conflict` result over guessing.

---
> Source: [roktas/tilde](https://github.com/roktas/tilde) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
