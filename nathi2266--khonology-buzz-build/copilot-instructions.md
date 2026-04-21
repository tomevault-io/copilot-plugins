## khonology-buzz-build

> For debugging or fixing issues, present 1 line describing the problem and 1 lines describing the solution.

For debugging or fixing issues, present 1 line describing the problem and 1 lines describing the solution.

For every code change, provide a combined commit message summarizing all edits made in the current chat.

Ensure all code/script changes pass linting with zero lint errors before completion.

When adding or replacing an asset image:
- Use filenames and folder paths without spaces. Replace spaces with underscores "_" in both filenames and directories.
- Example: "Send Plane.png" -> "Send_Plane.png". A folder like "Project Management" -> "Project_Management".
- After any rename, update all references and ensure the asset path is declared under flutter.assets in pubspec.yaml.
- Size the image appropriately for its placement so it renders correctly.

When the user sends the text "cursor-push" in chat:
- Read all code/script changes in the repo, generate a single combined commit message summarizing changes, then commit and push to the current branch.

After any code change or edit:
- Automatically generate a concise, descriptive commit message that follows conventional commit format (e.g., 'feat: add login validation', 'fix: resolve API connection issue')
- Include the main changes made in bullet points if there are multiple changes
- Ensure the message is clear, specific, and follows best practices for Git commits

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nathi2266) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
