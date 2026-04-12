## maamsfactv3

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a macOS invoicing application built with SwiftUI and SwiftData called "Facturation" (French for "Billing"). The app helps manage clients, invoices, products, and business information for a company named ExoTROPIC.

## Build & Development Commands

### Building the Project
- **Build**: `⌘ + B` in Xcode
- **Run**: `⌘ + R` in Xcode
- **Clean**: `⌘ + Shift + K` in Xcode

### Requirements
- macOS 14.0+ (Sonoma)
- Xcode 15.0+
- Swift 5.9+

### No Package Manager
This project uses native Apple frameworks only - no external dependencies or package managers.

## Architecture

### Core Pattern
- **Architecture**: MVVM with SwiftData persistence
- **UI Framework**: SwiftUI for macOS
- **Data Layer**: SwiftData (Apple's successor to Core Data)
- **Main Service**: `DataService` (singleton) handles all CRUD operations

### Key Components
```
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ Models          │ │ Services        │ │ Views           │
│ (SwiftData)     │─│ DataService     │─│ SwiftUI         │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

## Data Models

### Core Models (`Models/`)
- **`Facture`**: Invoice entity with status tracking, VAT calculations, and line items
- **`LigneFacture`**: Invoice line items with quantity, unit price, and product references  
- **`Client`**: Customer entity with contact info and address details
- **`Entreprise`**: Company settings and invoice numbering configuration
- **`Produit`**: Product catalog with pricing and descriptions
- **`Statistiques`**: Statistics and analytics data model

### Model Relationships
- `Facture` ↔ `Client` (many-to-one)
- `Facture` ↔ `LigneFacture` (one-to-many, cascade delete)
- `LigneFacture` ↔ `Produit` (many-to-one, optional)

### Important Enums
- `StatutFacture`: Invoice status (brouillon, envoyee, payee, EnRetard, annulee)
- `ConditionsPaiement`: Payment methods (virement, cheque, espece, carte)

### ViewModels (`ViewModels/`)
- **`EditableFacture`**: Editable wrapper for invoice editing
- **`EditableLigneFacture`**: Editable wrapper for invoice line items

## Data Service (`Services/DataService.swift`)

### Key Features
- **Singleton Pattern**: `DataService.shared`
- **Published Properties**: `@Published var clients, factures, produits`
- **Async/Await**: All CRUD operations are async
- **Resilient Init**: Handles SwiftData container failures with fallback to in-memory storage

### Common Operations
```swift
// Fetching data
await dataService.fetchClients()
await dataService.fetchFactures()

// CRUD operations
await dataService.addClient(client)
await dataService.updateFacture(facture)
await dataService.deleteClient(client)

// Utilities
let stats = dataService.getStatistiques()
let numero = await dataService.genererNumeroFacture()
```

## View Structure

### Main Views (`Views/`)
- **Dashboard**: Overview with statistics and recent activity (DashboardView/)
- **Clients**: Client management (list, add, edit, delete) (Client/)
- **Factures**: Invoice management with status filtering (FacturesView/)
- **Parametres**: Company settings and configuration (Parametres/)
- **Produits**: Product catalog management (Produits/)
- **Stats**: Charts and analytics (Stats/)
- **Developer**: Developer tools and debugging (Developer/)

### Navigation
- Modern sidebar-based navigation with `ModernContentView` as root
- Tab-based structure with enum `NavigationTab`

### Shared Components (`Shared/`)
- **Components/**: Reusable UI components (AppButton, AppCard, AppTextField, WindowManager)
- **Extensions/**: Utility extensions for common tasks

## PDF Generation

The app includes comprehensive PDF generation capabilities using:
- **Core Graphics**: For custom PDF layouts
- **PDFKit**: For PDF handling and display
- **PDFService**: Handles PDF creation and formatting with advanced layout
- **PageLayoutCalculator**: Calculates optimal page layouts for invoices
- **InvoicePageContent**: Structures invoice content for PDF generation
- **ClientFacturesPDFView**: SwiftUI view for client-specific PDF generation

## Extensions & Utilities

### Extensions (`Extensions/`)
- **`MacOSColors.swift`**: macOS-specific color adaptations
- **`Extensions.swift`**: Double, Date, String formatting utilities
- **`Client+Extensions.swift`**: Client-specific computed properties and methods

### Key Utilities
- Euro currency formatting
- French date formatting
- Email/SIRET validation
- Address formatting helpers

## Testing

### Test Structure
- **`FacturationTests/`**: Contains unit tests
- **`ImportPDFTests.swift`**: PDF import functionality tests

### Running Tests
- Use `⌘ + U` in Xcode to run all tests
- Individual test methods can be run via Xcode test navigator
- **Test Target**: `FacturationTests` with PDF import testing capabilities

### Testing Infrastructure
- Tests use in-memory SwiftData containers for isolation
- PDF and Excel import functionality is thoroughly tested
- XCTest framework with Vision and PDFKit testing support

## Import/Export Features

### Supported Formats
- **Excel Import**: `ExcelImporter.swift` with CoreXLSX dependency for importing client and invoice data
- **PDF Import**: `PDFImporter.swift` for extracting invoice data from PDF documents using Vision framework
- **JSON Export**: Built-in export functionality for clients and invoices

### Additional Services
- **StatistiquesService**: Handles statistics calculations and data aggregation
- **DataService**: Centralized data management with async/await patterns

## Special Considerations

### SwiftData Gotchas
- Always insert entities into ModelContext before establishing relationships
- Use `@Relationship(inverse:)` for bidirectional relationships
- Handle container initialization failures gracefully
- **Resilient initialization**: DataService implements fallback to in-memory storage if persistent storage fails
- **Migration handling**: Automatic schema migration with error recovery

### macOS Specifics
- Window minimum size: 1200x800
- Uses native macOS UI patterns and colors
- Supports keyboard shortcuts and menu commands

### French Localization
- UI is primarily in French
- Handles French business requirements (SIRET, TVA, etc.)
- Uses European number formatting
- Company name: ExoTROPIC (default business entity)

## Development Notes

### Code Style
- Swift naming conventions throughout
- Extensive use of computed properties for derived values
- Async/await pattern for all data operations
- SwiftUI view composition with clear separation of concerns

### Error Handling
- Comprehensive error handling in DataService
- Fallback mechanisms for data persistence failures
- User-friendly error messages in French

## Current Project Status

### Key Features Implemented
- Complete CRUD operations for clients, invoices, and products
- Modern SwiftUI interface with macOS-specific adaptations
- Advanced PDF generation with custom layouts
- Excel/PDF import capabilities with Vision framework
- Comprehensive statistics and analytics
- Robust data persistence with SwiftData
- French business compliance (SIRET, TVA validation)

### External Dependencies
- **CoreXLSX**: For Excel file import functionality
- **Vision**: For PDF text extraction and processing
- **PDFKit**: For PDF document handling

### Asset Management
- Extensive sunset image collection (37 images in ViewModels/Assets.xcassets/Sunsets/)
- Complete app icon set with multiple resolutions
- Organized asset catalogs for maintainability

This codebase represents a complete, production-ready macOS invoicing application with robust data management, PDF generation, and import/export capabilities.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YoungSlim971)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/YoungSlim971)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
