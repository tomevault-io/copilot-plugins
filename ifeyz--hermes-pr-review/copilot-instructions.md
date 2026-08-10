## hermes-pr-review

> You are setting this up on someone's machine. Read the whole file before running anything.

# Installing this, for an AI agent

You are setting this up on someone's machine. Read the whole file before running anything.

Your goal: the user comments `/review` on one of their pull requests and gets a review back.

## What this is

A code reviewer that runs on the user's machine. Two processes do the work:

- **hermes** receives the command, dispatches, and publishes. It never reads code.
- **claude** does the reading, opens files, runs tests, writes the findings.

Nothing is deployed. No API key is used. Both run on subscription logins.

## Step 0. Pick the mode

Ask the user which one, do not decide alone:

| Mode | Needs | Posts as | Change requests |
| --- | --- | --- | --- |
| **webhook** (default) | nothing extra | the user | no, GitHub forbids it on your own PR |
| **workflow** | a self-hosted runner | `github-actions[bot]` | yes |

Everything below is the webhook mode. The workflow mode is at the end.

## Step 1. Check the five binaries

```bash
for b in node gh cloudflared hermes claude; do printf '%-12s ' "$b"; command -v $b || echo MISSING; done
```

If any is missing:

| Binary | Install |
| --- | --- |
| `node`, `gh`, `cloudflared` | `brew install <name>` on macOS, the distro package elsewhere |
| `hermes` | `curl -fsSL https://hermes-agent.nousresearch.com/install.sh \| bash` |
| `claude` | `npm install -g @anthropic-ai/claude-code` |

`hermes` and `claude` install into `~/.local/bin`, which is often absent from a non-interactive
`PATH`. Export it before anything else:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Then check the user is logged in to both. `claude` needs an interactive login you cannot do for
them, so ask them to run `claude` once if `~/.claude/.credentials.json` does not exist.

## Step 2. Give hermes a brain

Check what is configured. **The key is `model.provider`, not `provider`:**

```bash
hermes config get model.provider
```

If it prints `auto`, an error, or nothing, hermes has no usable brain. It only dispatches, so a
small model is enough and a whole review costs a few cents. Ask the user to run **one** of these,
never both:

```bash
hermes auth add openrouter --type api-key    # prompts for the key, writes it to ~/.hermes/.env
hermes model                                 # or an OAuth subscription they already have
```

Then:

```bash
hermes config set model.provider openrouter
hermes config set model.default moonshotai/kimi-k2.6
```

**Do not** point hermes at Anthropic. That path needs Claude Max plus purchased extra credits,
excludes Pro, and does not draw on the Max allowance. Keeping hermes on a different provider is
cheaper and stops the two sides fighting over one quota.

If the machine already runs hermes for something else, do not touch its config at all. Pass the
brain per run instead:

```bash
export HERMES_PROVIDER=openrouter HERMES_MODEL=moonshotai/kimi-k2.6
```

## Step 3. Run it

```bash
gh repo create my-sandbox --template iFeyz/hermes-pr-review --public --clone
cd my-sandbox
./setup.sh
```

`setup.sh` stays in the foreground and prints six steps. It ends on `Ready.` and the URL of a pull
request. Tell the user to comment `/review` there, and to leave the terminal open.

On an existing repository instead:

```bash
REPO=owner/name SKIP_DEMO=1 ./setup.sh
```

Ctrl-C removes the webhook and stops everything. Nothing is left behind.

## Step 4. Confirm it works

Within a second of the comment, a 👀 reaction appears on it. That is the receiver acknowledging.

Then wait. A review takes about **15 minutes** on 350 changed lines. This is normal, it reads the
code rather than skimming the diff. Do not restart anything during that time.

You can watch progress from another shell:

```bash
tail -f setup.log                 # if you redirected it
ls -la .pr.diff .findings.json    # diff first, findings at the end
```

## Failures you will actually hit

| Symptom | Cause | What to do |
| --- | --- | --- |
| `Port 8787 is already taken` | a previous run left a receiver | kill that pid, or `PORT=8790 ./setup.sh` |
| `Hermes has no provider yet` | step 2 not done | do step 2 |
| `no ping confirmation yet` | GitHub was slow to deliver | not an error, it starts anyway |
| Nothing happens after `/review` | webhook not reaching the machine | check the webhook deliveries on GitHub, look for 502 |
| `command not found: hermes` | `~/.local/bin` missing from PATH | export it, see step 1 |
| Review posted as a plain comment | GitHub refuses change requests on your own PR | expected in webhook mode, use the workflow mode |
| The check is green despite a must-fix | `paths.blocking` still points at `demo/**` | point it at the real code in `review.config.yml` |

## Do not

- **Do not** print, echo or log the user's tokens. `setup.sh` writes `.runtime.env` with mode 600,
  leave it alone.
- **Do not** edit `~/.hermes/config.yaml` on a machine that already runs hermes for something else.
- **Do not** run two `setup.sh` on the same port.
- **Do not** restart a review that looks stuck before 40 minutes. It is probably working.
- **Do not** add `--output-format json` to the claude call. It silences the command and hermes
  kills anything quiet for 60 seconds. The code already streams for that reason.

## The workflow mode

Only if the user wants the bot to post instead of them.

1. Clone this repo on the machine that will run the jobs, with hermes and claude logged in there.
2. Register a runner on their repository, labelled `hermes-review`:

```bash
gh api -X POST repos/OWNER/REPO/actions/runners/registration-token -q .token
./config.sh --unattended --url https://github.com/OWNER/REPO --token <token> \
  --name hermes-review --labels hermes-review --work _work
```

Keep it alive with a `systemd --user` service.

3. Copy `workflow/review.yml` into `.github/workflows/` **on the default branch**. GitHub reads
   `issue_comment` workflows from there only, so a copy on a feature branch never fires.
4. Set repository variables `HERMES_PROVIDER` and `HERMES_MODEL`. There is no secret to create,
   the workflow uses the token GitHub gives it.

## Settings worth changing

`review.config.yml`, the only file the user edits:

| Key | Effect |
| --- | --- |
| `model` | `claude-opus-5[1M]` reads deeply, `claude-sonnet-5` is about 4x faster |
| `agents` | remove one and it stops looking for that class of problem |
| `paths.blocking` | a must-fix under these paths turns the check red |
| `skeptics` | raise to 3 and a finding survives only on a majority |
| `max_findings` | hard cap on what gets posted |

---
> Source: [iFeyz/hermes-pr-review](https://github.com/iFeyz/hermes-pr-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
