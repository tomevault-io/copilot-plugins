## zoh

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# AGENT.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ZOH is an embedded scripting language for interactive storytelling. Everything in ZOH is a "verb" - characters `/converse`, scenes `/show`, music `/play`. Even core language features like variables (`/set`), conditionals (`/if`), and loops (`/while`) are verbs.

**Key File/Dir**: `spec/` (complete language spec), `impl/` (implementation specs), `csharp/` (C# runtime), `projex/` (repository for tasks/projects/proposals)

## C# Reference Implementation

```bash
cd c# && dotnet build                                    # Build
cd c# && dotnet test                                     # Run all tests
cd c# && dotnet test --filter "FullyQualifiedName~Name"  # Run specific test
```

## Namespace Resolution

Nested namespace and forbid ambiguity.

```
/c.d.c;
/a.b.c;
/d.c; // VALID
/b.c; // VALID
/c; // INVALID
```

---

## Type System

| Type | Literal | Description |
|------|---------|-------------|
| nothing | `?` | Default/null type |
| boolean | `true`, `false` | Case-insensitive |
| integer | `42`, `-1` | Signed 64-bit |
| double | `3.14`, `-0.5` | Signed 64-bit float |
| string | `"text"`, `'text'` | Single or double quotes, supports `\n`, `\"`, `\\` |
| list | `[1, 2, 3]` | Collection of values |
| map | `{"key": value}` | String-keyed collection (keys must be strings) |
| channel | `<name>` | FIFO pipe for inter-context communication |
| expression | `` `*a + *b` `` | Unevaluated expression |
| reference | `*var`, `*data["k"][0]` | Pointer to variable with optional nested path |
| verb | `/verb params;` | Objectified verb call |

---

## Verb Syntax

```zoh
:: Line form
/verb [attr1] [attr2:value] named:param, param1, param2;

:: Block form (params delimited by whitespace/newlines)
/verb/
    [attr1] [attr2:value]
    named:param
    param1 param2
/;

:: Capture result
/verb param; -> *result;
```

---

## Syntactic Sugar

| Sugar | Desugars To |
|-------|-------------|
| `*var;` | `/set *var;` |
| `*var <- value;` | `/set *var, value;` |
| `*var [attr] <- value;` | `/set [attr] *var, value;` |
| `*list[i] <- value;` | `/set *list[i], value;` |
| `*map["k"] <- value;` | `/set *map["k"], value;` |
| `<- *var;` | `/get *var;` |
| `<- *var[index];` | `/get *var[index];` |
| `-> *var;` | `/capture *var;` |
| `` /`expr`; `` | `` /evaluate `expr`; `` |
| `/"Hello ${*name}";` | `/interpolate "Hello ${*name}";` |
| `====> @checkpoint;` | `/jump ?, "checkpoint";` |
| `====> @story:checkpoint;` | `/jump "story", "checkpoint";` |
| `====> @checkpoint *var;` | `/jump ?, "checkpoint", *var;` |
| `====+ @checkpoint;` | `/fork ?, "checkpoint";` |
| `<===+ @checkpoint;` | `/call ?, "checkpoint";` |

---

## Expression Grammar

Expressions are enclosed in backticks and evaluated by `/evaluate`:

```ebnf
expression := or_expr
or_expr    := and_expr ( '||' and_expr )*
and_expr   := equality_expr ( '&&' equality_expr )*
equality   := relational ( ( '==' | '!=' ) relational )*
relational := additive ( ( '<' | '>' | '<=' | '>=' ) additive )*
additive   := multiplicative ( ( '+' | '-' ) multiplicative )*
multiplicative := unary ( ( '*' | '/' | '%' ) unary )*
unary      := ( '!' | '-' ) unary | power_expr
power_expr := primary ( '**' unary )*
primary    := literal | '*' identifier [ '[' index ']' ] | '(' expression ')' | special_form
```

**Operator Precedence** (high to low): `**` (power) → `! -` (unary) → `* / %` → `+ -` → `< > <= >=` → `== !=` → `&&` → `||`

**Special Forms in Expressions**:
| Syntax                | Meaning |
|-----------------------|---------|
| `$*var`               | Interpolate the variable to string |
| `$*"string"`          | Interpolate the string |
| `$#(*var)`            | Count/length |
| `$?(*cond? *a \| *b)` | Conditional |
| `$?(*a\|*b\|*c)`      | First non-nothing |
| `$(1\|2\|3)[*i]`      | Index into options |
| `$(1\|2\|3)[!*i]`     | Index with wrap-around |
| `$(a\|b\|c)[%]`       | Random selection |
| `$(a:1\|b:2)[%]`      | Weighted random |

---

## Core Verbs Quick Reference

### Variables
| Verb | Purpose |
|------|---------|
| `/set *var, value;` | Declare/update variable |
| `/get *var;` | Get variable value |
| `/drop *var;` | Delete variable |
| `/capture *var;` | Store last return value |
| `/type *var;` | Get type as string |
| `/count *var;` | Get size/length |

### Control Flow
| Verb | Purpose |
|------|---------|
| `/if *cond, /verb;;` | Conditional execution |
| `/if *val, is: "x", /verb;, else: /other;;` | With comparison and else |
| `/sequence /v1;, /v2;, /v3;;` | Execute in order |
| `/loop N, /verb;;` | Repeat N times |
| `/while *cond, /verb;;` | Repeat while true |
| `/foreach *list, *it, /verb;;` | Iterate collection |
| `/switch *val, case1, result1, case2, result2, default;` | Pattern match |

### Navigation
| Verb | Purpose |
|------|---------|
| `/jump ?, "checkpoint";` | Jump to checkpoint in current story |
| `/jump "story", "checkpoint";` | Jump to checkpoint in another story |
| `/fork ?, "checkpoint";` | Start parallel context |
| `/call ?, "checkpoint";` | Fork and wait for completion |
| `/exit;` | Stop current context |

### Channels
| Verb | Purpose |
|------|---------|
| `/open <chan>;` | Create/reset channel |
| `/push <chan>, value;` | Send to channel |
| `/pull <chan>;` | Receive from channel (blocking) |
| `/pull <chan>, timeout: 5;` | Receive with timeout |
| `/close <chan>;` | Close channel |

### Collections
| Verb | Purpose |
|------|---------|
| `/append *list, value;` | Add to end of list |
| `/insert *list, index, value;` | Insert at position |
| `/remove *list, index;` | Remove by index/key |
| `/clear *list;` | Empty collection |
| `/has *list, value;` | Check membership |
| `/any *var;` | Check if not a nothing |
| `/first /v1;, /v2;;` | First non-nothing result |

### Evaluation
| Verb | Purpose |
|------|---------|
| `` /evaluate `expr`; `` | Evaluate expression |
| `/interpolate "Hello ${*name}";` | Interpolate string |
| `/do *verb_var;;` | Execute stored verb |

### Utility
| Verb | Purpose |
|------|---------|
| `/rand 1, 10;` | Random integer in range |
| `/roll *a, *b, *c;` | Random selection |
| `/wroll *a, 1, *b, 2;` | Weighted random |
| `/increase *var, 1;` | Increment value |
| `/decrease *var, 1;` | Decrement value |
| `/parse "1", "int";` | Parse string to value |
| `/sleep 1.5;` | Pause execution |
| `/wait "msg", timeout: 5;` | Wait for signal |
| `/signal "msg", val;` | Broadcast signal |
| `/diagnose;` | Get last diagnostics |
| `/try /risky;, catch: /handler;;` | Error handling |
| `/defer /cleanup;;` | Deferred execution |

### Persistence
| Verb | Purpose |
|------|---------|
| `/write *var;` | Save to store |
| `/read *var;` | Load from store |
| `/erase *var;` | Remove from store |
| `/purge;` | Clear all data |

### Debug
| Verb | Purpose |
|------|---------|
| `/info "msg";` | Log info |
| `/warning "msg";` | Log warning |
| `/error "msg";` | Log error |
| `/fatal "msg";` | Log fatal (exits) |

---

## Attributes

Attributes are optional metadata on verb calls: `[name]` or `[name:value]`

Common attributes:
- `[scope:"context"]` / `[scope:"story"]` - Variable scope for `/set`
- `[required]` - Require variable exists
- `[resolve]` - Evaluate verb/expr before storing
- `[typed:"integer"]` - Declare variable type
- `[OneOf:[1, 2]]` - Restrict value to list
- `[clone]` - Clone context state for fork/call
- `[inline]` - Copy variables back after call
- `[suppress]` - Suppress diagnostics in try

---

## Preprocessor Directives

```zoh
#embed "path/to/file.zoh";           :: Include file contents

|%MACRO_NAME%|                     :: Define macro
/converse "|%0|";                    :: Placeholder with |%0| (positional) or |%| (auto-inc)
|%MACRO_NAME%|                       :: End macro

|%MACRO_NAME|value|%|                :: Expand macro
```

---

## Story Structure

```zoh
Story Name
meta_key: meta_value;
===

:: Comment
::: Multi-line
    comment :::

@checkpoint_name *req *typed:integer
/verb;

====+ @parallel_context;   :: Fork
====> @next_scene;         :: Jump
```

---

## Scopes

- **Story scope**: Variables local to current story, dropped on story exit
- **Context scope**: Variables persist across stories within a context

Context variables are shadowed by story variables with same name.
Cross-context sharing only via channels.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/No3371) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
