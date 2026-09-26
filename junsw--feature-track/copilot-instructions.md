## feature-track

> Use Feature Track for feature-level project memory.

# Feature Track

Use Feature Track for feature-level project memory.

Before feature work:

1. Read `docs/features/README.md` if it exists.
2. Identify the feature id from the request, code module, route, domain name, or existing docs.
3. Read `docs/features/<feature-id>/README.md` if it exists.
4. If no track exists, create one and add it to the index.

During work, update the feature track when behavior, decisions, endpoints, data models, dependencies, rollout constraints, tests, or source-of-truth links change.

Before completion, update the feature track and run:

```bash
python3 cli/feature_track.py validate --root .
```

Feature Track spec: `spec/feature-track-spec.md`

---
> Source: [JunsW/feature-track](https://github.com/JunsW/feature-track) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
