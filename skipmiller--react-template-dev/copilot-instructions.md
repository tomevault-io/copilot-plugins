## react-template-dev

> - description: "Allow Cursor to access .env.example files but never real .env files."

# Define rules for how Cursor interacts with your project

rules:
  - description: "Allow Cursor to access .env.example files but never real .env files."
    include:
      - "*.example"  # Include example environment files for instructional purposes
    exclude:
      - ".env"
      - ".env.*"  # Exclude all real environment files

  - description: "Notify users about potential exposure of sensitive data."
    notify_on_detection:
      patterns:
        - "API_KEY="
        - "SECRET_KEY="
        - "PASSWORD="
        - "TOKEN="
        - "CREDENTIAL="
        - "[0-9]{3}-[0-9]{2}-[0-9]{4}"  # Example pattern for SSNs
      message: |
        "Sensitive data detected. Ensure that real secrets are excluded using '.cursorignore'. 
        Update the ignore file if necessary and confirm no secrets are exposed."

  - description: "Provide instructions for securing sensitive data."
    on_detected_sensitive_files:
      actions:
        - "Remind the user to verify '.cursorignore' and exclude sensitive files."
        - "Suggest enabling Privacy Mode in Cursor settings to prevent accidental transmission."

  - description: "Prevent AI from altering or suggesting changes to environment variables in example files."
    restrict_modifications:
      include:
        - "*.example"  # Allow reading but restrict modifications to example environment files

reminders:
  - frequency: "on_project_open"
    message: |
      "Reminder: Ensure that sensitive files (e.g., .env) are excluded from AI processing. 
      Use '.cursorignore' to specify exclusions and review regularly." 

---
> Source: [skipmiller/react.template.dev-](https://github.com/skipmiller/react.template.dev-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
