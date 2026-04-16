## webstk

> In general try to define things before they are used.


In general try to define things before they are used.
For example if there's two helper functions `a` and `b`, and `b` uses `a`, then
`a` should come before `b`.

If the file is relatively simple, then you can forego the #region comments.

Types only used in one place (e.g., component props) should be defined right
before they're used. Shared types should be defined at the top of the file.

# TSX File Structure (for page files)

TSX generally have the following structure:

```tsx
{library imports}

{project imports}


// #region Types (if any types to define)
// =============================================================================

type MyType = {...};

// #endregion

// #region Helper functions (if any)
// =============================================================================

function myHelperFunction(): OutputType {
    ...
};

// #endregion


// #region Hooks (if any)
// =============================================================================

function useMyHook(): HookOutputType {
    ...
};

// #endregion


// #region Components (if any)
// =============================================================================

type MyComponentProps = { prop1: Prop1Type };

const MyComponent: React.FC<MyComponentProps> = ({ prop1 }) => {
    ...
};

// #endregion


// #region Main component
// =============================================================================

const Page: NextPage = () => {
    ...
};

// OR

type PageProps  = {...};

const Page: NextPage = async (props: PageProps) => {
    ...
};

export default Page;

// #endregion

// #region Exports
// =============================================================================
```

# TSX File Structure (for component files)

TSX generally have the following structure:

```tsx
import type { NextPage } from 'next';
{library imports}

{project imports}


// #region Types (if any types to define)
// =============================================================================


// #endregion

// #region Helper functions (if any)
// =============================================================================

function myHelperFunction(): OutputType {
    ...
};

// #endregion

// #region SubComponents (if any)
// =============================================================================

type MySubComponentProps = { prop1: Prop1Type };

const MySubComponent: React.FC<MySubComponentProps> = ({ prop1 }) => {
    ...
};

// #endregion

// #region Main component
// =============================================================================

type MyComponentProps  = {...};

export const MyComponentName: React.FC<MyComponentProps> = (props) => {
    ...
};

// #endregion

```

# TS File Structure

TS generally have the following structure:

```ts
{library imports}

{project imports}


// #region Helper functions (if any)
// =============================================================================

function myHelperFunction() {
    ...
};

// #endregion

// #region Main function
// =============================================================================

export myFunction(params: {input types}): {output types} {
    ...
};

// #endregion

```

# React Rules

- Use arrow functions annotated with `React.FC<PropsType>` for react components
- Use regular functions only for top-level functions that are not react components
- Use arrow functions for functions defined within other functions (e.g., callbacks)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gavin-Hofer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
