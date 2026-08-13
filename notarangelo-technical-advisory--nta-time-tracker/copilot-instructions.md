## nta-time-tracker

> Commit and push **directly to `main`** — no feature branches or PRs for routine work.

# Project Rules

## Git Workflow

Commit and push **directly to `main`** — no feature branches or PRs for routine work.

1. Use conventional commit messages. Husky/commitlint enforces the type: one of `feat`, `fix`, `perf`, `docs`, `refactor`, `chore`, `style`, `test`. **`ci:` is not allowed — use `chore:` for CI/workflow changes.**

2. The CI/CD pipeline derives the semver bump from the commit-type prefix:
   - `feat:` → minor
   - `fix:` / `perf:` / others → patch
   - `BREAKING CHANGE` or `[major]` → major

3. **The push to `main` triggers the deploy** via `.github/workflows/deploy-and-release.yml` (version → build → deploy → release notes → GitHub Release). There are **no path filters**, so every push to main — including docs-only commits — runs the full pipeline and deploys.

## Firebase Deployment

### Automatic Deployment (Preferred)

The app automatically deploys to Firebase when code is pushed to the `main` branch via the GitHub Actions workflow (`.github/workflows/deploy-and-release.yml`).

**What gets deployed** (`firebase deploy --only hosting,firestore,functions`):

- **Hosting**: Angular app build to `https://fta-invoice-tracking.web.app`
- **Firestore Rules & Indexes**: from `firestore.rules` and `firestore.indexes.json`
- **Cloud Functions**: from `functions/` (e.g. `generateStatusReport`)

**Authentication**: Workload Identity Federation via `google-github-actions/auth@v2` — `workload_identity_provider` from the `WIF_PROVIDER` secret, impersonating service account `firebase-adminsdk-fbsvc@fta-invoice-tracking.iam.gserviceaccount.com`. There is **no** stored service-account JSON secret. CI pins `firebase-tools@15.6.0`.

### Manual Deployment (When Needed)

Use manual deployment to deploy changes without creating a new release (e.g., Firestore rules hotfix).

**Prerequisites:**

1. Authenticate with Google Cloud:

   ```bash
   gcloud auth application-default login
   ```

2. Set the quota project:

   ```bash
   gcloud auth application-default set-quota-project fta-invoice-tracking
   ```

**Commands:**

```bash
# Deploy everything (hosting + firestore rules/indexes)
npm run deploy

# Deploy only hosting
npm run deploy:hosting

# Deploy only Firestore rules and indexes
npm run deploy:firestore

# Deploy via non-interactive script
node deploy-non-interactive.js
```

### Troubleshooting

**Problem:** `Failed to authenticate` or `quota project` error

**Solution:**

```bash
gcloud auth application-default login
gcloud auth application-default set-quota-project fta-invoice-tracking
```

**Verify deployment:**

- Open [Firebase Console → Firestore → Rules](https://console.firebase.google.com/project/fta-invoice-tracking/firestore/rules)
- Check the timestamp to confirm rules were updated
- For hosting, visit `https://fta-invoice-tracking.web.app` and hard-refresh (Cmd+Shift+R)

## Project Notes

- Angular app deployed to Firebase Hosting via GitHub Actions.
- CI/CD pipeline: `.github/workflows/deploy-and-release.yml` triggers on push to main.
- The pipeline auto-versions (semver), builds, deploys to Firebase (hosting + firestore), generates AI release notes, and creates a GitHub Release.

## Domain Invariants

### Status Reports — zero-activity sections are orphans

Report sections (`StatusReportSection`) are keyed by `projectName`. Activities always come from the reporting period's time entries grouped by project, while outcomes persist across reports via the per-customer+project `OutcomeRecord` collection (the cumulative "living" record fed to the model as prior outcomes).

**Invariant: a section with no activities is an orphan, not real content.** It appears when a prior `OutcomeRecord` has no matching time entries this period — most often after a project rename leaves a stale record under the old name. The model then emits that record as an activity-less section that duplicates outcomes already carried into the active section, and `upsertOutcomes` re-persists it, so it recurs every report.

Zero-activity sections are therefore filtered out in three places — **do not "restore" them**:

- `functions/src/index.ts` — `generateStatusReport` drops them before returning (stops new reports saving/re-persisting the orphan)
- `src/app/components/status-reports/status-report-detail.component.ts` — page render (`*ngIf` inside `*ngFor`, preserving the true section index for inline edits) **and** both the PDF and DOCX export loops

If outcomes for a renamed project must survive, merge the stale `OutcomeRecord` into the active project's record and delete the old one — never reintroduce an activity-less section.

---
> Source: [Notarangelo-Technical-Advisory/nta-time-tracker](https://github.com/Notarangelo-Technical-Advisory/nta-time-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
