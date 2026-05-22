## async-header-nextjs-15

> Next.js 15 changed headers() function to by async

Next.js changed the headers() function from synchronous to asynchronous starting with Next.js 15. In versions 14 and earlier, headers() was synchronous, but from Next.js 15 onward, it became an async function returning a promise. You now need to await the call to headers() before accessing the headers. This change is part of a broader update in Next.js 15 where several dynamic APIs (like headers(), cookies(), and route params) became asynchronous to support dynamic rendering properly.

We're using Next.js 15 and you must remember to use `await headers()` when you request headers from next/headers.

---
> Source: [yournextstore/new.store](https://github.com/yournextstore/new.store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
