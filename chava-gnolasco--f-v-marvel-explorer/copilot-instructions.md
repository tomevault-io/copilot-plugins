## f-v-marvel-explorer

> Welcome! This project is designed for comic enthusiasts. We'll be building an explorer application that leverages the Marvel Comics API to browse and discover Marvel characters, comics, and related information.

# Project context

Welcome! This project is designed for comic enthusiasts. We'll be building an explorer application that leverages the Marvel Comics API to browse and discover Marvel characters, comics, and related information.

## Tech stack

- vue.js version 3
- tailwindcss
- typescript
- pinia for state management
- vite for bundling
- Router vue for routing
- axios for HTTP requests

## Project estructure

```text
src
├── assets          # Static assets like images, fonts and css
├── components      # Reusable components
├── composables     # Composition API functions
├── services        # API services and utilities
├── types           # TypeScript type definitions
├── pages           # Page components
├── router          # Vue Router setup
└── store           # Pinia store
```

## Technical requirements

- Register at [Marvel Developer Portal](https://developer.marvel.com/) to retrieve an API key for accessing the Marvel Comics API.
- Add [Tailwind CSS](https://tailwindcss.com/) to the project for styling the application.


## Public API key

The public and private variables are set on .env environment variables. You can create a .env file in the root of the project and add the following variables:

- VITE_APP_MARVEL_PUBLIC_API_KEY
- VITE_APP_MARVEL_PRIVATE_API_KEY

## Authentication for Server-Side Applications

Server-side applications must pass two parameters in addition to the apikey parameter:

ts - a timestamp (or other long string which can change on a request-by-request basis)
hash - a md5 digest of the ts parameter, your private key and your public key (e.g. md5(ts+privateKey+publicKey)
For example, a user with a public key of "1234" and a private key of "abcd" could construct a valid call as follows: http://gateway.marvel.com/v1/public/comics?ts=1&apikey=1234&hash=ffd275c5130566a2916217b101f26150 (the hash value is the md5 digest of 1abcd1234)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chava-gnolasco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
