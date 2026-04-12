## resto-ai-strategy

> **Rule: This rule is always on.**

# Always Prefer Packages Over Custom Code (Next.js, Tailwind v4, shadcn)

**Rule: This rule is always on.**

For this project, which uses Next.js (App Router, no `src` directory), Tailwind CSS v4, and shadcn:

- **If a feature, component, or utility can be installed via a well-known package or command line tool, always install and use the package instead of writing custom code.**
- This includes (but is not limited to): UI components, icons, utility libraries, authentication, form handling, and styling solutions.
- Always use the latest stable versions of packages.
- For UI, prefer shadcn/ui and Tailwind CSS components when available.
- Only write custom code if there is no suitable, well-maintained package available or if project requirements demand a custom solution.

## Examples
- Use `@shadcn/ui` for UI components instead of building from scratch.
- Use `clsx` or `classnames` for conditional class names instead of custom logic.
- Use `zod` for validation, `react-hook-form` for forms, etc.
- Use `next-auth` for authentication if needed.

## Project Structure
- The project uses the Next.js App Router (no `src` directory).
- Tailwind CSS v4 and shadcn are the default styling and component solutions.

## References
- @next.config.js (if present)
- @tailwind.config.js (if present)
- @app/ (main application directory)

---
**Summary:**
> Always prefer installing and using well-known packages or command line tools for features/components, and only write custom code when absolutely necessary.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/0xbhvn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
