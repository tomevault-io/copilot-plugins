## chanitec-frontend

> The following is a digest of the repository "chanitec-frontend V2".

The following is a digest of the repository "chanitec-frontend V2".
This digest is designed to be easily parsed by Large Language Models.

--- SUMMARY ---
Repository: chanitec-frontend V2
Files Analyzed: 82
Total Text Size: 336.77 KB
Estimated Tokens (text only): ~82,180

--- DIRECTORY STRUCTURE ---
chanitec-frontend V2/
├── .cursor/
│   └── rules/
│       ├── connection.mdc
│       ├── frontend-pages-analysis.mdc
│       └── universalguidlines.mdc
├── src/
│   ├── assets/
│   │   ├── apple-touch-icon.png [binary]
│   │   ├── CHANitec.png [binary]
│   │   ├── favicon-16x16.png [binary]
│   │   ├── favicon-32x32.png [binary]
│   │   ├── logo chanitecc.png [binary]
│   │   ├── logo192.png [binary]
│   │   ├── logo512.png [binary]
│   │   ├── signature-ayachi.png [binary]
│   │   ├── signature-perrache.png.png [binary]
│   │   └── Trane.png [binary]
│   ├── components/
│   │   ├── CustomNumberInput/
│   │   │   ├── CustomNumberInput.scss
│   │   │   └── CustomNumberInput.tsx
│   │   ├── LaborSection/
│   │   │   ├── LaborSection.scss
│   │   │   └── LaborSection.tsx
│   │   ├── Layout/
│   │   │   ├── Layout.scss
│   │   │   └── Layout.tsx
│   │   ├── QuoteActions/
│   │   │   ├── QuoteActions.scss
│   │   │   └── QuoteActions.tsx
│   │   ├── QuoteHeader/
│   │   │   ├── QuoteHeader.scss
│   │   │   └── QuoteHeader.tsx
│   │   ├── SuppliesSection/
│   │   │   ├── SuppliesSection.scss
│   │   │   └── SuppliesSection.tsx
│   │   ├── TotalSection/
│   │   │   ├── TotalSection.scss
│   │   │   └── TotalSection.tsx
│   │   ├── ClientList.js
│   │   └── ItemsList.js
│   ├── contexts/
│   │   └── QuoteContext.tsx
│   ├── data/
│   │   └── sample-items.json
│   ├── hooks/
│   │   ├── useClients.js
│   │   └── useItems.js
│   ├── models/
│   │   ├── PriceOffer.ts [binary]
│   │   └── Quote.ts [binary]
│   ├── pages/
│   │   ├── ClientsPage/
│   │   │   ├── ClientsPage.scss
│   │   │   └── ClientsPage.tsx
│   │   ├── HistoryPage/
│   │   │   ├── HistoryPage.scss
│   │   │   └── HistoryPage.tsx
│   │   ├── HomePage/
│   │   │   ├── HomePage.scss
│   │   │   └── HomePage.tsx
│   │   ├── ItemsPage/
│   │   │   ├── ItemsPage.scss
│   │   │   └── ItemsPage.tsx
│   │   ├── LoginPage/
│   │   │   ├── LoginPage.scss
│   │   │   └── LoginPage.tsx
│   │   ├── PriceOfferPage/
│   │   │   ├── PriceOfferPage.scss
│   │   │   └── PriceOfferPage.tsx
│   │   ├── QuotePage/
│   │   │   ├── QuotePage.scss
│   │   │   └── QuotePage.tsx
│   │   ├── QuoteTest/
│   │   │   ├── QuoteTest.scss
│   │   │   └── QuoteTest.tsx
│   │   ├── employeesPage.scss
│   │   ├── employeesPage.tsx
│   │   ├── interventionPage.scss
│   │   ├── interventionPage.tsx
│   │   ├── orgChartPage.scss
│   │   └── orgChartPage.tsx
│   ├── services/
│   │   ├── api-service.ts [binary]
│   │   ├── api.js
│   │   ├── employee-service.ts [binary]
│   │   ├── price-offer-service.ts [binary]
│   │   └── storage-service.ts [binary]
│   ├── types/
│   │   └── html2pdf.d.ts [binary]
│   ├── utils/
│   │   ├── calculations.ts [binary]
│   │   ├── generate-excel-sample.js
│   │   └── id-generator.ts [binary]
│   ├── App.css
│   ├── App.scss
│   ├── App.test.tsx
│   ├── App.tsx
│   ├── declarations.d.ts [binary]
│   ├── index.css
│   ├── index.tsx
│   ├── logo.png [binary]
│   ├── logo.svg
│   ├── react-app-env.d.ts [binary]
│   ├── reportWebVitals.ts [binary]
│   └── setupTests.ts [binary]
├── electron-main.js
├── package.json
├── script
└── tsconfig.json


--- FILE CONTENTS ---
============================================================
FILE: .cursor/rules/connection.mdc
============================================================
# Chanitec Backend Connection Information for Frontend
# Updated to match backend implementation as of current analysis.

API_BASE_URL = "http://localhost:5000/api"  # The base URL for the backend API (update as needed)
API_VERSION = null  # No explicit versioning in backend routes

# Authentication
AUTH_METHOD = null  # No authentication enforced by backend as per current code
AUTH_TOKEN_PLACEHOLDER = null  # Not required unless implemented later

# Endpoints (CRUD for each resource, matching backend)
ENDPOINTS = {
    # Clients
    "List Clients":        {"method": "GET",    "path": "/clients"},
    "Get Client":          {"method": "GET",    "path": "/clients/{id}"},
    "Create Client":       {"method": "POST",   "path": "/clients"},
    "Update Client":       {"method": "PUT",    "path": "/clients/{id}"},
    "Delete Client":       {"method": "DELETE", "path": "/clients/{id}"},
    # NOTE: As of latest backend update, 'Create Client' and 'Update Client' accept 'name' (required) and 'Taux_change' (optional decimal) fields in the request body.

    # Sites
    "List Sites":          {"method": "GET",    "path": "/sites"},
    "Get Site":            {"method": "GET",    "path": "/sites/{id}"},
    "Create Site":         {"method": "POST",   "path": "/sites"},
    "Update Site":         {"method": "PUT",    "path": "/sites/{id}"},
    "Delete Site":         {"method": "DELETE", "path": "/sites/{id}"},
    "Get Sites By Client": {"method": "GET",    "path": "/sites/by-client?clientId={clientId}"},

    # Items
    "List Items":          {"method": "GET",    "path": "/items"},
    "Get Item":            {"method": "GET",    "path": "/items/{id}"},
    "Create Item":         {"method": "POST",   "path": "/items"},
    "Update Item":         {"method": "PUT",    "path": "/items/{id}"},
    "Delete Item":         {"method": "DELETE", "path": "/items/{id}"},
    "Import Items":        {"method": "POST",   "path": "/items/import"},
    "Clear Items":         {"method": "DELETE", "path": "/items/clear"},
    # NOTE: As of latest backend update, 'Create Item' and 'Update Item' require 'description', 'price', and 'quantity' fields in the request body.

    # Quotes
    "List Quotes":         {"method": "GET",    "path": "/quotes"},
    "Get Quote":           {"method": "GET",    "path": "/quotes/{id}"},
    "Create Quote":        {"method": "POST",   "path": "/quotes"},
    "Update Quote":        {"method": "PUT",    "path": "/quotes/{id}"},
    "Delete Quote":        {"method": "DELETE", "path": "/quotes/{id}"},
    "Set Quote Reminder":  {"method": "PATCH",  "path": "/quotes/{id}/reminder"},
    "Confirm Quote":       {"method": "PATCH",  "path": "/quotes/{id}/confirm"},

    # Supply Items
    "List Supply Items By Quote":   {"method": "GET",    "path": "/supply-items/{quoteId}"},
    "Get Supply Item":              {"method": "GET",    "path": "/supply-items/item/{id}"},
    "Create Supply Item":           {"method": "POST",   "path": "/supply-items/{quoteId}"},
    "Update Supply Item":           {"method": "PUT",    "path": "/supply-items/{id}"},
    "Delete Supply Item":           {"method": "DELETE", "path": "/supply-items/{id}"},

    # Labor Items
    "List Labor Items By Quote":    {"method": "GET",    "path": "/labor-items/{quoteId}"},
    "Get Labor Item":               {"method": "GET",    "path": "/labor-items/item/{id}"},
    "Create Labor Item":            {"method": "POST",   "path": "/labor-items/{quoteId}"},
    "Update Labor Item":            {"method": "PUT",    "path": "/labor-items/{id}"},
    "Delete Labor Item":            {"method": "DELETE", "path": "/labor-items/{id}"},

    # Departments
    "List Departments":     {"method": "GET",    "path": "/departments"},
    "Get Department":       {"method": "GET",    "path": "/departments/{id}"},
    "Create Department":    {"method": "POST",   "path": "/departments"},
    "Update Department":    {"method": "PUT",    "path": "/departments/{id}"},
    "Delete Department":    {"method": "DELETE", "path": "/departments/{id}"},

    # Employees
    "List Employees":       {"method": "GET",    "path": "/employees"},
    "Get Employee":         {"method": "GET",    "path": "/employees/{id}"},
    "Create Employee":      {"method": "POST",   "path": "/employees"},
    "Update Employee":      {"method": "PUT",    "path": "/employees/{id}"},
    "Delete Employee":      {"method": "DELETE", "path": "/employees/{id}"},

    # Splits
    "List Splits":         {"method": "GET",    "path": "/splits"},
    "Get Split":           {"method": "GET",    "path": "/splits/{code}"},
    "Create Split":        {"method": "POST",   "path": "/splits"},
    "Update Split":        {"method": "PUT",    "path": "/splits/{code}"},
    "Delete Split":        {"method": "DELETE", "path": "/splits/{code}"},
    "Get Site For Split":  {"method": "GET",    "path": "/splits/{code}/site"},

    # Debug
    "Get Database Structure": {"method": "GET", "path": "/debug/database-structure"},
    "Test Site Lookup":       {"method": "GET", "path": "/debug/test-site-lookup/{clientId}"}
}

# CORS Settings (Cross-Origin Resource Sharing)
CORS = {
    "allowed_origins": ["http://localhost:3000", "http://localhost:5000"],  # List of allowed origins
    "allowed_methods": ["GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS"],  # HTTP methods allowed
    "allowed_headers": ["Content-Type", "Authorization"],  # Headers allowed in requests
    "allow_credentials": true  # Whether to allow credentials (cookies, authorization headers, etc.)
}

# Additional Metadata (if needed)
# DATABASE_TYPE = "MySQL"
# DATABASE_HOST = "<DB_HOST>"
# DATABASE_PORT = 3306
# DATABASE_NAME = "Chanitec"
# (Note: Frontend should NOT connect directly to the database for security reasons)

# Add any other relevant connection or environment details below
# ...
# DATABASE_NAME = "Chanitec"
# (Note: Frontend should NOT connect directly to the database for security reasons)

# Add any other relevant connection or environment details below
# ...

============================================================
FILE: .cursor/rules/frontend-pages-analysis.mdc
============================================================
---
description: 
globs: 
alwaysApply: true
---
# Frontend Pages Analysis

## 1. employeesPage.tsx / orgChartPage.tsx
- **employeesPage.tsx**: Manages employee records, listing, and possibly CRUD operations for employees.
- **orgChartPage.tsx**: Displays an organizational chart, showing the hierarchy or structure of employees or departments.

## 2. QuotePage
- **QuotePage.tsx / QuotePage.scss**: Main page for creating, editing, or viewing a single quote. Contains forms for quote details, item selection, client/site selection, and total calculations.

## 3. PriceOfferPage
- **PriceOfferPage.tsx / PriceOfferPage.scss**: For generating or viewing price offers based on quotes. Shows quote summary, pricing breakdown, and options to send/export the offer.

## 4. LoginPage
- **LoginPage.tsx / LoginPage.scss**: Handles user authentication. Provides a login form for users to access the application.

## 5. ItemsPage
- **ItemsPage.tsx / ItemsPage.scss**: Manages the catalog of items/products/services that can be added to quotes. Includes item listing, search/filter, and CRUD operations.

## 6. HomePage
- **HomePage.tsx / HomePage.scss**: The landing or dashboard page after login. May show recent activity, quick links, or statistics.

## 7. HistoryPage
- **HistoryPage.tsx / HistoryPage.scss**: Displays the history of quotes. Includes advanced filtering (by client, site, date, etc.), quote versioning, and actions like viewing, deleting, or setting reminders.

## 8. ClientsPage
- **ClientsPage.tsx / ClientsPage.scss**: Manages client records and their associated sites. Includes client listing, site management, and CRUD operations for both.

## General Observations
- Each main business entity (Quote, Price Offer, Item, Client, Employee) has its own management page.
- Pages are paired with SCSS files for styling.
- The structure is modular, with each business function separated into its own directory or file.
- The application supports advanced business logic, such as quote versioning, reminders, and organizational structure.



============================================================
FILE: .cursor/rules/universalguidlines.mdc
============================================================
---
description:
globs:
alwaysApply: true
---
---
description:
globs:
alwaysApply: true
---
description:Zidor Protocol
globs:
  - "**/*"
alwaysApply: true
---

1. Code Quality & Optimization
- Always generate clean, modular, and performant code.
- Prioritize simplicity, but never at the cost of functionality.
- Avoid adding new dependencies, rewriting existing logic, or modifying unrelated code unless explicitly instructed.
- Never refactor or "optimize" code without a direct prompt.

2. Precision & Accuracy
- Follow instructions **exactly** as written.
- Do **not assume, generalize, or infer missing details**.
- If a requirement is unclear or ambiguous, halt and prompt the user for clarification.
- Constants, formulas, and logic must exactly match what's provided, especially in domain-specific tasks.

3. Performance & Scalability
- Optimize only when requested.
- Avoid excessive loops, complex data structures, or premature optimization.
- Keep code scalable through modular structure, but do not over-engineer unless specified.

4. Security & Error Handling
- Implement basic input validation and error handling if it is **part of the instructions**.
- Do not auto-inject security logic unless specifically requested.
- Never expose sensitive data. No hardcoded credentials or keys.

5. Language & Style Consistency
- Adhere strictly to the syntax, style, and conventions of the language in use.
- Use consistent naming, indentation, and formatting across the file.
- Include docstrings or comments **only when explicitly requested** or where clarity is critical.

6. User-Focused Approach
- Ensure a clean, intuitive UX when UI/UX tasks are involved.
- For APIs, follow RESTful standards with clear status codes and messages.
- When building interfaces, avoid assumptions—follow only provided user flow specs.

7. AI Code Refinement Protocol
- Provide the **optimal** solution first—clearly justified.
- If multiple approaches are valid, suggest alternatives with pros and cons.
- **Never auto-implement improvements**—only propose them for review.
- Do not modify existing code unless explicitly instructed to do so.

8. Project-Specific Logic
- Honor all project-specific constants, tax rules, and business logic as described.
- Before editing existing code, check for compatibility and consistency with the existing structure.
- Avoid breaking existing flows or logic unless it's a confirmed bug and the fix is explicitly approved.

9. Powershell Specific
- When writing PowerShell commands, **always separate** commands clearly.
- Avoid aliases; prefer full command names for clarity.

10. GitHub Deployment Readiness
- Ensure code structure and dependencies are compatible with Vercel deployment when preparing for GitHub pushes.
- Add or adjust configuration files (e.g., `vercel.json`) **only if instructed**.

11. Non-Destructive Behavior
- Do not modify, reformat, or clean code segments that are unrelated to the current task.
- Never rewrite entire files unless explicitly told to.
- All changes must be scoped to the user's request only.

12. Change Approval Mechanism
- If a change might improve the code but is not directly requested, suggest it in a comment block or a prompt.
- Do not make structural or logic modifications based on "best practice" unless explicitly allowed.

13.Only act on what I tell you. Suggest improvements only in comments or prompts. Never make assumptions. Never change or touch unrelated code. Always explain your choices when multiple paths are possible.

14. File Creation Discipline
- Do **not create new files** unless it is confirmed that no existing file serves the same purpose.
- Before creating a file, **check all existing files** (by name, structure, and content) to ensure it doesn't already exist under a different name or structure.
- If there's a possible match or overlap, prompt the user for confirmation before proceeding.
- Default behavior: reuse or extend existing files when appropriate and approved.

15. scan this file to understand the project structure before you do anything
C:\Users\Zidor\Desktop\calcul de prix final\Chanitec\chanitec-frontend\.cursor\rules\frontend-pages-analysis.mdc

16. scan this file to understand backend connections
C:\Users\Zidor\Desktop\calcul de prix final\Chanitec\chanitec-frontend\.cursor\rules\connection.mdc




============================================================
FILE: src/components/CustomNumberInput/CustomNumberInput.scss
============================================================
.custom-number-input {
  .MuiTextField-root {
    width: 12rem;

    .MuiInputBase-root {
      padding-left: 0;
      padding-right: 0;
    }

    input[type="number"] {
      -moz-appearance: textfield;
      text-align: center;
      padding: 8px 40px !important;

      &::-webkit-inner-spin-button,
      &::-webkit-outer-spin-button {
        -webkit-appearance: none;
        margin: 0;
      }
    }

    .number-control-button {
      width: 32px;
      height: 32px;
      min-width: 32px;
      padding: 0;
      border-radius: 0;
      color: #666;

      &.decrease-button {
        margin-left: 2px;
      }

      &.increase-button {
        margin-right: 2px;
      }

      &:hover {
        background-color: rgba(0, 0, 0, 0.04);
      }

      &:active {
        background-color: rgba(0, 0, 0, 0.1);
      }

      &.Mui-disabled {
        opacity: 0.5;
        background-color: transparent;
      }

      svg {
        font-size: 18px;
      }
    }
  }
}

============================================================
FILE: src/components/CustomNumberInput/CustomNumberInput.tsx
============================================================
import React from 'react';
import { Box, IconButton, TextField } from '@mui/material';
import { Add as AddIcon, Remove as RemoveIcon } from '@mui/icons-material';
import './CustomNumberInput.scss';

interface CustomNumberInputProps {
  value: number;
  onChange: (value: number) => void;
  label?: string;
  min?: number;
  max?: number;
  step?: number;
  fullWidth?: boolean;
  disabled?: boolean;
  variant?: 'outlined' | 'filled' | 'standard';
  margin?: 'none' | 'dense' | 'normal';
  displayOnly?: boolean; // New prop
}

const CustomNumberInput: React.FC<CustomNumberInputProps> = ({
  value,
  onChange,
  label,
  min = 0,
  max,
  step = 1,
  fullWidth = false,
  disabled = false,
  variant = 'outlined',
  margin = 'dense',
  displayOnly = false // New prop
}) => {
  if (displayOnly) {
    return (
      <Box className="custom-number-input display-only" sx={{ display: 'flex', flexDirection: 'column', width: fullWidth ? '100%' : 'auto' }}>
        {label && <span className="custom-number-label" style={{ fontWeight: 500, marginBottom: 2 }}>{label}</span>}
        <span className="custom-number-value" style={{ padding: '8.5px 14px', background: '#f5f5f5', borderRadius: 4, color: '#888', minHeight: 38 }}>{value}</span>
      </Box>
    );
  }

  const handleIncrease = () => {
    if (max === undefined || value < max) {
      onChange(value + step);
    }
  };

  const handleDecrease = () => {
    if (value > min) {
      onChange(value - step);
    }
  };

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = parseFloat(e.target.value);
    if (!isNaN(newValue)) {
      if (max !== undefined && newValue > max) {
        onChange(max);
      } else if (newValue < min) {
        onChange(min);
      } else {
        onChange(newValue);
      }
    }
  };

  return (
    <Box className="custom-number-input" sx={{ display: 'inline-flex', alignItems: 'center', width: fullWidth ? '100%' : 'auto' }}>
      <TextField
        type="number"
        value={value}
        onChange={handleInputChange}
        label={label}
        variant={variant}
        margin={margin}
        disabled={disabled}
        fullWidth={fullWidth}
        InputProps={{
          inputProps: {
            min,
            max,
            step
          },
          startAdornment: (
            <IconButton
              size="small"
              onClick={handleDecrease}
              disabled={disabled || value <= min}
              className="number-control-button decrease-button"
            >
              <RemoveIcon fontSize="small" />
            </IconButton>
          ),
          endAdornment: (
            <IconButton
              size="small"
              onClick={handleIncrease}
              disabled={disabled || (max !== undefined && value >= max)}
              className="number-control-button increase-button"
            >
              <AddIcon fontSize="small" />
            </IconButton>
          )
        }}
      />
    </Box>
  );
};

export default CustomNumberInput;

============================================================
FILE: src/components/LaborSection/LaborSection.scss
============================================================
.labor-section {
  padding: 1.5rem;
  margin-bottom: 2rem;

  @media print {
    padding: 0;
    margin: 0;
  }

  .section-title {
    font-weight: 700;
    color: #1976d2;
    padding-bottom: 0rem;

    @media print {
      margin: 0;
      padding: 0;
      border-bottom: none;
      background-color: none;
    }
  }

  .description-field {
    height: 3rem;
    margin-bottom: 2rem;

    @media print {
      margin: 0;
    }
  }

  .rates-container {
    margin-bottom: 1.5rem;

    @media print {
      margin: 0;
    }
  }

  .add-labor-form {
    padding: 1rem;
    margin-bottom: 1.5rem;
    background-color: #f9f9f9;

    @media print {
      display: none;
    }

    .add-button-container {
      display: flex;
      justify-content: flex-end;
      margin-top: 1rem;

      .add-labor-button {
        background-color: #4caf50;

        &:hover {
          background-color: #388e3c;
        }
      }
    }
  }

  .items-table-container {
    margin-bottom: 1rem;
    max-height: 300px;
    overflow-y: auto;

    @media print {
      margin: 0;
      max-height: none;
      overflow-y: visible;
    }

    table {
      th {
        font-weight: 700;
        background-color: #f5f5f5;

        @media print {
          &:last-child {
            display: none;
          }
        }
      }

      td {
        @media print {
          &:last-child {
            display: none;
          }
        }
      }
    }
  }

  .rates-container-item {
    width: 40px !important;
  }

  .total-container {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    padding: 0.5rem 1rem;
    background-color: #f5f5f5;
    border-radius: 4px;

    @media print {
      margin: 0;
      padding: 0;
      border-radius: 0;
    }

    .total-label {
      font-weight: 700;
      margin-right: 0.5rem;

      @media print {
        margin: 0;
      }
    }

    .total-value {
      font-weight: 700;
      color: #1976d2;
    }
  }
}

============================================================
FILE: src/components/LaborSection/LaborSection.tsx
============================================================
import React, { useState, useEffect } from 'react';
import {
  Box,
  Button,
  IconButton,
  MenuItem,
  Paper,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  TextField,
  Typography
} from '@mui/material';
import {
  Add as AddIcon,
  Delete as DeleteIcon
} from '@mui/icons-material';
import { LaborItem } from '../../models/Quote';
import { calculateLaborItemTotal } from '../../utils/calculations';
import CustomNumberInput from '../CustomNumberInput/CustomNumberInput';
import './LaborSection.scss';

interface LaborSectionProps {
  items?: LaborItem[];
  description: string;
  exchangeRate: number;
  marginRate: number;
  totalHT: number;
  onAddItem: (item: Omit<LaborItem, 'id'>) => void;
  onRemoveItem: (id: string) => void;
  onUpdateDescription: (description: string) => void;
  onUpdateExchangeRate: (rate: number) => void;
  onUpdateMarginRate: (rate: number) => void;
}

const LaborSection: React.FC<LaborSectionProps> = ({
  items = [],
  description,
  exchangeRate,
  marginRate,
  totalHT,
  onAddItem,
  onRemoveItem,
  onUpdateDescription,
  onUpdateExchangeRate,
  onUpdateMarginRate
}) => {
  const [nbTechnicians, setNbTechnicians] = useState(1);
  const [nbHours, setNbHours] = useState(1);
  const [weekendMultiplier, setWeekendMultiplier] = useState(1);
  const [priceEuro, setPriceEuro] = useState(10);

  // Weekend multiplier options
  const weekendOptions = [
    { value: 1, label: '1 (Normal)' },
    { value: 1.6, label: '1.6 (Weekend)' }
  ];

  // Calculate dollar prices for all items when rates change
  useEffect(() => {
    const itemsWithDollarPrices = items.map(item =>
      calculateLaborItemTotal(item, exchangeRate, marginRate)
    );
    // Update items with calculated dollar prices
    itemsWithDollarPrices.forEach(item => {
      const existingItem = items.find(i => i.id === item.id);
      if (existingItem) {
        existingItem.priceDollar = item.priceDollar;
        existingItem.unitPriceDollar = item.unitPriceDollar;
        existingItem.totalPriceDollar = item.totalPriceDollar;
      }
    });
  }, [items, exchangeRate, marginRate]);

  // Handle adding a new labor item
  const handleAddLaborItem = () => {
    const newItem: Omit<LaborItem, 'id'> = {
      description,
      nbTechnicians,
      nbHours,
      weekendMultiplier,
      priceEuro
    };
    const calculatedItem = calculateLaborItemTotal(newItem as LaborItem, exchangeRate, marginRate);
    onAddItem({
      ...calculatedItem
    });

    // Reset form fields
    setNbTechnicians(1);
    setNbHours(1);
    setWeekendMultiplier(1);
    setPriceEuro(10);
  };

  return (
    <Paper className="labor-section" elevation={2}>
      <Typography variant="h6" className="section-title">
        MAIN D'OEUVRE
      </Typography>

      <TextField
        select
        fullWidth
        label="Description de la main d'oeuvre"
        value={description}
        onChange={(e) => onUpdateDescription(e.target.value)}
        variant="outlined"
        margin="normal"
        className="description-field"
      >
        <MenuItem value="Fixation, mise au point, raccordement, mise en vide et mise en service En weekend">
          Fixation, mise au point, raccordement, mise en vide et mise en service En weekend
        </MenuItem>
        <MenuItem value="Fixation, mise au point, raccordement, mise en vide et mise en service ">
          Fixation, mise au point, raccordement, mise en vide et mise en service
        </MenuItem>
      </TextField>

      <Box sx={{ display: 'flex', flexWrap: 'wrap', alignItems:  'right' , gap: 5 }} className="rates-container">
        <Box className="rates-container-item" sx={{ flex: '1 1 50px', width: '50%'}}>
          <CustomNumberInput
            label="Taux de change"
            value={exchangeRate}
            min={0}
            step={0.01}
            displayOnly={true}
            onChange={() => {}}
          />
        </Box>
        <Box className="rates-container-item" sx={{ flex: '1 1 50px' , width: '50%' }}>
          <CustomNumberInput
            label="Taux de marge"
            value={marginRate}
            min={0}
            max={1}
            step={0.01}
            displayOnly={true}
            onChange={() => {}}
          />
        </Box>
      </Box>

      <Paper className="add-labor-form" elevation={1}>
        <Typography variant="subtitle1" gutterBottom>
          Ajouter Main d'oeuvre
        </Typography>

        <Box sx={{ display: 'flex', flexWrap: 'wrap', gap: 2 }}>
          <Box sx={{ flex: '1 1 220px' }}>
            <CustomNumberInput
              label="Nb technicien"
              value={nbTechnicians}
              onChange={(value) => setNbTechnicians(value)}
              min={1}
              step={1}
              fullWidth
            />
          </Box>

          <Box sx={{ flex: '1 1 220px' }}>
            <CustomNumberInput
              label="Nb heures"
              value={nbHours}
              onChange={(value) => setNbHours(value)}
              min={1}
              step={1}
              fullWidth
            />
          </Box>

          <Box sx={{ flex: '1 1 220px' }}>
            <TextField
              select
              fullWidth
              label="Majo Weekend"
              value={weekendMultiplier}
              onChange={(e) => setWeekendMultiplier(parseFloat(e.target.value))}
              variant="outlined"
              margin="dense"
            >
              {(weekendOptions ?? []).map((option) => (
                <MenuItem key={option.value} value={option.value}>
                  {option.label}
                </MenuItem>
              ))}
            </TextField>
          </Box>

          <Box sx={{ flex: '1 1 220px' }}>
            <CustomNumberInput
              label="PR €"
              value={priceEuro}
              onChange={(value) => setPriceEuro(value)}
              min={0}
              step={0.01}
              fullWidth
            />
          </Box>
        </Box>

        <Box className="add-button-container">
          <Button
            variant="contained"
            color="primary"
            startIcon={<AddIcon />}
            onClick={handleAddLaborItem}
            className="add-labor-button"
          >
            Ajouter
          </Button>
        </Box>
      </Paper>

      <TableContainer className="items-table-container">
        <Table size="small" aria-label="labor table">
          <TableHead>
            <TableRow>
              <TableCell>Nb technicien</TableCell>
              <TableCell>Nb heures</TableCell>
              <TableCell>Majo Weekend</TableCell>
              <TableCell align="right">PR €</TableCell>
              <TableCell align="right">PR $</TableCell>
              <TableCell align="right">PV/u $</TableCell>
              <TableCell align="right">PV $ Total HT</TableCell>
              <TableCell align="center">Actions</TableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            {items.length === 0 ? (
              <TableRow>
                <TableCell colSpan={8} align="center">
                  Aucun élément de main d'oeuvre ajouté
                </TableCell>
              </TableRow>
            ) : (
              (items ?? []).map((item) => (
                <TableRow key={item.id}>
                  <TableCell>{item.nbTechnicians}</TableCell>
                  <TableCell>{item.nbHours}</TableCell>
                  <TableCell>{item.weekendMultiplier}</TableCell>
                  <TableCell align="right">{Number(item.priceEuro ?? 0).toFixed(2)}</TableCell>
                  <TableCell align="right">{Number(item.priceDollar ?? 0).toFixed(2)}</TableCell>
                  <TableCell align="right">{Number(item.unitPriceDollar ?? 0).toFixed(2)}</TableCell>
                  <TableCell align="right">{Number(item.totalPriceDollar ?? 0).toFixed(2)}</TableCell>
                  <TableCell align="center">
                    <IconButton
                      size="small"
                      color="error"
                      onClick={() => onRemoveItem(item.id)}
                    >
                      <DeleteIcon fontSize="small" />
                    </IconButton>
                  </TableCell>
                </TableRow>
              ))
            )}
          </TableBody>
        </Table>
      </TableContainer>

      <Box className="total-container">
        <Typography variant="subtitle1" className="total-label">
          TOTAL MAIN D'OEUVRE $ HT:
        </Typography>
        <Typography variant="subtitle1" className="total-value">
          {Number(totalHT ?? 0).toFixed(2)}
        </Typography>
      </Box>
    </Paper>
  );
};

export default LaborSection;

============================================================
FILE: src/components/Layout/Layout.scss
============================================================
.layout-root {
  display: flex;
  flex-direction: column;
  min-height: 100vh;

  .app-bar {
    background-color: #1976d2;

    @media print {
      display: none !important;
    }

    .toolbar-content {
      display: flex;
      width: 100%;
      align-items: center;
      justify-content: space-between;

      .main-menu-button {
        margin-right: 2rem;
        font-weight: 500;
        text-transform: none;
        color: white;
        border-right: 1px solid rgba(255, 255, 255, 0.3);
        padding-right: 1rem;
      }

      .logo-container {
        display: flex;
        align-items: center;
        position: absolute;
        left: 16px;

        .header-logo {
          height: 60px;
          margin-right: 1rem;
        }
      }

      .nav-links {
        display: flex;
        margin: 0 auto;
        justify-content: center;

        @media (max-width: 768px) {
          flex-direction: column;
          align-items: center;
        }

        .nav-button {
          margin: 0 0.5rem;
          font-weight: 500;
          text-transform: none;
          color: white;

          @media (max-width: 768px) {
            margin: 0.25rem 0;
          }

          &.active {
            font-weight: 700;
            text-decoration: underline;
          }
        }
      }

      .logout-button {
        margin-left: auto;
        font-weight: 500;
        text-transform: none;
        color: white;
        border-left: 1px solid rgba(255, 255, 255, 0.3);
        padding-left: 1rem;
      }
    }
  }

  .main-content {
    flex-grow: 1;
    padding: 2rem 1rem;
    background-color: #f5f5f5;
  }

  .footer {
    padding: 1.5rem;
    margin-top: auto;
    background-color: #f5f5f5;
    border-top: 1px solid #e0e0e0;
  }
}

// Print styles
@media print {
  .layout-root {
    background-color: white;
    display: flex !important;
    flex-direction: column !important;

    .app-bar {
      display: none !important;
    }

    .main-content {
      background-color: transparent;
      transform: none;
    }

    .footer {
      display: none;
    }
  }
}

============================================================
FILE: src/components/Layout/Layout.tsx
============================================================
import React, { ReactNode } from 'react';
import { AppBar, Box, Container, Toolbar, Typography, CssBaseline, Button } from '@mui/material';
import {
  HomeOutlined,
  HistoryOutlined,
  PeopleOutlineOutlined,
  InventoryOutlined,
  Menu,
  Logout
} from '@mui/icons-material';
import logo from '../../logo.png'; // Import the logo from src directory
import './Layout.scss';

interface LayoutProps {
  children: ReactNode;
  currentPath: string;
  onNavigate?: (path: string) => void;
  onHomeClick?: () => void;
  onLogout?: () => void;
}

const Layout: React.FC<LayoutProps> = ({
  children,
  currentPath = '/',
  onNavigate,
  onHomeClick,
  onLogout
}) => {
  // Navigation items
  const navItems = [
    { path: '/quote', label: 'Accueil', icon: <HomeOutlined /> },
    { path: '/history', label: 'Historique', icon: <HistoryOutlined /> },
    { path: '/clients', label: 'Clients', icon: <PeopleOutlineOutlined /> },
    { path: '/items', label: 'Gérer les articles', icon: <InventoryOutlined /> }
  ];

  const handleNavigate = (path: string) => {
    if (path === '/' && onHomeClick) {
      onHomeClick();
    }
    if (onNavigate) {
      onNavigate(path);
    }
  };

  return (
    <Box className="layout-root">
      <CssBaseline />
      <AppBar position="static" color="primary" className="app-bar">
        <Toolbar>
          <Box className="toolbar-content">
            <Button
              color="inherit"
              className="main-menu-button"
              startIcon={<Menu />}
              onClick={() => handleNavigate('/home')}
            >
              Menu Principal
            </Button>
            <Box className="nav-links">
              {(navItems ?? []).map((item) => (
                <Button
                  key={item.path}
                  color="inherit"
                  className={`nav-button ${currentPath === item.path ? 'active' : ''}`}
                  startIcon={item.icon}
                  onClick={() => handleNavigate(item.path)}
                >
                  {item.label}
                </Button>
              ))}
            </Box>
            {onLogout && (
              <Button
                color="inherit"
                className="logout-button"
                startIcon={<Logout />}
                onClick={onLogout}
              >
                Déconnexion
              </Button>
            )}
          </Box>
        </Toolbar>
      </AppBar>

      <Container component="main" className="main-content">
        {children}
      </Container>

      <Box component="footer" className="footer">
        <Typography variant="body2" color="textSecondary" align="center">
          © {new Date().getFullYear()} Chanitec
        </Typography>
      </Box>
    </Box>
  );
};

export default Layout;

============================================================
FILE: src/components/QuoteActions/QuoteActions.scss
============================================================
.quote-actions {
  padding: 1.5rem;
  margin-top: 1rem;
  margin-bottom: 2rem;
  border-radius: 4px;
  background-color: #f9f9f9;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);

  .actions-container {
    display: flex;
    gap: 1rem;
    justify-content: center;

    @media (max-width: 600px) {
      flex-direction: column;
      align-items: stretch;
    }

    .action-button {
      min-width: 150px;
      padding: 10px 20px;
      font-weight: 500;

      @media (max-width: 600px) {
        margin-bottom: 0.5rem;
      }

      &.save-button {
        background-color: #4caf50;

        &:hover {
          background-color: #388e3c;
        }
      }

      &.update-button {
        background-color: #ff9800;

        &:hover {
          background-color: #f57c00;
        }
      }

      &.print-button {
        background-color: #2196f3;

        &:hover {
          background-color: #1976d2;
        }
      }

      &.download-button {
        background-color: #3f51b5;

        &:hover {
          background-color: #303f9f;
        }
      }
    }
  }
}

// PDF specific styles
.pdf-container {
  .MuiPaper-root {
    box-shadow: none;
  }
}

// Print styles
@media print {
  .quote-actions {
    display: none;
  }
}

============================================================
FILE: src/components/QuoteActions/QuoteActions.tsx
============================================================
import React from 'react';
import { Box, Button, Paper, Snackbar, Alert } from '@mui/material';
import {
  SaveOutlined,
  PrintOutlined,
  GetApp as DownloadIcon,
  Update as UpdateIcon
} from '@mui/icons-material';
import { format } from 'date-fns';
import { usePDF } from 'react-to-pdf';
import { apiService } from '../../services/api-service';
import { useQuote } from '../../contexts/QuoteContext';
import './QuoteActions.scss';

interface QuoteActionsProps {
  clientName: string;
  siteName: string;
  date: string;
  isExistingQuote: boolean;
  onSave: () => Promise<boolean>;
  onUpdate?: () => Promise<boolean>;
  onViewHistory: () => void;
  contentRef: React.RefObject<HTMLDivElement>;
  onPrint: () => void;
  onDownloadPDF: () => void;
}

const QuoteActions: React.FC<QuoteActionsProps> = ({
  clientName,
  siteName,
  date,
  isExistingQuote,
  onSave,
  onUpdate,
  onViewHistory,
  contentRef,
  onPrint,
  onDownloadPDF
}) => {
  const { state, saveQuote, updateQuote, setQuoteField, clearQuote, createNewQuote } = useQuote();
  const [snackbarOpen, setSnackbarOpen] = React.useState(false);
  const [snackbarMessage, setSnackbarMessage] = React.useState('');
  const [snackbarSeverity, setSnackbarSeverity] = React.useState<'success' | 'error'>('success');

  // Setup PDF generation
  const { toPDF, targetRef } = usePDF({
    filename: generatePdfFilename(),
    page: {
      format: 'A4',
      margin: 0
    },
    method: 'open',
    canvas: {
      mimeType: 'image/png',
      qualityRatio: 1
    }
  });

  // Handle save action
  const handleSave = async () => {
    try {
      const success = await onSave();

      if (success) {
        setSnackbarMessage('Devis enregistré avec succès! Prêt pour un nouveau devis.');
        setSnackbarSeverity('success');
        window.location.href = '/quote';
        clearQuote();
        createNewQuote();
        alert('Devis enregistré avec succès! Prêt pour un nouveau devis.');
      } else {
        setSnackbarMessage('Erreur lors de l\'enregistrement du devis.');
        setSnackbarSeverity('error');
      }

      setSnackbarOpen(true);
    } catch (error) {
      setSnackbarMessage('Erreur lors de l\'enregistrement du devis.');
      setSnackbarSeverity('error');
      setSnackbarOpen(true);
    }
  };

  // Handle save and print action
  const handleSaveAndPrint = async () => {
    try {
      // First save the quote if it doesn't exist
      if (!isExistingQuote) {
        const success = await onSave();
        if (!success) {
          setSnackbarMessage('Erreur lors de l\'enregistrement du devis.');
          setSnackbarSeverity('error');
          setSnackbarOpen(true);
          return;
        }
        setSnackbarMessage('Devis enregistré avec succès!');
        setSnackbarSeverity('success');
        setSnackbarOpen(true);
      }

      // Then proceed with print functionality
      onPrint();
    } catch (error) {
      setSnackbarMessage('Erreur lors de l\'enregistrement du devis.');
      setSnackbarSeverity('error');
      setSnackbarOpen(true);
    }
  };

  // Handle update action
  const handleUpdate = async () => {
    try {
      const currentQuoteInfo = {
        id: state.currentQuote?.id || '',
        clientName: state.currentQuote?.clientName || '',
        siteName: state.currentQuote?.siteName || '',
        object: state.currentQuote?.object || '',
        date: state.currentQuote?.date || '',
        supplyDescription: state.currentQuote?.supplyDescription || '',
        laborDescription: state.currentQuote?.laborDescription || '',
        supplyExchangeRate: state.currentQuote?.supplyExchangeRate || 0,
        supplyMarginRate: state.currentQuote?.supplyMarginRate || 0,
        laborExchangeRate: state.currentQuote?.laborExchangeRate || 0,
        laborMarginRate: state.currentQuote?.laborMarginRate || 0,
        supplyItems: state.currentQuote?.supplyItems || [],
        laborItems: state.currentQuote?.laborItems || [],
        totalSuppliesHT: state.currentQuote?.totalSuppliesHT || 0,
        totalLaborHT: state.currentQuote?.totalLaborHT || 0,
        totalHT: state.currentQuote?.totalHT || 0,
        tva: state.currentQuote?.tva || 0,
        totalTTC: state.currentQuote?.totalTTC || 0,
        createdAt: state.currentQuote?.createdAt || new Date().toISOString(),
        updatedAt: new Date().toISOString(),
        confirmed: false,
        reminderDate: null
      };

      const originalQuote = await apiService.getQuoteById(currentQuoteInfo.id);
      const hasChanges = JSON.stringify(currentQuoteInfo) !== JSON.stringify(originalQuote);

      if (hasChanges) {
        const currentId = currentQuoteInfo.id;
        const parts = currentId.split('-');
        const version = parseInt(parts[2]) + 1;
        const newId = `${parts[0]}-${parts[1]}-${version}`;

        const updatedQuote = {
          ...currentQuoteInfo,
          id: newId,
          updatedAt: new Date().toISOString()
        };

        alert(`About to send the following payload to the database:\n\n${JSON.stringify(updatedQuote, null, 2)}`);

        const success = await updateQuote();

        if (success) {
          setSnackbarMessage('Devis mis à jour avec succès! Prêt pour un nouveau devis.');
          setSnackbarSeverity('success');
          clearQuote();
          createNewQuote();
          alert('Devis mis à jour avec succès! Prêt pour un nouveau devis.');
        } else {
          throw new Error('Failed to update quote');
        }
      } else {
        alert('No changes detected in the quote');
      }

      setSnackbarOpen(true);
    } catch (error) {
      setSnackbarMessage('Erreur lors de la mise à jour du devis.');
      setSnackbarSeverity('error');
      setSnackbarOpen(true);
    }
  };

  // Generate PDF filename
  function generatePdfFilename() {
    const formattedDate = date ? format(new Date(date), 'dd-MM-yyyy') : format(new Date(), 'dd-MM-yyyy');
    const cleanClientName = (clientName || 'Client').replace(/[^a-zA-Z0-9]/g, '-');
    const cleanSiteName = (siteName || 'Site').replace(/[^a-zA-Z0-9]/g, '-');

    return `${cleanClientName}-${cleanSiteName}-${formattedDate}.pdf`;
  }

  // Handle PDF download
  const handleDownloadPDF = () => {
    if (contentRef.current) {
      // Add pdf-print-mode class to apply print styling
      contentRef.current.classList.add('pdf-print-mode');

      // Force a reflow to ensure styles are applied
      const reflow = contentRef.current.offsetHeight;
      void reflow;

      // Use the ref from props instead of the one from usePDF
      targetRef.current = contentRef.current;

      // Generate PDF after a small delay to ensure styles are applied
      setTimeout(() => {
        toPDF();

        // Remove the class after PDF generation
        setTimeout(() => {
          contentRef.current?.classList.remove('pdf-print-mode');
        }, 1000);
      }, 100);

      setSnackbarMessage('PDF téléchargé avec succès!');
      setSnackbarSeverity('success');
      setSnackbarOpen(true);
    }
  };

  // Handle print action using browser print functionality
  const handlePrint = () => {
    if (contentRef.current) {
      // Instead of opening a new window, trigger the browser's native print dialog
      // This way, all print media queries will be applied
      window.print();
    }
  };

  // Close snackbar
  const handleCloseSnackbar = () => {
    setSnackbarOpen(false);
  };

  return (
    <Paper className="quote-actions" elevation={2}>
      <Box className="actions-container">
        {isExistingQuote ? (
          <Button
            variant="contained"
            color="primary"
            className="action-button update-button"
            startIcon={<UpdateIcon />}
            onClick={handleSave}
          >
            Mettre à jour
          </Button>
        ) : (
          <Button
            variant="contained"
            color="primary"
            className="action-button save-button"
            startIcon={<SaveOutlined />}
            onClick={handleSave}
          >
            Enregistrer
          </Button>
        )}

        <Button
          variant="contained"
          color="info"
          className="action-button download-button"
          startIcon={<DownloadIcon />}
          onClick={handleSaveAndPrint}
        >
          Enregistrer sous
        </Button>
      </Box>

      <Snackbar
        open={snackbarOpen}
        autoHideDuration={6000}
        onClose={handleCloseSnackbar}
        anchorOrigin={{ vertical: 'bottom', horizontal: 'center' }}
      >
        <Alert
          onClose={handleCloseSnackbar}
          severity={snackbarSeverity}
          sx={{ width: '100%' }}
        >
          {snackbarMessage}
        </Alert>
      </Snackbar>
    </Paper>
  );
};

export default QuoteActions;

============================================================
FILE: src/components/QuoteHeader/QuoteHeader.scss
============================================================
.quote-header {
  padding: 1.5rem;
  margin-bottom: 2rem;

  @media print {
    padding: 0;
    margin: 0;
  }

  .quote-id-display {
    display: flex;
    align-items: center;
    justify-content: flex-end;
    margin-bottom: 1rem;

    @media print {
      margin: 0;
      font-size: 1.5rem;
      padding-top: 0;
    }

    .id-label {
      margin-right: 0.5rem;
      font-weight: 500;

      @media print {
        margin: 0;
      }
    }

    .id-value {
      font-weight: 700;
      color: #1976d2;

      @media print {
        font-size: 0.8rem;
      }

      &.is-revision {
        color: #ff9800;
      }

      .version-number {
        font-weight: 800;
        background-color: #ff9800;
        color: white;
        padding: 2px 4px;
        border-radius: 4px;
        margin-left: 2px;

        @media print {
          padding: 1px 2px;
          font-size: 0.7rem;
        }
      }
    }
  }

  .info-grid {
    .header-field {
      margin-top: 0.5rem;
      margin-bottom: 0.5rem;

      @media print {
        margin: 0;
        padding: 0;
      }

      .MuiFormLabel-root {
        font-weight: 600;
        font-size: 0.875rem;

        @media print {
          font-size: 0.7rem;
          margin-bottom: 0;
        }
      }

      .MuiInputBase-input {
        font-size: 0.9rem;

        @media print {
          font-size: 0.7rem;
          padding: 2px 4px;
          margin-top: 0;
        }
      }
    }
  }
}


============================================================
FILE: src/components/QuoteHeader/QuoteHeader.tsx
============================================================
import React, { useEffect, useState } from 'react';
import { Box, Paper, TextField, Typography, MenuItem, Tooltip, CircularProgress } from '@mui/material';
import { Client, Site } from '../../models/Quote';
import { apiService } from '../../services/api-service';
import { extractVersion } from '../../utils/id-generator';
import './QuoteHeader.scss';
import CustomNumberInput from '../CustomNumberInput/CustomNumberInput';

interface QuoteHeaderProps {
  quoteId: string;
  clientName: string;
  siteName: string;
  object: string;
  date: string;
  onClientChange: (value: string) => void;
  onSiteChange: (value: string) => void;
  onObjectChange: (value: string) => void;
  onDateChange: (value: string) => void;
  onClientMarginChange?: (margin: number) => void;
}

const QuoteHeader: React.FC<QuoteHeaderProps> = ({
  quoteId,
  clientName,
  siteName,
  object,
  date,
  onClientChange,
  onSiteChange,
  onObjectChange,
  onDateChange,
  onClientMarginChange
}) => {
  const [clients, setClients] = useState<Client[]>([]);
  const [sites, setSites] = useState<Site[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [isSitesLoading, setIsSitesLoading] = useState(false);
  const [selectedClientId, setSelectedClientId] = useState<string | null>(null);
  const [siteError, setSiteError] = useState<string | null>(null);
  const [clientMargin, setClientMargin] = useState<number | null>(null);

  // Format quoteId to display version information
  const formatQuoteId = (id: string) => {
    const match = id.match(/^(F-\d{8})-(\d{3})$/);

    if (match) {
      const [_, baseId, version] = match;
      const versionNum = parseInt(version, 10);

      if (versionNum === 0) {
        return id;
      } else {
        return (
          <>
            {baseId}-<span className="version-number">{version}</span>
          </>
        );
      }
    }

    return id;
  };

  // Load clients on component mount
  useEffect(() => {
    const loadClients = async () => {
      try {
        setIsLoading(true);
        const loadedClients = await apiService.getClients();
        console.log('Loaded clients:', loadedClients);
        setClients(loadedClients);
      } catch (error) {
        console.error('Error loading clients:', error);
      } finally {
        setIsLoading(false);
      }
    };

    loadClients();
  }, []);

  // Load sites whenever clientName changes - separate from the client loading effect
  useEffect(() => {
    const loadSitesForClient = async () => {
      if (!clientName) {
        console.log('No client selected, clearing sites');
        setSites([]);
        setSelectedClientId(null);
        setSiteError(null);
        return;
      }

      try {
        setSiteError(null);
        setIsSitesLoading(true);

        const selectedClient = clients.find(c => c.name === clientName);
        console.log('Selected client:', selectedClient);

        if (!selectedClient) {
          console.log('No matching client found, clearing sites');
          setSites([]);
          setSelectedClientId(null);
          return;
        }

        setSelectedClientId(selectedClient.id);

        const API_BASE_URL = process.env.REACT_APP_API_URL;
        const sitesUrl = `${API_BASE_URL}/sites/by-client?clientId=${selectedClient.id}`;
        console.log('Sending request to:', sitesUrl);

        const sitesResponse = await fetch(sitesUrl);

        if (!sitesResponse.ok) {
          console.error('Error response from API:', sitesResponse.status, sitesResponse.statusText);
          throw new Error(`Failed to fetch sites: ${sitesResponse.statusText}`);
        }

        const clientSites = await sitesResponse.json();
        console.log(`Sites for client ${selectedClient.id}:`, clientSites);

        if (Array.isArray(clientSites) && clientSites.length === 0) {
          console.log('No sites found for this client');
        }

        setSites(clientSites);

      } catch (error) {
        console.error('Error loading sites for client:', error);
        setSiteError('Failed to load sites for this client');
        setSites([]);
      } finally {
        setIsSitesLoading(false);
      }
    };

    loadSitesForClient();
  }, [clientName, clients]);

  useEffect(() => {
    if (!clientName || clients.length === 0) return;
    // Use case-insensitive, trimmed match for robustness
    const selectedClient = clients.find(
      c => c.name.trim().toLowerCase() === clientName.trim().toLowerCase()
    );
    if (selectedClient && selectedClient.Taux_marge != null) {
      const margin = Number(selectedClient.Taux_marge);
      if (!isNaN(margin)) {
        setClientMargin(margin);
        if (onClientMarginChange) onClientMarginChange(margin);
      }
    }
  }, [clientName, clients, onClientMarginChange]);

  // Handle client selection change - with safeguards for debugging
  const handleClientChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    try {
      const value = event.target.value;
      // Find the selected client
      const selectedClient = clients.find(c => c.name === value);
      // Call the parent to update the client name
      onClientChange(value);
      // If the client has a Taux_marge, update local state and call margin change handler
      if (selectedClient && selectedClient.Taux_marge != null) {
        const margin = Number(selectedClient.Taux_marge);
        if (!isNaN(margin)) {
          setClientMargin(margin);
          if (onClientMarginChange) onClientMarginChange(margin);
        }
      }
      // Clear site when client changes
      onSiteChange('');
    } catch (error) {
      console.error('Error in client selection handler:', error);
    }
  };

  const version = extractVersion(quoteId);
  const isRevision = version !== null && version > 0;

  return (
    <Paper className="quote-header" elevation={2}>
      <Box className="quote-id-display">
        <Typography variant="subtitle1" className="id-label">
          Devis:
        </Typography>
        <Tooltip title={isRevision ? `Version ${version} du devis` : "Version originale"}>
          <Typography variant="subtitle1" className={`id-value ${isRevision ? 'is-revision' : ''}`}>
            {formatQuoteId(quoteId)}
          </Typography>
        </Tooltip>
      </Box>

      <Box sx={{ display: 'flex', flexWrap: 'wrap', gap: 2 }} className="info-grid">
        {/* Client selection dropdown - simplified for reliability */}
        <Box sx={{ flex: '1 1 220px' }}>
          <TextField
            select
            fullWidth
            label="CLIENT"
            value={clientName}
            onChange={handleClientChange}
            variant="outlined"
            margin="normal"
            className="header-field"
          >
            <MenuItem value="">Sélectionnez un client</MenuItem>
            {clients.map((client) => (
              <MenuItem key={client.id} value={client.name}>
                {client.name}
              </MenuItem>
            ))}
          </TextField>
          {isLoading && <Typography variant="caption" color="text.secondary">Chargement des clients...</Typography>}
        </Box>


        <Box sx={{ flex: '1 1 220px', position: 'relative' }}>
          <TextField
            select
            fullWidth
            label="SITE"
            value={siteName}
            onChange={(e) => {
              console.log('Site changed to:', e.target.value);
              onSiteChange(e.target.value);
            }}
            variant="outlined"
            margin="normal"
            className="header-field"
            disabled={!clientName || isSitesLoading}
            error={!!siteError}
            helperText={siteError}
          >
            <MenuItem value="">Sélectionnez un site</MenuItem>
            {sites.map((site) => (
              <MenuItem key={site.id} value={site.name}>
                {site.name}
              </MenuItem>
            ))}
          </TextField>
          {isSitesLoading && (
            <CircularProgress
              size={24}
              style={{
                position: 'absolute',
                right: 8,
                top: '50%',
                marginTop: -12
              }}
            />
          )}
          {sites.length === 0 && clientName && !isSitesLoading && (
            <Typography variant="caption" color="text.secondary">
              Aucun site disponible pour ce client
            </Typography>
          )}
        </Box>

        <Box sx={{ flex: '1 1 220px' }}>
          <TextField
            fullWidth
            label="OBJET"
            value={object}
            onChange={(e) => onObjectChange(e.target.value)}
            variant="outlined"
            margin="normal"
            className="header-field"
            placeholder="Objet de l'intervention"
          />
        </Box>

        <Box sx={{ flex: '1 1 220px' }}>
          <TextField
            fullWidth
            label="DATE"
            type="date"
            value={date}
            onChange={(e) => onDateChange(e.target.value)}
            variant="outlined"
            margin="normal"
            className="header-field"
            InputLabelProps={{
              shrink: true,
            }}
          />
        </Box>
      </Box>
    </Paper>
  );
};

export default QuoteHeader;



============================================================
FILE: src/components/SuppliesSection/SuppliesSection.scss
============================================================
.supplies-section {
  padding: 1.5rem;
  margin-bottom: 2rem;

  @media print {
    padding: 0;
    margin: 0;
  }

  .section-title {
    font-weight: 700;
    color: #1976d2;
    margin-bottom: 1rem;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid #e0e0e0;

    @media print {
      margin: 0;
      padding: 0;
      border-bottom: none;
    }
  }

  .description-field {
    margin-bottom: 1rem;

    @media print {
      margin: 0;
    }
  }

  .rates-container {
    margin-bottom: 1.5rem;

    @media print {
      margin: 0;
    }

    .rates-container-item {
      width: 40px !important;
    }
  }

  .item-actions {
    display: flex;
    justify-content: flex-start;
    margin-bottom: 1rem;

    @media print {
      margin: 0;
      display: none;
    }

    .add-item-button {
      background-color: #4caf50;

      &:hover {
        background-color: #388e3c;
      }
    }
  }

  .items-table-container {
    margin-bottom: 1rem;

    @media print {
      margin: 0;
    }

    table {
      th {
        font-weight: 700;
        background-color: #f5f5f5;

        @media print {
          &:last-child {
            display: none;
          }
        }
      }

      td {
        @media print {
          &:last-child {
            display: none;
          }
        }
      }
    }
  }

  .total-container {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    padding: 0.5rem 1rem;
    background-color: #f5f5f5;
    border-radius: 4px;

    @media print {
      margin: 0;
      padding: 0;
      border-radius: 0;
    }

    .total-label {
      font-weight: 700;
      margin-right: 0.5rem;

      @media print {
        margin: 0;
      }
    }

    .total-value {
      font-weight: 700;
      color: #1976d2;
    }
  }
}

// Dialog styles
.search-container {
  margin-bottom: 1rem;

  @media print {
    display: none;
  }
}

.search-results-container {
  height: 300px;
  overflow-y: auto;
  margin-bottom: 1rem;

  @media print {
    display: none;
  }

  .selected-item {
    background-color: #e3f2fd;
  }
}

.quantity-container {
  margin-top: 1rem;
  padding: 1rem;
  background-color: #f5f5f5;
  border-radius: 4px;

  @media print {
    display: none;
  }
}


============================================================
FILE: src/components/SuppliesSection/SuppliesSection.tsx
============================================================
import React, { useState, useEffect } from 'react';
import {
  Box,
  Button,
  Dialog,
  DialogActions,
  DialogContent,
  DialogTitle,
  DialogContentText,
  IconButton,
  Paper,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  TextField,
  Typography,
  MenuItem,
  Alert,
  Chip
} from '@mui/material';
import {
  Add as AddIcon,
  Delete as DeleteIcon,
  Search as SearchIcon,
  Edit as EditIcon,
  Warning as WarningIcon
} from '@mui/icons-material';
import { SupplyItem } from '../../models/Quote';
import { apiService } from '../../services/api-service';
import { calculateSupplyItemTotal } from '../../utils/calculations';
import CustomNumberInput from '../CustomNumberInput/CustomNumberInput';
import './SuppliesSection.scss';

interface SuppliesSectionProps {
  items?: SupplyItem[];
  description: string;
  exchangeRate: number;
  marginRate: number;
  totalHT: number;
  onAddItem: (item: Omit<SupplyItem, 'id'>) => void;
  onRemoveItem: (id: string) => void;
  onUpdateDescription: (description: string) => void;
  onUpdateExchangeRate: (rate: number) => void;
  onUpdateMarginRate: (rate: number) => void;
  onUpdateSupplyItem: (item: SupplyItem) => void;
}

const SuppliesSection: React.FC<SuppliesSectionProps> = ({
  items = [],
  description,
  exchangeRate,
  marginRate,
  totalHT,
  onAddItem,
  onRemoveItem,
  onUpdateDescription,
  onUpdateExchangeRate,
  onUpdateMarginRate,
  onUpdateSupplyItem
}) => {
  const [searchDialogOpen, setSearchDialogOpen] = useState(false);
  const [catalogItems, setCatalogItems] = useState<SupplyItem[]>([]);
  const [filteredItems, setFilteredItems] = useState<SupplyItem[]>([]);
  const [searchTerm, setSearchTerm] = useState('');
  const [selectedItem, setSelectedItem] = useState<SupplyItem | null>(null);
  const [quantity, setQuantity] = useState(1);
  const [customPriceDialogOpen, setCustomPriceDialogOpen] = useState(false);
  const [customPrice, setCustomPrice] = useState<number>(0);
  const [isLoading, setIsLoading] = useState(false);
  const [editPriceDialogOpen, setEditPriceDialogOpen] = useState(false);
  const [editingItem, setEditingItem] = useState<SupplyItem | null>(null);
  const [editPrice, setEditPrice] = useState<number>(0);
  const [editQuantity, setEditQuantity] = useState<number>(1);
  const [quantityWarningDialogOpen, setQuantityWarningDialogOpen] = useState(false);
  const [pendingItem, setPendingItem] = useState<SupplyItem | null>(null);
  const [pendingQuantity, setPendingQuantity] = useState<number>(1);

  // Load catalog items on component mount
  useEffect(() => {
    const loadCatalogItems = async () => {
      try {
        setIsLoading(true);
        const loadedItems = await apiService.getSupplies();

        // Map API response to expected structure if needed
        const mappedItems = loadedItems.map(item => {
          const itemAny = item as any;
          const description = itemAny.description || itemAny.Description || itemAny.name || itemAny.Name || '';

          return {
            id: itemAny.id,
            description: description,
            priceEuro: parseFloat(itemAny.price) || 0,
            quantity: itemAny.quantity !== undefined && itemAny.quantity !== null ? Number(itemAny.quantity) : 0
          } as SupplyItem;
        });

        setCatalogItems(mappedItems);
        setFilteredItems(mappedItems);
      } catch (error) {
        console.error('Error loading catalog items:', error);
      } finally {
        setIsLoading(false);
      }
    };

    loadCatalogItems();
  }, []);

  // Filter items based on search term
  useEffect(() => {
    if (searchTerm.trim() === '') {
      setFilteredItems(catalogItems);
    } else {
      const filtered = catalogItems.filter(item =>
        item.description.toLowerCase().includes(searchTerm.toLowerCase())
      );
      setFilteredItems(filtered);
    }
  }, [searchTerm, catalogItems]);

  // Calculate dollar prices for all items when rates change
  useEffect(() => {
    const itemsWithDollarPrices = items.map(item =>
      calculateSupplyItemTotal(item, exchangeRate, marginRate)
    );
    // Update items with calculated dollar prices
    itemsWithDollarPrices.forEach(item => {
      const existingItem = items.find(i => i.id === item.id);
      if (existingItem) {
        existingItem.priceDollar = item.priceDollar;
        existingItem.unitPriceDollar = item.unitPriceDollar;
        existingItem.totalPriceDollar = item.totalPriceDollar;
      }
    });
  }, [items, exchangeRate, marginRate]);

  // Handle opening the search dialog
  const handleOpenSearchDialog = () => {
    setSearchDialogOpen(true);
    setSearchTerm('');
    setSelectedItem(null);
    setQuantity(1);
  };

  // Handle closing the search dialog
  const handleCloseSearchDialog = () => {
    setSearchDialogOpen(false);
  };

  // Handle selecting an item from the catalog
  const handleSelectItem = (item: SupplyItem) => {
    setSelectedItem(item);
  };

  // Handle closing the custom price dialog
  const handleCloseCustomPriceDialog = () => {
    setCustomPriceDialogOpen(false);
    setCustomPrice(0);
  };

  // Handle adding an item with custom price
  const handleAddItemWithCustomPrice = () => {
    if (selectedItem && customPrice > 0) {
      const itemWithCustomPrice = {
        ...selectedItem,
        priceEuro: customPrice
      };
      const calculatedItem = calculateSupplyItemTotal(itemWithCustomPrice, exchangeRate, marginRate);
      onAddItem({
        description: calculatedItem.description,
        quantity: quantity,
        priceEuro: calculatedItem.priceEuro,
        priceDollar: calculatedItem.priceDollar,
        unitPriceDollar: calculatedItem.unitPriceDollar,
        totalPriceDollar: calculatedItem.totalPriceDollar
      });
      handleCloseCustomPriceDialog();
    }
  };

  // Handle adding a regular item
  const handleAddItem = () => {
    if (selectedItem) {
      // Check if selected quantity exceeds available quantity
      if (quantity > selectedItem.quantity && selectedItem.quantity > 0) {
        setPendingItem(selectedItem);
        setPendingQuantity(quantity);
        setQuantityWarningDialogOpen(true);
        return;
      }

      // Proceed with adding item
      addItemToQuote(selectedItem, quantity);
    }
  };

  // Helper function to add item to quote
  const addItemToQuote = (item: SupplyItem, qty: number) => {
    const calculatedItem = calculateSupplyItemTotal(item, exchangeRate, marginRate);
    onAddItem({
      description: calculatedItem.description,
      quantity: qty,
      priceEuro: calculatedItem.priceEuro,
      priceDollar: calculatedItem.priceDollar,
      unitPriceDollar: calculatedItem.unitPriceDollar,
      totalPriceDollar: calculatedItem.totalPriceDollar
    });
    handleCloseSearchDialog();
  };

  // Handle quantity warning confirmation
  const handleQuantityWarningConfirm = () => {
    if (pendingItem) {
      addItemToQuote(pendingItem, pendingQuantity);
      setQuantityWarningDialogOpen(false);
      setPendingItem(null);
      setPendingQuantity(1);
    }
  };

  // Handle quantity warning cancel
  const handleQuantityWarningCancel = () => {
    setQuantityWarningDialogOpen(false);
    setPendingItem(null);
    setPendingQuantity(1);
  };

  const handleOpenEditPriceDialog = (item: SupplyItem) => {
    setEditingItem(item);
    setEditPrice(item.priceEuro);
    setEditQuantity(item.quantity);
    setEditPriceDialogOpen(true);
  };

  const handleCloseEditPriceDialog = () => {
    setEditPriceDialogOpen(false);
    setEditingItem(null);
    setEditPrice(0);
    setEditQuantity(1);
  };

  const handleUpdatePrice = () => {
    if (editingItem) {
      const updatedItem = {
        ...editingItem,
        priceEuro: editPrice,
        quantity: editQuantity
      };
      onUpdateSupplyItem(updatedItem);
      handleCloseEditPriceDialog();
    }
  };

  return (
    <Paper className="supplies-section" elevation={2}>
      <Typography variant="h6" className="section-title">
        FOURNITURES
      </Typography>

      <TextField
        select
        fullWidth
        label="Description des fournitures"
        value={description}
        onChange={(e) => onUpdateDescription(e.target.value)}
        variant="outlined"
        margin="none"
        className="description-field"
      >
        <MenuItem value="Extension des raccordements électriques, frigorifique et condensat">
          Extension des raccordements électriques, frigorifique et condensat
        </MenuItem>
      </TextField>

      <Box sx={{ display: 'flex', flexWrap: 'wrap', gap: 2 }} className="rates-container">
        <Box className="rates-container-item" sx={{ flex: '1 1 50px' }}>
          <CustomNumberInput
            value={exchangeRate}
            label="Taux de change"
            min={0}
            step={0.01}
            fullWidth
            displayOnly={true}
            onChange={() => {}}
          />
        </Box>
        <Box className="rates-container-item" sx={{ flex: '1 1 50px' }}>
          <CustomNumberInput
            value={marginRate}
            label="Taux de marge"
            min={0}
            max={1}
            step={0.01}
            fullWidth
            displayOnly={true}
            onChange={() => {}}
          />
        </Box>
      </Box>

      <Box className="item-actions">
        <Button
          variant="contained"
          color="primary"
          startIcon={<AddIcon />}
          onClick={handleOpenSearchDialog}
          className="add-item-button"
        >
          Ajouter un article
        </Button>
      </Box>

      <TableContainer className="items-table-container">
        <Table size="small" aria-label="supplies table">
          <TableHead>
            <TableRow>
              <TableCell>N°</TableCell>
              <TableCell>Description</TableCell>
              <TableCell align="right">Qté</TableCell>
              <TableCell align="right">PR €</TableCell>
              <TableCell align="right">PR $</TableCell>
              <TableCell align="right">PV/u $</TableCell>
              <TableCell align="right">PV $ Total HT</TableCell>
              <TableCell align="center">Actions</TableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            {items.length === 0 ? (
              <TableRow>
                <TableCell colSpan={8} align="center">
                  Aucun article ajouté
                </TableCell>
              </TableRow>
            ) : (
              items.map((item, index) => (
                <TableRow key={item.id}>
                  <TableCell>{index + 1}</TableCell>
                  <TableCell>{item.description}</TableCell>
                  <TableCell align="right">{item.quantity}</TableCell>
                  <TableCell align="right">{Number(item.priceEuro || 0).toFixed(2)}</TableCell>
                  <TableCell align="right">{Number(item.priceDollar || 0).toFixed(2)}</TableCell>
                  <TableCell align="right">{Number(item.unitPriceDollar || 0).toFixed(2)}</TableCell>
                  <TableCell align="right">{Number(item.totalPriceDollar || 0).toFixed(2)}</TableCell>
                  <TableCell align="center">
                    <IconButton
                      size="small"
                      color="primary"
                      onClick={() => handleOpenEditPriceDialog(item)}
                    >
                      <EditIcon fontSize="small" />
                    </IconButton>
                    <IconButton
                      size="small"
                      color="error"
                      onClick={() => onRemoveItem(item.id)}
                    >
                      <DeleteIcon fontSize="small" />
                    </IconButton>
                  </TableCell>
                </TableRow>
              ))
            )}
          </TableBody>
        </Table>
      </TableContainer>

      <Box className="total-container">
        <Typography variant="subtitle1" className="total-label">
          TOTAL FOURNITURE $ HT:
        </Typography>
        <Typography variant="subtitle1" className="total-value">
          {Number(totalHT ?? 0).toFixed(2)}
        </Typography>
      </Box>

      {/* Item Search Dialog */}
      <Dialog
        open={searchDialogOpen}
        onClose={handleCloseSearchDialog}
        maxWidth="md"
        fullWidth
      >
        <DialogTitle>Ajouter un article</DialogTitle>
        <DialogContent>
          <Box className="search-container">
            <TextField
              fullWidth
              label="Rechercher un article"
              value={searchTerm}
              onChange={(e) => setSearchTerm(e.target.value)}
              variant="outlined"
              margin="normal"
              InputProps={{
                startAdornment: <SearchIcon color="action" />
              }}
            />
          </Box>

          <TableContainer className="search-results-container">
            <Table size="small">
              <TableHead>
                <TableRow>
                  <TableCell>Description</TableCell>
                  <TableCell align="right">Prix €</TableCell>
                  <TableCell align="center">Action</TableCell>
                </TableRow>
              </TableHead>
              <TableBody>
                {filteredItems.length === 0 ? (
                  <TableRow>
                    <TableCell colSpan={3} align="center">
                      Aucun article trouvé
                    </TableCell>
                  </TableRow>
                ) : (
                  filteredItems.map((item) => (
                    <TableRow
                      key={item.id}
                      className={selectedItem?.id === item.id ? 'selected-item' : ''}
                      onClick={() => handleSelectItem(item)}
                    >
                      <TableCell>
                        <Box sx={{ display: 'flex', alignItems: 'center', gap: 1 }}>
                          {item.description}
                          {item.quantity === 0 && (
                            <Chip
                              icon={<WarningIcon />}
                              label="Stock épuisé"
                              size="small"
                              color="warning"
                              variant="outlined"
                            />
                          )}
                        </Box>
                      </TableCell>
                      <TableCell align="right">{Number(item.priceEuro || 0).toFixed(2)}</TableCell>
                      <TableCell align="center">
                        <Button
                          size="small"
                          variant="outlined"
                          color="primary"
                          onClick={() => handleSelectItem(item)}
                        >
                          Sélectionner
                        </Button>
                      </TableCell>
                    </TableRow>
                  ))
                )}
              </TableBody>
            </Table>
          </TableContainer>

          {selectedItem && (
            <Box className="quantity-container">
              <Typography variant="subtitle1">
                Article sélectionné: {selectedItem.description}
              </Typography>
              <CustomNumberInput
                label="Quantité"
                value={quantity}
                onChange={setQuantity}
                min={1}
                step={1}
                fullWidth
                margin="normal"
              />
            </Box>
          )}
        </DialogContent>
        <DialogActions>
          <Button onClick={handleCloseSearchDialog} color="primary">
            Annuler
          </Button>
          <Button
            onClick={handleAddItem}
            color="primary"
            variant="contained"
            disabled={!selectedItem}
          >
            Ajouter
          </Button>
        </DialogActions>
      </Dialog>

      {/* Custom Price Dialog */}
      <Dialog
        open={customPriceDialogOpen}
        onClose={() => setCustomPriceDialogOpen(false)}
        maxWidth="xs"
        fullWidth
      >
        <DialogTitle>Entrer un prix personnalisé</DialogTitle>
        <DialogContent>
          <Typography variant="body2" color="textSecondary" paragraph>
            L'article "{selectedItem?.description}" a un prix de 0.00€.
            Veuillez entrer un prix personnalisé pour cet article.
          </Typography>
          <CustomNumberInput
            label="Prix (€)"
            value={customPrice}
            onChange={setCustomPrice}
            min={0.01}
            step={0.01}
            fullWidth
            margin="normal"
          />
        </DialogContent>
        <DialogActions>
          <Button onClick={() => setCustomPriceDialogOpen(false)} color="primary">
            Annuler
          </Button>
          <Button
            onClick={handleAddItemWithCustomPrice}
            color="primary"
            variant="contained"
            disabled={customPrice <= 0}
          >
            Ajouter
          </Button>
        </DialogActions>
      </Dialog>

      {/* Edit Price Dialog */}
      <Dialog
        open={editPriceDialogOpen}
        onClose={handleCloseEditPriceDialog}
        maxWidth="xs"
        fullWidth
      >
        <DialogTitle>Modifier le prix et la quantité</DialogTitle>
        <DialogContent>
          <Typography variant="body2" color="textSecondary" paragraph>
            Article: {editingItem?.description}
          </Typography>
          <CustomNumberInput
            label="Prix (€)"
            value={editPrice}
            onChange={setEditPrice}
            min={0.01}
            step={0.01}
            fullWidth
            margin="normal"
          />
          <CustomNumberInput
            label="Quantité"
            value={editQuantity}
            onChange={setEditQuantity}
            min={1}
            step={1}
            fullWidth
            margin="normal"
          />
        </DialogContent>
        <DialogActions>
          <Button onClick={handleCloseEditPriceDialog} color="primary">
            Annuler
          </Button>
          <Button
            onClick={handleUpdatePrice}
            color="primary"
            variant="contained"
            disabled={editPrice <= 0 || editQuantity < 1}
          >
            Mettre à jour
          </Button>
        </DialogActions>
      </Dialog>

      {/* Quantity Warning Dialog */}
      <Dialog
        open={quantityWarningDialogOpen}
        onClose={handleQuantityWarningCancel}
        maxWidth="sm"
        fullWidth
      >
        <DialogTitle>
          <Box sx={{ display: 'flex', alignItems: 'center', gap: 1 }}>
            <WarningIcon color="warning" />
            Quantité insuffisante
          </Box>
        </DialogTitle>
        <DialogContent>
          <DialogContentText>
            Vous avez sélectionné <strong>{pendingQuantity}</strong> unités de <strong>"{pendingItem?.description}"</strong>,
            mais seulement <strong>{pendingItem?.quantity}</strong> unités sont disponibles en stock.
          </DialogContentText>
          <DialogContentText sx={{ mt: 2 }}>
            Voulez-vous quand même ajouter cet article avec la quantité demandée ?
          </DialogContentText>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleQuantityWarningCancel} color="primary">
            Annuler
          </Button>
          <Button
            onClick={handleQuantityWarningConfirm}
            color="warning"
            variant="contained"
          >
            Ajouter quand même
          </Button>
        </DialogActions>
      </Dialog>
    </Paper>
  );
};

export default SuppliesSection;

============================================================
FILE: src/components/TotalSection/TotalSection.scss
============================================================
.total-section {
  padding: 1.5rem;
  margin-bottom: 2rem;
  width: 50%;
  margin-left: auto;

  @media (max-width: 768px) {
    width: 100%;
  }

  @media print {
    padding: 0;
    margin-right: 50%;
    width: 50%;
  }

  .section-title {
    font-weight: 700;
    color: #1976d2;
    margin-bottom: 1rem;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid #e0e0e0;

    @media print {
      padding: 0;
      border-bottom: none;
    }
  }

  table {
    .total-label {
      font-weight: 600;
      color: #333;
    }

    .total-value {
      font-weight: 600;
      min-width: 100px;
    }

    .grand-total-row {
      background-color: #f5f5f5;
    }

    .grand-total {
      font-weight: 700;
      color: #1976d2;
      font-size: 1.1rem;
    }
  }
}

// Print styles


============================================================
FILE: src/components/TotalSection/TotalSection.tsx
============================================================
import React from 'react';
import { Paper, Table, TableBody, TableCell, TableContainer, TableRow, Typography } from '@mui/material';
import './TotalSection.scss';

interface TotalSectionProps {
  totalSuppliesHT: number;
  totalLaborHT: number;
  totalHT: number;
  tva: number;
  totalTTC: number;
}

const TotalSection: React.FC<TotalSectionProps> = ({
  totalSuppliesHT,
  totalLaborHT,
  totalHT,
  tva,
  totalTTC
}) => {
  return (
    <Paper className="total-section" elevation={2}>

      <TableContainer>
        <Table size="small" aria-label="totals table">
          <TableBody>
            <TableRow>
              <TableCell className="total-label">TOTAL OFFRE USD HT:</TableCell>
              <TableCell align="right" className="total-value">{Number(totalHT ?? 0).toFixed(2)}</TableCell>
            </TableRow>
            <TableRow>
              <TableCell className="total-label">TVA:</TableCell>
              <TableCell align="right" className="total-value">{Number(tva ?? 0).toFixed(2)}</TableCell>
            </TableRow>
            <TableRow className="grand-total-row">
              <TableCell className="total-label grand-total">TOTAL OFFRE USD TTC:</TableCell>
              <TableCell align="right" className="total-value grand-total">{Number(totalTTC ?? 0).toFixed(2)}</TableCell>
            </TableRow>
          </TableBody>
        </Table>
      </TableContainer>
    </Paper>
  );
};

export default TotalSection;

============================================================
FILE: src/components/ClientList.js
============================================================
import React, { useState } from 'react';
import { useClients } from '../hooks/useClients';

export const ClientList = () => {
    const { clients, loading, error, createClient } = useClients();
    const [formData, setFormData] = useState({
        clientName: '',
        siteName: '',
        siteAddress: ''
    });
    const [formError, setFormError] = useState('');

    const handleChange = (e) => {
        const { name, value } = e.target;
        setFormData(prev => ({
            ...prev,
            [name]: value
        }));
    };

    const handleSubmit = async (e) => {
        e.preventDefault();
        setFormError('');

        try {
            if (!formData.clientName.trim()) {
                setFormError('Client name is required');
                return;
            }
            if (!formData.siteName.trim()) {
                setFormError('Site name is required');
                return;
            }
            if (!formData.siteAddress.trim()) {
                setFormError('Site address is required');
                return;
            }

            await createClient({
                name: formData.clientName.trim(),
                site: {
                    name: formData.siteName.trim(),
                    address: formData.siteAddress.trim()
                }
            });

            // Clear form after successful creation
            setFormData({
                clientName: '',
                siteName: '',
                siteAddress: ''
            });
        } catch (err) {
            setFormError(err.message || 'Failed to create client');
            console.error('Failed to create client:', err);
        }
    };

    return (
        <div className="container mx-auto p-4">
            <h1 className="text-2xl font-bold mb-4">Clients List</h1>

            {/* Add Client Form */}
            <form onSubmit={handleSubmit} className="mb-6">
                <div className="flex flex-col gap-4">
                    {formError && (
                        <div className="text-red-500 text-sm">{formError}</div>
                    )}
                    {error && (
                        <div className="text-red-500 text-sm">{error}</div>
                    )}
                    <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                        <div>
                            <label className="block text-sm font-medium text-gray-700 mb-1">
                                Client Name
                            </label>
                            <input
                                type="text"
                                name="clientName"
                                value={formData.clientName}
                                onChange={handleChange}
                                placeholder="Client Name"
                                className="border p-2 rounded w-full"
                                required
                            />
                        </div>
                        <div>
                            <label className="block text-sm font-medium text-gray-700 mb-1">
                                Site Name
                            </label>
                            <input
                                type="text"
                                name="siteName"
                                value={formData.siteName}
                                onChange={handleChange}
                                placeholder="Site Name"
                                className="border p-2 rounded w-full"
                                required
                            />
                        </div>
                        <div>
                            <label className="block text-sm font-medium text-gray-700 mb-1">
                                Site Address
                            </label>
                            <input
                                type="text"
                                name="siteAddress"
                                value={formData.siteAddress}
                                onChange={handleChange}
                                placeholder="Site Address"
                                className="border p-2 rounded w-full"
                                required
                            />
                        </div>
                    </div>
                    <div className="flex justify-end">
                        <button
                            type="submit"
                            className="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600 disabled:bg-blue-300"
                            disabled={loading}
                        >
                            {loading ? 'Adding...' : 'Add Client with Site'}
                        </button>
                    </div>
                </div>
            </form>

            {/* Clients Table */}
            {loading && <div>Loading...</div>}
            {!loading && clients.length === 0 ? (
                <div className="text-gray-500">No clients found</div>
            ) : (
                <table className="min-w-full bg-white">
                    <thead>
                        <tr>
                            <th className="text-left p-4">Client Name</th>
                            <th className="text-left p-4">Site Name</th>
                            <th className="text-left p-4">Site Address</th>
                        </tr>
                    </thead>
                    <tbody>
                        {clients.map((client) => (
                            <tr key={client.id} className="border-t">
                                <td className="p-4">{client.name}</td>
                                <td className="p-4">{client.site?.name || 'No site'}</td>
                                <td className="p-4">{client.site?.address || 'No address'}</td>
                            </tr>
                        ))}
                    </tbody>
                </table>
            )}
        </div>
    );
};

============================================================
FILE: src/components/ItemsList.js
============================================================
import React, { useState } from 'react';
import { useItems } from '../hooks/useItems';

export const ItemsList = () => {
    const { items, loading, error, createItem, updateItem, deleteItem } = useItems();
    const [newItemDescription, setNewItemDescription] = useState('');
    const [newItemPrice, setNewItemPrice] = useState('');
    const [formError, setFormError] = useState('');

    const handleSubmit = async (e) => {
        e.preventDefault();
        setFormError('');

        try {
            if (!newItemDescription || !newItemPrice) {
                setFormError('Description and price are required');
                return;
            }

            const price = parseFloat(newItemPrice);
            if (isNaN(price) || price <= 0) {
                setFormError('Please enter a valid price');
                return;
            }

            await createItem({
                description: newItemDescription,
                price: price
            });

            // Clear form after successful creation
            setNewItemDescription('');
            setNewItemPrice('');
        } catch (err) {
            setFormError(err.message || 'Failed to create item');
            console.error('Failed to create item:', err);
        }
    };

    return (
        <div className="container mx-auto p-4">
            <h1 className="text-2xl font-bold mb-4">Items List</h1>

            {/* Add Item Form */}
            <form onSubmit={handleSubmit} className="mb-6">
                <div className="flex flex-col gap-4">
                    {formError && (
                        <div className="text-red-500 text-sm">{formError}</div>
                    )}
                    {error && (
                        <div className="text-red-500 text-sm">{error}</div>
                    )}
                    <div className="flex gap-4">
                        <input
                            type="text"
                            value={newItemDescription}
                            onChange={(e) => setNewItemDescription(e.target.value)}
                            placeholder="Item Description"
                            className="border p-2 rounded flex-1"
                            required
                        />
                        <input
                            type="number"
                            value={newItemPrice}
                            onChange={(e) => setNewItemPrice(e.target.value)}
                            placeholder="Price"
                            className="border p-2 rounded w-32"
                            step="0.01"
                            min="0"
                            required
                        />
                        <button
                            type="submit"
                            className="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600 disabled:bg-blue-300"
                            disabled={loading}
                        >
                            {loading ? 'Adding...' : 'Add Item'}
                        </button>
                    </div>
                </div>
            </form>

            {/* Items Table */}
            {loading && <div>Loading...</div>}
            {!loading && items.length === 0 ? (
                <div className="text-gray-500">No items found</div>
            ) : (
                <table className="min-w-full bg-white">
                    <thead>
                        <tr>
                            <th className="text-left p-4">Description</th>
                            <th className="text-left p-4">Price</th>
                            <th className="text-left p-4">Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        {items.map((item) => (
                            <tr key={item.id} className="border-t">
                                <td className="p-4">{item.description}</td>
                                <td className="p-4">${item.price.toFixed(2)}</td>
                                <td className="p-4">
                                    <button
                                        onClick={() => deleteItem(item.id)}
                                        className="bg-red-500 text-white px-3 py-1 rounded hover:bg-red-600 mr-2"
                                        disabled={loading}
                                    >
                                        Delete
                                    </button>
                                </td>
                            </tr>
                        ))}
                    </tbody>
                </table>
            )}
        </div>
    );
};

============================================================
FILE: src/contexts/QuoteContext.tsx
============================================================
import React, { createContext, useContext, useReducer, ReactNode } from 'react';
import {
  LaborItem,
  Quote,
  SupplyItem
} from '../models/Quote';
import { apiService } from '../services/api-service';
import {
  calculateLaborItemTotal,
  calculateSupplyItemTotal,
  calculateTotalLabor,
  calculateTotalSupplies,
  calculateTotalTTC,
  calculateVAT
} from '../utils/calculations';
import { generateId, generateQuoteId, extractBaseId, extractVersion, getNextVersion } from '../utils/id-generator';

// Default values
const DEFAULT_EXCHANGE_RATE = 1.2;
const DEFAULT_MARGIN_RATE = 0.8;
const DEFAULT_LABOR_EXCHANGE_RATE = 1.2;
const DEFAULT_LABOR_MARGIN_RATE = 0.8;
const DEFAULT_DESCRIPTION = "";

// State interface
interface QuoteState {
  currentQuote: Quote | null;
  isEditing: boolean;
  isLoading: boolean;
  error: string | null;
  isExistingQuote: boolean;
  originalQuoteId: string | null;
}

// Action types
type QuoteAction =
  | { type: 'SET_QUOTE'; payload: Quote }
  | { type: 'SET_EXISTING_QUOTE'; payload: boolean }
  | { type: 'CLEAR_QUOTE' }
  | { type: 'START_EDIT' }
  | { type: 'CANCEL_EDIT' }
  | { type: 'SET_LOADING'; payload: boolean }
  | { type: 'SET_ERROR'; payload: string | null }
  | { type: 'UPDATE_QUOTE_FIELD'; payload: { field: keyof Quote; value: any } }
  | { type: 'ADD_SUPPLY_ITEM'; payload: Omit<SupplyItem, 'id'> }
  | { type: 'UPDATE_SUPPLY_ITEM'; payload: SupplyItem }
  | { type: 'REMOVE_SUPPLY_ITEM'; payload: string }
  | { type: 'ADD_LABOR_ITEM'; payload: Omit<LaborItem, 'id'> }
  | { type: 'UPDATE_LABOR_ITEM'; payload: LaborItem }
  | { type: 'REMOVE_LABOR_ITEM'; payload: string }
  | { type: 'RECALCULATE_TOTALS' }
  | { type: 'SET_ORIGINAL_QUOTE_ID'; payload: string | null };

// Initial state
const initialState: QuoteState = {
  currentQuote: null,
  isEditing: false,
  isLoading: false,
  error: null,
  isExistingQuote: false,
  originalQuoteId: null,
};

// Reducer function
const quoteReducer = (state: QuoteState, action: QuoteAction): QuoteState => {
  switch (action.type) {
    case 'SET_QUOTE':
      return {
        ...state,
        currentQuote: action.payload,
        isEditing: false,
        error: null,
      };

    case 'SET_EXISTING_QUOTE':
      return {
        ...state,
        isExistingQuote: action.payload,
      };

    case 'CLEAR_QUOTE':
      return {
        ...state,
        currentQuote: null,
        isEditing: false,
        isExistingQuote: false,
      };

    case 'START_EDIT':
      return {
        ...state,
        isEditing: true,
      };

    case 'CANCEL_EDIT':
      return {
        ...state,
        isEditing: false,
      };

    case 'SET_LOADING':
      return {
        ...state,
        isLoading: action.payload,
      };

    case 'SET_ERROR':
      return {
        ...state,
        error: action.payload,
        isLoading: false,
      };

    case 'UPDATE_QUOTE_FIELD':
      if (!state.currentQuote) return state;

      return {
        ...state,
        currentQuote: {
          ...state.currentQuote,
          [action.payload.field]: action.payload.value,
        },
      };

    case 'ADD_SUPPLY_ITEM': {
      if (!state.currentQuote) return state;

      const newItem: SupplyItem = {
        id: generateId(),
        ...action.payload,
      };

      // Calculate prices
      const calculatedItem = calculateSupplyItemTotal(
        newItem,
        state.currentQuote.supplyExchangeRate,
        state.currentQuote.supplyMarginRate
      );

      const updatedItems = [...state.currentQuote.supplyItems, calculatedItem];
      const totalSuppliesHT = calculateTotalSupplies(updatedItems);
      const totalHT = Number(totalSuppliesHT) + Number(state.currentQuote.totalLaborHT);
      const tva = calculateVAT(totalHT);
      const totalTTC = totalHT + tva;

      return {
        ...state,
        currentQuote: {
          ...state.currentQuote,
          supplyItems: updatedItems,
          totalSuppliesHT,
          totalHT,
          tva,
          totalTTC,
        },
      };
    }

    case 'UPDATE_SUPPLY_ITEM': {
      if (!state.currentQuote) return state;

      const updatedItems = (state.currentQuote.supplyItems ?? []).map(item =>
        item.id === action.payload.id
          ? calculateSupplyItemTotal(
              action.payload,
              state.currentQuote!.supplyExchangeRate,
              state.currentQuote!.supplyMarginRate
            )
          : item
      );

      const totalSuppliesHT = calculateTotalSupplies(updatedItems);
      const totalHT = Number(totalSuppliesHT) + Number(state.currentQuote.totalLaborHT);
      const tva = calculateVAT(totalHT);
      const totalTTC = totalHT + tva;

      return {
        ...state,
        currentQuote: {
          ...state.currentQuote,
          supplyItems: updatedItems,
          totalSuppliesHT,
          totalHT,
          tva,
          totalTTC,
        },
      };
    }

    case 'REMOVE_SUPPLY_ITEM': {
      if (!state.currentQuote) return state;

      const updatedItems = state.currentQuote.supplyItems.filter(
        item => item.id !== action.payload
      );

      const totalSuppliesHT = calculateTotalSupplies(updatedItems);
      const totalHT = Number(totalSuppliesHT) + Number(state.currentQuote.totalLaborHT);
      const tva = calculateVAT(totalHT);
      const totalTTC = totalHT + tva;

      return {
        ...state,
        currentQuote: {
          ...state.currentQuote,
          supplyItems: updatedItems,
          totalSuppliesHT,
          totalHT,
          tva,
          totalTTC,
        },
      };
    }

    case 'ADD_LABOR_ITEM': {
      if (!state.currentQuote) return state;

      const newItem: LaborItem = {
        id: generateId(),
        ...action.payload,
      };

      // Calculate prices
      const calculatedItem = calculateLaborItemTotal(
        newItem,
        state.currentQuote.laborExchangeRate,
        state.currentQuote.laborMarginRate
      );

      const updatedItems = [...state.currentQuote.laborItems, calculatedItem];
      const totalLaborHT = calculateTotalLabor(updatedItems);
      const totalHT = Number(totalLaborHT) + Number(state.currentQuote.totalSuppliesHT);
      const tva = calculateVAT(totalHT);
      const totalTTC = totalHT + tva;

      return {
        ...state,
        currentQuote: {
          ...state.currentQuote,
          laborItems: updatedItems,
          totalLaborHT,
          totalHT,
          tva,
          totalTTC,
        },
      };
    }

    case 'UPDATE_LABOR_ITEM': {
      if (!state.currentQuote) return state;

      const updatedItems = (state.currentQuote.laborItems ?? []).map(item =>
        item.id === action.payload.id
          ? calculateLaborItemTotal(
              action.payload,
              state.currentQuote!.laborExchangeRate,
              state.currentQuote!.laborMarginRate
            )
          : item
      );

      const totalLaborHT = calculateTotalLabor(updatedItems);
      const totalHT = Number(totalLaborHT) + Number(state.currentQuote.totalSuppliesHT);
      const tva = calculateVAT(totalHT);
      const totalTTC = totalHT + tva;

      return {
        ...state,
        currentQuote: {
          ...state.currentQuote,
          laborItems: updatedItems,
          totalLaborHT,
          totalHT,
          tva,
          totalTTC,
        },
      };
    }

    case 'REMOVE_LABOR_ITEM': {
      if (!state.currentQuote) return state;

      const updatedItems = state.currentQuote.laborItems.filter(
        item => item.id !== action.payload
      );

      const totalLaborHT = calculateTotalLabor(updatedItems);
      const totalHT = Number(totalLaborHT) + Number(state.currentQuote.totalSuppliesHT);
      const tva = calculateVAT(totalHT);
      const totalTTC = totalHT + tva;

      return {
        ...state,
        currentQuote: {
          ...state.currentQuote,
          laborItems: updatedItems,
          totalLaborHT,
          totalHT,
          tva,
          totalTTC,
        },
      };
    }

    case 'RECALCULATE_TOTALS': {
      if (!state.currentQuote) return state;

      const recalculatedSupplyItems = (state.currentQuote.supplyItems ?? []).map(item =>
        calculateSupplyItemTotal(
          item,
          state.currentQuote!.supplyExchangeRate,
          state.currentQuote!.supplyMarginRate
        )
      );

      const recalculatedLaborItems = (state.currentQuote.laborItems ?? []).map(item =>
        calculateLaborItemTotal(
          item,
          state.currentQuote!.laborExchangeRate,
          state.currentQuote!.laborMarginRate
        )
      );

      // Recalculate totals
      const totalSuppliesHT = calculateTotalSupplies(recalculatedSupplyItems);
      const totalLaborHT = calculateTotalLabor(recalculatedLaborItems);
      const totalHT = Number(totalSuppliesHT) + Number(totalLaborHT);
      const tva = calculateVAT(totalHT);
      const totalTTC = totalHT + tva;

      return {
        ...state,
        currentQuote: {
          ...state.currentQuote,
          supplyItems: recalculatedSupplyItems,
          laborItems: recalculatedLaborItems,
          totalSuppliesHT,
          totalLaborHT,
          totalHT,
          tva,
          totalTTC,
        },
      };
    }

    case 'SET_ORIGINAL_QUOTE_ID': {
      return {
        ...state,
        originalQuoteId: action.payload,
      };
    }

    default:
      return state;
  }
};

// Context interface
interface QuoteContextProps {
  state: QuoteState;
  createNewQuote: () => void;
  loadQuote: (id: string, createdAt: string, fromHistory?: boolean) => void;
  saveQuote: () => Promise<boolean>;
  updateQuote: () => Promise<boolean>;
  setQuoteField: <K extends keyof Quote>(field: K, value: Quote[K]) => void;
  addSupplyItem: (item: Omit<SupplyItem, 'id'>) => void;
  updateSupplyItem: (item: SupplyItem) => void;
  removeSupplyItem: (id: string) => void;
  addLaborItem: (item: Omit<LaborItem, 'id'>) => void;
  updateLaborItem: (item: LaborItem) => void;
  removeLaborItem: (id: string) => void;
  recalculateTotals: () => void;
  clearQuote: () => void;
}

// Create context
const QuoteContext = createContext<QuoteContextProps | undefined>(undefined);

// Provider props
interface QuoteProviderProps {
  children: ReactNode;
}

// Provider component
export const QuoteProvider: React.FC<QuoteProviderProps> = ({ children }) => {
  const [state, dispatch] = useReducer(quoteReducer, initialState);

  // Create a new quote
  const createNewQuote = async () => {
    let exchangeRate = DEFAULT_EXCHANGE_RATE;
    try {
      exchangeRate = await apiService.getExchangeRate('EUR', 'USD');
      exchangeRate = Math.round(exchangeRate * 1000) / 1000;
    } catch (e) {
      console.warn('Failed to fetch real-time exchange rate, using default:', e);
    }
    const newQuote: Quote = {
      id: generateQuoteId(), // This will create ID with version 000
      clientName: '',
      siteName: '',
      object: '',
      date: new Date().toISOString().split('T')[0],
      supplyDescription: DEFAULT_DESCRIPTION,
      laborDescription: DEFAULT_DESCRIPTION,
      supplyExchangeRate: exchangeRate,
      supplyMarginRate: DEFAULT_MARGIN_RATE,
      laborExchangeRate: exchangeRate,
      laborMarginRate: DEFAULT_LABOR_MARGIN_RATE,
      supplyItems: [],
      laborItems: [],
      totalSuppliesHT: 0,
      totalLaborHT: 0,
      totalHT: 0,
      tva: 0,
      totalTTC: 0,
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString(),
      version: 0,
      totalAmount: 0,
      status: 'draft',
      metadata: {
        createdAt: new Date().toISOString(),
        updatedAt: new Date().toISOString(),
        version: 0
      }
    };

    dispatch({ type: 'SET_QUOTE', payload: newQuote });
    dispatch({ type: 'SET_EXISTING_QUOTE', payload: false });
  };

  // Load a quote by ID
  const loadQuote = async (id: string, createdAt: string, fromHistory: boolean = false) => {
    dispatch({ type: 'SET_LOADING', payload: true });
    try {
      const quote = await apiService.getQuoteById(id);
      if (!quote) {
        throw new Error('Quote not found');
      }
      dispatch({ type: 'SET_ORIGINAL_QUOTE_ID', payload: id });
      dispatch({ type: 'SET_QUOTE', payload: quote });
      dispatch({ type: 'SET_EXISTING_QUOTE', payload: true });
      dispatch({ type: 'SET_ERROR', payload: null });
    } catch (error) {
      dispatch({ type: 'SET_ERROR', payload: error instanceof Error ? error.message : 'Failed to load quote' });
      dispatch({ type: 'CLEAR_QUOTE' });
    } finally {
      dispatch({ type: 'SET_LOADING', payload: false });
    }
  };

  // Save or update quote
  const saveQuote = async (): Promise<boolean> => {
    if (!state.currentQuote) return false;
    try {
      dispatch({ type: 'SET_LOADING', payload: true });

      let quoteToSave = { ...state.currentQuote };
      let newId = state.currentQuote.id;
      let isUpdate = false;
      let parentId: string | undefined = undefined;

      // Always refresh exchange rate before saving
      let exchangeRate = DEFAULT_EXCHANGE_RATE;
      try {
        exchangeRate = await apiService.getExchangeRate('EUR', 'USD');
        exchangeRate = Math.round(exchangeRate * 1000) / 1000;
      } catch (e) {
        console.warn('Failed to fetch real-time exchange rate, using default:', e);
      }
      quoteToSave.supplyExchangeRate = exchangeRate;
      quoteToSave.laborExchangeRate = exchangeRate;

      // If it's a new quote (no ID or version 0), create it
      if (!state.currentQuote.id || state.currentQuote.version === 0) {
        // Create new quote
        quoteToSave = {
          ...quoteToSave,
          parentId: "",
        };
        const savedQuote = await apiService.saveQuote(quoteToSave);
        dispatch({ type: 'SET_QUOTE', payload: savedQuote });
        dispatch({ type: 'SET_EXISTING_QUOTE', payload: true });
        dispatch({ type: 'SET_ORIGINAL_QUOTE_ID', payload: savedQuote.id });
        return true;
      } else {
        // For updating an existing quote: create a new quote with a new ID and parent reference
        isUpdate = true;
        // If the current quote has a parentId, use it (unless it's 0 or '0'); otherwise, use the current quote's id
        let rawParentId = state.currentQuote.parentId;
        if (rawParentId === '0') {
          rawParentId = undefined;
        }
        parentId = rawParentId ? rawParentId : state.currentQuote.id;
        newId = generateQuoteId();
        quoteToSave = {
          ...state.currentQuote,
          id: newId,
          parentId: parentId,
          version: 1,
          createdAt: new Date().toISOString(),
          updatedAt: new Date().toISOString(),
          supplyExchangeRate: exchangeRate,
          laborExchangeRate: exchangeRate,
          metadata: {
            ...state.currentQuote.metadata,
            version: 1,
            updatedAt: new Date().toISOString(),
          }
        };
      }

      // Save the new quote (as a new entry)
      const savedQuote = await apiService.saveQuote(quoteToSave);

      // Create new supply items
      if (state.currentQuote.supplyItems.length > 0) {
        const supplyItemPromises = state.currentQuote.supplyItems.map(item => {
          const newItem = {
            ...item,
            id: undefined,
            quote_id: newId
          };
          return apiService.createSupplyItem(newId, newItem);
        });
        await Promise.all(supplyItemPromises);
      }

      // Create new labor items
      if (state.currentQuote.laborItems.length > 0) {
        const laborItemPromises = state.currentQuote.laborItems.map(item => {
          const newItem = {
            ...item,
            id: undefined,
            quote_id: newId
          };
          return apiService.createLaborItem(newId, newItem);
        });
        await Promise.all(laborItemPromises);
      }

      // Fetch the complete updated quote with new items
      const completeQuoteResponse = await apiService.getQuoteById(newId);
      if (!completeQuoteResponse) {
        throw new Error('Failed to fetch updated quote');
      }
      dispatch({ type: 'SET_QUOTE', payload: completeQuoteResponse });
      dispatch({ type: 'SET_EXISTING_QUOTE', payload: true });
      dispatch({ type: 'SET_ORIGINAL_QUOTE_ID', payload: newId });
      return true;
    } catch (error) {
      dispatch({ type: 'SET_ERROR', payload: error instanceof Error ? error.message : 'Failed to save quote' });
      return false;
    } finally {
      dispatch({ type: 'SET_LOADING', payload: false });
    }
  };

  // Remove the updateQuote function since it's now consolidated with saveQuote
  const updateQuote = async (): Promise<boolean> => {
    return saveQuote();
  };

  // Update a field in the quote
  const setQuoteField = <K extends keyof Quote>(field: K, value: Quote[K]) => {
    dispatch({
      type: 'UPDATE_QUOTE_FIELD',
      payload: { field, value }
    });

    // Recalculate totals if rates are changed
    if (
      field === 'supplyExchangeRate' ||
      field === 'supplyMarginRate' ||
      field === 'laborExchangeRate' ||
      field === 'laborMarginRate'
    ) {
      dispatch({ type: 'RECALCULATE_TOTALS' });
    }
  };

  // Add a supply item
  const addSupplyItem = (item: Omit<SupplyItem, 'id'>) => {
    dispatch({ type: 'ADD_SUPPLY_ITEM', payload: item });
  };

  // Update a supply item
  const updateSupplyItem = (item: SupplyItem) => {
    dispatch({ type: 'UPDATE_SUPPLY_ITEM', payload: item });
  };

  // Remove a supply item
  const removeSupplyItem = (id: string) => {
    dispatch({ type: 'REMOVE_SUPPLY_ITEM', payload: id });
  };

  // Add a labor item
  const addLaborItem = (item: Omit<LaborItem, 'id'>) => {
    dispatch({ type: 'ADD_LABOR_ITEM', payload: item });
  };

  // Update a labor item
  const updateLaborItem = (item: LaborItem) => {
    dispatch({ type: 'UPDATE_LABOR_ITEM', payload: item });
  };

  // Remove a labor item
  const removeLaborItem = (id: string) => {
    dispatch({ type: 'REMOVE_LABOR_ITEM', payload: id });
  };

  // Recalculate all totals
  const recalculateTotals = () => {
    dispatch({ type: 'RECALCULATE_TOTALS' });
  };

  const clearQuote = () => {
    dispatch({ type: 'CLEAR_QUOTE' });
  };

  const value = {
    state,
    createNewQuote,
    loadQuote,
    saveQuote,
    updateQuote,
    setQuoteField,
    addSupplyItem,
    updateSupplyItem,
    removeSupplyItem,
    addLaborItem,
    updateLaborItem,
    removeLaborItem,
    recalculateTotals,
    clearQuote
  };

  return (
    <QuoteContext.Provider value={value}>
      {children}
    </QuoteContext.Provider>
  );
};

// Custom hook for using the quote context
export const useQuote = (): QuoteContextProps => {
  const context = useContext(QuoteContext);

  if (context === undefined) {
    throw new Error('useQuote must be used within a QuoteProvider');
  }

  return context;
};

============================================================
FILE: src/data/sample-items.json
============================================================
[
  {
    "description": "Compresseur C1",
    "priceEuro": 1500.00
  },
  {
    "description": "Ventilateur V2000",
    "priceEuro": 350.75
  },
  {
    "description": "Filtre à air F100",
    "priceEuro": 45.50
  },
  {
    "description": "Condensateur 20kW",
    "priceEuro": 780.25
  },
  {
    "description": "Tuyau flexible 2m",
    "priceEuro": 25.99
  },
  {
    "description": "Kit de montage mural",
    "priceEuro": 120.00
  },
  {
    "description": "Thermostat digital",
    "priceEuro": 89.95
  },
  {
    "description": "Pompe à chaleur 5kW",
    "priceEuro": 1200.50
  },
  {
    "description": "Fluide frigorigène R410A (kg)",
    "priceEuro": 65.00
  },
  {
    "description": "Joint étanchéité",
    "priceEuro": 12.75
  },
  {
    "description": "Isolation thermique 10m",
    "priceEuro": 35.50
  },
  {
    "description": "Unité extérieure complète",
    "priceEuro": 2300.00
  },
  {
    "description": "Évaporateur E300",
    "priceEuro": 450.00
  },
  {
    "description": "Câble électrique spécial climatisation (m)",
    "priceEuro": 5.25
  },
  {
    "description": "Support anti-vibration",
    "priceEuro": 28.90
  }
]

============================================================
FILE: src/hooks/useClients.js
============================================================
import { useState, useEffect } from 'react';
import { clientsApi } from '../services/api';

export const useClients = () => {
    const [clients, setClients] = useState([]);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState(null);

    // Fetch all clients
    const fetchClients = async () => {
        setLoading(true);
        try {
            const data = await clientsApi.getAllClients();
            setClients(data);
            setError(null);
        } catch (err) {
            setError(err.message);
        } finally {
            setLoading(false);
        }
    };

    // Create new client
    const createClient = async (clientData) => {
        setLoading(true);
        try {
            const newClient = await clientsApi.createClient(clientData);
            setClients(prevClients => [...prevClients, newClient]);
            setError(null);
            return newClient;
        } catch (err) {
            setError(err.message);
            throw err;
        } finally {
            setLoading(false);
        }
    };

    // Load clients on component mount
    useEffect(() => {
        fetchClients();
    }, []);

    return {
        clients,
        loading,
        error,
        fetchClients,
        createClient
    };
};

============================================================
FILE: src/hooks/useItems.js
============================================================
 


============================================================
FILE: src/pages/ClientsPage/ClientsPage.scss
============================================================
.clients-page {
  padding: 2rem 0;


  .page-header {
    background-color: #1976d2;
    color: white;

    .header-title {
      font-weight: 500;
    }
  }

  .content-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1rem;

    .search-box {
      flex: 1;
      max-width: 300px;
      margin-right: 1rem;
    }

    .add-button {
      white-space: nowrap;
    }
  }

  .client-card {
    margin-bottom: 1rem;
    border: 1px solid #e0e0e0;
    border-radius: 4px;

    .client-header {
      display: flex;
      align-items: center;
      padding: 1rem;
      background-color: #f5f5f5;

      .client-name {
        flex: 1;
        display: flex;
        align-items: center;
        gap: 0.5rem;
        font-size: 1rem;
        color: #333;

        .MuiIconButton-root {
          padding: 4px;
          margin-right: 4px;
          color: #666;
        }

        .client-id {
          color: #666;
          font-size: 0.875rem;
        }
      }

      .actions {
        display: flex;
        gap: 0.5rem;

        .action-button {
          padding: 4px;

          &.edit {
            color: #1976d2;
          }

          &.delete {
            color: #d32f2f;
          }
        }
      }
    }

    .sites-section {
      padding: 1rem;

      .sites-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 1rem;

        .sites-title {
          display: flex;
          align-items: center;
          gap: 0.5rem;
          font-size: 0.875rem;
          color: #666;
        }
      }

      .sites-list {
        .site-item {
          display: flex;
          align-items: center;
          padding: 0.5rem;
          border-radius: 4px;
          margin-bottom: 0.5rem;

          .site-name {
            flex: 1;
            display: flex;
            align-items: center;
            gap: 0.5rem;

            .location-icon {
              color: #1976d2;
              font-size: 1.25rem;
            }
          }

          .site-actions {
            display: flex;
            gap: 0.5rem;

            .action-button {
              padding: 4px;

              &.edit {
                color: #1976d2;
              }

              &.delete {
                color: #d32f2f;
              }
            }
          }
        }
      }
    }
  }
}

// Styling for the client dialog
.sites-input-section {
  .sites-divider {
    margin: 0.5rem 0 1rem;
  }

  .sites-list {
    max-height: 200px;
    overflow-y: auto;
    border: 1px solid #e0e0e0;
    border-radius: 4px;
    padding: 0.5rem 0;
    background-color: #f9f9f9;

    .MuiListItem-root {
      border-bottom: 1px solid #f0f0f0;

      &:last-child {
        border-bottom: none;
      }

      .MuiListItemText-primary {
        font-size: 0.9rem;
      }
    }

    .no-sites {
      padding: 1rem;
      text-align: center;
    }
  }

  .add-site-form {
    margin-top: 1rem;

    .MuiButton-root {
      min-width: 40px;
      padding: 5px;
    }
  }
}

@media print {
  .clients-page-container {
    display: none;
  }
}

============================================================
FILE: src/pages/ClientsPage/ClientsPage.tsx
============================================================
import React, { useState, useEffect } from 'react';

import logo from '../../logo.png';
import {
  Box,
  Button,
  Container,
  Dialog,
  DialogActions,
  DialogContent,
  DialogTitle,
  IconButton,
  Paper,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  TextField,
  Typography,
  Accordion,
  AccordionSummary,
  AccordionDetails,
  Divider,
  List,
  ListItem,
  ListItemText,
  ListItemSecondaryAction,
  Chip,
  Card,
  CardContent,
  InputBase,
  Collapse,
  Snackbar,
  Alert,
  Menu,
  MenuItem,
  ListItemIcon
} from '@mui/material';
import {
  Add as AddIcon,
  Delete as DeleteIcon,
  Edit as EditIcon,
  ExpandMore as ExpandMoreIcon,
  Place as PlaceIcon,
  KeyboardArrowDown as KeyboardArrowDownIcon,
  KeyboardArrowUp as KeyboardArrowUpIcon,
  MoreVert as MoreVertIcon
} from '@mui/icons-material';
import Layout from '../../components/Layout/Layout';
import { Client, Site, Split } from '../../models/Quote'; // <-- Import Split here
import { apiService } from '../../services/api-service';
import { generateClientId } from '../../utils/id-generator';
import CustomNumberInput from '../../components/CustomNumberInput/CustomNumberInput';
import './ClientsPage.scss';

const API_BASE_URL = process.env.REACT_APP_API_URL;

interface ClientsPageProps {
  currentPath: string;
  onNavigate: (path: string) => void;
}


const ClientsPage: React.FC<ClientsPageProps> = ({ currentPath, onNavigate }) => {
  // State for clients data
  const [clients, setClients] = useState<Client[]>([]);
  const [filteredClients, setFilteredClients] = useState<Client[]>([]);
  const [searchTerm, setSearchTerm] = useState('');
  const [loading, setLoading] = useState(false);

  // State for dialog
  const [dialogOpen, setDialogOpen] = useState(false);
  const [isEditing, setIsEditing] = useState(false);
  const [currentClient, setCurrentClient] = useState<Partial<Client>>({
    name: '',
    sites: [],
    Taux_marge: 0
  });

  // State for new site
  const [newSiteName, setNewSiteName] = useState('');
  const [newSiteAddress, setNewSiteAddress] = useState('');

  // State for snackbar
  const [snackbarOpen, setSnackbarOpen] = useState(false);
  const [snackbarMessage, setSnackbarMessage] = useState('');
  const [snackbarSeverity, setSnackbarSeverity] = useState<'success' | 'error'>('success');

  const [anchorEl, setAnchorEl] = useState<null | HTMLElement>(null);
  const [selectedClient, setSelectedClient] = useState<Client | null>(null);

  const [expandedClient, setExpandedClient] = useState<string | null>(null);

  const [originalClientSites, setOriginalClientSites] = useState<Site[]>([]); // Store original sites when editing
  const [deletedSplits, setDeletedSplits] = useState<string[]>([]); // Track split codes to delete

  // Load clients on component mount
  useEffect(() => {
    loadClients();
  }, []);

  // Load all clients with their sites
  const loadClients = async () => {
    try {
      setLoading(true);
      const response = await fetch(`${API_BASE_URL}/clients`);
      if (!response.ok) {
        throw new Error('Failed to fetch clients');
      }
      const clientsData = await response.json();

      // Fetch sites for each client
      const clientsWithSites = await Promise.all(
        clientsData.map(async (client: { id: string; name: string }) => {
          const sitesResponse = await fetch(`${API_BASE_URL}/sites/by-client?clientId=${client.id}`);
          const sites = sitesResponse.ok ? await sitesResponse.json() : [];

          // Fetch splits for each site
          const sitesWithSplits = await Promise.all(
            sites.map(async (site: Site) => {
              const splitsResponse = await fetch(`${API_BASE_URL}/splits/by-site/${site.id}`);
              const splits = splitsResponse.ok ? await splitsResponse.json() : [];
              return { ...site, splits };
            })
          );

          return { ...client, sites: sitesWithSplits };
        })
      );

      setClients(clientsWithSites);
      setFilteredClients(clientsWithSites);
    } catch (error) {
      console.error('Error loading clients:', error);
      showSnackbar('Erreur lors du chargement des clients', 'error');
    } finally {
      setLoading(false);
    }
  };

  // Open dialog to add a new client
  const handleAddClient = () => {
    setCurrentClient({
      name: '',
      sites: [],
      Taux_marge: 0
    });
    setNewSiteName('');
    setNewSiteAddress('');
    setIsEditing(false);
    setDialogOpen(true);
  };

  // Add a new site (either locally for update, or trigger create for new client)
  const handleAddSite = async () => {
    if (!newSiteName.trim()) {
      showSnackbar('Le nom du site est requis', 'error');
      return;
    }

    const tempSite: Site = {
      id: `temp-${Date.now()}`, // Temporary ID for local state
      name: newSiteName.trim(),
      client_id: currentClient.id || '', // Ensure client_id is present
      splits: [] // <-- Initialize splits
    };

    setCurrentClient(prev => ({
      ...prev,
      sites: [...(prev.sites || []), tempSite]
    }));
    setNewSiteName(''); // Clear input
  };

  // Remove a site (locally during editing)
  const handleRemoveSite = (siteIdToRemove: string) => {
    if (!isEditing && !currentClient.id) {
        // If creating a new client, just remove locally
        setCurrentClient(prev => ({
            ...prev,
            sites: (prev.sites || []).filter(site => site.id !== siteIdToRemove)
        }));
        return;
    }

    // If editing an existing client, also remove locally
    setCurrentClient(prev => ({
      ...prev,
      sites: (prev.sites || []).filter(site => site.id !== siteIdToRemove)
    }));
    // Note: The actual deletion API call happens in handleUpdateClientAndSites
  };

  // Save client (either create new or update existing)
  const handleSave = () => {
    if (isEditing) {
      handleUpdateClientAndSites();
    } else {
      handleCreateClientAndSite(); // Renamed the original save function
    }
  };

  // Renamed function for creating a NEW client and its first site
  const handleCreateClientAndSite = async () => {
    if (!currentClient.name?.trim()) {
      showSnackbar('Le nom du client est requis', 'error');
      return;
    }

    if (!newSiteName.trim()) {
      showSnackbar('Le nom du site est requis', 'error');
      return;
    }

    try {
      setLoading(true);

      // 1. Create client first
      const clientResponse = await fetch(`${API_BASE_URL}/clients`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          name: currentClient.name.trim(),
          Taux_marge: currentClient.Taux_marge || 0
        })
      });

      if (!clientResponse.ok) {
        throw new Error('Failed to create client');
      }

      const newClient = await clientResponse.json();

      // 2. Create the site with the new client's ID
      const siteResponse = await fetch(`${API_BASE_URL}/sites`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          name: newSiteName.trim(),
          client_id: newClient.id
        })
      });

      if (!siteResponse.ok) {
        await fetch(`${API_BASE_URL}/clients/${newClient.id}`, { method: 'DELETE' });
        throw new Error('Failed to create site');
      }

      // After site creation:
      const newSite = await siteResponse.json();

      // Create splits for this site
      const splits = (currentClient.sites?.[0]?.splits ?? []);
      await Promise.all(splits.map(split =>
        fetch(`${API_BASE_URL}/splits`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            code: split.Code,
            name: split.name,
            description: split.description,
            puissance: split.puissance,
            site_id: newSite.id // <-- use site_id, not site
          })
        })
      ));

      showSnackbar('Client et site créés avec succès', 'success');
      handleCloseDialog();
      await loadClients();
    } catch (error) {
      console.error('Error creating client and site:', error);
      showSnackbar('Erreur lors de la création du client et du site', 'error');
    } finally {
      setLoading(false);
    }
  };

  // Function to handle updating an existing client and its sites
  const handleUpdateClientAndSites = async () => {
    if (!currentClient || !currentClient.id) {
      console.error('Update error: No current client ID');
      showSnackbar('Erreur: Client non identifiable pour la mise à jour', 'error');
      return;
    }

    if (!currentClient.name?.trim()) {
      showSnackbar('Le nom du client est requis', 'error');
      return;
    }

    setLoading(true);
    try {
      // 1. Update Client Name
      const clientUpdateResponse = await fetch(`${API_BASE_URL}/clients/${currentClient.id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          name: currentClient.name.trim(),
          Taux_marge: currentClient.Taux_marge || 0
        })
      });

      if (!clientUpdateResponse.ok) {
        throw new Error('Failed to update client name');
      }

      // 2. Manage Sites (Add new, Delete removed)
      const currentSites = currentClient.sites || [];
      const originalSites = originalClientSites;

      const sitesToAdd = currentSites.filter(cs => !originalSites.some(os => os.id === cs.id));
      const sitesToDelete = originalSites.filter(os => !currentSites.some(cs => cs.id === os.id));
      const sitesToUpdate = currentSites.filter(cs => originalSites.some(os => os.id === cs.id));

      // Add new sites
      const addedSitesResponses = await Promise.all(sitesToAdd.map(site =>
        fetch(`${API_BASE_URL}/sites`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ name: site.name, client_id: currentClient.id })
        })
      ));

      // Get new site IDs for splits
      const addedSites = await Promise.all(addedSitesResponses.map(async (res, idx) => {
        if (!res.ok) throw new Error('Failed to add site');
        return await res.json();
      }));

      // Create splits for newly added sites
      await Promise.all(addedSites.map((site, idx) =>
        Promise.all((sitesToAdd[idx].splits ?? []).map(split =>
          fetch(`${API_BASE_URL}/splits`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
              code: split.Code,
              name: split.name,
              description: split.description,
              puissance: split.puissance,
              site_id: site.id
            })
          })
        ))
      ));

      // Always use POST for splits of updated sites
      await Promise.all(sitesToUpdate.map(site =>
        Promise.all((site.splits ?? []).map(split =>
          fetch(`${API_BASE_URL}/splits`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
              code: split.Code,
              name: split.name,
              description: split.description,
              puissance: split.puissance,
              site_id: site.id
            })
          })
        ))
      ));

      // Delete removed sites
      await Promise.all(sitesToDelete.map(site =>
        fetch(`${API_BASE_URL}/sites/${site.id}`, {
          method: 'DELETE'
        })
      ));

      // Delete removed splits
      await Promise.all(deletedSplits.map(code =>
        fetch(`${API_BASE_URL}/splits/${code}`, {
          method: 'DELETE'
        })
      ));

      showSnackbar('Client mis à jour avec succès', 'success');
      handleCloseDialog();
      await loadClients();
      setDeletedSplits([]); // Reset after update

    } catch (error) {
      console.error('Error updating client and sites:', error);
      showSnackbar('Erreur lors de la mise à jour du client', 'error');
    } finally {
      setLoading(false);
    }
  };

  // Close dialog and reset form
  const handleCloseDialog = () => {
    setDialogOpen(false);
    setCurrentClient({
      name: '',
      sites: [],
      Taux_marge: 0
    });
    setNewSiteName('');
    setIsEditing(false);
  };

  // Show snackbar with message
  const showSnackbar = (message: string, severity: 'success' | 'error') => {
    setSnackbarMessage(message);
    setSnackbarSeverity(severity);
    setSnackbarOpen(true);
  };

  // Close snackbar
  const handleCloseSnackbar = () => {
    setSnackbarOpen(false);
  };

  const handleMenuClick = (event: React.MouseEvent<HTMLElement>, client: Client) => {
    setAnchorEl(event.currentTarget);
    setSelectedClient(client);
  };

  const handleMenuClose = () => {
    setAnchorEl(null);
    setSelectedClient(null);
  };

  const handleEditClient = (client: Client) => {
    setCurrentClient(client); // Load current data
    setOriginalClientSites(client.sites || []); // Store original sites for comparison
    setIsEditing(true);
    setNewSiteName(''); // Clear new site name field
    setDialogOpen(true);
    handleMenuClose();
  };

  const handleDeleteClient = async (client: Client) => {
    try {
      setLoading(true);
      await fetch(`${API_BASE_URL}/clients/${client.id}`, {
        method: 'DELETE'
      });
      showSnackbar('Client supprimé avec succès', 'success');
      await loadClients();
    } catch (error) {
      console.error('Error deleting client:', error);
      showSnackbar('Erreur lors de la suppression du client', 'error');
    } finally {
      setLoading(false);
      handleMenuClose();
    }
  };

  return (
    <Layout currentPath={currentPath} onNavigate={onNavigate}>
        <Box sx={{ display: 'flex', position: 'relative', width: '100%',height: '80px', backgroundColor: '#1976d2' , color: 'white'}} className="page-header">
        <Box sx={{ position: 'absolute', left: 0 , display: 'flex', alignItems: 'center',gap: 25 }}>
          <img
            src={logo}
            alt="Logo"
            style={{ height: '60px' }}
          />
          <Typography variant="h6" className="header-title">
            Clients et Sites
          </Typography>
        </Box>
         </Box>

      <Container maxWidth="lg" sx={{ mb: 4 }}>
        <Paper sx={{ p: 3, borderRadius: 2 }}>
          <Box sx={{
            display: 'flex',
            justifyContent: 'space-between',
            alignItems: 'center',
            mb: 3
          }}>
            <Typography variant="h6">
              CLIENTS ET SITES
            </Typography>
            <Box sx={{ display: 'flex', gap: 2 }}>
              <TextField
                placeholder="Rechercher un client"
                size="small"
                sx={{
                  width: 250,
                  '& .MuiOutlinedInput-root': {
                    borderRadius: 1
                  }
                }}
                value={searchTerm}
                onChange={(e) => setSearchTerm(e.target.value)}
              />
              <Button
                variant="contained"
                startIcon={<AddIcon />}
                onClick={handleAddClient}
                disabled={loading}
                sx={{
                  bgcolor: '#1976d2',
                  '&:hover': { bgcolor: '#1565c0' }
                }}
              >
                Nouveau Client
              </Button>
            </Box>
          </Box>

          <Box sx={{ mt: 2 }}>
            {loading ? (
              <Box sx={{ textAlign: 'center', py: 3 }}>
                Chargement...
              </Box>
            ) : filteredClients.length === 0 ? (
              <Box sx={{ textAlign: 'center', py: 3 }}>
                Aucun client trouvé
              </Box>
            ) : (
              filteredClients.map((client) => (
                <Paper
                  key={client.id}
                  elevation={1}
                  sx={{
                    mb: 2,
                    overflow: 'hidden',
                    border: '1px solid #e0e0e0'
                  }}
                >
                  <Box sx={{
                    display: 'flex',
                    justifyContent: 'space-between',
                    alignItems: 'center',
                    p: 2,
                    '&:hover': { bgcolor: '#f5f5f5' }
                  }}>
                    <Box>
                      <Typography>
                        {client.name} <Typography component="span" color="text.secondary">#{client.id}</Typography>
                      </Typography>
                    </Box>
                    <Box sx={{ display: 'flex', gap: 1 }}>
                      <IconButton
                        size="small"
                        onClick={() => setExpandedClient(expandedClient === client.id ? null : client.id)}
                      >
                        {expandedClient === client.id ? <KeyboardArrowUpIcon /> : <KeyboardArrowDownIcon />}
                      </IconButton>
                      <IconButton
                        size="small"
                        onClick={(e) => handleMenuClick(e, client)}
                      >
                        <MoreVertIcon />
                      </IconButton>
                    </Box>
                  </Box>
                  <Collapse in={expandedClient === client.id}>
                    <Box sx={{ px: 2, pb: 2 }}>
                      <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 2 }}>
                        <Typography variant="subtitle2" color="text.secondary">
                          Sites ({client.sites?.length || 0})
                        </Typography>
                      </Box>
                      {client.sites?.map((site) => (
                        <React.Fragment key={site.id}>
                          <Box
                            sx={{
                              display: 'flex',
                              justifyContent: 'space-between',
                              alignItems: 'center',
                              p: 1,
                              borderRadius: 1,
                              '&:hover': { bgcolor: '#f5f5f5' }
                            }}
                          >
                            <Box sx={{ display: 'flex', alignItems: 'center', gap: 1 }}>
                              <PlaceIcon fontSize="small" color="action" />
                              <Typography>{site.name}</Typography>
                            </Box>
                          </Box>
                          {/* Splits rendering must be inside the map block */}
                          {site.splits && site.splits.length > 0 && (
                            <Box sx={{ pl: 5, pb: 1 }}>
                              <Typography variant="body2" color="text.secondary" sx={{ fontWeight: 500 }}>
                                Splits:
                              </Typography>
                              <List dense>
                                {site.splits.map((split, splitIdx) => (
                                  <ListItem key={splitIdx} sx={{ pl: 0 }}>
                                    <ListItemText
                                      primary={`${split.Code || '-'} : ${split.name || '-'} ${split.puissance ?? '-'} btu/Kw`}
                                      secondary={split.description}
                                    />
                                  </ListItem>
                                ))}
                              </List>
                            </Box>
                          )}
                        </React.Fragment>
                      ))}
                    </Box>
                  </Collapse>
                </Paper>
              ))
           ) }
          </Box>
        </Paper>
      </Container>

      <Menu
        anchorEl={anchorEl}
        open={Boolean(anchorEl)}
        onClose={handleMenuClose}
      >
        {selectedClient?.sites?.map((site) => (
          <MenuItem key={site.id} disabled>
            <ListItemText primary={site.name} />
          </MenuItem>
        ))}
        <Divider />
        <MenuItem onClick={() => selectedClient && handleEditClient(selectedClient)}>
          <ListItemIcon>
            <EditIcon fontSize="small" />
          </ListItemIcon>
          <ListItemText>Modifier</ListItemText>
        </MenuItem>
        <MenuItem onClick={() => selectedClient && handleDeleteClient(selectedClient)}>
          <ListItemIcon>
            <DeleteIcon fontSize="small" />
          </ListItemIcon>
          <ListItemText>Supprimer</ListItemText>
        </MenuItem>
      </Menu>

      {/* Add/Edit Client Dialog */}
      <Dialog open={dialogOpen} onClose={handleCloseDialog} maxWidth="sm" fullWidth>
        <DialogTitle>
          {isEditing ? 'Modifier le client' : 'Nouveau Client'}
        </DialogTitle>
        <DialogContent>
          <TextField
            autoFocus
            margin="dense"
            label="Nom du client"
            type="text"
            fullWidth
            value={currentClient.name}
            onChange={(e) => setCurrentClient({ ...currentClient, name: e.target.value })}
          />
          <CustomNumberInput
            label="Taux de Marge"
            value={currentClient.Taux_marge || 0}
            onChange={(value) => setCurrentClient({ ...currentClient, Taux_marge: value })}
            step={0.01}
            min={0}
            fullWidth
            margin="dense"
            variant="outlined"
          />
          <Box sx={{ display: 'flex', gap: 2, mb: 2 }}>
            <TextField
              label="Nom du site"
              size="small"
              fullWidth
              value={newSiteName}
              onChange={(e) => setNewSiteName(e.target.value)}
            />
            <Button
              variant="outlined"
              onClick={handleAddSite} // Calls the updated function
              disabled={!newSiteName.trim()}
            >
              Ajouter
            </Button>
          </Box>
          <List>
            {currentClient.sites?.map((site, siteIdx) => (
              <Box key={site.id} sx={{ mb: 2, border: '1px solid #eee', borderRadius: 1, p: 1 }}>
                <ListItem
                  secondaryAction={
                    <IconButton edge="end" onClick={() => handleRemoveSite(site.id)}>
                      <DeleteIcon />
                    </IconButton>
                  }
                >
                  <ListItemText primary={site.name} />
                </ListItem>
                {/* Splits for this site */}
                <Typography variant="subtitle2" sx={{ mt: 1 }}>Splits</Typography>
                {(site.splits ?? []).map((split, splitIdx) => (
                  <Box key={splitIdx} sx={{ display: 'flex', gap: 1, alignItems: 'center', mb: 1 }}>
                    <TextField
                      label="Code split"
                      size="small"
                      value={split.Code}
                      onChange={e => {
                        const newSites = (currentClient.sites || []).map((s, idx) =>
                          idx === siteIdx
                            ? {
                                ...s,
                                splits: (s.splits ?? []).map((sp, spIdx) =>
                                  spIdx === splitIdx ? { ...sp, Code: e.target.value } : sp
                                )
                              }
                            : s
                        );
                        setCurrentClient({ ...currentClient, sites: newSites });
                      }}
                    />
                    <TextField
                      label="Nom du split"
                      size="small"
                      value={split.name}
                      onChange={e => {
                        const newSites = (currentClient.sites || []).map((s, idx) =>
                          idx === siteIdx
                            ? {
                                ...s,
                                splits: (s.splits ?? []).map((sp, spIdx) =>
                                  spIdx === splitIdx ? { ...sp, name: e.target.value } : sp
                                )
                              }
                            : s
                        );
                        setCurrentClient({ ...currentClient, sites: newSites });
                      }}
                    />
                    <TextField
                      label="Puissance"
                      size="small"
                      type="number"
                      value={split.puissance}
                      onChange={e => {
                        const newSites = (currentClient.sites || []).map((s, idx) =>
                          idx === siteIdx
                            ? {
                                ...s,
                                splits: (s.splits ?? []).map((sp, spIdx) =>
                                  spIdx === splitIdx ? { ...sp, puissance: Number(e.target.value) } : sp
                                )
                              }
                            : s
                        );
                        setCurrentClient({ ...currentClient, sites: newSites });
                      }}
                    />
                    <IconButton
                      size="small"
                      onClick={() => {
                        // If split has a Code, mark for deletion
                        if (split.Code) {
                          setDeletedSplits(prev => [...prev, split.Code]);
                        }
                        const newSites = (currentClient.sites || []).map((s, idx) =>
                          idx === siteIdx
                            ? {
                                ...s,
                                splits: (s.splits ?? []).filter((_, spIdx) => spIdx !== splitIdx)
                              }
                            : s
                        );
                        setCurrentClient({ ...currentClient, sites: newSites });
                      }}
                    >
                      <DeleteIcon fontSize="small" />
                    </IconButton>
                  </Box>
                ))}
                {/* Add Split Button */}
                <Button
                  size="small"
                  variant="outlined"
                  sx={{ mt: 1 }}
                  onClick={() => {
                    const newSplit: Split = {
                      Code: '',           // or generate a code if needed
                      name: '',
                      description: '',
                      puissance: 0,
                      site: site.id       // associate with the current site
                    };
                    const newSites = (currentClient.sites || []).map((s, idx) =>
                      idx === siteIdx
                        ? { ...s, splits: [...(s.splits ?? []), newSplit] }
                        : s
                    );
                    setCurrentClient({ ...currentClient, sites: newSites });
                  }}
                >
                  Ajouter un split
                </Button>
              </Box>
            ))}
          </List>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleCloseDialog}>Annuler</Button>
          <Button
            onClick={handleSave}
            variant="contained"
            disabled={loading || !currentClient.name?.trim()}
          >
            {loading ? 'Enregistrement...' : (isEditing ? 'Mettre à jour' : 'Enregistrer')}
          </Button>
        </DialogActions>
      </Dialog>

      {/* Snackbar for notifications */}
      <Snackbar
        open={snackbarOpen}
        autoHideDuration={6000}
        onClose={handleCloseSnackbar}
        anchorOrigin={{ vertical: 'bottom', horizontal: 'center' }}
      >
        <Alert
          onClose={handleCloseSnackbar}
          severity={snackbarSeverity}
          sx={{ width: '100%' }}
        >
          {snackbarMessage}
        </Alert>
      </Snackbar>
    </Layout>
  );
};

export default ClientsPage;


============================================================
FILE: src/pages/HistoryPage/HistoryPage.scss
============================================================
.history-page {
  padding: 2rem 0;

  .page-header {
    background-color: #1976d2;
    color: white;

    .header-title {
      font-weight: 500;
    }
  }

  .filters-section {
    padding: 1.5rem;
    margin: 1rem 0 2rem;

    .filters-grid {
      display: flex;
      flex-wrap: wrap;
      gap: 1rem;
      margin-top: 1rem;

      > * {
        flex: 1 1 200px;
        min-width: 200px;
      }
    }
  }

  .quote-card {
    margin-bottom: 1rem;
    border: 1px solid #e0e0e0;
    border-radius: 4px;
    overflow: hidden;

    .card-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 1rem;
      background-color: #f5f5f5;

      .quote-id {
        display: flex;
        align-items: center;
        gap: 0.5rem;

        .id-number {
          font-weight: 500;
          margin-right: 8px;
        }

        .version-label {
          padding: 2px 8px;
          border-radius: 12px;
          font-size: 0.75rem;
          margin-left: 8px;
          font-weight: 500;

          &.original {
            background-color: #e8f5e9;
            color: #2e7d32;
          }

          &.update {
            background-color: #fff3e0;
            color: #f57c00;
          }
        }
      }
    }

    .card-content {
      padding: 1rem;

      .info-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 1rem;

        .info-item {
          .label {
            color: #666;
            font-size: 0.875rem;
            margin-bottom: 0.25rem;
          }

          .value {
            font-size: 1rem;
            color: #333;

            &.total {
              font-weight: 600;
              color: #1976d2;
            }
          }
        }
      }
    }

    .card-actions {
      display: flex;
      align-items: center;
      padding: 8px 16px;

      .action-button {
        &.view {
          color: #1976d2;
        }

        &.delete {
          color: #d32f2f;
        }
      }

      .MuiButton-root {
        text-transform: none;
        min-width: 100px;

        &.Mui-disabled {
          background-color: #a5d6a7;
          color: white;
        }
      }
    }
  }

  .version-info {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    margin-top: 1rem;
    padding: 0.5rem;
    background-color: #e3f2fd;
    border-radius: 4px;

    .info-icon {
      color: #1976d2;
    }

    .info-text {
      font-size: 0.875rem;
      color: #1976d2;
    }
  }

  .reminder-section {
    display: flex;
    align-items: center;
    gap: 16px;
    margin-top: 16px;
    padding-top: 16px;
    border-top: 1px solid rgba(0, 0, 0, 0.12);

    .custom-number-input {
      .MuiTextField-root {
        width: 150px;
      }
    }

    .MuiIconButton-root {
      margin-left: 8px;

      &.Mui-disabled {
        opacity: 0.5;
      }

      &:hover {
        background-color: rgba(25, 118, 210, 0.04);
      }
    }
  }
}

.version-badge {
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 0.75rem;
  font-weight: 500;

  &.original {
    background-color: #e8f5e9;
    color: #2e7d32;
  }

  &.update {
    background-color: #fff3e0;
    color: #f57c00;
  }
}

// Responsive adjustments
@media (max-width: 600px) {
  .history-container {
    .filters-section {
      padding: 1rem;
    }
  }
}

============================================================
FILE: src/pages/HistoryPage/HistoryPage.tsx
============================================================
import React, { useState, useEffect } from 'react';
import {
  Box,
  Container,
  Typography,
  TextField,
  MenuItem,
  Select,
  FormControl,
  InputLabel,
  Button,
  Paper,
  Card,
  IconButton,
  Divider
} from '@mui/material';
import DeleteIcon from '@mui/icons-material/Delete';
import VisibilityIcon from '@mui/icons-material/Visibility';
import Layout from '../../components/Layout/Layout';
import './HistoryPage.scss';
import { ReceiptLongOutlined } from '@mui/icons-material';
import logo from '../../logo.png';
import AddAlarmIcon from '@mui/icons-material/AddAlarm';
import { Client, Site, Quote } from '../../models/Quote';
import { apiService } from '../../services/api-service';
import { extractBaseId } from '../../utils/id-generator';
import CheckCircleIcon from '@mui/icons-material/CheckCircle';
import CheckCircleOutlineIcon from '@mui/icons-material/CheckCircleOutline';
import CustomNumberInput from '../../components/CustomNumberInput/CustomNumberInput';

interface HistoryPageProps {
  currentPath: string;
  onNavigate: (path: string, quoteId?: string) => void;
}

const HistoryPage: React.FC<HistoryPageProps> = ({ currentPath, onNavigate }) => {
  // Filter state
  const [filters, setFilters] = useState({
    id: '',
    client: '',
    site: '',
    period: 'all',
    startDate: '',
    endDate: '',
  });
  const [clients, setClients] = useState<Client[]>([]);
  const [sites, setSites] = useState<Site[]>([]);
  const [quotes, setQuotes] = useState<Quote[]>([]);
  const [filteredQuotes, setFilteredQuotes] = useState<Quote[]>([]);

  // Group quotes by parentId (originals by their own id, updates by parentId)
  const groupedQuotes = React.useMemo(() => {
    const groups: { [groupId: string]: Quote[] } = {};
    for (const quote of filteredQuotes) {
      // Treat parentId of '', undefined, or '0' as no parent (original)
      const hasValidParent = quote.parentId && quote.parentId !== '' && quote.parentId !== '0';
      let groupId = hasValidParent ? quote.parentId : quote.id;
      if (!groupId) groupId = quote.id; // Ensure groupId is always a string
      if (!groups[groupId]) groups[groupId] = [];
      groups[groupId].push(quote);
    }
    // Sort each group by createdAt (oldest first)
    Object.values(groups).forEach(group => {
      group.sort((a, b) => new Date(a.createdAt).getTime() - new Date(b.createdAt).getTime());
    });
    return groups;
  }, [filteredQuotes]);

  // Local state for reminder days per quote
  const [reminderDays, setReminderDays] = useState<{ [quoteId: string]: number }>({});

  // Fetch clients, sites, and quotes on mount
  useEffect(() => {
    const fetchData = async () => {
      const [allClients, allQuotes] = await Promise.all([
        apiService.getClients ? apiService.getClients() : [],
        apiService.getQuotes(),
      ]);
      // Fetch sites for each client and attach
      const clientsWithSites = await Promise.all(
        (allClients || []).map(async (client) => {
          const sites = await apiService.getSitesByClientId(client.id);
          return { ...client, sites: sites || [] };
        })
      );
      setClients(clientsWithSites);
      setQuotes(allQuotes || []);
    };
    fetchData();
  }, []);

  // Update sites when client changes
  useEffect(() => {
    if (filters.client) {
      const client = clients.find(c => c.id === filters.client);
      setSites(client ? client.sites : []);
      // If the selected site does not belong to the new client, reset it
      if (filters.site && !(client && client.sites.some(s => s.id === filters.site))) {
        setFilters(prev => ({ ...prev, site: '' }));
      }
    } else {
      setSites([]);
      setFilters(prev => ({ ...prev, site: '' }));
    }
    // eslint-disable-next-line
  }, [filters.client, clients]);

  // Helper to parse date safely
  const parseQuoteDate = (dateStr: string | Date): Date => {
    if (dateStr instanceof Date) return dateStr;
    if (/^\d{4}-\d{2}-\d{2}$/.test(dateStr)) {
      return new Date(dateStr + 'T00:00:00');
    }
    return new Date(dateStr);
  };

  // Apply filters to quotes
  useEffect(() => {
    let result = [...quotes];
    if (filters.id) {
      result = result.filter(q => q.id.includes(filters.id));
    }
    if (filters.client) {
      result = result.filter(q => q.clientName === (clients.find(c => c.id === filters.client)?.name));
    }
    if (filters.site) {
      result = result.filter(q => q.siteName === (sites.find(s => s.id === filters.site)?.name));
    }
    // Date filtering logic for period
    if (filters.period !== 'all') {
      const today = new Date();
      today.setHours(0, 0, 0, 0);
      if (filters.period === 'today') {
        result = result.filter(q => {
          const quoteDate = parseQuoteDate(q.date);
          quoteDate.setHours(0, 0, 0, 0);
          return quoteDate.getTime() === today.getTime();
        });
      } else if (filters.period === 'week') {
        const startOfWeek = new Date(today);
        startOfWeek.setDate(today.getDate() - today.getDay());
        result = result.filter(q => {
          const quoteDate = parseQuoteDate(q.date);
          return quoteDate >= startOfWeek && quoteDate <= today;
        });
      } else if (filters.period === 'month') {
        const startOfMonth = new Date(today.getFullYear(), today.getMonth(), 1);
        result = result.filter(q => {
          const quoteDate = parseQuoteDate(q.date);
          return quoteDate >= startOfMonth && quoteDate <= today;
        });
      } else if (filters.period === 'year') {
        const startOfYear = new Date(today.getFullYear(), 0, 1);
        result = result.filter(q => {
          const quoteDate = parseQuoteDate(q.date);
          return quoteDate >= startOfYear && quoteDate <= today;
        });
      } else if (filters.period === 'custom' && filters.startDate && filters.endDate) {
        const startDate = parseQuoteDate(filters.startDate);
        const endDate = parseQuoteDate(filters.endDate);
        endDate.setHours(23, 59, 59, 999);
        result = result.filter(q => {
          const quoteDate = parseQuoteDate(q.date);
          return quoteDate >= startDate && quoteDate <= endDate;
        });
      }
    }
    setFilteredQuotes(result);
    // eslint-disable-next-line
  }, [filters, quotes, clients, sites]);

  // Handlers
  const handleFilterChange = (field: string, value: string) => {
    setFilters(prev => ({ ...prev, [field]: value }));
  };
  const handleClearFilters = () => {
    setFilters({ id: '', client: '', site: '', period: 'all', startDate: '', endDate: '' });
  };

  // Handlers for quote card actions
  const handleLoadQuote = (quoteId: string) => {
    const quote = quotes.find(q => q.id === quoteId);
    if (quote) {
      if (quote.reminderDate) {
        const today = new Date();
        today.setHours(0, 0, 0, 0);
        const reminder = new Date(quote.reminderDate);
        reminder.setHours(0, 0, 0, 0);
        if (reminder < today) {
          alert("Attention : la date de rappel de ce devis est dépassée !");
        }
      }
      if (quote.confirmed) {
        onNavigate(`/quote?id=${quoteId}`);
      } else {
        onNavigate(`/quote?id=${quoteId}&showConfirm=true`);
      }
    }
  };
  const handleDeleteQuote = async (quoteId: string) => {
    if (window.confirm('Êtes-vous sûr de vouloir supprimer ce devis ?')) {
      // Call backend to delete by id only
      await apiService.deleteQuote(quoteId);
      setQuotes(prev => prev.filter(q => q.id !== quoteId));
    }
  };
  const handleViewPriceOffer = (quoteId: string) => {
    onNavigate('/price-offer', quoteId);
  };
  const handleReminderChange = (quoteId: string, value: number) => {
    setReminderDays(prev => ({ ...prev, [quoteId]: value }));
  };
  const handleSetReminder = async (quoteId: string) => {
    const days = reminderDays[quoteId] || 0;
    if (days < 1) return;
    const currentDate = new Date();
    const reminderDate = new Date(currentDate.setDate(currentDate.getDate() + days));
    const formattedDate = reminderDate.toISOString().split('T')[0];
    // Use fetch directly since fetchApi is private
    await fetch(`${process.env.REACT_APP_API_URL}/quotes/${quoteId}/reminder`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ reminderDate: formattedDate }),
    });
    setQuotes(prev => prev.map(q => q.id === quoteId ? { ...q, reminderDate: formattedDate } : q));
    alert('Rappel configuré avec succès');
  };

  return (
    <Layout currentPath={currentPath} onNavigate={onNavigate}>
      <Container maxWidth="lg" className="history-page">
        <Box sx={{ display: 'flex', position: 'relative', width: '100%', height: '80px', backgroundColor: 'white', color: 'black' }} className="page-header">
          <Box sx={{ position: 'absolute', left: 0, display: 'flex', alignItems: 'center', gap: 40 }}>
          <img
            src={logo}
            alt="Logo"
            style={{ height: '60px' }}
          />
          <Typography variant="h6" className="header-title">
            HISTORIQUE
          </Typography>
        </Box>
         </Box>
        <Paper elevation={3} className="filters-section">
          <Typography variant="h6" gutterBottom>
            Filtres
          </Typography>

          <Box className="filters-grid">
            <TextField
              fullWidth
              label="ID"
              variant="outlined"
              size="small"
              value={filters.id}
              onChange={e => handleFilterChange('id', e.target.value)}
            />

            <FormControl fullWidth size="small">
              <InputLabel>Client</InputLabel>
              <Select
                label="Client"
                value={filters.client}
                onChange={e => handleFilterChange('client', e.target.value as string)}
              >
                <MenuItem value="">Tous les clients</MenuItem>
                {(clients || []).map(client => (
                  <MenuItem key={client.id} value={client.id}>{client.name}</MenuItem>
                ))}
              </Select>
            </FormControl>

            <FormControl fullWidth size="small">
              <InputLabel>Site</InputLabel>
              <Select
                label="Site"
                value={filters.site}
                onChange={e => handleFilterChange('site', e.target.value as string)}
                disabled={!filters.client}
              >
                <MenuItem value="">Tous les sites</MenuItem>
                {(sites || []).map(site => (
                  <MenuItem key={site.id} value={site.id}>{site.name}</MenuItem>
                ))}
              </Select>
            </FormControl>

            <FormControl fullWidth size="small">
              <InputLabel>Période</InputLabel>
              <Select
                label="Période"
                value={filters.period}
                onChange={e => handleFilterChange('period', e.target.value as string)}
              >
                <MenuItem value="all">Toutes les dates</MenuItem>
                <MenuItem value="today">Aujourd'hui</MenuItem>
                <MenuItem value="week">Cette semaine</MenuItem>
                <MenuItem value="month">Ce mois</MenuItem>
                <MenuItem value="year">Cette année</MenuItem>
                <MenuItem value="custom">Période personnalisée</MenuItem>
              </Select>
            </FormControl>

            {/* Show date pickers if custom period is selected */}
            {filters.period === 'custom' && (
              <>
                <TextField
                  label="Date de début"
                  type="date"
                  size="small"
                  value={filters.startDate}
                  onChange={e => handleFilterChange('startDate', e.target.value)}
                  InputLabelProps={{ shrink: true }}
                  sx={{ minWidth: 180, marginRight: 2, marginTop: 1 }}
                />
                <TextField
                  label="Date de fin"
                  type="date"
                  size="small"
                  value={filters.endDate}
                  onChange={e => handleFilterChange('endDate', e.target.value)}
                  InputLabelProps={{ shrink: true }}
                  sx={{ minWidth: 180, marginTop: 1 }}
                />
              </>
            )}
          </Box>

          <Box sx={{ display: 'flex', justifyContent: 'space-between', mt: 2 }}>
            <Button variant="outlined" size="small" onClick={handleClearFilters}>
              Effacer les filtres
            </Button>
          </Box>
        </Paper>

        <Box sx={{ mt: 3 }}>
          {/* Render grouped quotes by base ID */}
          {Object.entries(groupedQuotes).map(([baseId, versions]) => (
            <Box key={baseId} sx={{ mb: 4, border: '2px solid #1976d2', borderRadius: 2, p: 2 }}>
              <Typography variant="h6" color="primary" sx={{ mb: 2 }}>
                Groupe de devis: {baseId}
              </Typography>
              {versions.map((quote, idx) => {
                const isOriginal = idx === 0;
                const isLatest = idx === versions.length - 1;
                return (
                  <Card className="quote-card" key={quote.id} sx={{ mb: 2, background: isLatest ? '#e3f2fd' : undefined }}>
                    <Box className="card-header">
                      <Box className="quote-id">
                        <Typography component="span" className="id-number">
                          {quote.id}
                        </Typography>
                        {/* Check icon for confirmed status */}
                        {quote.confirmed ? (
                          <CheckCircleIcon sx={{ color: '#4caf50', ml: 1 }} titleAccess="Confirmé" />
                        ) : (
                          <CheckCircleOutlineIcon sx={{ color: '#bdbdbd', ml: 1 }} titleAccess="Non confirmé" />
                        )}
                        <Typography component="span" className={`version-label ${isOriginal ? 'original' : 'update'}`}
                          sx={{ ml: 1 }}>
                          {isOriginal ? 'Version originale' : `Version ${idx}`}
                        </Typography>
                      </Box>
                      <Typography variant="caption" color="text.secondary">
                        {new Date(quote.createdAt).toLocaleDateString('fr-FR')}
                      </Typography>
                    </Box>
                    <Divider />
                    <Box className="card-content">
                      <Box className="info-grid">
                        <Box className="info-item">
                          <Typography className="label">Client</Typography>
                          <Typography className="value">{quote.clientName || 'Non spécifié'}</Typography>
                        </Box>
                        <Box className="info-item">
                          <Typography className="label">Site</Typography>
                          <Typography className="value">{quote.siteName || 'Non spécifié'}</Typography>
                        </Box>
                        <Box className="info-item">
                          <Typography className="label">Date</Typography>
                          <Typography className="value">{new Date(quote.date).toLocaleDateString('fr-FR')}</Typography>
                        </Box>
                        <Box className="info-item">
                          <Typography className="label">Total TTC</Typography>
                          <Typography className="value total">
                            {new Intl.NumberFormat('fr-FR', { style: 'currency', currency: 'USD' }).format(quote.totalTTC)}
                          </Typography>
                        </Box>
                      </Box>
                      {isLatest && (
                        <Box sx={{ display: 'flex', alignItems: 'center', gap: 2, mt: 2, borderTop: '1px solid rgba(0, 0, 0, 0.12)', paddingTop: 2 }}>
                          <CustomNumberInput
                            label="Jours de rappel"
                            value={reminderDays[quote.id] || 0}
                            onChange={value => handleReminderChange(quote.id, value)}
                            min={0}
                            fullWidth={false}
                          />
                          <IconButton color="primary" sx={{ ml: 1 }} onClick={() => handleSetReminder(quote.id)}>
                            <AddAlarmIcon />
                          </IconButton>
                        </Box>
                      )}
                    </Box>
                    <Box className="card-actions">
                      <Button className="action-button view" size="small" startIcon={<VisibilityIcon />} onClick={() => handleLoadQuote(quote.id)}>
                        Voir
                      </Button>
                      {isLatest && (
                        <>
                          <Button className="action-button" size="small" color="primary" startIcon={<ReceiptLongOutlined />} onClick={() => handleViewPriceOffer(quote.id)}>
                            Voir l'offre de prix
                          </Button>
                          <Button className="action-button delete" size="small" color="error" startIcon={<DeleteIcon />} onClick={() => handleDeleteQuote(quote.id)}>
                            Supprimer
                          </Button>
                        </>
                      )}
                    </Box>
                  </Card>
                );
              })}
            </Box>
          ))}
        </Box>
      </Container>
    </Layout>
  );
};

export default HistoryPage;

============================================================
FILE: src/pages/HomePage/HomePage.scss
============================================================
.home-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f5f5f5;
  padding: 2rem;
  position: relative;
}

.home-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 4rem;
}

.home-logo {
  height: 120px;
  width: auto;
  margin-bottom: 2rem;
}

.home-title {
  color: #1976d2;
  font-weight: 600;
  text-align: center;
}

.home-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 2rem;
  justify-content: center;
  margin-top: 2rem;
}

.button-container {
  flex: 1;
  min-width: 300px;
  max-width: 400px;
}
.logout-button-container {
  position: absolute;
  top: 25px;
  right: 0px;
}
.logout-button {
  font-weight: 500;
  text-transform: none;
  color: #1976d2;
  border: 1px solid rgba(25, 118, 210, 0.3);
  border-radius: 4px;
  background-color: white;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);

  &:hover {
    background-color: #1976d2;
    color: white;
  }

  svg {
    font-size: 1.5rem;
    margin-right: 8px;
  }
}

.home-button {
  height: 120px;
  font-size: 2rem !important;
  font-weight: 600;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s, box-shadow 0.2s;

  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 8px rgba(0, 0, 0, 0.15);
  }

  &:disabled {
    opacity: 0.7;
    cursor: not-allowed;
  }
}

============================================================
FILE: src/pages/HomePage/HomePage.tsx
============================================================
import React from 'react';
import { Box, Container, Button, Typography } from '@mui/material';
import { useNavigate } from 'react-router-dom';
import LogoutIcon from '@mui/icons-material/Logout';
import logo from '../../logo.png';
import './HomePage.scss';

const HomePage: React.FC = () => {
  const navigate = useNavigate();

  const handleNavigate = (path: string) => {
    navigate(path);
  };

  const handleLogout = () => {
    // Add logout logic here
    navigate('/login');
  };

  return (
    <Box className="home-container">
      <Box className="logout-button-container">
        <Button
          color="primary"
          className="logout-button"
          startIcon={<LogoutIcon />}
          onClick={handleLogout}
        >
          Déconnexion
        </Button>
      </Box>
      <Container maxWidth="md">
        <Box className="home-header">
          <img
            src={logo}
            alt="Logo"
            className="home-logo"
          />
          <Typography variant="h4" component="h1" className="home-title">
            Bienvenue sur Chanitec
          </Typography>
        </Box>

        <Box className="home-buttons">
          <Box className="button-container">
            <Button
              variant="contained"
              color="primary"
              size="large"
              fullWidth
              className="home-button"
              onClick={() => handleNavigate('/quote')}
            >
              Calcul de Prix
            </Button>
          </Box>
          <Box className="button-container">
            <Button
              variant="contained"
              color="secondary"
              size="large"
              fullWidth
              className="home-button"
              onClick={() => handleNavigate('/org-chart')}
            >
              Organigramme
            </Button>
          </Box>
          <Box className="button-container">
            <Button
              variant="contained"
              color="primary"
              size="large"
              fullWidth
              className="home-button"
              onClick={() => handleNavigate('/intervention')}
            >
              Fiche d'intervention
            </Button>
          </Box>
        </Box>
      </Container>
    </Box>
  );
};

export default HomePage;

============================================================
FILE: src/pages/ItemsPage/ItemsPage.scss
============================================================
.items-page-container {
  padding-bottom: 2rem;

  .items-paper {
    padding: 1.5rem;
    margin-bottom: 2rem;

    .items-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 1.5rem;
      flex-wrap: wrap;

      @media (max-width: 600px) {
        flex-direction: column;
        align-items: stretch;
      }

      .section-title {
        margin-bottom: 0.5rem;
      }

      .search-and-add {
        display: flex;
        gap: 1rem;
        align-items: center;
        flex-wrap: wrap;

        @media (max-width: 600px) {
          flex-direction: column;
          align-items: stretch;
          width: 100%;
        }

        .search-field {
          min-width: 250px;
        }

        .button-group {
          display: flex;
          gap: 0.5rem;

          @media (max-width: 600px) {
            margin-top: 0.5rem;
            width: 100%;
            justify-content: space-between;
          }
        }
      }
    }

    .items-table-container {
      margin-top: 1rem;

      .MuiTableCell-root {
        padding: 0.75rem;
      }
    }
  }
}

@media print {
  .items-page-container {
    display: none;
  }
}

============================================================
FILE: src/pages/ItemsPage/ItemsPage.tsx
============================================================
import React, { useState, useEffect, useRef, FC } from 'react';
import {
  Box,
  Button,
  Container,
  Dialog,
  DialogActions,
  DialogContent,
  DialogTitle,
  IconButton,
  Paper,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  TextField,
  Typography,
  Tooltip,
  Snackbar,
  Alert
} from '@mui/material';
import {
  Add as AddIcon,
  Delete as DeleteIcon,
  Edit as EditIcon,
  FileUpload as FileUploadIcon,
  WarningAmber as WarningAmberIcon
} from '@mui/icons-material';
import * as XLSX from 'xlsx';
import Layout from '../../components/Layout/Layout';
import { SupplyItem } from '../../models/Quote';
import { itemsApi } from '../../services/api';
import './ItemsPage.scss';
import { v4 as uuidv4 } from 'uuid';
import CustomNumberInput from '../../components/CustomNumberInput/CustomNumberInput';
import logo from '../../logo.png';
const API_BASE_URL = process.env.REACT_APP_API_URL;


interface ItemsPageProps {
  currentPath: string;
  onNavigate: (path: string) => void;
}

interface ExcelItem {
  Description: string;
  price: number;
}

interface ProcessedItem {
  valid: boolean;
  rowIndex: number;
  item?: {
    description: string;
    priceEuro: number;
  };
  error?: string;
  rawData?: any;
}

interface ImportResultItem {
  rowIndex: number;
  item?: any;
  error?: string;
  rawData?: any;
}

interface ImportSummary {
  total: number;
  valid: number;
  invalid: number;
  imported: number;
  errors: number;
}

interface ImportResults {
  totalProcessed: number;
  successful: ImportResultItem[];
  failed: ImportResultItem[];
  summary: ImportSummary;
}

interface ProcessedExcelItem {
  description: string;
  priceEuro: number;
  quantity: number;
  rowIndex: number;
  isValid: boolean;
  validationError?: string;
  originalRow: {
    description: any;
    quantity: any;
    price: any;
  };
}

// Add type for worksheet with index signature
interface ExtendedWorksheet {
  [key: string]: any;
  '!ref'?: string;
}

// Add type for cell address
interface CellAddress {
  r: number;
  c: number;
}

const ItemsPage: FC<ItemsPageProps> = ({ currentPath, onNavigate }) => {
  // State for items data
  const [items, setItems] = useState<SupplyItem[]>([]);
  const [filteredItems, setFilteredItems] = useState<SupplyItem[]>([]);
  const [searchTerm, setSearchTerm] = useState('');

  // State for dialog
  const [dialogOpen, setDialogOpen] = useState(false);
  const [isEditing, setIsEditing] = useState(false);
  const [currentItem, setCurrentItem] = useState<Partial<SupplyItem>>({
    description: '',
    priceEuro: 0,
    quantity: 0
  });

  // State for loading and feedback
  const [loading, setLoading] = useState(false);
  const [snackbarOpen, setSnackbarOpen] = useState(false);
  const [snackbarMessage, setSnackbarMessage] = useState('');
  const [snackbarSeverity, setSnackbarSeverity] = useState<'success' | 'error' | 'info' | 'warning'>('success');

  // State for file import
  const fileInputRef = useRef<HTMLInputElement>(null);

  // State for loading
  const [deletingId, setDeletingId] = useState<string | null>(null);

  // Load items when component mounts
  useEffect(() => {
    loadItems();
  }, []);

  // Filter items when search term changes
  useEffect(() => {
    filterItems();
  }, [searchTerm, items]);

  // Load all items from API
  const loadItems = async () => {
    try {
      setLoading(true);
      const data = await itemsApi.getAllItems();

      // Transform the data to match our frontend structure
      const transformedItems = data.map((item: any) => ({
        id: item.id,
        description: item.description,
        priceEuro: item.price,
        quantity: item.quantity
      }));

      setItems(transformedItems);
      setFilteredItems(transformedItems);
    } catch (error) {
      console.error('Error loading items:', error);
      showSnackbar('Erreur lors du chargement des articles', 'error');
    } finally {
      setLoading(false);
    }
  };

  // Filter items based on search term
  const filterItems = () => {
    if (!searchTerm.trim()) {
      setFilteredItems(items);
    } else {
      const filtered = items.filter(item =>
        item.description.toLowerCase().includes(searchTerm.toLowerCase())
      );
      setFilteredItems(filtered);
    }
  };

  // Show snackbar with message
  const showSnackbar = (message: string, severity: 'success' | 'error' | 'info' | 'warning') => {
    setSnackbarMessage(message);
    setSnackbarSeverity(severity);
    setSnackbarOpen(true);
  };

  // Close snackbar
  const handleCloseSnackbar = () => {
    setSnackbarOpen(false);
  };

  // Open dialog to add a new item
  const handleAddItem = () => {
    setCurrentItem({
      description: '',
      priceEuro: 0,
      quantity: 0
    });
    setIsEditing(false);
    setDialogOpen(true);
  };

  // Close dialog
  const handleCloseDialog = () => {
    setDialogOpen(false);
  };

  // Handle input change in dialog
  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setCurrentItem({
      ...currentItem,
      [name]: name === 'priceEuro' ? parseFloat(value) : name === 'quantity' ? (value === '' ? 0 : parseInt(value)) : value
    });
  };

  // Save item (create or update)
  const handleSaveItem = async () => {
    if (!currentItem.description) {
      showSnackbar('La description est requise', 'error');
      return;
    }

    try {
      setLoading(true);
      const itemData = {
        description: currentItem.description,
        price: currentItem.priceEuro,
        quantity: currentItem.quantity === undefined || currentItem.quantity === null ? 0 : currentItem.quantity
      };

      if (isEditing && currentItem.id) {
        await itemsApi.updateItem(currentItem.id, itemData);
      } else {
        await itemsApi.createItem(itemData);
      }

      showSnackbar(`Article ${isEditing ? 'mis à jour' : 'créé'} avec succès`, 'success');
      handleCloseDialog();
      await loadItems();
    } catch (error) {
      showSnackbar(`Erreur lors de ${isEditing ? 'la mise à jour' : 'la création'} de l'article`, 'error');
    } finally {
      setLoading(false);
    }
  };

  // Delete item
  const handleDeleteItem = async (id: string) => {
    if (window.confirm('Êtes-vous sûr de vouloir supprimer cet article ?')) {
      try {
        setDeletingId(id);
        await itemsApi.deleteItem(id);
        setItems(prevItems => prevItems.filter(item => item.id !== id));
        setFilteredItems(prevItems => prevItems.filter(item => item.id !== id));
        showSnackbar('Article supprimé avec succès', 'success');
      } catch (error) {
        showSnackbar('Erreur lors de la suppression de l\'article', 'error');
      } finally {
        setDeletingId(null);
      }
    }
  };

  // Handle file selection
  const handleFileSelect = (event: React.ChangeEvent<HTMLInputElement>) => {
    const file = event.target.files?.[0];
    if (file) {
      handleFileUpload(file);
    }
  };

  // Add this new function after handleSaveItem
  const handleImportItems = async (items: { description: string; priceEuro: number }[]) => {
    const results = {
      successful: [] as any[],
      failed: [] as any[],
      total: items.length,
      imported: 0
    };

    try {
      setLoading(true);

      // Process each item one by one
      for (const item of items) {
        try {
          // Validate item
          if (!item.description) {
            results.failed.push({
              item,
              error: 'La description est requise'
            });
            continue;
          }

          // Prepare item data (same structure as handleSaveItem)
          const itemData = {
            description: item.description,
            price: item.priceEuro,
            quantity: 0 // Default quantity for import
          };

          // Create item using the same API call as handleSaveItem
          const response = await itemsApi.createItem(itemData);

          results.successful.push({
            original: item,
            created: response
          });
          results.imported++;

          // Show progress
          showSnackbar(
            `Import en cours: ${results.imported}/${items.length} articles traités...`,
            'info'
          );
        } catch (error) {
          results.failed.push({
            item,
            error: `Erreur: ${(error as Error).message}`
          });
        }
      }

      // Show final results
      if (results.failed.length > 0) {
        if (results.successful.length > 0) {
          showSnackbar(
            `Import partiel: ${results.successful.length} articles importés, ${results.failed.length} échecs`,
            'warning' as const
          );
        } else {
          showSnackbar('Échec de l\'import: aucun article importé', 'error');
        }
        console.error('Failed imports:', results.failed);
      } else {
        showSnackbar(`${results.successful.length} articles importés avec succès`, 'success');
      }

      // Refresh the list
      await loadItems();
    } catch (error) {
      console.error('Error during import:', error);
      showSnackbar('Erreur lors de l\'import', 'error');
    } finally {
      setLoading(false);
    }

    return results;
  };

  // Update handleFileUpload function
  const handleFileUpload = async (file: File) => {
    try {
      setLoading(true);
      const reader = new FileReader();

      reader.onload = async (event: ProgressEvent<FileReader>) => {
        try {
          const data = event.target?.result;
          if (typeof data !== 'string') {
            throw new Error('Invalid file data');
          }

          // Read the Excel file
          const workbook = XLSX.read(data, { type: 'binary' });
          const sheetName = workbook.SheetNames[0];
          const worksheet = workbook.Sheets[sheetName];

          // Convert worksheet to JSON
          const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });

          // Skip header row and process data
          const processedItems: ProcessedExcelItem[] = [];
          for (let i = 1; i < jsonData.length; i++) {
            const row = jsonData[i] as any[];
            if (!row || row.length < 4) continue;

            const description = String(row[1] || '').trim(); // Column B
            if (!description) continue;

            let quantity = 0;
            const quantityValue = row[2]; // Column C
            if (typeof quantityValue === 'number') {
              quantity = Math.max(0, Math.floor(quantityValue));
            } else if (typeof quantityValue === 'string') {
              const cleanQuantity = quantityValue.replace(/[^0-9]/g, '');
              quantity = parseInt(cleanQuantity) || 0;
            }

            let price = 0;
            const priceValue = row[3]; // Column D
            if (typeof priceValue === 'number') {
              price = priceValue;
            } else if (typeof priceValue === 'string') {
              const cleanPrice = priceValue.replace(/[^0-9.,]/g, '').replace(',', '.');
              price = parseFloat(cleanPrice);
              if (isNaN(price)) price = 0;
            }

            const processedItem: ProcessedExcelItem = {
              description,
              priceEuro: price,
              quantity: quantity,
              rowIndex: i + 1,
              isValid: true,
              originalRow: {
                description: row[1],
                quantity: row[2],
                price: row[3]
              }
            };

            processedItems.push(processedItem);
          }

          if (processedItems.length > 0) {
            // Save each processed item individually
            let successCount = 0;
            let errorCount = 0;

            for (const item of processedItems) {
              try {
                // Create a new item using the API
                const response = await fetch(`${API_BASE_URL}/items`, {
                  method: 'POST',
                  headers: {
                    'Content-Type': 'application/json',
                  },
                  body: JSON.stringify({
                    description: item.description,
                    price: item.priceEuro,
                    quantity: item.quantity
                  })
                });

                if (!response.ok) {
                  throw new Error(`Failed to save item: ${response.statusText}`);
                }

                await response.json();
                successCount++;
              } catch (error) {
                errorCount++;
              }
            }

            // Show final summary
            showSnackbar(
              `Import terminé: ${successCount} articles importés avec succès, ${errorCount} erreurs`,
              successCount > 0 ? 'success' : 'error'
            );

            // Refresh the items list
            await loadItems();
          }

        } catch (error) {
          showSnackbar(
            'Erreur lors du traitement du fichier: ' + (error as Error).message,
            'error'
          );
        }
      };

      reader.readAsBinaryString(file);
    } catch (error) {
      showSnackbar(
        'Erreur lors de la lecture du fichier: ' + (error as Error).message,
        'error'
      );
    } finally {
      setLoading(false);
      if (fileInputRef.current) {
        fileInputRef.current.value = '';
      }
    }
  };

  return (
    <Layout currentPath={currentPath} onNavigate={onNavigate}>
      <Box sx={{ display: 'flex', position: 'relative', width: '100%',height: '80px', backgroundColor: '#1976d2' , color: 'white'}} className="page-header">
        <Box sx={{ position: 'absolute', left: 0 , display: 'flex', alignItems: 'center',gap: 25 }}>
          <img
            src={logo}
            alt="Logo"
            style={{ height: '60px' }}
          />
          <Typography variant="h6" className="header-title">
          ARTICLES DE FOURNITURE
          </Typography>
        </Box>
         </Box>
      <Container className="items-page-container">
        <Paper elevation={2} className="items-paper">
          <Box className="items-header">
            <Box sx={{ display: 'flex', alignItems: 'center', gap: 2 }}>
              <Typography variant="subtitle1" sx={{ color: 'text.secondary',marginRight: 2 }}>
                ({items.length} articles)
              </Typography>
            </Box>
            <Box className="search-and-add">
              <TextField
                label="Rechercher un article"
                variant="outlined"
                size="small"
                value={searchTerm}
                onChange={(e) => setSearchTerm(e.target.value)}
                className="search-field"
              />
              <Button
                variant="contained"
                color="primary"
                startIcon={<AddIcon />}
                onClick={handleAddItem}
                disabled={loading}
              >
                Ajouter un article
              </Button>
              <input
                type="file"
                ref={fileInputRef}
                onChange={handleFileSelect}
                accept=".xlsx,.xls"
                style={{ display: 'none' }}
              />
              <Button
                variant="outlined"
                color="primary"
                startIcon={<FileUploadIcon />}
                onClick={() => fileInputRef.current?.click()}
                disabled={loading}
              >
                Importer Excel
              </Button>
            </Box>
          </Box>

          <TableContainer className="items-table-container">
            <Table size="small">
              <TableHead>
                <TableRow>
                  <TableCell>Description</TableCell>
                  <TableCell align="right">Prix (€)</TableCell>
                  <TableCell align="right">Quantité</TableCell>
                  <TableCell align="center">Actions</TableCell>
                </TableRow>
              </TableHead>
              <TableBody>
                {loading ? (
                  <TableRow>
                    <TableCell colSpan={5} align="center">
                      Chargement...
                    </TableCell>
                  </TableRow>
                ) : filteredItems.length === 0 ? (
                  <TableRow>
                    <TableCell colSpan={5} align="center">
                      Aucun article trouvé
                    </TableCell>
                  </TableRow>
                ) : (
                  filteredItems.map((item) => (
                    <TableRow key={item.id}>
                      <TableCell>
                        {item.description}
                        {item.quantity === 0 && (
                          <Tooltip title="Quantité nulle : veuillez réapprovisionner" placement="right">
                            <WarningAmberIcon style={{ color: '#FFA000', marginLeft: 8, verticalAlign: 'middle' }} fontSize="small" />
                          </Tooltip>
                        )}
                      </TableCell>
                      <TableCell align="right">
                        {item.priceEuro ? Number(item.priceEuro).toFixed(2) : '0.00'}
                      </TableCell>
                      <TableCell align="right">
                        {item.quantity !== undefined && item.quantity !== null ? item.quantity : 0}
                      </TableCell>
                      <TableCell align="center">
                        <IconButton
                          size="small"
                          color="primary"
                          onClick={() => {
                            setCurrentItem(item);
                            setIsEditing(true);
                            setDialogOpen(true);
                          }}
                        >
                          <EditIcon fontSize="small" />
                        </IconButton>
                        <IconButton
                          size="small"
                          color="error"
                          onClick={() => handleDeleteItem(item.id)}
                          disabled={deletingId === item.id}
                        >
                          <DeleteIcon fontSize="small" />
                        </IconButton>
                      </TableCell>
                    </TableRow>
                  ))
                )}
              </TableBody>
            </Table>
          </TableContainer>
        </Paper>
      </Container>

      {/* Dialog for adding/editing items */}
      <Dialog
        open={dialogOpen}
        onClose={handleCloseDialog}
        maxWidth="sm"
        fullWidth
      >
        <DialogTitle>
          {isEditing ? 'Modifier l\'article' : 'Ajouter un nouvel article'}
        </DialogTitle>
        <DialogContent>
          <TextField
            autoFocus
            margin="dense"
            name="description"
            label="Description"
            type="text"
            fullWidth
            variant="outlined"
            value={currentItem.description}
            onChange={handleInputChange}
          />
          <CustomNumberInput
            label="Prix (€)"
            value={currentItem.priceEuro || 0}
            onChange={(value) => setCurrentItem({ ...currentItem, priceEuro: value })}
            step={0.01}
            min={0}
            fullWidth
            margin="dense"
            variant="outlined"
          />
          <CustomNumberInput
            label="Quantité"
            value={currentItem.quantity ?? 0}
            onChange={(value) => setCurrentItem({ ...currentItem, quantity: value })}
            step={1}
            min={0}
            fullWidth
            margin="dense"
            variant="outlined"
          />
        </DialogContent>
        <DialogActions>
          <Button onClick={handleCloseDialog}>Annuler</Button>
          <Button onClick={handleSaveItem} color="primary" variant="contained" disabled={loading}>
            {loading ? 'Enregistrement...' : 'Enregistrer'}
          </Button>
        </DialogActions>
      </Dialog>

      {/* Snackbar for notifications */}
      <Snackbar
        open={snackbarOpen}
        autoHideDuration={6000}
        onClose={handleCloseSnackbar}
        anchorOrigin={{ vertical: 'bottom', horizontal: 'center' }}
      >
        <Alert
          onClose={handleCloseSnackbar}
          severity={snackbarSeverity}
          sx={{ width: '100%' }}
        >
          {snackbarMessage}
        </Alert>
      </Snackbar>
    </Layout>
  );
};

export default ItemsPage;

============================================================
FILE: src/pages/LoginPage/LoginPage.scss
============================================================
.login-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f5f5f5;
  padding: 2rem;
}

.login-paper {
  padding: 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2rem;
}

.login-logo-container {
  display: flex;
  justify-content: center;
  margin-bottom: 1rem;
}

.login-logo {
  height: 120px;
  width: auto;
}

.login-title {
  color: #1976d2;
  font-weight: 600;
  margin-bottom: 1rem;
}

.login-form {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.login-button {
  margin-top: 1rem;
  padding: 0.75rem;
  font-weight: 600;
}

============================================================
FILE: src/pages/LoginPage/LoginPage.tsx
============================================================
import React, { useState } from 'react';
import { Box, Container, Paper, TextField, Button, Typography } from '@mui/material';
import logo from '../../logo.png';
import './LoginPage.scss';

interface LoginPageProps {
  onLogin: (username: string, password: string) => void;
}

const LoginPage: React.FC<LoginPageProps> = ({ onLogin }) => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onLogin(username, password);
  };

  return (
    <Box className="login-container">
      <Container maxWidth="sm">
        <Paper elevation={3} className="login-paper">
          <Box className="login-logo-container">
            <img
              src={logo}
              alt="Logo"
              className="login-logo"
            />
          </Box>

          <Typography variant="h5" component="h1" className="login-title">
            Connexion
          </Typography>

          <form onSubmit={handleSubmit} className="login-form">
            <TextField
              fullWidth
              label="Nom d'utilisateur"
              variant="outlined"
              margin="normal"
              value={username}
              onChange={(e) => setUsername(e.target.value)}
              required
            />

            <TextField
              fullWidth
              label="Mot de passe"
              type="password"
              variant="outlined"
              margin="normal"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              required
            />

            <Button
              type="submit"
              variant="contained"
              color="primary"
              fullWidth
              size="large"
              className="login-button"
            >
              Se connecter
            </Button>
          </form>
        </Paper>
      </Container>
    </Box>
  );
};

export default LoginPage;

============================================================
FILE: src/pages/PriceOfferPage/PriceOfferPage.scss
============================================================
.price-offer-page {
  padding: 2rem 0;

  .actions-bar {
    display: flex;
    justify-content: flex-end;
    margin-bottom: 1rem;
  }

  .price-offer-content {
    padding: 2rem;
    background-color: white;
    min-height: 29.7cm;
    width: 21cm;
    margin: 0 auto;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);

    .header {
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
      margin-bottom: 2rem;
      padding-bottom: 0;

      .logo {
        width: 120px;
        height: auto;
        object-fit: contain;
      }

      .chanitec-logo {
        width: 350px;
        height: auto;
        object-fit: contain;
        margin-top: -20px;
      }
    }

    .quote-info-grid {
      width: 60%;
      margin-bottom: 2rem;
      border-collapse: collapse;
      display: table;
      margin-left: 40%;

      .info-box {
        display: table-cell;
        padding: 0.5rem;
        border: 1px solid #000;

        &:first-child {
          width: 120px;
          background-color: white;
        }

        .MuiTypography-subtitle2 {
          color: #000;
          font-weight: 500;
          font-size: 0.75rem;
          font-weight: 700;
          padding: 0.3rem 0;
          text-transform: uppercase;
        }

        .MuiTypography-body1 {
          color: #000;
          font-size: 0.9rem;
          padding: 0.3rem 0;
        }
      }
    }

    .items-table {
      width: 100%;
      border-collapse: collapse;
      margin-bottom: 0;
      th, td {
        padding: 4px;
        text-align: left;
        border: 1px solid #000;
        font-size: 0.7rem;
      }

      th {
        background-color: white;
        font-weight: bold;
        text-align: center;
        text-transform: uppercase;
      }

      td {
        vertical-align: middle;
        text-align: center;
        height: 20px;
      }

      th:nth-child(6),th:nth-child(4),th:nth-child(5) {
        text-align: right;
      }
      td:nth-child(6),td:nth-child(4),td:nth-child(5) {
        text-align: right;
      }
      th:nth-child(6){
        width: 120px;
      }
      th:nth-child(5){
        width: 80px;
      }
      th:nth-child(4){
        width: 60px;
      }
    }

    .totals {
      display: table;
      width: 100%;
      margin-top: 0;
      border-collapse: collapse;

      .total-line , .total-row {
        width: 17%;
        display: table-cell;

        color: #000; border: 1px solid #000;
        font-size: 0.9rem;
        padding: .4rem;
        .MuiTypography-subtitle1 {

          font-weight: 500;
          font-size: 0.75rem;
          font-weight: 700;
          padding: 0.3rem 0;
          text-transform: uppercase;

        }

        .MuiTypography-body1 {
          color: #000;
          font-size: 0.9rem;
          padding: 0.3rem 0;
          font-weight: 700;
        }
      }
      .total-line {
        width: 85%;
      }
    }

    .conditions {
      margin-top: 1rem;
      margin-bottom: 1rem;

      h6 {
        color: #000;
        font-size: 0.9rem;

        margin-bottom: 0.5rem;
      }

      .condition-grid {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
        gap: 0.5rem;

        .MuiTypography-subtitle2 {
          color: #000;
          font-size: 0.9rem;
        font-weight: 700;
        }

        .MuiTypography-body2 {
          color: #000;
          font-size: 0.9rem;

        }
      }
    }

    .bank-details {
      margin-top: 2rem;
      font-size: 0.9rem;

      .MuiTypography-subtitle2 {
        color: #000;
        font-weight: normal;
        margin-bottom: 0.5rem;
      }

      .MuiTypography-body2 {
        color: #000;
        line-height: 1.4;
        margin-bottom: 0.3rem;
      }
    }

    .signatures-row {
      display: flex;
      justify-content: space-between;
      align-items: flex-end;
      margin-top: 4rem;
      margin-bottom: 2rem;
      width: 100%;

      .signature {
        display: flex;
        flex-direction: column;
        align-items: flex-start;
        width: 10%;
      }
      .signature-right {
        align-items: flex-start;
        text-align: left;
        margin-left: 20%;
      }
      .signature-line {
        margin-top: 1.5rem;
        border-bottom: 1.5px solid #000;
        width: 100%;
        height: 2px;
      }
    }
    .signatures-row.updated-signatures {
      margin-top: 2rem;
      margin-bottom: 1rem;
      gap: 3rem;
      .signature {
        width: 45%;
      }
      .signature-placeholder {
        min-height: 60px;
        width: 100%;
        margin-top: 16px;
        margin-bottom: 8px;
      }
    }
    .price-offer-note {
      text-align: center;
      font-size: 0.85rem;
      margin-bottom: 1.5rem;
      color: #222;
    }
    .price-offer-footer {
      border-top: 1px solid #000;
      padding-top: 0.5rem;
      text-align: center;
      font-size: 0.8rem;
      color: #222;
      div {
        margin-bottom: 2px;
      }
    }
  }
}

@media print {
  .actions-bar {
    display: none !important;

  }

  .price-offer-content {
    padding: 1cm !important;
    box-shadow: none !important;
  }
}

============================================================
FILE: src/pages/PriceOfferPage/PriceOfferPage.tsx
============================================================
import React, { useEffect, useState, useRef } from 'react';
import {
  Box,
  Container,
  Typography,
  Paper,
  Divider,
  Button,
  Select,
  MenuItem,
  FormControl,
  InputLabel
} from '@mui/material';
import { PrintOutlined, Download } from '@mui/icons-material';
import { format } from 'date-fns';
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';
import Layout from '../../components/Layout/Layout';
import { PriceOffer } from '../../models/PriceOffer';
import { priceOfferService } from '../../services/price-offer-service';
import { apiService } from '../../services/api-service';
import './PriceOfferPage.scss';
import logo512 from '../../assets/logo512.png';
import logoChanitec from '../../assets/logo chanitecc.png';
import signatureAyachi from '../../assets/signature-ayachi.png';
import signaturePerrache from '../../assets/signature-perrache.png.png';

interface PriceOfferPageProps {
  currentPath: string;
  onNavigate: (path: string, quoteId?: string) => void;
  quoteId?: string;
}

const PriceOfferPage: React.FC<PriceOfferPageProps> = ({ currentPath, onNavigate, quoteId }) => {
  const [priceOffer, setPriceOffer] = useState<PriceOffer | null>(null);
  const [numberToDisplay, setNumberToDisplay] = useState<string>('');
  const [error, setError] = useState<string | null>(null);
  const contentRef = useRef<HTMLDivElement>(null);
  const [signatureOption, setSignatureOption] = useState<'single' | 'double' | 'none'>('double');

  useEffect(() => {
    const loadPriceOffer = async () => {
      // Try to get quoteId from URL, fallback to prop
      const urlParams = new URLSearchParams(window.location.search);
      const quoteIdFromUrl = urlParams.get('id') || quoteId;
      console.log('Quote ID from URL or prop:', quoteIdFromUrl);

      if (!quoteIdFromUrl) {
        setError('No quote ID provided');
        return;
      }

      try {
        // Find the quote in the list to get createdAt
        let createdAt = '';
        let numberToDisplayLocal = '';
        try {
          const allQuotes = await fetch(`${process.env.REACT_APP_API_URL}/quotes`).then(res => res.json());
          const found = allQuotes.find((q: any) => q.id === quoteIdFromUrl);
          if (found) {
            createdAt = found.createdAt;
            numberToDisplayLocal = found.number_chanitec && found.number_chanitec.trim() !== '' ? found.number_chanitec : found.id;
          } else {
            setError('Quote not found');
            return;
          }
        } catch (e) {
          setError('Failed to fetch quote list');
          return;
        }
        const quote = await apiService.getQuoteById(quoteIdFromUrl);
        console.log('Fetched quote data:', quote);

        if (!quote) {
          setError('Quote not found');
          return;
        }

        // Create a new price offer from the quote
        const newOffer: PriceOffer = {
          quoteId: quote.id,
          clientName: quote.clientName,
          siteName: quote.siteName,
          object: quote.object,
          date: quote.date,
          supplyDescription: quote.supplyDescription,
          supplyTotalHT: quote.totalSuppliesHT,
          laborDescription: quote.laborDescription,
          laborTotalHT: quote.totalLaborHT,
          totalHT: quote.totalHT,
          tva: quote.tva,
          totalTTC: quote.totalTTC,
          createdAt: new Date().toISOString(),
          updatedAt: new Date().toISOString()
        };

        // Save the new price offer
        const savedOffer = priceOfferService.createFromQuote(quote);
        console.log('Created and saved price offer:', savedOffer);

        setPriceOffer(savedOffer);
        setNumberToDisplay(numberToDisplayLocal || (quote.number_chanitec && quote.number_chanitec.trim() !== '' ? quote.number_chanitec : quote.id));
      } catch (error) {
        console.error('Error loading price offer:', error);
        setError('Failed to load price offer. Please try again.');
      }
    };

    loadPriceOffer();
  }, [quoteId]); // Add quoteId to dependencies

  const handlePrint = () => {
    window.print();
  };

  const handleDownloadPDF = async () => {
    if (!contentRef.current) return;

    const canvas = await html2canvas(contentRef.current, {
      scale: 2,
      useCORS: true,
      logging: false,
      backgroundColor: '#ffffff'
    });

    const imgData = canvas.toDataURL('image/png');
    // Set PDF size to match the rendered content (no A4 limitation)
    const pdf = new jsPDF({
      orientation: 'portrait',
      unit: 'px',
      format: [canvas.width, canvas.height]
    });

    pdf.addImage(imgData, 'PNG', 0, 0, canvas.width, canvas.height);
    pdf.save(`Offer de Prix_${numberToDisplay || priceOffer?.quoteId || 'unknown'}.pdf`);
  };

  if (error) {
    return (
      <Layout currentPath={currentPath} onNavigate={onNavigate}>
        <Box className="error-container">
          <Typography variant="h6" color="error">
            {error}
          </Typography>
          <Button
            variant="contained"
            onClick={() => onNavigate('/history', undefined)}
            sx={{ mt: 2 }}
          >
            Retour à l'historique
          </Button>
        </Box>
      </Layout>
    );
  }

  if (!priceOffer) {
    return (
      <Layout currentPath={currentPath} onNavigate={onNavigate}>
        <Box className="loading-container">
          <Typography variant="h6">Chargement...</Typography>
        </Box>
      </Layout>
    );
  }

  return (
    <Layout currentPath={currentPath} onNavigate={onNavigate}>
      <Container maxWidth="lg" className="price-offer-page">


        {/* Wrap the PDF content in a plain div with the ref */}
        <div ref={contentRef}>
          <Paper className="price-offer-content" elevation={2}>
            {/* Header with logos */}
            <Box className="header" sx={{ marginBottom: 4 }}>
              <img src={logo512} alt="Logo" className="logo" />
              <img src={logoChanitec} alt="Chanitec Logo" className="chanitec-logo" />
            </Box>

            {/* Quote Information */}
            <Box className="quote-info-grid">
              <Box className="info-box">
                <Typography variant="subtitle2">NUMERO DEVIS</Typography>
                <Typography variant="subtitle2">CLIENT</Typography>
                <Typography variant="subtitle2">OBJET</Typography>
                <Typography variant="subtitle2">DATE</Typography>
              </Box>
              <Box className="info-box">
                <Typography variant="body1">{numberToDisplay || priceOffer.quoteId}</Typography>
                <Typography variant="body1">{priceOffer.clientName}</Typography>
                <Typography variant="body1">{priceOffer.object}</Typography>
                <Typography variant="body1">{format(new Date(priceOffer.date), 'dd/MM/yyyy')}</Typography>
              </Box>
            </Box>

            {/* Items Table */}
            <Box sx={{ mb: 4 }}>
              <table className="items-table">
                <thead>
                  <tr>
                    <th>ITEM</th>
                    <th>SERVICE / PRODUIT</th>
                    <th>DESIGNATION</th>
                    <th>QTE</th>
                    <th>P.U HT</th>
                    <th>P.T HT</th>
                  </tr>
                </thead>
                <tbody>
                  <tr>
                    <td>1</td>
                    <td>Equipement</td>
                    <td>{priceOffer.supplyDescription}</td>
                    <td>1,00</td>
                    <td>{(Number(priceOffer.supplyTotalHT ?? 0)).toFixed(2)}</td>
                    <td>{(Number(priceOffer.supplyTotalHT ?? 0)).toFixed(2)}</td>
                  </tr>
                  <tr>
                    <td>2</td>
                    <td>Prestation</td>
                    <td>{priceOffer.laborDescription}</td>
                    <td>1,00</td>
                    <td>{(Number(priceOffer.laborTotalHT ?? 0)).toFixed(2)}</td>
                    <td>{(Number(priceOffer.laborTotalHT ?? 0)).toFixed(2)}</td>
                  </tr>
                </tbody>
              </table>
            </Box>

            {/* Totals */}
            <Box className="totals" sx={{ mb: 4 }}>
              <Box className="total-line">
                <Typography variant="subtitle1">TOTAL USD HT</Typography>
                <Typography variant="subtitle1">TVA: 16%</Typography>
                <Typography variant="subtitle1">TOTAL USD TTC</Typography>
              </Box>
              <Box className="total-row">
                <Typography variant="body1">
                  {(Number(priceOffer.totalHT ?? 0)).toFixed(2)}
                </Typography>
                <Typography variant="body1">
                  {(Number(priceOffer.tva ?? 0)).toFixed(2)}
                </Typography>
                <Typography variant="body1" fontWeight="bold">
                  {(Number(priceOffer.totalTTC ?? 0)).toFixed(2)}
                </Typography>
              </Box>
            </Box>

            {/* Conditions */}
            <Box className="conditions" sx={{ mb: 4 }}>
              <Typography variant="h6" gutterBottom>
                Conditions:
              </Typography>
              <Box sx={{ display: 'flex', flexWrap: 'wrap', gap: 2 }}>
                <Box sx={{ flex: '1 1 45%' }}>
                  <Typography style={{fontWeight: 'bold'}} variant="subtitle2">Validité de l'offre:</Typography>
                  <Typography style={{fontWeight: 'bold'}} variant="subtitle2">Garantie:</Typography>
                  <Typography style={{fontWeight: 'bold'}} variant="subtitle2">Poids / Volume:</Typography>
                  <Typography style={{fontWeight: 'bold'}} variant="subtitle2">Délai de Livraison:</Typography>
                  <Typography style={{fontWeight: 'bold'}} variant="subtitle2">Pays d'origine / provenance:</Typography>
                  <Typography style={{fontWeight: 'bold'}} variant="subtitle2">Modalité de paiement:</Typography>
                </Box>
                <Box sx={{ flex: '1 1 45%' }}>
                  <Typography style={{ marginBottom: '.5rem' }} variant="body2">14 jours à dater de l'offre</Typography>
                  <Typography style={{ marginBottom: '.5rem' }} variant="body2">N/A</Typography>
                  <Typography style={{ marginBottom: '.5rem' }} variant="body2">N/A</Typography>
                  <Typography style={{ marginBottom: '.5rem' }} variant="body2">+/- 1 jour</Typography>
                  <Typography style={{ marginBottom: '.5rem' }} variant="body2">N/A</Typography>
                  <Typography style={{ marginBottom: '.5rem' }} variant="body2">100% à la commande</Typography>
                </Box>
              </Box>
            </Box>

            <Typography style={{fontWeight: 'bold', marginBottom: '2rem',fontSize: '.8rem'}} variant="subtitle2" gutterBottom>
              NB: Après prise de RDV, le client doit faciliter l'acces au bureau/pièce concerné. Si l'intervnetion ne peut se faire faute de disponibilité, un déplacment pourra être facturé
            </Typography>

            {/* Bank Details */}
            <Box className="bank-details">
              <Typography variant="subtitle2" gutterBottom>
                PAIEMENT PAR VIREMENT BANCAIRE SUR L'UN DES COMPTES SUIVANT OU EN ESPECES A NOTRE CAISSE
              </Typography>
              <Typography variant="body2">
                BOA USD 00 029-01001-01033700012-13 EQUITYBCDC USD 00 011-00101-00000117533-24
              </Typography>
              <Typography variant="body2">
                FBN USD 00 084-84001-20300170009-55  RAWBANK USD 05 100-05101-01000022102-18
              </Typography>
              <Typography variant="body2">
                SOFIBANQUE USD 00 023-20190-01661660200-15 TMB USD 00 017-11000-50045270101-56
              </Typography>
              <Typography variant="body2">
                ECOBANK USD 00 026-00001-03600841201-27
              </Typography>
            </Box>
            {/* Signatures and Footer (Updated) */}
            <Box className="signatures-row updated-signatures">
              <Box className="signature signature-left">
                <Typography variant="subtitle2" fontWeight="bold">Bilel AYACHI</Typography>
                <Typography variant="body2" fontWeight="bold">Responsable Dpt Climatisation et Froid</Typography>
                <Box className="signature-placeholder" sx={{ minHeight: '60px', margin: '16px 0', display: 'block', textAlign: 'center' }}>
                  {(signatureOption === 'double' || signatureOption === 'single') && (
                    <img src={signatureAyachi} alt="Signature Bilel Ayachi" style={{ maxHeight: '60px', maxWidth: '100%', display: 'block', margin: '0 auto' }} />
                  )}
                </Box>
              </Box>
              <Box className="signature signature-right" sx={{ textAlign: 'left' }}>
                <Typography variant="subtitle2" fontWeight="bold">Amandine PERRACHE - MINESI</Typography>
                <Typography variant="body2" fontWeight="bold">Directrice Commerciale</Typography>
                <Box className="signature-placeholder" sx={{ minHeight: '60px', margin: '16px 0', display: 'flex', alignItems: 'center', textAlign: 'center' }}>
                  {signatureOption === 'double' && (
                    <>
                      <img src={signatureAyachi} alt="Signature Bilel Ayachi" style={{ maxHeight: '60px', maxWidth: '100%', display: 'inline-block', margin: '0 auto' }} />
                      <span style={{ marginLeft: '0.1rem', fontWeight: 'bold', fontSize: '1.1em' }}>P/I</span>
                    </>
                  )}
                </Box>
              </Box>
            </Box>
            {/* No signature if 'none' is selected */}
            <Box className="price-offer-note" sx={{ textAlign: 'center', fontSize: '0.85rem', marginBottom: '1.5rem' }}>
              NB: Votre commande implique l’acceptation de nos conditions générales de vente consultables sur notre site web www.chanic.com
            </Box>
            <Box className="price-offer-footer" sx={{ borderTop: '1px solid #000', paddingTop: '0.5rem', textAlign: 'center', fontSize: '0.8rem' }}>
              <div>Groupe Chanimetal</div>
              <div>Avenue de la Montagne n°2297, C/Ngaliema, Kinshasa, R.D. CONGO - +243(0)81 715 27 20 - groupechanimetal@chanic.com</div>
              <div>RCCM 14-B-3324 Kin – Id Nat 01-F4200-A22984H – NIF A070005Q2 – TVA 0390</div>
            </Box>
          </Paper>
        </div>
        <Box className="actions-bar" sx={{ mb: 2, display: 'flex', alignItems: 'center' }}>
          <Button
            variant="contained"
            startIcon={<PrintOutlined />}
            onClick={handlePrint}
            sx={{ mr: 2 }}
          >
            Imprimer
          </Button>
          <FormControl size="small" sx={{ minWidth: 180 }}>
            <InputLabel id="signature-select-label">Signatures</InputLabel>
            <Select
              labelId="signature-select-label"
              id="signature-select"
              value={signatureOption}
              label="Signatures"
              onChange={e => setSignatureOption(e.target.value as 'single' | 'double' | 'none')}
            >
              <MenuItem value="single">Signature</MenuItem>
              <MenuItem value="double">Double signature</MenuItem>
              <MenuItem value="none">No signature</MenuItem>
            </Select>
          </FormControl>
        </Box>
      </Container>
    </Layout>
  );
};

export default PriceOfferPage;

============================================================
FILE: src/pages/QuotePage/QuotePage.scss
============================================================
.loading-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 200px;
  background-color: #f5f5f5;
  border-radius: 4px;
  margin-top: 2rem;
}

.quote-content {
  padding-bottom: 2rem;
  position: relative;

  &::before {
    content: '';
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-image: url('../../../public/logo512.png');
    background-repeat: no-repeat;
    background-position: center;
    background-size: 400px;
    opacity: 0.15;
    z-index: -1;
    pointer-events: none;
  }

  &.pdf-print-mode {
    @page {
      size: A4;
      margin: 0;
    }

    padding: 0;
    margin: 0;
    width: 100%;
    height: 100%;
    transform: scale(1);
    transform-origin: top center;
    page-break-inside: avoid;

    .quote-header,
    .supplies-section,
    .labor-section,
    .total-section {
      margin: 0;
      padding: 0;
      break-inside: avoid;
      page-break-inside: avoid;
    }

    .total-section {
      padding-bottom: 0;
      margin-top: 0;
    }

    // Adjust table styles for print
    table {
      width: 100%;
      border-collapse: collapse;
      font-size: 9pt;
      margin: 0;

      th, td {
        padding: 2px;
        border: 1px solid #ddd;
        font-size: 8pt;
      }

      th {
        background-color: #f5f5f5;
      }
    }

    // Hide action buttons and other unnecessary elements
    .quote-actions,
    .MuiButton-root,
    .MuiIconButton-root,
    .add-item-button,
    .add-labor-button,
    .item-actions,
    .add-labor-form {
      display: none !important;
    }

    // Optimize spacing in sections
    .section-title {
      font-size: 10pt;
      margin: 0;
      padding: 0;
      border: none;
    }

    // Reduce spacing in form fields
    .MuiTextField-root {
      margin: 0;

      .MuiInputBase-root {
        font-size: 8pt;
        padding: 0;
      }
    }

    // Adjust total containers
    .total-container {
      margin: 0;
      padding: 0;
      background: none;
      border: none;
    }

    // Ensure proper page breaks
    .supplies-section,
    .labor-section {
      page-break-after: auto;
      page-break-inside: avoid;
    }

    // Hide decorative elements
    .MuiPaper-root {
      box-shadow: none;
      background: none;
    }

    // Page header styles
    .page-header {
      background-color: transparent;

      .page-title {
        color: black;
      }
    }
  }
}

.page-header {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0.5rem 0;
  margin-bottom: 1rem;
  flex-direction: column;

  .page-title {
    font-size: 1.25rem;
    font-weight: 600;
    text-align: center;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    max-width: 100%;
    margin: 0;
    padding: 0;
  }

  .original-quote-id {
    font-size: 0.875rem;
    color: #666;
    margin-top: 0.5rem;
    font-style: italic;
  }
}

// Print styles
@media print {
  @page {
    size: A4;
    margin: 0;
  }

  body {
    margin: 0;
    padding: 0;
  }

  .quote-content {
    padding: 0;
    margin: 0;
    width: 100%;
    height: 100%;
    transform: scale(1);
    transform-origin: top center;
    page-break-inside: avoid;

    .quote-header,
    .supplies-section,
    .labor-section,
    .total-section {
      margin: 0;
      padding: 0;
      break-inside: avoid;
      page-break-inside: avoid;
    }

    .total-section {
      padding-bottom: 0;
      margin-top: 0;
    }

    // Adjust table styles for print
    table {
      width: 100%;
      border-collapse: collapse;
      font-size: 9pt;
      margin: 0;

      th, td {
        padding: 2px;
        border: 1px solid #ddd;
        font-size: 8pt;
      }

      th {
        background-color: #f5f5f5;
      }
    }

    // Hide action buttons and other unnecessary elements
    .quote-actions,
    .MuiButton-root,
    .MuiIconButton-root,
    .add-item-button,
    .add-labor-button,
    .item-actions,
    .add-labor-form {
      display: none !important;
    }

    // Optimize spacing in sections
    .section-title {
      font-size: 10pt;
      margin: 0;
      padding: 0;
      border: none;
    }

    // Reduce spacing in form fields
    .MuiTextField-root {
      margin: 0;

      .MuiInputBase-root {
        font-size: 8pt;
        padding: 0;
      }
    }

    // Adjust total containers
    .total-container {
      margin: 0;
      padding: 0;
      background: none;
      border: none;
    }

    // Ensure proper page breaks
    .supplies-section,
    .labor-section {
      page-break-after: auto;
      page-break-inside: avoid;
    }

    // Hide decorative elements
    .MuiPaper-root {
      box-shadow: none;
      background: none;
    }

    // Page header styles
    .page-header {
      background-color: transparent;

      .page-title {
        color: black;
      }
    }
  }
}

.quote-actions {
  display: flex;
  gap: 16px;
  margin-bottom: 16px;
  justify-content: flex-end;

  .MuiButton-root {
    text-transform: none;

    &.confirm-button {
      background-color: #4caf50;
      color: white;

      &:hover {
        background-color: #388e3c;
      }

      &.Mui-disabled {
        background-color: #a5d6a7;
        color: white;
      }
    }
  }
}


============================================================
FILE: src/pages/QuotePage/QuotePage.tsx
============================================================
import React, { useRef, useEffect, useState } from 'react';
import { Box, Container, Paper, Typography, AppBar, Toolbar, Button } from '@mui/material';
import Layout from '../../components/Layout/Layout';
import QuoteHeader from '../../components/QuoteHeader/QuoteHeader';
import SuppliesSection from '../../components/SuppliesSection/SuppliesSection';
import LaborSection from '../../components/LaborSection/LaborSection';
import TotalSection from '../../components/TotalSection/TotalSection';
import QuoteActions from '../../components/QuoteActions/QuoteActions';
import { useQuote } from '../../contexts/QuoteContext';
import './QuotePage.scss';
import logo from '../../logo.png';
import { useLocation } from 'react-router-dom';
import CheckCircleOutlineIcon from '@mui/icons-material/CheckCircleOutline';
import { apiService } from '../../services/api-service';
import Dialog from '@mui/material/Dialog';
import DialogTitle from '@mui/material/DialogTitle';
import DialogContent from '@mui/material/DialogContent';
import DialogActions from '@mui/material/DialogActions';
import TextField from '@mui/material/TextField';

interface QuotePageProps {
  currentPath: string;
  onNavigate: (path: string) => void;
  onLogout?: () => void;
}

const QuotePage: React.FC<QuotePageProps> = ({ currentPath, onNavigate, onLogout }) => {
  const {
    state,
    createNewQuote,
    saveQuote,
    updateQuote,
    setQuoteField,
    addSupplyItem,
    removeSupplyItem,
    addLaborItem,
    removeLaborItem,
    recalculateTotals,
    clearQuote,
    loadQuote,
    updateSupplyItem
  } = useQuote();

  const { currentQuote, isLoading, isExistingQuote, originalQuoteId } = state;
  const contentRef = useRef<HTMLDivElement>(null) as React.RefObject<HTMLDivElement>;
  const location = useLocation();
  const quoteId = new URLSearchParams(location.search).get('id');

  const [isFromHistory, setIsFromHistory] = useState(false);
  const [isConfirmed, setIsConfirmed] = useState(false);
  const [isReadOnly, setIsReadOnly] = useState(false);
  const [showConfirm, setShowConfirm] = useState(false);
  const [numberChanitec, setNumberChanitec] = useState(currentQuote?.number_chanitec || '');
  const [confirmDialogOpen, setConfirmDialogOpen] = useState(false);

  // Load quote if ID is provided in URL
  useEffect(() => {
    if (quoteId && !isLoading && (!currentQuote || currentQuote.id !== quoteId)) {
      const queryParams = new URLSearchParams(window.location.search);
      const fromHistory = queryParams.get('fromHistory') === 'true';
      // Fetch createdAt for the quoteId
      const fetchAndLoad = async () => {
        try {
          const allQuotes = await fetch(`${process.env.REACT_APP_API_URL}/quotes`).then(res => res.json());
          const found = allQuotes.find((q: any) => q.id === quoteId);
          if (found) {
            loadQuote(quoteId, found.createdAt, fromHistory);
          }
        } catch (e) {
          // fallback: try to load with empty createdAt
          loadQuote(quoteId, '', fromHistory);
        }
      };
      fetchAndLoad();
    }
  }, [quoteId, isLoading, loadQuote, currentQuote]);

  // Create a new quote if none exists and we're not loading one
  useEffect(() => {
    if (!currentQuote && !isLoading && !quoteId) {
      createNewQuote();
    }
  }, [currentQuote, isLoading, createNewQuote, quoteId]);

  useEffect(() => {
    const queryParams = new URLSearchParams(window.location.search);
    const fromHistory = queryParams.get('fromHistory') === 'true';
    const confirmed = queryParams.get('confirmed') === 'true';
    const showConfirmParam = queryParams.get('showConfirm') === 'true';
    setIsFromHistory(fromHistory);
    setIsConfirmed(confirmed);
    setShowConfirm(showConfirmParam);
  }, []);

  // Update isReadOnly when currentQuote changes
  useEffect(() => {
    if (currentQuote) {
      setIsReadOnly(currentQuote.confirmed || false);
      setIsConfirmed(currentQuote.confirmed || false);
      setNumberChanitec(currentQuote.number_chanitec || '');
    }
  }, [currentQuote]);

  // Handle home button click
  const handleHomeClick = () => {
    if (currentPath === '/') {
      clearQuote();
    }
    onNavigate('/');
  };

  const handleOpenConfirmDialog = () => {
    setConfirmDialogOpen(true);
  };

  const handleCloseConfirmDialog = () => {
    setConfirmDialogOpen(false);
  };

  const handleSaveConfirm = async () => {
    if (!currentQuote) return;
    if (!numberChanitec) {
      alert('Veuillez saisir le numéro CHANitec.');
      return;
    }
    try {
      const response = await apiService.confirmQuote(currentQuote.id, true, numberChanitec);
      setIsConfirmed(true);
      setIsReadOnly(true);
      setConfirmDialogOpen(false);
      alert(response.message || 'Devis confirmé avec succès');
    } catch (error: any) {
      if (error instanceof Error && error.message.includes('400')) {
        alert('Requête invalide : le champ "confirmed" doit être un booléen ou le numéro CHANitec est manquant.');
      } else if (error instanceof Error && error.message.includes('404')) {
        alert('Devis introuvable.');
      } else {
        alert('Erreur lors de la confirmation du devis. Veuillez réessayer.');
      }
      console.error('Error confirming quote:', error);
    }
  };

  // If no quote is loaded or is still loading, show loading
  if (!currentQuote || isLoading) {
    return (
      <Layout currentPath={currentPath} onNavigate={onNavigate} onLogout={onLogout} onHomeClick={handleHomeClick}>
        <Box className="page-header">
          <Typography variant="h6" component="h1" className="page-title">
            CALCUL DE PRIX OFFRE CLIMATISATION
          </Typography>
        </Box>
        <Box className="loading-container">
          <Typography variant="h6">Chargement...</Typography>
        </Box>
      </Layout>
    );
  }

  return (
    <Layout currentPath={currentPath} onNavigate={onNavigate} onLogout={onLogout} onHomeClick={handleHomeClick}>
      <Box sx={{ display: 'flex', position: 'relative', width: '100%' , backgroundColor: 'white' , color: 'black',height:'20%'}} className="page-header">
        <Box sx={{ position: 'absolute', left: 0 }}>
          <img
            src={logo}
            alt="Logo"
            style={{ height: '60px' }}
          />
        </Box>
        <Box sx={{  }}>
          <Typography variant="h6" component="h1" className="page-title">
            CALCUL DE PRIX OFFRE CLIMATISATION
          </Typography>

        </Box>
      </Box>
      <Container ref={contentRef} className="quote-content">
        <QuoteHeader
          quoteId={currentQuote.id}
          clientName={currentQuote.clientName}
          siteName={currentQuote.siteName}
          object={currentQuote.object}
          date={currentQuote.date}
          onClientChange={(value) => setQuoteField('clientName', value)}
          onSiteChange={(value) => setQuoteField('siteName', value)}
          onObjectChange={(value) => setQuoteField('object', value)}
          onDateChange={(value) => setQuoteField('date', value)}
          onClientMarginChange={(margin) => {
            setQuoteField('supplyMarginRate', margin);
            setQuoteField('laborMarginRate', margin);
          }}
        />

        <SuppliesSection
          items={currentQuote.supplyItems}
          description={currentQuote.supplyDescription}
          exchangeRate={currentQuote.supplyExchangeRate}
          marginRate={currentQuote.supplyMarginRate}
          totalHT={currentQuote.totalSuppliesHT}
          onAddItem={addSupplyItem}
          onRemoveItem={removeSupplyItem}
          onUpdateDescription={(value) => setQuoteField('supplyDescription', value)}
          onUpdateExchangeRate={(value) => setQuoteField('supplyExchangeRate', value)}
          onUpdateMarginRate={(value) => setQuoteField('supplyMarginRate', value)}
          onUpdateSupplyItem={updateSupplyItem}
        />

        <LaborSection
          items={currentQuote.laborItems}
          description={currentQuote.laborDescription}
          exchangeRate={currentQuote.laborExchangeRate}
          marginRate={currentQuote.laborMarginRate}
          totalHT={currentQuote.totalLaborHT}
          onAddItem={addLaborItem}
          onRemoveItem={removeLaborItem}
          onUpdateDescription={(value) => setQuoteField('laborDescription', value)}
          onUpdateExchangeRate={(value) => setQuoteField('laborExchangeRate', value)}
          onUpdateMarginRate={(value) => setQuoteField('laborMarginRate', value)}
        />

        <TotalSection
          totalSuppliesHT={currentQuote.totalSuppliesHT}
          totalLaborHT={currentQuote.totalLaborHT}
          totalHT={currentQuote.totalHT}
          tva={currentQuote.tva}
          totalTTC={currentQuote.totalTTC}
        />
      </Container>

      <Box sx={{ display: 'flex', justifyContent: 'flex-end', gap: 2, mb: 2 }}>
        {(showConfirm && !isConfirmed) || isFromHistory ? (
          <Button
            variant="contained"
            startIcon={<CheckCircleOutlineIcon />}
            onClick={handleOpenConfirmDialog}
            disabled={isConfirmed}
            sx={{
              backgroundColor: isConfirmed ? '#4caf50' : '#1976d2',
              '&:hover': {
                backgroundColor: isConfirmed ? '#4caf50' : '#1565c0',
              },
              '&.Mui-disabled': {
                backgroundColor: '#4caf50',
                color: 'white',
              }
            }}
          >
            {isConfirmed ? 'Devis Confirmé' : 'Confirmer le Devis'}
          </Button>
        ) : null}
      </Box>
      <Dialog open={confirmDialogOpen} onClose={handleCloseConfirmDialog}>
        <DialogTitle>Confirmer le Devis</DialogTitle>
        <DialogContent>
          <TextField
            autoFocus
            margin="dense"
            label="Numéro CHANitec"
            type="text"
            fullWidth
            value={numberChanitec}
            onChange={e => setNumberChanitec(e.target.value)}
            disabled={isConfirmed}
          />
        </DialogContent>
        <DialogActions>
          <Button onClick={handleCloseConfirmDialog} color="secondary">Annuler</Button>
          <Button onClick={handleSaveConfirm} color="primary" variant="contained" disabled={isConfirmed}>Sauvegarder</Button>
        </DialogActions>
      </Dialog>

      <QuoteActions
        clientName={currentQuote.clientName}
        siteName={currentQuote.siteName}
        date={currentQuote.date}
        isExistingQuote={isExistingQuote}
        onSave={saveQuote}
        onUpdate={updateQuote}
        onViewHistory={() => onNavigate('/history')}
        contentRef={contentRef}
        onPrint={() => onNavigate(`/quote-test?id=${currentQuote.id}`)}
        onDownloadPDF={() => onNavigate(`/quote-test?id=${currentQuote.id}`)}
      />
    </Layout>
  );
};

export default QuotePage;

============================================================
FILE: src/pages/QuoteTest/QuoteTest.scss
============================================================
// --- Added for pixel-perfect match with reference HTML ---
.reference-header {
  display: flex;
  width: 100%;
  align-items: center;
  justify-content: left;
  gap: 7%;
  margin: 0 auto;
}
.reference-logo {
  width: 80px;
  margin-bottom: 10px;
}
.reference-title {
  font-size: 24px;
  font-weight: bold;
  text-transform: uppercase;
  text-align: center;
  letter-spacing: 1px;
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  width: max-content;
  margin: 0 auto;
  right: unset;
}
.reference-subtitle {
  font-size: 18px;
  font-weight: bold;
  text-transform: uppercase;
  text-align: center;
  margin-top: 0;
}
.client-info-box {
  background: #f8f9fa;
  border: 1px solid #d3d3d3;
  border-radius: 8px;
  padding: 18px 5px 10px 5px;
}
.client-info-grid {
  display: grid;
  grid-template-columns: 70px 1fr 70px 1fr;
  gap: 10px 5px;
  align-items: center;
}
.client-info-label {
  font-weight: bold;
  color: #222;
  text-align: right;
}
.client-info-value input,
.client-info-value select {
  width: 100%;
  padding: 6px 10px;
  border: 1px solid #d3d3d3;
  border-radius: 4px;
  background: #fff;
  font-size: 18px;
}
.summary-table {
  float: right;
  margin-top: 10px;
  border-collapse: collapse;
  min-width: 270px;
}
.summary-table th, .summary-table td {
  border: 1px solid #d3d3d3;
  padding: 7px 14px;
  text-align: right;
  font-size: 15px;
}
.summary-table th {
  background: #e3f0fa;
  color: #1976d2;
  font-weight: bold;
  text-align: left;
}
.section-title {
  color: #1976d2;
  font-size: 18px;
  font-weight: bold;
  text-transform: uppercase;
  margin: 0px 0 10px 0;
  text-align: left;
  background: transparent !important;
}
.input-row {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
  align-items: center;
  justify-content: space-between;
  background: transparent !important;
}
.input-row input, .input-row select {
  padding: 6px 10px;
  border: 1px solid #d3d3d3;
  border-radius: 4px;
  font-size: 15px;
  background: #fff;
}
.input-row label {
  font-weight: bold;
  font-size: 15px;
  color: #222;
}
.tx-row span {
  font-weight: bold;
  font-size: 15px;
  color: #222;
}
.tx-row {
  display: flex;
  gap: 10px;
  align-items: center;

}
.input-row .add-btn {
  background: #1976d2;
  color: #fff;
  border: none;
  border-radius: 4px;
  padding: 7px 18px;
  font-size: 15px;
  cursor: pointer;
  font-weight: bold;
  transition: background 0.2s;
}
.input-row .add-btn:hover {
  background: #1256a3;
}
.item-actions{
  margin-bottom: 10px;
}
.data-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 10px;
  background: #fff;
  background: transparent !important;
}
.data-table th, .data-table td {
  border: 1px solid #d3d3d3;
  padding: 7px 10px;
  font-size: 15px;
  text-align: left;
}
.data-table th {
  background: #e3f0fa;
  color: #1976d2;
  font-weight: bold;
}
.data-table input {
  width: 80px;
  padding: 4px 6px;
  border: 1px solid #d3d3d3;
  border-radius: 4px;
  font-size: 15px;
  background: #f8f9fa;
  text-align: right;
}
.data-table .delete-btn {
  background: #e53935;
  color: #fff;
  border: none;
  border-radius: 4px;
  padding: 4px 10px;
  font-size: 16px;
  cursor: pointer;
  font-weight: bold;
  transition: background 0.2s;
}
.data-table .delete-btn:hover {
  background: #b71c1c;
}
.data-table tfoot td {
  font-weight: bold;
  background: #f8f9fa;
  background: transparent !important;
}
.totals-row td {
  background: #f8f9fa;
  font-weight: bold;
  background: transparent !important;
}
.clearfix::after {
  content: "";
  display: table;
  clear: both;
}
.btn-update {
  background-color: #2196F3;
  color: white;
  margin-right: 10px;
  border: none;
  border-radius: 4px;
  padding: 10px 20px;
  font-size: 15px;
  cursor: pointer;
  font-weight: bold;
  transition: background 0.2s;
}
.btn-update:disabled {
  background: #b0bec5;
  color: #fff;
  cursor: not-allowed;
}
.btn-update:hover {
  background-color: #1976D2;
}
.btn-save {
  background-color: #4CAF50;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 10px;
  font-size: 15px;
  font-weight: bold;
  transition: background 0.2s;
}
.btn-save:hover {
  background-color: #45a049;
}
.btn-print {
  background-color: #008CBA;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 15px;
  font-weight: bold;
  transition: background 0.2s;
}
.btn-print:hover {
  background-color: #007399;
}

// Hide action buttons in print and PDF mode only
@media print, screen and (class: is-pdf-mode) {
  .btn-print,
  .btn-save,
  .btn-update,
  .add-btn,
  .delete-btn {
    display: none !important;
  }
}

// Hide background logos by default
.background-logo,
.background-logo-second {
  display: none;
}

// --- Unified Print and PDF Styles ---
@media print, screen and (class: is-pdf-mode) {
  .background-logo {
    display: block;
    position: absolute;
    top: 40%;
    left: 50%;
    transform: translate(-50%, -50%);
    opacity: 0.1;
    z-index: 1000;
    width: 90%;
    max-width: 600px;
    pointer-events: none;
  }
  .background-logo-second {
    display: block;
    position: absolute;
    top: 75%;
    left: 50%;
    transform: translate(-50%, -50%);
    opacity: 0.1;
    z-index: 1000;
    width: 60%;
    max-width: 400px;
    pointer-events: none;
  }
  body, .main-content, .quote-test-root, #root {
    background: transparent !important;
  }
}

// --- End of added styles ---



============================================================
FILE: src/pages/QuoteTest/QuoteTest.tsx
============================================================
import React, { useRef, useEffect, useState } from 'react';
import { Box, Container, Typography, Button, TextField, Select, MenuItem, FormControl, InputLabel } from '@mui/material';
import Layout from '../../components/Layout/Layout';
import { useQuote } from '../../contexts/QuoteContext';
import './QuoteTest.scss';
import logo from '../../logo.png';
import { useLocation, useNavigate } from 'react-router-dom';
import html2pdf from 'html2pdf.js';
import {
  Add as AddIcon,
  Delete as DeleteIcon,
  Search as SearchIcon
} from '@mui/icons-material';
import { SupplyItem, LaborItem } from '../../models/Quote';
import { generateId } from '../../utils/id-generator';
import logo512 from '../../assets/logo512.png';
import CHANitec from '../../assets/CHANitec.png';

// Add a helper function for date formatting at the top level
function formatDate(dateString: string) {
  if (!dateString) return '';
  const d = new Date(dateString);
  const day = String(d.getDate()).padStart(2, '0');
  const month = String(d.getMonth() + 1).padStart(2, '0');
  const year = d.getFullYear();
  return `${day}/${month}/${year}`;
}

interface QuoteTestProps {
  currentPath: string;
  onNavigate: (path: string) => void;
}

const QuoteTest: React.FC<QuoteTestProps> = ({ currentPath, onNavigate }) => {
  const {
    state,
    createNewQuote,
    saveQuote,
    updateQuote,
    setQuoteField,
    addSupplyItem,
    removeSupplyItem,
    addLaborItem,
    removeLaborItem,
    recalculateTotals,
    clearQuote,
    loadQuote,
    updateLaborItem
  } = useQuote();

  const { currentQuote, isLoading, isExistingQuote, originalQuoteId } = state;
  const contentRef = useRef<HTMLDivElement>(null);
  const location = useLocation();
  const navigate = useNavigate();
  const quoteId = new URLSearchParams(location.search).get('id');

  const [isFromHistory, setIsFromHistory] = useState(false);
  const [isConfirmed, setIsConfirmed] = useState(false);
  const [isReady, setIsReady] = useState(false);
  const [isReadOnly, setIsReadOnly] = useState(false);
  const [isPdfMode, setIsPdfMode] = useState(false);

  // Load quote if ID is provided in URL
  useEffect(() => {
    const loadQuoteData = async () => {
      if (!isLoading && quoteId && (!currentQuote || currentQuote.id !== quoteId)) {
        try {
          console.log('Attempting to load quote with ID:', quoteId);
          // Fetch createdAt for the quoteId
          let createdAt = '';
          if (currentQuote && currentQuote.id === quoteId) {
            createdAt = currentQuote.createdAt;
          } else {
            try {
              const allQuotes = await fetch(`${process.env.REACT_APP_API_URL}/quotes`).then(res => res.json());
              const found = allQuotes.find((q: any) => q.id === quoteId);
              if (found) {
                createdAt = found.createdAt;
              }
            } catch (e) {}
          }
          await loadQuote(quoteId, createdAt);
          console.log('Quote loaded successfully');
          setIsReady(true);
        } catch (error) {
          console.error('Error loading quote:', error);
          navigate('/');
        }
      } else if (!currentQuote && !isLoading && !quoteId) {
        console.log('No quote ID provided, creating new quote');
        createNewQuote();
        setIsReady(true);
      }
    };

    loadQuoteData();
  }, [quoteId]);

  useEffect(() => {
    const queryParams = new URLSearchParams(window.location.search);
    const fromHistory = queryParams.get('fromHistory') === 'true';
    const confirmed = queryParams.get('confirmed') === 'true';
    setIsFromHistory(fromHistory);
    setIsConfirmed(confirmed);
  }, []);

  // Update isReadOnly when currentQuote changes
  useEffect(() => {
    if (currentQuote) {
      setIsReadOnly(currentQuote.confirmed || false);
      setIsConfirmed(currentQuote.confirmed || false);
    }
  }, [currentQuote]);

  // Handle navigation from other tabs
  useEffect(() => {
    const handleStorageChange = (e: StorageEvent) => {
      if (e.key === 'quoteToLoad' && e.newValue) {
        const quoteToLoad = JSON.parse(e.newValue);
        if (quoteToLoad.id) {
          navigate(`/quote-test?id=${quoteToLoad.id}`);
        }
      }
    };

    window.addEventListener('storage', handleStorageChange);
    return () => window.removeEventListener('storage', handleStorageChange);
  }, [navigate]);

  const handleHomeClick = () => {
    clearQuote();
    createNewQuote();
    onNavigate('/');
  };

  const handleConfirmQuote = async () => {
    if (!currentQuote) return;

    try {
      const response = await fetch(`${process.env.REACT_APP_API_URL}/quotes/${currentQuote.id}/confirm`, {
        method: 'PATCH',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ confirmed: true }),
      });

      if (!response.ok) {
        throw new Error('Failed to confirm quote');
      }

      setIsConfirmed(true);
      setIsReadOnly(true);
      alert('Devis confirmé avec succès. Prêt pour un nouveau devis.');
      clearQuote();
      createNewQuote();
    } catch (error) {
      console.error('Error confirming quote:', error);
      alert('Erreur lors de la confirmation du devis');
    }
  };

  const handleGeneratePDF = async () => {
    let timeoutId: NodeJS.Timeout | null = null;
    try {
      setIsPdfMode(true);
      // Fallback: in case PDF generation hangs, reset after 5 seconds
      timeoutId = setTimeout(() => {
        setIsPdfMode(false);
      }, 5000);
      console.log('Current state:', {
        contentRef: contentRef.current,
        currentQuote: currentQuote,
        isLoading: isLoading
      });

      if (!contentRef.current) {
        console.error('Content reference is missing');
        alert('Impossible de générer le PDF: la référence au contenu est manquante');
        setIsPdfMode(false);
        if (timeoutId) clearTimeout(timeoutId);
        return;
      }

      if (!currentQuote) {
        console.error('Current quote is missing');
        alert('Impossible de générer le PDF: le devis actuel est manquant');
        setIsPdfMode(false);
        if (timeoutId) clearTimeout(timeoutId);
        return;
      }

      if (isLoading) {
        console.error('Quote is still loading');
        alert('Veuillez patienter pendant le chargement du devis');
        setIsPdfMode(false);
        if (timeoutId) clearTimeout(timeoutId);
        return;
      }

      console.log('Starting PDF generation...');
      const element = contentRef.current;

      const opt = {
        margin: 5, // Set margin to 0 for closer print match
        filename: `devis-${currentQuote.id}.pdf`,
        image: { type: 'jpeg', quality: 0.98 },
        html2canvas: {
          scale: 1.25, // Adjust scale for closer print match
          useCORS: true,
          logging: true
        },
        enableLinks: true,
        pagebreak: { mode: ['avoid-all', 'css', 'legacy'] }
      };

      // Generate PDF with save dialog
      await html2pdf()
        .from(element)
        .set(opt)
        .save();

      console.log('PDF generation completed');
    } catch (error) {
      console.error('Error generating PDF:', error);
      alert('Erreur lors de la génération du PDF. Veuillez réessayer.');
    } finally {
      setIsPdfMode(false);
      if (timeoutId) clearTimeout(timeoutId);
    }
  };

  const handlePrint = () => {
    setIsPdfMode(true);
    window.print();

    // Reset PDF mode after print dialog closes
    const handleAfterPrint = () => {
      setIsPdfMode(false);
      window.removeEventListener('afterprint', handleAfterPrint);
      if (timeoutId) clearTimeout(timeoutId);
    };

    window.addEventListener('afterprint', handleAfterPrint);

    // Fallback: in case afterprint is not fired, reset after 5 seconds
    const timeoutId = setTimeout(() => {
      setIsPdfMode(false);
      window.removeEventListener('afterprint', handleAfterPrint);
    }, 5000);
  };

  // Show loading or error state
  if (isLoading) {
    return (
      <Layout currentPath={currentPath} onNavigate={onNavigate} onHomeClick={handleHomeClick}>
        <Box className="loading-container">
          <Typography variant="h6">Chargement en cours...</Typography>
        </Box>
      </Layout>
    );
  }

  if (state.error) {
    return (
      <Layout currentPath={currentPath} onNavigate={onNavigate} onHomeClick={handleHomeClick}>
        <Box className="error-container">
          <Typography variant="h6" color="error">
            Erreur: {state.error}
          </Typography>
          <Button
            variant="contained"
            color="primary"
            onClick={() => navigate('/')}
            style={{ marginTop: '20px' }}
          >
            Retour à l'accueil
          </Button>
        </Box>
      </Layout>
    );
  }

  if (!currentQuote) {
    return (
      <Layout currentPath={currentPath} onNavigate={onNavigate} onHomeClick={handleHomeClick}>
        <Box className="error-container">
          <Typography variant="h6">
            Aucun devis trouvé
          </Typography>
          <Button
            variant="contained"
            color="primary"
            onClick={() => navigate('/')}
            style={{ marginTop: '20px' }}
          >
            Retour à l'accueil
          </Button>
        </Box>
      </Layout>
    );
  }

  return (
    <Layout currentPath={currentPath} onNavigate={onNavigate} onHomeClick={handleHomeClick}>
      <div ref={contentRef} className={isPdfMode ? 'is-pdf-mode' : ''}>
        {/* Background Logo */}
        <img src={logo512} alt="Background Logo" className="background-logo" />
        {/* Second Background Logo */}
        <img src={CHANitec} alt="CHANitec Logo" className="background-logo-second" />

        {/* Header Section */}
        <div className="reference-header">
          <img src={logo} alt="Logo" className="reference-logo" />
          <div className="reference-title">CALCUL DE PRIX OFFRE CLIMATISATION</div>
        </div>

        {/* Client Info Section */}
        <div className="client-info-box">
          <div className="client-info-grid">
            <div className="client-info-label">CLIENT:</div>
            <div className="client-info-value">
              <input
                type="text"
                value={currentQuote.clientName}
                onChange={e => setQuoteField('clientName', e.target.value)}
                disabled={isReadOnly}
              />
            </div>
            <div className="client-info-label">SITE:</div>
            <div className="client-info-value">
              <input
                type="text"
                value={currentQuote.siteName}
                onChange={e => setQuoteField('siteName', e.target.value)}
                disabled={isReadOnly}
              />
            </div>
            <div className="client-info-label">OBJET:</div>
            <div className="client-info-value">
              <input
                type="text"
                value={currentQuote.object}
                onChange={e => setQuoteField('object', e.target.value)}
                disabled={isReadOnly}
              />
            </div>
            <div className="client-info-label">DATE:</div>
            <div className="client-info-value">
              {isPdfMode ? (
                <input
                  type="text"
                  value={formatDate(currentQuote.date)}
                  disabled
                />
              ) : (
                <input
                  type="date"
                  value={currentQuote.date}
                  onChange={e => setQuoteField('date', e.target.value)}
                  disabled={isReadOnly}
                />
              )}
            </div>
          </div>
        </div>

        {/* Totals Section */}
        <div className="clearfix">
          <table className="summary-table" style={{ float: 'right' }}>
            <tbody>
              <tr><th>TOTAL OFFRE USD HT:</th><td>{currentQuote.totalHT}</td></tr>
              <tr><th>TVA:</th><td>{(currentQuote.totalHT * (16 / 100)).toFixed(2)}</td></tr>
              <tr><th>TOTAL OFFRE USD TTC:</th><td>{currentQuote.totalTTC}</td></tr>
            </tbody>
          </table>
        </div>

        {/* Fournitures Section */}
        <div className="section-title">FOURNITURES</div>
        <div className="input-row">
        <span> {currentQuote.supplyDescription || ''} </span>
          <div className='tx-row'>
          <label>Tx de chg:</label>
          <span> {currentQuote.supplyExchangeRate || 1.15}</span>
          <label>Tx de marge:</label>
          <span>{currentQuote.supplyMarginRate || 0.75}</span>
        </div>

        </div>
        <table className="data-table">
          <thead>
            <tr>
              <th>Description</th>
              <th>Qté</th>
              <th>PR €</th>
              <th>PR $</th>
              <th>PV/u $</th>
              <th>PV $ Total HT</th>
            </tr>
          </thead>
          <tbody>
            {currentQuote.supplyItems.map((item, idx) => (
              <tr key={idx}>
                <td>{item.description}</td>
                <td>{item.quantity} </td>
                <td>{item.priceEuro}</td>
                <td>{(item.priceEuro * (currentQuote.supplyExchangeRate || 1.15)).toFixed(2)}</td>
                <td>{((item.priceEuro * (currentQuote.supplyExchangeRate || 1.15)) * (1 / (currentQuote.supplyMarginRate || 0.75))).toFixed(2)}</td>
                <td>{((item.quantity * item.priceEuro * (currentQuote.supplyExchangeRate || 1.15)) * (1 / (currentQuote.supplyMarginRate || 0.75))).toFixed(2)} </td>
              </tr>
            ))}
          </tbody>
          <tfoot>
            <tr className="totals-row">
              <td colSpan={5} style={{ textAlign: 'right' }}>TOTAL FOURNITURE $ HT:</td>
              <td colSpan={2}>{currentQuote.supplyItems.reduce((sum, item) => sum + ((item.quantity * item.priceEuro * (currentQuote.supplyExchangeRate || 1.15)) * (1 / (currentQuote.supplyMarginRate || 0.75))), 0).toFixed(2)}</td>
            </tr>
          </tfoot>
        </table>

        {/* Main d'oeuvre Section */}
        <div className="section-title">MAIN D'OEUVRE</div>
        <div className="input-row">
          <span>{currentQuote.laborDescription || ''}</span>
          <div className='tx-row'>
          <label>Tx de chg:</label>
          <span>{currentQuote.laborExchangeRate || 1.2}</span>
          <label>Tx de marge:</label>
          <span>{currentQuote.laborMarginRate || 0.8}</span>
          </div>
        </div>


        <table className="data-table">
          <thead>
            <tr>
              <th>Nb technicien</th>
              <th>Nb heures</th>
              <th>Majo Weekend</th>
              <th>PR €</th>
              <th>PR $</th>
              <th>PV/u $</th>
              <th>PV $ Total HT</th>
            </tr>
          </thead>
          <tbody>
            {currentQuote.laborItems.map((item, idx) => (
              <tr key={idx}>
                <td>{item.nbTechnicians}</td>
                <td>{item.nbHours}</td>
                <td>{item.weekendMultiplier}</td>
                <td>{item.priceEuro}</td>
                <td>{(item.priceEuro * (currentQuote.laborExchangeRate || 1.2)).toFixed(2)}</td>
                <td>{((item.priceEuro * (currentQuote.laborExchangeRate || 1.2)) * (1 / (currentQuote.laborMarginRate || 0.8))).toFixed(2)}</td>
                <td>{((item.nbTechnicians * item.nbHours * item.priceEuro * (currentQuote.laborExchangeRate || 1.2)) * (1 / (currentQuote.laborMarginRate || 0.8))).toFixed(2)}</td>
              </tr>
            ))}
          </tbody>
          <tfoot>
            <tr className="totals-row">
              <td colSpan={6} style={{ textAlign: 'right' }}>TOTAL MO $ HT:</td>
              <td colSpan={2}>{currentQuote.laborItems.reduce((sum, item) => sum + ((item.nbTechnicians * item.nbHours * item.priceEuro * (currentQuote.laborExchangeRate || 1.2)) * (1 / (currentQuote.laborMarginRate || 0.8))), 0).toFixed(2)}</td>
            </tr>
          </tfoot>
        </table>

        {/* Action Buttons */}
        {!isPdfMode && (
          <div style={{ display: 'flex', justifyContent: 'flex-end', gap: 10, margin: '30px 0' }}>
            <button className="btn-save" onClick={handlePrint}> {'Imprimer'}</button>
          </div>
        )}
      </div>
    </Layout>
  );
};

export default QuoteTest;

============================================================
FILE: src/pages/employeesPage.scss
============================================================
.employees-container {
  min-height: 100vh;
  background-color: #f5f5f5;

  .MuiTableContainer-root {
    margin-top: 20px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  .MuiTableCell-root {
    padding: 16px;
  }

  .MuiIconButton-root {
    margin: 0 4px;
  }

  .MuiDialog-paper {
    min-width: 400px;
  }

  .MuiTextField-root {
    margin-bottom: 16px;
  }
}

============================================================
FILE: src/pages/employeesPage.tsx
============================================================
import React, { useState, useEffect } from 'react';
import {
  AppBar,
  Toolbar,
  Button,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  Paper,
  IconButton,
  Dialog,
  DialogTitle,
  DialogContent,
  DialogActions,
  TextField,
  Box,
  Typography,
  MenuItem,
  Select,
  FormControl,
  InputLabel,
  SelectChangeEvent,
  CircularProgress,
} from '@mui/material';
import { Edit as EditIcon, Delete as DeleteIcon, Add as AddIcon } from '@mui/icons-material';
import { useNavigate } from 'react-router-dom';
import { employeeService, Employee, CreateEmployeeDTO } from '../services/employee-service';
import './employeesPage.scss';
import dayjs from 'dayjs';

const civilStatusOptions = ['C', 'M']; // Celibataire, Marié
const contractTypeOptions = ['CDI', 'CDD', 'Interim'];

const EmployeesPage = () => {
  const navigate = useNavigate();
  const [employees, setEmployees] = useState<Employee[]>([]);
  const [openDialog, setOpenDialog] = useState(false);
  const [editingEmployee, setEditingEmployee] = useState<Employee | null>(null);
  const [formData, setFormData] = useState<CreateEmployeeDTO>({
    full_name: '',
    civil_status: '',
    birth_date: '',
    entry_date: '',
    seniority: '',
    contract_type: '',
    job_title: '',
    fonction: '',
    sub_type_id: undefined,
    type_description: '',
  });
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    loadEmployees();
  }, []);

  const loadEmployees = async () => {
    try {
      setLoading(true);
      const data = await employeeService.getAllEmployees();
      setEmployees(data);
      setError(null);
    } catch (err) {
      setError('Failed to load employees');
      console.error('Error loading employees:', err);
    } finally {
      setLoading(false);
    }
  };

  const handleOpenDialog = (employee?: Employee) => {
    if (employee) {
      setEditingEmployee(employee);
      setFormData({
        full_name: employee.full_name,
        civil_status: employee.civil_status,
        birth_date: employee.birth_date,
        entry_date: employee.entry_date,
        seniority: employee.seniority,
        contract_type: employee.contract_type,
        job_title: employee.job_title,
        fonction: employee.fonction,
        sub_type_id: employee.sub_type_id,
        type_description: employee.type_description,
      });
    } else {
      setEditingEmployee(null);
      setFormData({
        full_name: '',
        civil_status: '',
        birth_date: '',
        entry_date: '',
        seniority: '',
        contract_type: '',
        job_title: '',
        fonction: '',
        sub_type_id: undefined,
        type_description: '',
      });
    }
    setOpenDialog(true);
  };

  const handleCloseDialog = () => {
    setOpenDialog(false);
    setEditingEmployee(null);
    setFormData({
      full_name: '',
      civil_status: '',
      birth_date: '',
      entry_date: '',
      seniority: '',
      contract_type: '',
      job_title: '',
      fonction: '',
      sub_type_id: undefined,
      type_description: '',
    });
  };

  const handleTextChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value,
    }));
  };

  const handleSelectChange = (e: SelectChangeEvent<string>) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value,
    }));
  };

  const handleSubmit = async () => {
    try {
      if (editingEmployee) {
        await employeeService.updateEmployee(editingEmployee.id, formData);
      } else {
        await employeeService.createEmployee(formData);
      }
      await loadEmployees();
      handleCloseDialog();
    } catch (err) {
      console.error('Error saving employee:', err);
      setError('Failed to save employee');
    }
  };

  const handleDelete = async (id: number) => {
    if (window.confirm('Are you sure you want to delete this employee?')) {
      try {
        await employeeService.deleteEmployee(id);
        await loadEmployees();
      } catch (err) {
        console.error('Error deleting employee:', err);
        setError('Failed to delete employee');
      }
    }
  };

  const calculateSeniority = (entryDate: string) => {
    if (!entryDate) return '';
    const start = dayjs(entryDate);
    const end = dayjs();
    const years = end.diff(start, 'year');
    const months = end.diff(start.add(years, 'year'), 'month');
    const days = end.diff(start.add(years, 'year').add(months, 'month'), 'day');
    return `${years} years ${months} months ${days} days`;
  };

  const getTypeDescription = (subTypeId: number | undefined) => {
    switch (subTypeId) {
      case 1:
        return 'Chef de service Chargé de clim-domestique';
      case 2:
        return 'Polyvalent';
      case 3:
      case 4:
      case 5:
        return 'Chef de service adj chargé du climatisation centralisé';
      default:
        return '';
    }
  };

  if (loading) {
    return (
      <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '100vh' }}>
        <CircularProgress />
      </Box>
    );
  }

  if (error) {
    return (
      <Box sx={{ p: 3, color: 'error.main' }}>
        Error: {error}
      </Box>
    );
  }

  return (
    <div className="employees-container">
      <AppBar position="static" color="primary">
        <Toolbar>
          <Button color="inherit" onClick={() => navigate('/org-chart')}>
            Back to Org Chart
          </Button>
          <Typography variant="h6" style={{ marginLeft: '20px' }}>
            Employees Management
          </Typography>
        </Toolbar>
      </AppBar>

      <Box sx={{ p: 3 }}>
        <Button
          variant="contained"
          color="primary"
          startIcon={<AddIcon />}
          onClick={() => handleOpenDialog()}
          sx={{ mb: 2 }}
        >
          Add Employee
        </Button>

        <TableContainer component={Paper}>
          <Table>
            <TableHead>
              <TableRow>
                <TableCell>Full Name</TableCell>
                <TableCell>Civil Status</TableCell>
                <TableCell>Job Title</TableCell>
                <TableCell>Fonction</TableCell>
                <TableCell>Contract Type</TableCell>
                <TableCell>Seniority</TableCell>
                <TableCell>Actions</TableCell>
              </TableRow>
            </TableHead>
            <TableBody>
              {employees.map((employee) => (
                <TableRow key={employee.id}>
                  <TableCell>{employee.full_name}</TableCell>
                  <TableCell>{employee.civil_status}</TableCell>
                  <TableCell>{employee.job_title}</TableCell>
                  <TableCell>{employee.fonction}</TableCell>
                  <TableCell>{employee.contract_type}</TableCell>
                  <TableCell>{employee.seniority}</TableCell>
                  <TableCell>
                    <IconButton onClick={() => handleOpenDialog(employee)}>
                      <EditIcon />
                    </IconButton>
                    <IconButton onClick={() => handleDelete(employee.id)}>
                      <DeleteIcon />
                    </IconButton>
                  </TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </TableContainer>
      </Box>

      <Dialog open={openDialog} onClose={handleCloseDialog} maxWidth="md" fullWidth>
        <DialogTitle>
          {editingEmployee ? 'Edit Employee' : 'Add New Employee'}
        </DialogTitle>
        <DialogContent>
          <Box sx={{ pt: 2, display: 'flex', flexDirection: 'column', gap: 2 }}>
            <TextField
              name="full_name"
              label="Full Name"
              value={formData.full_name}
              onChange={handleTextChange}
              fullWidth
              required
            />
            <FormControl fullWidth required>
              <InputLabel>Civil Status</InputLabel>
              <Select
                name="civil_status"
                value={formData.civil_status}
                onChange={handleSelectChange}
                label="Civil Status"
              >
                <MenuItem value="C">C</MenuItem>
                <MenuItem value="M">M</MenuItem>
              </Select>
            </FormControl>
            <TextField
              name="birth_date"
              label="Birth Date"
              type="date"
              value={formData.birth_date}
              onChange={handleTextChange}
              fullWidth
              required
              InputLabelProps={{ shrink: true }}
            />
            <TextField
              name="entry_date"
              label="Entry Date"
              type="date"
              value={formData.entry_date}
              onChange={handleTextChange}
              fullWidth
              required
              InputLabelProps={{ shrink: true }}
            />
            <TextField
              name="seniority"
              label="Seniority"
              value={calculateSeniority(formData.entry_date)}
              fullWidth
              disabled
            />
            <FormControl fullWidth required>
              <InputLabel>Contract Type</InputLabel>
              <Select
                name="contract_type"
                value={formData.contract_type}
                onChange={handleSelectChange}
                label="Contract Type"
              >
                {contractTypeOptions.map((type) => (
                  <MenuItem key={type} value={type}>
                    {type}
                  </MenuItem>
                ))}
              </Select>
            </FormControl>
            <TextField
              name="job_title"
              label="Job Title"
              value={formData.job_title}
              onChange={handleTextChange}
              fullWidth
              required
            />
            <TextField
              name="fonction"
              label="Fonction"
              value={formData.fonction}
              onChange={handleTextChange}
              fullWidth
              required
            />
            <FormControl fullWidth required>
              <InputLabel>Sub Type</InputLabel>
              <Select
                name="sub_type_id"
                value={formData.sub_type_id !== undefined ? String(formData.sub_type_id) : ''}
                onChange={(e) => {
                  setFormData(prev => ({
                    ...prev,
                    sub_type_id: e.target.value === '' ? undefined : Number(e.target.value),
                  }));
                }}
                label="Sub Type"
              >
                <MenuItem value="1">UTEX</MenuItem>
                <MenuItem value="2">POLIVALONT</MenuItem>
                <MenuItem value="3">PULLMAN</MenuItem>
                <MenuItem value="4">SNEL</MenuItem>
                <MenuItem value="5">BCDC</MenuItem>
                <MenuItem value="6">AUCUN</MenuItem>
              </Select>
            </FormControl>
            <TextField
              name="type_description"
              label="Type Description"
              value={getTypeDescription(formData.sub_type_id)}
              fullWidth
              disabled
            />
          </Box>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleCloseDialog}>Cancel</Button>
          <Button onClick={handleSubmit} variant="contained" color="primary">
            {editingEmployee ? 'Update' : 'Add'}
          </Button>
        </DialogActions>
      </Dialog>
    </div>
  );
};

export default EmployeesPage;

============================================================
FILE: src/pages/interventionPage.scss
============================================================
.intervention-a4-container {
  height: 60%;
  width: 80%;
  background: #fff;
  font-family: 'Arial', 'Helvetica Neue', Arial, sans-serif;
  box-sizing: border-box;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  color: #000;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%) scale(.8);
  box-shadow: 0 0 10px rgba(0,0,0,0.1);
}

body {
  margin: 0;
  padding: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #f0f0f0;
}

.intervention-header-logos {
  display: grid;
  grid-template-columns: 1.2fr 1fr 1.2fr;
  align-items: center;
  width: 100%;
  padding: 0.8em 1.5em 0.3em 1.5em;
  background: #fff;
}

.intervention-header-logos img {
  max-width: 100%;
  height: auto;
  display: block;
}

.intervention-header-logos img:first-child {
  justify-self: start;
  max-height: 3.5em;
}

.intervention-header-logos .center-logo {
  display: flex;
  justify-content: center;
}

.intervention-header-logos .center-logo img {
  max-height: 6em;
}

.intervention-header-logos img:last-child {
  justify-self: end;
  max-height: 3.5em;
}

.title-row-table {
  display: grid;
  grid-template-columns: 60% 20% 7% 7% 6%;
  border: 2px solid #222;
  border-radius: 0.2em;
  margin-bottom: 0;
  background: #fff;
  width: 100%;
  grid-template-rows: 3em , 3em;
}

.title-row-cell {
  background: #fff;
  color: #222;
  font-size: 1.08em;
  padding: 1.1em 0.7em;
  border-right: 1px solid #222;
  border-bottom: none;
  display: flex;
  align-items: center;
}

.title-row-cell:last-child {
  border-right: none;
}

.title-row-main {
  justify-content: flex-start;
}

.title-row-date {
  justify-content: flex-end;
}

.title-row-center {
  justify-content: center;
}

.info-row-table {
  display: grid;
  grid-template-columns: 22% repeat(6, 1fr);
  grid-template-rows: 3em 3em;
  border: 2px solid #222;
  border-radius: 0.2em;
  margin-bottom: 0;
  background: #fff;
  width: 100%;
}

.info-row-address {
  grid-row: 1 / span 2;
  grid-column: 1 / 2;
  background: #fff;
  color: #222;
  font-size: 1em;
  padding: 1em 0.7em;
  border-right: 1px solid #222;
  border-bottom: none;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}

.info-row-header {
  background: #bcbcbc;
  color: #222;
  font-weight: bold;
  text-align: center;
  padding: 1em 0.2em;
  border-bottom: 2px solid #222;
  border-right: 1px solid #222;
  font-size: 1.08em;
  display: flex;
  align-items: center;
  justify-content: center;
}

.info-row-header:last-child {
  border-right: none;
}

.info-row-cell {
  background: #fff;
  border-right: 1px solid #222;
  border-bottom: none;
  min-height: 3em;
}

.info-row-cell:last-child {
  border-right: none;
}

.intervention-main-table {
  display: grid;
  grid-template-columns: 25% 25% 50%;
  border-bottom: none;
  border-top: 0.1em solid #000;
  font-size: 0.97em;
  margin-bottom: 0;
  border-radius: 0 0 0 0;
}

.intervention-main-table > div {
  border-right: 0.08em solid #000;
}

.intervention-main-table > div:last-child {
  border-right: none;
}

.cell-orange { background: #ff9900; color: #fff; }
.cell-green { background: #00ff00; color: #000; }
.cell-white { background: #fff; color: #000; }
.cell-border { border: 1.5px solid #000; }

.intervention-section {
  padding: 0.3em 0.4em;
  border-bottom: 0.08em solid #bbb;
}

.intervention-section-title {
  font-weight: bold;
  background: #eaeaea;
  padding: 0.3em 0.5em;
  margin-bottom: 0.2em;
  letter-spacing: 0.03em;
}

.intervention-checkbox-row {
  display: flex;
  align-items: flex-start;
  gap: 0.5em;
  margin-bottom: 0.1em;
}

.intervention-checkbox-row label {
  display: flex;
  align-items: center;
  font-weight: normal;
  gap: 0.4em;
}

.intervention-checkbox-row input[type='checkbox'] {
  width: 1.1em;
  height: 1.1em;
  margin-right: 0.3em;
}

.intervention-measure-table > div,
.essais-electrique-table > div,
.mesure-essais-combined-table > div {
  padding: 0.2em 0.3em;
  min-height: 1.6em;
  margin: 0;
}

.compte-observations-cell,
.signatures-cell {
  min-height: 2.5em;
}

.compte-observations-textarea,
.signatures-input {
  width: 100%;
  border: none;
  font-size: 1em;
  background: transparent;
  outline: none;
}

.compte-observations-textarea {
  min-height: 4em;
  resize: vertical;
}

@media print {
  .intervention-a4-container {
    position: static;
    transform: none;
    box-shadow: none;
    width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  body {
    background: #fff;
    display: block;
    margin: 0;
    padding: 0;
    width: 297mm;
    height: 210mm;
  }

  @page {
    size: landscape;
    margin: 0;
  }

  * {
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
  }

  .action-buttons {
    display: none;
  }
}

// Main Table Styles
.unite-coffret-combined-table {
  display: grid;
  grid-template-columns: 2.5fr .25fr;
  border: 2px solid #222;
  border-radius: 0.2em;
  margin-bottom: 0.5em;
  background: #fff;
  font-size: 1em;
  margin: 0;
}

.unite-coffret-header-main,
.unite-coffret-header-section {
  grid-column: 1 / span 2;
  background: #bcbcbc;
  color: #222;
  font-weight: bold;
  text-align: center;
  padding: 0.5em 0;
  border-bottom: 1px solid #222;
  font-size: 1.08em;
  display: flex;
  align-items: center;
  justify-content: center;
}
.unite-coffret-header-section {
  background: #eaeaea;
  border-top: 2px solid #222;
  border-bottom: 1px solid #222;
}

.unite-coffret-label {
  padding: 0.3em 0.5em;
  border-bottom: 1px solid #222;
  border-right: 1px solid #222;
  display: flex;
  align-items: center;
  min-height: 2em;
  background: #fff;
}
.unite-coffret-cell {
  padding: 0.3em 0.5em;
  border-bottom: 1px solid #222;
  min-height: 2em;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
}

// Remove bottom border for last row
.unite-coffret-combined-table > .unite-coffret-label:last-of-type,
.unite-coffret-combined-table > .unite-coffret-cell:last-child {
  border-bottom: none;
}

// Interior and Connections Table
.interieure-liaisons-combined-table {
  display: grid;
  grid-template-columns: 2.5fr .25fr;
  border: 2px solid #222;
  border-radius: 0.2em;
  margin-bottom: 0.5em;
  background: #fff;
  font-size: 1em;
  margin: 0;
}

.interieure-liaisons-header-main,
.interieure-liaisons-header-section {
  grid-column: 1 / span 2;
  background: #bcbcbc;
  color: #222;
  font-weight: bold;
  text-align: center;
  padding: 0.5em 0;
  border-bottom: 1px solid #222;
  font-size: 1.08em;
  display: flex;
  align-items: center;
  justify-content: center;
}
.interieure-liaisons-header-section {
  background: #eaeaea;
  border-top: 2px solid #222;
  border-bottom: 1px solid #222;
}

.interieure-liaisons-label {
  padding: 0.3em 0.5em;
  border-bottom: 1px solid #222;
  border-right: 1px solid #222;
  display: flex;
  align-items: center;
  min-height: 2em;
  background: #fff;
}
.interieure-liaisons-cell {
  padding: 0.3em 0.5em;
  border-bottom: 1px solid #222;
  min-height: 2em;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
}

// Remove bottom border for last row
.interieure-liaisons-combined-table > .interieure-liaisons-label:last-of-type,
.interieure-liaisons-combined-table > .interieure-liaisons-cell:last-child {
  border-bottom: none;
}

// Measurements and Tests Table
.mesure-essais-combined-table {
  display: grid;
  grid-template-columns: 2.5fr 1fr 1fr 1fr 1fr;
  border: 2px solid #222;
  border-radius: 0.2em;
  margin-bottom: 0.5em;
  background: #fff;
  width: 100%;
  font-size: 1em;
  margin: 0;
}

.mesure-essais-header-main {
  grid-column: 1 / span 5;
  background: #bcbcbc;
  color: #222;
  font-weight: bold;
  text-align: center;
  padding: 0.5em 0;
  border-bottom: 1px solid #222;
  font-size: 1.08em;
  display: flex;
  align-items: center;
  justify-content: center;
}

.mesure-essais-header-clim {
  background: #bcbcbc;
  color: #222;
  font-weight: bold;
  text-align: center;
  padding: 0.5em 0;
  border-bottom: 1px solid #222;
  border-right: 1px solid #222;
  display: flex;
  align-items: center;
  justify-content: center;
}
.mesure-essais-header-clim:last-child {
  border-right: none;
}

.mesure-essais-label {
  padding: 0.3em 0.5em;
  border-bottom: 1px solid #222;
  border-right: 1px solid #222;
  display: flex;
  align-items: center;
  min-height: 2em;
  background: #fff;
}
.mesure-essais-cell {
  padding: 0.3em 0.5em;
  border-bottom: 1px solid #222;
  border-right: 1px solid #222;
  min-height: 2em;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
}
.mesure-essais-cell:last-child {
  border-right: none;
}

.mesure-essais-header-section {
  grid-column: 1 / span 5;
  background: #eaeaea;
  color: #222;
  font-weight: bold;
  text-align: center;
  padding: 0.5em 0;
  border-top: 2px solid #222;
  border-bottom: 1px solid #222;
  font-size: 1.08em;
  display: flex;
  align-items: center;
  justify-content: center;
}

// Remove bottom border for last row
.mesure-essais-combined-table > .mesure-essais-label:last-of-type,
.mesure-essais-combined-table > .mesure-essais-cell:last-child {
  border-bottom: none;
}

// Observations and Signatures Table
.compte-observations-table {
  display: grid;
  grid-template-columns: 1fr 1fr;
  border: 2px solid #222;
  border-radius: 0.2em;
  margin-bottom: 0.5em;
  background: #fff;
  width: 100%;
}

.compte-observations-header {
  background: #bcbcbc;
  color: #222;
  font-weight: bold;
  text-align: center;
  padding: 0.5em;
  border-bottom: 2px solid #222;
  font-size: 1.08em;
  border-right: 1px solid #222;
}

.compte-observations-header:last-child {
  border-right: none;
}

.compte-observations-cell {
  padding: 0.5em;
  border-right: 1px solid #ddd;
  min-height: 4em;
}

.compte-observations-cell:last-child {
  border-right: none;
}

// Signatures Table
.signatures-table {
  display: grid;
  grid-template-columns: 1fr 1fr;
  border: 2px solid #222;
  border-radius: 0.2em;
  margin-bottom: 0.5em;
  background: #fff;
  width: 100%;
}

.signatures-header {
  background: #bcbcbc;
  color: #222;
  font-weight: bold;
  text-align: center;
  padding: 0.5em;
  border-bottom: 2px solid #222;
  font-size: 1.08em;
  border-right: 1px solid #222;
}

.signatures-header:last-child {
  border-right: none;
}

.signatures-cell {
  padding: 0.5em;
  border-right: 1px solid #ddd;
  min-height: 2.5em;
}

.signatures-cell:last-child {
  border-right: none;
}

.interieure-liaisons-combined-table input[type='checkbox'],
.unite-coffret-combined-table input[type='checkbox'] {
  width: 2.2em;
  height: 2.2em;
}

.intervention-navbar {
  background-color: #1976d2 !important;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1) !important;
  margin-bottom: 2rem;

  .nav-buttons {
    display: flex;
    gap: 1rem;
    width: 100%;
    justify-content: flex-start;
  }

  .nav-button {
    background-color: transparent !important;
    color: white !important;
    text-transform: none !important;
    font-weight: 500 !important;
    padding: 0.5rem 1.5rem !important;
    border-radius: 4px !important;
    box-shadow: none !important;
    border: 1px solid rgba(255, 255, 255, 0.3) !important;

    &:hover {
      background-color: rgba(255, 255, 255, 0.1) !important;
    }
  }
}

============================================================
FILE: src/pages/interventionPage.tsx
============================================================
import React, { useRef } from 'react';
import { useNavigate } from 'react-router-dom';
import html2canvas from 'html2canvas';
import jsPDF from 'jspdf';
import './interventionPage.scss';
import { Button, Box, AppBar, Toolbar } from '@mui/material';
import PrintIcon from '@mui/icons-material/Print';
import DownloadIcon from '@mui/icons-material/Download';
import ArrowBackIcon from '@mui/icons-material/ArrowBack';

// Correct logo imports for React (public folder)
const logoChanic = process.env.PUBLIC_URL + '/chanic.png';
const logoChanitec = process.env.PUBLIC_URL + '/chanitec.png';
const logoTrane = process.env.PUBLIC_URL + '/trane.png';

export default function InterventionPage() {
  const navigate = useNavigate();
  const interventionRef = useRef<HTMLDivElement>(null);

  const handlePrint = () => {
    window.print();
  };

  const handleDownloadPDF = async () => {
    if (!interventionRef.current) return;

    try {
      const canvas = await html2canvas(interventionRef.current, {
        scale: 2,
        useCORS: true,
        logging: false,
        allowTaint: true,
        width: interventionRef.current.offsetWidth,
        height: interventionRef.current.offsetHeight,
        windowWidth: interventionRef.current.offsetWidth,
        windowHeight: interventionRef.current.offsetHeight
      });

      const imgData = canvas.toDataURL('image/png');
      const pdf = new jsPDF({
        orientation: 'landscape',
        unit: 'mm',
        format: 'a4'
      });

      // A4 dimensions in mm
      const pageWidth = 297;
      const pageHeight = 210;

      // Calculate scaling to fit the page
      const imgRatio = canvas.width / canvas.height;
      const pageRatio = pageWidth / pageHeight;

      let finalWidth, finalHeight;

      if (imgRatio > pageRatio) {
        // Image is wider than page ratio
        finalWidth = pageWidth;
        finalHeight = pageWidth / imgRatio;
      } else {
        // Image is taller than page ratio
        finalHeight = pageHeight;
        finalWidth = pageHeight * imgRatio;
      }

      // Center the content
      const xOffset = (pageWidth - finalWidth) / 2;
      const yOffset = (pageHeight - finalHeight) / 2;

      pdf.addImage(imgData, 'PNG', xOffset, yOffset, finalWidth, finalHeight);
      pdf.save('fiche-intervention.pdf');
    } catch (error) {
      console.error('Error generating PDF:', error);
    }
  };

  return (
    <Box className="intervention-page">
      <AppBar position="static" color="primary" className="intervention-navbar">
        <Toolbar>
          <Box className="nav-buttons">
            <Button
              variant="contained"
              startIcon={<ArrowBackIcon />}
              onClick={() => navigate('/home')}
              className="nav-button"
            >
              Retour
            </Button>
            <Button
              variant="contained"
              startIcon={<PrintIcon />}
              onClick={handlePrint}
              className="nav-button"
            >
              Imprimer
            </Button>
            <Button
              variant="contained"
              startIcon={<DownloadIcon />}
              onClick={handleDownloadPDF}
              className="nav-button"
            >
              Télécharger PDF
            </Button>
          </Box>
        </Toolbar>
      </AppBar>

      <div className="intervention-a4-container" ref={interventionRef}>
        {/* Header Logos */}
        <div className="intervention-header-logos cell-white">
          <img src={logoChanitec} alt="CHANITEC Logo" />
          <div className="center-logo">
            <img src={logoChanic} alt="CHANIC Logo" />
          </div>
          <img src={logoTrane} alt="TRANE Logo" />
        </div>

        {/* Title Row */}
        <div className="title-row-table">
          <div className="title-row-cell title-row-main"><b>Fiche d'intervention N°</b></div>
          <div className="title-row-cell title-row-date"><b>DATE :</b></div>
          <div className="title-row-cell title-row-center"><b>DÉP</b></div>
          <div className="title-row-cell title-row-center"><b>ENT</b></div>
          <div className="title-row-cell"></div>
        </div>

        {/* Info Row */}
        <div className="info-row-table">
          <div className="info-row-address">Avenue Colonel Mondjiba BP 8512<br />KINSHASA NGALIEMA</div>
          <div className="info-row-header">CENTRE</div>
          <div className="info-row-header">LOCAL</div>
          <div className="info-row-header">N° DE SÉRIE</div>
          <div className="info-row-header">MARQUE</div>
          <div className="info-row-header">P(KW) / BTU</div>
          <div className="info-row-header">CODE</div>
          <div className="info-row-cell"></div>
          <div className="info-row-cell"></div>
          <div className="info-row-cell"></div>
          <div className="info-row-cell"></div>
          <div className="info-row-cell"></div>
          <div className="info-row-cell"></div>
        </div>

        {/* Main Table */}
        <div className="intervention-main-table">
          {/* 1. UNITE EXTERIEURE + 4. COFFRET ELECTRIQUE COMMANDE & PUISSANCE (Combined Table) */}
          <div className="unite-coffret-combined-table">
            <div className="unite-coffret-header unite-coffret-header-main">I. UNITE EXTERIEURE</div>
            <div className="unite-coffret-label">Vérification  de l'absence d'échauffement</div><div className="unite-coffret-cell"><input type="checkbox" /></div>
            <div className="unite-coffret-label">Vérification  de l'absence de vibration</div><div className="unite-coffret-cell"><input type="checkbox" /></div>
            <div className="unite-coffret-label">Dépoussiérage  câblage  électrique</div><div className="unite-coffret-cell"><input type="checkbox" /></div>
            <div className="unite-coffret-label">Serrage des connexions  électriques</div><div className="unite-coffret-cell"><input type="checkbox" /></div>
            <div className="unite-coffret-label">Nettoyage du condenseur  (Eau & Produit détergent)</div><div className="unite-coffret-cell"><input type="checkbox" /></div>
            <div className="unite-coffret-label">Vérification  de l'unité extérieure</div><div className="unite-coffret-cell"><input type="checkbox" /></div>
            <div className="unite-coffret-label">Vérification  fonctionnement du variateur de vitesse</div><div className="unite-coffret-cell"><input type="checkbox" /></div>
            <div className="unite-coffret-header unite-coffret-header-section">4. COFFRET ELECTRIQUE  COMMANDE & PUISSANCE</div>
            <div className="unite-coffret-label">Nettoyage & Dépoussiérage  Coffret électrique</div><div className="unite-coffret-cell"><input type="checkbox" /></div>
            <div className="unite-coffret-label">Serrage des connexions  électriques</div><div className="unite-coffret-cell"><input type="checkbox" /></div>
            <div className="unite-coffret-label">Vérification  Etat des Fusibles Coffret de puissance</div><div className="unite-coffret-cell"><input type="checkbox" /></div>
            <div className="unite-coffret-label">Vérification  Etat des Voyants & Fonctionnement Sirène</div><div className="unite-coffret-cell"><input type="checkbox" /></div>
            <div className="unite-coffret-label">Vérification  fonctionnement minuterie</div><div className="unite-coffret-cell"><input type="checkbox" /></div>
          </div>

          {/* 2. UNITE INTERIEURE + 3. LIAISONS ELECTRIQUES ET FRIGORIFIQUES (Combined Table) */}
          <div className="interieure-liaisons-combined-table">
            <div className="interieure-liaisons-header interieure-liaisons-header-main">2. UNITE INTERIEURE</div>
            <div className="interieure-liaisons-label">Nettoyage de l'évaporateur</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
            <div className="interieure-liaisons-label">Vérification de la fixation de l'unité intérieure</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
            <div className="interieure-liaisons-label">Nettoyage  à l'eau ou  à l'air comprimé  du filtre</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
            <div className="interieure-liaisons-label">Nettoyage du bac condensat</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
            <div className="interieure-liaisons-label">Vérifier la fixation  du circuit de condensat</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
            <div className="interieure-liaisons-label">Vérifier la fixation  du Plenum & grille de soufflage</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
            <div className="interieure-liaisons-label">Serrage des connexions  électriques (comp, MV)</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
            <div className="interieure-liaisons-label">Vérification  de l'absence d'échauffement anormal</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
            <div className="interieure-liaisons-label">Vérification  de l'absence de vibration</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
            <div className="interieure-liaisons-label">Vérification  des paramètres  au niveau</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
            <div className="interieure-liaisons-label">Vérification  de l'état des portes</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
            <div className="interieure-liaisons-label">Changement du filtre à air</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
            <div className="interieure-liaisons-header interieure-liaisons-header-section">3. LIAISONS ELECTRIQUES ET FRIGORIFIQUES</div>
            <div className="interieure-liaisons-label">Vérification  Fixation des circuits Frigorifiques</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
            <div className="interieure-liaisons-label">Vérification  Calorifuge des circuits</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
            <div className="interieure-liaisons-label">Vérification  Fixation des circuits Electriques</div><div className="interieure-liaisons-cell"><input type="checkbox" /></div>
          </div>

          {/* 5. MESURE & RELEVE + 6. ESSAIS ELECTRIQUE & FRIGORIFIQUE (Combined Table) */}
          <div className="mesure-essais-combined-table">
            <div className="mesure-essais-header-clim">5. MESURE & RELEVE</div>
            <div className="mesure-essais-header-clim">CLIM1</div>
            <div className="mesure-essais-header-clim">CLIM2</div>
            <div className="mesure-essais-header-clim">CLIM3</div>
            <div className="mesure-essais-header-clim">CLIM4</div>
            <div className="mesure-essais-label">Tension Générale  Climatiseur</div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div>
            <div className="mesure-essais-label">Intensité Générale Climatiseur</div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div>
            <div className="mesure-essais-label">Intensité Compresseur</div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div>
            <div className="mesure-essais-label">Intensité Moteurs Ventilateurs Cond</div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div>
            <div className="mesure-essais-label">Intensité Moteurs Ventilateurs Evap</div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div>
            <div className="mesure-essais-label">Haute pression (HP)</div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div>
            <div className="mesure-essais-label">Basse pression (BP)</div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div>
            <div className="mesure-essais-label">Température de soufflage</div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div>
            <div className="mesure-essais-label">Température  du local</div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div>
            <div className="mesure-essais-label">Débit d'air de soufflage</div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div>
            <div className="mesure-essais-header-section">6. ESSAIS ELECTRIQUE & FRIGORIFIQUE</div>
            <div className="mesure-essais-label">Essai de la sécurité BP</div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div>
            <div className="mesure-essais-label">Essai de la sécurité HP</div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div>
            <div className="mesure-essais-label">Essai Marche Forcée en cas HT°</div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div>
            <div className="mesure-essais-label">Essai du basculement  en cas de défaut</div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div><div className="mesure-essais-cell"></div>
          </div>
        </div>

        {/* 7. COMPTE RENDU RESPONSABLE TECHNIQUE (CLIENT) + 8. OBSERVATIONS (Combined Table) */}
        <div className="compte-observations-table">
          <div className="compte-observations-header">7. COMPTE RENDU RESPONSABLE TECHNIQUE (CLIENT)</div>
          <div className="compte-observations-header">8. OBSERVATIONS</div>
          <div className="compte-observations-cell">
            <textarea className="compte-observations-textarea" />
          </div>
          <div className="compte-observations-cell">
            <textarea className="compte-observations-textarea" />
          </div>
        </div>

        {/* Signatures (Combined Table) */}
        <div className="signatures-table">
          <div className="signatures-header">Nom, Prénom & Signature Responsable Technique (Client)</div>
          <div className="signatures-header">Nom, Prénom & Signature Technicien (Chanitec)</div>
          <div className="signatures-cell">
            <input className="signatures-input" type="text" />
          </div>
          <div className="signatures-cell">
            <input className="signatures-input" type="text" />
          </div>
        </div>
      </div>
    </Box>
  );
}

============================================================
FILE: src/pages/orgChartPage.scss
============================================================
.orgchart-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px 0;
  background: #f5f6f8;
  min-height: 100vh;
}

.orgchart-leader {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 16px;
}

.orgchart-card {
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 24px 16px 16px 16px;
  min-width: 180px;
  margin: 0 12px;
  transition: box-shadow 0.2s;
}

.orgchart-card.leader {
  border: 2px solid #007bff;
  min-width: 220px;
  padding: 32px 24px 20px 24px;
}

.orgchart-avatar {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  object-fit: cover;
  margin-bottom: 12px;
  border: 2px solid #e0e0e0;
}

.orgchart-avatar.leader {
  width: 80px;
  height: 80px;
  border: 3px solid #007bff;
}

.orgchart-name {
  font-weight: 600;
  font-size: 1.1rem;
  color: #007bff;
  margin-bottom: 4px;
  text-align: center;
}

.orgchart-name.leader {
  font-size: 1.25rem;
}

.orgchart-title {
  font-size: 0.98rem;
  color: #444;
  margin-bottom: 2px;
  text-align: center;
}

.orgchart-title.leader {
  font-size: 1.05rem;
  color: #222;
}

.orgchart-location {
  font-size: 0.92rem;
  color: #888;
  text-align: center;
}

.orgchart-location.leader {
  color: #666;
}

.orgchart-line {
  width: 80vw;
  max-width: 900px;
  height: 2px;
  background: #e0e0e0;
  margin: 18px 0 24px 0;
}

.orgchart-sections {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  gap: 100px;
  width: 100%;
  max-width: 1400px;
  padding: 0 20px;
}

.orgchart-section {
  display: flex;
  flex-direction: column;
  gap: 20px;
  flex: 1;
  min-width: 0;
}

.orgchart-section-title {
  font-size: 1.5rem;
  color: #333;
  text-align: center;
  margin: 0;
  padding-bottom: 10px;
  border-bottom: 2px solid #e0e0e0;
}

.orgchart-advisors {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.orgchart-row {
  display: flex;
  justify-content: center;
  gap: 20px;
}

.orgchart-card.advisor {
  min-width: 150px;
  max-width: 180px;
  padding: 20px 10px 14px 10px;
}

.orgchart-app-bar {
  background-color: #1976d2 !important;
  box-shadow: none;
}

.orgchart-toolbar {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: flex-start;
  min-height: 64px;
  gap: 16px;
  padding-left: 24px;
}

.orgchart-nav-btn {
  color: #fff !important;
  font-weight: 500;
  text-transform: none;
  margin-right: 1rem;
  border-radius: 4px;
  transition: background 0.2s;
}

.orgchart-nav-btn:disabled {
  color: #e0e0e0 !important;
  opacity: 0.7;
}

.orgchart-nav-btn:not(:disabled):hover {
  background: rgba(255,255,255,0.08);
}

.orgchart-navbar {
  display: none !important;
}

============================================================
FILE: src/pages/orgChartPage.tsx
============================================================
import React from 'react';
import { AppBar, Toolbar, Button, Box } from '@mui/material';
import { useNavigate } from 'react-router-dom';
import './orgChartPage.scss';

interface Employee {
  id: number;
  name: string;
  title: string;
  location: string;
  avatar: string;
  subType?: string;
  children?: Employee[];
}

const employees: Employee[] = [
  {
    id: 1,
    name: 'Bilel AYACHI',
    title: 'Departement Froid et climatisation',
    location: 'TUN Tunis - Extension',
    avatar: 'https://randomuser.me/api/portraits/men/61.jpg',
    children: [
      {
        id: 2,
        name: 'BALU MAVINGA Jean',
        title: 'Chef de service Chargé de clim-domestique',
        subType: 'UTEX',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/62.jpg',
      },
      {
        id: 3,
        name: 'IKALABA NKOSI Louison',
        title: 'Chef de service Chargé de clim-domestique',
        subType: 'UTEX',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/63.jpg',
      },
      {
        id: 4,
        name: 'MATALATALA WISAMAU Richard',
        title: 'Chef de service Chargé de clim-domestique',
        subType: 'UTEX',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/64.jpg',
      },
      {
        id: 5,
        name: 'MBENZA VUAMISA Willy',
        title: 'Chef de service Chargé de clim-domestique',
        subType: 'SNEL',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/65.jpg',
      },
      {
        id: 6,
        name: 'MFIKA MFUNDU KIMPEMBE Roc',
        title: 'Chef de service Chargé de clim-domestique',
        subType: 'UTEX',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/66.jpg',
      },
      {
        id: 7,
        name: 'TOKO ZABANA Juvénal',
        title: 'Chef de service Chargé de clim-domestique',
        subType: 'UTEX',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/67.jpg',
      },
      {
        id: 8,
        name: 'KAKUTALUA NGUVU Bienvenu',
        title: 'Polyvalent',
        subType: 'POLIVALONT',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/68.jpg',
      },
      {
        id: 9,
        name: 'KAMAKAMA MBALA Joseph',
        title: 'Polyvalent',
        subType: 'POLIVALONT',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/69.jpg',
      },
      {
        id: 10,
        name: 'KUMBANA MOYO Beckers',
        title: 'Polyvalent',
        subType: 'POLIVALONT',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/70.jpg',
      },
      {
        id: 11,
        name: 'LUVUALU Thomas',
        title: 'Polyvalent',
        subType: 'POLIVALONT',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/71.jpg',
      },
      {
        id: 12,
        name: 'MENANKUTIMA NSOMI Marc',
        title: 'Polyvalent',
        subType: 'POLIVALONT',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/72.jpg',
      },
      {
        id: 13,
        name: 'MOBATUE MBEMBA Rigaen',
        title: 'Polyvalent',
        subType: 'POLIVALONT',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/73.jpg',
      },
      {
        id: 14,
        name: 'DIANABO KALIMUNDA Marius',
        title: 'Chef de service adj chargé du climatisation centralisé',
        subType: 'PULLMAN',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/74.jpg',
      },
      {
        id: 15,
        name: 'MALONGA KUAMA Isidore',
        title: 'Chef de service adj chargé du climatisation centralisé',
        subType: 'PULLMAN',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/75.jpg',
      },
      {
        id: 16,
        name: 'MBIYAVANGA MATALA Antoine',
        title: 'Chef de service adj chargé du climatisation centralisé',
        subType: 'PULLMAN',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/76.jpg',
      },
      {
        id: 17,
        name: 'MUSOMONI KAFUTI Trésor-Benjamin',
        title: 'Chef de service adj chargé du climatisation centralisé',
        subType: 'BCDC',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/77.jpg',
      },
      {
        id: 18,
        name: 'NDOMBASI NGOMBO Diego',
        title: 'Chef de service adj chargé du climatisation centralisé',
        subType: 'BCDC',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/78.jpg',
      },
      {
        id: 19,
        name: 'NTOTO PHUATI Sylvain',
        title: 'Chef de service adj chargé du climatisation centralisé',
        subType: 'BCDC',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/79.jpg',
      },
      {
        id: 20,
        name: 'SADI TONDASE Dodo',
        title: 'Chef de service adj chargé du climatisation centralisé',
        subType: 'BCDC',
        location: 'TUN Tunis - Extension',
        avatar: 'https://randomuser.me/api/portraits/men/80.jpg',
      },
    ],
  },
];

const OrgChartPage = () => {
  const leader = employees[0];
  const navigate = useNavigate();
  const isAdmin = localStorage.getItem('role') === 'admin';

  // Group employees by title
  const climDomestiqueEmployees = leader.children?.filter(emp => emp.title === 'Chef de service Chargé de clim-domestique') || [];
  const polyvalentEmployees = leader.children?.filter(emp => emp.title === 'Polyvalent') || [];
  const climatisationCentraliseEmployees = leader.children?.filter(emp => emp.title === 'Chef de service adj chargé du climatisation centralisé') || [];

  // Function to render employee rows
  const renderEmployeeRows = (employees: Employee[]) => {
    const rows = [];
    for (let i = 0; i < employees.length; i += 2) {
      const row = employees.slice(i, i + 2);
      rows.push(
        <div className="orgchart-row" key={i}>
          {row.map((employee: Employee) => (
            <div className="orgchart-card advisor" key={employee.id}>
              <img src={employee.avatar} alt={employee.name} className="orgchart-avatar" />
              <div className="orgchart-name">{employee.name}</div>
              <div className="orgchart-title">{employee.subType || employee.title}</div>
              <div className="orgchart-location">{employee.location}</div>
            </div>
          ))}
        </div>
      );
    }
    return rows;
  };

  return (
    <div className="orgchart-container">
      <AppBar position="static" color="primary" className="orgchart-app-bar">
        <Toolbar className="orgchart-toolbar">
          <Button color="inherit" onClick={() => navigate('/home')} className="orgchart-nav-btn">
            Back to Home
          </Button>
          <Button color="inherit" onClick={() => navigate('/employees')} className="orgchart-nav-btn">
            Employees
          </Button>
          <Button color="inherit" disabled className="orgchart-nav-btn">
            Profile
          </Button>
          {isAdmin && (
            <Button color="inherit" disabled className="orgchart-nav-btn">
              Edit Employee
            </Button>
          )}
        </Toolbar>
      </AppBar>
      <div className="orgchart-leader">
        <div className="orgchart-card leader">
          <img src={leader.avatar} alt={leader.name} className="orgchart-avatar leader" />
          <div className="orgchart-name leader">{leader.name}</div>
          <div className="orgchart-title leader">{leader.title}</div>
          <div className="orgchart-location leader">{leader.location}</div>
        </div>
      </div>
      <div className="orgchart-line" />

      <div className="orgchart-sections">
        <div className="orgchart-section">
          <h2 className="orgchart-section-title">Chef de service Chargé de clim-domestique</h2>
          <div className="orgchart-advisors">
            {renderEmployeeRows(climDomestiqueEmployees)}
          </div>
        </div>

        <div className="orgchart-section">
          <h2 className="orgchart-section-title">Polyvalent</h2>
          <div className="orgchart-advisors">
            {renderEmployeeRows(polyvalentEmployees)}
          </div>
        </div>

        <div className="orgchart-section">
          <h2 className="orgchart-section-title">Chef de service adj chargé du climatisation centralisé</h2>
          <div className="orgchart-advisors">
            {renderEmployeeRows(climatisationCentraliseEmployees)}
          </div>
        </div>
      </div>
    </div>
  );
};

export default OrgChartPage;

============================================================
FILE: src/services/api.js
============================================================
const API_BASE_URL = process.env.REACT_APP_API_URL;

// Items API calls
export const itemsApi = {
    getAllItems: async () => {
        try {
            const response = await fetch(`${API_BASE_URL}/items`);
            if (!response.ok) throw new Error('Failed to fetch items');
            return await response.json();
        } catch (error) {
            console.error('Error fetching items:', error);
            throw error;
        }
    },

    getItemById: async (id) => {
        try {
            const response = await fetch(`${API_BASE_URL}/items/${id}`);
            if (!response.ok) throw new Error('Failed to fetch item');
            return await response.json();
        } catch (error) {
            console.error('Error fetching item:', error);
            throw error;
        }
    },

    createItem: async (itemData) => {
        try {
            const response = await fetch(`${API_BASE_URL}/items`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    ...itemData,
                    quantity: itemData.quantity === undefined || itemData.quantity === null ? 0 : itemData.quantity
                }),
            });
            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(errorData.error || 'Failed to create item');
            }
            return await response.json();
        } catch (error) {
            console.error('Error creating item:', error);
            throw error;
        }
    },

    updateItem: async (id, itemData) => {
        try {
            const response = await fetch(`${API_BASE_URL}/items/${id}`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    ...itemData,
                    quantity: itemData.quantity === undefined || itemData.quantity === null ? 0 : itemData.quantity
                }),
            });
            if (!response.ok) throw new Error('Failed to update item');
            return await response.json();
        } catch (error) {
            console.error('Error updating item:', error);
            throw error;
        }
    },

    deleteItem: async (id) => {
        try {
            const response = await fetch(`${API_BASE_URL}/items/${id}`, {
                method: 'DELETE',
            });
            if (!response.ok) throw new Error('Failed to delete item');
            return true;
        } catch (error) {
            console.error('Error deleting item:', error);
            throw error;
        }
    },

    importItems: async (formData) => {
        try {
            const response = await fetch(`${API_BASE_URL}/items/import`, {
                method: 'POST',
                body: formData
            });
            if (!response.ok) throw new Error('Failed to import items');
            return await response.json();
        } catch (error) {
            console.error('Error importing items:', error);
            throw error;
        }
    },

    clearItems: async () => {
        try {
            const response = await fetch(`${API_BASE_URL}/items/clear`, {
                method: 'DELETE',
            });
            if (!response.ok) throw new Error('Failed to clear items');
            return await response.json();
        } catch (error) {
            console.error('Error clearing items:', error);
            throw error;
        }
    }
};

// Quotes API calls
export const quotesApi = {
    getAllQuotes: async () => {
        try {
            const response = await fetch(`${API_BASE_URL}/quotes`);
            if (!response.ok) throw new Error('Failed to fetch quotes');
            return await response.json();
        } catch (error) {
            console.error('Error fetching quotes:', error);
            throw error;
        }
    },

    getQuoteById: async (id) => {
        try {
            const response = await fetch(`${API_BASE_URL}/quotes/${id}`);
            if (!response.ok) throw new Error('Failed to fetch quote');
            return await response.json();
        } catch (error) {
            console.error('Error fetching quote:', error);
            throw error;
        }
    },

    createQuote: async (quoteData) => {
        try {
            const response = await fetch(`${API_BASE_URL}/quotes`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(quoteData),
            });
            if (!response.ok) throw new Error('Failed to create quote');
            return await response.json();
        } catch (error) {
            console.error('Error creating quote:', error);
            throw error;
        }
    },

    updateQuote: async (id, quoteData) => {
        try {
            const response = await fetch(`${API_BASE_URL}/quotes/${id}`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(quoteData),
            });
            if (!response.ok) throw new Error('Failed to update quote');
            return await response.json();
        } catch (error) {
            console.error('Error updating quote:', error);
            throw error;
        }
    },

    deleteQuote: async (id) => {
        try {
            const response = await fetch(`${API_BASE_URL}/quotes/${id}`, {
                method: 'DELETE',
            });
            if (!response.ok) throw new Error('Failed to delete quote');
            return true;
        } catch (error) {
            console.error('Error deleting quote:', error);
            throw error;
        }
    },
};

// Clients API calls
export const clientsApi = {
    getAllClients: async () => {
        try {
            const response = await fetch(`${API_BASE_URL}/clients`);
            if (!response.ok) throw new Error('Failed to fetch clients');
            const clients = await response.json();

            // Fetch sites for each client
            const clientsWithSites = await Promise.all(
                clients.map(async (client) => {
                    const siteResponse = await fetch(`${API_BASE_URL}/sites?client_id=${client.id}`);
                    if (siteResponse.ok) {
                        const sites = await siteResponse.json();
                        return { ...client, site: sites[0] }; // Assuming one site per client for now
                    }
                    return client;
                })
            );

            return clientsWithSites;
        } catch (error) {
            console.error('Error fetching clients:', error);
            throw error;
        }
    },

    createClient: async (clientData) => {
        try {
            // First create the client
            const clientResponse = await fetch(`${API_BASE_URL}/clients`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ name: clientData.name }),
            });

            if (!clientResponse.ok) {
                const errorData = await clientResponse.json();
                throw new Error(errorData.error || 'Failed to create client');
            }

            const newClient = await clientResponse.json();

            // Then create the site for this client
            if (clientData.site) {
                const siteResponse = await fetch(`${API_BASE_URL}/sites`, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({
                        ...clientData.site,
                        client_id: newClient.id
                    }),
                });

                if (!siteResponse.ok) {
                    const errorData = await siteResponse.json();
                    throw new Error(errorData.error || 'Failed to create site');
                }

                const newSite = await siteResponse.json();
                return { ...newClient, site: newSite };
            }

            return newClient;
        } catch (error) {
            console.error('Error creating client:', error);
            throw error;
        }
    },
};

// Sites API calls
export const sitesApi = {
    getAllSites: async () => {
        try {
            const response = await fetch(`${API_BASE_URL}/sites`);
            if (!response.ok) throw new Error('Failed to fetch sites');
            return await response.json();
        } catch (error) {
            console.error('Error fetching sites:', error);
            throw error;
        }
    },

    createSite: async (siteData) => {
        try {
            const response = await fetch(`${API_BASE_URL}/sites`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(siteData),
            });
            if (!response.ok) throw new Error('Failed to create site');
            return await response.json();
        } catch (error) {
            console.error('Error creating site:', error);
            throw error;
        }
    },
};

============================================================
FILE: src/utils/generate-excel-sample.js
============================================================
const XLSX = require('xlsx');
const fs = require('fs');
const path = require('path');

// Read the sample JSON data
const sampleData = JSON.parse(fs.readFileSync(path.join(__dirname, '../data/sample-items.json'), 'utf8'));

// Create a new workbook
const workbook = XLSX.utils.book_new();

// Convert the JSON data to a worksheet
const worksheet = XLSX.utils.json_to_sheet(sampleData);

// Add the worksheet to the workbook
XLSX.utils.book_append_sheet(workbook, worksheet, 'Items');

// Create output directory if it doesn't exist
const outputDir = path.join(__dirname, '../data/excel');
if (!fs.existsSync(outputDir)) {
    fs.mkdirSync(outputDir, { recursive: true });
}

// Write the workbook to a file
const outputPath = path.join(outputDir, 'sample-items.xlsx');
XLSX.writeFile(workbook, outputPath);

console.log(`Sample Excel file created at: ${outputPath}`);

============================================================
FILE: src/App.css
============================================================
.App {
  text-align: center;
}

.App-logo {
  height: 40vmin;
  pointer-events: none;
}

@media (prefers-reduced-motion: no-preference) {
  .App-logo {
    animation: App-logo-spin infinite 20s linear;
  }
}

.App-header {
  background-color: #282c34;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-size: calc(10px + 2vmin);
  color: white;
}

.App-link {
  color: #61dafb;
}

@keyframes App-logo-spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}


============================================================
FILE: src/App.scss
============================================================
// Reset some browser defaults
html, body {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Roboto', 'Arial', sans-serif;
}

// General style resets and defaults
* {
  box-sizing: border-box;
  margin-top: 0;
  padding-top: 0;
}

// Base styles
body {
  background-color: #f5f5f5;
  color: rgba(0, 0, 0, 0.87);
  line-height: 1.43;
  letter-spacing: 0.01071em;
}

// Custom number input styling
.custom-number-input {
  position: relative;
  display: inline-flex;
  align-items: center;

  input[type="number"] {
    -moz-appearance: textfield;
    appearance: textfield;
    width: 60px;
    padding: 4px 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
    text-align: center;

    &::-webkit-inner-spin-button,
    &::-webkit-outer-spin-button {
      -webkit-appearance: none;
      margin: 0;
    }
  }

  .number-controls {
    display: inline-flex;
    margin-left: 4px;

    button {
      width: 24px;
      height: 24px;
      padding: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      background: #fff;
      border: 1px solid #ddd;
      cursor: pointer;
      font-size: 16px;
      line-height: 1;
      color: #666;

      &:first-child {
        border-radius: 4px 0 0 4px;
        border-right: none;
      }

      &:last-child {
        border-radius: 0 4px 4px 0;
      }

      &:hover {
        background: #f5f5f5;
      }

      &:active {
        background: #e0e0e0;
      }
    }
  }
}

// Page header styling
.page-header {
  background-color: #1976d2;
  color: white;
  padding: 1rem;
  text-align: center;
  margin-bottom: 1rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);

  .page-title {
    font-weight: bold;
    letter-spacing: 0.5px;
    margin: 0;
  }
}

// Print-specific styles
@media print {
  @page {
    size: A4;
    margin: 0;
  }


  .page-header {
    display: none; // Hide the page header when printing
  }

  button, input[type="submit"], input[type="reset"] {
    display: none !important;
  }



  .quote-content,
  .quote-content > div,
  .layout-root {
    page-break-inside: avoid !important;
    page-break-before: avoid !important;
    page-break-after: avoid !important;
  }

  // Hide all input fields and form elements
  .MuiInputBase-root, .MuiTextField-root, .MuiFormLabel-root, .MuiOutlinedInput-notchedOutline {
    border: none !important;
  }

  .MuiInputLabel-root {
    position: relative !important;
    transform: none !important;
    font-weight: bold !important;
  }

  .MuiInputBase-input, .MuiSelect-select {
    padding-left: 0 !important;
    padding-top: 0 !important;
    padding-bottom: 0 !important;
  }

  .rates-container {
    display: none !important;
  }

  * {
    -webkit-print-color-adjust: exact !important;
    color-adjust: exact !important;
  }

  // Hide all add/edit buttons and forms globally
  button[class*="add"],
  button[class*="edit"],
  form[class*="add"],
  .MuiButtonBase-root[class*="add"],
  div[class*="add-labor"],
  div[class*="add-item"],
  .add-labor-form,
  .item-actions {
    display: none !important;
    visibility: hidden !important;
    height: 0 !important;
    overflow: hidden !important;
  }
}

.App {
  text-align: center;
  padding: 20px;
}

h1 {
  color: #1976d2;
}

============================================================
FILE: src/App.test.tsx
============================================================
import React from 'react';
import { render, screen } from '@testing-library/react';
import App from './App';

test('renders learn react link', () => {
  render(<App />);
  const linkElement = screen.getByText(/learn react/i);
  expect(linkElement).toBeInTheDocument();
});


============================================================
FILE: src/App.tsx
============================================================
import React, { useState, useEffect } from 'react';
import { ThemeProvider, createTheme } from '@mui/material/styles';
import CssBaseline from '@mui/material/CssBaseline';
import { Snackbar, Alert } from '@mui/material';
import { HashRouter, Routes, Route, Navigate, useNavigate } from 'react-router-dom';
import { QuoteProvider } from './contexts/QuoteContext';
import QuotePage from './pages/QuotePage/QuotePage';
import HistoryPage from './pages/HistoryPage/HistoryPage';
import ClientsPage from './pages/ClientsPage/ClientsPage';
import ItemsPage from './pages/ItemsPage/ItemsPage';
import PriceOfferPage from './pages/PriceOfferPage/PriceOfferPage';
import LoginPage from './pages/LoginPage/LoginPage';
import HomePage from './pages/HomePage/HomePage';
import QuoteTest from './pages/QuoteTest/QuoteTest';
import InterventionPage from './pages/interventionPage';
import { storageService } from './services/storage-service';
import OrgChartPage from './pages/orgChartPage';
import EmployeesPage from './pages/employeesPage';
import './App.scss';

const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
    },
    secondary: {
      main: '#2196f3',
    },
    background: {
      default: '#f5f5f5',
    },
  },
  typography: {
    fontFamily: 'Roboto, Arial, sans-serif',
    h6: {
      fontWeight: 600,
    },
  },
  components: {
    MuiButton: {
      styleOverrides: {
        root: {
          textTransform: 'none',
          borderRadius: 4,
        },
      },
    },
    MuiPaper: {
      styleOverrides: {
        root: {
          borderRadius: 4,
        },
      },
    },
    MuiInputBase: {
      styleOverrides: {
        input: {
          '&[type="number"]': {
            '-moz-appearance': 'textfield',
            padding: '8px 12px',
            paddingRight: '20px',
          },
          '&::-webkit-inner-spin-button, &::-webkit-outer-spin-button': {
            height: '100%',
            width: '20px',
            opacity: 1,
            margin: 0,
            padding: 0,
            cursor: 'pointer',
            position: 'absolute',
            right: 0,
            background: 'transparent',
            '&:hover': {
              background: 'rgba(0, 0, 0, 0.05)',
            },
            '&:active': {
              background: 'rgba(0, 0, 0, 0.1)',
            },
          },
        },
      },
    },
  },
});

// Protected Route component
const ProtectedRoute = ({ children }: { children: React.ReactNode }) => {
  const isAuthenticated = localStorage.getItem('isAuthenticated') === 'true';
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  return <>{children}</>;
};

// Create a wrapper component that uses useNavigate
const AppContent = () => {
  const navigate = useNavigate();
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [currentPath, setCurrentPath] = useState('/');
  const [notification, setNotification] = useState({
    open: false,
    message: '',
    severity: 'info' as 'info' | 'success' | 'warning' | 'error'
  });
  const [priceOfferQuoteId, setPriceOfferQuoteId] = useState<string | undefined>();

  // --- Warm-up logic start ---
  const [isWarmingUp, setIsWarmingUp] = useState(false);

  useEffect(() => {
    const hasWarmedUp = localStorage.getItem('hasWarmedUp') === 'true';
    if (!hasWarmedUp) {
      setIsWarmingUp(true);
      const controller = new AbortController();
      const timeout = setTimeout(() => {
        controller.abort();
        setIsWarmingUp(false);
        localStorage.setItem('hasWarmedUp', 'true');
      }, 3000); // 3 seconds instead of 30
      fetch('/api/health', { signal: controller.signal })
        .then(() => {
          clearTimeout(timeout);
          setIsWarmingUp(false);
          localStorage.setItem('hasWarmedUp', 'true');
        })
        .catch(() => {
          // Proceed even if error
          setIsWarmingUp(false);
          localStorage.setItem('hasWarmedUp', 'true');
        });
      return () => clearTimeout(timeout);
    }
  }, []);
  // --- Warm-up logic end ---

  // Initialize app data
  useEffect(() => {
    const isInitialized = localStorage.getItem('app_initialized') === 'true';
    const justInitialized = localStorage.getItem('app_just_initialized') !== 'true';

    if (isInitialized && justInitialized) {
      setNotification({
        open: true,
        message: 'Données d\'exemple chargées avec succès',
        severity: 'success'
      });
      localStorage.setItem('app_just_initialized', 'true');
    }
  }, []);

  // Handle login
  const handleLogin = (username: string, password: string) => {
    if (username && password) {
      setIsAuthenticated(true);
      localStorage.setItem('isAuthenticated', 'true');
    } else {
      setNotification({
        open: true,
        message: 'Veuillez remplir tous les champs',
        severity: 'error'
      });
    }
  };

  // Handle logout
  const handleLogout = () => {
    setIsAuthenticated(false);
    localStorage.removeItem('isAuthenticated');
    navigate('/login');
  };

  // Handle navigation
  const handleNavigate = (path: string, quoteId?: string) => {
    if (quoteId) {
      const queryParams = new URLSearchParams(window.location.search);
      const fromHistory = queryParams.get('fromHistory');
      const confirmed = queryParams.get('confirmed');

      let newPath = `${path}?id=${quoteId}`;
      if (fromHistory) newPath += `&fromHistory=${fromHistory}`;
      if (confirmed) newPath += `&confirmed=${confirmed}`;

      setPriceOfferQuoteId(quoteId);
      setCurrentPath(newPath);
      navigate(newPath);
    } else {
      setCurrentPath(path);
      navigate(path);
    }
  };

  // Close notification
  const handleCloseNotification = () => {
    setNotification({...notification, open: false});
  };

  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      {/* Warm-up overlay */}
      {isWarmingUp && (
        <div style={{
          position: 'fixed',
          top: 0,
          left: 0,
          width: '100vw',
          height: '100vh',
          background: 'rgba(255,255,255,0.95)',
          zIndex: 9999,
          display: 'flex',
          flexDirection: 'column',
          alignItems: 'center',
          justifyContent: 'center',
          fontSize: 24
        }}>
          <div className="spinner" style={{ marginBottom: 20 }}>
            <svg width="48" height="48" viewBox="0 0 48 48" fill="none" xmlns="http://www.w3.org/2000/svg">
              <circle cx="24" cy="24" r="20" stroke="#1976d2" strokeWidth="4" strokeDasharray="100" strokeDashoffset="60"/>
            </svg>
          </div>
          Initialisation du serveur... Veuillez patienter
        </div>
      )}
      {/* App content */}
      {!isWarmingUp && (
        <QuoteProvider>
          <Routes>
            <Route
              path="/login"
              element={
                isAuthenticated ?
                <Navigate to="/home" replace /> :
                <LoginPage onLogin={handleLogin} />
              }
            />

            <Route
              path="/home"
              element={
                <ProtectedRoute>
                  <HomePage />
                </ProtectedRoute>
              }
            />

            <Route
              path="/quote"
              element={
                <ProtectedRoute>
                  <QuotePage currentPath="/quote" onNavigate={handleNavigate} onLogout={handleLogout} />
                </ProtectedRoute>
              }
            />

            <Route
              path="/quote-test"
              element={
                <ProtectedRoute>
                  <QuoteTest currentPath="/quote-test" onNavigate={handleNavigate} />
                </ProtectedRoute>
              }
            />

            <Route
              path="/history"
              element={
                <ProtectedRoute>
                  <HistoryPage currentPath="/history" onNavigate={handleNavigate} />
                </ProtectedRoute>
              }
            />

            <Route
              path="/clients"
              element={
                <ProtectedRoute>
                  <ClientsPage currentPath="/clients" onNavigate={handleNavigate} />
                </ProtectedRoute>
              }
            />

            <Route
              path="/items"
              element={
                <ProtectedRoute>
                  <ItemsPage currentPath="/items" onNavigate={handleNavigate} />
                </ProtectedRoute>
              }
            />

            <Route
              path="/price-offer"
              element={
                <ProtectedRoute>
                  <PriceOfferPage
                    currentPath="/price-offer"
                    onNavigate={handleNavigate}
                    quoteId={priceOfferQuoteId}
                  />
                </ProtectedRoute>
              }
            />

            <Route
              path="/intervention"
              element={
                <ProtectedRoute>
                  <InterventionPage />
                </ProtectedRoute>
              }
            />

            <Route
              path="/org-chart"
              element={
                <ProtectedRoute>
                  <OrgChartPage />
                </ProtectedRoute>
              }
            />

            <Route
              path="/employees"
              element={
                <ProtectedRoute>
                  <EmployeesPage />
                </ProtectedRoute>
              }
            />

            <Route path="/" element={<Navigate to="/home" replace />} />
          </Routes>
        </QuoteProvider>
      )}
      <Snackbar
        open={notification.open}
        autoHideDuration={6000}
        onClose={handleCloseNotification}
        anchorOrigin={{ vertical: 'bottom', horizontal: 'center' }}
      >
        <Alert onClose={handleCloseNotification} severity={notification.severity}>
          {notification.message}
        </Alert>
      </Snackbar>
    </ThemeProvider>
  );
};

// Main App component
function App() {
  return (
    <HashRouter>
      <AppContent />
    </HashRouter>
  );
}

export default App;

============================================================
FILE: src/index.css
============================================================
body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
    'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
    sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

code {
  font-family: source-code-pro, Menlo, Monaco, Consolas, 'Courier New',
    monospace;
}


============================================================
FILE: src/index.tsx
============================================================
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import reportWebVitals from './reportWebVitals';

const root = ReactDOM.createRoot(
  document.getElementById('root') as HTMLElement
);

root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();

============================================================
FILE: src/logo.svg
============================================================
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 841.9 595.3"><g fill="#61DAFB"><path d="M666.3 296.5c0-32.5-40.7-63.3-103.1-82.4 14.4-63.6 8-114.2-20.2-130.4-6.5-3.8-14.1-5.6-22.4-5.6v22.3c4.6 0 8.3.9 11.4 2.6 13.6 7.8 19.5 37.5 14.9 75.7-1.1 9.4-2.9 19.3-5.1 29.4-19.6-4.8-41-8.5-63.5-10.9-13.5-18.5-27.5-35.3-41.6-50 32.6-30.3 63.2-46.9 84-46.9V78c-27.5 0-63.5 19.6-99.9 53.6-36.4-33.8-72.4-53.2-99.9-53.2v22.3c20.7 0 51.4 16.5 84 46.6-14 14.7-28 31.4-41.3 49.9-22.6 2.4-44 6.1-63.6 11-2.3-10-4-19.7-5.2-29-4.7-38.2 1.1-67.9 14.6-75.8 3-1.8 6.9-2.6 11.5-2.6V78.5c-8.4 0-16 1.8-22.6 5.6-28.1 16.2-34.4 66.7-19.9 130.1-62.2 19.2-102.7 49.9-102.7 82.3 0 32.5 40.7 63.3 103.1 82.4-14.4 63.6-8 114.2 20.2 130.4 6.5 3.8 14.1 5.6 22.5 5.6 27.5 0 63.5-19.6 99.9-53.6 36.4 33.8 72.4 53.2 99.9 53.2 8.4 0 16-1.8 22.6-5.6 28.1-16.2 34.4-66.7 19.9-130.1 62-19.1 102.5-49.9 102.5-82.3zm-130.2-66.7c-3.7 12.9-8.3 26.2-13.5 39.5-4.1-8-8.4-16-13.1-24-4.6-8-9.5-15.8-14.4-23.4 14.2 2.1 27.9 4.7 41 7.9zm-45.8 106.5c-7.8 13.5-15.8 26.3-24.1 38.2-14.9 1.3-30 2-45.2 2-15.1 0-30.2-.7-45-1.9-8.3-11.9-16.4-24.6-24.2-38-7.6-13.1-14.5-26.4-20.8-39.8 6.2-13.4 13.2-26.8 20.7-39.9 7.8-13.5 15.8-26.3 24.1-38.2 14.9-1.3 30-2 45.2-2 15.1 0 30.2.7 45 1.9 8.3 11.9 16.4 24.6 24.2 38 7.6 13.1 14.5 26.4 20.8 39.8-6.3 13.4-13.2 26.8-20.7 39.9zm32.3-13c5.4 13.4 10 26.8 13.8 39.8-13.1 3.2-26.9 5.9-41.2 8 4.9-7.7 9.8-15.6 14.4-23.7 4.6-8 8.9-16.1 13-24.1zM421.2 430c-9.3-9.6-18.6-20.3-27.8-32 9 .4 18.2.7 27.5.7 9.4 0 18.7-.2 27.8-.7-9 11.7-18.3 22.4-27.5 32zm-74.4-58.9c-14.2-2.1-27.9-4.7-41-7.9 3.7-12.9 8.3-26.2 13.5-39.5 4.1 8 8.4 16 13.1 24 4.7 8 9.5 15.8 14.4 23.4zM420.7 163c9.3 9.6 18.6 20.3 27.8 32-9-.4-18.2-.7-27.5-.7-9.4 0-18.7.2-27.8.7 9-11.7 18.3-22.4 27.5-32zm-74 58.9c-4.9 7.7-9.8 15.6-14.4 23.7-4.6 8-8.9 16-13 24-5.4-13.4-10-26.8-13.8-39.8 13.1-3.1 26.9-5.8 41.2-7.9zm-90.5 125.2c-35.4-15.1-58.3-34.9-58.3-50.6 0-15.7 22.9-35.6 58.3-50.6 8.6-3.7 18-7 27.7-10.1 5.7 19.6 13.2 40 22.5 60.9-9.2 20.8-16.6 41.1-22.2 60.6-9.9-3.1-19.3-6.5-28-10.2zM310 490c-13.6-7.8-19.5-37.5-14.9-75.7 1.1-9.4 2.9-19.3 5.1-29.4 19.6 4.8 41 8.5 63.5 10.9 13.5 18.5 27.5 35.3 41.6 50-32.6 30.3-63.2 46.9-84 46.9-4.5-.1-8.3-1-11.3-2.7zm237.2-76.2c4.7 38.2-1.1 67.9-14.6 75.8-3 1.8-6.9 2.6-11.5 2.6-20.7 0-51.4-16.5-84-46.6 14-14.7 28-31.4 41.3-49.9 22.6-2.4 44-6.1 63.6-11 2.3 10.1 4.1 19.8 5.2 29.1zm38.5-66.7c-8.6 3.7-18 7-27.7 10.1-5.7-19.6-13.2-40-22.5-60.9 9.2-20.8 16.6-41.1 22.2-60.6 9.9 3.1 19.3 6.5 28.1 10.2 35.4 15.1 58.3 34.9 58.3 50.6-.1 15.7-23 35.6-58.4 50.6zM320.8 78.4z"/><circle cx="420.9" cy="296.5" r="45.7"/><path d="M520.5 78.1z"/></g></svg>

============================================================
FILE: electron-main.js
============================================================
const { app, BrowserWindow } = require('electron');
const path = require('path');

console.log("Electron main process started");

function createWindow() {
  console.log("Creating Electron window...");
  const win = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: {
      nodeIntegration: false,
      contextIsolation: true,
    },
  });

  const isDev = process.env.NODE_ENV === 'development' || process.env.ELECTRON_START_URL;
  const startUrl = isDev
    ? process.env.ELECTRON_START_URL || 'http://localhost:3000'
    : `file://${path.join(__dirname, 'build', 'index.html')}`;
  console.log("Loading URL:", startUrl);
  win.loadURL(startUrl);

  win.on('ready-to-show', () => {
    console.log("Window ready to show");
    win.show();
  });

  win.on('closed', () => {
    console.log("Window closed");
  });
}

app.whenReady().then(createWindow);

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});

app.on('activate', () => {
  if (BrowserWindow.getAllWindows().length === 0) {
    createWindow();
  }
});

============================================================
FILE: package.json
============================================================
{
    "name": "chanitec-frontend",
    "version": "0.1.0",
    "private": true,
    "dependencies": {
        "@emotion/react": "^11.14.0",
        "@emotion/styled": "^11.14.0",
        "@hookform/resolvers": "^5.0.1",
        "@mui/icons-material": "^7.0.1",
        "@mui/material": "^7.0.1",
        "@testing-library/jest-dom": "^6.6.3",
        "@testing-library/react": "^16.3.0",
        "@testing-library/user-event": "^13.5.0",
        "@types/react": "^19.1.0",
        "@types/react-dom": "^19.1.1",
        "axios": "^1.9.0",
        "date-fns": "^4.1.0",
        "dayjs": "^1.11.13",
        "html2canvas": "^1.4.1",
        "html2pdf.js": "^0.10.3",
        "jspdf": "^3.0.1",
        "react": "^19.1.0",
        "react-dom": "^19.1.0",
        "react-hook-form": "^7.55.0",
        "react-router-dom": "^6.30.0",
        "react-scripts": "5.0.1",
        "react-to-pdf": "^2.0.0",
        "sass": "^1.86.3",
        "typescript": "^4.9.5",
        "web-vitals": "^4.2.4",
        "xlsx": "^0.18.5",
        "yup": "^1.6.1"
    },
    "scripts": {
        "start": "react-scripts start",
        "build": "react-scripts build",
        "test": "react-scripts test",
        "eject": "react-scripts eject",
        "dev": "react-scripts start",
        "electron:dev": "concurrently \"npm:start\" \"wait-on http://localhost:3000 && set ELECTRON_START_URL=http://localhost:3000&& electron .\"",
        "electron:prod": "npm run build && electron .",
        "electron:build": "electron-builder --win --x64"
    },
    "eslintConfig": {
        "extends": [
            "react-app",
            "react-app/jest"
        ]
    },
    "browserslist": {
        "production": [
            ">0.2%",
            "not dead",
            "not op_mini all"
        ],
        "development": [
            "last 1 chrome version",
            "last 1 firefox version",
            "last 1 safari version"
        ]
    },
    "devDependencies": {
        "@types/jest": "^29.5.14",
        "@types/react-router-dom": "^5.3.3",
        "@types/uuid": "^10.0.0",
        "concurrently": "^9.2.0",
        "electron": "^37.2.4",
        "electron-builder": "24.13.3",
        "wait-on": "^8.0.3"
    },
    "homepage": ".",
    "main": "electron-main.js",
    "description": "Chanitec desktop application built with React and Electron.",
    "author": "Zidor",
    "build": {
        "appId": "com.chanitec.app",
        "productName": "Chanitec",
        "files": [
            "build/**/*",
            "electron-main.js"
        ],
        "directories": {
            "buildResources": "public"
        },
        "win": {
            "target": [
                "nsis"
            ]
        },
        "extraMetadata": {
            "main": "electron-main.js"
        }
    }
}


============================================================
FILE: tsconfig.json
============================================================
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src"]
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Zidor200)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Zidor200)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
