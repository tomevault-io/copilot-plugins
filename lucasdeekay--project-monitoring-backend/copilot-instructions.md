## project-monitoring-backend

> This file contains guidelines and commands for agentic coding agents working in this repository.

# AGENTS.md

This file contains guidelines and commands for agentic coding agents working in this repository.

## Project Overview

This is a Node.js/Express backend API for a Project Monitoring & Evaluation System. It uses:
- **Runtime**: Node.js with Express.js framework
- **Database**: MySQL with raw SQL queries (no ORM)
- **Authentication**: JWT with bcryptjs for password hashing
- **File Uploads**: Multer for handling file uploads
- **Environment**: dotenv for configuration management

## Build/Development Commands

### Development
```bash
# Start development server with auto-reload
npm run dev
# or
nodemon src/server.js

# Start server without auto-reload
node src/server.js
```

### Testing
```bash
# Run tests (currently no test framework configured)
npm test

# Run specific test file (when tests are added)
npm test -- --grep "test name"
# or
mocha test/path/to/test.js
```

### Linting/Formatting
```bash
# No linting tools currently configured
# Consider adding: npm install --save-dev eslint prettier
```

## Code Style Guidelines

### Import Statements
- Use CommonJS `require()` syntax (this project uses CommonJS, not ES modules)
- Group imports in this order:
  1. Node.js built-in modules
  2. Third-party dependencies
  3. Local modules (use relative paths with `./` or `../`)
```javascript
const express = require("express");
const cors = require("cors");
const dotenv = require("dotenv");
const path = require("path");
const { testConnection } = require("./config/database");
```

### File Structure
- **Main entry point**: `src/server.js`
- **Database configuration**: `src/config/database.js`
- **Routes**: Place in `src/routes/` directory (to be created)
- **Middleware**: Place in `src/middleware/` directory (to be created)
- **Models**: Place in `src/models/` directory (to be created)
- **Controllers**: Place in `src/controllers/` directory (to be created)

### Naming Conventions
- **Files**: kebab-case for routes, camelCase for utilities (e.g., `auth-routes.js`, `databaseHelper.js`)
- **Variables**: camelCase
- **Functions**: camelCase
- **Constants**: UPPER_SNAKE_CASE
- **Database tables**: snake_case (e.g., `users`, `projects`, `feedback`)
- **Database columns**: snake_case

### Error Handling
- Use try-catch blocks for async operations
- Return consistent error response format:
```javascript
res.status(err.status || 500).json({
  success: false,
  message: err.message || "Internal server error",
  ...(process.env.NODE_ENV === "development" && { stack: err.stack }),
});
```
- Include error logging with `console.error()`

### Database Operations
- Use the database helper functions from `src/config/database.js`
- All queries should use parameterized statements to prevent SQL injection
- Example pattern:
```javascript
const { query } = require("./config/database");

const getUserById = async (id) => {
  try {
    const results = await query("SELECT * FROM users WHERE id = ?", [id]);
    return results[0];
  } catch (error) {
    console.error("Database error:", error.message);
    throw error;
  }
};
```

### API Response Format
- Use consistent JSON response format:
```javascript
// Success response
res.json({
  success: true,
  data: result,
  message: "Operation completed successfully"
});

// Error response
res.status(400).json({
  success: false,
  message: "Validation failed",
  errors: validationErrors
});
```

### Middleware Usage
- Always use CORS middleware before routes
- Use `express.json()` and `express.urlencoded({ extended: true })` for parsing
- Include request logging middleware in development
- Use authentication middleware for protected routes

### Environment Variables
- Use `.env` file for local development
- Never commit `.env` file to version control
- Required environment variables:
  - `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`
  - `PORT` (defaults to 5000)
  - `CLIENT_URL` (defaults to http://localhost:5173)
  - `NODE_ENV` (development/production)
  - `JWT_SECRET` (for authentication)

### Security Best Practices
- Always validate and sanitize user input
- Use parameterized queries for database operations
- Hash passwords with bcryptjs before storing
- Use JWT for authentication with proper expiration
- Enable CORS only for trusted origins
- Never expose sensitive information in error messages (unless in development)

### Code Comments
- Add comments for complex business logic
- Document API endpoints with expected request/response formats
- Include TODO comments for future improvements

## Testing Guidelines

When adding tests:
- Use Mocha/Chai or Jest for testing framework
- Place test files in `test/` or `tests/` directory
- Name test files with `.test.js` or `.spec.js` suffix
- Test both success and error scenarios
- Mock database connections for unit tests

## Development Workflow

1. Always test database connection before starting development
2. Use the existing database schema as reference for new tables
3. Follow the established API endpoint patterns
4. Ensure all async operations have proper error handling
5. Test endpoints with tools like Postman or curl
6. Verify database queries before implementation

## Common Patterns

### Route Structure
```javascript
const express = require("express");
const router = express.Router();
const { someControllerFunction } = require("../controllers/someController");

router.get("/", someControllerFunction);
router.post("/", someControllerFunction);
router.put("/:id", someControllerFunction);
router.delete("/:id", someControllerFunction);

module.exports = router;
```

### Controller Structure
```javascript
const { query } = require("../config/database");

const controllerFunction = async (req, res) => {
  try {
    // Validation
    // Database operation
    // Response
  } catch (error) {
    console.error("Error:", error.message);
    res.status(500).json({
      success: false,
      message: "Internal server error"
    });
  }
};

module.exports = { controllerFunction };
```

Remember: This project uses raw SQL queries, not an ORM. Always use parameterized queries and handle database errors appropriately.

---
> Source: [Lucasdeekay/project-monitoring-backend](https://github.com/Lucasdeekay/project-monitoring-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
