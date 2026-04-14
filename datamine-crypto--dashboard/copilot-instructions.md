## dashboard

> Read the information about this project in docs/aiContext.md and README.md files

Read the information about this project in docs/aiContext.md and README.md files

Source is located in `/src` folder. Read that folder as it contains a lot of comments which explain many concepts

Be sure to read package.json for all the packages/frameworks that we are using as it's very important to know what versions of React and Material UI we are using.

I will sometimes ask you to add some suggested content for GEMINI.md to improve your understand of this project in future runs (so you don't have to do as much research).

Be sure to make your own suggestions to update GEMINI.md as you learn of new concepts you might find useful on a restart. But these have to be new contexts, not your list of updates.

### Context Map Instructions

The "Context Map" has been added to `README.md` right after the "Quick Start Guide" section. The "Context Map" represents a nested navigation of all the paths and concepts within the Datamine Network.

This project is a React-based web application for interacting with the Datamine Network. It uses Material-UI for components and styling and Viem for blockchain interactions. The project is configured to work with multiple blockchain
ecosystems, as defined in the `src/configs/ecosystems` directory. The core logic is separated into three main directories: `react` for UI components, `utils` for utility functions, and `web3` for blockchain-related logic. The project uses
Vite for building and development, with scripts defined in `package.json`.

When making Material UI updates please use <Grid size={{}}> which is new in MUI v7.1 (Do not use <Grid item xs=...> style)

<Grid size={{ xs: 12, md:6 }}> is CORRECT. <Grid item xs={12} md={6}> is INCORRECT

# Gemini Project Context: Datamine Network Dashboard

## Project Overview

This project is a web-based dashboard for the Datamine Network, built with React and TypeScript. It provides users with tools to interact with Datamine smart contracts, view analytics, and manage their assets across different blockchain layers.

## Core Technologies

- **Framework:** React v19.2.0
- **UI Library:** Material-UI (MUI) v7.3.5
- **Blockchain Interaction:** Viem v2.40.0
- **Language:** TypeScript v5.9.3
- **Build Tool:** Vite v7.2.2
- **Package Manager:** Yarn v4.10.3

## Key Architectural Patterns

The `src` directory is organized with a clear separation of concerns:

- **`src/`**: Contains the application's core logic.
  - **`src/react/`**: Houses all React components, pages, and UI-related elements.
  - **`src/web3/`**: Manages all blockchain interactions using Viem, including ABI definitions (located in `src/web3/abis/`).
  - **`src/utils/`**: A collection of helper functions for tasks like formatting, calculations, and clipboard interaction.
  - **`src/app/`**: Contains application configuration and state management logic.
    - **`src/app/configs/`**: Manages all environment and application configurations.
    - **`src/app/state/`**: Contains the reducer logic and state interfaces.

## Development Workflow

The following commands are used for development:

- `yarn start`: Runs the application in development mode using Vite.
- `yarn build`: Compiles and bundles the application for production.
- `yarn deploy`: Creates a production build with a specific base path for deployment.
- `yarn format`: Formats code using Prettier.
- `yarn lint`: Lints the project using ESLint.

Extra information help you understand the Datamine ecosystem better:

#### 1. Styling Conventions & Theming

- For styling we're using `tss-react` and `useStyles` from `tss.create()`
- `src/react/styles.ts` contains our MUI themes

#### 2. State Management

- **Modular Reducer:** The `Web3Reducer` logic has been refactored into modules within `src/app/state/`.
- **Zustand Store:** We now use `zustand` for state management instead of React Context. The store is defined in `src/react/appStore.ts`.
- `src/react/appStore.ts` wraps the `sideEffectReducer` and handles the execution of side effects (queries).
- `src/app/state/` contains the reducer logic. We use `commonLanguage` (in `src/app/state/commonLanguage.ts`) as "Commands & Queries" pattern.
- `sideEffectReducer.ts` contains the logic for handling queries
- `handleCommand.ts` and `handleQuery.ts` control state and update `pendingQueries`. `pendingQueries` are converted into async calls. This is a creative way to manage state & separate out async logic.
- The `AppState` interface (in `src/app/state/initialState.ts`) defines the complete application state.
- The reducer persists user settings (e.g., selected ecosystem, currency) in `localStorage`.

#### 3. Ecosystem Configuration Details

- There are 3 ecosystems in Datamine Network: DAM->FLUX(L1), FLUX(L2)->ArbiFLUX(L2), ArbiFLUX(L2)->LOCK(L2)
- You can toggle between any ecosystem in the decentralized dashboard.
- Settings for each of these ecosystems are located in `/src/app/configs/ecosystems`. Each file (e.g., `config.ecosystem.dam_flux_l1.ts`) contains all necessary parameters for a specific chain.
- L2 here means Arbitrum Layer 2.

#### 4. Build & Development (`vite.config.mts`)

- **Build Output:** The project builds to a `build` directory with hashed asset filenames.
- **Development Server:** Runs on port `3000` and opens the browser on start. HTTPS is enabled via `vite-plugin-mkcert` for secure local development.
- **Optimizations:** Includes experimental optimizations for `@mui/material` and `@mui/icons-material`.

#### 5. Linting (`.eslintrc.json`)

- The `.eslintrc.json` file has been configured to exclude the `build` directory from linting and to recognize `BigInt` and `globalThis` by setting the `browser`, `es2021`, and `node` environments.

#### 5. Application Structure (`src/App.tsx`)

- **Root Component:** `App.tsx` is the main entry point.
- **Core Providers:** It sets up `ThemeProvider` (MUI), `ErrorBoundary`, and `Web3ContextProvider`.
- **Code Splitting:** Uses `React.lazy` and `Suspense` to lazy-load major components, improving initial load times.

#### 6. Key Smart Contracts and ABIs

- `src/web3/abis/dam.ts`: ABI for the Datamine (DAM) token contract.
- `src/web3/abis/flux.ts`: ABI for the Flux (FLUX) token contract.
- `src/web3/abis/uniswapv2router.ts`: ABI for the Uniswap V2 Router.
- `src/web3/abis/uniswapPair.ts`: ABI for Uniswap V2 Pair contracts.
- `src/web3/abis/uniswapPairV3.ts`: ABI for Uniswap V3 Pair contracts.
- `src/web3/abis/multicall.ts`: ABI for the Multicall contract.
- `src/web3/abis/batchMinter.ts`: ABI for the Batch Minter contract.
- `src/web3/abis/games/datamineGems.ts`: ABI for the core Datamine Network market contract (minting, burning, staking).
- `src/web3/abis/games/gameHodlClicker.ts`: ABI for the HODL Clicker game contract.

#### 7. Error Handling Strategy

- Global error handling for Web3 transactions is managed via `web3Reducer.ts` to update the `error` state. User-facing error messages are typically displayed via Material-UI Snackbars or custom dialogs triggered by the `dialog` state in `web3Reducer`.

#### 8. Third-Party Integrations (Beyond Web3)

- Help article content is fetched from Markdown files in `public/helpArticles/` via standard `fetch` API calls.
- Search functionality for help articles uses `fuse.js`.

#### 9. Testing Strategy

- No tests are currently implemented.

#### 10. GitHub Pages Deployment

- The `homepage` field in `package.json` is set to `.`, which is crucial for GitHub Pages deployments when the site is hosted in a subfolder (e.g., `your-username.github.io/your-repo-name/`). This ensures that relative paths for assets are correctly resolved.
- The GitHub Actions workflow (`.github/workflows/deploy.yml`) now explicitly sets `yarn-version: 4.10.3` in the `setup-node` step, ensuring the correct Yarn version is used during the build process.

#### 11. Additional Development Setup & Tools

- **HTTPS for Localhost**: Implemented using `vite-plugin-mkcert` in `vite.config.mts` for secure local development.
- **ESLint Configuration**: The `.eslintrc.json` file has been updated to exclude the `build` directory from linting and to correctly recognize `BigInt` and `globalThis` by setting the `browser`, `es2021`, and `node` environments.
- **Pre-commit Hooks**: `husky` and `lint-staged` are configured to automatically run ESLint and Prettier on staged files before committing, ensuring code quality and consistency.
- **VS Code Workspace Hiding**: The `.vscode/settings.json` file is configured to hide various development-related files and folders (e.g., `.husky`, `.yarn`, `.vscode`, `.github`, `.env`, `.eslintrc.json`, `.gitignore`, `.pnp.cjs`, `.pnp.loader.mjs`, `.prettierrc.json`, `index.html`, `LICENSE`, `tsconfig.json`, `tsconfig.node.json`, `vite.config.mts`, `yarn.lock`, `build`, `node_modules`, `src/react-app-env.d.ts`, `src/service-worker.ts`, `src/serviceWorkerRegistration.ts`, `src/vite-env.d.ts`) from the VS Code explorer to reduce clutter.
- **`yarn lint` script**: A new `lint` script has been added to `package.json` to easily run ESLint across the project.

#### 12. New Features & GameFi (from 2025-05-27 onwards)

- **Datamine Market (Decentralized "Time-in-market" solution):** A revolutionary way to decentralize "Time-in-market" and create true decentralized demand. Validators can offer a percentage reward for others to burn LOCK to their account, effectively decentralizing demand and increasing market efficiency. This happens seamlessly in a single Ethereum transaction.
- **Datamine Gems #GameFi:** A real-time game where users collect "gems" (public market addresses with unminted balances) by burning tokens. It features a new V2 Public Market smart contract for even greater throughput, atomic batch burning from multiple addresses, and optimized reward distribution. A "Collect all gems" button allows for single-transaction collection from multiple addresses. This aims to increase monetary velocity and transactional throughput.

#### 13. Lockquidity (LOCK) Token

- **Purpose:** Enhances stability by contributing to a permanent liquidity pool.
- **Minting:** Minted by locking ArbiFLUX.
- **Burning Mechanism:** Burning LOCK redirects value to the liquidity pool instead of reducing supply, ensuring long-term stability.
- **Market Efficiency:** A new metric defined as 100% - percentage of LOCK inside the market. Higher market efficiency indicates more trading volatility and benefits validators.

#### 14. Core Values & Ecosystem Principles

- **Transaction-incentivized Liquidity Pools:** DAM and FLUX offer unique incentives for providing liquidity on Uniswap & Balancer pools, ensuring constant token movement.
- **On-Chain Linear Deflation:** FLUX supply is non-fixed and features predictable deflation through a burning mechanism, with its generation tied to DAM staking.
- **Realtime Multi-Smart Contract Analytics:** The dashboard provides real-time on-chain market sentiment, balances in USD, and analytics through deep Uniswap integration.
- **Secure By Design & Professionally Audited:** All business logic is executed via audited smart contracts, ensuring fund safety and security without third-party involvement.
- **Built For The Community:** Utilizes serverless, web3, and mobile technologies for a seamless and secure user experience.
- **Global Problem Solved:** The Datamine Ecosystem aims to solve inflation through its deflationary tokenomics and on-chain demand generation.

#### 15. Token Specifications & Mechanics

- **DAM (Datamine Token):** Fixed supply (16,876,778 tokens), ERC-777 standard, primarily used for staking to power validators.
- **FLUX (Flux Token):** Non-fixed supply, base currency of the ecosystem, features linear and predictable deflation through burning.
- **Validator (Mint Start/Stop):** Process of locking DAM tokens to generate FLUX.
- **Delegated Minting:** Allows a different Ethereum address to mint FLUX tokens on behalf of a validator.
- **Remote Minting/Burning:** Features for minting FLUX from a phone to any Ethereum address, and burning FLUX to any address with an active mint.
- **Partial Minting:** Ability to specify a percentage of minting (0-100%) and mint smaller amounts to other addresses.
- **Mint Age Multiplier:** Increases over time (up to 3x after 28 days) for continuous validator operation.
- **Burn Multiplier:** Variable multiplier (up to 10x) based on FLUX burned relative to global averages.

#### 16. Uniswap Integration

- **Adding Liquidity:** Guides on how to add liquidity to Uniswap pools, specifically the ETH-DAM pair, to earn fees from trading activity.
- **Buying Datamine Tokens:** Instructions on how to purchase DAM tokens on Uniswap by swapping with ETH or other ERC-20 tokens.

#### 17. Smart Contracts - In-Depth Breakdown

- **Technology Stack:** Smart contracts are written in Solidity (v0.6.9) and are ERC-777 compatible, built upon OpenZeppelin secure libraries.
- **Security Features:**
  - **SafeMath:** Used for all arithmetic operations to prevent integer overflow and underflow.
  - **Mutex & Checks-Effects-Interactions Pattern:** Over-used for re-entrancy attack protection and ensuring state changes occur after checks and before external interactions.
  - **Modifiers:** Custom modifiers like `preventSameBlock()` and `requireLocked()` enhance security and prevent user errors.
  - **Immutable State Variables:** Key variables are set at contract creation and cannot be changed, improving security.
  - **ERC-1820 ERC777TokensRecipient Implementation:** Unique implementation to control which tokens can be sent to the FLUX smart contract.
- **Core Functions:** Detailed explanation of `lock()`, `unlock()`, `burnToAddress()`, and `mintToAddress()` functions, including their security considerations and how they modify the contract's state.
- **View-Only Functions:** Explanation of `getMintAmount()`, `getAddressTimeMultiplier()`, `getAddressBurnMultiplier()`, `getAddressRatio()`, and `getGlobalRatio()` functions, which provide real-time analytics without modifying state.
- **Data Aggregation:** Helper functions like `getAddressDetails()` and `getAddressTokenDetails()` are provided to reduce network calls for dashboard data.
- **Additional Security Considerations (ConsenSys):** Adherence to best practices like preparing for failure, careful rollout, keeping contracts simple, staying up to date, awareness of blockchain properties, and secure development recommendations (external calls, public data, integer handling, assert/require/revert usage, modifiers, rounding, fallback functions, visibility, pragma locking, events, `tx.origin`, timestamp dependence, EIP20 approve/transferFrom attack).

#### 18. Help Article Conventions

- **Location:** Help articles are Markdown files located in `public/helpArticles/`.
- **Rendering:** These Markdown files are rendered by the `HelpDialog` component (`src/react/elements/Dialogs/HelpDialog.tsx`) and displayed on the `HelpPage` (`src/react/pages/help/HelpPage.tsx`).
- **Styling:**
  - Code blocks (`<code>`, `<pre>`) within help articles are styled in `src/react/elements/Dialogs/HelpDialog.tsx` to use `theme.palette.primary.main` (white) for background and `theme.palette.secondary.main` (teal) for text, improving readability.
  - Help article category tags (Material-UI `Chip` components) on the `HelpPage` are styled in `src/styles.ts`.
    - `primary` colored chips (selected) use a teal background (`#00FFFF`) with white text (`#fff`).
    - `default` colored chips (unselected) use the `classes.palette.background` (dark) with white text (`#fff`).
- **Titles:** All help article titles should include a unique emoji for better visual organization and engagement.

# 🤝 Vibe Code Contribute

We welcome contributions from the community, and we're excited to introduce a new way to contribute that aligns with the "vibe coding" philosophy, powered by Google Antigravity. This approach aims to streamline the development process, reduce friction, and allow you to focus on the creative aspects of building.
 
 ### What is "Vibe Coding"?
 
 "Vibe coding" is about staying in a creative flow state, where the technical details and tedious tasks are handled by intelligent tools, allowing you to concentrate on the core problem-solving and innovation. With Google Antigravity, you can interact with the codebase in a more intuitive and efficient manner.
 
 ### How to Contribute with Google Antigravity
 
 1.  **Install Google Antigravity:** If you haven't already, install Google Antigravity on your system. (Instructions for installation can be found in the Google Antigravity documentation).
 2.  **Clone the Repository:**
     ```bash
     git clone https://github.com/Datamine-Crypto/dashboard.git
     cd dashboard
     ```
 3.  **Start Vibe Coding:** Launch Google Antigravity within your cloned repository. The tool will act as your intelligent co-pilot, understanding your intent and executing complex tasks with simple commands.
     - **Intent-Driven Development:** Instead of manually navigating files and performing repetitive tasks, you can express your intentions directly. For example:
       - "Add a new feature to the dashboard."
       - "Refactor this component for better performance."
       - "Fix this bug in the smart contract interaction."
       - "Update all help articles to include information about the LOCK token."
     - **Automated Code Generation & Modification:** Google Antigravity can read, search, and modify code across the entire project. It can help you:
       - Generate new components or modules based on existing patterns.
       - Apply consistent styling changes across multiple files.
       - Perform complex refactoring operations with precision.
       - Ensure adherence to project conventions and best practices.
     - **Contextual Awareness:** The tool maintains a deep understanding of the project's structure, dependencies, and coding style, ensuring that your contributions seamlessly integrate with the existing codebase.
 4.  **Create a Pull Request:** Once you've completed your changes with the help of Google Antigravity:
     - **Review Changes:** The tool can help you review your changes and ensure everything is as expected.
     - **Commit with Clarity:** It can assist in crafting clear and concise commit messages.
     - **Open a Pull Request:** Push your changes to your forked repository and open a pull request to the `main` branch.
 
 ### Why Vibe Code?
 
 By leveraging Google Antigravity, we aim to:
 
 - **Reduce Friction:** Minimize the time spent on mundane tasks, allowing you to focus on innovation.
 - **Accelerate Development:** Speed up the development cycle, bringing new features and improvements to the Datamine Network faster.
 - **Improve Code Quality:** Ensure consistency and adherence to coding standards with automated assistance.
 - **Foster Creativity:** Empower developers to stay in their creative flow, making contributions more enjoyable and impactful.

## src Folder Structure

The `src` directory is organized into the following main subdirectories:

- **`src/app/`**: Contains application configuration and state management logic.
  - **`src/app/configs/`**: Contains application configuration files.
    - `config.base.ts`, `config.common.ts`, `config.network.ts`, `config.overrides.ts`, `config.ts`: Various configuration files.
    - **`src/app/configs/ecosystems/`**: Holds configurations for different blockchain ecosystems.
      - `config.ecosystem.arbiflux_lockquidity_l2.ts`
      - `config.ecosystem.dam_flux_l1.ts`
      - `config.ecosystem.flux_arbiflux_l2.ts`
  - **`src/app/state/`**: Contains state management logic.
    - `commonLanguage.ts`, `handleCommand.ts`, `handleQuery.ts`, `initialState.ts`

- **`src/`**: Contains the core application logic.
  - **`src/react/`**: Houses all React components, pages, and UI-related elements.
    - `ErrorBoundary.tsx`: Error boundary component.
    - `appStore.ts`: Zustand store definition.
    - `styles.ts`: MUI theme definitions.
    - **`src/react/elements/`**: Reusable UI components.
      - `LightTooltip.tsx`, `Web3Account.tsx`
      - **`src/react/elements/Cards/`**: Card components for displaying data.
      - **`src/react/elements/Dialogs/`**: Dialog components.
      - **`src/react/elements/Fragments/`**: Smaller, reusable UI fragments.
      - **`src/react/elements/Onboarding/`**: Components related to the onboarding process.
    - **`src/react/pages/`**: Top-level page components.
      - `CommunityPage.tsx`, `DashboardPage.tsx`, `HelpPage.tsx`, `OnboardingPage.tsx`, `PageFragment.tsx`, `RealtimeRewardsGameFiPage.tsx`, `Terms.tsx`, `TokenPage.tsx`
  - **`src/utils/`**: Collection of helper functions.
    - `copyToClipboard.ts`, `devLog.ts`, `formatMoney.ts`, `getApy.ts`, `web3multicall.ts`, `mathHelpers.ts`
    - **`src/utils/swap/`**: Functions related to token swapping.
      - `performSwap.ts`, `performSwapUniswapV2.ts`, `sampleQuoteSingleSwap.ts`, `swapOptions.ts`
  - **`src/web3/`**: Manages all blockchain interactions.
    - **`src/web3/abis/`**: ABI (Application Binary Interface) JSON files for smart contracts.
      - `dam.ts`, `flux.ts`, `multicall.ts`, `uniswapPair.ts`, `uniswapPairV3.ts`, `uniswapv2router.ts`
      - **`src/web3/abis/games/`**:
        - `datamineGems.ts`, `gameHodlClicker.ts`

- **`src/svgs/`**: Contains SVG assets.

- **Root `src` files**:
  - `App.tsx`: Main application component.
  - `index.tsx`: Entry point for React application.
  - `react-app-env.d.ts`, `service-worker.ts`, `serviceWorkerRegistration.ts`, `vite-env.d.ts`: Environment and service worker related files.

# 🗺️ Context Map: Datamine Network Knowledge Base

This map outlines the key concepts, components, and principles of the Datamine Network, serving as a shared knowledge base.

- **🌐 Datamine Network Overview**
  - **🎯 Purpose:** Web-based dashboard for interacting with Datamine smart contracts, viewing analytics, and managing assets across different blockchain layers.
  - **✨ Core Values & Principles:**
    - 💧 Transaction-incentivized Liquidity Pools
    - 🔥 On-Chain Linear Deflation (FLUX)
    - 📊 Realtime Multi-Smart Contract Analytics
    - 🔒 Secure By Design & Professionally Audited
    - 🤝 Built For The Community
    - 🌍 Global Problem Solved (Inflation)

- **🪙 Tokens**
  - **💰 DAM (Datamine Token)**
    - 🔢 Fixed Supply: 16,876,778 tokens
    - 📜 Standard: ERC-777
    - 💡 Primary Use: Staking to power validators
  - **⚡ FLUX (Flux Token)**
    - 📈 Supply: Non-fixed, linear and predictable deflation through burning
    - 💡 Primary Use: Base currency of the ecosystem, earned through minting
  - **🔵 ArbiFLUX**
    - 🔗 Layer: L2 (Arbitrum)
    - 💡 Primary Use: Combats gas costs of minting FLUX on L1
  - **🔐 LOCK (Lockquidity Token)**
    - 🎯 Purpose: Enhances stability, contributes to permanent liquidity pool
    - ⛏️ Minting: Minted by locking ArbiFLUX
    - 🔥 Burning Mechanism: Redirects value to a permanent liquidity pool (not supply reduction)
    - 📊 Market Efficiency: 100% - percentage of LOCK inside the market

- **⚙️ Core Mechanisms & Features**
  - **✅ Validator (Mint Start/Stop)**
    - 🔄 Process: Locking DAM tokens to generate FLUX
  - **🤝 Delegated Minting:** Allows another address to mint FLUX on behalf of a validator
  - **📱 Remote Minting/Burning:** Mint/burn FLUX from phone to any Ethereum address
  - **✂️ Partial Minting:** Specify percentage (0-100%) to mint smaller amounts
  - **⏳ Mint Age Multiplier:** Increases over time (up to 3x after 28 days) for continuous validator operation
  - **🔥 Burn Multiplier:** Variable (up to 10x) based on FLUX burned relative to global averages
  - **🛒 Datamine Market (Decentralized "Time-in-market" solution)**
    - 💡 Concept: Decentralizes "Time-in-market", creates decentralized demand
    - 🔄 Mechanism: Validators offer rewards for burning LOCK to their account (single ETH transaction)
  - **💎 Datamine Gems #GameFi**
    - 🎮 Concept: Real-time game to collect "gems" (public market addresses with unminted balances)
    - 🔄 Mechanism: Burning tokens, V2 Public Market smart contract, atomic batch burning, optimized reward distribution, "Collect all gems" button
    - 🎯 Goal: Increase monetary velocity and transactional throughput

- **🔗 Ecosystems & Layers**
  - **⛓️ Multi-chain Support:** Ethereum Mainnet (L1), Arbitrum (L2)
  - **🌳 Ecosystems Defined:**
    - DAM->FLUX (L1)
    - FLUX (L2)->ArbiFLUX (L2)
    - ArbiFLUX (L2)->LOCK (L2)
  - **⚙️ Configuration:** Managed in `src/app/configs/ecosystems/`

- **📜 Smart Contracts & ABIs**
  - **📍 Location:** `src/web3/abis/`
  - **🔑 Key Contracts:**
    - DAM Token (`dam.ts`)
    - FLUX Token (`flux.ts`)
    - Market Contract (`datamineGems.ts`)
    - Uniswap V2 Router (`uniswapv2router.ts`)
    - Uniswap Pair (`uniswapPair.ts`)
    - Uniswap Pair V3 (`uniswapPairV3.ts`)
    - Multicall (`multicall.ts`)
  - **🛡️ Security Features:** SafeMath, Mutex, Checks-Effects-Interactions, Modifiers (`preventSameBlock()`, `requireLocked()`), Immutable State Variables, ERC-1820 ERC777TokensRecipient
  - **🛠️ Core Functions:** `lock()`, `unlock()`, `burnToAddress()`, `mintToAddress()`
  - **🔍 View-Only Functions (Analytics):** `getMintAmount()`, `getAddressTimeMultiplier()`, `getAddressBurnMultiplier()`, `getAddressRatio()`, `getGlobalRatio()`
  - **📦 Data Aggregation:** `getAddressDetails()`, `getAddressTokenDetails()`

- **🏗️ Application Architecture (High-Level)**
  - **🖥️ UI:** React components (`src/react/`)
  - **🔗 Blockchain Interaction:** Viem, Web3Context, web3Reducer (`src/web3/`)
  - **🧠 State Management:** `appStore.ts` (Zustand), `handleCommand.ts` and `handleQuery.ts` (Commands & Queries pattern), `sideEffectReducer.ts`
  - **🔧 Utilities:** Helper functions (`src/utils/`)
  - **⚙️ Configuration:** `src/app/configs/`
  - **🎨 Styling:** `tss-react`, `useStyles`, Material-UI themes (`src/react/styles.ts`)
  - **🚨 Error Handling:** `web3Reducer.ts` (error state), Material-UI Snackbars/Dialogs

- **🔌 Third-Party Integrations**
  - **📚 Help Articles:** Markdown files in `public/helpArticles/`, fetched via `fetch` API
  - **🔍 Search:** `fuse.js` for help articles

- **🛠️ Development & Deployment**
  - **🏗️ Build Tool:** Vite (`vite.config.mts`)
  - **📦 Package Manager:** Yarn
  - **📜 Scripts:** `yarn start`, `yarn build`, `yarn deploy`, `yarn format`, `yarn lint`
  - **💻 Local Development:** HTTPS via `vite-plugin-mkcert`
  - **🧹 Code Quality:** ESLint, Prettier, Husky, Lint-staged
  - **🚀 Deployment:** GitHub Pages (`.github/workflows/deploy.yml`, `package.json` homepage field)

# 💡 Codebase Insights

### Multicall Implementation

- The application utilizes a multicall contract to batch multiple read-only blockchain calls into a single request. This is implemented in `src/utils/web3multicall.ts`.
- The `encodeMulticall` function prepares the data for the multicall contract, and `decodeMulticall` parses the aggregated response.
- This approach significantly improves performance by reducing the number of RPC calls to the blockchain.

### Uniswap V2 Integration

- The application integrates with Uniswap V2 for token swapping, with the core logic located in `src/utils/swap/`.
- `performSwapUniswapV2.ts` handles the entire swap process, including:
  - Fetching quotes using `getAmountsOut`.
  - Calculating slippage tolerance.
  - Checking and approving token allowances.
  - Executing the swap transaction.
- The `availableSwapTokens` array in `performSwap.ts` defines the tokens that can be swapped and their corresponding Uniswap V2 router addresses.

### Gas Fee Estimation

- The `getGasFees` function in `src/web3/utils/web3Helpers.ts` estimates gas fees for transactions.
- It supports both legacy gas price and EIP-1559 `maxFeePerGas` and `maxPriorityFeePerGas` based on client settings.
- This ensures that transactions are processed in a timely manner without overpaying for gas.

### Development Logging

- The `devLog` utility in `src/utils/devLog.ts` provides a conditional logging mechanism.
- Logs are only displayed when the `devLog=1` query parameter is present in the URL, which is useful for debugging without affecting the production build.

### Question-Driven Context Expansion

- **Q: What is the purpose of the `sideEffectReducer.ts` file and how does it relate to the existing state management pattern?**
- **A:** `sideEffectReducer.ts` is a pattern where a reducer can append queries to its state. These queries are then picked up and handled by a separate module (`handleQuery.ts`), decoupling asynchronous logic from the synchronous state updates. This ensures that state updates are always instantaneous and predictable, as they are not waiting for asynchronous operations to complete. The `sideEffectReducer` listens for changes in the `pendingQueries` array in the state and executes the corresponding asynchronous logic.
- **Q: Are there any plans to integrate with other DEXs besides Uniswap V2, and if so, which ones are being considered?**
- **A:** We're already using a number of DEXes:
  DAM (L1) is using Uniswap V3 (1% pool)
  FLUX (L1) is using Uniswap V3 (1% pool)
  FLUX (L2) is using Sushiswap (which is a fork of Uniswap V2) (0.3% pool)
  ArbiFLUX (L2) is using Sushiswap (which is a fork of Uniswap V2) (0.3% pool)
  LOCK (L2) is using Uniswap V2 (0.3% pool). The Lockquidity pool is hardcoded into the smart contract and can never be changed again
- **Q: What is the role of `Web3Context.tsx` in the application's architecture?**
- **A:** `Web3Context.tsx` has been removed. The application now uses `zustand` for state management, and blockchain interactions are handled via `viem` directly in the components or through the `web3ProviderUtils.ts` and `sideEffectReducer.ts` pattern.
- **Q: Could you elaborate on the "forecasting calculator" feature? What is its purpose, and how does it work from a user's perspective?**
- The forecasting tool allows a user to toggle a "forecasting mode" that shows a forecasted amount that they would mint over a certain time. Here they can drag sliders for burn & time multipliers, enter prediced price and also pick start/end times for their unminted time. This way they can calculate potential amount that can be minted in the future.
  The forecasting tool is available for all ecosystems.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Datamine-Crypto)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Datamine-Crypto)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
