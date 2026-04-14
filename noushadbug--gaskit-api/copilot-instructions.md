## gaskit-api

> **Project: GASKit-API**

**Project: GASKit-API**

**Focus:** API-First Framework for Google Apps Script

**Overview:**
GASKit is a lightweight, modular framework designed to simplify building robust APIs in Google Apps Script. It provides an Express-like routing system, middleware support, and standardized request/response handling.

**Core Components:**
- `GASKit.js`: The core module container (Dependency Injection).
- `GASKit_api.js`: The API framework module.
- `App.js`: Main entry point and configuration.

**Features:**
- **Router:** Handle `GET`, `POST`, `PUT`, `DELETE` with path parameters (e.g., `/users/:id`).
- **Middleware:** Global and route-specific middleware (like Express).
- **Request/Response Wrappers:** Simplified access to parameters, body, and standardized JSON responses.
- **Config-Driven:** Initialize and configure modules centrally.

**Usage:**

1. **Initialize:**
   ```javascript
   var app = Kit.init({
     use: ['api']
   }).get('api');
   ```

2. **Define Routes:**
   ```javascript
   app.get('/hello', function(req, res) {
     res.json({ message: 'Hello World' });
   });

   app.post('/data', function(req, res) {
     var data = req.body;
     res.status(201).json({ received: data });
   });
   ```

3. **Wire up Triggers:**
   ```javascript
   function doGet(e) { return app.doGet(e); }
   function doPost(e) { return app.doPost(e); }
   ```

**Testing:**
- Run `runTests()` in `tests.js` to verify functionality in the Apps Script Editor.

**Note:**
This project has been refactored to focus solely on API support. Previous UI/Web modules have been removed to ensure a clean, focused utility solution.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NoushadBug)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NoushadBug)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
