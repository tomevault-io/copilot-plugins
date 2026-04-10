## werfrem

> **Lightweight PHP framework for rapid prototyping using Illuminate Collections instead of database.**

# WERFREM - AI Assistant Guidelines

**Lightweight PHP framework for rapid prototyping using Illuminate Collections instead of database.**

**Author:** haizim (wahihasyim@gmail.com | @haizim_)

---

## Critical Rules

### Documentation-First Approach
1. **User** creates documentation in `docs/{feature-name}.md`
2. **AI** reads the documentation
3. **AI** implements based on documentation
4. **User** tests and provides feedback

**AI MUST NOT:** Create or modify files in `docs/` directory

---

## Architecture

```
app/
├── Component/     # UI generators (Table, Form, Detail)
├── Controller/    # Request handlers
├── Core/         # Router, View
├── Datas/        # Collections (no DB)
├── Helper/       # Global functions
├── Router/       # Route definitions
├── Template/     # Layouts (app.php, auth.php)
└── View/         # Blade-like directive views
```

---

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Class | PascalCase | `OutletController`, `OutletData` |
| File | PascalCase.php | `OutletController.php` |
| Route | lowercase-hyphen | `/outlet`, `/master-produk` |
| Menu Key | lowercase_underscore | `'outlet'`, `'master_produk'` |
| Property | camelCase | `$this->data`, `$this->form` |
| View Dir | lowercase | `app/View/outlet/` |

---

## Data Layer (`app/Datas/`)

### Template
```php
<?php
namespace App\Datas;

class {FeatureName}Data extends DataBase
{
    function setData()
    {
        $this->data = [
            "data" => [
                [
                    "id" => 1,
                    "{field}" => "{value}",
                    "action" => "<a href='/{route}/1' class='btn btn-sm btn-secondary'>Detail</a>
                    <a href='/{route}/1/edit' class='btn btn-sm btn-secondary'>Edit</a>",
                ],
            ],
            'form' => ["{field}" => "{input-type}"]
        ];
    }
}
```

### Field Types
| Type | Use |
|------|-----|
| `integer` | Numeric values |
| `string` | Short text |
| `text` | Long content |

### Input Types
| Type | HTML |
|------|------|
| `text` | `<input type="text">` |
| `textarea` | `<textarea>` |
| `email` | `<input type="email">` |
| `select` | `<select>` |

---

## Controller (`app/Controller/`)

### Standard Template
```php
<?php
namespace App\Controller;

use App\Core\Router;

class {FeatureName}Controller extends Controller
{
    private $data;
    private $form;
    private $menu;
    private $title = '{Title}';
    private $url = '/{route}';

    function __construct()
    {
        $data = data('{FeatureName}');
        $menu = data('Menu');
        $role = capital_to_snake($_SESSION['user']['role']);
        $this->menu = $menu->get($role);
        $this->data = $data->data;
        $this->form = $data->form;
    }

    public function index()
    {
        $table = $this->data;
        $remove = $_SESSION['user']['role'] == 'Admin Aplikasi' ? ['_action'] : ['action'];
        $table = $table->map(function ($value) use ($remove) {
            foreach ($remove as $r) unset($value[$r]);
            return $value;
        });

        return $this->view('index', [
            'title' => $this->title,
            'user' => $_SESSION['user']['nama'],
            'menu' => $this->menu,
            'table' => $table,
            'addLink' => $this->url . '/create',
        ]);
    }

    public function create() {
        return $this->view('form', [
            'title' => "Tambah $this->title",
            'user' => $_SESSION['user']['nama'],
            'menu' => $this->menu,
            'form' => $this->form,
            'action' => $this->url . '/store',
        ]);
    }

    public function edit($id) {
        return $this->view('form', [
            'title' => "Edit $this->title",
            'user' => $_SESSION['user']['nama'],
            'menu' => $this->menu,
            'form' => $this->form,
            'action' => $this->url . '/store',
        ]);
    }

    public function store() { Router::to($this->url); }

    public function show($id) {
        $detail = $this->data[$id-1];
        unset($detail['action'], $detail['_action']);
        return $this->view('detail', [
            'title' => "Detail $this->title",
            'user' => $_SESSION['user']['nama'],
            'menu' => $this->menu,
            'detail' => $detail,
        ]);
    }

    public function update($id) { Router::to($this->url); }
    public function destroy($id) { Router::to($this->url); }
}
```

---

## Routing (`app/Router/route.php`)

```php
// 1. Import controller
use App\Controller\{FeatureName}Controller;

// 2. Add resource route
Router::resource('/{route-slug}', {FeatureName}Controller::class);
```

### Resource Routes Generated
| Method | Path | Controller |
|--------|------|------------|
| GET | `/{route}` | index() |
| GET | `/{route}/create` | create() |
| POST | `/{route}/store` | store() |
| GET | `/{route}/{id}` | show($id) |
| GET | `/{route}/{id}/edit` | edit($id) |
| PUT/PATCH/POST | `/{route}/{id}` | update($id) |
| DELETE | `/{route}/{id}` | destroy($id) |

---

## Menu (`app/Datas/MenuData.php`)

```php
$admin = [
    'pesanan' => '/pesanan',
    'stok' => '/stok',
    '{menu-key}' => '/{route-slug}',  // Add here
];
```

### Roles
- `$admin` - Admin role
- `$adminApp` - Admin Aplikasi role (merged with $admin)
- `$gudang` - Warehouse role

---

## Views (`app/View/`)

### Standard Views
```php
// index.php
<p><a href='{{ $addLink }}' class='btn btn-secondary'><i class="fa-solid fa-plus"></i> Tambah</a></p>
{{ component()->Table($table) }}

// form.php
{{ component()->Form($form)->setAction($action)->setMethod('post') }}
<p>
    <a href='{{ $_SERVER["HTTP_REFERER"] }}' class='btn btn-secondary'><i class="fa-solid fa-arrow-left"></i> Kembali</a>
    <button class="btn btn-secondary" onclick="$('#form').submit()">Simpan</button>
</p>

// detail.php
{{ component()->Detail($detail) }}
<p><a href='{{ $_SERVER["HTTP_REFERER"] }}' class='btn btn-secondary'><i class="fa-solid fa-arrow-left"></i> Kembali</a></p>
```

---

## Template Directives

| Directive | Output |
|-----------|--------|
| `{{ $var }}` | Echo variable |
| `@if($cond):` / `@endif` | If statement |
| `@elseif($cond):` | Else if |
| `@switch($var):` / `@case($val):` / `@break` / `@endswitch` | Switch |
| `@for($cond):` / `@endfor` | For loop |
| `@foreach($arr):` / `@endforeach` | Foreach loop |
| `@while($cond):` / `@endwhile` | While loop |

---

## Components

```php
component()->Table($data)->setClass()->setStyle()->setDecimal(2)
component()->Form($fields)->setAction()->setMethod()->setClass()->setStyle()
component()->Detail($data)->setClass()->setStyle()
```

---

## Helper Functions

| Function | Description |
|----------|-------------|
| `component()` | Get component instance |
| `data('{Name}')` | Get data class |
| `snake_to_capital()` | `nama_lengkap` → `Nama Lengkap` |
| `capital_to_snake()` | `Nama Lengkap` → `nama_lengkap` |
| `dump()` / `dd()` | Debug output |
| `err($error)` | Format error |
| `collect($arr)` | Create Collection |
| `number_id($num)` | Format Indonesian number |
| `date_id($ts)` | Format Indonesian date |

---

## Common Patterns

### Action Buttons
```php
"action" => "<a href='/{route}/{$id}' class='btn btn-sm btn-secondary'>Detail</a>
<a href='/{route}/{$id}/edit' class='btn btn-sm btn-secondary'>Edit</a>"
```

### Redirect
```php
Router::to($path);  // 302 redirect
```

### Session
```php
$_SESSION['user']['nama']
$_SESSION['user']['role']
```

---

## Implementation Checklist

### User (Before AI)
- [ ] Create `docs/{feature}.md` with route, menu, data table

### AI Agent
- [ ] Read and validate documentation
- [ ] Create `app/Datas/{FeatureName}Data.php`
- [ ] Create `app/Controller/{FeatureName}Controller.php`
- [ ] Register route in `app/Router/route.php`
- [ ] Add menu in `app/Datas/MenuData.php`
- [ ] Create views (if needed)

### User (After AI)
- [ ] Test all CRUD operations
- [ ] Verify menu and views work

---

## Framework Notes

**Limitations:**
- Prototyping only, not production-ready
- No database (in-memory Collections)
- Simple session authentication
- Uses `eval()` in View (security risk)

---

*For detailed workflow: `.ai/workflow/STANDARD-WORKFLOW.md`*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/haizim) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
