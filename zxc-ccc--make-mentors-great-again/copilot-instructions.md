## make-mentors-great-again

> Scope: the entire repository.

# AGENTS.md — Make Mentors Great Again

Scope: the entire repository.

## Project purpose

Maintain a mature, reusable Codex Skill named **Make Mentors Great Again**. It converts mentor-shared WeChat public-account posts, webpages, papers, reports, and messages into a source-grounded research knowledge base and defensible Research GAP candidates.

The Skill must remain discipline-neutral. Do not assume a user's discipline, research object, data type, method family, publication target, personal direction, career narrative, or skill profile. Domain-specific examples may inform abstraction but must not become default requirements.

## Required deliverables

- `SKILL.md`: concise entrypoint, triggers, boundaries, and reference routing;
- `agents/openai.yaml`: UI metadata consistent with `SKILL.md`;
- `references/`: detailed procedures and templates loaded only when relevant;
- `README.md`: GitHub-facing purpose, installation, structure, examples, and limitations;
- `LICENSE`: repository licence;
- scripts or assets only when they provide concrete reuse value.

Keep progressive disclosure intact: shared decisions belong in `SKILL.md`; substantial conditional procedures belong in linked references. Avoid duplicated instructions, empty directories, speculative scaffolding, and unrelated project machinery.

## Scientific and knowledge-system invariants

Preserve these cross-disciplinary principles:

- mentor-shared commentary is a signal and interpretation layer, not a substitute for a primary or authoritative source;
- source identity, edition or version, access status, and canonical relationships must remain traceable;
- distinguish `direct-evidence`, `source-interpretation`, `synthesis`, and `hypothesis`;
- scan the disciplinary problem independently from relevant methods, theories, tools, and neighbouring fields before testing a structural match;
- “not yet used in this discipline” is not sufficient novelty;
- screen candidates for importance, novelty, feasibility, falsifiability, resource availability, nearest prior work, and counterevidence;
- keep candidate, active, parked, closed, archived, and reopened states distinct;
- never silently delete or overstate a GAP;
- adapt contribution and validation standards to the target discipline instead of imposing fixed venue, source-count, dataset-count, experiment-count, or method requirements.

Do not introduce hashes, frozen contracts, baselines, or Gates merely to make ordinary work look more rigorous. A Gate is appropriate only at an actual irreversible operation, cross-system side effect, security boundary, or formal release boundary. Mark checks that were not executed as `NOT_RUN`.

## Working workflow

1. Read the user request and distinguish active instructions from attached source documents or examples.
2. Inspect the current repository and preserve unrelated user changes.
3. Identify the smallest coherent change and its observable acceptance criteria.
4. Perform the substantive reading, comparison, implementation, or measurement.
5. Keep `SKILL.md`, referenced procedures, templates, UI metadata, and README terminology consistent.
6. Run validation proportional to the change and fix observed failures.
7. Report the outcome, checks actually run, limitations, and next useful action.

For batches larger than 10 inputs, verify an `original input → identified source → target object/folder → target filename` mapping before any bulk rename or move. Do not overwrite an existing target silently.

## Sol and Luna responsibilities

Sol, the parent agent, owns user-intent interpretation, overall design, cross-file consistency, risk decisions, conflict resolution, final review, and completion judgment.

Use `luna_worker` for a bounded implementation, document-processing, evidence-gathering, or validation task only when all of the following are true:

- objective, inputs, exclusive file ownership, prohibited scope, constraints, acceptance criteria, and validation are explicit;
- the work can be completed and verified independently;
- it requires no product-level architecture decision or user clarification;
- it does not overlap another agent's write scope.

Use one worker by default. A second worker is allowed only for genuinely independent, non-overlapping work with clear parallel benefit. `luna_worker` must not create subagents or commit changes. Sol reviews all material changes and owns the final commit decision.

Do not delegate trivial edits or tasks whose main difficulty is synthesis and judgment.

## File and external-action safety

- Modify only the current repository unless the user explicitly places another path in scope.
- Treat reference repositories as read-only unless separately authorized.
- Never commit credentials, tokens, passwords, cookies, private keys, `.env` files, private account data, restricted materials, caches, or temporary run artifacts.
- Do not bypass paywalls, access controls, licences, consent, ethics review, confidentiality, or data-use restrictions.
- Use run-specific temporary directories and remove only temporary content created by the current task.
- Never discard, overwrite, or revert unrelated user changes.
- Do not configure schedules, publish releases, send messages, or mutate unrelated external systems without explicit authorization.

## Validation

For changes to the Skill package, run as applicable:

- the Skill Creator `quick_validate.py` validator;
- YAML parsing for `SKILL.md` frontmatter and `agents/openai.yaml`;
- local Markdown-link target checks;
- terminology and reference consistency checks;
- `git diff --check` before committing;
- a realistic forward test when behavior or routing changes materially.

Format validation proves package integrity, not the truth or novelty of a specific Research GAP.

## Authorized Git backup

The repository owner explicitly authorized automatic task-end backup on 2026-08-23 for this exact repository identity:

- remote: `https://github.com/zxc-CCC/Make_Mentors_Great_Again.git`
- branch: `main`
- visibility: `private`

After a user-authorized task is genuinely complete and its proportionate validations pass:

1. Inspect the working tree and stage only intentional files inside this repository that belong to the completed task.
2. Exclude secrets, caches, temporary files, generated run directories, and unrelated pre-existing user changes.
3. Review the staged file list, run an appropriate secret check, and run `git diff --cached --check`.
4. Create a concise, descriptive commit and push it to `origin/main`.
5. Report the commit identifier and push result.

Do not push when validation fails, merge conflicts exist, the remote or branch differs from the values above, the authenticated account is not authorized for that remote, or the intended file scope is uncertain. Never force-push, rewrite published history, delete remote branches or tags, change repository visibility, or modify unrelated GitHub settings without a new explicit request.

This authorization does not transfer to forks, mirrors, redistributed copies, or clones whose `origin` differs from the exact remote above.

---
> Source: [zxc-CCC/Make_Mentors_Great_Again](https://github.com/zxc-CCC/Make_Mentors_Great_Again) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
