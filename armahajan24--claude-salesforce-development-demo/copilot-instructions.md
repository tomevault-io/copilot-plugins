## claude-salesforce-development-demo

> Sample Salesforce DX project demonstrating a Claude Code + Salesforce development

# claude-salesforce-development-demo — Project Instructions

Sample Salesforce DX project demonstrating a Claude Code + Salesforce development
workflow: repository-level instructions, specialized sub-agents, and a
Service Request management app used as the running example. All data, object
names, and business logic in this repository are synthetic and were created
specifically for this demo — nothing here is copied from a real project.

## Org & connection

- No org alias is pre-configured in this repository. Before running any
  `sf` command that targets an org, authenticate your own scratch org or
  Developer Edition org and pass `--target-org <your-alias>` (see
  [docs/DEPLOYMENT-GUIDE.md](docs/DEPLOYMENT-GUIDE.md)).
- Never hardcode a username, org alias, org Id, or login URL in source files,
  scripts, or documentation in this repository.
- This is a public repository. Do not connect it to any org that contains
  real customer, employee, or business data.

## Tooling

- Salesforce CLI `sf` (see [docs/DEPLOYMENT-GUIDE.md](docs/DEPLOYMENT-GUIDE.md) for the
  minimum supported version).
- `@salesforce/sfdx-scanner` for static analysis (PMD + retire.js).
- ESLint, Prettier, Husky pre-commit hook, `sfdx-lwc-jest` for LWC unit tests.
- API version: see `sfdx-project.json`.

## Metadata layout — `force-app/main/default/`

- `objects/Service_Request__c/` — the sample custom object (fields: Request
  Type, Priority, Status, Description, Resolution, Requested Completion
  Date, Escalated).
- `classes/` — `ServiceRequestSelector` (SOQL), `ServiceRequestService`
  (business logic + DML), `ServiceRequestTriggerHandler` (thin routing),
  `ServiceRequestController` (`@AuraEnabled` entry points for the LWC),
  `ServiceRequestTestFactory` (synthetic test data), and matching test classes.
- `triggers/ServiceRequestTrigger.trigger` — routes to the handler only; no
  logic belongs here.
- `lwc/serviceRequestPanel/` — displays and updates open requests.
- `flows/Service_Request_Escalation_Notification.flow-meta.xml` — creates a
  follow-up Task when a request becomes escalated.
- `permissionsets/Service_Request_User.permissionset-meta.xml` — assign this
  to grant access; do not grant object/field access via profiles.

## Architecture rules (see [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) and

[docs/CODE-REVIEW-STANDARDS.md](docs/CODE-REVIEW-STANDARDS.md) for full detail)

- **Selector classes own SOQL.** Never write inline SOQL against
  `Service_Request__c` outside `ServiceRequestSelector`.
- **Service classes own DML and business rules.** Trigger handlers and
  controllers delegate to the service layer; they do not contain business
  logic themselves.
- **Trigger handlers are thin.** One method per trigger context, no logic
  beyond calling the service layer.
- No SOQL or DML inside loops, anywhere.
- Every public method that isn't a `@IsTest` method has an ApexDoc comment
  (`@param`/`@return`/`@throws` as applicable).
- Reuse existing selector/service methods before adding new ones — check
  `ServiceRequestSelector` and `ServiceRequestService` first.

## Folders Claude should inspect before making a change

- The full execution path for whatever is being touched: trigger → handler →
  service → selector, plus the LWC and any Flow that reads/writes the same
  object. Don't review only the file being edited — a change to
  `Escalated__c` logic, for example, affects the trigger, the service method,
  the LWC's `Escalated` display, and the escalation Flow's filter condition.
- `force-app/main/default/classes/*Test.cls` for existing coverage patterns
  before writing new tests.
- `docs/CODE-REVIEW-STANDARDS.md` and `docs/TESTING-GUIDE.md` before
  proposing a change.

## Folders/files Claude must ignore or never edit automatically

- `.sfdx/`, `.sf/`, any `*.log`, and anything matched by `.gitignore` —
  these can contain local auth/session state.
- Never read, print, or write real credentials, tokens, or connected-app
  secrets. This project should never need any — it targets scratch/Developer
  Edition orgs authenticated interactively.
- Do not modify `.git/` directly.

## Conventions

- After changing metadata in the org via Setup UI, retrieve it back into
  source before committing (`sf project retrieve start`).
- After editing source locally, deploy the specific component while
  iterating, then run Apex/Jest tests before opening a PR.
- Keep object/field/class/LWC names as declared above — do not rename
  without updating every reference (Flow, permission set, LWC import).
- Specialized sub-agents are available and described in
  [docs/SUB-AGENTS.md](docs/SUB-AGENTS.md) — route Apex work to the Apex
  Developer agent, test work to the Test Engineer agent, review work to the
  Code Reviewer and Security Reviewer agents, and deployment work to the
  Deployment and Metadata Analyst agent.

---
> Source: [armahajan24/claude-salesforce-development-demo](https://github.com/armahajan24/claude-salesforce-development-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
