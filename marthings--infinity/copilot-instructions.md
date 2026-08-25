## infinity

> Infinity is a personal, global place to collect links, images, files, and inspiration found across social media and other platforms. It is similar to Pinterest, but private and personal.

# Infinity

Infinity is a personal, global place to collect links, images, files, and inspiration found across social media and other platforms. It is similar to Pinterest, but private and personal.

## Stack

- Rails is the application framework.
- Ruby Native delivers the Rails app as native iOS and Android apps. Build the web experience first, and preserve compatibility with its native shell and navigation.
- The web app is deployed on Hetzner with Kamal. Deployment configuration is managed manually unless a task explicitly requires changes.
- Use import maps, Turbo, Stimulus, and vanilla CSS. Do not introduce Node, a bundler, Tailwind, or other frontend dependencies without a clear, agreed need.
- Use CSS features supported by current evergreen browsers.

## Rails

- Follow Rails conventions and DHH-style judgment: simple CRUD resources, thin controllers, rich models, and database-backed data integrity.
- Prefer Rails' built-in features and the existing codebase over gems, service objects, custom frameworks, or premature abstractions.
- Use `params.expect` for strong parameters.
- Scope all records through the current user or tenant boundary. Never fetch user-owned records globally.
- Prefer database constraints and indexes for integrity. Application validations should provide useful user-facing feedback, not replace constraints.
- Avoid boolean state columns when a state record is more expressive and queryable.
- Keep domain logic out of helpers and views.
- Before adding an abstraction, find at least three real variations or keep the code explicit.

## Views And CSS

- Reuse partials and existing patterns. Check for an established component or partial before creating a new one.
- Keep markup semantic, accessible, and readable. Use human-readable CSS class names tied to the domain.
- Use vanilla CSS with the cascade and native browser capabilities. Avoid utility-class styling and unnecessary JavaScript.
- Build responsive interfaces that work with mouse, touch, keyboard, and screen readers.
- Do not create nested cards or decorative visual clutter. Favor clear, work-focused layouts.

## Accessibility And Privacy

- Meet WCAG 2.2 AA as a minimum: semantic HTML, keyboard operation, visible focus states, correctly associated labels and errors, sufficient contrast, and appropriately described media and controls.
- Design for GDPR from the start: data minimization, explicit purpose boundaries, secure handling of personal data, and support for access, export, and deletion where applicable.
- Treat third-party URLs, remote media, and uploaded files as untrusted input. Validate, authorize, and avoid exposing private user data.

## Git Workflow

- Start every new feature from an up-to-date `main` branch.
- Break features into GitHub issues with appropriate labels.
- Implement work in focused branches and submit pull requests.
- Squash merge PRs to keep a flat history, then delete the merged branch.
- Do not revert or overwrite unrelated work already present in the working tree.

## Engineering Expectations

- Keep dependencies minimal and justify every new gem or toolchain addition.
- Refactor when complexity obscures the domain or harms the user experience.
- Flag practices that conflict with these principles, especially unnecessary dependencies, inaccessible UI, missing authorization boundaries, avoidable complexity, or violations of Rails conventions.

---
> Source: [marthings/infinity](https://github.com/marthings/infinity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
