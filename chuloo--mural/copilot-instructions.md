## mural

> When changing Mural's UI or native apps, check whether the experience depends on server changes: API contracts, error responses, prompts, capabilities, configuration, migrations or runtime permissions.

# Mural agent instructions

## Ship UI and server changes together

When changing Mural's UI or native apps, check whether the experience depends on server changes: API contracts, error responses, prompts, capabilities, configuration, migrations or runtime permissions.

William requested on September 16, 2026 that required server deployment be part of delivering an authorized app/UI release. Deploy the matching tested server changes before reporting that release work is complete. A merged backend PR or published APK does not establish that production has the required behavior. If deployment is unnecessary, say why; if blocked, report the exact remaining step.

- Inspect the actual production revision and deployment wrapper. Preserve private configuration, enabled features, credentials and retained data. The backend README includes historical activation instructions; confirm current production settings before using them.
- Test the affected server behavior and app/server compatibility. Highlight every new UI/UX change before proposing or performing a merge.
- Before deployment, check active calls, retain the previous image and configuration, and take the existing encrypted backup. Apply only required migrations and runtime grants. Avoid interrupting active conversations.
- Deploy in a compatible order. Do not enable a client feature before its required server behavior is available.
- Verify the deployed source/image, public health and database readiness, affected endpoint contracts, and sanitized logs. Use non-billable checks unless a live provider call or purchase is separately authorized.
- Record the deployed revision, verification results, rollback location and any remaining limits. Distinguish local tests, live verification, merging and deployment in the handover.

Use the user's current instructions to resolve release scope and deferred work. Do not ask again for deployment approval already provided for that scope. This workflow does not authorize unrelated feature activation, pricing changes or destructive data operations.

---
> Source: [Chuloo/mural](https://github.com/Chuloo/mural) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
