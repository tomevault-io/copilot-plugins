## liberap

> **CRITICAL**: Never manually edit `angular/src/app/proxy/trading/models.ts` or any other file inside the `proxy` directory. These files are automatically generated from the backend.

# Project Rules & Workflow

## 🚀 Frontend Proxy Generation
**CRITICAL**: Never manually edit `angular/src/app/proxy/trading/models.ts` or any other file inside the `proxy` directory. These files are automatically generated from the backend.

### Correct Workflow for Adding Properties:
1.  **Modify Backend DTO**: Add or modify the property in the corresponding `.cs` file in the `Verge.Application.Contracts` project.
2.  **Compile & Run Backend**: Ensure the backend builds and the `Verge.HttpApi.Host` project is running (required for Swagger/OpenAPI discovery).
3.  **Generate Proxy**: Run the following command from the `angular` directory:
    ```bash
    abp generate-proxy -t ng
    ```
4.  **Verify**: Check the modified `models.ts` in the frontend to ensure the property is present.

## 🛠 Backend Standards
- **DataJson**: Always ensure `DataJson` in `AnalysisLog` has a default value of `"{}"` to avoid database `null` constraint violations.
- **SignalR**: Use `VergeAlertDto` to broadcast real-time updates (like Score decay) to the dashboard.

## 🎨 UI/UX Design
- **Readability**: Avoid dark text on dark backgrounds. Use `text-light` or dynamic colors (`[style.color]`) for console logs.
- **Real-time**: The dashboard should listen to SignalR alerts to update the session state without waiting for the next polling cycle.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nicolas-Arteaga) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
