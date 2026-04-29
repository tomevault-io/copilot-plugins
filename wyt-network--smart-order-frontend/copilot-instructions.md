## smart-order-frontend

> **Write down your thinking process as comment inside the code. Label it either is is requested by user or you think it is best pratice**

**Write down your thinking process as comment inside the code. Label it either is is requested by user or you think it is best pratice**

1. Use tailwind for styling, use server component if possible
2. This is NextJS app, put your page inside src/app, components inside src/components
3. Use functional component export default function MyComponent() {}
4. Split screen into multiple sections, split components into multiple small components, component should have its own file
5. Ignore the font in the given code. do not include font style in the tailwind classname
6. We are using tailwind v4 so no tailwind-config.js is needed. Edit src/app/globals.css instead
7. We are using latest animation timeline scroll in css.
8. Page's max width is set to 1440px
9. When a component is using useState, or useEffect, put it into a client component with "use client" directive. Minimize the use of the "use client". Try to achieve using basic server component. For example, when dealing with pagination, store the current page to the query parameters, and then when next page is clicked, increase the query parameter by one. When dealing with data fetching, fetch the data as close as the component, and wrap that server component with suspend to provide better performance. Unless, you cannot achieve the effect with above methods, use client component instead. Only make the part that must be client component to be client component!
10. Use animation-timeline to build a scroll based animation. Use framer-motion for animation. Use pnpm to install package.
11. Write down your thinking process in the comment so future improvements won't override existing code
12. NextJS Config file is at the root `next.config.ts` file
13. Some code are directly copied from Figma which needs you to extract duplicated elements into a for each mapping to make it robust. And second you need to fix the responsivness issue. For example, lots of code are hard coded width, you need to use max-width to improve the layout.
14. Section is the name for the page content. Page may contain multiple Sections, and each section may content multiple components. If a page is at route /home, and IntroSection is for the home page. Then put IntroSection component under /src/components/home/1.intro/IntroSection.tsx, and its components at /src/components/home/1.intro/SomeComponent.tsx. Moduler css is located at /src/components/home/1.intro/intro.module.css. Name the folder as their apperence order. Like 1.intro, 2.welcome
15. Use browser tools to see any potential layout mismatch. If user complain about layout mismatch, or doesn't seem right, use browser tool to help. Always check **browser tool's** error if you implement a new component!!!
16. Use heroicons and lucide for all svg!
17. Always check package.json if need to add new package, it may already installed
18. If user mentioned the code is copied from figma, check if the element centered using browser tools. Otherwise, change the width of the element.
19. If an UI component needed, we will use `shadcn` UI library.
20. ** Important! One component per file!** Split components into multiple files if possible!
21. When asking to create a style for mobile screen, create a seperate file contains a single component with current desktop version's code, make another file that has the new requested mobile style code.
22. Prefer server component/server actions over client side.
23. Do not put component at root level of components folder

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wyt-network) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
