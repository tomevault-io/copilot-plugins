## training-app-v2

> 1. Always make the React component mobile first and mobile friendly.

## React-components-guidelines.md

1. Always make the React component mobile first and mobile friendly.    
2. Always use Material-UI (MUI) for UI components and styling.
3. Make React components small and simple.
4. Always seperate logic and UI to different files
5. Always split large components into a small components in different files
6. Always split logic to different files (if more than a few lines of code)



## Typescript-guildelines.md

# TypeScript Guidelines

## Core Principles

- **Type Safety First**: Leverage TypeScript's static typing to catch errors at compile time rather than runtime.
- **Explicit over Implicit**: Always prefer explicit type annotations where they improve code clarity.
- **Simplicity over Complexity**: Avoid overly complex type structures when simpler ones will suffice.
- **Consistency**: Maintain consistent typing patterns throughout the codebase.

## Generics

- Use generics to create reusable, type-safe components and functions
- Keep generic type names descriptive (avoid single-letter names except for simple cases)
- Constrain generic types when possible using the `extends` keyword
- Document complex generic structures with comments


## Best Practices

### Type Inference

- Let TypeScript infer types when it's clear and unambiguous
- Explicitly type complex objects, function parameters, and return types
- Always type function declarations, especially exported ones

### Enable `strict` Mode in `tsconfig.json`
- This ensures TypeScript performs full type-checking and catches issues early.
- Includes: `strictNullChecks`, `noImplicitAny`, `alwaysStrict`, etc.

### Avoid Using `any`
- `any` disables type checking—defeats the purpose of TypeScript.
- Use `unknown` when needed, then narrow down with type checks.
- If `any` is absolutely necessary, isolate and comment it.
- **Never cast types to `any`** (e.g., `as any`). This bypasses TypeScript's type checking completely and can lead to runtime errors.
- Instead, use proper type narrowing, type guards, or more specific type assertions when absolutely necessary.

### Always Type Function Parameters and Return Values
- Explicit types prevent bugs and make functions easier to understand.
- Especially important for exported or public functions.

### Use `type` and `interface` Appropriately
- Use `interface` for objects that might be extended or implemented.
- Use `type` for unions, intersections, or when composing complex types.
- Avoid mixing both for the same shape.

### Prefer Union Types Over Enums
- Use string literal unions (`'pending' | 'approved' | 'rejected'`) instead of enums when possible.
- They're simpler, easier to infer, and fully type-safe.

### Use Generics to Create Reusable Types
- Avoid duplicating types by leveraging generics in functions, components, and utility types.
- Makes your code more flexible and type-safe.

### Avoid Type Assertions (`as`) Unless You're Certain
- Type assertions can bypass the compiler and lead to runtime errors.
- Only use `as` when you've validated the value shape manually or through safe parsing.
- **NEVER create and cast to custom types when the function you're calling already defines its parameter types.**
- **ALWAYS import and use the exact parameter and return types defined alongside the functions you're calling.**

### Use Proper Type Imports for API Functions
- **NEVER create duplicate types or cast to custom types when calling API functions.**
- **ALWAYS import and use the exact parameter types from the API's type definitions.**
- **For example:**
  ```typescript
  // DO THIS:
  import { SomeApiParams } from '@/apis/someApi/types';
  
  const params: SomeApiParams = { /* parameters */ };
  const result = await someApiFunction(params);
  
  // DON'T DO THIS:
  interface MyCustomApiParams { /* similar but not identical */ }
  const result = await someApiFunction({ /* parameters */ } as MyCustomApiParams);
  ```

### Use `readonly` for Immutable Data
- Protect values from accidental mutation by using `readonly` with arrays and objects.
- Improves predictability and safety.

### Narrow Types Before Using Them
- Use `typeof`, `instanceof`, or optional chaining to ensure values are safe before accessing them.
- Prevents runtime errors in loosely typed or third-party data.

### Keep Types Close to the Data They Describe
- If a type is only used in one file, define it there.
- Move it to a shared `types/` folder only when reused across multiple parts of the app.


### Keep Your `tsconfig` Clean and Strict
- Don't disable important compiler options unless absolutely necessary.
- Keep configurations consistent across environments and projects.

### DO NOT Create COMPLEX TYPES
- Avoid creating complex types that are not immediately obvious.
- Use comments to explain why a type is structured a certain way, especially if it's not obvious.
- Helps others (and future you) understand your decisions.
- Prefer simple, self-explanatory types.


## ai-models-api-usage.md

# AI Models API Usage Guidelines

## Core Principles

1. **NEVER use AI models directly** - Always use the adapter pattern
2. **All AI calls must be server-side only** - Never call AI APIs from client-side code
3. **All AI calls must include caching** - To reduce costs and improve performance
4. **All AI calls must include proper error handling** - Always return 200 status codes with error fields

## Adapter Pattern for AI Models

### Directory Structure

```
/src
  /ai
    /adapters
      /openai.ts       - OpenAI-specific implementation
      /anthropic.ts    - Anthropic-specific implementation
      /other-models.ts - Other model implementations
      /index.ts        - Exports all adapters
    /baseModelAdapter.ts - Base adapter class/interface
    /types.ts          - Shared types for AI models
```

### Using the AI Model Adapter

1. **Always initialize the adapter with a model ID**:
   ```typescript
   import { AIModelAdapter } from "../ai/baseModelAdapter";
   
   // Initialize with a specific model ID
   const adapter = new AIModelAdapter("model-id-here");
   ```

2. **Use the adapter's methods to process prompts**:
   ```typescript
   // For text completion
   const response = await adapter.processPromptToText(prompt);
   
   // For other response types (if implemented)
   const jsonResponse = await adapter.processPromptToJson(prompt);
   ```

3. **Always handle costs and errors properly**:
   ```typescript
   try {
     const response = await adapter.processPromptToText(prompt);
     
     // Handle the response
     console.log("Result:", response.result);
     console.log("Cost:", response.cost.totalCost);
     
     return {
       result: response.result,
       cost: response.cost
     };
   } catch (error) {
     console.error("Error with AI adapter:", error);
     return {
       result: "",
       cost: { totalCost: 0 },
       error: `AI service error: ${error instanceof Error ? error.message : String(error)}`
     };
   }
   ```

## Implementation Guidelines

1. **Model Selection**:
   - Use environment variables to configure default models
   - Allow overriding the model at runtime when needed
   - Keep a list of supported models in a central configuration

2. **Error Handling**:
   - Wrap all AI API calls in try/catch blocks
   - Always return proper error messages in the response
   - Never throw errors that would result in 500 status codes
   - Log errors for debugging but sanitize sensitive information

3. **Caching**:
   - Implement file-system caching for identical prompts
   - Use a deterministic hash of the prompt as the cache key
   - Include cache invalidation mechanisms
   - Make caching configurable (enable/disable) for development

4. **Cost Tracking**:
   - Always track and return the cost of each AI call
   - Implement budget limits to prevent excessive spending
   - Consider implementing usage quotas per user/session

5. **Security**:
   - Never expose API keys in client-side code
   - Sanitize all user input before sending to AI models
   - Implement content filtering for both input and output
   - Consider implementing rate limiting

## Example Usage in Server-Side Code

```typescript
import { AIModelAdapter } from "../ai/baseModelAdapter";
import { AIModelAdapterResponse } from "../ai/types";

async function generateAIResponse(userInput: string): Promise<{
  result: string;
  cost: { totalCost: number };
  error?: string;
}> {
  try {
    // Initialize the AI model adapter
    const adapter = new AIModelAdapter("default-model-id");

    // Create a prompt
    const prompt = `User: ${userInput}\n\nAssistant:`;

    // Process the prompt and get the response
    const response: AIModelAdapterResponse<string> = await adapter.processPromptToText(prompt);

    // Return the response
    return {
      result: response.result,
      cost: response.cost
    };
  } catch (error) {
    console.error("Error with AI adapter:", error);
    return {
      result: "",
      cost: { totalCost: 0 },
      error: `AI service error: ${error instanceof Error ? error.message : String(error)}`
    };
  }
}
```

## What NOT to Do

1. **NEVER call AI APIs directly**:
   ```typescript
   // DON'T DO THIS
   import OpenAI from 'openai';
   
   const openai = new OpenAI({
     apiKey: process.env.OPENAI_API_KEY,
   });
   
   const response = await openai.chat.completions.create({
     model: "gpt-4",
     messages: [{ role: "user", content: "Hello!" }],
   });
   ```

2. **NEVER call AI APIs from client-side**:
   ```typescript
   // DON'T DO THIS in React components or other client-side code
   const callAI = async () => {
     const response = await fetch('https://api.openai.com/v1/chat/completions', {
       method: 'POST',
       headers: {
         'Authorization': `Bearer ${apiKey}`, // NEVER expose API keys in client code
         'Content-Type': 'application/json',
       },
       body: JSON.stringify({
         model: 'gpt-4',
         messages: [{ role: 'user', content: userInput }],
       }),
     });
     
     const data = await response.json();
     // ...
   };
   ```

3. **NEVER hardcode model IDs**:
   ```typescript
   // DON'T DO THIS
   const adapter = new AIModelAdapter("gpt-4"); // Hardcoded model ID
   ```

4. **NEVER ignore costs**:
   ```typescript
   // DON'T DO THIS
   const response = await adapter.processPromptToText(prompt);
   return { result: response.result }; // Cost information is ignored
   ```

## Best Practices

1. Use environment variables for API keys and default model IDs
2. Implement proper caching to reduce costs
3. Track and monitor usage and costs
4. Implement rate limiting and budget controls
5. Handle errors gracefully and provide meaningful error messages
6. Always use the adapter pattern to abstract away specific AI provider implementations
7. Keep the adapter implementation flexible to easily switch between different AI providers

## app-guidelines-checklist.md

DO THE FOLLOWING:

# Application Guidelines Compliance Checklist

This checklist helps ensure that your application follows all established guidelines. Use this as a systematic approach to verifying compliance throughout the codebase.

## 1. API Guidelines Check

Start by reviewing all API modules registered in `src/apis/apis.ts`.

For each API module:

- [ ] Check file structure: `index.ts`, `types.ts`, `server.ts`, and `client.ts` exist
- [ ] Verify API naming pattern:
  - [ ] API names defined ONLY in `index.ts`
  - [ ] Server re-exports API names from `index.ts` 
  - [ ] Client imports API names from `index.ts` (NEVER from `server.ts`)
- [ ] Confirm types are defined in `types.ts` and never duplicated elsewhere
- [ ] Verify client functions return `CacheResult<ResponseType>`
- [ ] Check that business logic is implemented in `server.ts`
- [ ] Ensure API handlers in `apis.ts` use consistent API names

**Reference**: See `client-server-communications.md` for detailed guidelines on API structure

## 2. Routes Check

Review all routes in `client/routes` folder:

- [ ] Ensure proper route organization follows the app guidelines
- [ ] Verify each route implements appropriate loading states
- [ ] Confirm routes use proper error handling
- [ ] Verify dynamic routes follow naming conventions
- [ ] Ensure routes use layout components appropriately
- [ ] Ensure that the routes are simple, code is orgenized, an splited into mutliple React components if needed.

**Reference**: See `pages-and-routing-guidelines.md` for detailed guidelines on routing

## 3. React Components Check

Review components in `client/components`:

- [ ] Verify components follow established naming conventions
- [ ] Check that components use TypeScript interfaces for props
- [ ] Ensure components don't import server-side code
- [ ] Confirm components make proper use of React hooks
- [ ] Check for clean separation of presentation and logic
- [ ] Verify that components don't redefine API types (should import from API types.ts)
- [ ] Ensure proper error handling in components
- [ ] Check consistent styling approach

**Reference**: See `React-components-guidelines.md` for detailed guidelines on components

## 4. Server Code Check

Review code in the `server` folder:

- [ ] Ensure server code doesn't import client-side code
- [ ] Verify proper error handling in server functions
- [ ] Ensure clean separation of concerns

**Reference**: See `general-code-guidelines.md` and `ai-models-api-usage.md` for server-side practices

## 5. TypeScript and Coding Standards Check

- [ ] Verify consistent type definitions
- [ ] Check for any usage of `any` type (should be avoided)
- [ ] Ensure proper use of TypeScript features
- [ ] Check for consistent formatting
- [ ] Verify error handling approaches
- [ ] Ensure no circular dependencies
- [ ] Ensure no types duplications across the project
- [ ] Check for improper type handling in API calls:
  - [ ] Never cast parameters or return values of API calls
  - [ ] Never create custom types that duplicate API types
  - [ ] Always import and use the exact types from the API's types.ts files
  - [ ] No `as any` or similar type assertions when calling API functions
  - [ ] Use proper type imports rather than recreating similar interfaces

**Reference**: See `Typescript-guildelines.md` for TypeScript best practices

## 6. Final Verification

Run the Project checks (typscript and lint)
```bash
yarn checks
```

The application is not compliant with guidelines until `yarn checks` completes with 0 errors. All TypeScript and linting errors must be fixed before considering the guidelines check complete.
Finish the task by making sure `yarn checks` is not reporting any issues.



## client-server-communications.md

# Client-Server Communication Guidelines

## Simplified API Architecture

This project uses a simplified client-server communication pattern with a single API endpoint that handles all server-side operations:

```
/src
  /apis
    /apis.ts           - Registry of all API handlers (imports directly from server.ts files)
    /processApiCall.ts - Central processing logic with caching 
    /types.ts          - Shared API types
    /<domain>
      /types.ts        - Shared types for this domain
      /server.ts       - Server-side implementation (ALL business logic + exports name)
      /client.ts       - Client-side function to call the API
      /index.ts        - Exports name and types ONLY (not process or client functions)
  /pages
    /api
      /process.ts      - Single Next.js API route handler for all requests
```

## Creating a New API Endpoint

1. **Define ALL Domain Types in types.ts** (`/src/apis/<domain>/types.ts`):
   - Define request and response types
   - Keep types simple and focused on the specific domain
   - **IMPORTANT: These types MUST be used consistently across client.ts and server.ts**
   - **CRITICAL: ALL domain-related types MUST be defined in types.ts and imported from there**
   - **NEVER duplicate or redefine types in React components or other files**
   - Example:
     ```typescript
     export type ChatRequest = {
       modelId: string;
       text: string;
     };

     export type ChatResponse = {
       result: string;
       cost: {
         totalCost: number;
       };
       error?: string;
     };
     ```

   ### ✅ CORRECT: Import types from the domain's types.ts file
   ```typescript
   // In a React component
   import { ChatRequest, ChatResponse } from "@/apis/chat/types";
   
   const ChatComponent = () => {
     const [request, setRequest] = useState<ChatRequest>({
       modelId: "gpt-4",
       text: ""
     });
     
     // Use the imported types
     // ...
   };
   ```

   ### ❌ INCORRECT: Redefining types in React components
   ```typescript
   // In a React component - DON'T DO THIS
   
   // DON'T redefine types that should come from the API domain
   type ChatRequestType = { // WRONG: Duplicating the API type
     modelId: string;
     text: string;
   };
   
   const ChatComponent = () => {
     const [request, setRequest] = useState<ChatRequestType>({
       modelId: "gpt-4",
       text: ""
     });
     
     // Using the duplicated type
     // ...
   };
   ```

2. **Implement Server Logic** (`/src/apis/<domain>/server.ts`):
   - Create a `process` function that handles the request and returns a response
   - **IMPORTANT: ALL business logic MUST be implemented here**
   - Handle all business logic, validation, error cases, and external API calls here
   - **MUST use the shared types for both input parameters and return values**
   - **NEVER import any client-side code or client.ts functions here**
   - **IMPORTANT: MUST re-export the API name from index.ts**
   - Example:
     ```typescript
     import { ChatRequest, ChatResponse } from "./types";
     export { name } from './index'; // Re-export the API name from index.ts

     // Must use ChatRequest as input type and ChatResponse as return type
     export const process = async (request: ChatRequest): Promise<ChatResponse> => {
       try {
         // Input validation
         if (!request.modelId || !request.text) {
           return {
             result: "",
             cost: { totalCost: 0 },
             error: "Missing required fields: modelId and text are required."
           };
         }

         // Business logic here
         // External API calls
         // Data processing
         
         return {
           result: "Success",
           cost: { totalCost: 0 }
         };
       } catch (error) {
         return {
           result: "",
           cost: { totalCost: 0 },
           error: `Error: ${error instanceof Error ? error.message : String(error)}`
         };
       }
     };
     ```

3. **Create Client Function** (`/src/apis/<domain>/client.ts`):
   - Implement a function that calls the API using the apiClient.call method
   - **IMPORTANT: This is the ONLY place that should call apiClient.call with this API name**
   - **MUST use the exact same types for input parameters and return values as server.ts**
   - **NEVER import any server-side code or server.ts functions here**
   - **ALWAYS wrap the response type with CacheResult<T> to handle caching metadata**
   - **IMPORTANT: MUST import the API name from index.ts, NEVER from server.ts**
   - Example:
     ```typescript
     import { ChatRequest, ChatResponse } from "./types";
     import apiClient from "../../clientUtils/apiClient";
     import { name } from "./index"; // Always import from index.ts, never from server.ts
     import type { CacheResult } from "@/serverUtils/cache/types";

     // The return type must include CacheResult wrapper since caching is applied automatically
     export const chatWithAI = async (request: ChatRequest): Promise<CacheResult<ChatResponse>> => {
       return apiClient.call<CacheResult<ChatResponse>, ChatRequest>(
         name,
         request
       );
     };
     ```

4. **Create Index File** (`/src/apis/<domain>/index.ts`):
   - Export ONLY the API name and types (not process or client functions)
   - **IMPORTANT: Do NOT export process or client functions to prevent bundling server code with client code**
   - Example:
     ```typescript
     // Export types for both client and server
     export * from './types';
     
     // Export the API name - must be unique across all APIs
     export const name = "chat";
     ```

5. **Register the API in apis.ts** (`/src/apis/apis.ts`):
   - Import the server module directly and add it to the apiHandlers object
   - **IMPORTANT: Import directly from server.ts, NOT from index.ts**
   - **IMPORTANT: The key in the apiHandlers object MUST match the name exported from the server.ts**
   - Example:
     ```typescript
     import { ApiHandlers } from "./types";
     import * as chat from "./chat/server";
     import * as newDomain from "./newDomain/server";

     export const apiHandlers: ApiHandlers = {
       [chat.name]: { process: chat.process as (params: unknown) => Promise<unknown> },
       [newDomain.name]: { process: newDomain.process as (params: unknown) => Promise<unknown> }
     };
     ```

## Multiple API Routes Under the Same Namespace

When a domain needs to expose multiple API routes (e.g., search and details), follow these guidelines:

1. **Define All API Names in index.ts**:
   ```typescript
   // src/apis/books/index.ts
   export * from './types';
   
   // Base namespace
   export const name = "books"; 
   
   // All API endpoint names MUST be defined here
   export const searchApiName = `${name}/search`;
   export const detailsApiName = `${name}/details`;
   ```

2. **Re-export API Names from server.ts**:
   - Re-export the API names from index.ts
   - Create separate handler functions for each endpoint
   - Example:
   ```typescript
   // src/apis/books/server.ts
   import { BookSearchRequest, BookSearchResponse, BookDetailsRequest, BookDetailsResponse } from './types';
   // Import all API names from index.ts
   import { name, searchApiName, detailsApiName } from './index';
   
   // Re-export all API names - this pattern is crucial
   export { name, searchApiName, detailsApiName };
   
   // Search books endpoint
   export const searchBooks = async (request: BookSearchRequest): Promise<BookSearchResponse> => {
     // Implementation...
   };
   
   // Get book by ID endpoint
   export const getBookById = async (request: BookDetailsRequest): Promise<BookDetailsResponse> => {
     // Implementation...
   };
   ```

3. **Import API Names in client.ts FROM INDEX.TS, NOT server.ts**:
   ```typescript
   // src/apis/books/client.ts
   import { BookSearchRequest, BookSearchResponse, BookDetailsRequest, BookDetailsResponse } from './types';
   import apiClient from "../../clientUtils/apiClient";
   // IMPORTANT: Always import API names from index.ts, NEVER from server.ts
   import { searchApiName, detailsApiName } from "./index";
   import type { CacheResult } from "@/serverUtils/cache/types";
   
   // Client function to call the book search API
   export const searchBooks = async (request: BookSearchRequest): Promise<CacheResult<BookSearchResponse>> => {
     return apiClient.call<CacheResult<BookSearchResponse>, BookSearchRequest>(
       searchApiName,
       request
     );
   };
   
   // Client function to call the book details API
   export const getBookById = async (request: BookDetailsRequest): Promise<CacheResult<BookDetailsResponse>> => {
     return apiClient.call<CacheResult<BookDetailsResponse>, BookDetailsRequest>(
       detailsApiName,
       request
     );
   };
   ```

4. **Register Multiple Endpoints in apis.ts**:
   ```typescript
   // src/apis/apis.ts
   import * as books from "./books/server"; // Import from server.ts to get both API names and handlers
   
   export const apiHandlers: ApiHandlers = {
     // Other API handlers...
     [books.searchApiName]: { // API names are re-exported from server.ts
       process: (params: unknown) => books.searchBooks(params as BookSearchRequest) 
     },
     [books.detailsApiName]: { // API names are re-exported from server.ts
       process: (params: unknown) => books.getBookById(params as BookDetailsRequest) 
     },
   };
   ```

This approach provides several benefits:
- Clear organization of related API endpoints under a common namespace
- Explicit and self-documenting API names
- Type safety for each endpoint's request and response
- Separation of concerns with dedicated handler functions
- Consistent client-side access pattern

**CRITICAL: The client code must NEVER import directly from server.ts**
- API names MUST be defined in index.ts
- Server.ts MUST re-export API names from index.ts 
- Client.ts MUST import API names from index.ts
- This pattern ensures client code never imports server code directly
- Importing server code directly in client code will BREAK the application

## Using the API from Client Components

```typescript
import { chatWithAI } from "../api/chat/client";

// In your component:
const handleSubmit = async () => {
  const response = await chatWithAI({ 
    modelId: "gpt-4",
    text: "Hello, AI!"
  });
  
  if (response.data.error) {
    // Handle error
  } else {
    // Handle success
    // Access cache information if needed: response.fromCache, response.timestamp
  }
};
```

## Important Guidelines

1. **Single API Endpoint**:
   - **NEVER add new Next.js API routes to the /src/pages/api folder**
   - All API requests go through the single /api/process endpoint
   - The central processApiCall.ts handles routing to the correct API handler

2. **API Registration and Naming Flow**:
   - **ALWAYS register new APIs in apis.ts by importing directly from server.ts**
   - The API name flow MUST follow this pattern:
     1. DEFINE API names in index.ts
     2. IMPORT and RE-EXPORT API names in server.ts from index.ts
     3. IMPORT API names in apis.ts from server.ts
     4. IMPORT API names in client.ts from index.ts (NEVER from server.ts)
   - This pattern ensures client code never imports server code directly

3. **Client Access**:
   - **NEVER call apiClient directly from components or pages**
   - **ALWAYS use the domain-specific client functions** (e.g., chatWithAI)
   - **ALWAYS import client functions directly from client.ts, not from index.ts**
   - This ensures proper typing and consistent error handling

4. **API Type Safety**:
   - **NEVER create custom types when making API calls**
   - **NEVER use type casting (`as SomeType`) with API parameters**
   - **ALWAYS import and use the exact parameter and return types defined in the API's types.ts file**
   - Example:
     ```typescript
     // CORRECT: Import and use the exact type
     import { GetExerciseDefinitionByIdRequestParams } from '@/apis/exerciseDefinitions/types';
     
     const params: GetExerciseDefinitionByIdRequestParams = {
       definitionId: id
     };
     const result = await getExerciseDefinitionById(params);
     
     // INCORRECT: Creating custom types and casting
     interface CustomParams { /* similar but not identical */ }
     const result = await getExerciseDefinitionById({ definitionId: id } as CustomParams);
     
     // INCORRECT: Type casting with 'as any' or similar
     const result = await getExerciseDefinitionById({ /* params */ } as any);
     ```
   - This ensures complete type safety across the API boundary

5. **Caching**:
   - Caching is automatically applied at the processApiCall.ts level
   - **ALWAYS wrap response types with CacheResult<T> in client functions**
   - The CacheResult type includes:
     ```typescript
     type CacheResult<T> = {
       data: T;           // The actual API response
       fromCache: boolean; // Whether the result came from cache
       timestamp: number;  // When the result was generated/cached
     };
     ```

6. **Error Handling**:
   - Never return non-200 status codes from API routes
   - Always return status code 200 with proper error fields in the response
   - Handle all errors gracefully in the process function

7. **Type Safety**:
   - **CRITICAL: Ensure perfect type consistency across the entire API flow**
   - The client.ts function MUST use the exact same parameter types as server.ts
   - The return type in client.ts should be CacheResult<ResponseType>
   - Never use `any` as a type
   - **ALWAYS define domain-related types in types.ts and import them where needed**
   - **NEVER duplicate types in components or other files**
   - **NEVER create similar but slightly different versions of the same type**

8. **Separation of Concerns**:
   - **NEVER import server.ts in client-side code**
   - **NEVER import client.ts in server-side code**
   - **NEVER export process function from index.ts**
   - **NEVER export client functions from index.ts**
   - This prevents bundling server-side code with client-side code
   - Keep business logic in server.ts and API calls in client.ts

## general-code-guidelines.md

# Module System

* All code must use ES modules (import/export) for both client and server.
* Never use export default, always use named exports.

# Function Design

* Avoid general options parameters. 
* Always pass explicit parameters to functions for clarity and maintainability.
* Each function should perform one logical task, and the function name should clearly reflect its purpose.

# Code Organization

* Separate server-side code and client-side code in different folders and keep a clear separation between them.
* When needing to share logic between client and server, always move the shared logic to a shared folder without external dependencies.
* Never import server-side code to client-side code and vice versa.
* Keep the files relatively small when possible. 
* When files are getting too big - split them into smaller files with logical separation of concerns.
* Avoid using switch case, instead use object mapping. 
* Avoid large functions - split them into smaller functions with a single responsibility.

# Logical Separation of Concerns

* Keep UI, state management, and business logic separate.
* Business logic should be in pure functions and should not directly manipulate UI state.
* State management should not contain API calls—instead, separate API interactions into services or repositories.
* Avoid deeply nested dependencies between modules to keep code maintainable and testable.

# Prefer simplicity over optimizations 

* Avoid unnecessary abstractions or complex patterns when a simple solution would work just as well.
* If I did not ask to imrpove performance, do not implement any optimizations and keep the code simple.

# Unused variables
* Never leave unused variables in the code. Always remove them. double check that you did not miss any.

# ES LINT
* Never disable eslint without asking for permission

## nextjs-guidelines.md

# Next.js Guidelines

## Dynamic APIs

* In Next 15, these APIs: params and searchParams - have been made asynchronous. `params` and `searchParams` should be awaited before using its properties.



## pages-and-routing-guidelines.md

# SPA Routing Guidelines

This document outlines the process for adding new routes to our Single Page Application (SPA) routing system.

## Overview

Our application uses a custom SPA routing system built on top of Next.js. The routing system is implemented in the `src/client/router` directory and consists of:

1. A `RouterProvider` component that manages navigation and renders the current route
2. Route components organized in folders within the `src/client/routes` directory
3. Route configuration in the `src/client/routes/index.ts` file
4. Navigation components in the `src/client/components/layout` directory

## Adding a New Route

Follow these steps to add a new route to the application:

### 1. Create a Route Component Folder

Create a new folder in the `src/client/routes` directory with the name of your route component:

```
src/client/routes/
├── NewRoute/
│   ├── NewRoute.tsx
│   └── index.ts
```

#### Create the route component file:

```tsx
// src/client/routes/NewRoute/NewRoute.tsx
import { Box, Typography } from '@mui/material';
import { useRouter } from '../../router';

export const NewRoute = () => {
  const { routeParams, queryParams } = useRouter();
  
  return (
    <Box>
      <Typography variant="h4">New Route</Typography>
      <Typography paragraph>This is a new route in our application.</Typography>
    </Box>
  );
};
```

#### Create the index.ts file to export the component:

```tsx
// src/client/routes/NewRoute/index.ts
export { NewRoute } from './NewRoute';
```

### Component Organization Guidelines

Follow these best practices for route components:

- **Keep route components focused and small**: The main route component should be primarily responsible for layout and composition, not complex logic.
  
- **Split large components**: If a route component is getting too large (over 200-300 lines), split it into multiple smaller components within the same route folder.
  
- **Route-specific components**: Components that are only used by a specific route should be placed in that route's folder.
  
- **Shared components**: If a component is used by multiple routes, move it to `src/client/components` directory.
  
- **Component hierarchy**:
  ```
  src/client/routes/NewRoute/           # Route-specific folder
  ├── NewRoute.tsx                      # Main route component (exported)
  ├── NewRouteHeader.tsx                # Route-specific component
  ├── NewRouteContent.tsx               # Route-specific component
  ├── NewRouteFooter.tsx                # Route-specific component
  └── index.ts                          # Exports the main component
  
  src/client/components/                # Shared components
  ├── SharedComponent.tsx               # Used by multiple routes
  └── ...
  ```

- Extract business logic into separate hooks or utility functions
- Follow the naming convention of PascalCase for component files and folders
- Use named exports (avoid default exports as per our guidelines)
- Keep related components and utilities in the same folder

### 2. Register the Route in the Routes Configuration

Add your new route to the routes configuration in `src/client/routes/index.ts`:

```tsx
// Import your new route component
import { NewRoute } from './NewRoute';

// Add it to the routes configuration
export const routes = createRoutes({
  '/': Home,
  '/ai-chat': AIChat,
  '/settings': Settings,
  '/file-manager': FileManager,
  '/new-route': NewRoute, // Add your new route here
  '/not-found': NotFound,
});
```

Route path naming conventions:
- Use kebab-case for route paths (e.g., `/new-route`, not `/newRoute`)
- Keep paths descriptive but concise
- Avoid deep nesting when possible

### 3. Add Navigation Item

Update the navigation items in `src/client/components/NavLinks.tsx` to include your new route:

```tsx
import NewRouteIcon from '@mui/icons-material/Extension'; // Choose an appropriate icon

export const navItems: NavItem[] = [
  { path: '/', label: 'Home', icon: <HomeIcon /> },
  { path: '/ai-chat', label: 'AI Chat', icon: <ChatIcon /> },
  { path: '/file-manager', label: 'Files', icon: <FolderIcon /> },
  { path: '/new-route', label: 'New Route', icon: <NewRouteIcon /> }, // Add your new route here
  { path: '/settings', label: 'Settings', icon: <SettingsIcon /> },
];
```

Navigation item guidelines:
- Choose a descriptive but concise label
- Select an appropriate Material UI icon that represents the route's purpose
- Consider the order of items in the navigation (most important/frequently used routes should be more accessible)

## Using the Router

### Navigation

To navigate between routes in your components, use the `useRouter` hook:

```tsx
import { useRouter } from '../../router';

const MyComponent = () => {
  const { navigate } = useRouter();
  
  const handleClick = () => {
    navigate('/new-route');
  };
  
  // You can also replace the current history entry
  const handleReplace = () => {
    navigate('/new-route', { replace: true });
  };
  
  return (
    <Button onClick={handleClick}>Go to New Route</Button>
  );
};
```

### Navigation Guidelines

- **Always use the navigation API from useRouter**: Never use `window.location.href` for navigation as it causes a full page reload and breaks the SPA behavior.

```tsx
// ❌ Don't do this
window.location.href = '/some-route';

// ✅ Do this instead
const { navigate } = useRouter();
navigate('/some-route');
```

- This ensures consistent navigation behavior throughout the application
- Preserves the SPA (Single Page Application) experience
- Maintains application state during navigation
- Enables proper history management

### Navigating with Parameters

When navigating to routes that require parameters (like IDs), construct the path with the parameters included:

```tsx
// Navigating to a route with a parameter
const { navigate } = useRouter();

// Navigate to a video page with a specific video ID
const handleVideoClick = (videoId) => {
  navigate(`/video/${videoId}`);
};

// Navigate to a channel page with a specific channel ID
const handleChannelClick = (channelId) => {
  navigate(`/channel/${channelId}`);
};
```

For routes with multiple parameters, include all parameters in the path:

```tsx
// Route with multiple parameters
navigate(`/category/${categoryId}/product/${productId}`);
```

You can also include query parameters:

```tsx
// Navigate with query parameters
navigate(`/search?q=${encodeURIComponent(searchQuery)}&filter=${filter}`);

// Note: When using query parameters, always use `encodeURIComponent()` for any user-provided values to ensure proper URL encoding.
```

### Getting Current Route

You can access the current route path using the `useRouter` hook:

```tsx
import { useRouter } from '../../router';

const MyComponent = () => {
  const { currentPath } = useRouter();
  
  return (
    <div>
      <p>Current path: {currentPath}</p>
    </div>
  );
};
```

## Advanced Routing Features

### Route Parameters

Our router automatically parses route parameters from the URL path. To define a route with parameters, use the colon syntax in your route path:

```tsx
// In src/client/routes/index.ts
export const routes = createRoutes({
  // Other routes...
  '/items/:id': ItemDetail,
});
```

Then access the parameters in your component using the `useRouter` hook:

```tsx
// src/client/routes/ItemDetail/ItemDetail.tsx
import { useRouter } from '../../router';

export const ItemDetail = () => {
  const { routeParams } = useRouter();
  const itemId = routeParams.id;
  
  return (
    <div>
      <h1>Item Detail</h1>
      {itemId ? <p>Item ID: {itemId}</p> : <p>Invalid item ID</p>}
    </div>
  );
};
```

### Query Parameters

The router also automatically parses query parameters from the URL. Access them in your component using the `useRouter` hook:

```tsx
// src/client/routes/SearchResults/SearchResults.tsx
import { useRouter } from '../../router';

export const SearchResults = () => {
  const { queryParams } = useRouter();
  const searchQuery = queryParams.q || '';
  
  return (
    <div>
      <h1>Search Results</h1>
      <p>Query: {searchQuery}</p>
    </div>
  );
};


#### WHAT NOT TO DO:
1. NEVER add new pages to /pages folder
2. NEVER add new apis to /pages/api folder.



## tasks-guidelines.md

1. Always run `yarn checks` after completion of tasks.
2. Keep running `yarn checks` until no errors are reported.
3. DONT STOP Running `yarn checks` until no errors are reported.
2. Always verify that the code you added comply with all the rules and guidelines in the app-guildelines.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gileck) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
