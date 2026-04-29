## veo3-app

> validateStructure: true

# project.rules.yaml
project:
  name: veo3-video-generator
  version: 0.1.0
  description: "AI Video Generation App with Veo3/Fal.ai integration"
  
  structure:
    backend:
      language: typescript
      framework: express
      promptRules: "./ai.rules.json"
      validators:
        - promptValidator
        - turkishDialogueValidator
    frontend:
      language: dart
      framework: flutter
      
  build:
    backend:
      command: "npm run build"
      output: "./dist"
    frontend:
      command: "flutter build"
      output: "./build"
      
  lint:
    backend:
      - eslint
      - prettier
    frontend:
      - flutter analyze
      - dart format
      
  test:
    backend:
      framework: jest
      coverage: 80
    frontend:
      framework: flutter_test
      coverage: 80
      
  promptValidation:
    enforceRules: true
    validateStructure: true
    checkTurkishDialogue: true

    maxLength: 800

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Furkiti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
