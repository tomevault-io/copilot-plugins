## las-lira

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Las-Lira is a comprehensive flower shop management system designed for a floristry business operating through two sales channels:
- **Shopify**: Predefined arrangements sold by style and color
- **WhatsApp**: Custom orders with personalized specifications

The system manages inventory (flowers, containers), product catalog with recipes, order tracking through Kanban-style states, delivery route optimization, and customer relationship management.

## Tech Stack

**Backend:**
- Python 3.x with Flask
- Flask-SQLAlchemy (SQLite database)
- Flask-CORS for cross-origin requests
- openpyxl and pandas for Excel file handling

**Frontend:**
- React 18 with Vite
- React Router for navigation
- Tailwind CSS for styling
- react-beautiful-dnd for drag-and-drop Kanban board
- axios for API calls
- date-fns for date manipulation
- lucide-react for icons

## Development Commands

### Backend (from `/backend` directory)

```bash
# Install dependencies
pip install -r requirements.txt

# Run development server (port 5001)
python app.py
# or
python run.py

# Initialize/recreate database
python recrear_db.py

# Import demo data
python importar_datos_demo.py
```

The backend runs on `http://localhost:5001` by default. API endpoints are at `/api/*`.

### Frontend (from `/frontend` directory)

```bash
# Install dependencies
npm install

# Run development server (port 3001)
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Run linter
npm run lint
```

The frontend runs on `http://localhost:3001` and proxies API requests to the backend.

## Architecture

### Backend Structure

The backend follows a layered architecture pattern:

**Models** (`backend/models/`):
- Define SQLAlchemy ORM models
- Each model includes `to_dict()` method for JSON serialization
- Key models: `Pedido` (orders), `Cliente` (customers), `Producto` (products), `Flor` (flowers), `Contenedor` (containers), `Evento` (events)
- Relationships are defined using SQLAlchemy's relationship() and foreign keys

**Services** (`backend/services/`):
- Business logic layer separated from routes
- Handle complex operations, validations, and data transformations
- Key services:
  - `pedidos_service.py`: Order management, status updates, payment tracking
  - `inventario_service.py`: Stock management, availability checks
  - `eventos_service.py`: Event management
  - `reportes_service.py`: Analytics and reporting
  - `clientes_service.py`: Customer relationship management

**Routes** (`backend/routes/`):
- Blueprint-based route handlers
- Thin layer that calls service methods
- Handle HTTP request/response, validation, error handling
- All routes are prefixed with `/api/`

**Utils** (`backend/utils/`):
- Helper functions and utilities
- `fecha_helpers.py`: Date classification logic for order states (today, tomorrow, this week)

**Config** (`backend/config/`):
- Configuration modules for business rules
- `comunas.py`: Delivery zones and pricing
- `etiquetas.py`: Label definitions
- `motivos.py`: Order occasion types
- `plazos_pago.py`: Payment term rules by customer type
- `cobranza.py`: Payment methods

**Scripts** (`backend/scripts/`):
- Standalone scripts for database migrations, data imports, one-time operations
- Not part of the main application flow

### Frontend Structure

**Pages** (`frontend/src/pages/`):
- Top-level route components
- Key pages:
  - `TableroPage.jsx`: Kanban board for order workflow
  - `PedidosPage.jsx`: Order management
  - `TallerPage.jsx`: Workshop/production view
  - `EventosPage.jsx`: Event management
  - `ClientesPage.jsx`: Customer management
  - `CobranzaPage.jsx`: Payment tracking
  - `RutasPage.jsx`: Delivery route optimization
  - `InsumosPage.jsx`: Inventory management
  - `ProductosPage.jsx`: Product catalog
  - `SimuladorCostosPage.jsx`: Cost estimation
  - `ReportesPage.jsx`: Reports and analytics

**Components** (`frontend/src/components/`):
- Reusable UI components
- `Tablero/`: Kanban board components (ColumnaKanban, TarjetaPedido)
- `common/`: Shared components (Layout, ImageUpload)
- `Pedidos/`: Order-specific components (SelectorInsumosColores)

**Services** (`frontend/src/services/`):
- API client layer using axios
- Centralized API endpoint definitions

### Database Schema

The SQLite database (`backend/instance/laslira.db`) contains these main tables:

**Core Tables:**
- `pedidos`: Orders with full lifecycle tracking
- `clientes`: Customer information and history
- `productos`: Product catalog
- `flores`: Flower inventory
- `contenedores`: Container inventory (vases, pots, baskets)
- `eventos`: Special events requiring multiple orders

**Relationship Tables:**
- `pedidos_insumos`: Materials used per order
- `recetas_productos`: Product recipes (materials needed per product)
- `proveedores`: Supplier information
- `bodegas`: Warehouse locations

### Order State Flow

Orders move through these states (similar to Trello/Kanban):
1. **Pedidos Semana** (Orders This Week) - Initial state for future orders
2. **Entregas para Mañana** (Deliveries Tomorrow) - Auto-classified by date
3. **Entregas de Hoy** (Deliveries Today) - Auto-classified by date
4. **En Proceso** (In Progress) - Being prepared
5. **Listo para Despacho** (Ready for Dispatch) - Completed and ready
6. **Despachados** (Dispatched) - Out for delivery
7. **Entregado** (Delivered) - Final state
8. **Cancelado** (Cancelled) - Cancelled orders

The `utils/fecha_helpers.py` module contains the `clasificar_pedido()` function that automatically determines the initial state based on delivery date. The `pedidos_service.py` has an `actualizar_estados_por_fecha()` method that reclassifies orders, but it never downgrades orders that are in active work states (En Proceso, Listo para Despacho) and never touches Despachados state.

### Inventory Management

The system tracks three quantities for each inventory item (flores and contenedores):
- `cantidad_stock`: Total physical stock
- `cantidad_en_uso`: Reserved for confirmed orders (in pedidos_insumos)
- `cantidad_en_evento`: Reserved for events
- `cantidad_disponible` (computed property): stock - en_uso - en_evento

When an order moves to "En Proceso", stock can be reserved. When cancelled, stock is returned.

### Service Layer Pattern

When adding or modifying business logic:
1. Place logic in the appropriate service class (e.g., `PedidosService`, `InventarioService`)
2. Use static methods for service functions
3. Routes should be thin - just call service methods and handle HTTP concerns
4. Services should return tuples: `(success: bool, result/error, message: str)`
5. Services handle database transactions (commit/rollback)

Example pattern:
```python
# In service
class PedidosService:
    @staticmethod
    def crear_pedido(data):
        try:
            # Business logic here
            db.session.commit()
            return True, pedido, 'Success message'
        except Exception as e:
            db.session.rollback()
            return False, None, str(e)

# In route
@bp.route('/pedidos', methods=['POST'])
def crear():
    data = request.get_json()
    success, result, message = PedidosService.crear_pedido(data)
    if success:
        return jsonify({'pedido': result.to_dict()}), 201
    else:
        return jsonify({'error': message}), 400
```

### Date Handling

The system uses `datetime.datetime` objects throughout. Key considerations:
- `fecha_entrega`: Delivery date (datetime)
- `fecha_pedido`: Order creation date (datetime)
- `fecha_maxima_pago`: Payment deadline (datetime, can be null)
- Dates are classified relative to "today" to determine order urgency
- The `clasificar_pedido()` function in `utils/fecha_helpers.py` handles all date-based state logic

### Customer Types and Payment Terms

Customer types (`clientes.tipo_cliente`):
- Nuevo (New): 0 days payment term
- Regular: 7 days
- Frecuente (Frequent): 15 days
- Empresarial (Business): 30 days
- VIP: 45 days

Payment terms are defined in `config/plazos_pago.py` and automatically applied when creating orders.

## Common Development Patterns

### Adding a New Route

1. Create or modify a blueprint in `backend/routes/`
2. Import and register the blueprint in `backend/app.py`
3. Use the appropriate service for business logic
4. Return JSON responses with proper HTTP status codes

### Adding a New Model

1. Create model class in `backend/models/`
2. Include `to_dict()` method for JSON serialization
3. Define relationships using SQLAlchemy
4. Run database recreation if schema changes: `python recrear_db.py`

### Frontend API Calls

Use the centralized API service in `frontend/src/services/api.js`:
```javascript
import api from '../services/api'

const response = await api.get('/pedidos')
const data = await api.post('/pedidos', { ...orderData })
```

### Working with the Kanban Board

The TableroPage uses react-beautiful-dnd. When modifying:
- Columns represent order states
- Drag-and-drop updates the `estado` field via API
- The board fetches data from `/api/pedidos/tablero`
- Filters can exclude "Despachados" state for performance

## Important Notes

- The database file is in `backend/instance/laslira.db` (SQLite)
- CORS is configured to allow requests from frontend ports (3001, 5173)
- Models use the extension pattern (`extensions.py`) to avoid circular imports
- Many scripts in `backend/scripts/` are for one-time data migrations or imports
- The system supports both manual orders (WhatsApp) and automated orders (Shopify)
- Product "recipes" (recetas_productos) define which flowers/containers are needed for each product
- Events can have multiple orders associated with them
- Photos can be attached to orders via `foto_enviado_url` field

## File Naming Conventions

- Backend: snake_case for Python files and modules
- Frontend: PascalCase for React components (.jsx), camelCase for utilities
- Models are singular (e.g., `pedido.py`, `cliente.py`)
- Routes are plural (e.g., `pedidos_routes.py`, `clientes_routes.py`)
- Services are singular with "_service" suffix (e.g., `pedidos_service.py`)

---
> Source: [juliettegramatges/Las-Lira](https://github.com/juliettegramatges/Las-Lira) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
