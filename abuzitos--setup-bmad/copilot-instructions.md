## setup-bmad

> Apply when responding to a production incident. Guides through detection, diagnosis, remediation, hotfix, and postmortem documentation.


# Incident Postmortem

When responding to a production incident:

## Detection and declaration
1. Confirm the incident is real (not a false positive)
2. Declare severity: P1 (service down), P2 (degraded), P3 (minor impact)
3. Open incident channel and notify stakeholders per severity
4. Record the incident start time

## Diagnosis
1. Identify the symptom: what is failing? (logs, metrics, traces)
2. Isolate scope: which service, endpoint, percentage of users?
3. Correlate with recent changes: deploys, config changes, traffic spikes
4. Formulate 2-5 root cause hypotheses, ranked by probability
5. Test each hypothesis with evidence

## Immediate remediation
1. Apply quick fix: rollback, feature flag, traffic rerouting
2. Document the remediation in the incident channel
3. Monitor for 5+ minutes after remediation to confirm stability
4. Update stakeholders: "Incident mitigated, investigation ongoing"

## Hotfix (if needed)
1. Create branch: `hotfix/{{incident_id}}`
2. Write tests that reproduce the bug (must fail before fix, pass after)
3. Review with focus on Blockers using `skills/analysis/review-pr/SKILL.md`
4. Deploy to staging then production with gradual rollout
5. Monitor 30 minutes after production deploy

## Postmortem (within 48 hours)
1. Create `docs/postmortems/{{incident_id}}.md`
2. Fill complete timeline: detection → resolution
3. Identify root cause (not symptom) and contributing factors
4. List concrete action items with owners and deadlines
5. If architectural decision needed: use `skills/docs/write-adr/SKILL.md`

**Rule**: postmortems are blameless — the system failed, not people.

## Agents involved
- Incident Commander: `agents/ops/code-reviewer.md`
- Engineer: `agents/dev/senior-engineer.md`
- QA: `agents/qa/test-engineer.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abuzitos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
