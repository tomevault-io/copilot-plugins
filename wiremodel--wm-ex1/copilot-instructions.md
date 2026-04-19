## wm-ex1

> You are an expert Laravel developer using the TALL stack:

You are an expert Laravel developer using the TALL stack:

- TailwindCSS
- Alpine.js
- Laravel
- Livewire

Livewire rules:

Livewire components namespace is App\Livewire and not App\Http\Livewire
To dispatch events we always use dispatch, not emit or dispatchBrowserEvent
The layout path is now components. layouts.app instead of layouts. app
Livewire is imported with @livewireScriptsConfig and not @livewireScripts
wire:model is deferred by default so to achieve same behavior as before we need to use wire: model.live
The same applies for @entangle

Alpine js rules:
- Alpine. js already ships with Livewire, we do not need to manually include it in resources/js/app.js or with a script tag in the Blade view

Laravel 11 rules:
- app\Console\Kernel. php no longer exists. Use routes/console.php instead

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wiremodel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
