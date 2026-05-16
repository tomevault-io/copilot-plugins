## rvutracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an RVU (Relative Value Unit) tracking application for medical practice management. It tracks dermatology procedure codes, calculates RVUs and estimated income, and provides analytics for Mohs surgery cases.

## Technology Stack

- **Frontend**: Vanilla HTML, CSS, JavaScript
- **Backend**: PHP
- **Database**: MySQL (database name: `kbr37_rvutable`)
- **Charts**: Chart.js
- **Authentication**: Simple PIN-based access control

## Database Tables

The application uses two main tables:

1. **rvudata**: Stores daily RVU entries
   - Fields: `inputs` (comma-separated codes), `totalrvus`, `totalincome`, `date`

2. **mohsdata**: Stores Mohs surgery case details
   - Fields: `date`, `name`, `site`, `NYU`, `diagnosis`, `stages`, `repair`, `comments`, `referral`, `addon`

3. **rvu_tracker_2024**: Lookup table mapping CPT codes to RVU values
   - Fields: `code`, `value`

## Application Architecture

### Frontend Structure (index.html)

The application uses a tabbed interface with four main sections:

1. **RVU Code Entry**: Quick-entry buttons for common CPT codes, manual code input, multiplier support, and 50% reduction checkbox
2. **RVU Summary**: Displays aggregated statistics (today, this week, this month, this year, etc.) with charts showing cumulative income and daily income histograms
3. **Mohs Data Entry**: Form for entering detailed Mohs surgery case information
4. **Mohs Summary**: Analytics on Mohs cases including closure types, tumor types, referral sources, and rolling repair trends

### Key JavaScript Functions (script.js)

- **Tab Management**: Handles switching between the four main tabs
- **RVU Entry Flow**:
  - Code buttons trigger `fetchAndDisplayValue()` which queries `lookup.php`
  - Codes are added to table with multipliers and reduction checkboxes
  - "Approve" button saves to database via `save_data.php`
- **Summary Updates**: `updateSummaryTable()` and `updateMohsSummaryTable()` fetch and display aggregated data
- **Chart Generation**: Three charts using Chart.js:
  - Cumulative Mohs cases (this year vs last year)
  - Cumulative income (this year vs last year)
  - Rolling repair proportions (200-case window)

### Backend PHP Scripts

- **lookup.php**: Fetches RVU value for a given CPT code
- **save_data.php**: Inserts completed RVU entry into database
- **get_rvus.php**: Aggregates RVU statistics across multiple time periods
- **submit_mohs.php**: Inserts Mohs case data into database
- **get_mohs_data.php**: Aggregates Mohs surgery statistics
- **getRecords.php**: Retrieves recent Mohs entries for display
- **delete_last_entry.php**: Removes the most recent RVU entry
- **delete_last_Mohs_entry.php**: Removes the most recent Mohs entry
- **verify_pin.php**: Validates PIN for application access
- **getCumulativeMohsData.php**: Returns daily Mohs counts for charting
- **getDataForIncome.php**: Returns daily income data for charting
- **get_repair_data.php**: Returns repair types for rolling proportion chart
- **get_income.php**: Additional income calculation endpoint

## Important Constants

- **RVU to Income Multiplier**: 73 (hardcoded in `script.js:121` and `script.js:342`)
- **Rolling Window Size**: 200 cases (for repair proportion chart in `script.js:822`)

## Common CPT Code Categories

The application organizes codes into:
- E&M (99212-99204)
- Mohs (17311-17315)
- Primary Repair (13132, 13133, 13151, 13152, 13121, 12032, 12034, 12042, 12052)
- Flaps and Grafts (14040, 14041, 14060, 14061, 14301, 15260, 15240)
- Destruction (17262, 17263, 17272, 17281, 17282)
- Clinic (11102-11105, 17000, 17003, 17004, 17110, 88331, 88305, 11900, 11901, 11200)

## Security Implementation

### Authentication System
- **Session-based authentication** with `auth_check.php` middleware
- **Password hashing** using bcrypt (`password_hash`/`password_verify`)
- **Session timeout** configurable in `auth_config.php` (default: 1 hour)
- **Session regeneration** on login to prevent session fixation attacks
- **Brute force protection** with 1-second delay on failed login attempts
- Failed login attempts are logged with IP addresses

### Database Security
- **Centralized connection** via `db_connect.php`
- **Prepared statements** used in `lookup.php`, `save_data.php`, and `submit_mohs.php`
- UTF-8 charset configured to prevent encoding issues
- Database credentials stored in `db_connect.php` (not in version control)

### Configuration Files (Not in Git)
- `db_connect.php` - Database credentials
- `auth_config.php` - Hashed PIN and session settings
- `generate_hash.php` - Utility to generate PIN hashes (should be deleted after use)

### Protected Endpoints
All PHP endpoints require authentication via `require_once 'auth_check.php'`:
- lookup.php, save_data.php, get_rvus.php
- submit_mohs.php, getRecords.php, get_mohs_data.php
- delete_last_entry.php, delete_last_Mohs_entry.php
- getCumulativeMohsData.php, getDataForIncome.php, get_repair_data.php, get_income.php

### Logout
- `logout.php` - Properly destroys sessions and cookies
- Frontend has logout button in header bar

## Development Workflow

This is a simple LAMP-style application with no build process:

1. Edit HTML/CSS/JS files directly
2. Refresh browser to see frontend changes
3. PHP changes are effective immediately (ensure PHP server is running)
4. Database changes require direct SQL execution

## Testing

There is no automated testing infrastructure. Testing is done manually:

1. Test RVU entry by adding codes and clicking "Approve"
2. Verify summary calculations by checking database values
3. Test Mohs data entry through the form submission
4. Check chart rendering by navigating to summary tabs

---
> Source: [kurtisreed/rvutracker](https://github.com/kurtisreed/rvutracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
