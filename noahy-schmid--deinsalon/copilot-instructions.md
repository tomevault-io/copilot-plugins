## deinsalon

> In this repository you find a Project whose aim is to provide a Saas booking service for salons. Salon managers can configure an online presence for their salon in the `apps/manage-salon-webpage` nextjs website. This includes (for now and in future), configuring a website, managing stylist employees, services offered to customers and booked appointments by customers. Customers can find the salon presence in the `apps/salon-webpage` nextjs app, there at the path leading to the correct salon, they can view the configured webpage and book appointments.

# dein.salon Monorepo Project

In this repository you find a Project whose aim is to provide a Saas booking service for salons. Salon managers can configure an online presence for their salon in the `apps/manage-salon-webpage` nextjs website. This includes (for now and in future), configuring a website, managing stylist employees, services offered to customers and booked appointments by customers. Customers can find the salon presence in the `apps/salon-webpage` nextjs app, there at the path leading to the correct salon, they can view the configured webpage and book appointments.

# Package structure

The Project strictly follows domain-driven-design and therefore consists of multiple domains which might be extended in the future and dont oblidge to the ddd rules completely yet:

- `packages/salon-domain`: This domain cares about the fundamental properties regarding a salon, so data like address, name, opening times, stylists and services provided by the salon, more to come.
- `packages/website-domain`: Needs to be renamed to `website-domain` in the future. This domain cares about the configuration and visual appearance of the website regarding each salon. Therefore it contains sections for the website, titles, styles etc.
- `packages/auth-domain`: This domain cares about user managemant and authorization for both stylists and managers accessing the salon management page and authenticating users trying to book and manage their appointments.

Most code is written using Effect-TS (documentation can be found [here](https://effect.website/llms.txt))

# More Information

You should read the following files on a need to know basis.
But if the rule applies you **MUST** read them before proceeding.

- `AGENTS-Backend-Service.md`: Read this file before planning or modifying anythin that touches one of the backend domains (Also backend logic within the frontends).
- `AGENTS-Frontend-UI.md`: Read this file before planning or modifying anything that touches the frontend.

# Important

**NEVER use dynamic imports** (e.g., `await import(...)`)
Make sure after you made changes, all formatting and linter rules pass. You can run a fix on them using `pnpm format:fix` and `pnpm lint:fix` in the root folder
You should also check whether everything builds properly you can do that by running `pnpm build --filter <package-name>` in the root directory. You only need to do that for packages where you actually did changes. If you made a larger change though just execute the command without any filter to build everything.

---
> Source: [noahy-schmid/deinsalon](https://github.com/noahy-schmid/deinsalon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
