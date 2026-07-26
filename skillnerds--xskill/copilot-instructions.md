## xskill

> xskill has 3 types of agents: TaskAgent, TaskClusterAgent and SkillEditAgent.

## Agent design
xskill has 3 types of agents: TaskAgent, TaskClusterAgent and SkillEditAgent.

## TaskAgent

Trajectories are not the minimal unit xskill uses to construct skill. A trajectory is split into `AtomTask`, which is a slice of user chat trajectory. AtomTask is the minimal unit of skill-construction, split from chat trajectory by TaskAgent.

The TaskAgent reads trajectories from dirs like `~/.xskill/cc_sessions/traj_cc_dsv4_890da9d9.md` and agentically splits them into AtomTasks based on semantic boundaries, such as topic changes, task completions, or user intent shifts. Each AtomTask represents a coherent segment of the interaction that can be independently analyzed and potentially used as a skill component.

**Triggered:** every 30s, the watcher scans the registered coding-agent dirs. If a new traj or new appended content (delta) is found, the watcher triggers a TaskAgent to decompose this delta-traj.

**Input:** trajectories collected from different coding agents by xskill-watcher, e.g. `~/.xskill/cc_sessions/traj_cc_dsv4_890da9d9.md` or `~/.xskill/codex_sessions/traj_codex_learn_019e2630.md`.

**Output:** AtomTask files located at `~/.xskill/cc_sessions/traj_cc_dsv4_890da9d9/tasks/atom_traj_cc_dsv4_890da9d9_0002.json`. Usually many atom files in the same folder.

### AtomTask

AtomTask has the following key fields:

| field | description |
|-------|-------------|
| `atom_id` | unique id, format: `atom_<traj_id>_NNNN` |
| `intent` | user intent of this segment |
| `summary` | short summary of what happened |
| `tags` | semantic tags |
| `used_skills` | skills the coding agent self-reported using |
| `ux_score` | UX score used for canary/grey-testing skill eval |
| `offset_start` / `offset_end` | character offsets in the original `.md` file |
| `pre_atom_id` / `post_atom_id` | linked-list neighbor atoms |

Path: `~/.xskill/cc_sessions/<traj_id>/tasks/atom_<traj_id>_NNNN.json`

```json
{
  "atom_id": "atom_traj_cc_dsv4_890da9d9_0001",
  "traj_id": "traj_cc_dsv4_890da9d9",
  "offset_start": 120,
  "offset_end": 2400,
  "intent": "部署 xquiz 到 1717 端口",
  "summary": "agent 克隆 repo、读 README、配置端口启动",
  "tags": ["deploy", "fastapi"],
  "used_skills": ["python-deploy"],
  "ux_score": 7,
  "pre_atom_id": null,
  "post_atom_id": "atom_traj_cc_dsv4_890da9d9_0002",
  "context_prefix": "",
  "raw_segment": ""
}
```

## TaskClusterAgent (TCA)

Each AtomTask triggers a TaskClusterAgent, which analyzes the atom against existing skills using a skill-listing mechanism (name + description catalog, similar to Claude Code's approach), then either creates a new skill folder (baby state) with the atom's contribution, or adds the atom into an existing skill's `.candidates.yml`.

TCA can also rename a baby skill, or move an atom to another skill folder it considers a better fit.

**Triggered:** async per-atom from watcher's `_do_cluster` pipeline (or via `process_atom_task` in process.py), one TCA invocation per atom.

**Input:** a single `AtomTask` object (plus `skill_dir` to scan existing skill catalog). The LLM sees the full skill catalog (name + truncated description) as routing context.

**Output:** side effect only — writes to `.candidates.yml` of the chosen skill (with `weightscore` 0–10), and optionally creates a new skill folder via `new_skill_folder` tool. Returns the agent's final content string (for logging).

## SkillEditAgent (SEA)

Triggered when a skill's `.candidates.yml` cumulative `weightscore >= 10`. The agent reads atom content (via `AtomTaskRead`), optionally reads trajectory raw text (via `ReadTraj`), then produces or updates the skill's `SKILL.md` and any auxiliary files (scripts, references). After writing, it must call one of two commit tools depending on the current branch state:

- **baby branch** → `commit_baby_to_main()` — first public version
- **main branch** → `commit_to_staging()` — grey candidate for canary comparison

**Triggered:** watcher's `_check_pending_skill_edits` loop; guarded by:
1. No staging branch exists (skill not already in grey-test)
2. Candidates weightscore >= threshold (default 10)
3. If on main branch: must have at least 1 real `side=main` UX score (proves main is used)

**Input:** accumulated candidates from the skill's `.candidates.yml` (list of atom contributions with weightscore).

**Output:** SKILL.md and auxiliary files in the skill directory, plus a git commit (baby→main or main→staging) to version the result.

---
> Source: [SkillNerds/xskill](https://github.com/SkillNerds/xskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
