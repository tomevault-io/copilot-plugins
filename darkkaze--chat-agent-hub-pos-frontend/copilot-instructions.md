## chat-agent-hub-pos-frontend

> Point of Sale (POS) application built with Vue 3 + TypeScript + Vuetify. Modern POS interface with product catalog, customer management, sales processing, and sales history with responsive design and Pink Candy theme.

# Agent Hub POS Frontend

Point of Sale (POS) application built with Vue 3 + TypeScript + Vuetify. Modern POS interface with product catalog, customer management, sales processing, and sales history with responsive design and Pink Candy theme.

## Project Structure

```
src/
├── components/
│   ├── admin/
│   │   ├── CreateProductModal.vue         # Product creation modal
│   │   ├── EditProductModal.vue           # Product editing modal
│   │   └── DeleteProductDialog.vue        # Product deletion confirmation
│   ├── pos/
│   │   ├── CustomerSearch.vue             # Customer lookup by phone
│   │   ├── ProductSearch.vue              # Product search and cart actions
│   │   ├── CartItems.vue                  # Shopping cart display
│   │   ├── PaymentMethods.vue             # Payment processing interface
│   │   ├── SaleConfirmation.vue           # Sale completion confirmation
│   │   └── SaleDetailsModal.vue           # Sale details modal
│   └── common/
│       ├── StatusChip.vue                 # Generic status indicator
│       └── LoadingSpinner.vue             # Loading states
├── layouts/
│   └── POSLayout.vue                      # Main POS layout with tab navigation
├── views/
│   ├── POSPage.vue                        # Main POS sale interface
│   ├── ProductsAdminView.vue              # Product catalog management
│   └── SalesHistoryView.vue               # Sales history with pagination
├── router/
│   └── index.ts                           # Vue Router configuration
├── stores/
│   ├── pos.ts                            # POS cart and payment state
│   └── customer.ts                       # Customer state management
├── services/
│   ├── api.ts                            # Base API client (axios)
│   └── pos/
│       ├── index.ts                      # Service exports
│       ├── productsService.ts            # Product CRUD operations
│       ├── customersService.ts           # Customer management
│       └── salesService.ts               # Sales processing and history
├── plugins/
│   └── vuetify.ts                        # Vuetify configuration with Pink Candy theme
└── types/
    ├── api.ts                            # Base API types and pagination
    └── pos.ts                            # POS domain types (Customer, Product, Sale)
```

## Features

- **Product Management** - CRUD operations for product catalog
- **Customer Management** - Phone-based customer lookup and loyalty points
- **Sales Processing** - Multi-item cart with quantity management
- **Multiple Payment Methods** - Cash, card, transfer, loyalty points
- **Loyalty Points System** - Points generation and redemption
- **Sales History** - Paginated history with search and filtering
- **Responsive Design** - Mobile-first approach for tablet POS systems
- **Real-time Calculations** - Dynamic totals and payment validation

## POS Workflow

1. **Customer Identification** - Search by phone number or create new customer
2. **Product Selection** - Search products and add to cart with quantities
3. **Cart Management** - Review items, modify quantities, apply discounts
4. **Payment Processing** - Select payment methods (supports multiple methods)
5. **Sale Completion** - Process loyalty points and generate receipt
6. **History Tracking** - All sales stored with complete transaction details

## Routing Structure

- `/` → redirects to `/pos/sale`
- `/pos/sale` → Main POS sale interface
- `/products` → Product catalog management
- `/tickets` → Sales history view

## State Management

### POS Store (`stores/pos.ts`)
- Cart items management
- Payment methods state
- Sale calculations (subtotal, discount, total)
- Loyalty points calculation
- Sale processing state

### Customer Store (`stores/customer.ts`)
- Current customer selection
- Customer search results
- Loyalty points balance

## API Integration

### Services Architecture
- **productsService**: Product CRUD, search functionality
- **customersService**: Customer lookup, creation, wallet updates
- **salesService**: Sale creation, history retrieval with pagination

### API Endpoints
- `GET /products` - List active products
- `GET /products/search?q=query` - Search products
- `POST /products` - Create product
- `PUT /products/{id}` - Update product
- `DELETE /products/{id}` - Deactivate product
- `GET /customers/search?phone=number` - Customer lookup
- `POST /customers` - Create customer
- `POST /sales` - Process sale
- `GET /sales` - Sales history with pagination

## Theme and Design

**Pink Candy Theme** (Vuetify configuration):
- Primary: Pink 500
- Secondary: Purple 400
- Accent: Deep Purple 300
- Success: Teal 400
- Warning: Orange 400
- Error: Red 500

**Design Principles**:
- Clean, modern POS interface
- High contrast for readability
- Touch-friendly components for tablets
- Consistent spacing and typography
- Responsive grid system

## Run Commands

- `npm run dev` → Development server
- `npm run type-check` → TypeScript type checking
- `npm run build` → Production build
- `npm run preview` → Preview production build

## Development Notes

- All prices stored as strings for decimal precision
- Loyalty points calculated as percentage of sale total
- Vector search prepared but disabled (embedding_vector field reserved)
- Full TypeScript coverage with strict type checking
- Vuetify 3 components with Material Design Icons

---
> Source: [darkkaze/chat-agent-hub-pos-frontend](https://github.com/darkkaze/chat-agent-hub-pos-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
