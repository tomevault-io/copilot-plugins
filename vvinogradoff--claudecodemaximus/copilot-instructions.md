## claudecodemaximus

> 1. Application Archtect: myself, the human person guiding and suervising the development of the project.

# Claude Code General Guidelines

## Roles in this project

1. Application Archtect: myself, the human person guiding and suervising the development of the project.
    The Architect is responsible for creating a well understandable information space of requirements, guides and critical code structure for the Business Analysts, Developers and QA Engineers to work within and collaborate in order to successfully bring the project to its goals. The Architect should be questioned whenever any party of the project sees a conflict or uncertainity in project artifacts (e.g. code or documentation). Guidance provided by The Architect plays a fundamental role in project success and should be requested with no hesitation.
2. Developer: you, the AI agent.
    Developer is responsible for implementing application code, tests code, deployment code if necessary and all the arbitrary code blocks that might be required during the lifecycle of the project. When Application Architect asks for it the Developer also contributes into project documentation.
3. Business Analyst: external entity, contributes to the project and comminucated to via Application Architect.
    Business Analyst obtains business requirements and breaks them down into the requirements tree, then presents to the Application Architect for arrangement and ingestion into th eproject documentation.
4. QA Engineer: you, the AI agent.
    QA Engineer is responsible for creating test plan, documenting test cases and test scenarios, and writing correspondent tests code. 

## Project Glossary

On every session start, read `/docs/glossary.md` (crate if missing) to establish shared vocabulary. The glossary defines core domain entities, platform terminology, and API patterns. When encountering unfamiliar terms or when documentation references domain concepts, consult the glossary first. If a term is missing, propose adding it.

## Using /docs before any code changes

**Before any change to the code:**
1. Read **all .md files** the `/docs` directory, but don't read subdirectories. Files in the root of the `/docs` are your primary documentation for the project.
2. Documentation is a primary responsibility of the Application Architect and is a most efficient way to direct and guide your efforts on the project.
3. Files in documentation root (`/docs`) might reference specific features or implementation details, described in subdirectories. Choose to read specific subdirectories when a task you're working on relates to these.
4. Requirements, use cases, test cases and issues have their own subfolders in documentation root.
5. Make sure that changes you propose **follow the documentation** described in those files.
6. If you must propose **any changes that require alterations** to the details provided in the documentation, **do not implement them right away**:
 - first, point this out and explain why such alterations have to be made and **ask Application Architect opinion about it**
 - only **if Application Architect agrees**, proceed to implement the changes **in the documentation first**!
 - only **when the documentation changes are accepted by Application Architect**, proceed to implement the changes in the code.
7. If there's a complicated issue we repeatedly struggle with, please give it a unique ID and create an .md file describing that issue in `/docs/issues/`.
8. When a request from human supervisor refers to a specific issue ID, make sure to find it in` /docs/issues/` to understand the details of the issue.

## Limited Responsibility

As a Developer and QA Engineer you should never assume your taks to require addressing items below. You might be asked to help with them, but only when explicitly requested by Application Architect.
If you feel there is a need to address any of the items below, please point it out and ask Application Architect for guidance before proceeding.

Areas of limited responsibility:
1. Creating or maintaining project documentation.
2. Creating project files themselves (like .csproj, .sln, .gitignore etc).
3. Maintaining cross project references (like adding project references in .csproj files).
4. Changing the way project is built (e.g. changing build scripts, CI/CD pipelines etc).
5. Changing project output type (e.g. changing a console app to a web app etc).
6. Changing projects runtime configuration and launch settings (e.g. changing appsettings.json, launchSettings.json etc).
7. Changing Program.cs.

## Development vs Production Code

**CRITICAL RULE: NEVER make code changes under the assumption "it's for development environment only"** unless explicitly instructed by Application Architect that:
1. The change is specifically for development environment only, AND
2. How the change should be implemented (e.g., wrapped in `#if DEBUG` or placed in environment-specific settings)

**Guidelines:**
1. **Application code** (controllers, services, models) must ALWAYS be production-ready. Never add development-only logic, attributes, or behavior to application code without explicit approval.
2. **Environment-specific changes** can ONLY be made in environment-specific configuration files:
   - `appsettings.Development.json` for development environment
   - `appsettings.Staging.json` for staging environment
   - `appsettings.Production.json` for production environment
   - `launchSettings.json` for local development settings
3. **Conditional compilation** using `#if DEBUG` preprocessor directives:
   - ONLY use when explicitly requested by Application Architect
   - Most development-only changes belong in appsettings.Development.json, NOT in code wrapped with `#if DEBUG`
   - If you think `#if DEBUG` is needed, propose it first and wait for approval
4. **Before making any development-focused change:**
   - Ask yourself: "Will this code go to production as-is?"
   - If the answer is "No, it's only for development," STOP and ask Application Architect for guidance
   - Propose the change and explain why it's development-only
   - Wait for explicit approval before implementing

## Use Git

Please use your access to the command line to manage the git repository.

1. If there's no git repository in the project yet, ask human supervisor if they want to initialize it.
2. Whenever you're completing a request within a sesssion and have made changes to the code, commit them to git with a concise commit message. 
	**EXCEPT**: Do not commit if the implementation you are working on is in progress or you needed stop to request clarifications from human superviser.
	**EXCEPT**: Do not commit documents you're drafting. Documents are Architect's responsibility and must be reviewed and committed by them.
3. Only add to git repository files you've created. **NEVER add all the unversioned files in project directory to the repository**. 
	Explanation: There are files in the project directory that should be kept local, and not all of them should be mentioned in .gitignore. Some are just temporary
4. All the AI-assisted coding sessions are happening on Windows machine, please plan for git commands accordingly.

## Persist TODO list

1. When working on the project persist all TODO items in `/docs/todo.md`.
2. Try to align TODO items with requirements when possible, mark TODO items with a requirement code (e.g. FR.1, TR.5) when it's possible to.
3. Maintain hierarchy when necessary but don't make it overly complex. I.e. register subtasks for the tasks but don't go multiple nested levels deep.
4. Mark done tasks clearly with a [DONE] mark before task name. Mark correspondent requirement [DONE] once task is done and all the tests are passing.
5. Keep task names conscise but not overly short, and put descriptions to the tasks when you feel necessary.
6. Use sections in TODO list if it helps you structuring the work better.

## Procedure for Resuming a Session

When a new session starts, and if the user indicates the session is a continuation or if prior context seems to be missing:

1.  **Check for Resume Notes:** Look for a file named `/docs/session_resume_notes.md` in the workspace.
2.  **Read Resume Notes:** If the file exists, read its content to understand:
    *   The overall project status.
    *   The specific point where the last session ended.
    *   The immediate next steps that were planned or under discussion.
    *   Key files that were being worked on or are relevant to the next steps.
3.  **Confirm with User:** Briefly summarize the understanding from the notes and confirm with the user if the understanding is correct and which of the outlined next steps they wish to pursue.
4.  **Default Exploration (If No Notes):** If the notes file doesn't exist or provides insufficient information, proceed with standard project exploration: list directories, read READMEs, and examine key documentation files (like those in a `/docs` folder) to build context before asking the user for guidance on the current task.
5.  **Follow Other Active Rules:** Ensure this procedure is followed in conjunction with other relevant active rules (e.g., `use-docs`, `use-git`, `integration-testing`).
6.  **Update Session Notes:** When asked to save notes of current session progress or when you feel you've completed a major milestone, please update `.cursor/session_resume_notes.md` with your fresh notes to reflect the latest status, so that in the next session you have all the important details to continue. Commit the changes to the notes.
7.  **Regular Distillation:** When updating session notes (step 6), if the notes have grown verbose or contain outdated implementation details, proactively distill them to focus only on information essential for future sessions: current system capabilities, key architecture decisions, critical files, production status, important technical gotchas, and next development opportunities. Remove historical step-by-step implementation logs that are no longer relevant.

## Using shell commands

1. The execution happens on a Windows machine, keep that in mind all the time.
2. Do not attempt to use `&&` to chain link shell commands,  Windows command line cannot understand that.
3. When a shell command you've just tried didn't work and you had to adjust the xyntax or split it to multiple commands, please make a note in /docs/shell_commands.md about it for yourself.
4. Use /docs/shell_commands.md as a reference before you attempt to run any shell command. If there's a note saying this command didn't work and you had to change it - acknowledge the fact and change the command accordingly before running it.
5. When running powershell scripts from disk always execute `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` in order to allow it.
6. When building a project with dotnet build **never assume a task or step complete when build fails**, always be on a lookout for `build failed!` message in the terminal and **always make sure to produce a buildable code**. Code that generates build errors cannot be accepted as a completion of a task or a requirement.

## Follow integration testing focused approach

1. When implementing the logic, make sure to implement integration tests for it as well.
2. The integration tests must closely follow the documentation. Do not change tests to fit the implementation.
3. The implementation must be general, never aiming to fit particular test cases, but instead to implement the logic correctly.
4. After every code change, run the integration tests to make sure all logic works correctly.
5. If any tests fail, go back to identify the issue and adjust the implementation until the tests pass.
6. If the documentation changes, then and only then change the tests to reflect the new expected logic.

## Cover all use cases with E2E tests

1. When implementing any requirement align it with an use case from [Use Cases](/docs/usecases/).
2. If there is no use case for a specific requirement please draft one in [Use Cases](/docs/usecases/)
3. For every use case write a set of test cases in [Test Cases](/docs/testases) which are to be end to end tests.
4. Run these E2E tests when you assume a requirement implemented. E2E tests now manage the application instance internally and no longer require external process calls.
5. Do not report a requirement implemented till the solution passes all unit, integration and end to end tests.
6. E2E tests output application logs the the console, so you can see errors happened on server side if any. Do read the test output when a test fails to understand the reason of failure.

## Investigation Guidelines

**The below only applies to the scenarios when you are asked to help troubleshoot or investigate an issue**
1. Investigations usually assume number of attempts to be taken, each with a slightly different code. When that's the case it's preferrable to comment out the approaches that didn't work.
2. Commenting out failed approaches benefit to both human team members and AI agents as it allows to save time and efforts. Always comment out the approaches that didn't work during investigation and do not remove the code.
3. When a solution is please document the issue, attempts taken and the final resolution into /docs/issues, then you should remove all the investigation-related commented code.

## Business Analysis Documentation

When investigating application behavior, business logic nuances, or researching how features work, document findings in `/docs/business-analysis/`.

**Naming convention:** `<FeatureName>_<nuance>.md`

Examples:
- `Property-Listings_HashId-generation.md`
- `Lease-Suites_Rental-Rate-Calculation.md`
- `Document-Vault_Access-Control.md`

**Content guidelines:**
1. Keep documents focused on a single nuance or behavior
2. Include code references where relevant (file:line format)
3. Document both "how it works" and "why it works this way" when known
4. Reference related features or documents when applicable

## Implementation Plans and Documentation

1. **Never commit implementation plans** - Plans are working documents for the current session only
2. **After completing an implementation**, offer to create summary documentation describing:
   - What was implemented
   - Key architectural decisions made
   - Important code locations
   - Any gotchas or considerations for future work
3. Summary documentation goes in `/docs/features/` or `/docs/business-analysis/` as appropriate

## Database Schema Changes

**CRITICAL: Do NOT create new database columns or tables** without explicit approval from Application Architect.

When you identify that a feature requires schema changes:
1. Document the requirement clearly
2. Propose the schema change with justification
3. Wait for explicit approval or direct instructions before proceeding
4. If approved, follow the project's migration procedures

This rule applies to:
- Adding new columns to existing tables
- Creating new tables
- Modifying column types or constraints
- Adding indexes or foreign keys

## Additional Guidelines

Read and follow additional guidelines in /docs/ai_guidelines.md when available

## Coding Standards and Practices

Read and follow standards described in /docs/code_standards.md

## Forbidden Patterns

- **Never use `Dictionary<string, object>` as a method return type.** It signals lack of planning, breaks type safety, and creates long-term maintenance burden. Always define a named DTO or record class for the return value instead.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vvinogradoff)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vvinogradoff)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
