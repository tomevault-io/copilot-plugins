## browser-cli

> Control the user's current Chrome through the local browser-cli daemon and native CDP. Use when a task needs the already-open logged-in browser, daemon reuse, worker tabs, snapshots, clicks, typing, or low-friction current-browser automation instead of a fresh browser session.


# browser-cli

Use this skill when the best browser path is the current Chrome session, not a fresh browser instance.

## Preconditions

1. Chrome is already running
2. `chrome://inspect/#remote-debugging` has `Allow remote debugging for this browser instance` enabled
3. `browser-cli` is available in this repo or local install

## Default Workflow

1. If a daemon is already running, inspect it first:

```cmd
bin\browser-cli.cmd status --port 54000
```

2. If no daemon is running, check reachability and start one:

```cmd
bin\browser-cli.cmd doctor
bin\daemon-start.cmd 54000
```

3. Open or reuse a worker tab:

```cmd
bin\browser-cli.cmd tab-open --port 54000 --url https://example.com --id task-tab
```

4. Keep sending actions through the same worker:

```cmd
bin\browser-cli.cmd snapshot --port 54000 --worker task-tab
bin\browser-cli.cmd click --port 54000 --worker task-tab --json "{\"ref\":\"button.submit\"}"
bin\browser-cli.cmd type --port 54000 --worker task-tab --json "{\"ref\":\"input[name=q]\",\"text\":\"hello\"}"
```

5. Stop the daemon when the task line is complete:

```cmd
bin\daemon-stop.cmd 54000
```

## Operating Rules

- Prefer reusing the same daemon and the same worker for one task line.
- Prefer `status` over repeated fresh `doctor` probes once a daemon is already attached.
- Snapshot before high-risk click or type steps.
- Treat current-browser mode as shared state.
- Do not assume multi-agent parallel work is isolated just because different workers exist.
- If `doctor` fails, troubleshoot the browser session first before redesigning the task.

## What This Skill Does Not Try To Do

- It does not replace full browser frameworks.
- It does not provide a generic MCP server abstraction.
- It does not hide the fact that current-browser mode shares state.
- It does not bundle site-specific browsing strategies into the skill itself.

## References

- [README.md](./README.md)
- [docs/comparison.md](./docs/comparison.md)
- [docs/windows-validation.md](./docs/windows-validation.md)

---
> Source: [li-xiu-qi/browser-cli](https://github.com/li-xiu-qi/browser-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
