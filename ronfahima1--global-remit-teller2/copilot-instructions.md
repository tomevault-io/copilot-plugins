## global-remit-teller2

> ✅ Project Structure Rules




✅ Project Structure Rules
	1.	Use a Clean Folder Structure:

/app (or /pages)
/components
/ui
/lib
/hooks
/services
/styles
/types
/public


	2.	Group Related Files Together (Atomic design or feature-based):

/components/Button.tsx
/components/Navbar.tsx
/components/Sidebar/
    index.tsx
    SidebarItem.tsx


	3.	Use a /ui Folder for design system components (buttons, inputs, cards):
	•	Reusable, styled, low-level UI elements.

⸻

⚙️ Code Structure & Maintainability Rules
	4.	Keep Components Small & Focused:
	•	One component = one purpose.
	•	Use children or composition for flexibility.
	5.	Avoid Logic in UI:
	•	Use hooks/ for business logic (e.g., useAuth.ts, useFetch.ts).
	•	Use lib/ for utilities (e.g., formatDate.ts, apiClient.ts).
	6.	Use TypeScript Always:
	•	Define clear types/interfaces in /types.
	7.	Use Constants:
	•	Store static values in constants.ts.

⸻

🧱 UI & Styling Best Practices
	8.	Use Tailwind CSS or a UI library (e.g., ShadCN, Chakra UI, Radix):
	•	Write minimal CSS.
	•	Keep design consistent across the app.
	9.	Avoid Inline Styles:
	•	Use Tailwind or classNames.
	10.	Theme & Design Tokens:
	•	Centralize theming (colors, spacing, typography).

⸻

🚀 Performance & Optimization
	11.	Lazy Load Components with dynamic imports:

const DynamicComponent = dynamic(() => import('../components/HeavyComponent'))

	12.	Memoize Heavy Components:

export default memo(HeavyComponent)

	13.	Use SWR/React Query for API state instead of useEffect.

⸻

🧪 Testing & Validation
	14.	Write Unit Tests with Jest or Vitest.
	15.	Use Zod or Yup for input validation.

⸻

🧼 Code Quality & Developer Experience
	16.	Use ESLint + Prettier with custom rules.
	17.	Create VSCode Settings for consistent formatting.
	18.	Use Aliases in tsconfig.json:

"@components/*": ["./components/*"],
"@lib/*": ["./lib/*"]



⸻

💡 Example: Folder Setup

/components
  Navbar.tsx
  Sidebar/
    index.tsx
    SidebarItem.tsx
/ui
  Button.tsx
  Input.tsx
/lib
  api.ts
  auth.ts
/hooks
  useUser.ts
  useToggle.ts
/pages
  index.tsx
  dashboard.tsx
/types
  user.ts
  common.ts
Absolutely, Ron. Here are more advanced and practical rules to build a clean, maintainable, and scalable Next.js app, especially useful for startup or production-grade apps:

⸻

🔁 Component Design & Reusability

19. Use Composition Over Props Explosion
	•	Prefer passing children or composing small components instead of endless props.

// ❌ Not Ideal
<Card title="..." subtitle="..." footer="..." />

// ✅ Better
<Card>
  <CardHeader>Title</CardHeader>
  <CardBody>Content</CardBody>
  <CardFooter>Footer</CardFooter>
</Card>

20. Use a Layout Component
	•	Put headers, sidebars, auth wrappers in a layout component.

export default function DashboardLayout({ children }) {
  return (
    <Sidebar>
      <Header />
      <main>{children}</main>
    </Sidebar>
  );
}



⸻

🌍 Routing & API Design

21. Use App Router (/app) Instead of Pages
	•	Enables file-based layouts, loading states, error boundaries, and co-location of server/client components.

22. Use Server Components (Default) When Possible
	•	Avoid bloating client bundles. Only use "use client" when interaction is needed.

23. Use route.ts Files for API Abstractions
	•	Keep backend logic in /app/api/, and use a lib/api.ts to wrap API calls from the frontend.

⸻

🧠 State Management & Logic

24. Avoid Global State Unless Necessary
	•	Prefer local state, useState, or useReducer.
	•	If needed, use Zustand, Jotai, or Recoil instead of Redux.

25. Keep Business Logic in Custom Hooks

export function useSubmitForm() {
  const [loading, setLoading] = useState(false);
  return async (data: FormData) => {
    setLoading(true);
    await fetch("/api/submit", { method: "POST", body: JSON.stringify(data) });
    setLoading(false);
  };
}



⸻

🔐 Security & Access Control

26. Always Validate Data on the Server
	•	Use Zod, Joi, or custom validation on /api endpoints.

27. Protect Routes with Middleware
	•	Use middleware.ts to check authentication before rendering pages.

⸻

⚙️ Dev Experience & Collaboration

28. Use Git Hooks with Lint-Staged
	•	Prevent bad commits:

npx husky add .husky/pre-commit "npx lint-staged"

29. Use .env.local and NEVER commit secrets
	•	Keep API keys and environment variables local and secure.

30. Document Shared Components & Conventions
	•	Maintain a README.md or CONTRIBUTING.md for new devs.

⸻

📦 Package Management & Dependencies

31. Stick to One Package Manager
	•	Choose pnpm, yarn, or npm and enforce with .npmrc.

32. Avoid Installing Unnecessary Dependencies
	•	Audit with:

pnpm why package-name



⸻

📁 Naming Conventions

33. Use Consistent Naming
	•	Files: camelCase or PascalCase for components.
	•	Folders: lower-case-dash or camelCase.

⸻

🗂 Code Splitting & Performance

34. Split Large Pages Into Modules
	•	If a single page grows too large, break into:

dashboard/
  Overview.tsx
  Analytics.tsx
  Reports.tsx



35. Cache Expensive Calls (SSR or ISR)
	•	Use revalidate options in getStaticProps or RSC fetch.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RonFahima1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
