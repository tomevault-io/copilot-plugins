## git-push-and-npm-publish

> When the user says git push and publish, perform the following actions in order:

When the user says git push and publish, perform the following actions in order:
	1.	Commit any pending changes and push to the main branch.
	2.	Increment the patch version in package.json using npm version patch.
	3.	Push commits and tags to Git: git push && git push --tags.

Note: The GitHub Actions workflow (.github/workflows/release.yml) will automatically:
	- Run linter (npm run lint)
	- Build the project (npm run build)
	- Create a GitHub Release with auto-generated release notes
	- Publish the package to npm

The manual steps of linting, building, and publishing to npm are no longer required as they are handled by the CI/CD pipeline.

---
> Source: [arpit-patel1/n8n-nodes-netdevices](https://github.com/arpit-patel1/n8n-nodes-netdevices) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
