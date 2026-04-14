## tickettoride

> Structure du projet et organisation du code


## **Organisation du Code et Conventions - Backend "J'ai une place"**

Ce document définit les règles et les standards à suivre pour l'organisation du code source du backend de l'application "J'ai une place".

### **Architecture en Couches (Clean Architecture)**

Le projet suit une architecture en couches stricte avec une séparation claire des préoccupations, garantissant la maintenabilité et la testabilité du code.

1.  **`J1P.Domain`** : Le cœur du projet. Contient les entités métier et les énumérations. N'a aucune dépendance vers les autres couches.
2.  **`J1P.Application`** : La couche de logique métier. Elle contient les DTOs, les interfaces, la logique des services (cas d'usage), les validateurs et les profils de mapping.
3.  **`J1P.Infrastructure`** : La couche d'accès aux données et aux services externes. Elle contient les implémentations concrètes des repositories et les services d'infrastructure (ex: email, stockage de fichiers).
4.  **`J1P.Api`** : La couche de présentation. Elle expose les points d'entrée de l'API (contrôleurs) et gère la configuration du pipeline HTTP.

### **Organisation des Dossiers**

#### `J1P.Domain`
*   `Enums/` : Toutes les énumérations spécifiques au domaine (ex: `UserStatus`).
*   `Entities/` : Toutes les entités du domaine (classes POCO comme `User`, `Client`, etc.).

#### `J1P.Application`
*   `DTOs/` : Objets de Transfert de Données (DTOs) utilisés par les contrôleurs.
*   `Interfaces/` : Abstractions (interfaces) des services et repositories.
    *   `Repositories/` : Interfaces des repositories (ex: `IUserRepository`).
    *   `Services/` : Interfaces des services métier (ex: `IAuthService`).
*   `Mapping/` : Profils AutoMapper pour le mapping entre les entités et les DTOs.
*   `Features/` (ou `Services/`) : Implémentations des services contenant la logique métier.
*   `Validators/` : Validateurs FluentValidation pour les DTOs entrants.

#### `J1P.Infrastructure`
*   `Persistence/` : Configuration de l'accès aux données.
    *   `Data/` : Le `DbContext` de l'application.
    *   `Configurations/` : Configurations des entités EF Core (Fluent API).
    *   `Seeding/` : La logique du `DataSeeder` pour l'amorçage des données de test.
*   `Repositories/` : Implémentations concrètes des repositories définis dans `J1P.Application`.
*   `Migrations/` : Dossier généré automatiquement par EF Core pour les migrations de base de données.
*   `Services/` : Implémentations de services spécifiques à l'infrastructure (ex: un futur client pour l'API Supabase Storage).

#### `J1P.Api`
*   `Controllers/` : Contrôleurs RESTful, un par ressource principale.
*   `Middlewares/` : Middlewares personnalisés (ex: un futur gestionnaire d'exceptions global).
*   `Extensions/` : Méthodes d'extension pour simplifier la configuration dans `Program.cs` (ex: `services.AddInfrastructure()`).

### **Conventions de Nommage**

1.  **Interfaces** : Préfixées par "I" (ex: `IUserService`).
2.  **DTOs** : Suffixés par "Dto" (ex: `UserDto`, `RegisterUserDto`).
3.  **Validateurs** : Suffixés par "Validator" (ex: `RegisterUserDtoValidator`).
4.  **Repositories** : Suffixés par "Repository" (ex: `UserRepository`).
5.  **Services** : Suffixés par "Service" (ex: `AuthService`).
6.  **Contrôleurs** : Suffixés par "Controller" (ex: `UsersController`).

### **Principes Fondamentaux**

1.  **Séparation des Préoccupations (SoC)** : Les interfaces doivent être séparées de leurs implémentations pour faciliter le mocking et la testabilité.
2.  **Principe d'Inversion de Dépendances** : Les couches externes dépendent des abstractions des couches internes, jamais l'inverse. `Api` dépend de `Application`, `Infrastructure` dépend de `Application`, `Application` dépend de `Domain`.
3.  **Injection de Dépendances (DI)** : Utiliser systématiquement le conteneur d'injection de dépendances de .NET pour fournir les services et repositories à leurs consommateurs.
4.  **Validation à l'Entrée** : Utiliser **FluentValidation** pour valider les DTOs entrants au plus près de la couche de présentation (API).
5.  **Mapping Explicite** : Utiliser **AutoMapper** pour les conversions entre les Entités et les DTOs afin de garder les contrôleurs et services propres.

### **Règles Impératives**

1.  Les interfaces doivent toujours être placées dans le dossier `Interfaces/` du projet `JHP.Application`.
2.  Les DTOs doivent être dans `JHP.Application/DTOs/`. Jamais dans le projet `JHP.Api`.
3.  Les validateurs FluentValidation doivent être dans `JHP.Application/Validators/`.
4.  Chaque classe, interface ou énumération doit être dans son propre fichier, nommé de manière identique.
5.  Les entités du domaine (`JHP.Domain/Entities/`) ne doivent avoir aucune dépendance vers les autres couches du projet. Elles ne connaissent rien de la base de données, de l'API ou des services.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/galvin59) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
