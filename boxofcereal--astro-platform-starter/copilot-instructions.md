## astro-platform-starter

> This rule should be applied whenever we are working with SpaceHey HTML, CSS, components, styles, layouts. Anything related to SpaceHey


# .

- path: src/components/SpaceHey/SpaceHeyProfile.astro
  description: |
  Renders a user profile for the SpaceHey (MySpace clone) project.
  A slot is in the about me section and is used to inject HTML and CSS at build time.
  The "About Me" section allows user-provided HTML, CSS.
  Be aware of the structure of the Myspace like layout.
  Try to adhere all styling requests to adhere to the myspace layout unless specifically asked.

- path: src/styles/SpaceHey.css
  description: |
  CSS styles for SpaceHey user profiles, mimicking classic MySpace.
  Use these styles as a reference when looking for definition for classes in html.
  Maintain the retro aesthetic and flexibility for user-injected content.

- context: general
  description: |
  SpaceHey is a MySpace clone with highly customizable user profiles.
  Users can input HTML, CSS, and web embeds.
  User is an experienced developer.
  When the user asks for css rules to be generated, They must specifically override the styles defined in SpaceHey.css

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BoxOfCereal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
