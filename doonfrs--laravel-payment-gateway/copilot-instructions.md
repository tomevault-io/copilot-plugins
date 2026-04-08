## laravel-payment-gateway

> - Always take care of dark / light mode


# Rules

## Design Rules

- Always take care of dark / light mode
- Please try not to create svg as much as possible, and use blade icons
- We are using tailwind 4, please make sure to not to use deprecated classes / code <https://tailwindcss.com/docs/upgrade-guide#removed-deprecated-utilities> and if you encounter an old code, upgrade it
- Deprecated Replacement
bg-opacity-*Use opacity modifiers like bg-black/50
text-opacity-* Use opacity modifiers like text-black/50
border-opacity-*Use opacity modifiers like border-black/50
divide-opacity-* Use opacity modifiers like divide-black/50
ring-opacity-*Use opacity modifiers like ring-black/50
placeholder-opacity-* Use opacity modifiers like placeholder-black/50
flex-shrink-*shrink-*
flex-grow-*grow-*
overflow-ellipsis text-ellipsis
decoration-slice box-decoration-slice
decoration-clone box-decoration-clone

Renamed utilities
We've renamed the following utilities in v4 to make them more consistent and predictable:

v3 v4
shadow-sm shadow-xs
shadow shadow-sm
drop-shadow-sm drop-shadow-xs
drop-shadow drop-shadow-sm
blur-sm blur-xs
blur blur-sm
backdrop-blur-sm backdrop-blur-xs
backdrop-blur backdrop-blur-sm
rounded-sm rounded-xs
rounded rounded-sm
outline-none outline-hidden
ring ring-3

## Framework

- We are using laravel 12, and this is a package, not a full project, there is no artisan command for example
- make sure to keep the text translated and update the translations files

## Importrant

- Do not do try catch and hide the error, don't use it a lot, and if needed always use report($e)

## Unit test

- If you create or update a unit test, please run it and make sure it is successful
- Make sure to create integration tests also, not just a unit test ( low level ) so test the http request for the routes
- For livewire components ( if used / created ) use Livewire::test
- When you test a url, test also the locale version /en/mymethod so we make sure it works and does not return 404 or an error.

## Coding Style

- I prefer always to use named arguments when calling function as var:val
- Always try to use Auth::user instead of auth()->user if possible.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/doonfrs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/doonfrs)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
