## ecosystemref

> Always make use of sesh-tracker.com, kush.observer, and my-cannabis-tracker.com to effectively navigate and manage the three integral components of our ecosystem. You'll find information about each aspect of the ecosystem below:

Always make use of sesh-tracker.com, kush.observer, and my-cannabis-tracker.com to effectively navigate and manage the three integral components of our ecosystem. You'll find information about each aspect of the ecosystem below: 

----------------------------------------------------

Each of these platforms serves a specific purpose: 

- **sesh-tracker.com** is the main user interface. Users will log in to this app to help log and monitor their sessions, inventory, providing insights into their stock and consumption patterns. Users will initially interact with a landing page with a guided tour of some sort, they can sign in or login, which afterwards brings them to their personalized dashboard, something that will be configurable with a drag and drop dashboard. My vision for this is that information entered will be captured, tracked, and made reportable through an extensive report builder. All data entered by users will ultimately be stored on a D1 Database tied directly to the Cloudflare worker. Users can then use these attributes, fileds, and data to create those reports, custom dashboards, and gain all sorts of insights from inventory, consumption, and so much more. 

- **kush.observer** is strictly the authenciation portal for **sesh-tracker.com** and **my-cannabis-tracker.com**. App users will interface primarily with **sesh-tracker.com**, with the account, authentication, and token system occuring with **kush.observer**. RBAC will primarily be housed here, with the admin dashboard mentioned next being the interface for admins to manmage those permissions and controls. This user information, preferences, subscription tiers, status, emails, usernames, display names, and so one will all be stored in a D1 database directly tied to the worker in Cloudflare. 

- **my-cannabis-tracker.com** will act as the backend technician/admin dashboard. Customer/subscription management will take place here as a nice and simple interface to ultize for managing and making those changes. 

In addition to these dedicated resources, don’t forget to leverage the worker page URLs that may already be present in the codebase. These links can offer additional functionalities and support, further enhancing your ability to manage and engage with our ecosystem.

---
> Source: [seshtrackerdev/sesh-trackeroldKUSH](https://github.com/seshtrackerdev/sesh-trackeroldKUSH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
