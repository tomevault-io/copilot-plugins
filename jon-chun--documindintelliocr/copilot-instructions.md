## documindintelliocr

> - **The Spec is the Single Source of Truth**

- **The Spec is the Single Source of Truth**
  - The `health-coach-ai-api-spec.json` file is the definitive contract for our API.
  - All API development **must** start with an update to this specification. This is a non-negotiable, spec-first approach.

- **When You MUST Update the API Specification**
  - **Before Creating New Endpoints:** If you are creating a new API route (e.g., `app/api/clients/route.ts`), you must first define the path, method, parameters, and schemas in `health-coach-ai-api-spec.json`.
  - **Before Modifying Existing Endpoints:** Any change to an endpoint's request body, response payload, parameters, or behavior must be reflected in the spec first.
  - **When Data Models Change:** If you alter a database table (e.g., in a new `supabase/migrations/*.sql` file) that affects the data returned by an API, the corresponding schema in the spec file must be updated.
  - **When Business Logic Changes:** If a server action in `app/actions/` introduces a new interaction that should be exposed via an API, the spec must be updated to include it.

- **How to Update the Specification**
  1.  **Locate and Edit:** Open the `health-coach-ai-api-spec.json` file.
  2.  **Follow OpenAPI 3.0 Standards:** Adhere strictly to the OpenAPI 3.0 syntax.
  3.  **Be Descriptive:** Add clear and concise `summary` and `description` fields for every new path, operation, and schema property. This is crucial for auto-generated documentation.
  4.  **Reference Existing Schemas:** Whenever possible, reuse existing schemas using `$ref` to keep the specification DRY (e.g., `"$ref": "#/components/schemas/Client"`).

- **Verification and Synchronization**
  - After updating the spec, you **must** synchronize it with the Apidog platform to ensure the entire team has access to the latest version.
  - Refer to the [Apidog Workflow Guide](mdc:docs/apidog-workflow.md) for detailed instructions on how to sync the file and test your changes.
  - **DO NOT** merge any pull request with API changes unless the corresponding specification update is included and verified.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jon-chun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
