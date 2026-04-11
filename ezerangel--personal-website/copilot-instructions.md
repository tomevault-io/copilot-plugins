## personal-website

> - This is our knowledge context. Before any task read this file to get better understanding about hows and whys.

- This is our knowledge context. Before any task read this file to get better understanding about hows and whys.

- Before each commit, reflect on the lessons we've learned together and update GEMINI.md for future use. Focus on generic patterns and techniques, not specifics of a single feature.

## SvelteKit Form Handling

- For handling form submissions, the idiomatic SvelteKit approach is to use Form Actions defined in a `+page.server.ts` file.
- Actions should return a plain object (e.g., `return { success: true }`), not a `json()` response. The framework handles the serialization.
- In the corresponding `+page.svelte` component, use the `form` prop to access the data returned from the action. Use `export let form: ActionData;` to type it, importing `ActionData` from `./$types`.
- If the page does not have a `load` function, do not include a `data` prop or import `PageData`. Keep the component's props minimal.
- Use the `use:enhance` action from `$app/forms` for progressive enhancement. This provides a better user experience by avoiding full-page reloads on submission.

## Project Tooling

- Be mindful of pre-commit hooks (e.g., linters, formatters). If a commit fails due to a formatting error, check the tool's output. Often, the tool will automatically fix the files, and the only required action is to stage the changes again and re-commit.

## Third-Party Integrations

### Airtable

- When integrating with Airtable from a SvelteKit server action, use the official `airtable` package for a cleaner implementation.
- Store sensitive credentials like API keys (`AIRTABLE_ACCESS_TOKEN`) and Base IDs (`AIRTABLE_BASE_ID`) in a `.env` file.
- Access these credentials securely in `+page.server.ts` by importing them from `$env/static/private`.
- Initialize the Airtable client and base connection outside of the `actions` object to avoid re-configuring on every request.
- When creating records, ensure the `fields` object keys exactly match the column names in your Airtable table, including spaces or special characters (e.g., `"URL Website"`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EzeRangel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EzeRangel)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
