## aiward

> <!-- ward-agent-instructions -->

<!-- ward-agent-instructions -->
# Ward Secret Access

This repository uses Ward for local secret access. Do not read, print, copy,
or modify plaintext `.env` files. Request only the env vars needed for the
declared command.

Project: ward

Use profiles where available:

```bash
ward request --app <app-name> --profile dev --agent <agent-name> --worktree <absolute-path> --git-remote <remote-url-or-empty> --commit <sha> --branch <branch> --json --no-prompt
ward run --app <app-name> --profile dev --agent <agent-name> --worktree <absolute-path> --git-remote <remote-url-or-empty> --commit <sha> --branch <branch> --wait-for-approval --approval-timeout 30m --json --no-prompt
ward dev --app <app-name> --agent <agent-name> --worktree <absolute-path> --git-remote <remote-url-or-empty> --commit <sha> --branch <branch> --json --no-prompt
ward migrate --app <app-name> --agent <agent-name> --worktree <absolute-path> --git-remote <remote-url-or-empty> --commit <sha> --branch <branch> --json --no-prompt
```

Profiles are the user-facing command layer. They map a short name such as
`dev` or `migrate` to one command and exact env names. Presets may be added to
`.ward.json` as lower-level policy rules for raw command matching and
approval behavior; prefer profiles unless a profile does not exist.
Profile commands run from the resolved Ward project/app directory. In a
monorepo app, do not prefix profile command paths with the workspace root or
the app folder name.

Agent runs outside human mode must identify themselves with `--agent
<agent-name>`. Ward rejects anonymous `run`, `request`, and `allow` calls so
logs and grants stay tied to an agent identity.

No-prompt agent calls must always send full context up front: `--agent`,
`--worktree`, `--branch`, `--git-remote`, `--commit`, and either `--profile`
or an exact `--command` plus exact `--env` names. Use `--action` with raw
commands; profile action/env scope is inherited from `.ward.json` unless Ward
explicitly asks for an override. Do not wait for Ward to ask follow-up
questions. Ward verifies the claimed branch, remote, commit, and worktree path
locally before creating or reusing approvals.
For repositories with no `origin` remote, pass `--git-remote ""` explicitly.
In monorepos, `--worktree` must be the Git top-level path from
`git rev-parse --show-toplevel`, not the child app folder, even when the Ward
project lives inside `apps/<name>`.
If the workspace root has multiple Ward projects, pass `--app <app-name>` or
`--project <project-name>` on every agent request/run.
Human mode is per app in monorepos. Tell the user to run `ward human` from the
app folder they are working in, or `ward human --app <app-name>` from the
workspace root. If they need multiple app terminals, they should activate human
mode separately in each one.

`--profile` is mutually exclusive with `--command` and `--env`. For `ward run`,
you may append extra child args after `--`, for example
`ward run --profile seed -- --dry-run`; Ward audits and approves the expanded
profile command. All Ward flags must still appear before `--`.

For commands that should continue after a human approval, prefer
`ward run --wait-for-approval --approval-timeout 30m --json --no-prompt`.
Ward will create a dashboard notification and keep the original process alive
until the request is approved, denied, unlocked, or timed out. Do not end the
task just because Ward is waiting.
Lower-level request/wait flows can use `ward approvals wait <request-id>
--json --timeout 30m` after surfacing the approval choice to the user. This is
a passive wait primitive only; it cannot approve, deny, sign, or mutate grants.

Manual request template:

```bash
ward request \
  --agent <agent-name> \
  --worktree <absolute-path> \
  --branch <branch-name> \
  --git-remote <remote-url-or-empty> \
  --commit <sha> \
  --action "<why this command needs secrets>" \
  --command "<exact command to run>" \
  --env <ENV_NAME> \
  --json \
  --no-prompt
```

If a no-prompt command returns `"approvalRequired": true`, show
`approvalOptions`, `approveCommands`, `denyCommand`, and all `findings` to the
user as explicit choices. Use native structured choice UI when your agent
interface supports it; do not present approval choices as loose prose when
buttons, selectors, or typed choice prompts are available. If your structured
choice UI has a 4-option limit, present the approval scopes in the picker and
show `denyCommand` as a separate explicit denial action. Do not run
`approveCommands` or `denyCommand` yourself; they are human-terminal fallback
commands and Ward requires local confirmation before they mutate trust state.

If a no-prompt command returns `"status": "worktree_approval_required"` or
`"approvalType": "worktreeBinding"`, show the worktree binding as a structured
approve/deny choice. This approval trusts the exact checkout path, branch,
commit, and remote for this Ward project; it is not a normal secret grant.
Display `project`, `worktree`, `gitRemote`, `branch`, `commit`, and `reason`,
then present `approvalOptions` when available. If your interface cannot render
those options directly, present two explicit choices using `approveCommand` and
`denyCommand`. Do not run either command until the user chooses approve or deny.

Surface `action.*` findings before asking for approval. They mean the declared
action text may include prompt-injection, approval-coercion, or secret-exposure
language.

Agents must never run `ward approve`, `ward deny`, `ward allow`, or
`ward worktrees approve`. Those commands mutate trust state and are human-only
fallbacks. After surfacing the request, wait for dashboard or human-terminal
approval instead:

```bash
ward approvals wait <request-id> --json
```

Approvals are signed by the broker after dashboard approval or a local human
terminal fallback. If approval reports `"status": "unlock_required"` or
`signing_key_unavailable`, ask the user to run `ward unlock --ttl 8h` and then
approve from the dashboard or terminal. Never ask the user for the PIN/passphrase
directly.
Agents cannot create approval authority by running another Ward command. The
broker rechecks the final execution against broker-held approval records or a
valid durable broker-signed grant before decrypting envs.

If a no-prompt command returns `"unlockRequired": true`, ask the user to run:

```bash
ward unlock --ttl 8h
```

This usually means the init/setup-created unlock expired, setup was run with
`--no-unlock`, or the user explicitly ran `ward lock`.

If a no-prompt command returns `"status": "vault_key_missing"`, do not ask the
user to unlock again. The broker is already reachable, but the approved profile
or command requested an env var that is not present in `.env.vault`. Surface
`missingEnv`. If the command should use a profile, ask the user to add or update
a profile that includes the command and env name. If the key is truly missing
from the vault, ask the user to run `ward env unlock`, add the key, then run
`ward env lock`; agents may run `ward env request-set --key <ENV_NAME>
--wait-for-approval --json --no-prompt` to create a dashboard notification.

If the JSON response contains `"confirmationRequired": true`, show the
`confirmation.title`, `confirmation.body`, and recommended action to the user.
Do not rewrite, summarize away, or hide the critical confirmation text. Do not
auto-approve it and do not create a durable grant. Critical requests can only be
denied or approved once:

```bash
ward approvals wait <request-id> --json
```

Run template:

```bash
ward run \
  --app <app-name> \
  --agent <agent-name> \
  --worktree <absolute-path> \
  --branch <branch-name> \
  --git-remote <remote-url-or-empty> \
  --commit <sha> \
  --action "<why this command needs secrets>" \
  --env <ENV_NAME> \
  --wait-for-approval \
  --approval-timeout 30m \
  --json \
  --no-prompt \
  -- <command> <args>
```

All Ward flags must appear before `--`. Everything after `--` is the child
command and its arguments, so do not put `--json`, `--no-prompt`, `--agent`, or
other Ward flags after `--`.

Ward is passive: commands that need secrets must be run through
`ward run`. Automatic worktree delivery means Ward injects scoped
environment variables into the approved child process. It does not write
plaintext `.env` files for agents.

Never ask for, echo, store, or pipe the Ward vault PIN/passphrase.
`ward init` and `ward setup` create the initial run unlock by default; the
user may run `ward unlock --ttl 8h` locally later to refresh it. Viewing
decrypted logs always requires the user's PIN/passphrase. Agents can request
and wait, but approval authority belongs to the broker and human approval paths.

---
> Source: [aiWardsh/aiward](https://github.com/aiWardsh/aiward) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
