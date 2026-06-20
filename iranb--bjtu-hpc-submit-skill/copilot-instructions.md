## bjtu-hpc-submit-skill

> Use when an agent needs to refresh/save BJTU HPC portal auth, add or switch BJTU portal accounts/tokens, run a local transfer dashboard, upload/download files, reuse shared datasets across accounts, copy account-local runtime environments, schedule two execution-slot experiments plus two queued follow-up experiments per saved account, submit CPU/GPU jobs, inspect job status including GPU counts, monitor resumable dataset uploads, or probe the runtime environment from a BJTU HPC helper workspace.


# BJTU HPC Submit

Tool-first workflow for BJTU HPC portal work from a local helper workspace. This public version is sanitized: replace placeholder paths, account names, and project directories with your own local values before use.

## Runtime Defaults

- Work from the helper workspace unless the project says otherwise:

```bash
PY=/path/to/python3
SLURM_DIR="/path/to/bjtu-hpc-helper"
PROJECT_DIR="/path/to/your/project"
```

- Prefer the same Python environment used to install the helper dependencies. If dependencies are missing, install the helper requirements and Playwright Chromium:

```bash
cd "$SLURM_DIR"
"$PY" -m pip install -r requirements.txt
"$PY" -m playwright install chromium
```

- When working from a project, save portal and Slurm evidence under a project-local log directory such as `$PROJECT_DIR/hpc_stdout/`.
- Use broad keywords for general queue checks. Use narrow keywords only for targeted follow-ups.

## Entry Points

- Start with `hpc_doctor.py --json`; it checks dependencies, account state, browser profile, and token validity without printing secrets.
- For a local GUI, run `hpc_transfer_web.py` from the helper workspace and open the reported localhost URL.
- Upload and download through helper wrappers such as `hpc_upload.py` and `hpc_download.py`; include `--auth-account <name>` when scripts support it.
- Use portal-app submit wrappers only for resource-shape-compatible jobs or lightweight probes. Prefer verified wrappers over raw submit scripts when a portal app is used.
- For CPU/GRES-sensitive jobs, use uploaded native `sbatch` scripts through the portal SSH path instead of portal-generated PyTorch app scripts, then verify native Slurm allocation.
- For MCP clients, prefer tools that expose auth status, submit-and-verify, pending reason, allocation verification, stdout tailing, and SFTP info.

Useful status commands:

```bash
cd "$PROJECT_DIR"
"$PY" "$SLURM_DIR/hpc_jobs.py" list --keyword <keyword> --size 30 --paths
"$PY" "$SLURM_DIR/hpc_jobs.py" list --keyword <keyword> --size 30 --paths --json > hpc_stdout/bjtu_jobs_YYYYMMDD_HHMM.json
"$PY" "$SLURM_DIR/hpc_pending_reason.py" <slurm_job_id> --no-sinfo
```

## Auth

- Saved accounts usually live in `~/.bjtu_hpc_accounts.json`; a legacy single-token cache may live in `~/.bjtu_hpc_token`.
- Treat the saved account store as the source of truth; the legacy file is only a compatibility cache for older scripts.
- Select accounts with `--auth-account <name>` or `HPC_AUTH_ACCOUNT=<name>`.
- Never print portal tokens, cookies, temporary certificates, passwords, or captured browser storage.
- Treat portal codes `11009`, `11011`, and `11012` as expired or invalid auth.
- Treat portal HTTP `401`, token-validation transport errors, and missing profile tokens as auth-blocked for user-requested live status until a fresh validation succeeds. Stale snapshots may be reported only as `last trusted`.
- If the user explicitly asks for a captcha/verification-code-only login flow, save CAS login credentials only through the local helper, with user-specific values supplied at runtime:

```bash
cd "$SLURM_DIR"
"$PY" hpc_credentials.py set <account_name> --login-name <portal_user>
"$PY" hpc_credentials.py list
```

  The helper should store credentials only on the controller machine with restrictive file permissions. Never commit credentials, passwords, browser storage, portal tokens, or temporary certificates to Git.
- Auth refresh is not an experiment launch. If an expired token blocks a user-requested BJTU status, progress, upload, download, pending-reason, or submit check, run the integrated visible refresh flow immediately unless the user explicitly forbids token refresh or browser use.

### Multi-Account Tokens

Use `hpc_accounts.py` for account-local tokens instead of copying a legacy token file between users:

```bash
cd "$SLURM_DIR"
"$PY" hpc_accounts.py list
"$PY" hpc_accounts.py add <account_name> --refresh --browser playwright --fresh-page --timeout 600
"$PY" hpc_accounts.py refresh <account_name> --browser playwright --headless --fresh-page
"$PY" hpc_accounts.py validate <account_name>
"$PY" hpc_accounts.py use <account_name>
```

- Adding or refreshing an account should discover the portal user, cluster, and cluster OS account from the portal token when the helper supports it. Do not copy metadata from another saved account unless the user explicitly provides it.
- Do not sync a secondary account into `~/.bjtu_hpc_token` unless the user intentionally wants to change the legacy default.
- Use `--auth-account <account_name>` on every submit, job-list, upload, download, or proxy-info command in a multi-account workflow.

### Auth Recovery State Machine

Use one integrated `hpc_refresh_flow.py` command that owns validation, profile probing, optional visible login, and post-login status collection. Do not manually bounce between doctor, job-list, and visible browser attempts unless the integrated command has exited and validation still fails.

1. For routine refreshes when no command is currently blocked:

```bash
cd "$SLURM_DIR" && "$PY" hpc_refresh_flow.py <account_name>
```

2. If invalid auth blocks a user-requested status check, progress check, pending-reason check, upload, or submit, run the integrated blocked-task flow in a PTY and keep it running:

```bash
cd "$SLURM_DIR" && "$PY" hpc_refresh_flow.py <account_name> --visible-only
```

3. For project progress checks, use the post-login status variant so the same command continues after any refresh or login:

```bash
cd "$PROJECT_DIR"
"$PY" "$SLURM_DIR/hpc_refresh_flow.py" <account_name> --visible-only \
  --after-jobs-keyword <keyword> --after-jobs-size 30 --after-jobs-paths \
  --after-snapshot-dir "$PROJECT_DIR/hpc_stdout" \
  --after-pending-job <slurm_job_id> --after-pending-no-sinfo
```

Interpret the integrated command by its output:

- `validate saved token ... ok`: token was already usable. Continue; do not open a browser.
- `refreshed ... headlessly` or `from the existing Playwright profile`: profile recovery succeeded. Continue; do not ask the user to log in.
- `[action] A Playwright Chromium window should open now`: only now ask the user to finish CAS/captcha, wait for the HPC portal home page to load, then close the Playwright window.
- A Playwright/Chromium window that opens and closes almost immediately after a recent successful login is usually normal profile validation. Keep the command running and wait for `[ok]`, a post-login job table, or an explicit validation error.

Operational rules:

- Run the refresh command in a PTY and keep it running while the user logs in.
- `--visible-only` does not blindly open a browser. It first validates the saved account token and briefly probes the selected Playwright profile.
- If there is no stdout for about 30 seconds, check whether `Google Chrome for Testing` or `hpc_refresh_flow` is running:

```bash
pgrep -afil "Google Chrome for Testing|playwright|hpc_refresh_flow"
```

- Do not screenshot or inspect login pages because they may contain account, CAPTCHA, or token material.
- If the command exits with `timed out waiting for token in visible browser`, first run `hpc_accounts.py validate <account_name>`. If validation succeeds, continue; if it still fails, rerun the integrated `--visible-only` flow once.
- Use `--force --visible-only --no-profile-probe-before-visible` only after one integrated attempt exits without a usable token and validation still fails, or when the user explicitly requests a visible login window. Do not use this as the first attempt.
- If the second visible attempt still fails to save a usable token, report the auth/token-save failure as the blocker and keep live status at the latest trusted snapshot.
- If a visible Playwright window was closed by the user but the command appears stuck, poll the PTY and process list, then validate the same account or run a headless profile refresh before opening another visible browser. A closed browser window is not by itself proof that token extraction failed.
- If a visible-browser timeout occurs, first try a headless refresh from the same selected account profile, then validate the account. The user may already have completed CAS login and left a usable token in the browser profile.

## Job Rules

- Target single-process GPU shape on `cluster2` for native Slurm:

```text
--gpu 1 --ntasks 1 --cpus-per-task 16 --gres-flags disable-binding
```

- For normal GPU training submissions through native Slurm, force `--gres-flags disable-binding` and start with `16` CPU cores per training task. If `sbatch --test-only` or scheduler constraints reject `16`, retry with `12`, then `8`; treat `8` as the minimum for evidence-producing GPU training. Do not request more than `16` CPU cores per task unless the user explicitly asks for a diagnostic probe or a high-CPU override.
- Native Slurm equivalent for one GPU:

```bash
#SBATCH --partition=GPU
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --gres=gpu:1
#SBATCH --gres-flags=disable-binding
```

- Portal PyTorch/GPU app templates may accept CPU/GRES fields in the local submit payload but omit those directives from the generated Slurm script. Treat portal-app CPU/GRES fields as advisory until native Slurm proves otherwise.
- Do not use portal-app GPU submissions for evidence-producing training when CPU/GRES shape matters. Use native `sbatch`, then verify the real Slurm job id with `scontrol show job <job_id>`.
- Request more GPUs only when the code actually uses them.
- Avoid `--gpu 1 --ntasks 8` without `--gres-flags disable-binding`; it can produce `BadConstraints`.
- After every submit, verify the portal job row. If the job is `PENDING`, report the native Slurm `Reason`, not just portal state.
- If CPU/GRES shape matters, verify native `NumCPUs`, `NumTasks`, `CPUs/Task`, and GPU TRES with `scontrol`; portal request fields are not enough.
- If a supposed GPU training launch reports `NumCPUs=1`, `CPUs/Task=1`, and `gres/gpu=1`, classify it as wrong-shape immediately. Do not count it as a valid evidence-producing run unless the user explicitly accepts the degraded allocation.
- Verified portal submit wrappers must resolve the real Slurm job id from either the immediate `job` row or the delayed `wait.job` row when `--wait` is used. If no Slurm id is found, or native allocation mismatches the requested CPU/GRES shape, mark the launch failed even when the portal API returned success.
- Do not cancel unrelated jobs. For per-user job-count limits, inspect existing jobs before canceling anything.
- Always run `sbatch --test-only` for a new native script or a new resource shape before real submission.

Known-good shapes on `cluster2`:

```text
1 GPU single process:  --ntasks=1 --cpus-per-task=16 --gres=gpu:1 --gres-flags=disable-binding
1 GPU minimum fallback: --ntasks=1 --cpus-per-task=8  --gres=gpu:1 --gres-flags=disable-binding
2 GPU packed job:      --ntasks=2 --cpus-per-task=16 --gres=gpu:2 --gres-flags=disable-binding
```

## Native Slurm Packed Jobs

Use packed jobs only when one Slurm allocation intentionally launches multiple child experiments.

Checklist:

1. Request one batch allocation with the required GPU count, `--gres-flags=disable-binding`, and enough CPU for all child experiments. For two single-GPU children, start with `--gres=gpu:2`, `--ntasks=2`, and `--cpus-per-task=16`, which gives 16 CPU cores per child. If rejected, retry with `--cpus-per-task=12`, then `8`; do not go below 8 CPU cores per child.
2. In the batch script, read allocation-provided `CUDA_VISIBLE_DEVICES` and split it into child lanes. Do not hardcode physical `0/1`.
3. For each child, set `CUDA_VISIBLE_DEVICES` to exactly one allocated id, run lightweight `nvidia-smi` and `torch.cuda.device_count()` checks, then launch the experiment.
4. Save a batch stdout plus one child log per lane.
5. After submission, run native checks:

```bash
cd "$PROJECT_DIR"
"$PY" "$SLURM_DIR/hpc_pending_reason.py" <job_id> --no-sinfo
```

6. Verify `JobState=RUNNING`, `Reason=None`, CPU fields, GPU TRES, and node name.
7. Tail child logs and verify that each child reports one visible CUDA device and has entered real training before calling the launch successful.

## Account Scheduling

For experiment batches, treat each saved portal account as having two execution slots and two queued follow-up slots:

```text
per account: 2 run-slot experiments + 2 queued follow-up experiments
```

- A run-slot experiment is one of the first two non-terminal experiments assigned to an account and intended to run as soon as resources permit.
- A queued follow-up experiment is an additional submitted experiment kept as backlog so the scheduler can start it after earlier work finishes.
- `DONE`, `FAILED`, and `CANCELLED` jobs no longer count toward either slot type.
- Before launching a batch, list saved accounts and inspect current jobs for each candidate account:

```bash
cd "$SLURM_DIR"
"$PY" hpc_accounts.py list
"$PY" hpc_jobs.py list --auth-account <account_a> --scope current --size 50 --paths
"$PY" hpc_jobs.py list --auth-account <account_b> --scope current --size 50 --paths
```

- Fill each account's two run slots first, then allow up to two queued follow-ups for that same account. Do not submit a fifth non-terminal experiment under the same account unless the user explicitly overrides the cap.
- Submit each CPU/GRES-sensitive GPU training job as a native Slurm script with explicit project and Python paths, `--gres-flags=disable-binding`, `--ntasks=1`, and `--cpus-per-task=16` first. If `16` is rejected, retry with `12`, then the minimum `8`. Make the job name encode the experiment and slot.
- Distinguish submit limits from run limits. A third job may be accepted by `sbatch` but remain pending because the user's current run limit is full. Native pending reason `QOSMaxJobsPerUserLimit` usually means a running-job cap, not necessarily a submit cap.
- To test whether another submit would be accepted without starting work or touching existing jobs, use a unique held native probe and cancel it immediately:

```bash
sbatch --test-only <probe>.sbatch
PROBE_JOB_ID=$(sbatch --hold --parsable <probe>.sbatch)
squeue -j "$PROBE_JOB_ID"
scancel "$PROBE_JOB_ID"
squeue -j "$PROBE_JOB_ID"
```

  The probe script should request the same partition/resource shape being tested, use a very short time limit, and have a unique name such as `submit-cap-probe-YYYYMMDDHHMMSS`. A held probe should show `JobHeldUser` before cancellation and disappear from `squeue` after cancellation. Never cancel or modify unrelated jobs during this check.

Example `exp-a-account-a-slot1.sbatch`:

```bash
#!/bin/bash
#SBATCH --job-name=exp-a-account-a-slot1
#SBATCH --partition=GPU
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --gres=gpu:1
#SBATCH --gres-flags=disable-binding
#SBATCH --output=logs/%x-%j.out
#SBATCH --error=logs/%x-%j.out

set -euo pipefail
cd /path/to/your/project
exec /path/to/python3 train_exp_a.py
```

Submit and verify natively:

```bash
sbatch --test-only exp-a-account-a-slot1.sbatch
JOB_ID=$(sbatch --parsable exp-a-account-a-slot1.sbatch)
scontrol show job "$JOB_ID"
```

- Portal app submission remains acceptable for compatibility probes and non-critical resource shapes, but it must be followed by native allocation verification:

```bash
"$PY" hpc_submit_verified.py ./gpu_probe.py --auth-account <account_a> --app gpu --gpu 1 \
  --ntasks 1 --cpus-per-task 16 --gres-flags disable-binding \
  --job-name gpu-compat-probe --submit --wait
```

- If strict "start after the previous experiment finishes" ordering is required, use native Slurm dependencies such as `--dependency=afterany:<job_id>` through the SSH/native `sbatch` path. Plain portal submissions may become runnable immediately if scheduler and QOS limits allow them.
- If `QOSMaxJobsPerUserLimit` blocks two single-GPU run slots for one account, use one native packed job with `--gres=gpu:2 --gres-flags=disable-binding` and start at `16` CPU cores per child experiment; fall back to `12`, then the minimum `8`, only if the larger shapes are rejected. Pack only two experiments per account unless the user explicitly approves more.
- If queued follow-up submissions hit a submit cap such as `QOSMaxSubmitJobPerUserLimit`, record them in the local launch plan and submit when a run slot clears instead of retrying in a loop.

## Paths

- Portal SSH/SFTP should go through the helper's SFTP-info command. Do not hardcode one-time certificate tokens.
- Portal SSH uses a temporary certificate token, not the local SSH key.
- Reusable code can live under a user-controlled cluster path such as `$BJTU_HOME/code/<project>`.
- Portal job work/output directories usually live under a user-controlled cluster jobs directory.
- Trust job-side probes for runtime facts, not login-node inference.

Evidence examples:

- Portal snapshots: `$PROJECT_DIR/hpc_stdout/bjtu_jobs_YYYYMMDD_HHMM*.json`
- Native Slurm snapshots: `$PROJECT_DIR/hpc_stdout/bjtu_pending_reason_YYYYMMDD_HHMMSS*.json`
- Downloaded launch logs: `$PROJECT_DIR/hpc_stdout/bjtu_<jobid>_<shortname>_launch_YYYYMMDD.log`

Download pattern:

```bash
"$PY" "$SLURM_DIR/hpc_download.py" "<remote_log_path>" -o "$PROJECT_DIR/hpc_stdout/bjtu_<jobid>_<shortname>_launch_YYYYMMDD.log" --no-progress
```

## Dataset Upload

- Prefer resumable, chunked uploads for large datasets.
- Never run two upload workers writing the same `.part` file.
- When a source host is slow or unreliable, use cluster-side file size/progress as the source of truth.

## Dataset Reuse

Reuse an existing cluster dataset across accounts instead of uploading another copy when filesystem permissions can safely allow it.

- Treat the portal Web "file share" UI as portal-managed share metadata, not as a reliable general-purpose way to expose arbitrary existing dataset paths to another cluster account.
- Observed frontend routes may include:

```text
GET  /pcp/clusters/{cluster}/file/share/list
POST /pcp/clusters/{cluster}/file/share
GET  /pcp/clusters/{cluster}/file/share/cancel?id=...
```

- If the helper has `hpc_share_check.py`, first inspect the source account, dataset root, and target cluster OS user in dry-run mode:

```bash
cd "$SLURM_DIR"
"$PY" hpc_share_check.py \
  --auth-account <source_auth_account> \
  --data-root /data/home/<source_cluster_user>/dataset/<dataset_name> \
  --target-user <target_cluster_user>
```

- Prefer the minimum permission that works. If the dataset subtree is already readable/executable by group or other users and only the source home directory blocks traversal, grant the target user execute-only traversal on the source home directory. If the dataset subtree blocks reads, apply read-only ACLs only after confirming the exact source path and target user.
- Always verify as the target account before launching real training. A direct proxy SSH read test is sufficient for filesystem access; a small CPU job-side probe is better when queue time is acceptable.
- After access is verified, optionally create a target-home symlink so training configs can use an account-local-looking path:

```bash
mkdir -p /data/home/<target_cluster_user>/dataset
ln -sfn /data/home/<source_cluster_user>/dataset/<dataset_name> \
  /data/home/<target_cluster_user>/dataset/<dataset_name>
```

- Use real cluster OS account names for filesystem permissions and paths. Portal usernames may differ from cluster OS users.

## Account-Local Environments

Shared datasets may cross account boundaries through ACLs or symlinks, but Python and conda runtime environments should live under the account that runs the job.

- Do not launch a target account's jobs with another account's Python, conda, cache, code, or output path.
- Copy or rebuild the environment under the target cluster OS account home, for example `/data/home/<target_cluster_user>/envs/<env_name>`.
- When cloning an existing conda environment across accounts, run the clone as the target cluster OS user and force real file copies with `--copy`; default conda clones may use hardlinks:

```bash
SRC=/data/home/<source_cluster_user>/envs/<env_name>
DST=/data/home/<target_cluster_user>/envs/<env_name>
CONDA=/data/home/<source_cluster_user>/software/miniconda3/bin/conda
mkdir -p /data/home/<target_cluster_user>/envs
"$CONDA" create --copy -y -p "$DST" --clone "$SRC"
```

- Verify owner, executable path, package imports, and sample inodes before using the copied environment. Login-node `torch.cuda.is_available()` may be false; use a GPU job-side probe when CUDA runtime availability matters.

## Post-Submit Evidence Checklist

Before reporting a job as running:

- Native Slurm job id is known. If using a portal app, the portal row is also recorded.
- Native Slurm reason was checked.
- CPU/GPU allocation matches the intended shape.
- Startup logs were downloaded or tailed locally.
- At least one real training/progress line was observed, not only environment setup.
- Evidence files were saved under the project log directory.

## Safety

- For cross-account dataset sharing, inspect ACLs first; do not apply ACL/chmod changes without explicit confirmation.
- Multi-account launches must keep account-local code, outputs, and environments under the corresponding cluster OS home. Shared datasets can cross accounts by ACL or symlink, but runtime paths should not cross accounts.
- For experiment batches, cap each saved auth account at two run-slot experiments plus two queued follow-up experiments unless the user explicitly overrides the cap.
- For CPU/GRES-sensitive GPU training, use native Slurm, force `--gres-flags=disable-binding`, and try `16` CPU cores per training task first. Fall back to `12`, then `8`, only if needed; do not go below `8` unless the user explicitly requests a non-training diagnostic probe.
- Do not rely on portal PyTorch/GPU app templates to enforce `--cpus-per-task` or `--gres-flags`; verify with native Slurm or treat the resource shape as untrusted.
- Do not publish tokens, cookies, passwords, one-time certificate strings, local absolute paths, student ids, or project-specific job evidence.

---
> Source: [Iranb/bjtu-hpc-submit-skill](https://github.com/Iranb/bjtu-hpc-submit-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
