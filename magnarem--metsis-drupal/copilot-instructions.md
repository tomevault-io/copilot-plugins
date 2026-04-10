## metsis-drupal

> AI coding guide for METSIS Search (metsis_drupal) Drupal module project.

# AGENTS.md

AI coding guide for METSIS Search (metsis_drupal) Drupal module project.

## AI Response Requirements

**Communication style:**

- Code over explanations - provide implementations, not descriptions
- Be direct, skip preambles
- Assume Drupal expertise - no over-explaining basics
- Suggest better approaches with code
- Show only changed code sections with minimal context
- Complete answers in one response when possible
- Use Drupal APIs, not generic PHP
- Ask if requirements are ambiguous

**Response format:**

- Production-ready code with imports/dependencies
- Inline comments explain WHY, not WHAT
- Include full file paths
- Proper markdown code blocks

## Project Overview

- **Platform**: Drupal 11.3.5 / single site (dev test instance in `web/`)
- **Context**: Custom Drupal module (`metno/metsis_drupal`, v4.x-dev) — interactive search frontend for MET Norway's Scientific Information System (METSIS). Integrates Search API + Solr for climate/weather dataset metadata discovery.
- **Architecture**: Standalone module with Views row/filter/field plugins, Solr integration, Preact + OpenLayers JS apps (built with Vite), SDC components, and a dev Drupal 11 instance in `web/` for testing.
- **Security**: Standard — no sensitive user data; public-facing read-only search. API keys for Solr in environment variables.
- **Languages**: Single (English)
- **Custom Entities**: None
- **Role System**: Standard Drupal roles (anonymous, authenticated, administrator) in dev instance
- **Use Cases**: Dataset/metadata search via Solr, bounding-box spatial filtering, temporal filtering, dataset detail views, metadata export, Bokeh data plots

## Date Verification Rule

**CRITICAL**: Before writing dates to `.md` files, run `date` first.

## Git Workflow

**Before PR**: Run `phpcs`, `phpstan`, tests, `drush cex` and `checkall` (inside container).

**.gitignore essentials**:

```gitignore
web/core/
web/modules/contrib/
web/themes/contrib/
vendor/
web/sites/*/files/
web/sites/*/settings.local.php
.ddev/
node_modules/
.env
```

## Development Environment

**Web root**: `web/`

**Command context rule**:

- VS Code in this project runs inside the DDEV web container.
- Inside container: run tools directly (`drush`, `composer`, `php`, `npm`).
- From host shell: use `ddev` prefixes (`ddev drush`, `ddev composer`, etc.).

### Setup

```bash
git clone git@github.com:magnarem/metsis_drupal.git && cd metsis_drupal
ddev start && ddev poser && ddev symlink-project && drush si -y && drush cr
```

**DDEV**: `ddev ssh` (open container shell), `ddev describe` (info)

### Custom DDEV Commands

Location: `.ddev/commands/host/<name>`
**WARNING**: Don't use `## #ddev-generated` comments - they break command recognition.

### Drush Commands

```bash
# Core commands
drush status                    # Status check
drush cr                        # Cache rebuild
drush cex                       # Config export
drush cim                       # Config import
drush updb                      # Database updates

# Database & PHP eval
drush sql:query "SELECT * FROM node_field_data LIMIT 5;"
drush php:eval "echo 'Hello World';"

# Test services and entities
drush php:eval "var_dump(\Drupal::hasService('entity_type.manager'));"
drush php:eval "\$node = \Drupal::entityTypeManager()->getStorage('node')->load(1); var_dump(\$node->getTitle());"
drush php:eval "var_dump(\Drupal::config('system.site')->get('name'));"
drush php:eval "var_dump(\Drupal::service('custom_module.service_name'));"

# Quick setup (pull from platform)
ddev pull platform -y && drush cim -y && drush cr && drush uli
```

### Composer

```bash
composer outdated 'drupal/*'                    # Check updates
composer update drupal/<module> --with-deps     # Update module
composer require drupal/core:X.Y.Z drupal/core-recommended:X.Y.Z --update-with-all-dependencies  # Core update
```

**Scripts** in `composer.json`: `build`, `deploy`, `test`, `phpcs`, `phpstan`

### Environment Variables

Store in `.ddev/.env` (gitignored). Access: `$_ENV['VAR']`. Restart DDEV after changes.

### Patches

Structure: `./patches/{core,contrib/<module>,custom}/`

In `composer.json` → `extra.patches`:

```json
"drupal/module": {"#123 Fix": "patches/contrib/module/fix.patch"}
```

Sources: local files, Drupal.org issue queue, GitHub PRs
Always include issue numbers in descriptions. Monitor upstream for merged patches.

## Code Quality Tools

```bash
# PHPStan - static analysis
ddev exec vendor/bin/phpstan analyze web/modules/custom --level=1

# PHPCS - coding standards check
ddev exec vendor/bin/phpcs --standard=Drupal web/modules/custom/

# PHPCBF - auto-fix coding standards
ddev exec vendor/bin/phpcbf --standard=Drupal web/modules/custom/

# Rector - code modernization (run in container)
ddev ssh && vendor/bin/rector process web/modules/custom --dry-run

# Upgrade Status - Drupal compatibility check
ddev drush upgrade_status:analyze --all
```

**Config files**: `phpstan.neon`, `phpcs.xml`, `rector.php`
**Run before**: commits, PRs, Drupal upgrades

## Testing

```bash
# PHPUnit
ddev exec vendor/bin/phpunit web/modules/custom
ddev exec vendor/bin/phpunit tests/src/Unit/MyTest.php
ddev exec vendor/bin/phpunit --coverage-html coverage web/modules/custom

# Codeception
ddev exec vendor/bin/codecept run <acceptance|functional|unit>
ddev exec vendor/bin/codecept run --steps --debug --html

# Debug failed tests
ddev exec vendor/bin/phpunit --testdox --verbose <test-file>
```

**Drupal test types** (in `tests/src/`): `Unit/` (isolated), `Kernel/` (minimal bootstrap), `Functional/` (full Drupal), `FunctionalJavascript/`

**Codeception structure**: `tests/{acceptance,functional,unit,_support,_data,_output}/`, config: `codeception.yml`

## Debugging

```bash
# Xdebug
ddev xdebug on|off                              # Toggle (disable when not debugging for perf)

# Container & DB access
ddev ssh                                         # Web container
ddev mysql                                       # MySQL CLI
ddev mysql -e "SELECT..."                        # Direct query
ddev export-db --file=backup.sql.gz             # Export
ddev import-db --file=backup.sql.gz             # Import

# Logs
ddev logs -f                                     # Container logs (follow)
ddev drush watchdog:show --count=50 --severity=Error

# State
ddev drush state:get|set|delete <key> <value>
```

**IDE**: PhpStorm (port 9003), VS Code (PHP Debug extension)
**Tips**: `ddev describe` (URLs/services), `ddev debug` (DDEV issues), Twig debug in `development.services.yml`

## Performance

```bash
# Cache
ddev drush cr                                    # Rebuild all
ddev drush cache:clear <render|dynamic_page_cache|config>

# Redis (if enabled)
ddev redis-cli INFO stats|memory
ddev redis-cli FLUSHALL                          # Clear Redis

# DB performance
ddev mysql -e "SELECT table_name, round(((data_length+index_length)/1024/1024),2) 'MB' FROM information_schema.TABLES WHERE table_schema=DATABASE() ORDER BY (data_length+index_length) DESC;"
ddev mysql -e "SHOW VARIABLES LIKE 'slow_query%';"
ddev drush sql:query "OPTIMIZE TABLE cache_bootstrap, cache_config, cache_data, cache_default, cache_discovery, cache_dynamic_page_cache, cache_entity, cache_menu, cache_render;"
```

**Optimization**: Enable page cache + dynamic page cache, CSS/JS aggregation, Redis/Memcache, CDN for assets, image styles with lazy loading

## Code Standards

### Core Principles

- **SOLID/DRY**: Follow SOLID principles, extract repeated logic
- **PHP 8.4**: Use strict typing: `declare(strict_types=1);`
- **Drupal Standards**: PSR-12 based, English comments only

### Module Structure

Location: `/web/modules/custom/metsis_[module_name]/`
Naming: `metsis_[descriptive_name]` — prevents conflicts with contrib

```
metsis_[module_name]/
├── metsis_[module_name].{info.yml,module,install,routing.yml,permissions.yml,services.yml,libraries.yml}
├── src/                          # PSR-4: \Drupal\[module_name]\[Subdir]\ClassName
│   ├── Entity/                   # Custom entities
│   ├── Form/                     # Forms (ConfigFormBase, FormBase)
│   ├── Controller/               # Route controllers
│   ├── Plugin/{Block,Field/FieldWidget,Field/FieldFormatter}/
│   ├── Service/                  # Custom services
│   └── EventSubscriber/          # Event subscribers
├── templates/                    # Twig templates
├── css/ & js/                    # Assets
```

**PSR-4**: `src/Form/MyForm.php` → `\Drupal\my_module\Form\MyForm`

### Entity Development Patterns

```php
// 1. Enums instead of magic numbers
enum EntityStatus: string {
  case Draft = 0;
  case Published = 1;
}

// 2. Getter methods instead of direct field access
public function getStatus(): int {
  return (int) $this->get('status')->value;
}

// 3. Safe migrations with backward compatibility
function metsis_drupal_update_XXXX() {
  $manager = \Drupal::entityDefinitionUpdateManager();
  $field = $manager->getFieldStorageDefinition('field_name', 'entity_type');
  if ($field) {
    $new_def = BaseFieldDefinition::create('field_type')->setSettings([...]);
    $manager->updateFieldStorageDefinition($new_def);
    drupal_flush_all_caches();
    \Drupal::logger('module')->info('Migration completed.');
  }
}
```

**Migration safety**: Backup DB, test on staging, ensure backward compatibility, log changes, have rollback plan.

### Drupal Best Practices

```php
// Database API - always use placeholders, never raw SQL
$query = \Drupal::database()->select('node_field_data', 'n')
  ->fields('n', ['nid', 'title'])->condition('status', 1)->range(0, 10);
$results = $query->execute()->fetchAll();

// Dependency Injection - avoid \Drupal:: static calls in classes
class MyService {
  public function __construct(
    private readonly EntityTypeManagerInterface $entityTypeManager,
  ) {}
}

// Caching - use tags and contexts
$build = [
  '#markup' => $content,
  '#cache' => ['tags' => ['node:' . $nid], 'contexts' => ['user'], 'max-age' => 3600],
];
\Drupal\Core\Cache\Cache::invalidateTags(['node:' . $nid]);
\Drupal::cache()->set($cid, $data, time() + 3600, ['my_module']);
```

```php
// Queue API - for heavy operations
$queue = \Drupal::queue('my_module_processor');
$queue->createItem(['data' => $data]);
// QueueWorker plugin: @QueueWorker(id="...", cron={"time"=60})

// Entity System - always use entity type manager
$storage = \Drupal::entityTypeManager()->getStorage('node');
$node = $storage->load($nid);
$query = $storage->getQuery()
  ->condition('type', 'article')->condition('status', 1)
  ->accessCheck(TRUE)->sort('created', 'DESC')->range(0, 10);
$nids = $query->execute();

// Form API - extend FormBase, implement getFormId(), buildForm(), validateForm(), submitForm()
$form['field'] = ['#type' => 'textfield', '#title' => $this->t('Name'), '#required' => TRUE];
$form_state->setErrorByName('field', $this->t('Error message.'));

// Translation - always use t() for user-facing strings
$this->t('Hello @name', ['@name' => $name]);

// Config API
$config = \Drupal::config('my_module.settings')->get('key');
\Drupal::configFactory()->getEditable('my_module.settings')->set('key', $value)->save();

// Permissions
user_role_grant_permissions($role_id, ['permission']);
user_role_revoke_permissions($role_id, ['permission']);
```

### Code Style

- Type declarations/hints required, PHPDoc for classes/methods
- Align `=>` in arrays, `=` in variable definitions
- Controllers: final classes, DI, keep thin
- Services: register in `services.yml`, single responsibility
- Logging: `\Drupal::logger('module')->notice('message')`
- Entity updates: always use update hooks in `.install`, maintain backward compatibility

## Directory Structure

**Module root**: `/` (module lives at repo root, not inside `web/`)
**Key paths**: `src/` (PHP), `templates/` (Twig), `css/` (styles), `js/` (behaviours + Vite apps), `components/` (SDC), `config/install/` (default config), `config/optional/` (optional config), `config-sync/` (dev-instance config), `tests/` (PHPUnit)
**Dev Drupal instance**: `web/` — used for local testing only, not deployed

**Development paths**: routes → `metsis_drupal.routing.yml`, forms → `src/Form/`, hooks → `src/Hook/`, services → `src/Service/`, views plugins → `src/Plugin/views/`, permissions → `metsis_drupal.permissions.yml`, updates → `metsis_drupal.install`

## Configuration Management

````bash
ddev drush cex                    # Export config
ddev drush cim                    # Import config
ddev drush config:status          # Show differences
## Security

**Principles**: HTTPS required, sanitize input, use DB abstraction (no raw SQL), env vars for secrets, proper access checks

```bash
# Security updates
ddev drush pm:security
ddev composer update drupal/core-recommended --with-dependencies
ddev composer update --security-only

# Audit
ddev drush role:perm:list
ddev drush watchdog:show --severity=Error --count=100
````

**Hardening**: `chmod 444 settings.php`, `chmod 755 sites/default/files`, disable PHP in files dir

**Code**: Use placeholders in queries, `Html::escape()` for output, `$account->hasPermission()` for access, Form API for validation

## Headless/API-First Development

### JSON:API (Core)

```bash
ddev drush pm:enable jsonapi
# Optional: ddev composer require drupal/jsonapi_extras
```

**Endpoints**:

```
GET  /jsonapi/node/article                                    # List all
GET  /jsonapi/node/article/{uuid}?include=field_image,uid    # With relations
GET  /jsonapi/node/article?filter[status]=1&sort=-created&page[limit]=10
POST /jsonapi/node/article  (Content-Type: application/vnd.api+json, Authorization: Bearer {token})
```

### GraphQL

```bash
ddev composer require drupal/graphql drupal/graphql_compose
ddev drush pm:enable graphql graphql_compose
# Explorer at /admin/config/graphql
```

### Authentication (Simple OAuth)

```bash
ddev composer require drupal/simple_oauth && ddev drush pm:enable simple_oauth
openssl genrsa -out keys/private.key 2048 && openssl rsa -in keys/private.key -pubout -out keys/public.key
# POST /oauth/token with grant_type, client_id, client_secret, username, password
# Use: Authorization: Bearer {access_token}
```

### CORS (in services.yml)

```yaml
cors.config:
  enabled: true
  allowedOrigins: ["http://localhost:3000"]
  allowedMethods: ["GET", "POST", "PATCH", "DELETE", "OPTIONS"]
  allowedHeaders: ["*"]
  supportsCredentials: true
```

### Architecture Patterns

- **Fully Decoupled**: Drupal API + React/Vue/Next.js frontend
- **Progressively Decoupled**: Drupal pages + JS framework for interactive components
- **Hybrid**: Mix of Drupal templates and API-driven sections

### API Best Practices

OAuth tokens (not basic auth), rate limiting, HTTPS, validate input, API documentation (`drupal/openapi`)

## SEO & Structured Data

### Core Modules

**Naming**: `node--[type]--[view-mode].html.twig`, `paragraph--[type].html.twig`, `block--[type].html.twig`, `field--[name]--[entity].html.twig`

**Override**: Enable debug → view source for suggestions → copy from core/themes → place in templates/ → `ddev drush cr`

**Template directory structure**:

```
templates/
├── block/           # Block templates
├── content/         # Node templates
├── field/           # Field templates
├── form/            # Form element templates
├── layout/          # Layout templates
├── misc/            # Miscellaneous templates
├── navigation/      # Menu and navigation
├── paragraph/       # Paragraph templates
└── views/           # Views templates
```

### Preprocess Functions (`metsis_drupal.module`)

Module-level preprocess/theme hooks live in `metsis_drupal.module`. Hook implementations are organised in `src/Hook/`.

```php
function metsis_drupal_preprocess_views_view(&$variables) {
  // ...
}
function metsis_drupal_theme_suggestions_alter(array &$suggestions, array $variables, $hook) {
  // ...
}
```

### Single Directory Components (SDC)

Drupal 10.1+ core, or `ddev composer require drupal/sdc` for 10.0

**Structure**: `components/[name]/` with `[name].component.yml`, `[name].twig`, optional `.css`/`.js`

**component.yml**:

```yaml
name: Card
props:
  {
    type: object,
    properties: { title: { type: string }, link: { type: string } },
  }
slots: { content: { title: Content } }
```

**Usage**: `{% include 'metsis_drupal:doi' with { doi_url: url } %}` or `{% embed %}` for slots

**Module SDC components** (in `components/`):

- `metsis_drupal:bbox_form_tabs` — bounding box filter tabs
- `metsis_drupal:cc_license` — Creative Commons license icon
- `metsis_drupal:doi` — DOI icon + link
- `metsis_drupal:search` — search component

**Commands**: `ddev drush sdc:list`, `ddev drush pm:enable sdc`

### Troubleshooting

```bash
rm -rf node_modules package-lock.json && npm install   # Reset deps
rm -rf dist/ css/ js/compiled/                          # Clear build cache
```

**Performance**: Minify for prod, imagemin, critical CSS, font-display:swap, CSS/JS aggregation, AdvAgg module

## Environment Indicators

- **Visual verification**: Check indicators display correctly on all pages
- **Color scheme**: GREEN (Local), BLUE (DEV), ORANGE (STG), RED (PROD)
- **Never commit "LOCAL"** as value in `environment_indicator.indicator.yml` for production! Always use "PROD" and red color.

## Documentation

**MANDATORY**: Document work in "Tasks and Problems" section. Use real date (`date` command). Document: modules, fixes, config changes, optimizations, problems/solutions.

## Common Tasks

### New Module

```bash
# Create /web/modules/custom/metsis_<name>/ with:
# - metsis_<name>.info.yml (name, type:module, core_version_requirement:^10||^11||^12, package:METNO)
# - metsis_<name>.module (hooks), .routing.yml, .permissions.yml, .services.yml as needed
ddev drush pm:enable metsis_<name> && ddev drush cr
```

### Update Core

```bash
ddev export-db --file=backup.sql.gz                                    # Backup
ddev composer update drupal/core-recommended drupal/core-composer-scaffold --with-dependencies
ddev drush updb && ddev drush cr                                       # Updates + cache
```

### Database Migration

```php
// In metsis_drupal.install
function metsis_drupal_update_10001() {
  // Use EntityDefinitionUpdateManager for field changes
  // Check field exists, update displays, log completion
  drupal_flush_all_caches();
}
```

### Tests

```bash
ddev exec vendor/bin/phpunit tests   # PHPUnit (run from module root)
ddev exec vendor/bin/codecept run                                 # Codeception
# Dirs: tests/src/Unit/, Kernel/, Functional/; tests/acceptance/
```

### Permissions

```bash
ddev drush role:perm:list [role]                                  # List
# PHP: user_role_grant_permissions($role_id, ['perm1']); drupal_flush_all_caches();
```

## Troubleshooting

### Quick Fixes

```bash
ddev drush cr                                                     # Clear cache
ddev restart                                                      # Restart containers
ddev xdebug on|off                                               # Debug mode
ddev drush watchdog:show --count=50                              # Check logs
```

### Cache Not Clearing

```bash
drush cr                                                     # Standard
rm -rf web/sites/default/files/php/twig/* && ddev drush cr       # Twig
ddev drush sql:query "TRUNCATE cache_render;" && ddev drush cr   # Nuclear
```

### Database Issues

```bash
ddev drush sql:cli                     # Check connection (SELECT 1;)
ddev drush updb && ddev drush entity:updates   # Pending updates
ddev mysql -e "REPAIR TABLE <name>;"   # Repair table
```

### DDEV Issues

```bash
ddev restart                           # Soft restart
ddev stop && ddev start                # Full restart
ddev delete -O && ddev start           # Recreate containers
ddev logs                              # View logs
```

### Module Installation

```bash
ddev composer why-not drupal/<module>  # Check deps
ddev composer require drupal/<module> && ddev drush pm:enable <module>
ddev drush updb && ddev drush entity:updates   # Schema issues
```

### Permissions

```bash
ddev exec chmod -R 775 web/sites/default/files
ddev exec chmod 444 web/sites/default/settings.php
```

### WSOD (White Screen)

```bash
ddev drush config:set system.logging error_level verbose
ddev logs && ddev drush watchdog:show --count=50
ddev exec tail -f /var/log/php/php-fpm.log    # Check fatal errors
```

### Config Import Fails

```bash
ddev drush config:status              # Check status
ddev drush config:set system.site uuid [correct-uuid]  # UUID mismatch
```

### Memory Issues

```bash
echo "memory_limit = 512M" >> .ddev/php/php.ini && ddev restart
# Or: ddev exec php -d memory_limit=1G vendor/bin/drush <cmd>
```

## Additional Resources

- **Project Documentation**: `AGENTS.md` (this file) and `docs/`
- **Drupal Documentation**: https://www.drupal.org/docs
- **DDEV Documentation**: https://ddev.readthedocs.io/

---

# <!--

PROJECT-SPECIFIC SECTIONS BELOW
Add sections specific to your project here
===========================================
-->

# <!--

# HOW TO DISCOVER FULL ENTITY STRUCTURE - GUIDE FOR AI/LLM

Use this guide to discover the complete entity structure of a Drupal project.
Priority order: 1) Config YAML files (most complete), 2) Drush commands, 3) PHP evaluation

### 1. CONFIG YAML FILES (Primary Source)

Default config directory: `./config/sync/` (see "Directory Structure" section for verification commands and multisite paths)

**Entity Type Config File Patterns:**

| Entity Type           | Config File Pattern                | Example                                             |
| --------------------- | ---------------------------------- | --------------------------------------------------- |
| Content Types         | `node.type.*.yml`                  | `node.type.article.yml`                             |
| Field Storage         | `field.storage.*.yml`              | `field.storage.node.field_image.yml`                |
| Field Instance        | `field.field.*.yml`                | `field.field.node.article.field_image.yml`          |
| Paragraph Types       | `paragraphs.paragraphs_type.*.yml` | `paragraphs.paragraphs_type.text.yml`               |
| Media Types           | `media.type.*.yml`                 | `media.type.image.yml`                              |
| Taxonomy Vocabularies | `taxonomy.vocabulary.*.yml`        | `taxonomy.vocabulary.tags.yml`                      |
| View Modes            | `core.entity_view_mode.*.yml`      | `core.entity_view_mode.node.teaser.yml`             |
| Form Modes            | `core.entity_form_mode.*.yml`      | `core.entity_form_mode.node.default.yml`            |
| View Display          | `core.entity_view_display.*.yml`   | `core.entity_view_display.node.article.default.yml` |
| Form Display          | `core.entity_form_display.*.yml`   | `core.entity_form_display.node.article.default.yml` |

**Commands to list config files:**

```bash
# List all content type configs
ls config/sync/node.type.*.yml

# List all field storage configs
ls config/sync/field.storage.*.yml

# List all paragraph type configs
ls config/sync/paragraphs.paragraphs_type.*.yml

# Read specific config file
cat config/sync/node.type.article.yml
```

### 2. DRUSH COMMANDS

```bash
# List all entity types
ddev drush entity:info

# List bundles for entity type
ddev drush entity:bundle-info node
ddev drush entity:bundle-info paragraph
ddev drush entity:bundle-info media
ddev drush entity:bundle-info taxonomy_term

# List fields for entity type and bundle
ddev drush field:list node article
ddev drush field:list paragraph text

# Get field info
ddev drush field:info node article field_image

# Export all config
ddev drush config:export

# Get specific config
ddev drush config:get node.type.article

# List all config
ddev drush config:list | grep node.type
```

### 3. PHP/DRUSH PHP:EVAL

For programmatic access to field definitions:

```bash
# Get all fields for content type
ddev drush php:eval "
\$fields = \Drupal::service('entity_field.manager')->getFieldDefinitions('node', 'article');
foreach (\$fields as \$name => \$field) {
  echo \$name . ' - ' . \$field->getLabel() . ' (' . \$field->getType() . ')' . PHP_EOL;
}
"

# Get field settings
ddev drush php:eval "
\$field = \Drupal\field\Entity\FieldConfig::loadByName('node', 'article', 'field_image');
print_r(\$field->getSettings());
"

# Get all bundles for entity type
ddev drush php:eval "
\$bundles = \Drupal::service('entity_type.bundle.info')->getBundleInfo('node');
foreach (\$bundles as \$id => \$info) {
  echo \$id . ' - ' . \$info['label'] . PHP_EOL;
}
"

# Export full entity structure as JSON
ddev drush php:eval "
\$entity_types = ['node', 'paragraph', 'media', 'taxonomy_term'];
\$result = [];
foreach (\$entity_types as \$entity_type) {
  \$bundles = \Drupal::service('entity_type.bundle.info')->getBundleInfo(\$entity_type);
  foreach (\$bundles as \$bundle_id => \$bundle_info) {
    \$fields = \Drupal::service('entity_field.manager')->getFieldDefinitions(\$entity_type, \$bundle_id);
    \$field_list = [];
    foreach (\$fields as \$name => \$field) {
      \$field_list[\$name] = [
        'label' => (string) \$field->getLabel(),
        'type' => \$field->getType(),
        'required' => \$field->isRequired(),
      ];
    }
    \$result[\$entity_type][\$bundle_id] = [
      'label' => \$bundle_info['label'],
      'fields' => \$field_list,
    ];
  }
}
echo json_encode(\$result, JSON_PRETTY_PRINT);
"
```

### RECOMMENDED WORKFLOW

1. **First**: Check if `config/sync/` directory exists and list YAML files
2. **Second**: Use `ddev drush entity:bundle-info [type]` for quick overview
3. **Third**: Use `ddev drush field:list [type] [bundle]` for field details
4. **Fourth**: Use `php:eval` for complex queries or full export

-->

## Drupal Entities Structure

Complete reference of content types, media types, taxonomies, and custom entities. See "HOW TO DISCOVER FULL ENTITY STRUCTURE" guide above for discovery commands.

### Content Types (Node Bundles)

```toon
content_types[4]{machine_name,label,description,features}:
  dataset,Dataset,Scientific dataset in the Meteorological context,"revisions,menu_ui"
  article,Article,Time-sensitive content like news and blog posts,"revisions"
  page,Basic page,Static content such as About us,"revisions"
  movie,Movie,(dev/demo content type),"revisions,menu_ui"
```

### Paragraph Types

None — Paragraphs module is not used in this project.

### Media Types

None — Media module types are not configured in this project.

### Taxonomy Vocabularies

```toon
taxonomies[5]{machine_name,label,description}:
  dataset_variables,dataset variables,Variables describing the dataset
  countries,Countries,(country tagging)
  keywords,Keywords,(keyword tagging)
  tags,Tags,Used to group articles on similar topics
  genres,Genres,(dev/demo vocabulary)
```

### Custom Entities

None — no custom content entities. All data is sourced from Solr index at query time.

### Entity Relationships

- **Dataset** → **dataset_variables** taxonomy (via field reference, for variable tagging)
- **Dataset** → **keywords** taxonomy (keyword tagging)
- **Article** → **tags** taxonomy (via `field_tags`)
- Search results are Solr documents, not node entities — relationships are encoded in Solr fields

### Field Patterns

**Common field naming patterns in this project**:

- `field_[name]` - Standard field prefix
- `field_[prefix]_[name]` - Module-specific fields (e.g., `field_meta_tags`)
- Base fields: `title`, `body`, `created`, `changed`, `uid`, `status`

**Key Field Types**:

- Reference fields: `entity_reference`, `entity_reference_revisions`
- Text fields: `string`, `text_long`, `text_with_summary`
- Date fields: `datetime`, `daterange`, `timestamp`
- Media: `image`, `file`
- Structured: `link`, `address`, `telephone`

### View Modes

**Node View Modes** (standard Drupal defaults in use):

- `full` - Full content display
- `teaser` - Summary/card display

### Module Constants

See `src/MetsisConstants.php` for module-wide constants (e.g. field names, Solr field mappings).

### Entity Access Patterns

- View: `access content` | Edit own: `edit own [type] content` | Delete own: `delete own [type] content` | Admin: `administer [type] content`

### Migration Patterns

Not applicable — no data migrations. Dataset records are indexed from external Solr/MMD sources.

## Project-Specific Features

1. Solr Search Integration

- Core configs: `config/install/search_api.server.metsis_solr.yml`, `config/install/search_api.index.metsis.yml`, `config/install/views.view.metsis_search.yml`
- Query customization: `src/EventSubscriber/SearchApiSolrSubscriber.php` and related subscribers
- Search result rows are Solr documents rendered via custom row plugin/service, not loaded node entities

2. Custom Views Plugins

- Row plugin: `src/Plugin/views/row/MetsisSearchRow.php`
- Filters: `src/Plugin/views/filter/MetsisSolrBboxFilter.php`, `src/Plugin/views/filter/MetsisSolrDateRangeFilter.php`, `src/Plugin/views/filter/MetsisParentFilter.php`
- Field/area plugins: `src/Plugin/views/field/LeafletGeometryField.php`, `src/Plugin/views/area/MetsisMapArea.php`

3. Frontend Apps (Vite + Preact + OpenLayers)

- Source root: `js/metsis/`
- Apps: `metsis-map-app`, `bbox-map-filter`
- Build output: `js/metsis/dist/` (consumed through `metsis_drupal.libraries.yml`)

4. Search Result Rendering Stack

- Main results layout template: `templates/views-view--metsis-search--results.html.twig`
- Row template: `templates/metsis-search-row-default.html.twig`
- Row renderer service: `src/Service/ResultRowRenderer.php`
- Row/search CSS split: `css/metsis_default_row_layout.css` (row), `css/metsis_search_layout.css` (view layout)

## Development Workflow

- Document all significant changes in "Tasks and Problems" section below
- Follow the format and examples provided
- Review existing entries before making architectural changes
- Always run `date` command to get current date before adding entries

---

## Tasks and Problems Log

**Format**: `YYYY-MM-DD | [TYPE] Description` — Types: TASK, PROBLEM/SOLUTION, CONFIG, PERF, SECURITY, NOTE

Run `date` first. Add new entries at top. Include file paths, module names, config keys.

```
[Add entries here - newest first]

2026-03-27 | TASK: Made search page meta description configurable via settings form and hook
          | FILES: src/Form/MetsisSettingsForm.php, src/Hook/SearchPageMetaTagHook.php
          | CONFIG: Added metsis_drupal.settings.search_meta_description in config/install/metsis_drupal.settings.yml and config/schema/metsis_drupal.schema.yml

Examples:
2024-01-15 | TASK: Created custom module d_custom_feature for special workflow
2024-01-15 | PROBLEM: Config import failing with UUID mismatch
          | SOLUTION: drush config:set system.site uuid [correct-uuid]
2024-01-14 | CONFIG: Enabled Redis cache backend in settings.php
2024-01-14 | PERF: Enabled CSS/JS aggregation and AdvAgg module
2024-01-13 | SECURITY: Applied security update for Drupal core 10.1.8
2024-01-13 | NOTE: Custom entity queries must include ->accessCheck(TRUE/FALSE)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/magnarem)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/magnarem)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
