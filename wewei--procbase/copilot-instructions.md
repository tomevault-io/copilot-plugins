## procbase

> This document defines the coding standards and patterns for writing functional TypeScript code in this project.

# Functional TypeScript Rules

This document defines the coding standards and patterns for writing functional TypeScript code in this project.

## Core Principles

### 1. Function-First Approach
- **Do not define classes** - Use functions and function composition instead
- **Prefer pure functions** - Functions should be predictable and have no side effects when possible
- **Use function composition** - Combine small functions to build complex behavior

### 2. Type Definitions
- **Use `type` instead of `interface`** - Always use `type` to define structure types
- **Avoid `enum`** - Use literal union types instead of enums
- **Prefer union types** - Use discriminated unions for complex state modeling

### 3. Function Design
- **Keep functions short** - Functions longer than 50 lines should be broken down
- **Single responsibility** - Each function should do one thing well
- **Descriptive names** - Function names should clearly describe their functionality

### 4. Documentation
- **TSDoc for all functions** - Every function must have TSDoc covering:
  - Description of what the function does
  - Parameters with types and descriptions
  - Return value with type and description
  - Examples for complex functions

## Type Definitions

### ✅ Good - Use `type`
```typescript
type User = {
  id: string;
  name: string;
  email: string;
  isActive: boolean;
};

type ApiResponse<T> = {
  data: T;
  status: 'success' | 'error';
  message?: string;
};

type ValidationResult = 
  | { isValid: true; value: unknown }
  | { isValid: false; errors: string[] };
```

### ❌ Bad - Don't use `interface`
```typescript
interface User {
  id: string;
  name: string;
  email: string;
  isActive: boolean;
}
```

### ✅ Good - Use literal union types
```typescript
type Status = 'pending' | 'loading' | 'success' | 'error';
type UserRole = 'admin' | 'user' | 'guest';
type FileType = 'text' | 'json' | 'markdown' | 'dot';
```

### ❌ Bad - Don't use `enum`
```typescript
enum Status {
  Pending = 'pending',
  Loading = 'loading',
  Success = 'success',
  Error = 'error'
}
```

### ✅ Good - Type Definition File (ProjectAnalysisResult.ts)
```typescript
import ts from 'typescript';
import type { ProjectSymbolTableState } from '../../createProjectSymbolTable/types/ProjectSymbolTableState';
import type { ProjectStatistics } from './ProjectStatistics';

/**
 * 项目分析结果
 * 
 * 包含项目分析完成后产生的所有信息，包括符号表、根文件列表、
 * 编译器选项、诊断信息和统计信息。
 * 
 * @example
 * ```typescript
 * const result: ProjectAnalysisResult = {
 *   symbolTable: createProjectSymbolTable(),
 *   rootFiles: ['src/main.ts', 'src/utils.ts'],
 *   compilerOptions: { target: 'ES2020', module: 'ESNext' },
 *   diagnostics: [],
 *   statistics: { totalFiles: 2, totalSymbols: 15, ... }
 * };
 * ```
 */
export type ProjectAnalysisResult = {
  /** 项目符号表，包含所有提取的符号信息 */
  symbolTable: ProjectSymbolTableState;
  /** 根文件列表，项目的主要入口文件 */
  rootFiles: string[];
  /** TypeScript 编译器选项 */
  compilerOptions: ts.CompilerOptions;
  /** 编译诊断信息，包含错误和警告 */
  diagnostics: ts.Diagnostic[];
  /** 项目统计信息 */
  statistics: ProjectStatistics;
};
```

### ✅ Good - Type Definition File (ProjectStatistics.ts)
```typescript
/**
 * 项目统计信息
 * 
 * 包含项目分析过程中收集的各种统计指标，用于了解项目的
 * 规模和复杂度。
 * 
 * @example
 * ```typescript
 * const stats: ProjectStatistics = {
 *   totalFiles: 10,
 *   totalSymbols: 150,
 *   exportedSymbols: 45,
 *   internalSymbols: 105,
 *   importCount: 67,
 *   dependencyCount: 234
 * };
 * ```
 */
export type ProjectStatistics = {
  /** 总文件数量 */
  totalFiles: number;
  /** 总符号数量（导出 + 内部） */
  totalSymbols: number;
  /** 导出符号数量 */
  exportedSymbols: number;
  /** 内部符号数量 */
  internalSymbols: number;
  /** 导入语句数量 */
  importCount: number;
  /** 依赖关系数量 */
  dependencyCount: number;
};
```

### ✅ Good - Type Definition File (AnalysisOptions.ts)
```typescript
/**
 * 分析选项
 * 
 * 控制项目分析行为的配置选项，允许用户自定义分析的范围
 * 和深度。
 * 
 * @example
 * ```typescript
 * const options: AnalysisOptions = {
 *   includeDeclarationFiles: false,
 *   includeNodeModules: false,
 *   includeSystemSymbols: false,
 *   followTypeOnlyImports: true,
 *   maxDepth: 5
 * };
 * ```
 */
export type AnalysisOptions = {
  /** 是否包含声明文件（.d.ts） */
  includeDeclarationFiles?: boolean;
  /** 是否包含 node_modules 中的依赖 */
  includeNodeModules?: boolean;
  /** 是否包含系统符号（如内置类型） */
  includeSystemSymbols?: boolean;
  /** 是否跟踪仅类型导入 */
  followTypeOnlyImports?: boolean;
  /** 最大分析深度，防止无限递归 */
  maxDepth?: number;
};
```

### ❌ Bad - Mixed Types in One File
```typescript
// Don't put multiple types in one file
export type ProjectAnalysisResult = { /* ... */ };
export type ProjectStatistics = { /* ... */ };
export type AnalysisOptions = { /* ... */ };
export type ProjectAnalysisContext = { /* ... */ };
```

### ❌ Bad - Types in Main Function Files
```typescript
// Don't define types in the same file as functions
const analyzeProject = () => { /* ... */ };

export type ProjectAnalysisResult = { /* ... */ }; // Wrong!

export default analyzeProject;
```

### ❌ Bad - Index.ts in Types Folder
```typescript
// Don't create index.ts in types folder
// types/index.ts - This should not exist
export * from './ProjectAnalysisResult';
export * from './ProjectStatistics';
```

## Function Patterns

### ✅ Good - Pure Functions
```typescript
/**
 * Validates if a string is a valid email address
 * @param email - The email string to validate
 * @returns True if the email is valid, false otherwise
 */
const isValidEmail = (email: string): boolean => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

/**
 * Formats a user object for display
 * @param user - The user object to format
 * @returns A formatted string representation of the user
 */
const formatUser = (user: User): string => {
  return `${user.name} (${user.email})`;
};
```

### ✅ Good - Function Composition
```typescript
/**
 * Processes a list of users by filtering active users and formatting them
 * @param users - Array of users to process
 * @returns Array of formatted active user strings
 */
const processActiveUsers = (users: User[]): string[] => {
  return users
    .filter(user => user.isActive)
    .map(formatUser);
};
```

### ❌ Bad - Classes
```typescript
class UserProcessor {
  private users: User[];
  
  constructor(users: User[]) {
    this.users = users;
  }
  
  processActiveUsers(): string[] {
    // implementation
  }
}
```

## Function Length Guidelines

### ✅ Good - Short, Focused Functions
```typescript
/**
 * Extracts file extension from a filename
 * @param filename - The filename to extract extension from
 * @returns The file extension without the dot, or empty string if no extension
 */
const getFileExtension = (filename: string): string => {
  const lastDotIndex = filename.lastIndexOf('.');
  return lastDotIndex > 0 ? filename.slice(lastDotIndex + 1) : '';
};

/**
 * Determines if a file is a supported type
 * @param filename - The filename to check
 * @returns True if the file type is supported
 */
const isSupportedFileType = (filename: string): boolean => {
  const extension = getFileExtension(filename);
  const supportedTypes = ['ts', 'js', 'json', 'md'];
  return supportedTypes.includes(extension);
};
```

### ⚠️ Warning - Function Too Long (Break Down)
```typescript
// This function is too long and should be broken down
const processFile = (filePath: string): ProcessResult => {
  // 60+ lines of code...
  // Should be broken into smaller functions like:
  // - validateFilePath
  // - readFileContent
  // - parseFileContent
  // - validateFileStructure
  // - createProcessResult
};
```

## Error Handling Patterns

### ✅ Good - Result Types
```typescript
type Result<T, E> = { success: T } | { error: E };

/**
 * Safely reads a file and returns a Result type
 * @param filePath - Path to the file to read
 * @returns Result containing file content or error
 */
const readFileSafely = async (filePath: string): Promise<Result<string, Error>> => {
  try {
    const content = await fs.readFile(filePath, 'utf8');
    return { success: content };
  } catch (error) {
    return { error: error instanceof Error ? error : new Error('Unknown error') };
  }
};
```

### ✅ Good - Type Guards
```typescript
/**
 * Type guard to check if a value is a success result
 * @param result - The result to check
 * @returns True if the result is a success
 */
const isSuccess = <T, E>(result: Result<T, E>): result is { success: T } => {
  return 'success' in result;
};

/**
 * Type guard to check if a value is an error result
 * @param result - The result to check
 * @returns True if the result is an error
 */
const isError = <T, E>(result: Result<T, E>): result is { error: E } => {
  return 'error' in result;
};
```

## Configuration and Options

### ✅ Good - Options Pattern
```typescript
type CompilerOptions = {
  target?: 'ES2020' | 'ESNext';
  module?: 'ESNext' | 'CommonJS';
  strict?: boolean;
  sourceMap?: boolean;
};

/**
 * Creates default compiler options with sensible defaults
 * @returns Default compiler options
 */
const createDefaultCompilerOptions = (): CompilerOptions => ({
  target: 'ES2020',
  module: 'ESNext',
  strict: true,
  sourceMap: true
});

/**
 * Merges user options with defaults
 * @param userOptions - User-provided options
 * @param defaults - Default options to merge with
 * @returns Merged options object
 */
const mergeOptions = <T extends object>(
  userOptions: Partial<T>,
  defaults: T
): T => ({
  ...defaults,
  ...userOptions
});
```

## Data Transformation Patterns

### ✅ Good - Immutable Transformations
```typescript
/**
 * Transforms a list of users into a map keyed by user ID
 * @param users - Array of users to transform
 * @returns Map of users keyed by their ID
 */
const usersToMap = (users: User[]): Map<string, User> => {
  return new Map(users.map(user => [user.id, user]));
};

/**
 * Filters and sorts users by name
 * @param users - Array of users to process
 * @param filterActive - Whether to filter only active users
 * @returns Filtered and sorted array of users
 */
const filterAndSortUsers = (
  users: User[],
  filterActive: boolean = false
): User[] => {
  return users
    .filter(user => !filterActive || user.isActive)
    .sort((a, b) => a.name.localeCompare(b.name));
};
```

## Async Function Patterns

### ✅ Good - Async Functions
```typescript
/**
 * Fetches user data from an API
 * @param userId - The ID of the user to fetch
 * @returns Promise resolving to user data or error
 */
const fetchUser = async (userId: string): Promise<Result<User, Error>> => {
  try {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    const user = await response.json();
    return { success: user };
  } catch (error) {
    return { error: error instanceof Error ? error : new Error('Unknown error') };
  }
};

/**
 * Processes multiple users concurrently
 * @param userIds - Array of user IDs to fetch
 * @returns Promise resolving to array of user results
 */
const fetchUsers = async (userIds: string[]): Promise<Result<User, Error>[]> => {
  const promises = userIds.map(fetchUser);
  return Promise.all(promises);
};
```

## Utility Function Patterns

### ✅ Good - Utility Functions
```typescript
/**
 * Debounces a function call
 * @param func - The function to debounce
 * @param delay - Delay in milliseconds
 * @returns Debounced function
 */
const debounce = <T extends (...args: unknown[]) => unknown>(
  func: T,
  delay: number
): ((...args: Parameters<T>) => void) => {
  let timeoutId: NodeJS.Timeout;
  return (...args: Parameters<T>) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func(...args), delay);
  };
};

/**
 * Memoizes a function result
 * @param func - The function to memoize
 * @returns Memoized function
 */
const memoize = <T extends (...args: unknown[]) => unknown>(
  func: T
): T => {
  const cache = new Map<string, unknown>();
  return ((...args: Parameters<T>) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = func(...args);
    cache.set(key, result);
    return result;
  }) as T;
};
```

## Testing Patterns

### ✅ Good - Test Functions
```typescript
/**
 * Creates a mock user for testing
 * @param overrides - Optional overrides for default values
 * @returns A mock user object
 */
const createMockUser = (overrides: Partial<User> = {}): User => ({
  id: 'test-id',
  name: 'Test User',
  email: 'test@example.com',
  isActive: true,
  ...overrides
});

/**
 * Asserts that a result is a success
 * @param result - The result to assert
 * @param message - Optional error message
 */
const assertSuccess = <T, E>(
  result: Result<T, E>,
  message?: string
): asserts result is { success: T } => {
  if (!isSuccess(result)) {
    throw new Error(message || 'Expected success result');
  }
};
```

## File Organization and Architecture

### 5. Single Responsibility File Organization
- **One main function per file** - Each `.ts` file should have exactly one `default export` function
- **File name matches function name** - The filename should correspond to the main function's name
- **Internal functions are private** - Other functions in the file should be internal implementation details
- **Folder organization by task** - Use folders to group related functions for a specific task
- **Main function exports from index.ts** - The main task function should be exported from `index.ts` in the folder
- **Folder name matches main function** - The folder name should correspond to the main task function name
- **Types as named exports** - All types used by the main function and its dependencies should be exported as named exports

### 6. Type Definition Organization
- **One type per file** - Each type should be defined in its own file
- **Types in types subfolder** - All types should be placed in a `types/` subfolder within the main function folder
- **File name matches type name** - The filename should exactly match the type name (e.g., `User.ts` for `type User`)
- **Complete TSDoc documentation** - Every type must have comprehensive TSDoc documentation
- **No index.ts in types folder** - Types folder should not have an index.ts file
- **Named exports only** - Types should be exported as named exports, not default exports

### ✅ Good - Single Responsibility File Structure
```
lang/
├── analyzeProject/
│   ├── index.ts                    # export default analyzeProject
│   ├── createProgram.ts            # internal function
│   ├── getSourceFiles.ts           # internal function
│   ├── analyzeSourceFile.ts        # internal function
│   ├── calculateStatistics.ts      # internal function
│   └── types/                      # types subfolder
│       ├── ProjectAnalysisResult.ts # export type ProjectAnalysisResult
│       ├── ProjectStatistics.ts    # export type ProjectStatistics
│       ├── AnalysisOptions.ts      # export type AnalysisOptions
│       └── ProjectAnalysisContext.ts # export type ProjectAnalysisContext
├── createProjectContext/
│   ├── index.ts                    # export default createProjectContext
│   ├── fromConfig.ts               # internal function
│   ├── fromFiles.ts                # internal function
│   ├── createDefaultOptions.ts     # internal function
│   └── types/                      # types subfolder
│       └── ProjectContext.ts       # export type ProjectContext
├── performTreeShaking/
│   ├── index.ts                    # export default performTreeShaking
│   ├── calculateClosure.ts         # internal function
│   ├── findUnusedSymbols.ts        # internal function
│   ├── groupSymbolsByFile.ts       # internal function
│   ├── calculateStatistics.ts      # internal function
│   └── types/                      # types subfolder
│       ├── TreeShakingResult.ts    # export type TreeShakingResult
│       ├── TreeShakingStatistics.ts # export type TreeShakingStatistics
│       └── TreeShakingOptions.ts   # export type TreeShakingOptions
└── types.ts                        # Shared types used across multiple modules
```

### ✅ Good - Main Function File (index.ts)
```typescript
import type { ProjectAnalysisResult, ProjectAnalysisContext, AnalysisOptions } from '../types';
import { createProgram } from './createProgram';
import { getSourceFiles } from './getSourceFiles';
import { analyzeSourceFile } from './analyzeSourceFile';
import { calculateStatistics } from './calculateStatistics';

/**
 * Analyzes the entire project and builds symbol table and dependency relationships
 * @param context - Project analysis context
 * @param options - Analysis options
 * @returns Project analysis result
 */
const analyzeProject = (
  context: ProjectAnalysisContext,
  options: AnalysisOptions = {}
): ProjectAnalysisResult => {
  const sourceFiles = getSourceFiles(context.program, context.rootFiles);
  
  sourceFiles.forEach(sourceFile => {
    const fileSymbols = analyzeSourceFile(sourceFile, context.typeChecker, options);
    // Process file symbols...
  });
  
  const statistics = calculateStatistics(context.symbolTable, context.rootFiles);

  return {
    symbolTable: context.symbolTable,
    rootFiles: context.rootFiles,
    compilerOptions: context.compilerOptions,
    diagnostics: [],
    statistics
  };
};

export default analyzeProject;
```

### ✅ Good - Internal Function File
```typescript
import ts from 'typescript';

/**
 * Creates a TypeScript program
 * @param rootFiles - List of root files
 * @param compilerOptions - Compiler options
 * @returns TypeScript program instance
 */
const createProgram = (
  rootFiles: string[], 
  compilerOptions: ts.CompilerOptions = {}
): ts.Program => {
  return ts.createProgram(rootFiles, compilerOptions);
};

export default createProgram;
```

### ✅ Good - Type Definition File
```typescript
import ts from 'typescript';
import type { ProjectSymbolTableState } from '../SymbolTable';

/**
 * Project analysis result
 */
export type ProjectAnalysisResult = {
  symbolTable: ProjectSymbolTableState;
  rootFiles: string[];
  compilerOptions: ts.CompilerOptions;
  diagnostics: ts.Diagnostic[];
  statistics: ProjectStatistics;
};

/**
 * Project statistics
 */
export type ProjectStatistics = {
  totalFiles: number;
  totalSymbols: number;
  exportedSymbols: number;
  internalSymbols: number;
  importCount: number;
  dependencyCount: number;
};

/**
 * Analysis options
 */
export type AnalysisOptions = {
  includeDeclarationFiles?: boolean;
  includeNodeModules?: boolean;
  includeSystemSymbols?: boolean;
  followTypeOnlyImports?: boolean;
  maxDepth?: number;
};

/**
 * Project analysis context
 */
export type ProjectAnalysisContext = {
  program: ts.Program;
  typeChecker: ts.TypeChecker;
  symbolTable: ProjectSymbolTableState;
  rootFiles: string[];
  compilerOptions: ts.CompilerOptions;
};
```

### ✅ Good - Usage Pattern
```typescript
// Import main functions
import analyzeProject from './lang/analyzeProject';
import createProjectContext from './lang/createProjectContext';
import performTreeShaking from './lang/performTreeShaking';

// Import types
import type { 
  ProjectAnalysisResult, 
  TreeShakingResult,
  ProjectAnalysisContext 
} from './lang/types';

// Usage
const context = createProjectContext(files, options);
const analysis = analyzeProject(context, analysisOptions);
const treeShakingResult = performTreeShaking(analysis, entryPoints, treeShakingOptions);
```

### ❌ Bad - Mixed Responsibilities
```typescript
// Don't mix multiple main functions in one file
const analyzeProject = () => { /* ... */ };
const performTreeShaking = () => { /* ... */ };
const createContext = () => { /* ... */ };

export { analyzeProject, performTreeShaking, createContext };
```

### ❌ Bad - Inconsistent Naming
```typescript
// Don't use different names for file and main function
// file: projectAnalyzer.ts
const analyzeProject = () => { /* ... */ };
export default analyzeProject;
```

### ❌ Bad - Exposing Internal Functions
```typescript
// Don't export internal implementation functions
const createProgram = () => { /* ... */ };
const getSourceFiles = () => { /* ... */ };

export { createProgram, getSourceFiles }; // Internal functions should not be exported
export default analyzeProject;
```

## Architecture Benefits

### 1. Single Responsibility Principle
- Each file has one clear purpose
- Function name matches filename for clarity
- Internal functions focus on supporting the main function

### 2. Clear Dependency Relationships
- Main functions serve as clear entry points
- Dependencies are organized in folders
- Type definitions are centralized and reusable

### 3. Excellent Testability
- Each main function can be tested independently
- Internal functions are tested through the main function
- Type definitions can be validated separately

### 4. Easy Maintenance
- Changes to internal implementation don't affect external interfaces
- Adding new functionality just requires new folders and main functions
- Clear separation of concerns makes debugging easier

### 5. Scalable Architecture
- New features can be added as new folders
- Existing functionality can be extended without breaking changes
- Clear boundaries prevent circular dependencies

### 6. Type Organization Benefits
- **Single type per file** makes types easy to find and modify
- **Types subfolder** keeps type definitions separate from logic
- **Comprehensive TSDoc** provides clear documentation for each type
- **Named exports** enable precise imports and better tree shaking
- **No index.ts in types** prevents circular dependencies and keeps imports explicit

## Summary

Follow these principles to write maintainable, functional TypeScript code:

1. **Functions over classes** - Use function composition and pure functions
2. **Types over interfaces** - Always use `type` for structure definitions
3. **Literal unions over enums** - Use union types for constants
4. **Keep functions small** - Break down functions longer than 50 lines
5. **Document everything** - Use TSDoc for all functions
6. **Be immutable** - Avoid mutating data, create new objects instead
7. **Handle errors gracefully** - Use Result types and proper error handling
8. **Write tests** - Create test utilities and mock functions
9. **Single responsibility files** - One main function per file with matching names
10. **Organized folder structure** - Group related functions in folders named after main tasks
11. **Clear type exports** - Export all types as named exports for reusability
12. **One type per file** - Each type in its own file within types subfolder
13. **Complete type documentation** - Comprehensive TSDoc for every type definition
14. **No index.ts in types** - Keep type imports explicit and avoid circular dependencies

These patterns will lead to more maintainable, testable, and predictable code with clear architectural boundaries.

## Import Style

- **Do not use `import * as ...`** — Always import specific named exports or default exports only.
- **Prefer**: `import { foo } from './bar'` or `import foo from './bar'`
- **Avoid**: `import * as bar from './bar'`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wewei)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wewei)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
