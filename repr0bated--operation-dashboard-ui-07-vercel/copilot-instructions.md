## always-load-core-skills

> Always load core agent skills (gRPC, OVS, Rust, backend architecture) before related work


# Always load core skills

At the start of work in this environment (especially `operation-dashboard-ui-07` and `odbus`), and whenever the task involves gRPC, protobuf, tonic, PluginService, StateSync, OVS, OVSDB, OpenFlow, bridge networking, Rust, or backend architecture:

1. Immediately **Read** these files (do not skip):
   - `/home/jeremy/.agents/skills/grpc-expert/SKILL.md`
   - `/home/jeremy/.agents/skills/grpc-protocol-expert/SKILL.md`
   - `/home/jeremy/.agents/skills/ovs-db-analysis/SKILL.md`
   - `/srv/git/odbus/.factory/droids/rust-pro.md`
   - `/srv/git/odbus/.factory/droids/backend-architect.md`
2. Follow those skills before investigating or editing.

Do not modify the skill files. Prefer `grpc-expert` for OP-DBUS PluginSchema / seal-freeze-hot bridge work; use `grpc-protocol-expert` for generic gRPC outside that pipeline; use `ovs-db-analysis` for sosreport/OVS/OpenFlow analysis; use `rust-pro` for Rust implementation; use `backend-architect` for API/service/schema architecture.

Note: `rust-pro` and `backend-architect` are Factory droid definitions under `.factory/droids/` — there is no `**/SKILL.md` package for either under `.agents/skills/`, `.claude/skills/`, `.cursor/skills-cursor/`, or `.codex/skills/`.

---
> Source: [repr0bated/operation-dashboard-ui-07-vercel](https://github.com/repr0bated/operation-dashboard-ui-07-vercel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
