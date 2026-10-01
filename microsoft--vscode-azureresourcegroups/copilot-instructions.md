## vscode-azureresourcegroups

> Securely parse and render untrusted Copilot on Rails workspace artifacts.


# Secure Copilot on Rails artifact handling

Treat `.azure/*`, `.azure/.preview-temp/*`, `.copilot-azure/sessions/*`, and `package.json` files inside
the Copilot on Rails user's project workspace as untrusted and potentially incomplete. Agents, users, and
other workspace tools can edit them, and readers can observe a partial write.

When code reads or renders these artifacts:

- Use one-argument `JSON.parse(text)`, assign the result to `unknown`, and narrow the root and every consumed
  field with runtime checks.
- Preserve valid fields and array entries in an incomplete artifact. Do not coerce unsupported values into
  strings.
- Validate artifact-supplied path parts before joining paths. Validate URLs, commands, process arguments,
  file operations, HTML, and SVG for their specific destination.
- Render agent-written Markdown as parsed React nodes rather than with `dangerouslySetInnerHTML`. Explicitly
  allowlist link protocols and supported HTML tags. Configure Mermaid with `securityLevel: "strict"` before
  rendering.
- Add targeted tests for malformed roots, wrong field types, partial objects, traversal strings, unsafe
  links or tags, and other invalid inputs that reach the changed destination.

Follow the full [Copilot on Rails artifact security contract](../../docs/copilot-create-project-security.md).
Update that document when the trust boundary or required handling changes.

---
> Source: [microsoft/vscode-azureresourcegroups](https://github.com/microsoft/vscode-azureresourcegroups) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
