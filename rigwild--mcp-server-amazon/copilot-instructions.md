## mcp-server-amazon

> This document provides guidance for Claude and other LLMs on how to approach development tasks related to the Amazon MCP Server project.

# Development Guidelines for Amazon MCP Server

This document provides guidance for Claude and other LLMs on how to approach development tasks related to the Amazon MCP Server project.

## General Development Approach

1. **Always understand the project structure first**

   - Review the key files: amazon.ts (core functionality), index.ts (MCP server setup), config.ts (configuration), utils.ts (helper functions)
   - Understand the MCP architecture and how tools are defined and exposed
   - Note the existing patterns for browser automation and data extraction

2. **For any feature development, follow these steps**:
   - Analyze requirements thoroughly
   - Create a plan with clear steps before writing any code
   - Break complex tasks into smaller functions with single responsibilities
   - Maintain the existing error handling patterns
   - Document your code with comments that explain "why", not just "what"

## Feature Implementation Guidelines

### Adding a New Amazon Feature

1. **Research Phase**:

   - Analyze the Amazon webpage structure for the feature
   - Identify the necessary selectors for data extraction
   - Check for edge cases (empty results, error states)

2. **Planning Phase**:

   - Define clear inputs and outputs for the function
   - Plan the browser automation steps needed
   - Identify potential error scenarios
   - Create a mock response structure

3. **Implementation Phase**:

   - Start by adding the core functionality in amazon.ts
   - Create separate extraction functions for complex data parsing
   - Follow the existing pattern of try/catch blocks with meaningful error messages
   - Support both live scraping and mock modes

4. **Test Phase**:

   - Create a dedicated test file (example.test.ts) with multiple test cases
   - Test edge cases (no results, errors, etc.)
   - Test with both mock data and live scraping if possible

5. **Integration Phase**:
   - Add the new tool to index.ts following existing patterns
   - Define appropriate parameter schemas using zod
   - Ensure error handling is consistent with other tools

### Code Structure Guidelines

1. **Function Organization**:

   - Core Amazon interactions should be in amazon.ts
   - Each major feature should have its own section clearly marked with comments
   - Extraction logic should be separate from browser automation logic

2. **Error Handling**:

   - Always use try/catch blocks for browser operations
   - Include the feature name in error console logs (e.g., "[ERROR][feature-name]")
   - Return user-friendly error messages that don't expose internal details

3. **Browser Automation**:
   - Use the createBrowserAndPage utility
   - Add appropriate waits for page loading/navigation
   - Check for login state with throwIfNotLoggedIn where appropriate
   - Close browsers in finally blocks to prevent resource leaks

### Testing Guidelines

1. **Always create a dedicated test file** for each major feature (following the pattern of amazon.getProductDetails.test.ts)

2. **Include multiple test cases**:

   - Happy path (successful operation)
   - Edge cases (empty results, special product types)
   - Error cases (invalid inputs, network issues)

3. **Test structure**:

   - Separate test functions for different scenarios
   - Clear console output showing what's being tested
   - Verification of critical response properties
   - Pass/fail indicators in the console output

4. **Mock Support**:
   - Ensure features work with mocked data
   - If adding a new feature, create mock files that cover various scenarios

### Code Quality and Documentation

1. **TypeScript Guidelines**:

   - Define interfaces for all complex return types
   - Use proper TypeScript typing to ensure type safety
   - Leverage zod for runtime validation of inputs

2. **Documentation Guidelines**:

   - Add JSDoc comments to all public functions
   - Document parameter types and return values
   - Note any special behavior or edge cases
   - Update README.md with new features

3. **Code Organization**:
   - Group related functions together
   - Use section comments to separate different feature areas
   - Keep functions focused on a single responsibility

### Example: Feature Development Plan Template

When asked to develop a new feature, provide a plan with this structure:

1. **Feature Analysis**:

   - Purpose and user requirements
   - Technical approach
   - Amazon page structure analysis
   - Key data points to extract

2. **Implementation Plan**:

   - Core function signature and return type
   - Browser automation steps
   - HTML parsing approach
   - Error handling strategy
   - Mock data creation plan

3. **Testing Strategy**:

   - Test cases to cover
   - Validation approach
   - Edge cases to handle

4. **Integration Plan**:
   - How to expose via MCP
   - Parameter validation
   - Response formatting

## Example Development Process

### Example: Adding a Product Search Feature

#### 1. Analysis:

- Amazon search uses URL format: https://www.amazon.es/s?k=[query]
- Search results are in div elements with data-component-type="s-search-result"
- Each product has: title, price, rating, image, ASIN
- Need to handle: no results, pagination, sponsored results

#### 2. Planning:

```typescript
// Function signature
async function searchProducts(query: string, page: number = 1): Promise<SearchResults>

// Return type
interface SearchResults {
  query: string
  totalResults: number
  currentPage: number
  totalPages: number
  products: ProductSearchItem[]
}

interface ProductSearchItem {
  title: string
  price: string | null
  imageUrl: string | null
  rating: number | null
  reviewCount: number | null
  asin: string
  url: string
  isPrime: boolean
  isSponsoredResult: boolean
}
```

#### 3. Implementation Steps:

1. Create search function in amazon.ts
2. Add browser navigation to search URL
3. Create HTML extraction function
4. Add pagination handling
5. Create mock data file
6. Handle error cases

#### 4. Test Plan:

1. Test with common search term
2. Test with no results
3. Test with special characters
4. Test pagination
5. Test invalid inputs

#### 5. MCP Integration:

1. Add tool definition to index.ts
2. Define parameters schema with zod
3. Format response data for MCP

Following this structured approach will lead to maintainable, well-tested code that fits with the existing project structure.

---
> Source: [rigwild/mcp-server-amazon](https://github.com/rigwild/mcp-server-amazon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
