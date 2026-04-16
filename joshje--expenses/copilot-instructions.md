## expenses

> Match receipt images to bank statement transactions and output a CSV.

# Expense Matching Assistant

Match receipt images to bank statement transactions and output a CSV.

## Folder Structure

```
input/
  statement/    # One bank statement screenshot (Zopa, GBP)
  receipts/     # One or more receipt images
projects/
  {project-name}/
    statement/
    receipts/
    expenses.csv
```

## Workflow

### Detecting Mode

- If `./input/` contains files → process new work
- If only a project name is mentioned → update that existing project

### Collecting Inputs

1. The user will share the bank statement screenshot directly in the chat
2. Save the statement image to `./input/statement/`
3. Run `open ./input/receipts` to open Finder so the user can drag in their receipt images
4. Wait for the user to confirm receipts are added before proceeding

### Creating a New Project

1. Examine input files and infer a project name from context (trip destination, date range, merchant city)
   - Example: `amsterdam-jan-2025`, `client-visit-march-2025`
2. If a confident name cannot be inferred, ask the user
3. Create `./projects/{project-name}/` with `statement/` and `receipts/` subdirectories
4. Move all files from `./input/statement/` and `./input/receipts/` into the project
5. Process and write `expenses.csv`

### Updating an Existing Project

1. Identify the target project (from context or ask if ambiguous)
2. Move new input files into the existing project folders
   - New statement replaces the old one
   - New receipts are added alongside existing ones
3. Re-process all receipts against the statement
4. Overwrite `expenses.csv` with the full updated table

## Processing

For each receipt in the project's `receipts/` folder:

1. Extract: merchant name, date, amount, currency
2. Find matching transaction on the bank statement by merchant and date proximity
3. Use the GBP amount from the statement — never estimate or convert
4. If two statement lines could match, list both candidates and mark confidence as `Low`
5. If no match is found, flag it explicitly

## Output

Write `expenses.csv` with columns:

```
#,File,Merchant,Receipt Date,Receipt Amount,Currency,GBP Amount,Match Confidence,Notes
```

The `File` column contains just the filename (no path) for easy matching.

Match Confidence values: `High` / `Low` / `No Match`

Also print the table to stdout as markdown, followed by a total GBP sum of matched items.

After the summary, provide the Deel expense submission link:
https://app.deel.com/expenses-claims/my-expenses

## Rules

- Never estimate or convert currency — GBP figures come from the statement only
- Always confirm project name before creating if it cannot be inferred
- `./input/` must be empty after processing — all files get moved to the project

## Git Branch Strategy

Two branches with different remotes:

| Branch | Remote | Repo | Purpose |
|--------|--------|------|---------|
| `main` | `origin` | `joshje/expenses` (public) | Code, README, CLAUDE.md — no project data |
| `private` | `private` | `joshje/expenses-private` (private) | Everything including project files |

### Workflow

- **README.md, CLAUDE.md, code changes**: Commit on `main`, push to origin, then rebase `private` onto `main` and push to private
- **Project files** (anything in `projects/`): Switch to `private` branch first, commit there only
- The `.gitignore` on `main` excludes `projects/` to prevent accidental exposure

### Example: Committing shared files

```bash
# On main branch
git add README.md CLAUDE.md
git commit -m "Update docs"
git push origin main

# Update private branch
git checkout private
git rebase main
git push -f private private
```

### Example: Committing project files

```bash
git checkout private
git add projects/
git commit -m "Add expense project"
git push private private
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joshje) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
