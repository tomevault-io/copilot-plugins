## careflow

> The application uses SQLite as its database system, with the following configuration:

# Database Configuration Guide

The application uses SQLite as its database system, with the following configuration:

## Database Location
- The database file is located at: `src/main/resources/database.db`
- Connection URL: `jdbc:sqlite:src/main/resources/database.db`

## Database Management
- Managed by [DatabaseManager.java](mdc:src/main/java/edu/careflow/manager/DatabaseManager.java)
- Uses singleton pattern for database connection management
- Timezone is set to GMT+8

## Key Features
- Local SQLite database (no server required)
- Automatic connection management
- Transaction support
- Auto-commit enabled by default

## Database Structure
The database contains the following main tables:
- `patients` - Patient information
- `doctors` - Doctor information
- `nurses` - Nurse information
- `appointments` - Appointment scheduling
- `prescriptions` - Medical prescriptions
- `vitals` - Patient vital signs
- `users` - User accounts and authentication
- `billing` - Financial records
- `roles` and `permissions` - Access control

## Data Access
- All database operations are performed through DAO (Data Access Object) classes
- Located in `src/main/java/edu/careflow/repository/dao/`
- Each entity has its corresponding DAO class

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nesqyk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Nesqyk)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
