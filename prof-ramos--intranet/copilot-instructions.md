## intranet

> - **Framework**: Laravel 11.x

# CLAUDE.md - Intranet ASOF

## Contexto do Projeto

- **Framework**: Laravel 11.x
- **PHP**: 8.2+
- **Frontend**: Blade + Alpine.js 3.x
- **Testes**: Pest (NÃO PHPUnit)
- **Banco de Dados**: MySQL 8.0+ / PostgreSQL 13+
- **Autenticação**: Laravel Breeze + Sanctum

---

## 🇧🇷 Idioma Obrigatório: Português (Brasil)

**REGRAS ABSOLUTAS** para TODAS as interações, código e documentação:

### Comunicação e Documentação
- **Todos** os comentários no código devem ser em **português**
- **Toda** a documentação (README, docs/, etc.) deve ser em **português**
- **Todas** as mensagens de commit devem ser em **português**
- **Todas** as interações com o usuário devem ser em **português**

### Código e Elementos
- **Variáveis, funções, classes**: em **inglês** (padrão da indústria)
- **Comentários**: em **português**
- **Mensagens de erro/exceção**: em **português** (para o usuário final)
- **Mensagens de validação**: em **português**
- **Nomes de banco de dados**: em **inglês** (`users`, `tasks`, `contacts`)
- **Enum values**: em **inglês** (`todo`, `progress`, `done`)
- **UI/Labels**: em **português** ("Título", "Descrição", "Prazo")

### Exemplos

```php
// ✅ CORRETO - comentário em PT-BR, código em inglês
public function scopeAtrasadas($query)
{
    return $query->where('deadline', '<', now())
        ->where('status', '!=', TaskStatus::Concluida);
}

// ✅ CORRETO - mensagem de validação em PT-BR
'title' => 'required|string|max:255', // Título da tarefa é obrigatório

// ✅ CORRETO - mensagem de exceção em PT-BR
throw new \Exception('Tarefa não encontrada');

// ❌ ERRADO - comentário em inglês
public function scopeOverdue($query) // não usar
```

### Mensagens de Commit (em português)

```
feat(kanban): adiciona arrastar e soltar para reordenar tarefas
fix(calendario): corrige tratamento de fuso horario para prazos
refactor(tarefas): extrai metricas para repositorio
test(tarefas): adiciona testes de conclusao de tarefas
```

---

## Diretrizes Específicas do Laravel

### Organização do Código

- Use **Enums** do PHP 8.1+ para status/prioridade (`TaskStatus`, `TaskPriority`)
- Siga a estrutura simplificada do Laravel 11 (sem `App\Http\Requests` a menos que necessário)
- Use Ações de propósito único em `app/Actions/`
- Use padrão Repository para queries complexas
- Mantenha controllers magros - lógica de negócio em Services/Actions

### Migrations

- Sempre use restrições de chave estrangeira apropriadas
- Adicione índices para colunas frequentemente consultadas
- Use `->nullable()` com valores padrão apropriados
- Soft deletes onde apropriado
- Use `->constrained()` para chaves estrangeiras

```php
$table->foreignId('assigned_to')
    ->nullable()
    ->constrained('users')
    ->nullOnDelete();
```

### Models

- Use casting de atributos para enums e datas
- Use atributos do PHP 8 para observers: `#[ObservedBy([])]`
- Defina relacionamentos com tipos de retorno
- Use query scopes para queries comuns

```php
#[ObservedBy([TaskObserver::class])]
class Task extends Model
{
    protected $casts = [
        'status' => TaskStatus::class,
        'priority' => TaskPriority::class,
        'deadline' => 'datetime',
    ];

    // Escopo para tarefas atrasadas
    public function scopeAtrasadas($query)
    {
        return $query->where('deadline', '<', now())
            ->where('status', '!=', TaskStatus::Concluida);
    }
}
```

### Controllers

- Use `Route::resource()` para CRUD padrão
- Use controllers invocáveis para ações únicas
- Retorne JSON para rotas de API
- Use validação inline para casos simples, FormRequest para complexos

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|string|max:255',
        'status' => ['required', Rule::enum(TaskStatus::class)],
    ]);
}
```

### Testes (Pest)

- Use sintaxe do Pest: `test()->expect()->toBe()`
- Use `()->expect()` em vez de `()->assertEquals()`
- Mock facades adequadamente com `Http::fake()`
- Teste operações de banco com `refreshDatabase()`

```php
test('tarefa pode ser criada', function () {
    $task = Task::factory()->create([
        'status' => TaskStatus::Todo,
    ]);

    expect($task->status)->toBe(TaskStatus::Todo);
});

test('tarefa pode ser marcada como concluída', function () {
    $task = Task::factory()->create(['status' => TaskStatus::Todo]);

    $task->update(['status' => TaskStatus::Concluida]);

    expect($task->status)->toBe(TaskStatus::Concluida);
});
```

### Frontend (Blade + Alpine)

- Use Alpine `x-data` para componentes interativos
- Use `@props` e `@attributes` para componentes
- Mantenha JS em `resources/js/alpine/`
- Use Livewire 3 para interatividade complexa (opcional)

```javascript
// resources/js/alpine/task-kanban.js
export default () => ({
    tasks: [],
    init() {
        this.fetchTasks();
    },
    async fetchTasks() {
        this.tasks = await (await fetch('/api/tasks')).json();
    }
});
```

---

## Convenção de Git Commit

Use conventional commits (em português):

| Tipo | Uso |
|------|-----|
| `feat` | Nova funcionalidade |
| `fix` | Correção de bug |
| `refactor` | Refatoração de código |
| `docs` | Documentação |
| `test` | Testes |
| `chore` | Manutenção |

Exemplos:
```
feat(kanban): adiciona arrastar e soltar para reordenar tarefas
fix(calendario): corrige tratamento de fuso horario para prazos
refactor(tarefas): extrai metricas para repositorio
test(tarefas): adiciona testes de conclusao de tarefas
```

---

## Padrões Específicos do Projeto

### Fluxo de Status da Tarefa

```
todo → progress → review → done
         ↓
      blocked
```

### Níveis de Prioridade

```
low < normal < high < urgent
```

### Entidades Principais

- `users` - Usuários do sistema (autenticação)
- `contacts` - CRM (relacionamentos institucionais)
- `tasks` - Tarefas operacionais com prazos
- `task_history` - Histórico de auditoria para mudanças de status

---

## Comandos de Desenvolvimento

```bash
# Executar testes
pest

# Testes com cobertura
pest --coverage

# Migration fresh com seed
php artisan migrate:fresh --seed

# Worker de fila
php artisan queue:work

# Limpar caches
php artisan optimize:clear
```

---

## Cuidados Importantes (Gotchas)

1. **Sempre use Query Builder em vez de SQL puro** para portabilidade de banco (MySQL/PostgreSQL)
2. **Use Pest, não PHPUnit** - este projeto usa o framework de testes Pest
3. **Valores de Enum são armazenados como strings** no banco, faça cast no model
4. **Observers são registrados via atributos do PHP 8** - não é necessário registro manual no provider
5. **Soft deletes estão ativos** - lembre-se de usar `withTrashed()` quando necessário

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prof-ramos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
