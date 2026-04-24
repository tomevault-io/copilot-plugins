## server-manager

> When listening for Laravel Echo events in both Livewire (PHP) and JavaScript, you **must prefix the event class name with a dot (`.`)**. This is required for Echo to match the event name broadcast by Laravel.

# Laravel Echo Event Naming in Livewire & JavaScript

## Important: Dot Prefix for Event Names

When listening for Laravel Echo events in both Livewire (PHP) and JavaScript, you **must prefix the event class name with a dot (`.`)**. This is required for Echo to match the event name broadcast by Laravel.

---

## Livewire Example

```php
// Correct usage in Livewire 3+ component:
#[On('echo:GlobalChat,.App\\Events\\Chat\\MessageEvent')]
public function receiveMessage($payload) { /* ... */ }
```

- The dot (`.`) before the event class is required.
- This applies to all Echo event listeners in Livewire using the #[On] attribute.

---

## JavaScript Example

```js
// Correct usage in JS with Echo:
Echo.channel('GlobalChat').listen('.App\\Events\\Chat\\MessageEvent', (e) => {
    console.log('Received:', e);
});
```

- The event name string must start with a dot (`.`).
- This matches the event name broadcast by Laravel.

---

## Why?

Laravel broadcasts events with a dot-prefixed name by default. Echo (and Livewire) require the dot to match the event name exactly. Omitting the dot will result in listeners not firing, even if the event is broadcast successfully.

**Always use the dot prefix for event class names in Echo listeners!**
description:
globs:
alwaysApply: false
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BCleverly) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
