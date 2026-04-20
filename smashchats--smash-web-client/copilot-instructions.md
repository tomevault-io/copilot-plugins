## smash-web-client

> This project uses a modular, feature-based architecture. Key points:


This project uses a modular, feature-based architecture. Key points:

- All UI uses Tailwind CSS and variables from [globals.css](mdc:src/styles/globals.css).
- All component props are wrapped in `Readonly<T>`.
- Functional code is preferred, and files are kept short and readable.
- Variable names are explicit and descriptive.
- Code duplication is avoided; helpers and hooks are refactored as needed.

src/
├── app/ # Global app-level setup
│ ├── App.tsx
│ ├── routes.tsx
│ ├── config/ # Global config, constants
│ │ └── env.ts
│ ├── providers/ # App-wide contexts
│ │ └── AppProviders.tsx
│ └── i18n/ # Localization files
│ ├── index.ts
│ └── resources.ts
│
├── features/ # Feature-based structure
│ ├── chat/
│ │ ├── components/
│ │ │ ├── ChatInput.tsx
│ │ │ └── MessageItem.tsx
│ │ ├── hooks/
│ │ │ ├── useChatStore.ts
│ │ │ └── useMessageStore.ts
│ │ ├── screens/
│ │ │ ├── ChatListScreen.tsx
│ │ │ └── ChatScreen.tsx
│ │ ├── types/
│ │ │ └── chat.d.ts
│ │ └── index.ts # Entry point for chat feature
│
│ ├── camera/
│ │ ├── components/
│ │ ├── hooks/
│ │ │ └── useCaptureMedia.ts
│ │ ├── screens/
│ │ ├── index.ts
│
│ ├── gallery/
│ └── profile/
│
├ # Shared across features
│── components/
│ ├── Button.tsx
│ ├── Spinner.tsx
│ └── ErrorBoundary.tsx
│── hooks/
│── utils/
│── styles/
│ ├── globals.css
│ └── variables.css
│── types/
|
├ services/ # API and service abstraction
│── smashService.ts
│── mediaStore.ts
│── messaging.ts
│
└── main.tsx # App entry point

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smashchats) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
