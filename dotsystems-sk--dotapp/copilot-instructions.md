## dotapp

> **You are working with the DOTAPP PHP FRAMEWORK - NOT Laravel, Symfony, CodeIgniter, or any other framework!**

# DotApp Framework - AI Assistant Configuration
# ⚠️  READ THIS ENTIRE FILE BEFORE EVERY CODE CHANGE! ⚠️
# ⚠️  RE-READ THESE RULES FOR EVERY REQUEST! ⚠️

## 🚨 CRITICAL FRAMEWORK IDENTITY
**You are working with the DOTAPP PHP FRAMEWORK - NOT Laravel, Symfony, CodeIgniter, or any other framework!**

**DOTAPP is a proprietary framework with unique syntax and patterns.** If you don't understand how something works, **study the actual code in `/app/parts/`** - do NOT assume it works like other frameworks.

## 🤖 AI ASSISTANT CRITICAL WARNING
**BEFORE ANY WORK ON A MODULE: Always read the MD guide files in that specific module!**
- Each module has its own `*_AI_guide.md` files (e.g., `CONTROLLERS_AI_guide.md`, `VIEWS_AI_guide.md`)
- **Example:** Working on `TestPaka` module? Read `TestPaka/CONTROLLERS_AI_guide.md` FIRST!
- **Why?** Module-specific guides override general rules and contain exact syntax examples!
- **AI Memory Tip:** If you forget this, you'll use wrong syntax and create bugs!

### Common Misconceptions to Avoid:
- ❌ NO `$this->db` or `DB::table()` like Laravel
- ❌ NO Doctrine ORM syntax
- ❌ NO Symfony dependency injection containers
- ❌ NO CodeIgniter active records
- ✅ YES `DB::module("RAW")` and `DB::module("ORM")`
- ✅ YES Static controllers with DI
- ✅ YES Custom template syntax `{{ variable }}`

---

## 🏗️ FRAMEWORK ARCHITECTURE

### Core Structure (`/app/parts/` - 🔴 DO NOT TOUCH!)
```
/app/parts/                    # 🔴 CORE FRAMEWORK - NEVER EDIT!
├── Controllers/              # Core controller classes
├── Middleware/               # Core middleware classes
├── Models/                   # Core model classes
├── Databaser.php             # Database abstraction layer
├── QueryBuilder.php          # SQL query builder
├── Entity.php               # ORM Entity class
├── Collection.php           # ORM Collection class
├── Renderer.php             # Template rendering engine
├── Router.php               # Route handling
├── DI.php                   # Dependency injection
├── DB.php                   # Database facade
├── Request.php              # HTTP request handling
├── Response.php             # HTTP response handling
├── Validator.php            # Data validation
├── Translator.php           # Internationalization
└── [many more core files...]
```

### Module Structure (`/app/modules/` - 🟢 SAFE TO EDIT!)
```
/app/modules/                 # 🟢 MODULES - EDIT HERE!
├── ModuleName/
│   ├── Controllers/          # Module controllers
│   │   └── CONTROLLERS_GUIDE.md  # Controller documentation
│   ├── Models/               # Module models
│   ├── views/                # Module templates
│   ├── assets/               # CSS, JS, images
│   │   └── ASSETS_GUIDE.md   # Asset management
│   ├── translations/         # Language files
│   │   └── TRANSLATOR_GUIDE.md   # Translation documentation
│   ├── tests/                # Module tests
│   │   └── TESTS_GUIDE.md    # Testing documentation
│   ├── MODULE_GUIDE.md       # Module documentation
│   └── module.init.php       # Module initialization
```

---

## 📚 DOCUMENTATION MAP

### Core Framework Guides
- **`database_guide.md`** - Complete database usage guide (RAW, ORM, QueryBuilder)
- **`ai_database_guide.md`** - Database operations guide specifically for AI assistants

### Module-Specific Guides
## 🔴 CRITICAL: ALWAYS READ MODULE MD FILES FIRST!
**Each module in `/app/modules/` contains its own documentation that YOU MUST READ before working on that module!**

**Example workflow:**
1. User asks: "Create a route in TestPaka module"
2. YOU: Read `TestPaka/CONTROLLERS_AI_guide.md` and `TestPaka/views/VIEWS_AI_guide.md`
3. Only then start coding with the correct syntax from those files

**If you skip this step, you'll use wrong syntax and create bugs!**

#### Controllers (`*/Controllers/CONTROLLERS_GUIDE.md` or `*_AI_guide.md`)
- Static class structure
- Dependency injection usage
- Request/response handling
- API dispatch patterns

#### Views & Templates (`*/views/` or `*/ASSETS_GUIDE.md`)
- Custom template syntax `{{ variable }}`
- Layout system
- Asset management

#### Translations (`*/translations/TRANSLATOR_GUIDE.md`)
- Key-based translation system
- JSON format for language files
- Fallback mechanisms

#### Tests (`*/tests/TESTS_GUIDE.md` or `*_AI_guide.md`)
- Testing framework usage
- Module-specific test patterns

#### Module Initialization (`*/MODULE_GUIDE.md` or `*_AI_guide.md`)
- Module lifecycle
- Configuration options
- Event listeners setup

---

## ⚠️ ABSOLUTE SECURITY RULES

### 1. NEVER EDIT CORE FILES
```bash
❌ NEVER: /app/parts/*.php
❌ NEVER: /app/DotApp.php
❌ NEVER: /app/config.php
✅ SAFE: /app/modules/*/*
```

**If you think there's an error in core files, ASK THE USER - DO NOT EDIT!**

### 2. CONTROLLER CALLING SYNTAX (CRITICAL!)
**NEVER use `Controller@function` format!**

**✅ CORRECT DotApp syntax:**
```php
// Module:Controller@function!
PharmListWeb:Home@indexSk!
PharmListWeb:Search@indexEn!

// Use ! at end to skip DI container (saves memory)
```

**❌ WRONG (Laravel/Symfony style):**
```php
// DO NOT USE THESE!
HomeController@index
Controller@function
Home@index
```

**❗ IMPORTANT: When using `!` (no DI):**
- **NO dependency injection parameters** in controller methods
- **Use `self::initRenderer()`** instead of `Renderer $renderer` parameter
- **Access services manually** or use static methods

**✅ With `!` (no DI container):**
```php
public static function index($request) {
    $renderer = Renderer::new(); // Create new renderer instance
    // OR: $renderer = Renderer::new('my_view'); // Share renderer with existing variables for layout reuse
    // OR if using BaseController:
    $renderer = self::initRenderer($request, 'sk'); // Only in controllers extending BaseController
    // ... rest of code
}
```

**❌ WRONG with `!` (AI often does this):**
```php
public static function index($request, Renderer $renderer) { // NO!
    // This will NOT work with ! - no DI parameters allowed!
}
```

**Why `!` matters:** Skips dependency injection container to save memory. AI often forgets this!
```php
// ✅ CORRECT - DotApp patterns
class MyController extends \Dotsystems\App\Parts\Controller {
    public static function index($request) {
        $data = DB::module("ORM")
            ->selectDb('main')
            ->q(function($qb) {
                $qb->select('*')->from('table');
            })
            ->all();

        $renderer = Renderer::new();
        return $renderer->module("ModuleName")
            ->setView("template")
            ->setViewVar("data", $data)
            ->renderView();
    }
}

// ❌ WRONG - Laravel/Symfony patterns
class MyController extends Controller {
    public function index() {
        $data = DB::table('users')->get(); // NO!
        return view('template', compact('data')); // NO!
    }
}
```

### 3. MODULE DEVELOPMENT RULES
- **Controllers**: Static methods, extend `\Dotsystems\App\Parts\Controller`
- **Controller Calling**: Use `Module:Controller@function!` format (e.g., `PharmListWeb:Home@indexSk!`)
- **DI Container**: Use `!` at end to skip DI container and save memory (e.g., `@function!`)
- **NO DI Parameters**: When using `!`, do NOT use dependency injection parameters like `Renderer $renderer`
- **Models**: Optional, can use ORM Entity
- **Routes**: Defined in `module.init.php`
- **Views**: Use `{{ variable }}` syntax, not Blade `{{ }}` or Twig `{{ }}`
- **Database**: Always `DB::module("RAW")` or `DB::module("ORM")`

---

## 🔧 CODING STANDARDS

### File Structure
- Use proper namespaces: `Dotsystems\App\Modules\ModuleName\Controllers`
- Controllers extend `\Dotsystems\App\Parts\Controller`
- Models extend `\Dotsystems\App\Parts\Model`
- Use static methods in controllers
- **NO dependency injection parameters** when using `!` in route calls

### Database Operations
- RAW queries: `DB::module("RAW")` with QueryBuilder
- ORM queries: `DB::module("ORM")` with Entity/Collection
- See `database_guide.md` and `ai_database_guide.md` for examples

### Template Rendering
- Use `{{ variable }}` syntax (NOT Blade/Twig)
- Layout system with `{{ $renderer->extend() }}`
- **CRITICAL: Templates receive only "clean" variables - ALL logic in controllers!**
- **NEVER use `{{ if isset(...) }}` or complex logic in templates**
- **Prepare all data in controllers, pass only final values to templates**
- Renderer initialization: `Renderer::new()` or `self::initRenderer()` (BaseController only)
- `Renderer::new(Optional NAME)` - optional name allows sharing renderer instance with existing variables, just changing layout/variables (useful when you want to reuse view variables but change layout)
- See module `*/views/` directories for examples

---

## 🚫 FORBIDDEN PATTERNS

**DO NOT use patterns from other frameworks:**
- Laravel: `Route::`, `DB::table()`, `Eloquent`, Blade templates, `Controller@function`
- Symfony: `@Route()`, Doctrine, Twig templates, `Controller@function`
- CodeIgniter: `$this->db`, Active Record
- **Also avoid with `!`:**
  - Dependency injection parameters like `Renderer $renderer` in controller methods
  - `self::initRenderer($request, 'sk')` - use `Renderer::new()` instead
- **Template anti-patterns:**
  - Complex `{{ if isset($var) && $var == 'value' }}` - move to controller
  - Business logic in templates - belongs in controller
  - Multiple conditions in one if - prepare boolean flags in controller
- **Always use DotApp equivalents:**
  - Database: `DB::module()`, Entity, `{{ }}` syntax
  - Controllers: `Module:Controller@function!` format with `!` for memory savings
  - Renderer: `Renderer::new()` or `self::initRenderer()` (only in BaseController)
  - Templates: Receive only "clean" variables, no logic

---

## 📖 LEARNING RESOURCES

**Study these resources:**
- **Core files**: `/app/parts/` - actual implementation
- **Module examples**: PharmListWeb, DotShop, Sabicare
- **Documentation**: See DOCUMENTATION MAP above for specific guides

---

## ✅ CORRECT USAGE PATTERNS

**For complete examples, check:**
- Module `CONTROLLERS_GUIDE.md` or `*_AI_guide.md` files
- Module `*/views/` directories for template examples
- `database_guide.md` and `ai_database_guide.md` for database patterns

### Template Logic Rules (CRITICAL!)

#### ✅ ALLOWED in Templates:
- **Basic isset checks**: `{{ if isset($variable) }}` (supported by DotApp)
- **Simple variable conditions**: `{{ if $isLoggedIn }}`
- **Foreach loops**: `{{ foreach $items as $item }}`
- **Basic variable output**: `{{ $variable }}`

#### ❌ NEVER in Templates (Complex Logic):
```php
// DO NOT DO THIS IN TEMPLATES!
{{ if isset($currentLang) && $currentLang == 'sk' }}
{{ if $user && $user->isLoggedIn() && $user->hasPermission('admin') }}
{{ if count($items) > 0 && $items[0]->status == 'active' }}
{{ foreach $items as $item }}{{ if $item->isActive() && $item->category == 'featured' }}
```

#### ✅ DO THIS INSTEAD - Prepare in Controller:
```php
// CONTROLLER - Prepare ALL logic
public static function index($request) {
    $renderer = Renderer::new();

    // Complex logic belongs HERE
    $user = getCurrentUser();
    $isLoggedIn = $user && $user->isLoggedIn();
    $isAdmin = $isLoggedIn && $user->hasPermission('admin');
    $activeItems = array_filter($items, fn($item) => $item->isActive());
    $featuredItems = array_filter($activeItems, fn($item) => $item->category == 'featured');
    $currentLang = $currentLang ?? 'sk';
    $isSlovak = isset($currentLang) && $currentLang == 'sk';

    // Pass ONLY final boolean values to template
    $renderer->setViewVar('isLoggedIn', $isLoggedIn);
    $renderer->setViewVar('isAdmin', $isAdmin);
    $renderer->setViewVar('isSlovak', $isSlovak);
    $renderer->setViewVar('activeItems', $activeItems);
    $renderer->setViewVar('featuredItems', $featuredItems);
    $renderer->setViewVar('currentLang', $currentLang);

    return $renderer->renderView();
}
```

#### ✅ THEN in Template - Only Simple Conditions:
```php
<!-- TEMPLATE - Only simple conditions -->
{{ if isset($currentLang) }}
    <span>Language: {{ $currentLang }}</span>
{{ endif }}

{{ if $isLoggedIn }}
    <div>Welcome!</div>
    {{ if $isAdmin }}
        <div>Admin Panel</div>
    {{ endif }}
{{ endif }}

{{ foreach $activeItems as $item }}
    <div>{{ $item->name }}</div>
{{ endforeach }}
```

#### Template Responsibility:
- **Display data only using simple conditions**
- **Use boolean flags prepared by controller**
- **No complex business logic or calculations**
- **Controller handles ALL data preparation and complex logic**

---

## 🐛 DEBUGGING
- Use `error_reporting(E_ALL); ini_set('display_errors', 1);` in development
- For CLI scripts, simulate HTTP request with `$_SERVER` variables
- Use DotApp logger: `DotApp::log('message', 'level')`

---

## 🔄 WORKFLOW RULES

0. **⚠️ BEFORE ANY CODE CHANGE: Re-read this entire .cursorrules file!**
1. **🔴 CRITICAL FOR AI: ALWAYS READ MODULE MD FILES FIRST!**
   - **Before any task on a module:** First read the `*_AI_guide.md` or `*GUIDE.md` file in that module!
   - **Example:** Working on `TestPaka` module? Read `TestPaka/CONTROLLERS_AI_guide.md`, `TestPaka/views/VIEWS_AI_guide.md`, etc. FIRST!
   - **Why?** Each module has its own specific guides for that module!
   - **Real example:** When I created HelloController, I should have read `TestPaka/CONTROLLERS_AI_guide.md` and `TestPaka/views/VIEWS_AI_guide.md` first!
2. **Controller syntax**: Always use `Module:Controller@function!` format (with `!` for memory)
3. **Never use**: `Controller@function`, `HomeController@index`, or similar patterns
4. **Template rule**: Controllers prepare complex logic - templates use simple conditions
5. **Basic isset OK**: `{{ if isset($var) }}` allowed, but complex logic belongs in controllers
6. **Always check module documentation first**: `*/GUIDE.md` or `*_AI_guide.md` files
6. **Study core implementation**: `/app/parts/` files
7. **Follow established patterns**: Look at existing modules
8. **Test thoroughly**: Use module test guides
9. **Ask if unsure**: Better to ask than break the framework

---

## 🛠️ DOTAPPER CLI TOOLS

DotApp provides CLI tools via `dotapper.php` for automated module and component creation. **Use these tools ONLY when explicitly permitted by the user.**

### Available Commands
- `--create-module=<name>` - Create new module
- `--create-controller=<name>` - Create controller in module
- `--create-model=<name>` - Create model in module
- `--create-middleware=<name>` - Create middleware in module
- `--list-routes` - Show all routes
- `--modules` - List modules
- See `php dotapper.php --help` for complete list

### CLI Tools Rules
- **Always ask user permission** before using CLI tools
- **Use for consistent structure** and rapid prototyping
- **Prefer manual creation** when user wants custom patterns
- **Review generated code** - never assume it's perfect

---

## 📝 CRITICAL REMINDERS

- **⚠️ BEFORE EVERY CODE CHANGE: Re-read this entire .cursorrules file!**
- **Controller calling**: `Module:Controller@function!` - always with `!` for memory savings!
- **With `!`: NO DI parameters** like `Renderer $renderer` in controller methods
- **Renderer initialization**: `Renderer::new()` - NOT `self::initRenderer()` unless extending BaseController
- **Template rule**: Controllers prepare complex logic - templates use simple conditions!
- **Basic isset OK**: `{{ if isset($var) }}` allowed, complex conditions belong in controllers
- **NEVER**: `Controller@function`, `HomeController@index`, or similar patterns
- **This is DOTAPP framework** - study the code, don't assume patterns from other frameworks!
- **Core files are sacred** - `/app/parts/` = read-only for you
- **Modules are your playground** - `/app/modules/` = edit freely
- **Documentation is your friend** - check `GUIDE.md` files first
- **When in doubt, study the core** - `/app/parts/` contains the truth

If you encounter unfamiliar patterns, **read the actual DotApp source code** in `/app/parts/` to understand how it works.

---

## 🔄 REINFORCEMENT MECHANISMS

### Always Include This Comment Template in New Files:
```php
<?php
/**
 * DOTAPP FRAMEWORK REMINDER:
 * - Controllers: Module:Controller@function! (with ! for memory)
 * - NO DI parameters when using ! (use self::initRenderer() instead)
 * - Database: DB::module("RAW") or DB::module("ORM")
 * - Templates: {{ variable }} syntax
 * - NEVER edit /app/parts/ files!
 *
 * See .cursorrules for complete guidelines.
 */
```

### Code Examples:
See module `*/GUIDE.md` files and existing controllers for code examples and patterns.

### Manual Validation:
Check your code manually against these rules before committing:
- Controller syntax: `Module:Controller@function!`
- NO DI parameters with `!`
- Proper renderer initialization: `Renderer::new()`
- Database calls use `DB::module()`

### Pre-Change Checklist (Use Before Every Edit):
- [ ] Re-read .cursorrules file
- [ ] Check if using correct controller syntax: `Module:Controller@function!`
- [ ] Verify NO DI parameters when using `!`
- [ ] Confirm database calls use `DB::module()`
- [ ] Ensure templates use `{{ }}` syntax
- [ ] Verify NOT editing `/app/parts/` files

### Workspace Settings for Better Compliance:
✅ **Workspace settings created at `.vscode/settings.json`** with DotApp-specific configurations including:
- File associations for `.view.php`, `.layout.php` files
- PHP validation settings
- Search excludes for generated files
- Code formatting rules
- DOTAPP FRAMEWORK REMINDERS in commentsraz sa pozri

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dotsystems-sk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
