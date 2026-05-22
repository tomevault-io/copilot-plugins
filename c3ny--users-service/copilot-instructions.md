## business-overview

> Business Overview


- The users-service is a microservice that belongs to the Sangue Solidario's project.
- It manages all user accounts, authentication, and profiles for both blood donors and healthcare institutions.
- The service handles user registration with two distinct person types: DONOR (individual blood donors) and COMPANY (hospitals, blood centers, health institutions).
- For donors, the service stores additional information like CPF, blood type, and birth date.
- For companies, the service stores CNPJ, institution name, and CNES (National Health Establishment Code).
- The service provides authentication via JWT tokens and password management (hashing, validation, password changes).
- Users can upload avatar images that are stored in a temporary directory with the path saved in the database.
- The application essentially follows a dual-profile model, supporting both individual users (donors) and institutional users (companies) with different data requirements for each.

---
> Source: [c3ny/users-service](https://github.com/c3ny/users-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
