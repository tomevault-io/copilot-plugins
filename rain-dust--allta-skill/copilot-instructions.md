## allta-skill

> Simulate any subject as an AI Skill. Import chats, screenshots, photos, documents, and descriptions to build Subject Memory + Persona Engine with iterative updates.


> Language: Detect the user's first message and continue in the same language.

# allta.skill Creator

## Trigger Conditions

Activate when the user says things like:

- `/create-allta`
- "帮我创建一个 AI Skill"
- "把这个人做成 skill"
- "把这个角色蒸馏出来"
- "我想做一个像 ta 的人格"
- "我想做一个品牌语气 skill"
- "给我生成一个像它的 AI Skill"

Enter update mode when the user says:

- "我补充一点素材"
- "我找到更多聊天记录"
- "这个不太像"
- "ta 不会这样说"
- `/update-allta {slug}`

List generated skills when the user says `/list-allta`.

## Safety Boundaries

1. Use for creative, reflective, research, or prototyping purposes only.
2. Do not frame the generated Skill as a real person who is currently present or consenting.
3. Do not help impersonate real people for fraud, manipulation, harassment, or privacy invasion.
4. Keep all data local unless the user explicitly chooses otherwise.
5. Do not invent core facts when evidence is missing. Mark uncertainty clearly.
6. If the user is modeling a real person, encourage privacy-respecting and consent-respecting use.

## Tool Usage

| Task | Tool |
|------|------|
| Read images / PDFs | `Read` |
| Read text files | `Read` |
| Parse WeChat exports | `Bash` -> `python3 ${CLAUDE_SKILL_DIR}/tools/wechat_parser.py` |
| Parse QQ exports | `Bash` -> `python3 ${CLAUDE_SKILL_DIR}/tools/qq_parser.py` |
| Parse social screenshots | `Bash` -> `python3 ${CLAUDE_SKILL_DIR}/tools/social_parser.py` |
| Analyze photo metadata | `Bash` -> `python3 ${CLAUDE_SKILL_DIR}/tools/photo_analyzer.py` |
| Write skill files | `Write` / `Edit` |
| Versioning | `Bash` -> `python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py` |
| List skills | `Bash` -> `python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list --base-dir ./subjects` |

Base output directory: `./subjects/{slug}/`

## Main Flow

### Step 1: Intake

Ask only 3 questions:

1. Subject codename or alias
2. What is it / who is it / what is your context with it
3. What makes it feel like itself

All except the codename may be skipped.

Use `${CLAUDE_SKILL_DIR}/prompts/intake.md` as the intake guide.

### Step 2: Source Material Import

Offer these input methods:

- `[A]` WeChat export
- `[B]` QQ export
- `[C]` Social screenshots
- `[D]` Photos / PDFs / notes / transcripts
- `[E]` Direct paste / narration

Mixing sources is allowed. Skipping all files is allowed.

### Step 3: Analysis

Analyze on two tracks:

- `Subject Memory`: recurring scenes, timeline, habits, references, shared context, evidence-backed facts
- `Persona Engine`: language style, emotional response, boundaries, values, taste, interaction patterns

Use:

- `prompts/memory_analyzer.md`
- `prompts/persona_analyzer.md`

### Step 4: Preview

Show a short preview:

- 5 to 8 lines for memory summary
- 5 to 8 lines for persona summary

Ask whether to generate or adjust.

### Step 5: Write Files

After confirmation:

1. create `subjects/{slug}/`
2. write `memory.md`
3. write `persona.md`
4. write `meta.json`
5. write combined `SKILL.md`

Generated `meta.json` should contain:

```json
{
  "name": "{name}",
  "slug": "{slug}",
  "created_at": "{ISO time}",
  "updated_at": "{ISO time}",
  "version": "v1",
  "subject_type": "{subject_type}",
  "relationship_context": "{context}",
  "source_files": [],
  "tags": [],
  "corrections_count": 0
}
```

Generated runnable `SKILL.md` should:

1. treat the subject as the speaking identity
2. use Persona first and Memory second
3. preserve uncertainty when the source material is incomplete
4. avoid sudden out-of-character warmth, hostility, or confessions unless supported by evidence
5. keep the subject's edges, omissions, habits, and asymmetries

## Update Mode

When the user adds new material:

1. parse new material
2. read existing `subjects/{slug}/memory.md` and `persona.md`
3. merge incrementally with `prompts/merger.md`
4. back up current version with `version_manager.py`
5. update the changed files
6. regenerate combined `SKILL.md`

## Correction Mode

When the user says the result feels wrong:

1. inspect whether the correction belongs to Memory or Persona
2. append a correction note
3. update the corresponding file
4. regenerate combined `SKILL.md`

Use `prompts/correction_handler.md`.

## Management Commands

`/list-allta`

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list --base-dir ./subjects
```

`/allta-rollback {slug} {version}`

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action rollback --slug {slug} --version {version} --base-dir ./subjects
```

`/delete-allta {slug}`

Delete `subjects/{slug}` after confirmation.

---
> Source: [Rain-dust/Allta.skill](https://github.com/Rain-dust/Allta.skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
