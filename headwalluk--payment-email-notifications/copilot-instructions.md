## payment-email-notifications

> **Purpose:** Core coding standards and patterns for WordPress plugin development

# WordPress Plugin Development Guidelines

**Version:** 1.6.0  
**Purpose:** Core coding standards and patterns for WordPress plugin development  
**Portability:** Copy this file to any WordPress plugin project  
**Last Updated:** 31 January 2026

**📚 Detailed Patterns:** See [`dev-notes/patterns/README.md`](../dev-notes/patterns/README.md) for implementation details

---

## Core Principles

1. **Follow WordPress Coding Standards** - Verify with phpcs/phpcbf
2. **Use Modern PHP** - PHP 8.0+ features (type hints, union types, nullable types)
3. **Write Self-Documenting Code** - Clear naming reduces need for comments
4. **Security First** - Always sanitize input, escape output, verify nonces
5. **Keep It Simple** - Favor clarity over cleverness

---

## Protected Directories

### Power Plugins Licence Controller (`pwpl/`)

**Rule:** If a `pwpl/` directory exists in the plugin's base directory, **DO NOT ALTER** any files within it.

**Purpose:** Contains the Power Plugins licence management system for commercial distribution.

**Integration:** The main plugin file must include the licence controller early:

```php
// Power Plugins licence control.
require_once __DIR__ . '/pwpl/pwpl.php';
```

**Important:**
- Never modify, refactor, or "improve" code in `pwpl/`
- Do not update coding standards in this directory
- Do not include `pwpl/` files in PHPCS checks
- Treat this directory as a sealed third-party dependency

---

## File Structure & Naming

### Directory Organization

```
example-plugin/
├── example-plugin.php        # Main plugin file
├── constants.php             # Plugin constants
├── functions-private.php     # Private/internal functions
├── phpcs.xml                 # Code standards configuration
├── includes/                 # Core classes
│   ├── class-plugin.php
│   ├── class-settings.php
│   └── class-admin-hooks.php
├── admin-templates/          # Admin template files
├── templates/                # Public template files
├── assets/
│   ├── admin/               # Admin CSS/JS
│   └── public/              # Public CSS/JS
├── dev-notes/               # Development documentation
└── languages/               # Translation files
```

### File Naming Conventions

- **Class files:** \`class-{classname}.php\` (lowercase with hyphens)
- **Template files:** \`{template-name}.php\` (lowercase with hyphens)
- **Asset files:** \`{descriptive-name}.{ext}\` (lowercase with hyphens)

---

## PHP Standards

### Namespaces & Classes

```php
namespace Example_Plugin;

defined( 'ABSPATH' ) || die();

class Settings {
    // No prefix needed - namespace handles it
}
```

**❌ Do NOT use \`declare(strict_types=1);\`**

WordPress and WooCommerce do not use strict types. Using it causes type errors when WordPress passes strings where you expect ints, breaks hook/filter interoperability, and fails with WooCommerce CRUD methods.

### Class Organization

**Order:**
1. Properties (public → protected → private)
2. \`__construct()\`
3. Methods (public → protected → private)

### Type Hints & Documentation

```php
/**
 * Brief description.
 *
 * @since 1.0.0
 *
 * @param string $param1 Description.
 * @param int    $param2 Description.
 *
 * @return array<string, mixed> Description.
 */
public function example_function( string $param1, int $param2 ): array {
    // Implementation
}
```

### Function Structure - Single-Entry Single-Exit (SESE)

**Rule:** Functions should have one return statement at the end.

**Why:** Easier debugging and fault tracing - you can see all logic paths that affect the final value.

```php
// ✅ Good - Single exit point
function get_value(): string {
    $result = null;
    
    if ( condition_a() ) {
        $result = 'value_a';
    }
    
    if ( is_null( $result ) && condition_b() ) {
        $result = 'value_b';
    }
    
    if ( is_null( $result ) ) {
        $result = 'default';
    }
    
    return $result;
}

// ❌ Avoid - Multiple exit points obscure which path executed
function get_value(): string {
    if ( condition_a() ) {
        return 'value_a';
    }
    
    if ( condition_b() ) {
        return 'value_b';
    }
    
    return 'default';
}
```

### Constants & Magic Values

**Rule:** All magic strings and numbers must be defined as constants in \`constants.php\`.

**Exception:** Translatable text strings use \`__()\` or \`_e()\` directly.

```php
// constants.php
namespace Example_Plugin;

// Default values - prefix with DEF_
const DEF_CHECKOUT_ERROR_MESSAGE = 'Item no longer available.';

// wp_options keys - prefix with OPT_
const OPT_TIME_FORMAT = 'example_plugin_time_format';

// Magic numbers
const MAX_SLOTS_PER_DAY = 20;
```

### Boolean Options - Handle Multiple Formats

Use \`filter_var()\` with \`FILTER_VALIDATE_BOOLEAN\` to handle all boolean formats:

```php
// ✅ Good - handles '1', 'yes', 'on', true, etc.
$enabled = (bool) filter_var( get_option( OPT_ENABLED, false ), FILTER_VALIDATE_BOOLEAN );

// ❌ Bad - only checks specific string
if ( 'yes' === get_option( OPT_ENABLED ) ) { }
```

---

## Date/Time Handling

**Rule:** Store date/time as human-readable strings in format \`Y-m-d H:i:s T\`, NOT Unix timestamps.

```php
/**
 * Get current timestamp in human-readable format.
 */
function get_now_formatted( string $format = 'Y-m-d H:i:s T' ): string {
    $now = new \DateTime( 'now', wp_timezone() );
    return $now->format( $format );
}

// ✅ Good - human-readable
update_option( 'plugin_last_sync', get_now_formatted() );
// Stores: "2025-07-16 19:33:23 BST"

// ❌ Bad - Unix timestamp
update_option( 'plugin_last_sync', time() );
```

**Why:** Easier troubleshooting, clear timezone info, better logging. Timestamps are easier to compare but human-readable format prioritizes debuggability.

---

## Security Essentials

### 1. Sanitize Input

```php
$id    = isset( $_POST['id'] ) ? absint( $_POST['id'] ) : 0;
$email = isset( $_POST['email'] ) ? sanitize_email( wp_unslash( $_POST['email'] ) ) : '';
$text  = isset( $_POST['text'] ) ? sanitize_text_field( wp_unslash( $_POST['text'] ) ) : '';
```

### 2. Escape Output

```php
echo esc_html( $user_text );
echo esc_url( $link );
echo esc_attr( $attribute_value );
```

### 3. Verify Nonces

```php
// In forms
wp_nonce_field( 'plugin_action', 'plugin_nonce' );

// In handlers
if ( ! wp_verify_nonce( $_POST['plugin_nonce'], 'plugin_action' ) ) {
    wp_die( 'Security check failed' );
}

// After verifying, disable phpcs warning
// phpcs:disable WordPress.Security.NonceVerification.Missing -- Nonce verified above
$data = isset( $_POST['data'] ) ? sanitize_text_field( wp_unslash( $_POST['data'] ) ) : '';
// phpcs:enable
```

### 4. Check Capabilities

```php
if ( ! current_user_can( 'manage_options' ) ) {
    wp_die( 'Insufficient permissions' );
}
```

### 5. Prepare Database Queries

```php
$wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}custom_table WHERE id = %d",
        $id
    )
);
```

---

## WordPress Integration

### Hook Registration

Register all hooks in main Plugin class:

```php
class Plugin {
    public function run(): void {
        add_action( 'admin_menu', array( $this, 'add_menu_items' ) );
        add_action( 'admin_enqueue_scripts', array( $this->admin_hooks, 'enqueue_assets' ) );
    }
}
```

### AJAX Endpoints

```php
public function ajax_handle_request(): void {
    check_ajax_referer( 'example_plugin_nonce', 'nonce' );
    
    if ( ! current_user_can( 'edit_posts' ) ) {
        wp_send_json_error( array( 'message' => __( 'Insufficient permissions.', 'example-plugin' ) ) );
    }
    
    $id = isset( $_POST['id'] ) ? absint( $_POST['id'] ) : 0;
    
    if ( ! $id ) {
        wp_send_json_error( array( 'message' => __( 'Invalid ID.', 'example-plugin' ) ) );
    }
    
    $result = $this->process_data( $id );
    wp_send_json_success( array( 'data' => $result ) );
}
```

---

## Error Handling with WP_Error

```php
function create_subscription( array $data ) {
    if ( empty( $data['user_id'] ) ) {
        return new \WP_Error(
            'missing_user_id',
            __( 'User ID is required.', 'example-plugin' )
        );
    }
    
    $post_id = wp_insert_post( $post_data );
    
    if ( is_wp_error( $post_id ) ) {
        return $post_id;
    }
    
    return $post_id;
}

// Usage
$result = create_subscription( $data );
if ( is_wp_error( $result ) ) {
    error_log( 'Failed: ' . $result->get_error_message() );
    return false;
}
```

---

## WooCommerce Integration

### Declare HPOS Compatibility

```php
add_action(
    'before_woocommerce_init',
    function () {
        if ( class_exists( \Automattic\WooCommerce\Utilities\FeaturesUtil::class ) ) {
            \Automattic\WooCommerce\Utilities\FeaturesUtil::declare_compatibility(
                'custom_order_tables',
                __FILE__,
                true
            );
        }
    }
);
```

### HPOS Data Access Rule

**❌ Never use \`get_post_meta()\` for Order data**  
**✅ Always use WC_Order methods**

```php
// ❌ Bad - Breaks in HPOS
$email = get_post_meta( $order_id, '_billing_email', true );

// ✅ Good - HPOS Compatible
$order = wc_get_order( $order_id );
$email = $order->get_billing_email();
$order->update_meta_data( 'custom_key', $value );
$order->save();
```

**📚 More WooCommerce patterns:** [`dev-notes/patterns/woocommerce.md`](../dev-notes/patterns/woocommerce.md)

---

## Common Patterns

### Global Variable Pattern

```php
// In main plugin file
global $example_plugin_instance;
$example_plugin_instance = new Example_Plugin\Plugin();
$example_plugin_instance->run();

// Access anywhere
function get_plugin_instance(): Plugin {
    global $example_plugin_instance;
    return $example_plugin_instance;
}
```

### Lazy Loading

```php
class Plugin {
    private ?Admin_Hooks $admin_hooks = null;
    
    public function get_admin_hooks(): Admin_Hooks {
        if ( is_null( $this->admin_hooks ) ) {
            $this->admin_hooks = new Admin_Hooks();
        }
        return $this->admin_hooks;
    }
}
```

**Exception:** Settings class must be instantiated early (before \`admin_init\`).

### No Inline HTML in Functions or Templates

**Rule:** All template files must be code-first using `printf()` or `echo`. Never mix inline HTML with PHP snippets.

**Why:** Prevents whitespace bleeding into attributes/values, eliminates formatting bugs, improves JIT compilation, easier debugging.

```php
// ❌ Bad - inline HTML with PHP snippets
<div class="wrapper">
    <button 
        data-params='
        <?php
        echo esc_attr(
            wp_json_encode(
                array(
                    'action' => 'my_action',
                )
            )
        );
        ?>
        '
    >
        <?php esc_html_e( 'Click Me', 'plugin' ); ?>
    </button>
</div>

// ✅ Good - code-first with printf/echo
<?php
printf(
    '<div class="wrapper"><button data-params="%s">%s</button></div>',
    esc_attr( wp_json_encode( array( 'action' => 'my_action' ) ) ),
    esc_html__( 'Click Me', 'plugin' )
);
?>
```

**Exception:** None. All files in `admin-templates/` and `templates/` must use this pattern.

---

## JavaScript Patterns

### Class-Based Selectors

```javascript
// ✅ Good - reusable
document.querySelector('.plugin-calendar');

// ❌ Avoid IDs (except unique admin elements)
document.getElementById('some-element');
```

### No Inline JavaScript

```php
<!-- ❌ Bad - inline JS -->
<script>
jQuery('#element').on('click', function() { });
</script>

<!-- ✅ Good - separate files -->
<!-- Load via wp_enqueue_script() -->
```

### Button Class Required

```php
<!-- ✅ Good - includes button class -->
<button type="button" class="plugin-action button">Click Me</button>
```

**📚 More JavaScript patterns:** [`dev-notes/patterns/javascript.md`](../dev-notes/patterns/javascript.md)

---

## Code Quality

### PHPCS Commands

```bash
phpcs              # Check all files
phpcs includes/    # Check specific files
phpcbf             # Auto-fix issues
```

### PHPCS Configuration

**📚 Setup guide:** [`dev-notes/workflows/code-standards.md`](../dev-notes/workflows/code-standards.md)

---

## Git Workflow

**📚 Complete workflow guide:** [`dev-notes/workflows/commit-to-git.md`](../dev-notes/workflows/commit-to-git.md)

### Commit Message Format

```
type: brief description

- Detailed point 1
- Detailed point 2
```

**Types:** \`feat:\` \`fix:\` \`chore:\` \`refactor:\` \`docs:\` \`style:\` \`test:\`

### Pre-Commit Checklist

1. Run `phpcs` to check for violations
2. Run `phpcbf` to auto-fix issues
3. Run `phpcs` again to verify
4. Stage and commit changes

---

## Advanced Patterns

**📚 See \`dev-notes/\` for detailed implementation guides:**

- **[`patterns/admin-tabs.md`](../dev-notes/patterns/admin-tabs.md)** - Hash-based tabbed navigation
- **[`patterns/caching.md`](../dev-notes/patterns/caching.md)** - Transients API, rate limiting
- **[`patterns/database.md`](../dev-notes/patterns/database.md)** - Custom tables, migrations
- **[`patterns/javascript.md`](../dev-notes/patterns/javascript.md)** - Modern JS patterns, AJAX
- **[`patterns/settings-api.md`](../dev-notes/patterns/settings-api.md)** - Settings registration, meta boxes
- **[`patterns/templates.md`](../dev-notes/patterns/templates.md)** - Template loading with overrides
- **[`patterns/woocommerce.md`](../dev-notes/patterns/woocommerce.md)** - HPOS, product tabs

---

## Translation Ready

```php
__( 'Text to translate', 'example-plugin' );
_e( 'Text to echo', 'example-plugin' );
esc_html__( 'Text to translate and escape', 'example-plugin' );
esc_html_e( 'Text to echo and escape', 'example-plugin' );
```

---

## Key Takeaways

✅ **DO:**
- Use type hints and return types
- Follow WordPress Coding Standards
- Sanitize input, escape output
- Document all functions
- Use namespaces for classes
- Store dates as human-readable strings
- Test with phpcs regularly

❌ **DON'T:**
- Use \`declare(strict_types=1);\`
- Skip nonce verification
- Trust user input
- Use \`get_post_meta()\` for WooCommerce orders
- Store Unix timestamps in options
- Use inline JavaScript or HTML in functions

---

**This is your portable base.** Copy to any WordPress plugin project and adapt as needed.

**For project-specific patterns:** See \`.instructions.md\` in your plugin root  
**For detailed implementations:** See `dev-notes/patterns/` directory

---
> Source: [headwalluk/payment-email-notifications](https://github.com/headwalluk/payment-email-notifications) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
