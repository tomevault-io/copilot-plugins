## terragon-oss

> **IMPORTANT**: When adding or updating links in documentation files, you MUST verify that all links are valid and accessible:

# CLAUDE.md - Documentation Guidelines

## General Documentation Guidelines

### Link Verification

**IMPORTANT**: When adding or updating links in documentation files, you MUST verify that all links are valid and accessible:

1. **Internal Links**: Verify that the target file or section exists in the documentation
2. **External Links**: Test that external URLs are accessible and return valid responses
3. **Anchor Links**: Ensure section anchors match the actual heading IDs in the target page

Use tools like `curl` or `WebFetch` to verify external links before committing documentation changes.

## Release Notes

When making changes to release notes (`apps/docs/content/docs/resources/release-notes.mdx`), always use the template and guidelines in `RELEASE_NOTES_TEMPLATE.md`.

**IMPORTANT**: After adding a new release notes entry, you MUST bump the `RELEASE_NOTES_VERSION` constant in `apps/www/src/lib/constants.ts` by incrementing it by 1. This triggers the release notes badge to appear for users.

The template ensures consistent formatting and includes guidance for:

- Writing user-focused descriptions
- Proper formatting and capitalization
- Handling early access features
- Required sections and organization

---
> Source: [terragon-labs/terragon-oss](https://github.com/terragon-labs/terragon-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
