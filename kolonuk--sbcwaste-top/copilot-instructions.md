## sbcwaste-top

> As an AI agent, I will follow the guidelines below when working on this repository.

As an AI agent, I will follow the guidelines below when working on this repository.

## Research and Investigation

When researching issues or making changes, always perform the following security checks:
- **Go Vulnerabilities:** Run `govulncheck ./...` (or `./src/...`) to identify Go-specific vulnerabilities.
- **General Vulnerabilities:** Run `trivy fs .` to scan the entire filesystem for security issues.

### Version Changes

**Before changing any version numbers** (Go version, dependencies, actions, etc.), always:
1. Check for security vulnerabilities in the new version
2. Verify compatibility with existing codebase
3. Test the change in a feature branch first
4. Ensure CI/CD pipeline passes with the new version
5. Review changelog/release notes for breaking changes

## Development Cycle and Deployment Process

### Workflow

- **Feature Branches:** All new work should be done on feature branches. These branches are created from the `dev` branch.
  - **Important:** Even for AI agent work (including VS Code Copilot agent chats), prefer creating feature branches and submitting pull requests instead of committing directly to `dev`. This ensures all changes go through the CI/CD pipeline with tests and security scans before merging.
- **Pull Requests:** When a feature is complete, a pull request is created to merge the feature branch into `dev`. This triggers the CI/CD pipeline, which runs tests and security scans.
  - All code changes, regardless of source, must be submitted via PR to benefit from automated testing and review workflows.
- **Development Environment:** Once the pull request is merged into `dev`, the changes are automatically deployed to the development environment.
- **Testing Environment:** When the `dev` branch is ready for testing, a pull request is created from `dev` to `test`. This deploys the changes to the testing environment.
- **Production Environment:** After testing is complete, a pull request is created from `test` to `main`. Merging this pull request deploys the changes to the production environment.

### Suggested Improvements

- **Semantic Versioning and Changelogs:**
  - Automatically generate a changelog and create a new release tag when a pull request is merged into `main`. This can be done using a tool like `semantic-release`. This will help you track changes and releases more effectively.
- **Infrastructure as Code:**
  - Use a tool like Terraform to manage your Google Cloud resources. This will allow you to version your infrastructure and make it easier to create new environments.
- **Container Security:**
  - Implement a container image signing process to ensure that only trusted images are deployed to your environments.
- **Code Quality:**
  - In addition to the existing validation jobs, consider adding a code quality tool like SonarCloud to your CI/CD pipeline. This will help you identify and fix code quality issues before they are merged into your codebase.
- **Automated Rollbacks:**
  - Configure your Cloud Run services to automatically roll back to the previous version if the new version fails to start. This will help you minimize downtime in the event of a bad deployment.
- **Dependency Management:**
  - Use a tool like Dependabot to automatically create pull requests to update your dependencies. This will help you keep your application secure and up-to-date.
- **Environment-Specific Configurations:**
  - Instead of using `if` conditions in your workflow to determine the environment, consider using different workflow files for each environment. This will make your workflows cleaner and easier to manage.
- **Review Apps:**
  - For pull requests, dynamically spin up a temporary "review app" with the changes. This allows stakeholders to test and review the changes in a live environment before they are merged.
- **Blue/Green or Canary Deployments:**
  - Instead of deploying directly to production, consider using a blue/green or canary deployment strategy. This will allow you to test the new version with a small subset of users before rolling it out to everyone. This can be achieved with the `--no-traffic` flag in the `gcloud run deploy` command, followed by gradually shifting traffic.
- **DORA Metrics:**
  - Track DORA (DevOps Research and Assessment) metrics like deployment frequency, lead time for changes, change failure rate, and time to restore service. This will help you measure and improve your DevOps performance.
- **ChatOps:**
  - Integrate your CI/CD pipeline with a chat tool like Slack or Microsoft Teams. This will allow you to receive notifications about your deployments and take action from within your chat client.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kolonuk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kolonuk)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
