## mdfinanceapk

> When generating, reading, importing, or backup-saving Excel workbooks:

# Custom Agent Instructions for MD Finance App

## Excel Export & Backup Format Rules
When generating, reading, importing, or backup-saving Excel workbooks:
1. **No Friday Sheet**: "Friday" group is completely excluded from the excel sheets and list.
2. **Sunday Groups**: Keep "Sunday mrg" (Sunday Morning) and "Sunday eve" (Sunday Evening) as distinct active day group sheets.
3. **Dynamic Sheets**: Generate day sheets dynamically based on the active collection groups present in the customer database, default to Monday, Tuesday, Wednesday, Thursday, Saturday, Sunday mrg, Sunday eve (excluding Friday).
4. **Column Schema (MANDATORY)**:
   - Column index 0: `Customer UUID`
   - Column index 1: `Route No (Sort Order)`
   - Column index 2: `Client Name`
   - Column index 3: `Phone Number`
   - Column index 4: `City`
   - Column index 5: `SMS Settings (Weekly & Entry Confirmation)` (Formated as: `Weekly Reminder: YES/NO, Entry Confirmation: YES/NO`)
   - Column index 6: `Loan ID (UUID)`
   - Column index 7: `Amount Disbursed`
   - Column index 8: `Principle (₹)`
   - Column index 9: `Interest (₹)`
   - Column index 10: `Date of Dispersal`
   - Column index 11 onwards: Dynamic pairing of `Week X Date & Time` and `Week X Amt Received` for weeks 1 to 30.
5. **No UPI Handle column**: The UPI Handle column is deleted from the Excel export & import schema.

## Attention Checklist Shaking Settings
- **Shaking Threshold**: An item should shake if it needs attention.
- **Days Range**: Check previous **2 days** and today (diff of 0, 1, or 2 from Today's collection day).
- **Grace Period Check**: If there's no payment within the past **2 days** (inclusive of today), trigger the attention shake indicator.

## Dashboard Pill & Deductions Rules
1. **Deductions Separated from Profit**: Upfront deductions MUST NEVER be added to the "Profit" calculations in the dashboard stats. Profit strictly displays only the realized collected interest share from installments.
2. **Deductions Pill**: There MUST be a distinct purple "Deductions" pill underneath the Collection, Disbursal, and Profit metrics on the Dashboard.
3. **Deductions Detail Screen**: Clicking on the Deductions pill MUST open a dedicated ledger page for deductions, just like the other financial metrics, displaying all upfront loan deduction elements generated for the selected day.

---
> Source: [cabharathikrishna-a11y/MDFinanceAPK.](https://github.com/cabharathikrishna-a11y/MDFinanceAPK.) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
