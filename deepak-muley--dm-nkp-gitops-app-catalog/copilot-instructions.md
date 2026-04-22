## dm-nkp-gitops-app-catalog

> Go language standards for this repository — interfaces, validation, constructors


# Golang Standards

When writing or modifying Go code in this repository, follow these standards.

## 1. Every struct has an interface

- For each **struct** that is part of the package’s API or is injected/tested, define a **interface** that describes its behavior.
- Name the interface after the behavior (e.g. `Cluster`, `Client`) or as `StructName` + `er`/abbreviation if it represents the struct’s role.
- Prefer **small interfaces** (few methods); consumers depend on the interface, not the concrete type.

```go
// Interface: behavior exposed to callers
type Cluster interface {
    Name() string
    KubeconfigFilePath() string
    Delete(ctx context.Context) error
}

// Struct: implementation
type KindCluster struct { ... }

func (k *KindCluster) Name() string { ... }
func (k *KindCluster) KubeconfigFilePath() string { ... }
func (k *KindCluster) Delete(ctx context.Context) error { ... }
```

## 2. Interface validation line

- Add a **compile-time check** that the concrete type implements the interface.
- Place it near the type definition (e.g. right after the struct or its methods).

```go
// Ensure KindCluster implements Cluster at compile time.
var _ Cluster = (*KindCluster)(nil)
```

- If the struct implements multiple interfaces, add one line per interface:

```go
var _ Cluster = (*KindCluster)(nil)
var _ io.Closer = (*KindCluster)(nil)
```

- This catches interface drift when methods are renamed or removed.

## 3. Constructors: `New<Type>()`

- **Create instances via constructors** named `New<Type>` (e.g. `NewKindCluster`, `NewClient`).
- Avoid constructing structs with `&StructName{...}` from other packages; keep that inside the defining package and expose `New<Type>(...)` instead.
- Use **consistent naming**: `New<Type>(requiredArgs ...) ( *Type, error)` or `New<Type>(...) *Type` when errors are not returned.

```go
// Good: constructor returns the interface or concrete type
func NewKindCluster(ctx context.Context, name string) (Cluster, error) {
    return createKindCluster(ctx, name)
}

// Good: constructor for a struct used only in this package
func NewClient(kubeconfigPath string) (ctrlClient.Client, *kubernetes.Clientset, error) { ... }
```

- For **optional configuration**, use functional options or a config struct:

```go
func NewFoo(opts ...FooOption) *Foo { ... }
// or
func NewFoo(cfg FooConfig) (*Foo, error) { ... }
```

## Summary checklist

- [ ] Struct has a corresponding interface (when it’s part of API or testable surface).
- [ ] Interface validation line present: `var _ Interface = (*Struct)(nil)`.
- [ ] Instances are created via `New<Type>(...)`; no external `&StructName{...}` for exported types.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/deepak-muley) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
