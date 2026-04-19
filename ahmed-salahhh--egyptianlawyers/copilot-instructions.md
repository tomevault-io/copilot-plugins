## egyptianlawyers

> - Always write clean, production-ready code.

# General Rules
- Always write clean, production-ready code.
- Create a single generic `NotFoundError` instead of a new one for each entity.

# .NET Backend Rules (Vertical Slice & MediatR)
- Architecture: Use Vertical Slice Architecture. Organize the system into resource slices inside a `Features` folder.
- Endpoint Pattern: Use Minimal APIs. Every slice must implement the `IEndpoint` interface for automatic registration.
- Feature Structure: Keep the Endpoint mapping, MediatR Request (Command/Query), and Handler in a single file per feature slice.
- CQRS: Use MediatR for all business logic. Controllers/Endpoints should only send commands/queries to the mediator.
- Database: Use EF Core with SQL Server. Keep core Domain Entities in a central `Domain/Entities` folder.
- Validation: Use FluentValidation for request validation within each slice.

# Frontend Rules
- React Native: Use Expo and functional components.
- React Web: Use functional components and Tailwind CSS for styling.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ahmed-Salahhh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
