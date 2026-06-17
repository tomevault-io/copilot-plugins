## boardgameleague

> This skill is used for adding or updating Entity Framework support in the project.

# Skill Configuration

## Entity Framework Skill

This skill is used for adding or updating Entity Framework support in the project.

- Update models with EF attributes and navigation properties.
- Add a `DbContext` implementation: `Models/BoardGameLeagueDbContext.cs`.
- Add a concrete repository implementation: `Models/EfLeagueRepository.cs`.
- Register EF services in `Program.cs` and configure a database connection string.
- Seed sample data on startup when the database is empty.

## List Page Skill

This skill is used to add list and details pages to the application.

- Add controller actions for `Index` and `Details`.
- Create supporting views under `Views/<ControllerName>/`.
- Register custom routes using attribute routing for list and detail pages.

## Edit/Create Form Skill

This skill is used to add edit/create forms for entities.

- Add GET and POST action pairs for create/edit operations.
- Use `asp-for` tag helpers in Razor views.
- Add validation attributes to the model.
- Update repository to persist changes through EF.

## Notes

- The EF skill is demonstrated by replacing the `MockLeagueRepository` with `EfLeagueRepository`.
- The list page skill is demonstrated by exposing custom routes such as `/teams`, `/players`, `/games`, `/tournaments`, `/matches`, and `/venues`.
- The edit/create form skill is documented as a supported extension point for future enhancements.

---
> Source: [GeneralBorN/BoardGameLeague](https://github.com/GeneralBorN/BoardGameLeague) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
