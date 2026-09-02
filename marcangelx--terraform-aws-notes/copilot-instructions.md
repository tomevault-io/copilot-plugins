## terraform-aws-notes

> |


## Project Structure
- `infra/`      — Terraform: Lambda + API Gateway + DynamoDB
- `backend/src/` — Lambda handler (Node.js): notes CRUD via DynamoDB
- `frontend/src/` — Angular 17 standalone app with signals

## API
| Method | Path          | Action       |
|--------|---------------|--------------|
| GET    | /notes        | List notes   |
| POST   | /notes        | Create note  |
| GET    | /notes/{id}   | Get note     |
| DELETE | /notes/{id}   | Delete note  |

## Angular Signals Pattern
```typescript
// notes.service.ts
private _notes = signal<Note[]>([]);
readonly notes = this._notes.asReadonly();
readonly count = computed(() => this._notes().length);
```
All state lives in `NotesService`. Components inject and read signals directly in templates via `service.notes()`.

## Operational Rules
- Always run `terraform init` before plan/apply.
- Never apply without showing a plan summary first.
- After changing backend code: `npm install --prefix backend/src && terraform apply`
- After changing frontend: `ng serve` picks up changes automatically.

## Quick Reference
```bash
# Deploy backend
cd infra && npm install --prefix ../backend/src
terraform init && terraform apply -auto-approve
terraform output api_url   # paste into frontend/src/environments/environment.ts

# Run frontend
cd frontend && npm install && ng serve
```

## Safety
- Do not run `terraform destroy` without explicit confirmation.
- DynamoDB has point-in-time recovery enabled.

---
> Source: [marcangelx/terraform-aws-notes](https://github.com/marcangelx/terraform-aws-notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
