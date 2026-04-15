## real-state-crm

> * Clean Architecture must be respected. Entities and business logic remain isolated from infrastructure and frameworks. Application layer orchestrates logic, Infrastructure layer handles external concerns. No direct coupling between layers. Separation of concerns is mandatory.

# GEMINI - Project Rules & Architecture

## Architecture (Clean Architecture)

* Clean Architecture must be respected. Entities and business logic remain isolated from infrastructure and frameworks. Application layer orchestrates logic, Infrastructure layer handles external concerns. No direct coupling between layers. Separation of concerns is mandatory.

## Tech Stack

* **Backend**: Node.js, NestJS (minimal API Architecture), TypeScript, Prisma, PostgreSQL
* **Frontend**: React, Material UI (MUI) only, Framer Motion (transitions)
* **HTTP**: Axios (not HttpService)
* **Testing**: Jest
* **DevOps**: Docker, Docker Compose
* **Docs**: OpenAPI/Swagger
* **Code Quality**: ESLint, Prettier
* **Package Manager**: npm CLI
* **Version Control**: GIT
* **Environment**: .env file
* **Logger**: NestJS Logger

## Core Rules

### Controllers (Application Layer)

* All routes driven by Controllers
* Each Controller = one Domain Entity
* Base routes: GET /{resource}, GET /{resource}/{id}, POST /{resource}, PUT /{resource}/{id}, DELETE /{resource}/{id}
* All "FindAll" routes must filter by tenant\_id
* Controllers handle:

  * User logic, roles, levels
  * Authentication and authorization of routes
  * Middleware application when necessary
  * Repassing routes to Services (never hold business logic directly)
* Controllers can accept filters for list/pagination queries. Filters:

  * Define which fields should be returned
  * Define which `where` conditions must be applied
* Filters are passed via jQuery payloads, forwarded to the Service layer, processed in the Repository, and converted into database-compatible query structures
* DTOs in `Application/DTOs/` - always create Response DTOs, conceal tenant\_id
* Validators in `Application/Validators` must validate DTOs before Domain. Validation errors must be returned directly
* Controllers must be created with all NestJS annotations correctly

### Services & Repositories

* **Service**: Orchestrates business logic, forwards validated DTOs, applies logical conditions, and passes filters to Repositories
* **Repository**: Database rules (CRUD + PURGE), including interpretation of filters passed by Services
* CRUD returns: CREATE/READ/UPDATE → Entity, DELETE → boolean (soft delete)
* PURGE → boolean (permanent delete, cascade, not accessible via controllers)
* All entities need tenant\_id (except root entities)
* UUID for primary keys

### Database

* Prisma for all DB operations
* Context manages connections, transactions, auditing
* Auto-generate UUID at creation
* Tenancy logic applied to all entities with tenant\_id
* Migration and Seeding patterns must be defined and followed
* Custom database queries and transactions can be defined within context

### Code Standards

* **No comments** in code
* **No dependency injection** for Repositories/Controllers
* **No interface contracts** (Clean Architecture approach)
* **Early returns** pattern
* **Switch statements** over if-else chains
* **Avoid nesting** methods/functions
* **Avoid class instantiation** within methods
* **No magic numbers/strings** - use named constants
* **Use SOLID principles** to guide design
* **Use meaningful names** for variables, functions, classes
* **Respect spacing and indentation**
* **Avoid unnecessary whitespace and blank lines**
* **Use consistent naming conventions**
* **Method structure**:

  ```typescript
  function methodName(parameters) {
    <constants and variables>
    
    <conditions and logic>
    
    <return statement>
  }
  ```
* **Avoid intermediate constants** when not necessary:

  ```typescript
  // Bad
  const aConst = object.GetValue();
  const bConst = aConst.GetAnotherValue();

  // Good
  const finalConst = object.GetValue().GetAnotherValue();
  ```

### Logging

* Use NestJS Logger in every catch block
* Use Logger at Repositories and Services
* Log errors with stack trace
* Don't throw errors, log them instead

### Testing

* Jest only
* Create test files for every new controller
* Purge created entities after tests
* Add detail to screen UI log

### Documentation

* Each route needs OpenAPI documentation
* Include: entity description, attributes, relationships, examples

### Environment & Files

* Use existing .env file (don't create new ones)
* Don't delete files under /public without permission
* Windows 11 PowerShell terminal
* Don't create new folders without approval

### Dependencies

* Only install from Project Stack
* Use latest stable versions
* Prefer native modules over third-party
* Avoid deprecated packages
* Prefer package manager's built-in commands over manual installation

---

## Frontend Rules

### Layout

* Always responsive-first (mobile → desktop)
* Sidebar fixed left (navigation)
* Header fixed top with user info/actions
* Content centered with whitespace allowed
* Use Grid/Flexbox for structure, avoid deep nesting

### Design System

* Base: Material UI (MUI) as primary component library
* Animation: Framer Motion for transitions
* Theme: follow Google Material palette (light/dark mode support)
* Palette: neutral base (white, gray, black) + semantic highlights (blue=primary, green=success, red=error, amber=warning)
* Typography: use modern, legible fonts (Roboto, Inter, or MUI defaults)
* Clear hierarchy between headings, subheadings, and body
* Maintain balance using whitespace and consistent grid spacing

### Components

* **AppBar**: fixed top navigation with logo, user avatar, quick actions
* **Drawer/Sidebar**: vertical navigation menu, collapsible in mobile, icons + labels
* **Card/Paper**: lightweight surfaces, subtle elevation, rounded corners, consistent spacing
* **Button**: contained, outlined, text variants. Hover + focus animations active. Support icons
* **Fab (Floating Action Button)**: only for primary CTAs in mobile
* **Chip/Badge**: small labels or notifications
* **Table/DataGrid**: must support pagination, sorting, searching, and collapse in mobile
* **Dialog/Modal**: clean, centered, clear CTA separation
* **Snackbar/Toast**: short feedback, auto-dismiss with manual override
* **Forms**:

  * TextField, Select, Checkbox, Switch, RadioGroup
  * Labels always visible
  * Inline error feedback
  * Clear success state indicators
* **Typography**:

  * h1: `2xl` desktop, `xl` mobile
  * h2: `xl` desktop, `lg` mobile
  * body: `base` default, `sm` captions
* **List/Accordion**: smooth expand/collapse animations, no heavy borders
* **Tabs**: for switching contexts, with clear active states

### UI/UX Standards

* Minimalist design, avoid clutter
* Consistency across devices
* All actions must provide feedback (loading spinners, hover states, disabled states)
* Transitions must be smooth, < 300ms
* Icons: Material Icons or Lucide
* Accessibility: ARIA labels mandatory, maintain color contrast AA+
* Mobile-first design: components scale naturally upward

### Implementation Rules

* Always use semantic HTML elements (`header`, `nav`, `main`, `aside`, `section`, `footer`)
* Use Material UI components as the default choice
* Avoid overriding MUI styles unless required
* Centralize theming with `ThemeProvider`
* Dark mode must be supported
* Use motion responsibly, guide attention without decoration
* Avoid unused imports
* All components must be reusable, stateless where possible
* Use hooks for UI logic separation
* Business logic never leaks into frontend components

---

# Front end enhancements
# Mobile-First Front-End Development with React, Vite, and Material-UI

Building a modern front-end with React involves choosing the right tools and practices. Vite is used here as a fast development build tool, and Material-UI (MUI) provides ready-made React components following Material Design. This guide covers setup, theming, responsive layout, and common UI patterns. The goal is to create a maintainable, performant UI that works well on all devices starting from mobile screens.

## Project Setup with Vite and React

Set up a new React project using Vite for a fast development experience. Make sure you have a recent version of Node.js installed, then run the following commands:

```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
```

This creates a React app in `my-app` with Vite. Next, install Material-UI and its styling engine:

```bash
npm install @mui/material @emotion/react @emotion/styled
```

Also install React Router for navigation:

```bash
npm install react-router-dom
```

After installation, run `npm run dev` to start the development server with hot module replacement. Vite watches changes instantly, which speeds up development.

## Project Structure and Tooling

Organize your project files to keep code maintainable. A common structure might look like:

```text
src/
├── assets/      # images, fonts, icons
├── components/  # reusable UI components (Buttons, Headers, etc.)
├── pages/       # page components (views or routes)
├── theme/       # MUI theme customization
├── App.jsx      # main app component
└── index.jsx    # application entry point
```

- **assets/**: Static resources such as images or SVG icons.  
- **components/**: Generic or shared UI components (e.g., custom buttons, cards).  
- **pages/**: Components corresponding to routes (e.g., `HomePage.jsx`, `AboutPage.jsx`).  
- **theme/**: Define and export your MUI theme here.  
- **App.jsx**: Use this to define layout (layouts, routing, and theme providers).  
- **index.jsx**: Renders `<App />` inside `<BrowserRouter>` and applies global providers.

Using consistent folders and naming conventions helps both developers and automated tools locate and update parts of the UI.

## Material-UI Theming and Global Styles

Create a global theme to define colors, typography, and other style variables. Wrap the app in MUI’s `ThemeProvider` so that all components use the theme.

```jsx
// theme/theme.js
import { createTheme } from '@mui/material/styles';

const theme = createTheme({
  palette: {
    primary: { main: '#1976d2' },    // blue
    secondary: { main: '#dc004e' },  // pinkish
    background: { default: '#f5f5f5' },
  },
  typography: {
    fontFamily: 'Roboto, Arial, sans-serif',
  },
});

export default theme;
```

```jsx
// index.jsx
import React from 'react';
import ReactDOM from 'react-dom';
import { ThemeProvider } from '@mui/material/styles';
import CssBaseline from '@mui/material/CssBaseline';
import { BrowserRouter } from 'react-router-dom';
import App from './App';
import theme from './theme/theme';

ReactDOM.render(
  <React.StrictMode>
    <ThemeProvider theme={theme}>
      <CssBaseline /> {/* Resets and global styles */}
      <BrowserRouter>
        <App />
      </BrowserRouter>
    </ThemeProvider>
  </React.StrictMode>,
  document.getElementById('root')
);
```

Key points:
- Use `createTheme` to customize colors and typography.  
- Wrap `<App>` with `<ThemeProvider>` and include `<CssBaseline />` for a consistent style reset.  
- Define your `primary` and `secondary` colors for branding, and any other theme overrides as needed.

## Layout Components: Box, Container, and Grid

Material-UI provides layout components to structure your pages:

- **Container**: Centers content horizontally and sets a maximum width. Ideal for wrapping main page content.  
- **Box**: A generic container (like a `div`) with extra props for spacing, display, and more.  
- **Grid**: Implements a 12-column responsive grid system for rows and columns.  

Example of using `Container` and `Box`:

```jsx
import React from 'react';
import { Box, Container, Typography } from '@mui/material';

function ExampleSection() {
  return (
    <Container maxWidth="md">
      <Box sx={{ bgcolor: 'background.paper', p: 2, my: 2, borderRadius: 1 }}>
        <Typography variant="h5">Section Title</Typography>
        <Typography>This is a section wrapped in a Box inside a Container.</Typography>
      </Box>
    </Container>
  );
}
```

For multi-column layouts, use `Grid`:

```jsx
import React from 'react';
import { Grid, Paper } from '@mui/material';

function TwoColumnLayout() {
  return (
    <Grid container spacing={2}>
      <Grid item xs={12} md={8}>
        <Paper sx={{ p: 2 }}>Main Content</Paper>
      </Grid>
      <Grid item xs={12} md={4}>
        <Paper sx={{ p: 2 }}>Sidebar</Paper>
      </Grid>
    </Grid>
  );
}
```

## Styling with the sx Prop and Responsive Design

Use the `sx` prop to apply custom styles directly on MUI components:

```jsx
<Box
  sx={{
    display: 'flex',
    flexDirection: { xs: 'column', md: 'row' },
    alignItems: 'center',
    justifyContent: 'space-between',
    p: 2,
    backgroundColor: 'background.default',
  }}
>
  <Typography sx={{ mb: { xs: 1, md: 0 } }}>Responsive text</Typography>
  <Button variant="contained">Click Me</Button>
</Box>
```

- Start with styles for small screens (`xs`) and override for larger ones (`md`).  
- Use theme-aware spacing and colors (`p: 2`, `backgroundColor: 'background.default'`).  

## Common UI Components and Patterns

- **Buttons**: `<Button variant="contained" color="primary">`.  
- **AppBar and Toolbar**: `<AppBar><Toolbar>...`.  
- **Typography**: `<Typography variant="h1">`.  
- **TextField**: `<TextField label="Email" fullWidth />`.  
- **Icons**: from `@mui/icons-material`.

Example AppBar:

```jsx
<AppBar position="static">
  <Toolbar>
    <Typography variant="h6" sx={{ flexGrow: 1 }}>My App</Typography>
    <Button color="inherit">Login</Button>
  </Toolbar>
</AppBar>
```

## Routing and Navigation

Use React Router:

```jsx
<Routes>
  <Route path="/" element={<HomePage />} />
  <Route path="/about" element={<AboutPage />} />
  <Route path="*" element={<NotFoundPage />} />
</Routes>
```

## State Management and Hooks

Example with `useState`:

```jsx
const [name, setName] = useState('');
<TextField label="Name" value={name} onChange={(e) => setName(e.target.value)} />
```

## Responsive Grid and Mobile-First Approach

```jsx
<Grid container spacing={2}>
  <Grid item xs={12} sm={6} md={4}><Paper>Item 1</Paper></Grid>
  <Grid item xs={12} sm={6} md={4}><Paper>Item 2</Paper></Grid>
  <Grid item xs={12} sm={6} md={4}><Paper>Item 3</Paper></Grid>
</Grid>
```

## Accessibility and Internationalization

- Use semantic HTML (`Typography` with correct variants).  
- Provide labels (`aria-label`, `alt` on images).  
- Ensure keyboard navigation works.  

```jsx
<IconButton aria-label="Open menu"><MenuIcon /></IconButton>
```

## Performance and Optimization

- Vite build is optimized.  
- Lazy load routes with `React.lazy`.  
- Use `React.memo`, `useMemo`, `useCallback`.  
- Optimize images.  

## Testing and Code Quality

- Unit tests with Jest and React Testing Library.  
- E2E tests with Cypress or Playwright.  
- ESLint + Prettier.  
- TypeScript or PropTypes for safety.  

## Summary of Best Practices

- **Mobile-first design**.  
- **Reusable components**.  
- **Consistent styling** with theme and `sx`.  
- **Semantic markup**.  
- **Performance awareness**.  
- **Accessibility compliance**.  

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/demetriomjr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/demetriomjr)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
