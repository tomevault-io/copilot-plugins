## pingoo

> * Be mindful about your token usage, we often have errors about too much tokens used.


## General

* Be mindful about your token usage, we often have errors about too much tokens used.
* Always ensure to use the latest version of package when creating or updating a project.
* Don't use emojis unless very necessary (e.g. to make a joke)
* when you need to name an error variable (e.g. in a catch), name it `err`, instead of `e`
& avoid single-letter variable names as much as possible (other than for numerical code).
* use yaml if a project needs a configuration file
* always ensure that tests pass when owrking on a feature.

## Rust

* use cargo check before any build to make sure that the code compiles.
* Use `aws-lc-rs` for crypto operations.
* Use `reqwest` when you need an HTTP client and `axum` when you need an HTTP server.
* Use `sqlx` when you need to interact with a database. Use the `sqlx::query_as` function for database queries when relevant, to deserialize directly into a struct.
* When working, always build the projects in debug mode to go faster.
* use the `tracing` crate for logs.
* never use unwrap. If it's a `Result` handle it, if it's an `Option`, use safe patterns such as `.ok_or`, `if let Some()`...

## Web applications

* projects setup: when asked to create a web application, always create a single page app with vite and preact, starting from the preact-ts template.  Use `@tailwindcss/vite` to setup tailwindcss with vite.
* Use only tailwindcss for design and mantine for components. NEVER use css-in-js libraries, don't use inline styles, don't use separate CSS files unless strictly necessary.
* Always use our own preact-router from this repo for routing.
* Always make sure that the webapp compiles.
* ALWAYS use preact signals for state management. NEVER use the `useState` hook for state management. Use the `useSignal` hook for component-scoped signals. More info here: https://preactjs.com/guide/v10/signals
* Always use the following "dev" script in package.json `"dev": "vite --strictPort --port 4000 --host"`
* Use native links (`<a>` HTML tags) as much as possible instaod of special components link `Anchor` or similar.
* when creating a button to move to another page, wrap it into a link insteaod of using `onClick={router.push(...)}`
* prefer minimalist, simple and elegant design.
* when building user interfaces for APIs, always allow the dev to configure the API_BASE_URL via en environment variable. And use a proxy in the vitejs config for the `/api` path to this custom API_BASE_URL env var.
* Always use native HTML tags for text and headings (`<p>`, `<span>`, `<h1>`, `<h2>`...) instead of components like `Text`, `Heading` etc...
* When working on user interfaces, always create responsive user interfaces, and take in account mobile touch devices (so no too small buttons etc...)

---
> Source: [pingooio/pingoo](https://github.com/pingooio/pingoo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
