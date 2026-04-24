## new-sport-timer

> A Flutter project is created to display the exercise time counter.

A Flutter project is created to display the exercise time counter.


The generated code must not contain comments; it should be self-explanatory through the naming of classes/functions/variables.
Support will only be provided for Android and iOS.

Architecture:
- The clean architecture pattern will be applied, consisting of the data, domain, and presentation layers.
- Provider will be used for state management.
- In the data layer, there will be data_mappers, repositories, DTOs, and data_sources.
- The domain layer will contain the business logic. Each screen to be displayed is considered a feature, so a folder will be created for each screen. Inside this folder, there may be one or more providers depending on the situation. This layer will also include entities, which are the data models used by the domain layer and passed to the presentation layer.
- The presentation layer is responsible only for the UI. Each screen is a page, and each page will have one or more views that compose it.
- A shared folder will also be created to extract common logic.
- A config folder will be created for necessary configurations.
- A di folder will be created for dependency injection.

Example of the structure: (Structure was mentioned, but no specific example was included in the original text.)

lib
---config
    app_config.dart
---data
    ---datasource
    ---dtos
    ---repositories
    data_mappers.dart
---domain
    ---login
        ---providers
        ---entities
    ---profile
        ---providers
        ---entities
    ---products
        ---providers
        ---entities
    ---user
        ---providers
        ---entities
---presentation
    ---home
        home_page.dart
        sale_view.dart
        products_view.dart
    ---login
        login_page.dart
---shared
    ---domain
    ---exceptions
    ---extensions
    ---mixins
    ---navigation
    ---network
---di
    di.dart
    di.config.dart
    di_module.dart
main.dart


State Management
As I mentioned before, we will use Provider for the state management

Data
We will use the freezed and freezed_annotation package to data creation, so we will have .freezed.dart and .g.dart

Networks
We will use the dio package for network calls

DataBase
We will not store anything, database will not be used

Navigation
Is mandatory use go_router for navigation

DI
We will use get_it for dependency injection

Lints
We will use the flutter_lints witha latest version and lint rules

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexaniko88) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
