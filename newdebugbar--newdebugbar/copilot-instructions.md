## newdebugbar

> - Make small commits as you work.

# New Debug Bar rules

## Commits

- Make small commits as you work.
- Keep each commit about one clear change. Commit its tests and built files with it.

## Product choices

- Build for Laravel only. Do not add support for other PHP frameworks.
- Match the minimum PHP and Laravel versions supported by Livewire 4.
- Use `NewDebugBar` or `newdebugbar` as one word in machine-facing names. Use “New Debug Bar” in text written for people.
- Make the full local debugging experience work immediately. Do not hide useful diagnostics behind opt-in flags or masked defaults only because the captured data may be sensitive.
- Add a config value only when developers have a real, repeated reason to change the behavior. Every value must have a distinct runtime effect and a clear reason for its default. Otherwise, use one fixed product behavior and remove the setting, branches, and tests.
- Use a protective default only when the normal behavior could change external state, break the host app, or create unbounded work or storage. Local diagnostic visibility by itself is not a reason to disable a feature.
- Treat the local MCP server as a main product feature. Explain that coding agents can read exact debug data instead of guessing from a web page.
- Keep the public README short. Explain why the package exists and how to start using it.
- Keep client-specific MCP setup in `docs/mcp.md`. Link to it from the README.
- Keep test reports, support tables, and long setup notes out of the README.
- Treat the first public release as v1. Do not add a changelog for work done before v1.
- Ask the user before changing the license or copyright owner.

## Interface

- Make the bar look clean and modern. It should feel at home on the page while a developer works.
- Do not use `·`, `•`, or `|` to split facts. Use space, labels, icons, or groups.
- Help developers answer: What happened? What is wrong? Why? Where? What should I check next?
- Show the request, errors, query count, and time first.
- Keep framework details, raw data, hashes, and repeated facts out of the main view.
- A finding should explain the problem, why it matters, where it came from, and what to do next.
- Do not show two findings for the same cause.

## Checking interface work

- Start with the built-in browser.
- Use the canonical example repositories to check Blade, Livewire, and Inertia:
  - Blade: local `../example-blade-app` at `http://example-blade-app.test`; GitHub `newdebugbar/example-blade-app` at `https://github.com/newdebugbar/example-blade-app`
  - Livewire: local `../example-livewire-app` at `http://example-livewire-app.test`; GitHub `newdebugbar/example-livewire-app` at `https://github.com/newdebugbar/example-livewire-app`
  - Inertia: local `../example-inertia-app` at `http://example-inertia-app.test`; GitHub `newdebugbar/example-inertia-app` at `https://github.com/newdebugbar/example-inertia-app`
- Do not change the example apps just to make a package test pass.
- Check one example app or browser test group at a time.
- Check light and dark themes, keyboard use, browser errors, and a 390px-wide screen.
- Update screenshot baselines only for planned changes. Look at each changed image, then run the same checks again.

---
> Source: [newdebugbar/newdebugbar](https://github.com/newdebugbar/newdebugbar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
