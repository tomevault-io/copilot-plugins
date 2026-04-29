## gbot-v01

> This directory contains comprehensive AI assistant rules and patterns for the Graffiti Generation Application. These files provide context, patterns, and best practices to help AI assistants work effectively with this codebase.

# AI Rules Documentation

This directory contains comprehensive AI assistant rules and patterns for the Graffiti Generation Application. These files provide context, patterns, and best practices to help AI assistants work effectively with this codebase.

## File Overview

### 📋 [context-gathering-guide.mdc](mdc:context-gathering-guide.mdc)
**Essential reading for all AI interactions**
- Project overview and key technologies
- Context gathering checklist for different types of tasks
- Common patterns and red flags to watch for
- Questions to ask users for clarification
- Systematic approach to understanding requirements

### ⚛️ [react-typescript-patterns.mdc](mdc:react-typescript-patterns.mdc)
**React and TypeScript development patterns**
- Component structure and prop typing
- Custom hooks patterns and return types
- Error handling in components and async operations
- Performance optimization with memoization
- Form handling and validation patterns
- TypeScript best practices and type definitions

### 🏪 [zustand-state-management.mdc](mdc:zustand-state-management.mdc)
**State management with Zustand**
- Store structure and organization patterns
- History management (undo/redo functionality)
- Persistent vs session state handling
- Async actions and optimistic updates
- Store composition and custom hooks
- Performance considerations and selectors

### 🎨 [svg-processing-optimization.mdc](mdc:svg-processing-optimization.mdc)
**SVG processing and performance optimization**
- SVG processing pipeline and validation
- Multi-level caching strategies
- Pixel analysis for collision detection
- Overlap calculation optimization
- SVGO integration and optimization levels
- Performance monitoring and metrics

### 🗄️ [supabase-database-patterns.mdc](mdc:supabase-database-patterns.mdc)
**Database design and Supabase integration**
- Table structure and RLS (Row Level Security) policies
- Migration patterns and best practices
- TypeScript integration with generated types
- Repository pattern for database operations
- Real-time subscriptions and error handling
- Performance optimization and security

## How to Use These Rules

### For AI Assistants
1. **Start with context-gathering-guide.mdc** to understand the project and gather proper context
2. **Reference specific pattern files** based on the type of work being done
3. **Follow established patterns** rather than creating new approaches
4. **Ask clarifying questions** when requirements are unclear

### For Developers
1. **Use as reference** for consistent coding patterns
2. **Follow for new feature development** to maintain consistency
3. **Reference for troubleshooting** common issues
4. **Update when patterns evolve** to keep documentation current

## Key Principles

### 🎯 **Context First**
Always gather sufficient context before starting any work. Understanding the specific feature, user goal, and existing patterns is crucial for effective assistance.

### 🔄 **Consistency**
Follow established patterns in the codebase. These rules document existing successful patterns rather than imposing new ones.

### 🚀 **Performance Aware**
Consider performance implications, especially for SVG processing, state management, and database operations.

### 🔒 **Security Conscious**
Follow security best practices, particularly for authentication, database access, and data validation.

### 🧪 **Testing Ready**
Write code that is testable and follows patterns that support good testing practices.

## MCP Tools Integration

### 🛠️ **Supabase MCP Tools**
Always use Supabase MCP functions for database operations:
- `mcp_supabase_list_projects()` - Project management
- `mcp_supabase_apply_migration()` - Schema changes
- `mcp_supabase_execute_sql()` - SQL operations
- `mcp_supabase_generate_typescript_types()` - Type generation
- `mcp_supabase_get_logs()` - Debugging and monitoring

### 🌐 **Browser Tools MCP**
Use browser tools for debugging and optimization:
- `mcp_browser-tools_takeScreenshot()` - Visual debugging
- `mcp_browser-tools_getConsoleLogs()` - JavaScript debugging
- `mcp_browser-tools_runPerformanceAudit()` - Performance analysis
- `mcp_browser-tools_runAccessibilityAudit()` - Accessibility testing
- `mcp_browser-tools_runSEOAudit()` - SEO optimization

### 📋 **MCP Best Practices**
1. **Prefer MCP tools** over manual operations when available
2. **Use appropriate tools** for the specific task (database vs browser)
3. **Combine tools** for comprehensive debugging (logs + screenshots)
4. **Follow MCP patterns** established in the codebase

## Project-Specific Context

### Core Features
- **Graffiti Generation**: SVG-based text rendering with customizable styles
- **Real-time Customization**: Live preview of style changes
- **User Authentication**: Supabase-based auth with session management
- **Preset Management**: Save and share customization presets
- **Performance Optimization**: Sophisticated caching and SVG optimization

### Technology Stack
- **Frontend**: React 18 + TypeScript + Vite
- **State**: Zustand with persistence middleware
- **UI**: Tailwind CSS + Shadcn UI components
- **Backend**: Supabase (PostgreSQL + Auth + Real-time)
- **SVG**: Custom processing with SVGO optimization
- **Testing**: Vitest + React Testing Library

### Architecture Patterns
- **Component-based**: Clear separation of concerns
- **Custom hooks**: Reusable logic encapsulation
- **Repository pattern**: Database access abstraction
- **Caching layers**: Multi-level performance optimization
- **Error boundaries**: Graceful error handling

## Common Scenarios

### 🐛 Bug Fixes
1. Read context-gathering-guide.mdc for systematic debugging approach
2. Reference relevant pattern files for proper implementation
3. Follow error handling patterns from react-typescript-patterns.mdc

### ✨ New Features
1. Understand requirements using context-gathering-guide.mdc
2. Follow component patterns from react-typescript-patterns.mdc
3. Use appropriate state management from zustand-state-management.mdc
4. Consider database changes using supabase-database-patterns.mdc

### ⚡ Performance Issues
1. Check SVG processing patterns in svg-processing-optimization.mdc
2. Review state management performance in zustand-state-management.mdc
3. Examine database query optimization in supabase-database-patterns.mdc

### 🔐 Security Concerns
1. Follow RLS patterns in supabase-database-patterns.mdc
2. Use proper error handling from react-typescript-patterns.mdc
3. Implement data validation and sanitization

## Quick Reference

| Task Type | Primary Files | Key Considerations | MCP Tools |
|-----------|---------------|-------------------|-----------|
| Bug Fix | context-gathering-guide.mdc, react-typescript-patterns.mdc | Error handling, debugging approach | Browser Tools for console logs/screenshots |
| New Component | react-typescript-patterns.mdc, zustand-state-management.mdc | Component structure, state integration | Browser Tools for testing/accessibility |
| SVG Feature | svg-processing-optimization.mdc, context-gathering-guide.mdc | Performance, caching, browser compatibility | Browser Tools for performance audits |
| Database Work | supabase-database-patterns.mdc | RLS policies, type safety, migrations | Supabase MCP for all database operations |
| State Management | zustand-state-management.mdc | Store patterns, persistence, performance | Browser Tools for debugging state changes |

These rules are living documents that evolve with the project. They represent the collective knowledge and best practices developed through building this graffiti generation application.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/filthywill) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
