## style-guide

> - Follow Google's Typescript style guide.

- Follow Google's Typescript style guide.
- Don't use relative import paths (e.g. "../some_file"). Use absolute path (e.g. "@/a/b/some_file"), unless it is in the same folder.
- For file names:
  - React components should use CamelCase for convention.
  - UI components under components/ui/ should use kebab-case for Shadcn UI convention.
  - Web routes in app/ should use kebab-case for URL readability.
  - Everything else should be snake_case as per Google style guide.
- All APIs under app/api should be RESTful.
- ALWAYS WRITE TESTS FOR WHATEVER YOU MODIFY!!! They will save you hours of compute time!!!

---
> Source: [openrp-ai/openrp-base](https://github.com/openrp-ai/openrp-base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
