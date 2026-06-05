## backend-service-wiring

> Service.go wiring conventions for qubership-apihub-backend


# Service.go Wiring

- Add new repositories, services, and controllers at the **end** of their corresponding sections in `Service.go`.
- Use `log.Fatalf` for fail-fast fatal errors during wiring/startup in `Service.go` when initialization cannot continue.

---
> Source: [Netcracker/qubership-apihub-backend](https://github.com/Netcracker/qubership-apihub-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
