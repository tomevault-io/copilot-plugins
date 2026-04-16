## portfoliov2

> "description": "Preferences for how files should be handled by AI",

{
  "rules": {
    "file_handling": {
      "description": "Preferences for how files should be handled by AI",
      "enabled": true,
      "prefer_edits_threshold": "30%",
      "full_replacement_threshold": "70%",
      "exclude_patterns": [
        "node_modules/**",
        "dist/**",
        ".astro/**",
        ".vercel/**",
        ".git/**",
        "*.lock",
        "package-lock.json",
        "yarn.lock",
        ".DS_Store",
        "public/assets/**"
      ],
      "always_replace": [
        "*.svg",
        "*.json"
      ]
    },
    "context": {
      "description": "Context window settings for AI understanding",
      "enabled": true,
      "default_context_window": 100,
      "file_type_context": {
        "*.astro": 150,
        "*.tsx": 100,
        "*.jsx": 100,
        "*.css": 50
      }
    },
    "languages": {
      "description": "Language-specific settings",
      "enabled": true,
      "astro": {
        "treat_as_first_class": true,
        "understand_frontmatter": true,
        "mixed_content_handling": true,
        "component_aware": true
      },
      "typescript": {
        "strict_type_checking": true,
        "react_aware": true
      }
    },
    "explanations": {
      "description": "Preferences for AI explanations",
      "enabled": true,
      "verbosity": "concise",
      "include_reasoning_for_complex_changes": true,
      "highlight_important_changes": true,
      "max_diff_size": 100
    },
    "diff_display": {
      "description": "Preferences for diff displays",
      "enabled": true,
      "context_lines": 3,
      "syntax_highlighting": true,
      "collapse_unchanged_sections": true,
      "max_diff_size_lines": 300
    },
    "project": {
      "description": "Project-specific customizations",
      "enabled": true,
      "framework": "astro",
      "component_libraries": [
        "react",
        "tailwindcss"
      ],
      "styling": "tailwind"
    },
    "editor": {
      "description": "Editor behavior preferences",
      "enabled": true,
      "auto_import": true,
      "format_on_edit": true,
      "suggest_refactors": true
    },
    "commit_message_guidelines": {
      "description": "Guidelines for creating conventional commit messages.",
      "enabled": true,
      "format": {
        "description": "The format for commit messages using the conventional commits spec.",
        "body": "[optional scope]: \n\n[optional body]\n\n[optional footer(s)]"
      },
      "rules": [
        {
          "description": "Always suggest a conventional commit with a type and optional scope in lowercase letters."
        },
        {
          "description": "Keep the commit message concise and within 60 characters."
        },
        {
          "description": "Ensure the commit message is ready to be pasted into the terminal without further editing."
        },
        {
          "description": "Provide the full command to commit, not just the message."
        }
      ],
      "examples": [
        {
          "prompt": "<diff_context> /commit",
          "response": "git commit -m 'feat: add responsive navbar with TailwindCSS'"
        }
      ]
    },
    "development_guidelines": {
      "description": "Guidelines for developing code with Astro, TypeScript, and TailwindCSS.",
      "enabled": true,
      "rules": [
        {
          "description": "Enforce strict TypeScript settings, ensuring type safety across the project."
        },
        {
          "description": "Use TailwindCSS for all styling, keeping the utility-first approach in mind."
        },
        {
          "description": "Ensure Astro components are modular, reusable, and maintain a clear separation of concerns."
        }
      ]
    },
    "coding_style": {
      "description": "Guidelines for maintaining consistent coding style.",
      "enabled": true,
      "rules": [
        {
          "description": "Code must start with path/filename as a one-line comment."
        },
        {
          "description": "Comments should describe purpose, not effect."
        },
        {
          "description": "Prioritize modularity, DRY principles, and performance."
        }
      ]
    },
    "custom_slash_commands": {
      "description": "Custom slash commands.",
      "enabled": true,
      "commands": [
        {
          "name": "/commit",
          "description": "Generate a Git commit message using the conventional commits spec.",
          "enabled": true
        }
      ]
    }
  }
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/scrimmie) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
