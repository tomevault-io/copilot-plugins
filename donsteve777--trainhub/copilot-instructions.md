## angular-signals

> Angular zoneless change detection — usar signals para estado reactivo


# Angular 21 — Zoneless: usar siempre signals para estado

Este proyecto usa **Angular 21 en modo zoneless** (sin Zone.js). En este modo, asignar variables de estado normales no dispara Change Detection y el template no se actualiza.

## Regla

Todo estado que se use en un template **debe declararse como `signal()`**.

## ✅ Correcto

```typescript
import { signal } from '@angular/core';

posts = signal<Post[]>([]);

// Asignar
this.posts.set(dtos.map(...));

// Actualizar
this.posts.update(posts => posts.map(p => p.id === id ? { ...p, liked: true } : p));
```

```html
@for (post of posts(); track post.id) { ... }
```

## ❌ Prohibido

```typescript
// ❌ Variable normal — el template NO se actualiza en zoneless
posts: Post[] = [];
this.posts = dtos.map(...);

// ❌ Parches manuales de CD — nunca son la solución correcta
this.cdr.detectChanges();
this.ngZone.run(() => { ... });
```

No usar `ChangeDetectorRef`, `NgZone.run()` ni ningún otro mecanismo manual de CD como solución a problemas de renderizado. Si el template no actualiza, la causa es una variable de estado que debería ser un signal.

---
> Source: [DonSteve777/trainhub](https://github.com/DonSteve777/trainhub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
