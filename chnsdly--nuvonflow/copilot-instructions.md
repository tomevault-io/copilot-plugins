## nuvonflow

> - The website serves industrial buyers, OEMs, system integrators, engineers, contractors, and project teams by providing clear, factual guidance on corrosion-resistant thermoplastic valves and related flow-control systems.

# AGENTS.md

## Purpose
- The website serves industrial buyers, OEMs, system integrators, engineers, contractors, and project teams by providing clear, factual guidance on corrosion-resistant thermoplastic valves and related flow-control systems.
- The main goal is to reduce customer uncertainty: help visitors judge product relevance, understand material and operating-condition constraints, identify required specifications, and take a clear next step.
- Decision priority: **customer utility and professional credibility first; user experience and visual quality second; implementation simplicity and performance third**. These goals should support one another, not be treated as excuses to neglect the others.
- The site should become simpler, clearer, more coherent, and easier to maintain as it evolves—not merely larger or more feature-rich.
- Avoid unsupported promotional claims. Do not fabricate project references, certifications, performance data, test results, product capabilities, or company strengths.

## Page and Experience Principles
- Every page must have a defined job. Before adding or changing a section, identify what the visitor needs to understand, judge, compare, or do on that page.
- Every module must earn its place by improving at least one of the following: comprehension, product or application judgment, trust, navigation, or conversion. Remove decorative or repetitive sections that do none of these.
- Build the customer journey around practical questions: Is this relevant to my system? Is the material suitable? What operating details matter? What information or documents are available? What should I do next?
- Use progressive disclosure: show the most decision-relevant information first, then provide deeper technical details, tables, diagrams, downloads, and supporting explanations.
- Visuals should clarify products, applications, system relationships, selection logic, or evidence. Do not add images merely to fill space, but do not leave important pages as undifferentiated walls of text when a product image, process diagram, comparison table, or technical illustration would communicate better.
- Maintain a clear visual hierarchy. Primary actions, navigation, auxiliary controls, and supporting information must not compete as if they have equal importance.
- Interaction must feel obvious without becoming visually loud. Clickable items should look actionable; supporting controls such as language selection should remain discoverable but subordinate to the primary CTA.

## Content Standards
- Write near-production-quality content rather than temporary filler. Content should be useful enough to publish after factual review, not merely indicate what a section may contain later.
- Write to the customer, not to the website owner or developer. Avoid sentences that explain what a professional website should do; directly provide the information or assistance the customer needs.
- Base content on real industrial practice, known standards, material behavior, operating conditions, procurement needs, and project workflows. Use specific data where it is verified and appropriate.
- Distinguish clearly between verified facts, typical industry practice, preliminary guidance, and conditions that require confirmation. Use qualifiers such as “typically,” “subject to actual operating conditions,” or “confirm against the manufacturer’s current data.”
- Do not present compatibility, pressure-temperature limits, dimensions, standards, certifications, or performance values as universal unless they are verified for the exact product or material.
- Focus on people-first content that answers real selection, specification, integration, maintenance, procurement, and documentation questions. Do not create pages solely to increase keyword coverage or page count.
- Keep content concise where the decision is simple and detailed where technical risk or purchasing uncertainty justifies depth. Professional does not mean verbose.
- Use clear headings, descriptive links, meaningful image `alt` text, concise page titles, and useful meta descriptions.
- Include complete trust and operational content where appropriate, including contact details, privacy policy, terms or disclaimer, document limitations, and transparent inquiry expectations.

## Information Architecture and SEO
- Each page must fit into a deliberate site structure and connect to relevant product families, applications, resources, or support paths. Avoid orphan pages and arbitrary page proliferation.
- Product, application, project-support, and resource content should have distinct roles. Do not repeat the same generic copy across multiple page types.
- Create separate SKU pages only when they provide meaningful, model-specific value such as dimensions, materials, pressure classes, connection standards, options, documents, or selection context. Do not generate thin near-duplicate pages.
- Use internal links to support real research paths, not merely to distribute keywords.
- Keep URLs, navigation labels, breadcrumbs, canonical relationships, metadata, and structured content consistent across languages.
- Prefer Hugo page bundles for article- or page-specific media so content and its unique images remain maintainable together. Keep genuinely shared brand, UI, and product-family assets in shared asset directories.
- Provide the basic brand and sharing assets expected of a finished site, including favicon/app icons, appropriate logo files, Open Graph images, and suitable social-card metadata.

## Visual and Interaction Standards
- Maintain one coherent design language across templates and languages: typography, spacing, alignment, containers, radii, borders, shadows, buttons, cards, forms, navigation, and interaction states should feel related.
- Use shared containers, spacing tokens, and layout rules so breadcrumbs, page titles, sections, grids, and body content align to intentional common baselines.
- Solve alignment and spacing problems at the container or component level. Do not use arbitrary offsets, negative margins, or page-specific patches unless the layout is genuinely unique and the exception is documented.
- Judge layouts as systems, not isolated screenshots. Check visual weight, balance, rhythm, whitespace, line length, image-to-text proportion, CTA prominence, and the relationship between adjacent sections.
- Favor restrained, precise visual treatment over excessive cards, borders, shadows, dividers, badges, arrows, and hover motion. Use decoration only when it communicates structure, state, or action.
- Hover, focus, active, expanded, and disabled states must follow a small, consistent interaction system. Do not invent a new effect for each component.
- Menus must support fast scanning and clear hierarchy. Mega menus, compact menus, and card-like entries may differ in structure, but should share typography, spacing, interaction logic, and visual restraint.
- Forms should make completion easy: use readable labels, subdued placeholders, sensible field widths, clear required states, useful validation, and a layout that gives the form sufficient visual priority.
- Responsive behavior must be designed, not merely compressed. Re-evaluate hierarchy, line wrapping, controls, diagrams, tap targets, and content order for mobile rather than copying the desktop composition unchanged.
- Test multilingual layouts with realistic long labels. Language controls must be scalable to additional locales, keyboard-accessible, dismissible by clicking outside or pressing `Escape`, and visually subordinate to the primary navigation and CTA.

## Technical Guidelines
- Use the existing Hugo static-site framework with SCSS and BEM. Favor semantic HTML, static rendering, low JavaScript usage, and predictable build output over adding frameworks or complex tooling.
- Reuse and improve existing tokens, functions, mixins, base styles, templates, partials, data files, and components before adding parallel implementations.
- Existing foundations are not untouchable. Modify or extend them when a site-wide need is proven, but first assess the impact on current components and languages.
- Prefer a smaller number of clear, reusable abstractions over many narrowly scoped utilities or page-specific selectors. Less code is valuable only when clarity, accessibility, and behavior are preserved.
- Keep component ownership clear. A component’s structure, style, state behavior, and responsive rules should be easy to locate and extend.
- Write semantic HTML with ordered headings, correct landmarks, lists, tables, form labels, buttons, and links. Add ARIA only where native semantics are insufficient.
- Ensure keyboard navigation, visible focus states, adequate contrast, usable tap targets, and reduced-motion consideration where motion is used.
- Optimize images and SVGs according to purpose. Use SVG for suitable diagrams and icons; use appropriately sized modern raster formats for photography or detailed product imagery. Avoid large assets that provide little visible value.
- Minimize scripts and third-party dependencies. Measure or reasonably assess the performance cost of visually complex features before keeping them.
- Hugo should handle what it can cleanly handle. When a feature requires Cloudflare Pages Functions, Workers, D1, Turnstile, caching rules, form processing, or another external service, implement the site-side contract cleanly and leave an explicit integration point rather than forcing an unsuitable Hugo-only workaround.

## Multilingual Guidelines
- Maintain separate content files for each language using Hugo’s multilingual conventions, such as `page.en.md` and `page.zh.md`, while preserving equivalent information architecture and stable translation relationships.
- Do not hard-code interface text in templates when it belongs in i18n data or language-specific content.
- Treat every navigation, layout, form, metadata, and component change as multilingual by default. Check all configured languages before considering the work complete.
- Translations should preserve meaning, decision value, and professional tone rather than mechanically matching sentence structure.
- Design for future locales without assuming every language has similar word length, punctuation, capitalization, or line-breaking behavior.

## Development Practices
- Read this file and inspect all relevant templates, styles, data, content, and configuration before editing. Trace the current implementation far enough to understand the actual source of the behavior.
- Start with the underlying user or system problem, not the first visible symptom. Check for duplicate containers, conflicting rules, inherited styles, repeated markup, stale selectors, and temporary overrides before adding new code.
- Prefer root-cause fixes in shared components when the problem is systemic. Do not reflexively refactor unrelated areas; keep the scope proportional to the issue and regression risk.
- For substantial work, follow this sequence: **inspect → define the page or component goal → identify constraints → design the smallest coherent solution → implement → test → review visually and technically → remove obsolete code**.
- The agent may proactively identify and resolve closely related defects that materially affect the same goal, but should not expand a focused task into an unrelated redesign.
- Do not stop for routine implementation choices when the requested goal and constraints are clear. Ask for confirmation only when a decision is materially ambiguous, irreversible, or dependent on unavailable business facts.
- When product or industry details are incomplete, create a realistic and clearly qualified structure rather than vague filler. Mark facts that require later verification; never disguise assumptions as confirmed specifications.
- After iterative UI work, inspect the affected module for redundant selectors, dead code, duplicate media queries, contradictory states, and temporary patches. Consolidate or remove them.
- Add or update tests, content lints, and configuration checks when available. At minimum, ensure the Hugo build succeeds for all configured languages and inspect key responsive states.
- Document meaningful changes through clear code, focused comments where necessary, and commit or change notes explaining what changed, why, and how it was verified.

## Definition of Done
- The site builds successfully for every configured language, with no new template, content, asset, or console errors.
- The requested feature or correction works at the root level rather than being visually patched in one viewport or page.
- The result helps the customer understand, judge, trust, navigate, or act more effectively.
- Content is customer-facing, factually responsible, and consistent across related pages and languages.
- Layout, hierarchy, interaction states, alignment, spacing, typography, and responsive behavior have been visually reviewed at representative desktop and mobile widths.
- SEO, accessibility, metadata, internal linking, image treatment, and performance implications have been considered where relevant.
- No unnecessary dependency, duplicated component, obsolete selector, or unexplained one-off exception remains.
- The resulting implementation is simpler or at least no harder to understand, maintain, and extend than before.
- The next human or AI developer can identify where the change lives, why it exists, and how to extend it safely.

---
> Source: [chnsdly/NuvonFlow](https://github.com/chnsdly/NuvonFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
