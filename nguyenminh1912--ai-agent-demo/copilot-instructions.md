## entity-class-conventions

> Sets the standards for entity class design including annotations, ID generation strategies, and relationship configurations for database interaction.

- Must annotate entity classes with @Entity.
- Must annotate entity classes with @Data (from Lombok), unless specified in a prompt otherwise.
- Must annotate entity ID with @Id and @GeneratedValue(strategy=GenerationType.IDENTITY).
- Must use FetchType.LAZY for relationships, unless specified in a prompt otherwise.
- Annotate entity properties properly according to best practices, e.g., @Size, @NotEmpty, @Email, etc.

---
> Source: [NguyenMinh1912/ai-agent-demo](https://github.com/NguyenMinh1912/ai-agent-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
