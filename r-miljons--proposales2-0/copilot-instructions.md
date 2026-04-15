## proposales2-0

> We are using Next.js 15, shadcn ui components, tailwind v3 and TypeScript.


We are using Next.js 15, shadcn ui components, tailwind v3 and TypeScript.

When writing code use shadcn components and global colors and styles defined at globals.css. Use icons from lucide-react. For text use @components/ui/Typography.tsx defined components. And write type-safe code.

Avoid hard-coded colors or styles unless spcified.

Use the correct alias (@/components/ui) for importing UI components.

When asked to write tests with functions, use Jest and keep the function name the same as the file name.

When creating new ui component files always use the project root components folder.

When using the Typography.tsx never assume the contents of the file and always inspect to make sure it's used correctly, when using existing ui components that have predefined text subcomponents such as Alert's AlertTitle, etc, typography should not be used.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/r-miljons) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
