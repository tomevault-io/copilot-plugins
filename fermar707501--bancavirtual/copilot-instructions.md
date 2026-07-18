## bancavirtual

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Laravel 12 virtual banking system (Banco Virtual) with Livewire/Volt for reactive UI components. The application manages banking operations including client management, accounts, transactions, loans, and scheduled payments with role-based permissions.

## Development Commands

### Initial Setup
```bash
composer setup
```
Runs complete setup: installs composer dependencies, copies .env.example, generates app key, runs migrations, installs npm dependencies, and builds assets.

### Development Server
```bash
composer dev
```
Starts three concurrent services:
- PHP development server (port 8000)
- Queue listener
- Vite dev server

Or run individually:
```bash
php artisan serve           # Development server
php artisan queue:listen --tries=1    # Queue worker
npm run dev                 # Vite for assets
```

### Testing
```bash
composer test               # Run PHPUnit tests
php artisan test            # Alternative test command
php artisan test --filter=TestName  # Run specific test
```

### Building Assets
```bash
npm run build               # Production build
npm run dev                 # Development mode with hot reload
```

### Other Common Commands
```bash
php artisan migrate         # Run migrations
php artisan migrate:fresh --seed  # Fresh migrations with seeders
php artisan tinker          # Interactive console
php artisan pail            # View logs in real-time
vendor/bin/pint             # Code style fixer (Laravel Pint)
```

## Architecture

### Technology Stack
- **Backend**: Laravel 12 with PHP 8.2+
- **Frontend**: Livewire 3.x + Livewire Flux UI components
- **Authentication**: Laravel Fortify with 2FA support
- **Authorization**: Spatie Laravel Permission (roles & permissions)
- **Styling**: Tailwind CSS 4.x with Vite
- **Database**: Configured for MySQL/PostgreSQL (supports SQLite for testing)
- **PDF Generation**: DomPDF (barryvdh/laravel-dompdf)
- **Excel Export**: Maatwebsite Excel

### Core Domain Models

The system follows a hierarchical banking structure:

1. **Agency** (Agencia): Bank branches
   - Top-level organizational unit
   - Related to Users, Clients, Accounts, Transactions

2. **Client** (Cliente): Bank customers
   - Has personal information (DPI, NIT, address, phone)
   - Belongs to an Agency
   - Can have multiple Accounts and Loans
   - Supports soft deletes
   - Uses scopes: `active()`, `inactive()`

3. **Account** (Cuenta): Banking accounts
   - Types: `savings`, `checking`, `fixed_deposit`
   - Tracks balance, interest rate, status
   - Methods: `deposit()`, `withdraw()`, `hasSufficientBalance()`
   - Related to Client, Agency, Transactions
   - Supports soft deletes

4. **Transaction** (Transacción): All financial movements
   - Types: `deposit`, `withdrawal`, `transfer_out`, `transfer_in`, `loan_payment`, `reversal`
   - Records balance before/after, can be reversed
   - Tracks user who performed transaction and agency
   - Scopes: `deposits()`, `withdrawals()`, `transfers()`, `notReversed()`, `dateRange()`

5. **Loan** (Préstamo): Customer loans
   - Complex amortization calculations (French system)
   - Status: `pending`, `approved`, `rejected`, `active`, `paid`, `defaulted`
   - Portfolio status: `current`, `overdue`, `charged_off`
   - Methods: `calculateMonthlyPayment()`, `generatePaymentSchedule()`
   - Related to LoanPayment for payment history
   - Has analyst and approver users

6. **ScheduledPayment**: Recurring automated payments

7. **AuditLog**: Activity tracking for compliance

8. **SystemParameter**: Configurable system parameters
   - Categories: `interest_rates`, `thresholds`, `fees`
   - Data types: `string`, `integer`, `decimal`, `boolean`, `json`
   - Helper methods: `SystemParameter::get($key)`, `SystemParameter::set($key, $value)`
   - Examples: interest rates, low balance threshold, fee amounts

9. **User**: System users with roles/permissions
   - Belongs to an Agency
   - Uses Spatie Permission traits
   - Can be analysts or approvers for loans

### Key Relationships

```
Agency
  └─ hasMany: Users, Clients, Accounts, Transactions, Loans

Client
  ├─ belongsTo: Agency
  ├─ hasMany: Accounts
  └─ hasMany: Loans

Account
  ├─ belongsTo: Client, Agency
  ├─ hasMany: Transactions (as source)
  ├─ hasMany: Transactions (as destination via transfersReceived)
  └─ hasMany: ScheduledPayments

Transaction
  ├─ belongsTo: Account (source)
  ├─ belongsTo: Account (destination, nullable)
  ├─ belongsTo: User (performer)
  ├─ belongsTo: User (reverser, nullable)
  └─ belongsTo: Agency

Loan
  ├─ belongsTo: Client, Agency
  ├─ belongsTo: User (analyst)
  ├─ belongsTo: User (approver)
  └─ hasMany: LoanPayments
```

### Livewire Components

The application uses Livewire for reactive components located in `app/Livewire/`:

- **Settings Components**: Profile, Password, TwoFactor, Appearance, DeleteUserForm
- **Auth Components**: Via Laravel Fortify (login, register, password reset, 2FA challenge)
- Components follow the pattern: `app/Livewire/[Feature]/[Component].php` ↔ `resources/views/livewire/[feature]/[component].blade.php`

### Authorization Pattern

Uses Spatie Laravel Permission for role-based access control:
- Roles and permissions are stored in database (via `permission_tables` migration)
- Assign roles to users: `$user->assignRole('admin')`
- Check permissions: `$user->hasPermissionTo('edit-accounts')`
- Use in controllers: `$this->authorize('update', $account)`

### Financial Operations Pattern

When implementing financial operations:
1. Always use database transactions
2. Record in `transactions` table with `balance_before` and `balance_after`
3. Update account balances atomically
4. For transfers, create two Transaction records (transfer_out and transfer_in)
5. Support reversal via `is_reversed` flag

Example pattern:
```php
DB::transaction(function () use ($account, $amount) {
    $balanceBefore = $account->balance;
    $account->deposit($amount);

    Transaction::create([
        'type' => 'deposit',
        'account_id' => $account->id,
        'amount' => $amount,
        'balance_before' => $balanceBefore,
        'balance_after' => $account->balance,
        'user_id' => auth()->id(),
        'agency_id' => auth()->user()->agency_id,
    ]);
});
```

### Loan Calculation Pattern

Loans use French amortization system:
- Monthly payment calculated with: `P * (r * (1+r)^n) / ((1+r)^n - 1)`
- Payment schedule splits each payment into capital and interest
- Balance reduces with each payment
- Use `generatePaymentSchedule()` method when creating/updating loans

## File Structure Notes

- Models use soft deletes where appropriate (Client, Account, Loan)
- All monetary values use `decimal:2` casting
- Dates use Carbon (Laravel's date library)
- Migrations are numbered chronologically, follow order for dependencies
- Routes are in `routes/web.php` (no API routes currently)
- Livewire views in `resources/views/livewire/`
- Flux UI components customized in `resources/views/flux/`

## Testing Configuration

- Tests use SQLite in-memory database (`:memory:`)
- PHPUnit configured with Unit and Feature test suites
- Test environment variables set in `phpunit.xml`

---
> Source: [FerMar707501/bancavirtual](https://github.com/FerMar707501/bancavirtual) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
