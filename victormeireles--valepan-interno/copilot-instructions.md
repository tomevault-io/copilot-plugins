## valepan-interno

> <file_length_and_structure>

<file_length_and_structure>
- Never allow a file to exceed 500 lines.
- If a file approaches 400 lines, break it up immediately.
- Treat 1000 lines as unacceptable, even temporarily.
- Use folders and naming conventions to keep small files logically grouped.
</file_length_and_structure>

<oop_first>
- Every functionality should be in a dedicated class, struct, or protocol, even if it’s small.
- Favor composition over inheritance, but always use object-oriented thinking.
- Code must be built for reuse, not just to “make it work.”
</oop_first>

<single_responsibility_principle>
- Every file, class, and function should do one thing only.
- If it has multiple responsibilities, split it immediately.
- Each view, manager, or utility should be laser-focused on one concern.
</single_responsibility_principle>

<modular_design>
- Code should connect like Lego – interchangeable, testable, and isolated.
- Ask: “Can I reuse this class in a different screen or project?” If not, refactor it.
- Reduce tight coupling between components. Favor dependency injection or protocols.
</modular_design>

<manager_and_coordinator_patterns>
- Use ViewModel, Manager, and Coordinator naming conventions for logic separation:
  – UI logic → ViewModel
  – Business logic → Manager
  – Navigation/state flow → Coordinator
- Never mix views and business logic directly.
</manager_and_coordinator_patterns>

<function_and_class_size>
- Keep functions under 30–40 lines.
- If a class is over 200 lines, assess splitting into smaller helper classes.
</function_and_class_size>

<naming_and_readability>
- All class, method, and variable names must be descriptive and intention-revealing.
- Avoid vague names like data, info, helper, or temp.
</naming_and_readability>

<scalability_mindset>
- Always code as if someone else will scale this.
- Include extension points (e.g., protocol conformance, dependency injection) from day one.
</scalability_mindset>

<avoid_god_classes>
- Never let one file or class hold everything (e.g., massive ViewController, ViewModel, or Service).
- Split into UI, State, Handlers, Networking, etc.
</avoid_god_classes>

<database_and_rls>
- ALWAYS enable Row Level Security (RLS) on ALL new tables in Supabase.
- When creating RLS policies, ALWAYS use (SELECT auth.uid()) and (SELECT is_admin()) instead of auth.uid() and is_admin() directly to avoid performance issues.
- Pattern for RLS policies:
  * SELECT: All authenticated users can read (USING (true))
  * INSERT: All authenticated users can create (WITH CHECK (true)) OR specific conditions
  * UPDATE: Users can update their own records OR admins (USING (usuario_id = (SELECT auth.uid()) OR (SELECT is_admin())))
  * DELETE: Only admins (USING ((SELECT is_admin())))
- For nested policies (e.g., child tables), verify permissions through parent table relationships.
- Never create tables without RLS enabled and proper policies.
- Avoid SECURITY DEFINER on views unless absolutely necessary (creates warnings).
</database_and_rls>

---
> Source: [victormeireles/valepan-interno](https://github.com/victormeireles/valepan-interno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
