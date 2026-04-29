## moodle-local-edwiserreports-free

> name: Moodle AJAX and JavaScript Integration

---
name: Moodle AJAX and JavaScript Integration
---

# Moodle AJAX and JavaScript Integration

## Purpose
Ensure robust, secure, and maintainable integration between PHP and JavaScript in Moodle.

## Instructions
- Use `$PAGE->requires->js()` to load JS modules
- Use AMD modules for all new JS code
- Use Moodle's AJAX API for server communication
- Validate and sanitize all AJAX inputs in PHP
- Return JSON responses using Moodle's output functions
- Use `sesskey` for CSRF protection in AJAX requests
- Document JS/PHP interfaces and data contracts

## Examples
```php
// File: ./amd/src/myplugin.js
define(['jquery'], function($) {
    // JS code here
});

// File: ./ajax.php
require_sesskey();
$param = required_param('id', PARAM_INT);
echo json_encode(['result' => 'ok']);
```

## Exceptions
- Legacy JS may use YUI modules
- Inline JS allowed only for critical fixes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WisdmLabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
