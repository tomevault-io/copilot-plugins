## personal-finance

> This is a Laravel-based personal finance management application built with DDEV for local development. The application helps users track their income, expenses, and manage recurring transactions.

# Personal Finance App - AI Agent Instructions

## Project Overview

This is a Laravel-based personal finance management application built with DDEV for local development. The application helps users track their income, expenses, and manage recurring transactions.

## Tech Stack

- **Framework**: Laravel 11.x
- **PHP Version**: 8.3
- **Database**: MariaDB 10.11
- **Web Server**: nginx-fpm
- **Frontend**: Blade templates with AdminLTE theme
- **Development Environment**: DDEV
- **Package Manager**: Composer 2.x

## DDEV Commands

All commands must be executed inside the DDEV container:

```bash
# Execute PHP/Artisan commands
ddev exec --dir="/var/www/html" php artisan <command>

# Execute bash commands
ddev exec --dir="/var/www/html" bash -c "<command>"

# Access DDEV container shell
ddev ssh

# Start/Stop DDEV
ddev start
ddev stop

# View logs
ddev logs
```

## Project Structure

```
app/
├── Console/Commands/
│   └── ProcessRecurringTransactions.php  # Handles recurring transaction processing
├── Http/Controllers/
│   ├── DashboardController.php           # Dashboard with monthly summary
│   ├── ExpenseController.php             # Expense CRUD operations
│   ├── IncomeController.php              # Income CRUD operations
│   └── ReportController.php              # Financial reports
├── Models/
│   ├── Expense.php                       # Expense model with recurring support
│   ├── Income.php                        # Income model with recurring support
│   ├── RecurringExpense.php              # Legacy recurring expense model
│   └── RecurringTransactionLog.php       # Tracks processed recurring transactions
database/migrations/
├── 2025_08_09_222713_create_expenses_table.php
├── 2025_08_09_222713_create_incomes_table.php
├── 2025_08_09_232446_add_recurring_fields_to_expenses_table.php
├── 2025_08_09_232446_add_recurring_fields_to_incomes_table.php
└── 2026_02_14_125758_create_recurring_transaction_logs_table.php
resources/views/
├── dashboard.blade.php
├── expenses/
├── incomes/
└── layouts/
routes/
├── web.php                               # Web routes
└── console.php                           # Scheduled tasks
```

## Key Features

### 1. Income Management
- CRUD operations for income entries
- Support for recurring income
- Fields: amount, source, date, notes, recurring settings

### 2. Expense Management
- CRUD operations for expense entries
- Support for recurring expenses
- Fields: amount, category, date, notes, recurring settings

### 3. Recurring Transactions
- **Automatic Processing**: Daily scheduled task processes recurring transactions
- **Recurring Types**: daily, weekly, monthly, yearly
- **Duplicate Prevention**: Tracks processed transactions to avoid duplicates
- **End Date Support**: Optional end date for recurring transactions

### 4. Dashboard
- Monthly income summary
- Monthly expense summary
- Net balance calculation
- Current month view by default

### 5. Reports
- Financial reports (implementation in ReportController.php)

## Database Schema

### expenses table
- `id`: Primary key
- `amount`: Decimal(10,2)
- `category`: String
- `date`: Date
- `notes`: Text (nullable)
- `is_recurring`: Boolean (default: false)
- `recurring_type`: String (nullable) - 'daily', 'weekly', 'monthly', 'yearly'
- `recurring_day`: Integer (nullable) - Day of week/month or MMDD for yearly
- `recurring_end_date`: Date (nullable)

### incomes table
- `id`: Primary key
- `amount`: Decimal(10,2)
- `source`: String
- `date`: Date
- `notes`: Text (nullable)
- `is_recurring`: Boolean (default: false)
- `recurring_type`: String (nullable) - 'daily', 'weekly', 'monthly', 'yearly'
- `recurring_day`: Integer (nullable) - Day of week/month or MMDD for yearly
- `recurring_end_date`: Date (nullable)

### recurring_transaction_logs table
- `id`: Primary key
- `transaction_type`: String ('expense' or 'income')
- `transaction_id`: Foreign key to expense/income
- `processed_for_date`: Date
- `amount`: Decimal(10,2)
- Unique constraint on: transaction_type, transaction_id, processed_for_date

## Recurring Transactions System

### How It Works

1. **Definition**: Users mark expenses/incomes as recurring with:
   - `is_recurring`: true
   - `recurring_type`: daily/weekly/monthly/yearly
   - `recurring_day`: Day specification (0-6 for weekly, 1-31 for monthly, MMDD for yearly)
   - `recurring_end_date`: Optional end date

2. **Processing**: Command `recurring:process` runs daily at 00:01
   - Finds all active recurring transactions
   - Checks if they match today's date based on recurring rules
   - Creates new expense/income entries
   - Logs processing to prevent duplicates

3. **Manual Processing**: Run for specific date
   ```bash
   ddev exec --dir="/var/www/html" php artisan recurring:process --date=2026-02-15
   ```

### Scheduler Setup

The Laravel scheduler is configured in `routes/console.php`:
- Runs `recurring:process` daily at 00:01
- Processes all pending recurring transactions

**Important**: For the scheduler to work in production, add this to crontab:
```bash
* * * * * cd /path-to-project && php artisan schedule:run >> /dev/null 2>&1
```

For DDEV development, you can manually trigger:
```bash
ddev exec --dir="/var/www/html" php artisan schedule:run
```

## Development Workflow

### Adding New Features

1. **Create Migration**:
   ```bash
   ddev exec --dir="/var/www/html" php artisan make:migration create_table_name
   ```

2. **Create Model**:
   ```bash
   ddev exec --dir="/var/www/html" php artisan make:model ModelName
   ```

3. **Create Controller**:
   ```bash
   ddev exec --dir="/var/www/html" php artisan make:controller ControllerName
   ```

4. **Run Migrations**:
   ```bash
   ddev exec --dir="/var/www/html" php artisan migrate
   ```

### Testing Commands

```bash
# List all scheduled tasks
ddev exec --dir="/var/www/html" php artisan schedule:list

# Test recurring processing
ddev exec --dir="/var/www/html" php artisan recurring:process

# Check database
ddev exec --dir="/var/www/html" php artisan db:show
```

## Common Tasks

### Run Migrations
```bash
ddev exec --dir="/var/www/html" php artisan migrate
```

### Clear Cache
```bash
ddev exec --dir="/var/www/html" php artisan cache:clear
ddev exec --dir="/var/www/html" php artisan config:clear
ddev exec --dir="/var/www/html" php artisan view:clear
```

### Access Database
```bash
ddev exec --dir="/var/www/html" php artisan tinker
```

### View Routes
```bash
ddev exec --dir="/var/www/html" php artisan route:list
```

## Important Notes for AI Agents

1. **Always use DDEV commands**: Never run commands directly outside DDEV
2. **Follow Laravel conventions**: Use Eloquent ORM, follow PSR standards
3. **Comment all code**: Add meaningful PHPDoc comments to all functions
4. **Use existing patterns**: Follow the established controller/model patterns
5. **Test after changes**: Run the application and test functionality
6. **Recurring transactions**: Auto-generated transactions have `is_recurring=false`
7. **Database changes**: Always create migrations, never modify database directly
8. **Scheduler**: Remember to set up cron for production deployments

## URL Access

- **Local URL**: https://personal-finance-app.ddev.site
- **Dashboard**: https://personal-finance-app.ddev.site/dashboard
- **Incomes**: https://personal-finance-app.ddev.site/incomes
- **Expenses**: https://personal-finance-app.ddev.site/expenses
- **Reports**: https://personal-finance-app.ddev.site/reports

## Authentication

The application uses Laravel Breeze for authentication:
- Registration
- Login/Logout
- Password reset
- Email verification
- Profile management

All routes under `/dashboard`, `/incomes`, `/expenses`, `/reports` require authentication.

## Future Enhancements

Consider implementing:
- Budget tracking and alerts
- Category management
- Multi-currency support
- Export to CSV/PDF
- Charts and visualizations
- Mobile responsive improvements
- API for mobile apps
- Notification system for recurring transactions
- Bank integration
- Receipt upload and OCR

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hiteshmakvana)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hiteshmakvana)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
