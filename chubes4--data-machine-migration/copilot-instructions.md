## data-machine-migration

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Plugin Overview

**Data Machine Migration** is a standalone WordPress utility plugin for migrating Data Machine ecosystem prefixes from `dm_` to `datamachine_` to meet WordPress repository requirements.

**Version**: 1.0.0
**Purpose**: One-time prefix migration across database, meta, options, and block content
**Lifecycle**: Install → Execute → Remove
**Requirements**: WordPress 5.0+, PHP 7.4+, Data Machine plugins installed

**Migration Status:**
- Phase 1: ✓ Complete (migration plugin ready)
- Phase 2: ✓ Complete - Core plugin code refactoring from `dm_` to `datamachine_` fully completed
- Phase 3: ✓ Complete - Database table names migrated to `wp_datamachine_*`
- Phase 4: ✅ Complete - Extension migrations deployed and verified

**Current Reality:** Core plugin and all extensions fully migrated to `datamachine_` prefixes. Migration plugin available for production deployment on existing installations.

## Core Architecture

### PSR-4 Autoloading
**Namespace**: `DataMachineMigration\`
**Class Pattern**: `Class_Name` → `class-class-name.php`
**Autoloader**: SPL autoloader in main plugin file

### Plugin Constants
```php
DATAMACHINE_MIGRATION_VERSION  // Plugin version
DATAMACHINE_MIGRATION_PATH     // Plugin directory path
DATAMACHINE_MIGRATION_URL      // Plugin URL for assets
```

### Class Structure

**Admin_Interface** (`class-admin-interface.php`)
- Admin menu registration under Tools → Data Machine Migration
- 6-step UI: Overview → Database → Meta → Options → Post Types → Blocks → Validation
- AJAX handler for migration execution
- JavaScript/CSS asset management
- Capability checks: `manage_options` required

**Database_Migrator** (`class-database-migrator.php`)
- Table rename operations: `wp_dm_*` → `wp_datamachine_*`
- Dependency-ordered execution (child tables first)
- Foreign key validation
- Row count verification
- Atomic RENAME TABLE operations

**Meta_Migrator** (`class-meta-migrator.php`)
- Post meta key migration with batch processing
- User meta key migration (single query)
- Recursive serialized data handling
- Batch size: 100 records per iteration
- Safety limit: 100,000 records maximum

**Options_Migrator** (`class-options-migrator.php`)
- WordPress options name and value migration
- Recursive serialized data processing
- String value replacement in nested structures
- Transient exclusion (skips `_transient%` options)

**Post_Type_Migrator** (`class-post-type-migrator.php`)
- Custom post type migration in wp_posts table
- Migrates `dm_events` → `datamachine_events`
- Batch processing: 100 posts per iteration
- Comprehensive validation and rollback support
- Safety limit: 100,000 records maximum

**Block_Migrator** (`class-block-migrator.php`)
- Gutenberg block reference updates
- Block mappings: `dm-events/*` → `datamachine-events/*`, `dm-recipes/*` → `datamachine-recipes/*`
- Post content and reusable blocks migration
- Batch processing: 50 posts per iteration
- Attribute key updates within block JSON

**Validation** (`class-validation.php`)
- Comprehensive post-migration validation
- Table existence and accessibility checks
- Meta key validation (ensures no old prefixes remain)
- Options validation
- Block content validation
- Performance checks with warnings

## Migration Execution Flow

### 6-Step Process

**Step 1: Database Tables**
```php
$migrator = new Database_Migrator();
$results = $migrator->migrate_tables();
```

**Execution Order** (dependency-aware):
1. `wp_dm_processed_items` → `wp_datamachine_processed_items` (child table)
2. `wp_dm_jobs` → `wp_datamachine_jobs`
3. `wp_dm_flows` → `wp_datamachine_flows`
4. `wp_dm_pipelines` → `wp_datamachine_pipelines` (parent table)

**Validation**: Row counts, foreign key relationships, SELECT query tests

**Step 2: Meta Keys**
```php
$migrator = new Meta_Migrator();
$results = $migrator->migrate_meta_keys();
```

**Processing**:
- Post meta: Batch processing (100 records/iteration) with serialized data handling
- User meta: Single SQL UPDATE with REPLACE function
- Recursive key replacement in nested arrays/objects

**Step 3: Options**
```php
$migrator = new Options_Migrator();
$results = $migrator->migrate_options();
```

**Processing**:
- Find options with `dm_` in name or value (excludes transients)
- Update option names: `dm_*` → `datamachine_*`
- Process serialized values recursively
- Delete old option names after successful migration

**Step 4: Post Types**
```php
$migrator = new Post_Type_Migrator();
$results = $migrator->migrate_post_types();
```

**Processing**:
- Update custom post types in wp_posts table
- `dm_events` → `datamachine_events`
- Batch processing: 100 posts per iteration
- Verification: Ensures zero posts remain with old type

**Step 5: Block Content**
```php
$migrator = new Block_Migrator();
$results = $migrator->migrate_block_content();
```

**Processing**:
- Update block namespaces in post content
- Migrate reusable blocks (wp_block post type)
- Update block attributes containing old prefixes
- Batch processing: 50 posts per iteration
- Note: Queries for datamachine_events (post types already migrated)

**Step 6: Validation**
```php
$validator = new Validation();
$results = $validator->validate_migration();
```

**Checks**:
- All new tables exist and are queryable
- No old `dm_` meta keys remain
- No old `dm_` options remain
- No old `dm_events` post types remain
- Block content references updated
- Performance metrics (warnings for large datasets)

## AJAX Architecture

### Request Handler
**Action**: `wp_ajax_data_machine_migration_run`
**Method**: POST
**Nonce**: `data_machine_migration_nonce`
**Capability**: `manage_options`

### Request Parameters
```javascript
{
    step: 'database|meta|options|post_types|blocks|validate',
    action: 'migrate|rollback'
}
```

### Response Format
```javascript
{
    success: true|false,
    data: {
        // Step-specific results
        tables_renamed: [...],
        post_meta_processed: 123,
        options_processed: 45,
        errors: [...]
    }
}
```

## Safety Mechanisms

### Pre-Migration Checks
- Verify Data Machine plugins installed
- Check database write permissions
- Validate table structures
- Confirm no migration in progress

### Execution Safety
- **Atomic Operations**: RENAME TABLE (atomic at database level)
- **Batch Processing**: Prevents PHP timeouts on large datasets
- **Safety Limits**: 100,000 record maximum per batch operation
- **Error Isolation**: Continue processing on non-critical errors
- **Comprehensive Logging**: Every operation logged via `error_log()`

### Validation
- Post-execution validation required before completion
- Table existence and accessibility verification
- Row count consistency checks
- Old prefix detection (should be zero)

## Rollback Capabilities

### Database Rollback
```php
$migrator = new Database_Migrator();
$results = $migrator->rollback_tables();
```
- Reverse RENAME: `wp_datamachine_*` → `wp_dm_*`
- Dependency-reversed order (parent tables first)

### Meta Rollback
```php
$migrator = new Meta_Migrator();
$results = $migrator->rollback_meta_keys();
```
- Single SQL REPLACE operation for speed
- Post meta and user meta reversed simultaneously

### Options Rollback
```php
$migrator = new Options_Migrator();
$results = $migrator->rollback_options();
```
- Find `datamachine_*` options and revert
- Recursive value processing for serialized data

### Block Content Rollback
```php
$migrator = new Block_Migrator();
$results = $migrator->rollback_block_content();
```
- Reverse block namespace changes
- Restore original block attributes

## Logging Architecture

### Log Mechanism
**Method**: PHP `error_log()` function
**Output**: WordPress debug.log or server error logs
**Format**: `Data Machine Migration: [operation] [details]`

### Log Levels
```php
// Success operations
error_log("Data Machine Migration: Renamed table {$old} to {$new}");

// Error conditions
error_log("Data Machine Migration ERROR: Failed to rename {$table}: {$error}");

// Validation results
error_log("Data Machine Migration: Validation " . ($success ? 'PASSED' : 'FAILED'));
```

### Key Logging Points
- Table rename operations
- Meta key batch completions
- Options migration per-option
- Block content updates
- Validation results
- Error conditions with context

## Serialized Data Handling

### Recursive Key Replacement
**Pattern**: Handles nested arrays, objects, and mixed structures

```php
private function recursive_key_replace($data, $old_prefix, $new_prefix) {
    // Array processing: Replace keys starting with prefix
    // Object processing: Clone object, replace properties
    // String processing: Direct replacement in option values
    // Recursive: Process nested structures
}
```

### WordPress Functions Used
- `maybe_unserialize()`: Safe unserialization
- `maybe_serialize()`: Re-serialization after processing
- Maintains data type integrity throughout process

## Admin Interface

### Menu Location
**Path**: Tools → Data Machine Migration
**Capability**: `manage_options`
**Page Slug**: `datamachine-migration`

### UI Components
- **Sidebar Navigation**: Step-by-step progress tracking
- **Overview**: Migration plan and requirements
- **Step Pages**: Individual migration components with execute buttons
- **Status Display**: Real-time progress and error reporting
- **Validation Results**: Comprehensive post-migration checks

### JavaScript Integration
**File**: `assets/js/admin.js`
**Features**:
- AJAX migration execution
- Confirmation dialogs for destructive operations
- Real-time progress updates
- Error message display
- Success/failure notifications

**Localized Data**:
```javascript
dataMachineMigration = {
    ajax_url: admin_url('admin-ajax.php'),
    nonce: 'nonce_value',
    strings: { running, completed, error, confirm }
}
```

### CSS Styling
**File**: `assets/css/admin.css`
**Namespace**: `.datamachine-migration-*` classes
**Layout**: Sidebar navigation with content area

## Migration Status Tracking

### WordPress Option
**Name**: `data_machine_migration_status`
**Structure**:
```php
[
    'version' => '1.0.0',
    'installed' => '2024-11-02 12:00:00',
    'migration_completed' => false,
    'steps_completed' => [
        'database' => true,
        'meta' => true,
        'options' => true,
        'blocks' => true,
        'validation' => false
    ]
]
```

### Status Updates
- Created on plugin activation
- Updated after each successful step
- Checked to prevent duplicate migrations
- Used for UI state management

## Block Migration Patterns

### Block Namespace Mappings
```php
private $block_mappings = [
    'dm-events/event-details' => 'datamachine-events/event-details',
    'dm-events/calendar' => 'datamachine-events/calendar',
    'dm-recipes/recipe-schema' => 'datamachine-recipes/recipe-schema'
];
```

### Block Content Processing
1. Find posts containing old block references
2. Parse block content (WordPress block parser)
3. Update block names and attributes
4. Re-serialize block structure
5. Update post content via `wp_update_post()`

### Reusable Blocks
**Post Type**: `wp_block`
**Processing**: Same as regular post content
**Importance**: Affects all posts using the reusable block

## Batch Processing Patterns

### Standard Implementation
```php
$offset = 0;
$has_more = true;

while ($has_more) {
    $items = $wpdb->get_results($wpdb->prepare(
        "SELECT * FROM table WHERE condition LIMIT %d OFFSET %d",
        $batch_size, $offset
    ));

    if (empty($items)) {
        $has_more = false;
        break;
    }

    foreach ($items as $item) {
        // Process item
    }

    $offset += $batch_size;

    // Safety limit
    if ($offset > 100000) {
        $results['errors'][] = 'Safety limit reached';
        break;
    }
}
```

### Batch Sizes
- **Meta Keys**: 100 records/batch (handles serialized data)
- **Block Content**: 50 posts/batch (content processing intensive)
- **Options**: No batching (smaller dataset, direct processing)

## Error Handling Patterns

### Try-Catch Blocks
```php
try {
    // Migration operation
    if ($wpdb->last_error) {
        throw new \Exception($wpdb->last_error);
    }
    // Success logging
} catch (\Exception $e) {
    $results['success'] = false;
    $results['errors'][] = $e->getMessage();
    error_log("Data Machine Migration ERROR: " . $e->getMessage());
}
```

### Non-Fatal Errors
- Skip problematic records and continue
- Log warnings for manual review
- Accumulate errors in results array
- Don't halt entire migration for single failures

### Fatal Errors
- Database connection failures
- Permission errors
- Safety limit exceeded
- Return failure status immediately

## Development Guidelines

### Security Requirements
- All AJAX requests require `manage_options` capability
- Nonce verification mandatory (`wp_verify_nonce`)
- SQL injection prevention via `$wpdb->prepare()`
- Input sanitization with `sanitize_text_field()`

### WordPress Standards
- Use `$wpdb` for database operations (not direct SQL)
- WordPress functions for post updates (`wp_update_post`)
- Options API for configuration (`get_option`, `update_option`)
- Translation ready (`__()`, `_e()`, text domain: `datamachine-migration`)

### Code Style
- PSR-4 autoloading
- Clear method naming (descriptive, action-oriented)
- Comprehensive inline documentation
- Error context in all error messages

### Testing Approach
- Test on staging environment first
- Verify with small datasets before production
- Check logs after each step
- Validate rollback functionality
- Confirm Data Machine functionality post-migration

## Critical Warnings

### Execution Order
❌ **DO NOT** run migration before code refactoring is complete
❌ **DO NOT** skip validation step
❌ **DO NOT** run migration twice on same database
❌ **DO NOT** use on production without staging test

### Data Safety
✅ **ALWAYS** take database backups before migration
✅ **ALWAYS** test on staging environment first
✅ **ALWAYS** verify validation passes
✅ **ALWAYS** test Data Machine functionality after migration

### Plugin Lifecycle
1. Install migration plugin
2. Execute migration (all steps in order)
3. Verify validation passes
4. Test Data Machine functionality
5. Remove migration plugin
6. Keep backups for 30 days

## Performance Considerations

### Large Dataset Handling
- Batch processing prevents PHP timeouts
- Safety limits prevent infinite loops
- Memory-efficient processing (no full table loads)
- Direct SQL for bulk operations where safe

### Optimization Patterns
- User meta: Single SQL REPLACE (smaller dataset)
- Post meta: Batch processing with serialization handling
- Options: Individual processing (comprehensive validation)
- Blocks: Batch processing (parsing overhead)

### Performance Validation
- Check for tables with >10,000 rows
- Warn if meta keys >5,000 records
- Monitor execution time per batch
- Log batch completion times

## Troubleshooting

### Common Issues

**"Table does not exist"**
- Ensure Data Machine plugins are active
- Verify database prefix matches WordPress config
- Check database permissions

**"Permission denied"**
- User must have `manage_options` capability
- Database user needs ALTER TABLE permission
- Check file permissions for log writing

**"Timeout errors"**
- Adjust batch sizes (reduce from defaults)
- Increase PHP `max_execution_time`
- Process in smaller chunks

**"Validation failures"**
- Review specific validation errors
- Check for partial migration (some steps succeeded)
- Use rollback if needed, fix issues, retry

### Debug Mode
Enable WordPress debugging for detailed error information:
```php
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
```

Logs appear in `wp-content/debug.log`

## File Structure

```
datamachine-migration/
├── datamachine-migration.php          # Main plugin file
├── includes/
│   ├── class-admin-interface.php       # Admin UI and AJAX handler
│   ├── class-database-migrator.php     # Table rename operations
│   ├── class-meta-migrator.php         # Meta key migration
│   ├── class-options-migrator.php      # Options migration
│   ├── class-block-migrator.php        # Block content migration
│   └── class-validation.php            # Post-migration validation
├── assets/
│   ├── css/admin.css                   # Admin interface styling
│   └── js/admin.js                     # AJAX and UI interactions
├── README.md                           # User documentation
├── CLAUDE.md                           # Technical documentation
└── .gitignore                          # Git exclusions
```

## Integration with Data Machine Ecosystem

### Dependencies
- Requires Data Machine core plugin installed (for table structures)
- Works with DM Events, DM Recipes, DM Structured Data
- Handles block content from all Data Machine extensions

### Post-Migration
- Data Machine continues with new prefixes
- All extension plugins continue functioning
- OAuth configurations preserved
- Pipeline/Flow data intact

### Migration Plugin Removal
Safe to remove after:
1. Migration completed successfully
2. Validation passed
3. Data Machine functionality verified
4. 7+ days of stable operation (optional safety period)

---

**Important**: This is a one-time utility plugin. After successful migration and validation, it should be deactivated and removed from the WordPress installation. Keep database backups for at least 30 days post-migration.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chubes4) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
