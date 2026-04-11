## everlume

> Think step by step and show reasoning for complex problems. Use specific examples.

Think step by step and show reasoning for complex problems. Use specific examples.

When giving feedbacks, explain thought process and highlight issues and opportunities.

Break down large tasks and ask clarifying questions when needed.

During the development, ensure it follows and adheres to industry standards, best practices, principles, concepts, guidelines, frameworks, and methodologies. Specifically, ISO/IEC Standard 25010, OWASP Top 10, and Secure SDLC (SSDLC) for full-stack web application development, web security, secure coding, software engineering, system design and architecture, performance optimization, and UI/UX design.

Make sure a proper documentation and comments for the whole codebase and project. I require a comprehensive, step-by-step, detailed, informative, complete, and actual approach on resolving bugs and errors as well as implementation.

After finishing a task provide commit message for it.

---

## Project Architecture & Concrete Stack Constraints

The following technological constraints are absolute and must be followed for all future development on this project, overriding any previous plans or preferences:

- **Frontend & Deployment:** Next.js deployed on Vercel.
- **Styling:** Tailwind CSS.
- **Database & Authentication:** Supabase (Postgres & Auth).
- **Image Management (<100MB):** Cloudinary via their Upload Widget. Use Cloudinary URL transformations (e.g., adding `w_800,f_auto` to the link) for on-the-fly, mobile-optimized versions. _Do NOT use Supabase Storage with client-side compression code for images, as client-side resizing is unreliable on older mobile phones._
- **Large Media (Videos >50MB):** YouTube Unlisted. The UI should explicitly instruct users to "Upload videos to YouTube first, then paste the link here." _Do NOT attempt to upload large videos to Supabase Storage due to limits and poor user experience._
- **Routing & Short Links:** Cloudflare Workers with a short domain (e.g., `.ph` or `.com`). This allows physical memorial plaques to have an immutable QR code, while the destination URL can be updated via the Worker logic if needed.
- **Hosting / DNS:** Manage the DNS through Cloudflare to leverage free "Always Online" and "Under Attack" protections, while keeping the Next.js frontend deployed on Vercel.
- **Local Backups:** A shared Google Drive folder for original, high-res master files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CodeExplorer430)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CodeExplorer430)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
