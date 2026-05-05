## noveltools-statuses

> NovelTools scene status system — use when working with noveltools.json, scene statuses, or manuscript organization


# NovelTools Scene Status System

This project uses NovelTools, a VS Code/Cursor extension for novel writing. Scenes are individual `.md` files organized into chapter folders.

## Project File

The manuscript structure is defined in `noveltools.json` at the workspace root.

## Scene Statuses

Scenes have 6 possible statuses stored in `noveltools.json` under `sceneStatus`. Keys are relative file paths (forward slashes).

| Status     | Icon | Meaning                                    |
|------------|------|---------------------------------------------|
| `drafted`  | 🟡   | First draft written                         |
| `revision` | 🔵   | Undergoing revision                         |
| `review`   | 🟠   | Ready for / under review                    |
| `done`     | 🟢   | Final, complete                             |
| `spiked`   | 🔴   | Temporarily removed (may return)            |
| `cut`      | ⚫   | Permanently removed from manuscript         |

## How to Read Scene Status

Open `noveltools.json` and look at the `sceneStatus` object:

```json
{
  "sceneStatus": {
    "chapter-01/opening.md": "done",
    "chapter-01/conflict.md": "revision",
    "chapter-02/intro.md": "drafted"
  }
}
```

A scene with no entry in `sceneStatus` has no status set (untracked).

## How to Change Scene Status

Edit the `sceneStatus` object in `noveltools.json` directly. Set the value to one of: `drafted`, `revision`, `review`, `done`, `spiked`, `cut`.

```json
{
  "sceneStatus": {
    "chapter-01/opening.md": "review"
  }
}
```

To clear a status, remove the key from `sceneStatus`.

## Scene Metadata

Per-scene metadata (like synopses) is stored under `sceneMetadata`:

```json
{
  "sceneMetadata": {
    "chapter-01/opening.md": {
      "synopsis": "Elena arrives at the apartment building for the first time."
    }
  }
}
```

## VS Code Commands

These commands are also available via the command palette:
- `NovelTools: Set section status...` — quick-pick with all 6 options
- Individual status commands: `Set status: Done`, `Set status: Drafted`, `Set status: Revision`, `Set status: Review`, `Set status: Spiked out`, `Set status: Cut`
- `Clear section status` — removes status from a scene

---
> Source: [Arcadesys/NovelTools](https://github.com/Arcadesys/NovelTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
