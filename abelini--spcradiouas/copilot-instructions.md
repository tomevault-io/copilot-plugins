## spcradiouas

> When generating code for this project, follow these rules strictly.

# AGENTS.md

## Coding Guidelines

When generating code for this project, follow these rules strictly.

### General Principles
- Prefer **Object-Oriented Programming (OOP)**.
- Follow **clean code** and **modern best practices**.
- Write **clear, maintainable, and strongly typed code** whenever the language allows it.
- Keep implementations simple and readable.

### Language Conventions
- All **identifiers must be written in English**:
  - Variables
  - Constants
  - Properties
  - Methods
  - Classes
  - Functions
- Use **camelCase** for:
  - variables
  - methods
  - functions
  - properties
- Use **PascalCase** for:
  - class names

Example:

```php
class UserRepository
{
    private DatabaseConnection $databaseConnection;

    public function __construct(DatabaseConnection $databaseConnection)
    {
        $this->databaseConnection = $databaseConnection;
    }

    public function findUserById(int $userId): ?User
    {
        // Implementation
    }
}
````

### PHP Requirements

When writing PHP code:

* Use **strict typing**.

```php
declare(strict_types=1);
```

Everything in this project is under the namespace SPC, avoid using App

```php
namespace SPC\Controller\Api;
```

* Code must be **fully typed**:

  * Method parameters
  * Return types
  * Properties

Example:

```php
private int $userId;
```

### File Structure

* **One class per file**
* File name must match the class name.

Example:

```
UserService.php
UserRepository.php
DatabaseConnection.php
```

### Comments

Avoid excessive or obvious comments.

#### Inline comments

* Use **short and simple comments**
* Only when the logic is not immediately clear

Example:

```php
// Prevent duplicate user creation
```

#### Method documentation (Required)

Every method must include a **DocBlock in Spanish** describing what it does. Use **English** for the method name.
Even when the description takes one line, use a minimum of three lines for the DocBlock.

Example:

```php
/**
 * Obtiene un usuario a partir de su identificador.
 */
public function findUserById(int $userId): ?User
{
}
```

### Summary of Mandatory Rules

* OOP architecture
* One class per file
* English identifiers
* camelCase naming
* Fully typed code
* `declare(strict_types=1);`
* Minimal inline comments
* Spanish DocBlocks for every method

```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abelini)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/abelini)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
