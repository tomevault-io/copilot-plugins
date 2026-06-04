## file-namings

> File and folder naming rules


File and folder naming rules

- **General**:
  - All directories and files must use kebab-case.
  - Example: `user-avatar/user-avatar.tsx` (correct); `userAvatar/userAvatar.tsx` (incorrect).
- **Components**:
  - The component name inside the file remains camelCase, e.g., `userAvatar`.
  - This rule applies only to folder and file names, not to exported identifiers.
- **Hooks (exception)**:
  - Directory — kebab-case with the `use-` prefix.
  - Hook file — camelCase starting with `use`.
  - Examples: `use-user/useUser.ts`, `use-user/useUser.tsx`.
- **Tests and stories**:
  - Use the same kebab-case for paths.
  - Examples: `user-avatar/__tests__/user-avatar.test.tsx`, `user-avatar/user-avatar.stories.tsx`.

---
> Source: [Adil78124/AgroHub](https://github.com/Adil78124/AgroHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
