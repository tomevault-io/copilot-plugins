## formulario-srt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Brazilian Portuguese web application for managing Therapeutic Residential Services (SRT - Serviços Residenciais Terapêuticos) in the state of Rio de Janeiro. It's a form-based system built with vanilla JavaScript, Firebase Firestore for data storage, and EmailJS for email notifications.

## Architecture

### Frontend Structure
- **Pure HTML/CSS/JavaScript** - No build process or framework dependencies
- **Tailwind CSS** via CDN for styling
- **Firebase** (v9.23.0) for authentication and data storage (Firestore)
- **EmailJS** (v3) for sending email notifications
- **Chart.js** for data visualization in admin panel
- **SheetJS (xlsx)** (v0.18.5) for Excel export functionality
- **Sortable.js** (v1.15.0) for drag-and-drop field configuration
- **Particles.js** for animated background effects
- **AOS** (Animate On Scroll) for scroll animations

### Key Files
- `index.html` - Main registration form for new SRT entries
- `admin.html` - Administrative panel for managing entries
- `usuario.html` - User management interface
- `js/app.js` - Main form logic and data submission
- `js/admin.js` - Admin panel functionality
- `js/usuario.js` - User management logic
- `js/email.js` - Email service integration
- `js/utils.js` - Utility functions (validation, formatting)

### Data Flow
1. Users fill out multi-step form in `index.html`
2. Data is validated and saved to Firebase Firestore
3. Email notification is sent via EmailJS
4. Admins can view/manage entries in `admin.html`

## Development Commands

Since this is a static site with no build process:

```bash
# Serve locally (requires any HTTP server)
python -m http.server 8000
# or
npx http-server
# or using VS Code Live Server extension

# No build/test/lint commands - pure static files
# No package.json or dependency management - all libraries loaded via CDN
```

## Firebase Configuration

The Firebase configuration is hardcoded in JavaScript files. The project uses:
- **Firebase Auth** for admin authentication
- **Firestore** collections:
  - `srt` - Main form submissions
  - `users` - User management
  - `config` - Form field configuration

## Key Features

1. **Multi-step Form** (`index.html`)
   - Municipality information
   - SRT general data
   - Residence details with capacity tracking
   - Care team information
   - Individual resident data (dynamic)
   - Observations

2. **Admin Panel** (`admin.html`)
   - View all submissions in card format
   - Export to Excel (XLSX format)
   - Occupancy charts (Chart.js visualization)
   - Search/filter functionality
   - Dynamic form field configuration with drag-and-drop reordering
   - Firebase Auth required for access

3. **User Management** (`usuario.html`)
   - Add/edit/delete users
   - Role-based access (admin/user)
   - Status management (active/inactive)

## Important Patterns

1. **Form Validation**: Client-side validation with visual feedback (shake animation on errors)
2. **Dark Mode**: Theme toggle stored in localStorage
3. **Loading States**: Overlay with animated dots during async operations
4. **Toast Notifications**: Temporary success/error messages
5. **Responsive Design**: Mobile-first with Tailwind breakpoints

## Security Notes

- Firebase configuration is exposed in client-side code
- Authentication is handled via Firebase Auth
- Admin check: `userDoc.data().role === 'admin'`
- Email validation uses regex patterns in `utils.js`

## Common Tasks

### Adding a New Form Field
1. Navigate to admin panel (`admin.html` > Configurações tab)
2. Use the "Adicionar Campo" button to create new fields
3. Supported field types: text, number, date, select, multiselect, textarea, tel, email
4. Fields are dynamically generated from Firestore `config` collection
5. Use drag-and-drop to reorder fields (Sortable.js)

### Modifying Email Templates
- Email templates are in `js/email.js`
- Uses EmailJS with template ID `template_mc5kdip`
- Formats data into HTML tables for email body

### Updating Firebase Rules
- Ensure Firestore rules allow authenticated writes to `srt` collection
- Admin users need read/write access to `users` and `config` collections

### Working with Excel Export
- Uses SheetJS library for Excel generation
- Export function in `admin.js` creates XLSX files with all SRT data
- File naming convention: `Relatorio_SRT_DD-MM-YYYY.xlsx`

### Authentication Flow
1. Admin users authenticate via Firebase Auth
2. User role verification: `userDoc.data().role === 'admin'`
3. Session persisted in browser
4. Logout clears session and redirects to login

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thiagopvn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/thiagopvn)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
