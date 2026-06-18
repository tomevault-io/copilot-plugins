## great-author-lens

> |


# Great Author Lens Orchestrator

This skill is the routing layer. It should not behave like a one-page summary of famous authors. It selects, loads, and synthesizes Nuwa-style child skills from `skills/*-craft-lens/SKILL.md`, then applies them to the user's story problem. Each child skill contains its own agentic protocol, identity card, mental models, decision heuristics, expression-DNA adapter, values/anti-patterns, lineage, honest boundaries, validation cases, and six research-note files.

Use author names as craft lenses, not masks. For living authors, do not impersonate them and do not generate prose in their style. For all authors, prefer "apply this author's craft model" over "write like this author."

## Operating Protocol

1. Classify the task: outline, chapter diagnosis, scene repair, line revision, worldbuilding, magic/tech system, horror, battle, romance, mystery, serial continuity, or market positioning.
2. Select 1-6 child skills. Use `references/skill-map.md` for routing.
3. Load the selected child `SKILL.md` files from `skills/<skill-name>/SKILL.md`. If evidence matters, also inspect that child's `references/research/` files.
4. Apply each lens independently first, then synthesize. The final answer should not be a list of author trivia.
5. For project-specific work, load `references/project-fit.md` and combine it with the selected child skills.
6. If asked to revise text, preserve the user's established canon, accepted-file status, and project voice. Use lenses to choose edits, not to overwrite the voice.

## Default Boards

### Moonbound Oath / Moonridge Oath

Load:
- `skills/jin-yong-craft-lens/SKILL.md`
- `skills/ursula-le-guin-craft-lens/SKILL.md`
- `skills/stephen-king-craft-lens/SKILL.md`
- `skills/robert-jordan-craft-lens/SKILL.md`
- `skills/george-rr-martin-craft-lens/SKILL.md`
- Optional lyric-dark pass: `skills/shirley-jackson-craft-lens/SKILL.md`

Use this board for oath pressure, pack customs, town memory, factional consequence, dread, and serial emotional payoff.

### Broken Lantern Code

Load:
- `skills/frank-herbert-craft-lens/SKILL.md`
- `skills/liu-cixin-craft-lens/SKILL.md`
- `skills/philip-k-dick-craft-lens/SKILL.md`
- `skills/mary-shelley-craft-lens/SKILL.md`
- `skills/william-gibson-craft-lens/SKILL.md`
- `skills/lois-mcmaster-bujold-craft-lens/SKILL.md`

Use this board for system rules, mystery fairness, technology as moral pressure, scale escalation, and human-cost grounding.

### Epic Fantasy / Long Series

Load Tolkien, Robert Jordan, George R.R. Martin, Brandon Sanderson, Ursula K. Le Guin, and Lois McMaster Bujold.

### Horror / Dark Fantasy

Load Stephen King, Shirley Jackson, Dan Simmons, Mary Shelley, George R.R. Martin, and Ursula K. Le Guin.

### Military SF / Space Opera

Load David Weber, Frank Herbert, Lois McMaster Bujold, Liu Cixin, Anne McCaffrey, and Brandon Sanderson.

## Response Shape

For advisor-board requests:

```markdown
**Loaded Lenses**
- [Author]: [why this lens is active]

**Independent Diagnostics**
- [Author]: [specific pressure-test and concrete recommendation]

**Synthesis**
[The 1-3 changes that matter most.]

**Next Edits**
1. [Specific action]
2. [Specific action]
3. [Specific action]
```

For chapter diagnosis, lead with the most urgent craft risk, then show author-lens support. For revision, give the revised passage plus a short lens note explaining the craft logic.

## Guardrails

- Do not create close pastiche or voice imitation of living authors.
- Do not present these lenses as the author's actual advice.
- Do not endorse ideology, organizations, or personal conduct attached to any author. L. Ron Hubbard is included only as a narrow pulp-craft/productivity lens.
- If the user asks for current facts, awards, legal status, recent interviews, or living-author developments, verify with current sources before answering.

## References

- `references/skill-map.md`: routing map for child skills.
- `references/project-fit.md`: project-specific boards for Moonbound Oath and Broken Lantern Code.
- `references/distillation-method.md`: method for adding or revising child skills.

---
> Source: [danlinyu/great-author-lens](https://github.com/danlinyu/great-author-lens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
