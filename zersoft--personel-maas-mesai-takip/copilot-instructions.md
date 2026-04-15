## personel-maas-mesai-takip

> All security rules are in AGENTS.md under "Security Rules (MANDATORY for all agents)". Follow them strictly.

# Cursor Rules - OYS Project

## IMPORTANT: Read AGENTS.md first
All security rules are in AGENTS.md under "Security Rules (MANDATORY for all agents)". Follow them strictly.

## Quick Security Checklist

When creating or modifying PHP files:

- [ ] Every POST form has `<?php echo csrfField(); ?>` inside it
- [ ] Every *_islem.php file has `verifyCsrfToken()` for POST requests
- [ ] No GET requests for delete/update/insert operations
- [ ] All user output escaped with `escape()` or `htmlspecialchars()`
- [ ] All SQL uses PDO prepared statements (never string concatenation)
- [ ] Page has `requireRole('user')` or `requireRole('admin')` at top
- [ ] Errors logged with `error_log()`, generic message shown to user
- [ ] Money input parsed with `parseMoney()`, displayed with `formatMoney()`
- [ ] Redirects use `safeRedirect()` function
- [ ] Directory creation uses `0700` permissions, never `0777`
- [ ] No test/debug files created

## JavaScript Delete/Restore Pattern

```javascript
// Use existing helpers from main.js:
postDeleteForm('target_islem.php', recordId);   // for deletes
postRestoreForm('target_islem.php', recordId);  // for restores
```

## New Form Template

```php
<form action="target_islem.php" method="POST">
    <?php echo csrfField(); ?>
    <input type="hidden" name="action" value="insert">
    <!-- form fields -->
    <button type="submit" class="btn btn-primary">Kaydet</button>
</form>
```

## New Processing File Template

```php
<?php
// Session block (copy from existing *_islem.php)
require_once '../config/db.php';
require_once '../includes/functions.php';
require_once '../includes/auth.php';
requireRole('user');

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    verifyCsrfToken();
}

try {
    // Handle actions...
} catch (Throwable $e) {
    error_log("Hata: " . $e->getMessage());
    safeRedirect('target.php?error=' . urlencode('Bir hata olustu.'));
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zersoft) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
