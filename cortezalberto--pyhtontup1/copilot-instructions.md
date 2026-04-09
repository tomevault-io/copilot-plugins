## pyhtontup1

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Educational Python project (Equipo: Desarrollo Los Cortez) demonstrating four GoF design patterns through a simulated e-commerce scenario. The system models four concrete problems: a shared DB connection (Singleton), a mixed product catalogue of books and electronics (Factory), a customer notification system for promotions (Observer), and a flexible checkout with interchangeable payment methods (Strategy). Uses only the Python standard library — no external dependencies. Keep changes focused on the educational purpose of the repository.

All four patterns follow the same internal structure: an abstract base class (ABC) defines the interface, and one or more concrete classes implement it. Each pattern lives in its own subpackage under `patrones/`.

## Running the Project

```bash
python main.py
```

Quick sanity checks for individual patterns:

```bash
# Singleton — verifica que dos llamadas devuelven el mismo objeto
python -c "from patrones.singleton import DatabaseConnection; db1 = DatabaseConnection.get_instance(); db2 = DatabaseConnection.get_instance(); print('OK' if db1 is db2 else 'FAIL')"

# Factory — verifica creación de producto por método tipado
python -c "from patrones.factory import ProductoFactory; p = ProductoFactory.crear_libro('Test', 10.0, 'Autor'); print('OK' if p.get_nombre() == 'Test' else 'FAIL')"

# Factory — verifica creación genérica por string y error para tipo inválido
python -c "from patrones.factory import ProductoFactory; p = ProductoFactory.crear_producto('libro', 'Test', 10.0); print('OK' if p.get_nombre() == 'Test' else 'FAIL')"

# Observer — verifica suscripción, notificación y rechazo de duplicados
python -c "
from patrones.observer import Cliente, Tienda
t = Tienda('Test'); c = Cliente('Ana')
t.suscribir(c); t.suscribir(c)  # segundo suscribir debe rechazarse
print('OK' if len(t._clientes) == 1 else 'FAIL')
"

# Strategy — verifica checkout y reset del carrito
python -c "
from patrones.factory import ProductoFactory
from patrones.strategy import CarritoCompra, PagoEfectivo
c = CarritoCompra(); c.set_estrategia_pago(PagoEfectivo())
c.agregar_producto(ProductoFactory.crear_libro('Test', 10.0, 'Autor'))
c.checkout()
print('OK' if c._productos == [] else 'FAIL')
"
```

No build step, no linter configured, no test framework. Target Python 3.6+.

## Architecture

`main.py` is the entry point — imports from `patrones` and demos each pattern in sequence inside `main()`. Products created in the Factory section (`libro1`, `laptop`, etc.) are deliberately reused in the Strategy section to show realistic inter-pattern integration.

The `patrones/` package is organized into subpackages (one per pattern), each with an `__init__.py` that re-exports public classes:

```
patrones/
├── __init__.py                  # docstring listing all subpackages
├── singleton/
│   ├── __init__.py              # re-exports DatabaseConnection
│   └── database_connection.py
├── factory/
│   ├── __init__.py              # re-exports Producto, Libro, Electronico, ProductoFactory
│   ├── producto.py
│   └── producto_factory.py
├── observer/
│   ├── __init__.py              # re-exports Observer, Cliente, Tienda
│   ├── observer.py
│   └── tienda.py
└── strategy/
    ├── __init__.py              # re-exports EstrategiaPago, Pago*, CarritoCompra
    ├── estrategia_pago.py
    └── carrito_compra.py
```

| Pattern | Subpackage | Key Classes |
|---------|------------|-------------|
| Singleton | `singleton/` | `DatabaseConnection` — `__new__` + `_initialized` flag prevent re-init |
| Factory | `factory/` | Abstract `Producto` → `Libro`, `Electronico`; `ProductoFactory` (static methods, raises `ValueError` for unknown types) |
| Observer | `observer/` | Abstract `Observer` → `Cliente`; `Tienda` validates `isinstance`, rejects duplicates, handles missing subscriber gracefully |
| Strategy | `strategy/` | Abstract `EstrategiaPago` → `PagoTarjeta`, `PagoEfectivo`, `PagoPayPal`; `CarritoCompra` resets after checkout |

All imports use subpackage re-exports, e.g. `from patrones.singleton import DatabaseConnection`.

## Key Implementation Details

- **Singleton**: `get_instance()` simply calls `DatabaseConnection()` — `__new__` handles the guard. The `_initialized` flag in `__init__` prevents re-execution since Python always calls `__init__` after `__new__`.
- **Factory**: `ProductoFactory.crear_producto(tipo, ...)` accepts a case-insensitive string and raises `ValueError` for unknown types. `crear_libro` / `crear_electronico` are typed convenience methods.
- **Observer — Tienda.suscribir()**: validates `isinstance(cliente, Observer)` raising `TypeError`, and short-circuits on duplicate without error.
- **Observer — Tienda.desuscribir()**: checks `if cliente not in self._clientes` before `remove()` to avoid `ValueError`.
- **Strategy — PagoEfectivo.pagar()**: uses `round(monto * 0.95, 2)` — explicitly guards against floating-point precision errors.
- **Strategy — CarritoCompra.checkout()**: guards empty cart and `None` strategy before processing; clears `_productos` and `_estrategia_pago` after purchase.

## Code Conventions

- Classes: `CamelCase`. Methods/variables: `snake_case`. Protected attributes: `_single_underscore`.
- Abstract interfaces use `abc.ABC` + `@abstractmethod`. Factory methods use `@staticmethod`.
- All output is via `print()` to stdout — no real DB, network, or file I/O.
- Keep lines under ~100 chars. Use f-strings for formatting.
- All code, output, and documentation is in Spanish.
- Every module, class, and method has a Google-style docstring (Args/Returns/Raises sections). Maintain this when adding new code.
- No type hints currently; acceptable to add if they improve clarity.

## Extending the Project

When adding new product types, observers, or payment strategies follow the existing idiom: define a new concrete class that inherits from the relevant ABC (`Producto`, `Observer`, `EstrategiaPago`). For new payment methods, use constructor injection via `set_estrategia_pago`. Patterns are intentionally decoupled — avoid coupling modules from different subpackages. Suggested improvements noted in README: `@property` decorators, `dataclasses`, `decimal.Decimal` for monetary values, context managers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cortezalberto)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cortezalberto)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
