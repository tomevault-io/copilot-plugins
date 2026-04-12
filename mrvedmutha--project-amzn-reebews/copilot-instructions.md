## project-amzn-reebews

> how to write typescript interface and props

# TypeScript Interface and Props Design Rules

## File Structure

```
src/
├── enums/
│   ├── UserRoleEnum.ts           // User role enum definition
│   ├── ThemeEnum.ts              // Theme enum definition
│   ├── StatusEnum.ts             // Status enum definition
│   └── ApiErrorCodeEnum.ts       // API error code enum definition
└── types/
    ├── IUserInterface.types.ts         // User interface definition
    ├── IApiResponseInterface.types.ts  // API response interface
    ├── IConfigInterface.types.ts       // Configuration interface
    └── props/
        ├── NavbarProps.props.ts        // Navbar component props
        ├── ButtonProps.props.ts        // Button component props
        ├── ModalProps.props.ts         // Modal component props
        └── FormProps.props.ts          // Form component props
```

## Cursor Rules

### 1. Rule - Interface Location and Naming
- **Always place TypeScript interfaces in `./src/types` folder**
- Use PascalCase with "I" prefix for interface names
- **File names must end with `.types.ts` suffix**
- Import interfaces directly from their specific files

### 2. Rule - Props Interface Organization
- **Place all props interfaces in `./src/types/props` folder**
- Use PascalCase with "Props" suffix for props interface names
- **File names must end with `.props.ts` suffix**
- Import props interfaces directly from their specific files

### 3. Rule - Optional Properties Definition
- **Use `?:` syntax for optional properties**
- Place optional properties after required properties
- Document optional properties with JSDoc comments when behavior is complex
- Provide default values in component implementation, not interface

### 4. Rule - Interface Structure Standards
- Group related properties together
- Use consistent property ordering: required first, optional second
- Include JSDoc comments for complex properties
- Extend base interfaces when sharing common properties

### 5. Rule - Enum Usage and Integration
- **Check if required enums exist in `./src/enums` folder**
- If enum doesn't exist, create one by reading [enum-patterns.mdc](mdc:.cursor/rules/enum-patterns.mdc) and import back to interface
- Always use enums instead of string literals or union types for predefined values
- Import enums directly from their specific files

### 6. Rule - Import and Export Pattern Consistency
- Import interfaces directly from their specific files
- Import enums directly from their specific files
- Use named exports, avoid default exports
- Import with full path: `import { IUserInterface } from './src/types/IUserInterface.types'`
- Import props with full path: `import { NavbarProps } from './src/types/props/NavbarProps.props'`
- Import enums with full path: `import { UserRoleEnum } from './src/enums/UserRoleEnum'`

## Examples

### ✅ Good Interface Structure with Enums

```typescript
// ./src/types/IUserInterface.types.ts
import { UserRoleEnum } from '../enums/UserRoleEnum';
import { ThemeEnum } from '../enums/ThemeEnum';

/**
 * Interface for user data structure
 * Note: UserRoleEnum and ThemeEnum should exist in ./src/enums/
 * If they don't exist, create them by reading enum rules
 */
export interface IUserInterface {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  role: UserRoleEnum;  // Check if UserRoleEnum exists, if not create it
  createdAt: Date;
  updatedAt: Date;
  avatar?: string;
  isActive?: boolean;
  preferences?: {
    theme: ThemeEnum;  // Check if ThemeEnum exists, if not create it
    notifications: boolean;
  };
}
```

```typescript
// ./src/types/props/NavbarProps.props.ts
import { IUserInterface } from '../IUserInterface.types';
import { NavbarVariantEnum } from '../../enums/NavbarVariantEnum';

/**
 * Props interface for Navbar component
 */
export interface NavbarProps {
  title: string;
  user: IUserInterface;
  isAuthenticated: boolean;
  onLogout: () => void;
  showNotifications?: boolean;
  className?: string;
  variant?: NavbarVariantEnum;
}
```

```typescript
// Usage in components
import { IUserInterface } from './src/types/IUserInterface.types';
import { NavbarProps } from './src/types/props/NavbarProps.props';
import { NavbarVariantEnum, UserRoleEnum } from './src/enums';

const Navbar: React.FC<NavbarProps> = ({ 
  title, 
  user, 
  isAuthenticated, 
  variant = NavbarVariantEnum.DEFAULT 
}) => {
  // Component implementation with enum usage
  const isAdmin = user.role === UserRoleEnum.ADMIN;
  
  return (
    <nav className={`navbar navbar--${variant}`}>
      {/* Component JSX */}
    </nav>
  );
};
```

### ❌ Bad Interface Structure

```typescript
// Wrong location: ./components/navbar.ts
interface navbarProps {  // Wrong: should be NavbarProps in ./types/props
  title: string
  user?: User  // Wrong: should use IUserInterface
  logout: Function  // Wrong: should be specific function signature
  variant?: 'default' | 'compact'  // Wrong: should use enum
}

// Wrong: mixed required and optional without proper ordering
export interface userInterface {  // Wrong: should be IUserInterface
  avatar?: string
  id: string  // Should come before optional properties
  name: string
  role: 'admin' | 'user'  // Wrong: should use enum
}

// Wrong: enum not used
const THEME_OPTIONS = {
  LIGHT: 'light',
  DARK: 'dark'
};  // Should be enum instead
```

## Good vs Bad Practices

| Aspect | ✅ Good Practice | ❌ Bad Practice |
|-----|---|-----|
| **Location** | `./src/types/IUserInterface.types.ts` | `./components/User.ts` |
| **Naming** | `IUserInterface`, `NavbarProps` | `userInterface`, `navbarProps` |
| **Optional Properties** | `avatar?: string` | `avatar: string \| undefined` |
| **File Organization** | Separate file per interface | Multiple interfaces in one file |
| **Imports** | Direct imports from specific files | Imports through index files |
| **Props Location** | `./src/types/props/NavbarProps.props.ts` | `./components/Navbar.ts` |
| **Enum Usage** | `role: UserRoleEnum` | `role: 'admin' \| 'user'` |
| **Enum Location** | `./src/enums/UserRoleEnum.ts` | Constants object or inline unions |

## Interface Types Comparison

| Interface Type | Location | Naming Pattern | Example |
|-----|---|---|-----|
| **Data Interface** | `./src/types/` | `I` + PascalCase + `Interface` + `.types.ts` | `IUserInterface.types.ts` |
| **Props Interface** | `./src/types/props/` | PascalCase + `Props` + `.props.ts` | `NavbarProps.props.ts` |
| **API Interface** | `./src/types/` | `I` + PascalCase + `Interface` + `.types.ts` | `IApiResponseInterface.types.ts` |
| **Config Interface** | `./src/types/` | `I` + PascalCase + `Interface` + `.types.ts` | `IConfigInterface.types.ts` |
| **Enum** | `./src/enums/` | PascalCase + `Enum` | `UserRoleEnum` |

## Advanced Examples

### ✅ Extended Interface Pattern with Enums

```typescript
// ./src/types/IBaseEntityInterface.types.ts
import { EntityStatusEnum } from '../enums/EntityStatusEnum';

export interface IBaseEntityInterface {
  id: string;
  createdAt: Date;
  updatedAt: Date;
  status: EntityStatusEnum;
  isActive?: boolean;
}
```

```typescript
// ./src/types/IUserInterface.types.ts
import { IBaseEntityInterface } from './IBaseEntityInterface.types';
import { UserRoleEnum } from '../enums/UserRoleEnum';

export interface IUserInterface extends IBaseEntityInterface {
  email: string;
  firstName: string;
  lastName: string;
  role: UserRoleEnum;
  avatar?: string;
}
```

### ✅ Complex Props Interface

```typescript
// ./src/types/props/DataTableProps.props.ts
import { IUserInterface } from '../IUserInterface.types';

export interface DataTableProps<T = IUserInterface> {
  data: T[];
  columns: Array<{
    key: keyof T;
    label: string;
    sortable?: boolean;
    render?: (value: T[keyof T], item: T) => React.ReactNode;
  }>;
  onSort: (key: keyof T, direction: 'asc' | 'desc') => void;
  onRowClick: (item: T) => void;
  loading?: boolean;
  emptyMessage?: string;
  className?: string;
  pagination?: {
    currentPage: number;
    totalPages: number;
    onPageChange: (page: number) => void;
  };
}
```

### ✅ API Response Interface Pattern with Enums

```typescript
// ./src/types/IApiResponseInterface.types.ts
import { ApiErrorCodeEnum } from '../enums/ApiErrorCodeEnum';

export interface IApiResponseInterface<T = any> {
  success: boolean;
  data: T;
  message: string;
  timestamp: string;
  errors?: Array<{
    field: string;
    message: string;
    code: ApiErrorCodeEnum;
  }>;
  pagination?: {
    currentPage: number;
    totalPages: number;
    totalItems: number;
    itemsPerPage: number;
  };
}

// Usage example
// ./src/types/IUserApiResponseInterface.types.ts
import { IApiResponseInterface } from './IApiResponseInterface.types';
import { IUserInterface } from './IUserInterface.types';

export interface IUserApiResponseInterface extends IApiResponseInterface<IUserInterface[]> {}
```

## Implementation Checklist

### Before Creating Interfaces
- [ ] Determine if it's a data interface (goes in `./src/types/`) or props interface (goes in `./src/types/props/`)
- [ ] Check if similar interfaces already exist that can be extended
- [ ] Identify if any properties need predefined values (create enums first)
- [ ] Plan the property structure and identify required vs optional fields
- [ ] Consider if generic types would be beneficial
- [ ] Create necessary enums in `./src/enums/` before referencing them

### While Creating Interfaces
- [ ] Use correct naming convention: `I` prefix for interfaces, `Props` suffix for props, `Enum` suffix for enums
- [ ] Place file in correct location (`./src/types/`, `./src/types/props/`, or `./src/enums/`)
- [ ] Import required enums at the top of interface files
- [ ] Use enums instead of string literals or union types for predefined values
- [ ] Order properties: required first, optional second
- [ ] Use `?:` syntax for optional properties
- [ ] Add JSDoc comments for complex properties
- [ ] Use specific function signatures instead of generic `Function` type

### After Creating Interfaces
- [ ] Import interface directly in components using the interface
- [ ] Verify TypeScript compilation passes
- [ ] Check that optional properties work as expected
- [ ] Test interface with actual component implementations
- [ ] Ensure import paths are correct and relative

### File Organization Checks
- [ ] Interface files are in correct directories
- [ ] Enum files are in `./src/enums/` directory
- [ ] File names match interface/enum names exactly
- [ ] No duplicate interface or enum definitions
- [ ] Consistent naming patterns across all interfaces and enums
- [ ] Props interfaces are separate from data interfaces
- [ ] Enums are separate from interfaces
- [ ] Import statements use direct file paths
- [ ] Enums are used instead of string literals where appropriate

## References

- [TypeScript Handbook - Interfaces](mdc:https:/www.typescriptlang.org/docs/handbook/interfaces.html)
- [React TypeScript Patterns](mdc:https:/react-typescript-cheatsheet.netlify.app)
- [TypeScript Best Practices](mdc:https:/typescript-eslint.io/rules)
- [Interface vs Type in TypeScript](mdc:https:/www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces)

## Reminders

### ⚡ Key Interface Rules to Remember

1. **Location Matters**: Data interfaces in `./src/types/`, props in `./src/types/props/`, enums in `./src/enums/`
2. **Naming Convention**: `IUserInterface.types.ts` for data, `NavbarProps.props.ts` for props, `UserRoleEnum` for enums
3. **Use Enums**: Always create and use enums instead of string literals for predefined values
4. **Optional Properties**: Always use `?:` syntax, never `| undefined`
5. **File Organization**: One interface/enum per file, matching names
6. **Import Pattern**: Always import directly from specific files
7. **Property Ordering**: Required properties first, optional second
8. **JSDoc Comments**: Document complex properties and behaviors
9. **Extend When Possible**: Use inheritance for shared properties
10. **Specific Types**: Avoid `any` and `Function`, use specific signatures and enums
11. **Consistency**: Follow patterns across entire codebase

### 🎯 Interface Quality Indicators

- Are interfaces in the correct directories?
- Do interface names follow the established patterns?
- Are optional properties clearly marked with `?:`?
- Is each interface in its own appropriately named file?
- Are all interfaces imported directly from their specific files?
- Do components properly implement the props interfaces?

### 📋 Interface Creation Checklist

- [ ] Interface name follows `IExampleInterface` or `ExampleProps` pattern
- [ ] File located in `./src/types/` or `./src/types/props/` as appropriate
- [ ] File name follows `.types.ts` or `.props.ts` suffix pattern
- [ ] Properties ordered with required first, optional second
- [ ] Optional properties use `?:` syntax
- [ ] Complex properties have JSDoc documentation
- [ ] Interface imported directly in usage files
- [ ] No use of `any` or generic `Function` types
- [ ] Extends base interfaces when sharing properties
- [ ] TypeScript compilation passes without errors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mrvedmutha)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mrvedmutha)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
