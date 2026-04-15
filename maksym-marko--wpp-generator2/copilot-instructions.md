## wpp-generator2

> - This is a **base template** for the [WPPG-NEXT Generator](https://wppg-next.markomaksym.com.ua/)

# WordPress Plugin Development Rules

## Important Notes for CursorAI

### Repository Purpose
- This is a **base template** for the [WPPG-NEXT Generator](https://wppg-next.markomaksym.com.ua/)
- **Do NOT change prefixes** - "MXSFWN" and "mxsfwn" are placeholder strings
- **Always follow existing file structure** - maintain current organization
- **Ask before creating new files/folders** - only create when specifically requested

### Key Rules
1. **Follow existing patterns** from README.md
2. **Use existing file structure** - don't reorganize
3. **Preserve placeholder prefixes** - don't change "MXSFWN" or "mxsfwn"
4. **Ask user before creating new files/folders**
5. **Use established patterns** for new code

## Development Guidelines

### PHP Development
- Use namespaces instead of require/include functions for autoloading
- Follow PSR-4 autoloading via Composer
- Use `final` classes for main components
- Always check `defined('ABSPATH') || exit;`
- Sanitize inputs and escape outputs

### File Organization
- **Admin**: `includes/Admin/` with controllers and views
- **Frontend**: `src/frontend/` with React components
- **Gutenberg**: `src/gutenberg/` with block.json files
- **API**: Extend `AbstractRestRouteHandler`
- **Build**: Use webpack with `npm run watch/build`

### Common Patterns
```php
// Custom Post Types
PostTypeGenerator::create($postType, $labels, $properties);

// Custom Taxonomies  
TaxonomyGenerator::create($taxonomy, $postTypes, $labels, $properties);

// Meta Boxes
MetaBoxGenerator::add(['id' => 'unique-id', 'title' => 'Title']);

// REST API 
class CustomEndpoint extends AbstractRestRouteHandler {
    protected $route = '/endpoint';
    public function handleRequest($request) { /* implementation */ }
}
```

### Security & Performance
- Validate and sanitize all inputs
- Use nonces for forms/AJAX
- Check user capabilities
- Use WordPress transients for caching
- Optimize database queries

### Before Creating New Files
**Always ask the user:**
- "Should I create a new file/folder for this feature?"
- "Where should this new component be placed?"
- "Does this fit the existing structure?"

This ensures consistent development while preserving the template's role in the generator system. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Maksym-Marko) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
