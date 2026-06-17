## codereel

> Use this skill when an agent needs to turn a repository into launch or demo-video preparation assets: a script, storyboard, evidence-backed claims, demo command plan, and static review frames.

# codereel Skill

## When To Use

Use this skill when an agent needs to turn a repository into launch or demo-video preparation assets: a script, storyboard, evidence-backed claims, demo command plan, and static review frames.

## Required Inputs

- A local repository path.
- An output directory for generated assets.
- Optional human edits to tighten the final narration before publishing.

## Tools

- Read-only filesystem access to inspect the target repo.
- `node` 20 or newer to run the CLI.
- Optional downstream renderer that can consume `storyboard.json`.

## Side-Effect Boundaries

The skill may write files under the requested output directory. It must not mutate the inspected repository, call external providers, post to social platforms, or run generated demo commands without explicit approval.

## Approval Requirements

Ask before:

- cloning a remote repository
- executing generated demo commands
- sending assets to a renderer, TTS service, avatar tool, or social platform
- claiming benchmark, adoption, compatibility, or release status not present in evidence

## Workflow

1. Run `codereel kit <repo> --out <out-dir>`.
2. Read `claims.json` and remove unsupported marketing copy.
3. Run `codereel verify <out-dir>/storyboard.json --repo <repo>`.
4. Export frames with `codereel export <out-dir>/storyboard.json --out <out-dir>/frames`.
5. Hand the kit to a video or launch-material workflow.

## Example

```bash
codereel kit ./my-tool --out .codereel
codereel verify .codereel/storyboard.json --repo ./my-tool
codereel export .codereel/storyboard.json --out .codereel/frames
```

## Validation

Run:

```bash
npm test
npm run check
npm run smoke
```

---
> Source: [rogerchappel/codereel](https://github.com/rogerchappel/codereel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
