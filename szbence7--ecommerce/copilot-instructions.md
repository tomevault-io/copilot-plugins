## ecommerce

> "name": "Ecommerce Site PHP Rules",

{
    "name": "Ecommerce Site PHP Rules",
    "rules": [
      {
        "language": "php",
        "use": "pdo",
        "notes": [
          "Use PDO for secure database interactions with prepared statements.",
          "Ensure PHP code follows PSR-12 coding standards.",
          "Implement strict typing using declare(strict_types=1);"
        ],
        "bestPractices": [
          "Always use descriptive and self-explanatory variable names.",
          "Implement proper error handling using try-catch blocks.",
          "Write secure code to prevent SQL injection and other vulnerabilities.",
          "When db alteration needed, just give me the sql query and i will import it in the database"
        ]
      },
      {
        "language": "html",
        "framework": "bootstrap",
        "icons": "lucide",
        "notes": [
          "Adopt Bootstrap's grid system for responsive design.",
          "Utilize Lucide icons as per design requirements."
        ],
        "bestPractices": [
          "Maintain semantic HTML for better accessibility.",
          "Ensure compatibility with different browsers by adhering to Bootstrap classes."
        ]
      }
    ]
  }

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/szbence7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
