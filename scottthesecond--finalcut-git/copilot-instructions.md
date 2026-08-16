## logging

> logging functions are defined in logs.sh.

logging functions are defined in logs.sh.

log_message simply writes a log to our log file.
handle_error does the same, but also shows an alert to the user and gives them the option to copy the logs to their desktop.

You'll use log_message most of the time, but use handle_error for fatal errors where we are stopping the whole operation/script.

---
> Source: [scottthesecond/finalcut-git](https://github.com/scottthesecond/finalcut-git) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
