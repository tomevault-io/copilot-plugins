## smart-inventory-hub

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project: Smart Inventory Hub - Frontend-Only Enterprise Dashboard

### Overview
Production-ready **frontend-only** inventory management dashboard built with React and TypeScript. Features client-side Excel file processing, IndexedDB storage, interactive analytics, and comprehensive file management. Runs entirely in the browser with no backend server required. Handles enterprise-scale datasets (~3K inventory items, ~45M total value) with advanced analytics and PKR currency formatting.

### Architecture - Complete Client-Side Processing

```
Excel Upload → excelProcessor.ts → dataStorage.ts (IndexedDB) → inventoryService.ts → Dashboard Analytics
```

- **Frontend**: React 19 + TypeScript with Vite 7.0.4 build system
- **Storage**: Browser IndexedDB with custom service wrapper (~250MB capacity)
- **Processing**: Client-side Excel parsing with comprehensive validation
- **Analytics**: Real-time metrics, location/plant statistics, drill-down navigation
- **Deployment**: Static hosting only (Netlify/Vercel/GitHub Pages compatible)

## Key Commands

### Development (run from `inventory-frontend/`)
```bash
npm install                  # Install dependencies
npm run dev                  # Development server (localhost:5173)
npm run build                # Production build with optimization
npm run preview              # Preview production build
npm run lint                 # ESLint checking
```

### Testing Infrastructure
```bash
npm test                     # Run tests in watch mode
npm run test:run             # Run tests once
npm run test:ui              # Interactive test UI dashboard
npm run test:coverage        # Generate coverage report (75% target)
npm run test:watch           # Watch mode with file monitoring

# Run specific tests
npm run test:run src/services/dataStorage.test.ts
npm run test:run -- --reporter=verbose
```

### Build & Deployment
```bash
npm run build                # Generates dist/ folder for static hosting
npm run preview              # Test production build locally

# Netlify deployment ready - includes netlify.toml config
# Supports any static hosting: Vercel, GitHub Pages, CDN
```

## Core Service Architecture

### Service Layer (`src/services/`)
1. **`dataStorage.ts`** (383 lines) - IndexedDB wrapper and schema manager
   - Multi-store database: inventory, files, settings
   - File metadata with validation status tracking
   - Active file management and persistence
   - Debug utilities: `debugStorage()` in browser console

2. **`inventoryService.ts`** (681 lines) - Business logic engine
   - Excel data processing (object/array formats)
   - Metrics calculations: KPIs, location stats, plant analytics
   - Advanced filtering with drill-down functionality
   - Pagination with configurable page sizes (10, 20, 50, 100)
   - Search with 300ms debouncing for performance

3. **`excelProcessor.ts`** (792 lines) - File processing pipeline
   - File validation: type (.xlsx/.xls), size (10MB max), structure
   - Comprehensive Excel validation with error/warning reporting
   - Template generation and download
   - File lifecycle: upload → validate → process → activate → delete

4. **`clientApi.ts`** - API abstraction layer
   - Drop-in replacement for backend API interface
   - Service orchestration and initialization
   - Error handling and state management

### Component Architecture Patterns
- **Interactive KPI Cards**: Clickable metrics with drill-down navigation
- **Dynamic Charts**: Recharts integration with click handlers for navigation
- **Advanced File Management**: Drag-and-drop upload with real-time validation
- **Specialized Analytics Views**: 15+ dedicated views for different data perspectives
- **Responsive Design**: Mobile-first with corporate branding and PKR formatting

## Data Processing Capabilities

### IndexedDB Schema
```
InventoryHub Database (v1)
├── inventory (store) - Material records with indexes
│   ├── material, plant, storageLocation, status, fileId (indexes)
├── files (store) - File metadata and upload history  
├── settings (store) - Active file tracking and app configuration
```

### Excel Processing Pipeline
- **Supported Formats**: .xlsx, .xls files up to 10MB
- **Required Columns**: Material, Description, Plant, Storage Location, Unrestricted Stock, Blocked Stock
- **Validation**: Real-time structure validation with detailed error reporting
- **Processing**: Automatic data type conversion and status categorization
- **Performance**: Handles 3K+ items with in-memory processing for fast analytics

### Analytics Capabilities
- **Metrics**: Total inventory, blocked/unrestricted/restricted stock values
- **Location Analytics**: Storage location distribution and drill-downs
- **Plant Analytics**: Plant-wise inventory distribution and performance
- **Material Management**: Paginated listings with search and status filtering
- **Status Categories**: Blocked, unrestricted, restricted, in-transfer, quality-inspection

## Testing Infrastructure (130+ Test Cases)

### Test Categories
- **Unit Tests**: Core services (dataStorage, inventoryService, excelProcessor)
- **Component Tests**: React components with user interaction testing
- **Integration Tests**: Complete file upload and processing workflows
- **Coverage**: 75% minimum across lines, branches, functions, statements

### Test Utilities
- **Mock Data Generators**: Consistent test data creation
- **Browser API Mocking**: IndexedDB, File APIs, URL methods
- **Custom Render**: React Testing Library with all providers
- **Performance Testing**: Async operation and large dataset handling

### Development Testing Guidelines
- Make sure to add test cases for the functionality to develop and keep the test cases up-to-date
- Don't run the application to test, always run the test cases and add as much test cases as you can and test only using test cases

## Development Workflows

### File Processing Development
1. **Service Testing**: Use individual service tests for logic validation
2. **Integration Testing**: Full workflow tests for end-to-end verification
3. **Browser Debug**: Use `debugStorage()` for IndexedDB inspection
4. **Error Handling**: Test validation scenarios with various Excel formats

### UI Component Development
1. **Component Tests**: React Testing Library for user interaction
2. **Theme Integration**: Test with ThemeProvider for dark/light modes
3. **Responsive Testing**: Verify mobile, tablet, desktop layouts
4. **Accessibility**: ARIA labels, keyboard navigation, screen reader compatibility

### Performance Optimization
- **Debounced Search**: 300ms delay for search input
- **Pagination**: Configurable page sizes for large datasets
- **Lazy Loading**: Chart components load on demand
- **Memory Management**: Proper cleanup in useEffect hooks

## Browser Requirements & Limitations

- **IndexedDB Support**: Required for data persistence (~250MB typical limit)
- **ES2015+ Browser**: Modern JavaScript features required
- **File API Support**: For Excel upload and processing
- **Offline Capability**: Full functionality after initial load
- **Memory**: Large datasets (3K+ items) processed in browser memory

## Debugging Tools

### Browser Console
```javascript
debugStorage()                    # Inspect IndexedDB contents and structure
inventoryService.isDataLoaded_()  # Check if service has data loaded
inventoryService.getDataCount()   # Get current loaded record count
```

### Service Monitoring
- Monitor `[DataStorage]` logs for IndexedDB operations
- Check `[InventoryService]` logs for data processing and metrics
- Watch `[ExcelProcessor]` logs for file validation and processing
- Review activation sequence logs during file switching

## Build Configuration & Deployment

### Vite Configuration
- **Manual Chunk Splitting**: vendor, charts, excel libraries separated
- **Asset Optimization**: CSS/JS minification and compression
- **TypeScript Integration**: Strict type checking with project references
- **Static Output**: All resources bundled for offline capability

### Deployment Options
- **Netlify**: Includes netlify.toml with build configuration
- **Vercel**: Compatible with automatic deployment from Git
- **GitHub Pages**: Static site hosting from dist/ folder
- **Any CDN**: Standard static file hosting

### Performance Characteristics
- **Bundle Size**: ~1.26 MB total (compressed: ~357 KB)
- **Load Time**: Fast initial load with code splitting
- **Processing**: Real-time Excel validation and processing
- **Storage**: Persistent data across browser sessions
- **Analytics**: Sub-second response times for large datasets

## Common Issues & Solutions

### File Processing Issues
1. Check browser console for `[ExcelProcessor]` validation logs
2. Use `debugStorage()` to inspect saved data
3. Verify required columns match template format
4. Test with provided template files in `examples/` directory

### Performance Issues
1. Monitor memory usage with large Excel files (>1MB)
2. Use pagination for datasets >1000 items
3. Clear browser cache if IndexedDB becomes corrupted
4. Check Network tab for asset loading issues in production
```

---
> Source: [shariqbaig/smart-inventory-hub](https://github.com/shariqbaig/smart-inventory-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
