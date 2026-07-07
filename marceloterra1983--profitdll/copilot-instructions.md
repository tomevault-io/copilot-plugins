## profitdll

> - NEVER expose sensitive financial data (credentials, API keys, trading secrets) in code or logs

# Cursor Rules - ProfitDLL Project

## Security & Authorization Rules

### Financial Data Protection
- NEVER expose sensitive financial data (credentials, API keys, trading secrets) in code or logs
- All monetary values MUST use decimal precision (no float/double)
- Transaction logs MUST be structured with correlation IDs
- Failed operations MUST throw explicit exceptions (no silent failures)

### Code Access Control
- Read-only access to production configuration files
- Write access requires explicit approval for:
  - Database schema changes
  - Authentication/authorization changes
  - Financial calculation logic
  - Network security configurations

### File System Permissions
- Allow: Reading project files, writing to development directories
- Allow: Automatic deletion of Cursor IDE log files (`.cursor/debug.log`, `.cursor/*.log`)
- Deny: Modifying production .env files without approval
- Ask: Deleting files, modifying critical system files

### Command Execution
- Allow: Build commands, test execution, development server startup
- Deny: Production deployment commands, database migrations without backup
- Ask: System-level commands, network configuration changes

### Network & API Access
- Allow: Local development servers (localhost:5000, localhost:3101, localhost:5173)
- Deny: External API calls without explicit authorization
- Ask: Network configuration changes, firewall modifications

## Project-Specific Rules

### Windows Native Environment
- This project requires Windows native execution (no WSL2/Docker for ProfitDLL.dll)
- All paths should use Windows format when interacting with DLL
- PowerShell commands are preferred for Windows-specific operations

### Financial Precision
- Use decimal.js or big.js for monetary calculations
- Never use float/double for currency values
- All prices should be stored as integers (centavos) or decimal strings

### Logging Requirements
- All financial transactions must have structured logs (JSON)
- Include timestamp, correlation ID, and operation type
- Never log sensitive credentials or API keys

---
> Source: [marceloterra1983/PROFITDLL](https://github.com/marceloterra1983/PROFITDLL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
