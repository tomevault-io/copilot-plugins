## terminal-console-operations

> -When running PowerShell commands, always use the full path to executables


-When running PowerShell commands, always use the full path to executables
-Use .\gradlew.bat instead of .\gradlew to avoid "Open with" popup
-For gradle commands, use the .bat extension explicitly: .\gradlew.bat :module:task
-When running commands that might trigger file associations, use the full executable path
-For Java compilation, use .\gradlew.bat compileJava instead of .\gradlew compileJava
-If a command fails with "Open with" popup, retry with the .bat extension

---
> Source: [tollara/tollara-sdk](https://github.com/tollara/tollara-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
