## bookmarks-react-hooks

> This document provides a technical overview of the `bookmarked` React application, detailing its architecture, state management, and key components.

# Gemini Codebase Overview

This document provides a technical overview of the `bookmarked` React application, detailing its architecture, state management, and key components.

## 1. Project Overview

`bookmarked` is a single-page bookmark manager application. Its core purpose is to allow users to create, edit, view, and filter their bookmarks. The application is built entirely with React function components, emphasizing modern React practices like Hooks and the Context API.

## 2. Core Technologies

- **React:** The application is built using React, with a heavy reliance on React Hooks.
  - `useState`: Manages simple local component state.
  - `useEffect`: Handles side effects, primarily for data fetching.
  - `useReducer`: Manages complex state transitions for bookmarks and filters.
  - `useContext`: Provides global state to the entire component tree, avoiding "prop drilling".
- **State Management:**
  - **React Context API (`src/context.js`):** A `BookmarksContext` is created to hold the application's global state, which includes the list of bookmarks and the currently selected bookmark.
  - **Reducers (`src/reducers/`):**
    - `bookmarksReducer.js`: Manages all CRUD (Create, Read, Update, Delete) operations for bookmarks, as well as setting the current bookmark and toggling favorites.
    - `filterReducer.js`: Manages the logic for filtering bookmarks (e.g., show all, show favorites, show by rating).
- **Routing:**
  - **React Router (`react-router-dom`):** Handles client-side routing. The main routes are `/` for the `Landing` page and `/bookmarks` for the main `BookmarksList` view.
- **Styling:**
  - **Emotion (`@emotion/styled`):** Used for CSS-in-JS styling. A global theme is provided via `ThemeProvider` in `src/index.js`, and global styles (including font imports) are injected in `src/components/App.js`.
- **Data Fetching:**
  - **Axios:** Used within the `useAPI` custom hook in `App.js` to fetch the initial list of bookmarks from a remote API.

## 3. Project Structure

```
/home/maxjeffwell/GitHub_Projects/bookmarks-react-hooks/
├───.babelrc              # Babel configuration
├───package.json          # Project dependencies and scripts
├───README.md             # User-facing documentation
├───public/               # Static assets (HTML, images, fonts)
└───src/
    ├───index.js          # Application entry point, renders App component
    ├───context.js        # Defines the BookmarksContext for global state
    ├───components/
    │   ├───App.js        # Root component, initializes state and routing
    │   ├───BookmarksList.js # Main component for displaying bookmarks
    │   ├───BookmarkForm.js  # Form for adding/editing bookmarks
    │   ├───Landing.js       # The initial landing page
    │   └───...           # Other presentational components
    └───reducers/
        ├───bookmarksReducer.js # State logic for bookmarks
        └───filterReducer.js    # State logic for filtering
```

## 4. Data Flow

1.  **Initialization:** When `App.js` mounts, the custom `useAPI` hook is called to fetch data from the `https://hooks-api.maxjeffwell.now.sh/bookmarks` endpoint.
2.  **State Hydration:** A `useEffect` hook in `App.js` is triggered when the data is fetched. It dispatches the `GET_BOOKMARKS` action with the fetched data as a payload.
3.  **Reducer Logic:** The `bookmarksReducer` receives the action and updates the state, populating the `bookmarks` array.
4.  **Context Provision:** The `App` component wraps all its children in `BookmarksContext.Provider`, making the `state` and `dispatch` function available to any component that needs them.
5.  **Component Consumption:** Components like `BookmarksList.js` use `useContext(BookmarksContext)` to access the bookmarks from the state and display them.
6.  **User Actions:** When a user interacts with the UI (e.g., adds a bookmark), the component dispatches an action (e.g., `ADD_BOOKMARK`) with a payload. The reducer handles the action, updates the state, and the UI re-renders to reflect the change.

## 5. Available Scripts

You can run the following commands from the project root directory:

-   `npm start`: Starts the development server.
-   `npm run build`: Bundles the app for production.
-   `npm test`: Runs the test suite.
-   `npm run lint`: Lints the source code using ESLint.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maxjeffwell)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/maxjeffwell)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
