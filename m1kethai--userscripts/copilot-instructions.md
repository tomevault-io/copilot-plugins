## userscripts

> This repository contains a collection of custom JavaScript UserScripts designed to be used with browser extensions like Tampermonkey, Greasemonkey, or Violentmonkey. These scripts enhance the functionality and UI of various websites.

# UserScripts

## Project Overview
This repository contains a collection of custom JavaScript UserScripts designed to be used with browser extensions like Tampermonkey, Greasemonkey, or Violentmonkey. These scripts enhance the functionality and UI of various websites.

Currently, the project includes the following script(s):
- **NixOS Package Search Stargazers (`nixos-pkg-stargazers/`)**: A script that fetches and displays the GitHub repository star count for packages on the NixOS package search page (`https://search.nixos.org/packages`). It uses the GitHub public API to retrieve the stargazer counts.

## Building and Running
UserScripts do not require a traditional build process.
- **To Install**: Users can typically install these scripts by clicking on a raw `.user.js` file link or installing them through a distribution platform like Greasy Fork (as indicated by the `@downloadURL` and `@updateURL` metadata tags).
- **To Run/Test Locally**: You can copy the contents of a `.user.js` file into your browser's UserScript manager extension to test it live on the target website. Changes made to the file locally will need to be pasted back into the extension, or you can configure your extension to track the local file path.

## Development Conventions
- **Format**: Each script must be a standalone `.user.js` file.
- **Metadata Block**: Scripts must start with a standard UserScript metadata block (e.g., `// ==UserScript== ... // ==/UserScript==`) containing required tags such as `@name`, `@namespace`, `@version`, `@match` (target URLs), and `@grant`.
- **Scoping**: Code should be wrapped in an IIFE (Immediately Invoked Function Expression) `(function() { ... })();` to avoid polluting the global window scope.
- **Strict Mode**: Use `'use strict';` inside the IIFE.
- **External APIs**: When interacting with external APIs (like GitHub), handle rate limits appropriately. For example, the NixOS script supports providing a local GitHub token via `localStorage.ght` to bypass unauthenticated rate limits.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/m1kethai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/m1kethai)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
