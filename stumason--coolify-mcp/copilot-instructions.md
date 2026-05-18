## 803-npm-publish-workflow

> FOLLOW npm publishing workflow WHEN releasing package updates TO ensure proper versioning and release process

# NPM Package Publishing Workflow

## Context
- When releasing new package versions to NPM
- When changes need to be published for users
- When bumping package version numbers

## Requirements
- Run all tests before publishing
- Update CHANGELOG.md with version changes
- Use npm version command for version bumps
- Push git tags after version update
- Verify package installation post-publish

## Examples
<example>
# Release patch version
git checkout main
git pull
npm test
npm version patch
git push origin main --tags
npm publish --access public
npm install -g package-name@latest
</example>

<example type="invalid">
# Incorrect release process
vim package.json  # manually edit version
npm publish  # publish without tests
# forgot to push tags
# no verification
</example>

<critical>
- NEVER manually edit version in package.json
- ALWAYS run tests before publishing
- ALWAYS push tags after version bump
- ALWAYS verify published package
</critical>

---
> Source: [StuMason/coolify-mcp](https://github.com/StuMason/coolify-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
