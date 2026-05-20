## opensource-startup-crm

> BottleCRM is a dynamic, SaaS CRM platform designed to streamline the entire CRM needs of startups and enterprises. Built with modern web technologies, it offers a seamless experience for users through robust role-based access control (RBAC). Each user role is equipped with tailored functionalities to enhance efficiency, engagement, and management, ensuring a streamlined and secure business process.

Dear Copilot,

## Project Overview

BottleCRM is a dynamic, SaaS CRM platform designed to streamline the entire CRM needs of startups and enterprises. Built with modern web technologies, it offers a seamless experience for users through robust role-based access control (RBAC). Each user role is equipped with tailored functionalities to enhance efficiency, engagement, and management, ensuring a streamlined and secure business process.

user types we have
    
-   Org
    -   user(s)
    -   Admin
-   super admin - anyone with @micropyramid.com email to manage whole platform

## Project Context

BottleCRM is a modern CRM application built with:
- **Framework**: SvelteKit 2.21.x, Svelte 5.1, Prisma
- **Styling**: tailwind 4.1.x css
- **Database**: postgresql
- **Icons**: lucide icons
- **Form Validation**: zod

## Important Notes
- We need to ensure access control is strictly enforced based on user roles.
- No record should be accessible unless the user or the org has the appropriate permissions.
- When implementing forms in sveltekit A form label must be associated with a control
- svelte 5+ style coding standards should be followed

---
> Source: [MicroPyramid/opensource-startup-crm](https://github.com/MicroPyramid/opensource-startup-crm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
