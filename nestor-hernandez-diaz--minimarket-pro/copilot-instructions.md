## minimarket-pro

> **Last Updated:** January 17, 2026

# AGENTS.md - Minimarket Pro

**Version:** 1.0.0  
**Last Updated:** January 17, 2026  
**Purpose:** Guide for AI coding agents working on this codebase

---

## 1. PROJECT OVERVIEW

**Minimarket Pro** is a multi-tenant SaaS platform for minimarket/retail management.

- **Tech Stack:** Vanilla JavaScript (ES6+), HTML5, Tailwind CSS, jQuery (for DataTables)
- **Architecture:** Frontend-only (static pages with mock data for now)
- **Structure:** Component-based with shared utilities
- **No Build System:** Direct HTML/CSS/JS development (CDN dependencies)

### Project Structure
```
proyecto-frontend/
├── pages/
│   ├── sucursal/         # Branch/tenant pages
│   ├── superadmin/       # Platform admin pages
│   └── cliente/          # Customer-facing pages
├── js/
│   ├── app.js            # Global utilities
│   ├── app/sidebar.js    # Sidebar navigation
│   ├── services/api.js   # API wrapper (future backend)
│   ├── utils/            # Shared utilities
│   └── sucursal/         # Branch-specific modules
├── css/
│   └── styles.css        # Design system CSS
└── assets/               # Images, fonts, etc.
```

---

## 2. BUILD/TEST/LINT COMMANDS

**No build system required.** This is a static frontend project.

### Development
- Open files directly in browser or use any local server:
  ```bash
  # Option 1: Python
  python -m http.server 8000
  
  # Option 2: Node.js http-server
  npx http-server -p 8000
  
  # Option 3: VS Code Live Server extension
  ```

### Testing
- **No automated tests** currently
- Manual testing in browser (Chrome/Firefox recommended)
- Test responsive design with DevTools (F12)

### Validation
- **HTML:** Use W3C validator (https://validator.w3.org/)
- **CSS:** Use W3C CSS validator
- **JS:** Browser console for errors (no linter configured)

### Running a Single Page
```bash
# Navigate to project root, then:
# For pages/sucursal/catalogo/categorias.html:
open proyecto-frontend/pages/sucursal/catalogo/categorias.html
# or start local server and visit http://localhost:8000/proyecto-frontend/
```

---

## 3. CODE STYLE GUIDELINES

### 3.1 JavaScript Style

**File Structure (Module Pattern):**
```javascript
/**
 * MINIMARKET PRO - Module Name
 * Archivo: module-name.js
 * Descripción: Brief description
 */

(function() {
    'use strict';
    
    // Alias jQuery if needed
    const $ = jQuery;
    
    // Variables globales
    let moduleTable;
    let currentEditingId = null;
    
    // Inicialización
    function init() {
        console.log('📁 Module initialized');
        initDataTable();
        initFormHandlers();
        loadData();
    }
    
    // Export init function
    window.initModuleName = init;
    
    // Other functions...
    
})();
```

**Naming Conventions:**
- `camelCase` for functions and variables
- `PascalCase` for constructors/classes
- `UPPER_SNAKE_CASE` for constants
- Prefix private functions with `_` (convention only)
- Use descriptive names: `createCategoryRow()` not `createRow()`

**Functions:**
- One responsibility per function
- Max ~50 lines per function
- Use early returns to avoid deep nesting
- Add JSDoc comments for complex functions:
  ```javascript
  /**
   * Formatear moneda según estándar peruano
   * @param {number} amount - Cantidad a formatear
   * @param {boolean} showSymbol - Mostrar símbolo S/
   * @returns {string} Cantidad formateada
   */
  function formatCurrency(amount, showSymbol = true) { ... }
  ```

**Error Handling:**
```javascript
// Always wrap async operations in try-catch
try {
    await api.post('/endpoint', data);
    MinimarketApp.ui.showToast('Success', 'success');
} catch (error) {
    console.error('Error:', error);
    MinimarketApp.ui.showToast('Error al guardar', 'danger');
}
```

**Async/Await (Preferred over Promises):**
```javascript
// Good
async function loadData() {
    const data = await api.get('/categories');
    processData(data);
}

// Avoid
function loadData() {
    api.get('/categories').then(data => processData(data));
}
```

### 3.2 HTML Style

**Indentation:** 4 spaces (consistent with existing files)

**Structure:**
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>[Page Title] - Minimarket Pro</title>
    
    <!-- Favicon -->
    <!-- Fonts (DM Sans REQUIRED) -->
    <!-- Icons (Phosphor) -->
    <!-- Tailwind CSS -->
    <!-- DataTables CSS (if needed) -->
    <!-- SweetAlert2 CSS -->
    <!-- Custom styles.css -->
</head>
<body class="antialiased overflow-hidden">
    <!-- Sidebar -->
    <!-- Main content -->
    
    <!-- Scripts at bottom -->
    <!-- jQuery (if using DataTables) -->
    <!-- DataTables -->
    <!-- SweetAlert2 -->
    <!-- app.js -->
    <!-- sidebar.js -->
    <!-- Module-specific JS -->
</body>
</html>
```

**Semantic HTML:** Use `<header>`, `<nav>`, `<main>`, `<section>`, `<article>` appropriately

**Accessibility:**
- Use `alt` attributes on images
- Use `aria-label` on icon-only buttons
- Ensure proper heading hierarchy (h1 → h2 → h3)

### 3.3 CSS/Tailwind Style

**DO NOT write custom CSS** unless absolutely necessary. Use:
1. Tailwind utility classes (primary)
2. Design system classes from `styles.css` (secondary)
3. Inline styles only for dynamic values

**Component Classes (from styles.css):**
- `.btn`, `.btn-primary`, `.btn-secondary`, `.btn-danger`
- `.badge`, `.badge-success`, `.badge-warning`, `.badge-danger`
- `.kpi-card`, `.kpi-icon`, `.kpi-content`, `.kpi-label`, `.kpi-value`
- `.card`, `.modal`, `.form-input`, `.form-select`, `.form-label`
- `.action-icons`, `.action-icon`, `.action-icon-edit`, `.action-icon-delete`

**Tailwind Config (Always Include):**
```javascript
tailwind.config = {
    theme: {
        extend: {
            fontFamily: { sans: ['DM Sans', 'sans-serif'] },
            colors: {
                brand: {
                    dark: '#1B2559',
                    primary: '#4318FF',
                    light: '#F4F7FE',
                    gray: '#A3AED0',
                    success: '#05CD99',
                    warning: '#FFCE20',
                    danger: '#EE5D50',
                }
            },
            boxShadow: {
                'card': '0px 18px 40px rgba(112, 144, 176, 0.12)',
                'soft': '0px 4px 12px rgba(0, 0, 0, 0.05)',
            }
        }
    }
}
```

### 3.4 Imports and Dependencies

**Load Order (Critical):**
```html
<!-- 1. Fonts (DM Sans - REQUIRED) -->
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:opsz,wght@9..40,400;9..40,500;9..40,600;9..40,700&display=swap" rel="stylesheet">

<!-- 2. Icons (Phosphor - REQUIRED) -->
<script src="https://unpkg.com/@phosphor-icons/web@2.0.3"></script>

<!-- 3. Tailwind CSS -->
<script src="https://cdn.tailwindcss.com?plugins=forms"></script>

<!-- 4. DataTables (if needed) -->
<link rel="stylesheet" href="https://cdn.datatables.net/1.13.7/css/jquery.dataTables.min.css">

<!-- 5. SweetAlert2 -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/sweetalert2@11/dist/sweetalert2.min.css">

<!-- 6. Custom styles -->
<link rel="stylesheet" href="[RUTA]/css/styles.css">

<!-- Scripts at bottom -->
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script src="https://cdn.datatables.net/1.13.7/js/jquery.dataTables.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
<script src="[RUTA]/js/app.js"></script>
<script src="[RUTA]/js/app/sidebar.js"></script>
```

**Relative Paths:**
- `pages/sucursal/page.html` → `../../` (to root)
- `pages/sucursal/catalogo/page.html` → `../../../`

---

## 4. FORMATTING AND TYPES

### 4.1 Formatting

**NO PRETTIER/FORMATTER CONFIGURED**
- Manual formatting required
- 4-space indentation (HTML/JS/CSS)
- Max line length: ~100 characters (soft limit)
- Always add trailing commas in multiline objects/arrays:
  ```javascript
  const obj = {
      name: 'test',
      value: 123,  // trailing comma
  };
  ```

### 4.2 Types (No TypeScript)

**Use JSDoc for type hints:**
```javascript
/**
 * @typedef {Object} Category
 * @property {number} id
 * @property {string} name
 * @property {string} description
 * @property {boolean} is_active
 */

/**
 * @param {Category} category
 * @returns {Array<string>}
 */
function createCategoryRow(category) { ... }
```

**Validate inputs:**
```javascript
function formatCurrency(amount, showSymbol = true) {
    if (typeof amount !== 'number') {
        console.error('formatCurrency expects number, got:', typeof amount);
        return 'S/ 0.00';
    }
    // ...
}
```

---

## 5. NAMING CONVENTIONS

### 5.1 Files
- HTML: `kebab-case.html` (e.g., `historial-ventas.html`)
- JS: `kebab-case.js` (e.g., `sales-history.js`)
- CSS: `kebab-case.css`

### 5.2 HTML IDs and Classes
- IDs: `camelCase` (e.g., `btnExportExcel`, `categoryModal`)
- Classes: Use Tailwind utilities or design system classes
- Data attributes: `data-kebab-case` (e.g., `data-category-id`)

### 5.3 JavaScript
```javascript
// Variables & functions
const categoryTable = $('#categoriesTable');
function loadCategories() { ... }

// Constants
const API_BASE_URL = '/api';
const MAX_FILE_SIZE = 2 * 1024 * 1024; // 2MB

// Private functions (convention)
function _validateForm() { ... }

// Event handlers
function handleFormSubmit(e) { ... }
function onCategoryDelete(id) { ... }
```

---

## 6. ERROR HANDLING

### 6.1 User-Facing Errors (SweetAlert2)

```javascript
// Success
MinimarketApp.ui.showToast('Categoría guardada correctamente', 'success');

// Warning
MinimarketApp.ui.showToast('El nombre es requerido', 'warning');

// Error
MinimarketApp.ui.showToast('Error al guardar la categoría', 'danger');

// Confirmation
const confirmed = await MinimarketApp.ui.confirmAction(
    '¿Eliminar categoría?',
    'Esta acción no se puede deshacer',
    'Sí, eliminar'
);
if (confirmed) { /* delete */ }
```

### 6.2 Console Logging

```javascript
// Development logs (keep for debugging)
console.log('📁 Módulo de Categorías inicializado');

// Errors (always log)
console.error('Error al guardar:', error);

// Warnings
console.warn('Deprecated function used');
```

### 6.3 Validation

**Client-side validation is REQUIRED:**
```javascript
// Form validation example
if (!data.name || data.name.trim() === '') {
    MinimarketApp.ui.showToast('El nombre es requerido', 'warning');
    $('#categoryName').focus();
    return;
}

// Email validation
if (!MinimarketApp.utils.validateEmail(email)) {
    MinimarketApp.ui.showToast('Email inválido', 'warning');
    return;
}

// File validation
if (file.size > 2 * 1024 * 1024) {
    MinimarketApp.ui.showToast('La imagen no debe superar 2MB', 'warning');
    return;
}
```

---

## 7. STANDARD PATTERNS

### 7.1 DataTables Pattern

```javascript
// Initialization
categoriesTable = $('#categoriesTable').DataTable({
    dom: 't<"flex items-center justify-between pt-4"i<"flex items-center gap-2"lp>>',
    language: {
        url: '//cdn.datatables.net/plug-ins/1.13.7/i18n/es-ES.json'
    },
    pageLength: 10,
    order: [[0, 'asc']],
    columnDefs: [
        { orderable: false, className: 'text-center', targets: -1 }
    ]
});

// Loading data
categoriesTable.clear();
mockData.forEach(item => {
    const row = createRow(item);
    categoriesTable.row.add(row);
});
categoriesTable.draw();
```

### 7.2 Modal Pattern

```javascript
// Open modal
function openModal(itemId = null) {
    currentEditingId = itemId;
    
    if (itemId) {
        // Edit mode - populate form
        const item = data.find(x => x.id === itemId);
        $('#itemName').val(item.name);
        $('#modalTitle').text('Editar Item');
    } else {
        // Create mode - reset form
        $('#itemForm')[0].reset();
        $('#modalTitle').text('Nuevo Item');
    }
    
    $('#itemModal').removeClass('hidden');
    setTimeout(() => $('#itemName').focus(), 300);
}

// Close modal
function closeModal() {
    $('#itemModal').addClass('hidden');
    currentEditingId = null;
}
```

### 7.3 Export Pattern (Excel/PDF)

```javascript
// Use ExportManager utility
function exportToExcel() {
    const excelData = data.map(item => ({
        'ID': item.id,
        'Nombre': item.name,
        'Estado': item.is_active ? 'Activo' : 'Inactivo'
    }));
    
    ExportManager.toExcel({
        data: excelData,
        fileName: 'Categorias',
        sheetName: 'Categorías',
        onSuccess: (result) => {
            MinimarketApp.ui.showToast(
                `Exportado ${result.rows} registros`, 
                'success'
            );
        }
    });
}
```

---

## 8. DESIGN SYSTEM RULES

### 8.1 NO EMOJIS in UI
**Use Phosphor icons instead:**
```html
<!-- Bad -->
<span>🥤</span>

<!-- Good -->
<i class="ph-fill ph-coffee"></i>
```

### 8.2 Color Usage
- Primary actions: `brand-primary` (#4318FF)
- Success: `brand-success` (#05CD99)
- Warning: `brand-warning` (#FFCE20)
- Danger: `brand-danger` (#EE5D50)
- Text: `brand-dark` (#1B2559)
- Background: `brand-light` (#F4F7FE)

### 8.3 Component Standards

**Buttons:**
```html
<button class="btn btn-primary">
    <i class="ph-bold ph-plus-circle"></i>
    Crear
</button>
```

**Badges:**
```html
<span class="badge badge-success">Activo</span>
<span class="badge badge-warning">Pendiente</span>
<span class="badge badge-danger">Inactivo</span>
```

**KPI Cards:**
```html
<div class="kpi-card">
    <div class="kpi-icon kpi-icon-primary">
        <i class="ph-fill ph-basket"></i>
    </div>
    <div class="kpi-content">
        <p class="kpi-label">Total Productos</p>
        <p class="kpi-value">1,234</p>
    </div>
</div>
```

---

## 9. GLOBAL UTILITIES (MinimarketApp)

**Available utilities from `app.js`:**

```javascript
// Formatting
MinimarketApp.utils.formatCurrency(1234.56) // "S/ 1,234.56"
MinimarketApp.utils.formatDate('2024-01-15') // "15/01/2024"
MinimarketApp.utils.formatTime(new Date())   // "14:30"

// Validation
MinimarketApp.utils.validateRUC('12345678901')  // boolean
MinimarketApp.utils.validateDNI('12345678')     // boolean
MinimarketApp.utils.validateEmail('test@test.com') // boolean

// Tax calculations
MinimarketApp.utils.calculateIGV(118) 
// { subtotal: 100, igv: 18, total: 118 }
MinimarketApp.utils.addIGV(100) // 118

// UI helpers
MinimarketApp.ui.showToast('Message', 'success')
await MinimarketApp.ui.confirmAction('Title', 'Text', 'Confirm')
MinimarketApp.ui.toggleElement('#modal', true) // show

// Session
MinimarketApp.session.getCurrentUser()
MinimarketApp.session.getAuthToken()
MinimarketApp.session.logout()
```

---

## 10. REFERENCES

- **Design System:** `proyecto-frontend/DESIGN_SYSTEM.md` (comprehensive guide)
- **Page Pattern:** `proyecto-frontend/PATRON_PAGINAS_SUCURSAL.md` (detailed patterns)
- **Template:** `proyecto-frontend/pages/sucursal/_template.html` (boilerplate)
- **Example Module:** `proyecto-frontend/js/sucursal/categories.js` (reference implementation)

---

## 11. CRITICAL RULES

1. **NEVER use emojis** in user-facing interfaces (use Phosphor icons)
2. **ALWAYS load DM Sans font** (design system requirement)
3. **ALWAYS use try-catch** for async operations
4. **ALWAYS show user feedback** (toast/modal) for actions
5. **ALWAYS validate user inputs** before processing
6. **NEVER commit with console.log** for production (keep for dev)
7. **ALWAYS use Spanish** for user-facing text (code comments can be English)
8. **ALWAYS use SweetAlert2** for notifications (not native alerts)
9. **ALWAYS follow the DataTables DOM pattern** for consistency
10. **ALWAYS test responsive design** (mobile, tablet, desktop)

---

## 12. COMMON TASKS

### Creating a New Page
1. Copy `_template.html`
2. Update `<title>`, meta tags, breadcrumbs
3. Adjust relative paths (../../ or ../../../)
4. Implement content using design system components
5. Create corresponding JS module in `js/sucursal/`
6. Add page to sidebar navigation (if needed)

### Adding a New Module
1. Create `js/sucursal/module-name.js` using module pattern
2. Export init function: `window.initModuleName = init;`
3. Call from HTML: `<script>initModuleName();</script>`
4. Follow existing patterns (categories.js as reference)

### Debugging
1. Check browser console (F12)
2. Verify all scripts loaded (Network tab)
3. Check for 404s on assets
4. Validate HTML/CSS if layout issues
5. Test with mock data first

---

**End of AGENTS.md** | For updates, see `DESIGN_SYSTEM.md` and `PATRON_PAGINAS_SUCURSAL.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nestor-Hernandez-Diaz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Nestor-Hernandez-Diaz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
