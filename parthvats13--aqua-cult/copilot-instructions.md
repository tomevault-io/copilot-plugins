## aqua-cult

> **Type:** Aquaculture Management System

# AquaSense - Project Context for AI Assistants

## Project Overview

**Name:** AquaSense
**Type:** Aquaculture Management System
**Platform:** Android (Kotlin/Jetpack Compose) + Python Backend (FastAPI)
**Purpose:** Help fish farmers manage tanks, detect diseases, get AI recommendations, and purchase supplies
**Development Stage:** Initial implementation (backend first approach)

---

## Quick Start

### Running the Backend
```bash
cd aquasense_backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
export GEMINI_API_KEY="your-key-here"
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### Running the Android App
1. Open project in Android Studio
2. Update `NetworkModule.kt`:
   - Emulator: `BASE_URL = "http://10.0.2.2:8000/api/v1/"`
   - Physical device: `BASE_URL = "http://<your-local-ip>:8000/api/v1/"`
3. Run on emulator or device

---

## Technology Stack

### Backend (Python)
```
FastAPI          - Web framework
Uvicorn          - ASGI server
SQLAlchemy       - ORM for SQLite
Pydantic         - Data validation
google-generativeai - Gemini API
TensorFlow       - ML model inference (.keras)
gTTS             - Text-to-speech
python-multipart - File uploads
```

### Frontend (Android)
```
Kotlin 2.0       - Language
Jetpack Compose  - UI framework
Hilt             - Dependency injection
Retrofit         - REST client
OkHttp           - HTTP + WebSocket
Room             - Local database
CameraX          - Camera API
ExoPlayer        - Audio playback
```

---

## Key Architectural Decisions

### 1. No LangChain/LangGraph
**Decision:** Use Direct Gemini API instead
**Reason:** Simpler, fewer dependencies, easier to debug
**Trade-off:** Less abstraction, manual prompt management

### 2. Server-side ML
**Decision:** Disease detection runs on Python backend
**Reason:** ML team provides .keras model, easier to update
**Trade-off:** Requires network call, but images are small

### 3. On-device STT
**Decision:** Android SpeechRecognizer instead of cloud STT
**Reason:** Free, fast, works offline
**Trade-off:** Less accurate than Google Cloud STT, but sufficient

### 4. SQLite Database
**Decision:** SQLite instead of PostgreSQL
**Reason:** Simpler setup for local development
**Trade-off:** No advanced features, but not needed for demo

### 5. No Authentication
**Decision:** Single hardcoded user for local development
**Reason:** Simplifies development, not deploying to production
**Trade-off:** Not production-ready, but can add later

---

## File Structure Conventions

### Backend Structure
```
aquasense_backend/
├── main.py                      # FastAPI app entry point
├── requirements.txt             # Python dependencies
├── .env                         # Environment variables (not committed)
├── config/
│   ├── __init__.py
│   ├── settings.py              # Pydantic settings
│   └── database.py              # SQLAlchemy setup
├── models/                      # SQLAlchemy ORM models
│   ├── __init__.py
│   ├── user.py
│   ├── tank.py
│   ├── water_quality.py
│   ├── product.py
│   └── order.py
├── schemas/                     # Pydantic schemas (request/response)
│   ├── __init__.py
│   ├── tank.py
│   ├── analysis.py
│   └── voice.py
├── api/
│   └── v1/
│       ├── __init__.py
│       ├── router.py            # API router
│       └── endpoints/
│           ├── tanks.py
│           ├── products.py
│           ├── analysis.py
│           └── voice_agent.py   # WebSocket endpoint
├── services/                    # Business logic
│   ├── __init__.py
│   ├── tank_service.py
│   ├── analysis_service.py
│   └── voice_service.py
├── ai/                          # AI/Gemini integration
│   ├── __init__.py
│   ├── gemini_client.py
│   ├── prompts.py
│   └── session_memory.py
├── ml/                          # ML model
│   ├── __init__.py
│   ├── disease_classifier.py
│   ├── preprocessing.py
│   └── models/
│       └── fish_disease.keras   # Provided by ML team
├── knowledge/                   # Aquaculture knowledge base
│   ├── treatments.json
│   ├── diseases.json
│   └── species_info.json
├── websocket/                   # WebSocket handlers
│   ├── __init__.py
│   ├── handler.py
│   └── message_types.py
└── tests/
```

### Android Structure (Clean Architecture)
```
app/src/main/java/com/parth/aquasense/
├── AquaSenseApp.kt              # Application class
├── MainActivity.kt              # Single activity
├── di/                          # Hilt modules
│   ├── AppModule.kt
│   ├── NetworkModule.kt
│   └── DatabaseModule.kt
├── data/                        # Data layer
│   ├── local/
│   │   ├── dao/
│   │   ├── entity/
│   │   └── AquaSenseDatabase.kt
│   ├── remote/
│   │   ├── api/
│   │   │   ├── TankApi.kt
│   │   │   ├── ProductApi.kt
│   │   │   └── AnalysisApi.kt
│   │   └── websocket/
│   │       └── VoiceAgentWebSocket.kt
│   └── repository/
│       ├── TankRepository.kt
│       └── ProductRepository.kt
├── domain/                      # Domain layer
│   ├── model/
│   │   ├── Tank.kt
│   │   ├── WaterQuality.kt
│   │   └── Product.kt
│   └── usecase/
│       ├── GetTanksUseCase.kt
│       └── AnalyzeTankUseCase.kt
├── presentation/                # UI layer
│   ├── navigation/
│   │   ├── NavGraph.kt
│   │   └── Screen.kt
│   ├── theme/
│   │   ├── Color.kt
│   │   ├── Theme.kt
│   │   └── Type.kt
│   ├── components/
│   │   ├── TankCard.kt
│   │   ├── WaterQualityCard.kt
│   │   ├── StatisticCard.kt
│   │   ├── AlertCard.kt
│   │   ├── QuickActionCard.kt
│   │   └── BottomNavBar.kt
│   └── screens/
│       ├── dashboard/
│       │   ├── DashboardScreen.kt
│       │   ├── DashboardViewModel.kt
│       │   └── DashboardUiState.kt
│       ├── tanklist/
│       │   ├── TankListScreen.kt
│       │   ├── TankListViewModel.kt
│       │   └── TankListUiState.kt
│       ├── tankdetail/
│       │   ├── TankDetailScreen.kt
│       │   ├── TankDetailViewModel.kt
│       │   └── TankDetailUiState.kt
│       ├── tankform/
│       │   ├── TankFormScreen.kt
│       │   ├── TankFormViewModel.kt
│       │   └── TankFormUiState.kt
│       ├── disease/
│       ├── marketplace/
│       └── voiceagent/
└── util/
    ├── AudioManager.kt
    └── Extensions.kt
```

---

## API Conventions

### Base URL
- Local emulator: `http://10.0.2.2:8000/api/v1/`
- Physical device: `http://<your-ip>:8000/api/v1/`

### Endpoint Naming
- Use plural nouns: `/tanks`, `/products`, `/orders`
- Use kebab-case for multi-word: `/water-quality`
- Nested resources: `/tanks/{id}/water-quality`

### HTTP Methods
- `GET` - Retrieve resource(s)
- `POST` - Create resource
- `PUT` - Update entire resource
- `PATCH` - Partial update
- `DELETE` - Remove resource

### Response Format
All responses return JSON with consistent structure:

**Success Response:**
```json
{
  "data": { ... },
  "message": "Success",
  "timestamp": "2025-12-21T10:30:00Z"
}
```

**Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Tank name is required",
    "details": { ... }
  },
  "timestamp": "2025-12-21T10:30:00Z"
}
```

### Status Codes
- `200` - Success
- `201` - Created
- `400` - Bad request
- `404` - Not found
- `422` - Validation error
- `500` - Server error

---

## Database Conventions

### Table Naming
- Use lowercase with underscores: `water_quality_readings`
- Use plural: `tanks`, `products`

### Column Naming
- Use lowercase with underscores: `created_at`, `user_id`
- Foreign keys: `{table}_id` (e.g., `tank_id`)
- Timestamps: `created_at`, `updated_at`, `deleted_at`

### Primary Keys
- Use UUID for all primary keys
- Column name: `id`

### Timestamps
- All tables have `created_at TIMESTAMP`
- Updatable tables have `updated_at TIMESTAMP`

---

## Code Style Guidelines

### Python (Backend)
```python
# Use type hints
def get_tank(tank_id: str) -> Tank:
    pass

# Async for I/O operations
async def fetch_data(url: str) -> dict:
    pass

# Pydantic for validation
class TankCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=255)
    species: List[str]

# Docstrings for functions
def analyze_tank(tank_id: str) -> AnalysisResult:
    """
    Analyze tank health using Gemini API.

    Args:
        tank_id: UUID of the tank

    Returns:
        AnalysisResult with health score and recommendations
    """
```

### Kotlin (Android)
```kotlin
// Use data classes
data class Tank(
    val id: String,
    val name: String,
    val species: List<String>
)

// Use sealed classes for results
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val message: String) : Result<Nothing>()
}

// Composable naming
@Composable
fun TankCard(tank: Tank, modifier: Modifier = Modifier) {
    // ...
}

// ViewModel with StateFlow
class DashboardViewModel @Inject constructor(
    private val getTanksUseCase: GetTanksUseCase
) : ViewModel() {
    private val _uiState = MutableStateFlow<DashboardUiState>(Loading)
    val uiState: StateFlow<DashboardUiState> = _uiState.asStateFlow()
}
```

---

## UI/UX Implementation Patterns

### Material Design 3 - TankListScreen

The TankListScreen implements a clean, compact list layout following Material Design 3 guidelines with custom spacing optimizations.

#### TopAppBar Configuration
```kotlin
TopAppBar(
    title = { Text("My Tanks") },
    windowInsets = WindowInsets(0, 0, 0, 0)  // Remove default insets for compact header
)
```

**Key Points:**
- Uses regular `TopAppBar` (not Medium or Large) for minimal vertical space
- `WindowInsets(0, 0, 0, 0)` removes system window insets for compact appearance
- No scroll behavior - simple, static header
- Title appears immediately below status bar

#### Content Area Padding
```kotlin
PullToRefreshBox(
    modifier = Modifier
        .fillMaxSize()
        .padding(top = paddingValues.calculateTopPadding())  // Only top padding
)
```

**Key Points:**
- Only apply top padding (for TopAppBar)
- Bottom padding removed so content extends to bottom navigation
- Allows full vertical space utilization

#### Card List Layout
```kotlin
LazyColumn(
    modifier = Modifier.fillMaxSize(),
    contentPadding = PaddingValues(horizontal = 16.dp, vertical = 16.dp),  // Consistent spacing
    verticalArrangement = Arrangement.spacedBy(12.dp)
) {
    items(tanks, key = { it.id }) { tank ->
        TankCard(tank = tank, onClick = { ... })
    }
}
```

**Key Points:**
- **Horizontal padding**: 16.dp (standard Material Design spacing)
- **Vertical padding**: 16.dp (matches Dashboard screen for consistency)
- **Card spacing**: 12.dp between cards
- **Key function**: Uses tank.id for efficient recomposition

#### Layout Spacing Summary
```
┌─────────────────────────┐
│ Status Bar              │
├─────────────────────────┤
│ TopAppBar (compact)     │  ← windowInsets = 0
├─────────────────────────┤
│ 16dp │            │16dp │  ← horizontal padding
│      │  TankCard  │     │
│ 16dp │            │16dp │
├──────┴────────────┴─────┤
│      12dp spacing        │  ← verticalArrangement
├──────┬────────────┬─────┤
│ 16dp │  TankCard  │16dp │
│      │            │     │
├──────┴────────────┴─────┤
│ Bottom Navigation       │  ← cards extend to here
└─────────────────────────┘
```

#### Design Rationale

**Why TopAppBar with zero WindowInsets?**
- Minimizes vertical space consumption
- Keeps header compact and unobtrusive
- Users see content immediately

**Why only top padding?**
- Maximizes vertical content area
- Cards visible edge-to-edge vertically
- Better utilization of screen real estate

**Why 16dp horizontal padding?**
- Standard Material Design 3 spacing

**Why 16dp vertical padding?**
- Matches Dashboard screen for consistency
- Provides breathing room at top and bottom
- Better visual hierarchy

---

## Implemented Screens

### Dashboard Screen
**Route:** `dashboard`
**Purpose:** Overview of all tanks with statistics, alerts, and quick actions

**Features:**
- **Overview Statistics**: 4 cards showing total tanks, total fish, tanks needing attention, and health score
- **Recent Alerts**: List of water quality warnings with color-coded severity (Critical/Warning/Info)
- **Quick Actions**: Add Tank, View Tanks, Add Reading buttons
- **Pull-to-refresh**: Swipe down to reload dashboard data
- **Navigation**: Clickable stats navigate to relevant screens, alerts navigate to tank details

**Components Used:**
- `StatisticCard` - Displays icon, value, and label
- `AlertCard` - Shows tank alerts with color-coded severity
- `QuickActionCard` - Action buttons with icons

**Data Source:**
- `GET /api/v1/tanks/dashboard` - Returns aggregated dashboard summary

---

### Tank List Screen
**Route:** `tanks`
**Purpose:** View all tanks in a scrollable list

**Features:**
- **Tank Cards**: List of all tanks with name, species, stock, location, and status
- **Pull-to-refresh**: Swipe down to reload tank list
- **Add Tank FAB**: Floating action button to create new tank
- **Visual Indicators**:
  - Stock progress bar with overstocked warning (red if > capacity)
  - Status badge (Active=green, Inactive=gray, Maintenance=tertiary)
- **Navigation**: Tap card to view tank details

**Components Used:**
- `TankCard` - Displays individual tank summary

**Data Source:**
- `GET /api/v1/tanks` - Returns list of all tanks

---

### Tank Detail Screen
**Route:** `tank/{tankId}`
**Purpose:** View detailed information about a specific tank

**Features:**
- **Tank Information**: Name, status, location, species, capacity, current stock
- **Stock Visualization**: Progress bar showing stock percentage with overstocked warning
- **Water Quality Display**: Latest water quality readings with all parameters
  - Shows pH, temperature, dissolved oxygen, ammonia, nitrite, nitrate, salinity, turbidity
  - Color-coded card (green=safe, red=warnings)
  - Lists specific warnings for out-of-range parameters
- **Actions**: Edit tank, delete tank (with confirmation dialog)
- **Pull-to-refresh**: Swipe down to reload tank data
- **Empty State**: Shows "No Water Quality Data" placeholder if no readings exist

**Components Used:**
- `WaterQualityCard` - Displays all water quality parameters with safety indicators

**Data Sources:**
- `GET /api/v1/tanks/{tankId}` - Returns tank details
- `GET /api/v1/tanks/{tankId}/water-quality/latest` - Returns latest water quality reading

---

### Tank Form Screen
**Route:** `tank/form?tankId={tankId}`
**Purpose:** Create new tank or edit existing tank

**Features:**
- **Dual Mode**: Add mode (tankId=null) or Edit mode (tankId provided)
- **Form Fields**:
  - Tank Name* (required, 1-255 chars)
  - Capacity* (required, > 0, decimal input)
  - Current Stock* (required, ≥ 0, integer input)
  - Species* (required, at least 1, chip-based multi-entry)
  - Location (optional, free text)
  - Status (dropdown: Active, Inactive, Maintenance)
- **Validation**:
  - Real-time field-level error messages
  - Save button disabled until form is valid
  - Client-side validation matches backend rules
- **Species Management**: Add species via text input + button, remove via chip click
- **Auto-navigation**: Returns to previous screen after successful save
- **Loading States**: Shows spinner while loading/saving, disables inputs during save

**Validation Rules:**
| Field | Validation |
|-------|------------|
| Name | Required, 1-255 characters |
| Capacity | Required, > 0, valid decimal |
| Current Stock | Required, ≥ 0, valid integer |
| Species | Required, at least 1 species |
| Location | Optional |
| Status | One of: ACTIVE, INACTIVE, MAINTENANCE |

**Data Sources:**
- `POST /api/v1/tanks` - Create new tank
- `PUT /api/v1/tanks/{tankId}` - Update existing tank
- `GET /api/v1/tanks/{tankId}` - Load tank for editing

---

## Navigation Structure

### Bottom Navigation Bar
**Items:**
1. Dashboard - Home icon, navigates to dashboard
2. Tanks - List icon, navigates to tank list

**Behavior:**
- Clears back stack when switching tabs (clean navigation)
- Saves and restores screen state when re-selecting tabs
- Single instance of each tab (launchSingleTop)

### Navigation Flow
```
Dashboard
├─→ View Tanks (quick action) → Tank List
├─→ Add Tank (quick action) → Tank Form (add mode)
└─→ Tank Alert (click) → Tank Detail

Tank List
├─→ Tank Card (click) → Tank Detail
└─→ FAB (click) → Tank Form (add mode)

Tank Detail
├─→ Edit Button → Tank Form (edit mode)
├─→ Delete Button → Confirmation Dialog → Navigate Back
└─→ Back Button → Navigate Back

Tank Form
├─→ Save Button → Create/Update Tank → Navigate Back
└─→ Back Button → Navigate Back
```

---

## Common Patterns

### UI State Management
All screens follow the same pattern:
```kotlin
sealed class ScreenUiState {
    data object Loading : ScreenUiState()
    data class Success(...) : ScreenUiState()
    data class Error(val message: String) : ScreenUiState()
}
```

### ViewModel Pattern
```kotlin
@HiltViewModel
class ScreenViewModel @Inject constructor(
    private val repository: Repository
) : ViewModel() {
    private val _uiState = MutableStateFlow<ScreenUiState>(Loading)
    val uiState: StateFlow<ScreenUiState> = _uiState.asStateFlow()

    fun loadData() {
        viewModelScope.launch {
            when (val result = repository.getData()) {
                is Result.Success -> _uiState.value = Success(result.data)
                is Result.Error -> _uiState.value = Error(result.message)
            }
        }
    }
}
```

### Pull-to-Refresh
All list screens implement pull-to-refresh:
```kotlin
PullToRefreshBox(
    isRefreshing = state.isRefreshing,
    onRefresh = viewModel::refresh,
    modifier = Modifier.fillMaxSize()
) {
    // Content
}
```

### Navigation Arguments
- **Required arguments**: Path parameters in route (e.g., `tank/{tankId}`)
- **Optional arguments**: Query parameters with defaults (e.g., `tank/form?tankId={tankId}`)
- **Retrieval**: Via SavedStateHandle in ViewModel

---

## Bottom Navigation Fix

### Issue Fixed
When navigating from Dashboard → Tank List via quick action, users couldn't navigate back to Dashboard using the bottom nav bar.

### Root Cause
The `popUpTo` builder block in BottomNavBar was missing `inclusive = true`, causing back stack inconsistency.

### Solution
```kotlin
navController.navigate(item.screen.route) {
    popUpTo(navController.graph.startDestinationRoute ?: Screen.Dashboard.route) {
        saveState = true
        inclusive = true  // ✅ This clears the entire back stack
    }
    launchSingleTop = true
    restoreState = true
}
```

This ensures clean state transitions when switching between bottom nav tabs.

---

## Form Validation Pattern

The Tank Form Screen demonstrates the form validation pattern:

1. **Data Class for Form State**:
```kotlin
data class TankFormData(
    val name: String = "",
    val capacity: String = "",  // String for TextField binding
    val errors: Map<String, String> = emptyMap()
) {
    val isValid: Boolean
        get() = name.isNotBlank() && capacity.isNotBlank() && !hasErrors
}
```

2. **Validation Function**:
```kotlin
private fun validateForm(formData: TankFormData): Map<String, String> {
    val errors = mutableMapOf<String, String>()

    if (formData.name.isBlank()) {
        errors["name"] = "Tank name is required"
    }

    val capacity = formData.capacity.toDoubleOrNull()
    if (capacity == null) {
        errors["capacity"] = "Capacity must be a valid number"
    } else if (capacity <= 0) {
        errors["capacity"] = "Capacity must be greater than 0"
    }

    return errors
}
```

3. **UI Error Display**:
```kotlin
OutlinedTextField(
    value = formData.name,
    onValueChange = { onFieldChange(TankFormField.NAME, it) },
    isError = formData.errors.containsKey("name"),
    supportingText = formData.errors["name"]?.let { { Text(it) } }
)
```

---

## Why 16dp horizontal padding?**
- Standard Material Design 3 spacing
- Provides proper visual breathing room
- Cards feel contained and professional
- Better than edge-to-edge (8dp) which felt too wide

**Why 12dp card spacing?**
- Clear visual separation between cards
- Not too tight (cluttered) or too loose (disconnected)
- Comfortable scanning of list items

---

## Environment Variables

### Backend (.env file)
```bash
# Gemini API
GEMINI_API_KEY=your_gemini_api_key_here

# Database
DATABASE_URL=sqlite:///./aquasense.db

# Server
HOST=0.0.0.0
PORT=8000
DEBUG=True
```

### Android (local.properties - NOT committed)
```properties
sdk.dir=/Users/yourname/Library/Android/sdk
```

---

## Git Conventions

### Commit Message Format
```
<type>(<scope>): <subject>

<body>
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation
- `style` - Formatting
- `refactor` - Code restructuring
- `test` - Adding tests
- `chore` - Maintenance

**Examples:**
```
feat(backend): add tank analysis endpoint
fix(android): resolve WebSocket connection issue
docs: update API documentation
```

### Branch Naming
- `main` - Production-ready code
- `develop` - Development branch
- `feature/tank-management` - Feature branches
- `fix/websocket-connection` - Bug fix branches

---

## Common Tasks

### Adding a New API Endpoint

**Backend:**
1. Define Pydantic schema in `schemas/`
2. Create endpoint in `api/v1/endpoints/`
3. Add to router in `api/v1/router.py`
4. Implement service logic in `services/`
5. Test with curl or Postman

**Android:**
1. Define data class in `domain/model/`
2. Add API interface in `data/remote/api/`
3. Create repository method in `data/repository/`
4. Create use case in `domain/usecase/`
5. Call from ViewModel

### Adding a New Screen

1. Create package in `presentation/screens/`
2. Create `{Name}Screen.kt` composable
3. Create `{Name}ViewModel.kt`
4. Define UI state sealed class
5. Add route to `NavGraph.kt`
6. Test navigation

### Adding Database Model

**Backend:**
1. Create model in `models/`
2. Create schema in `schemas/`
3. Create migration: `alembic revision -m "description"`
4. Apply: `alembic upgrade head`

**Android:**
1. Create entity in `data/local/entity/`
2. Create DAO in `data/local/dao/`
3. Add to database class
4. Increment database version
5. Provide migration strategy

---

## Testing

### Backend Testing
```bash
# Run all tests
pytest

# With coverage
pytest --cov=app tests/

# Single test file
pytest tests/test_tanks.py
```

### Android Testing
```bash
# Unit tests
./gradlew test

# Instrumented tests
./gradlew connectedAndroidTest

# Specific test
./gradlew test --tests TankViewModelTest
```

---

## Debugging

### Backend Debugging
```python
# Add print statements
print(f"Tank data: {tank}")

# Use logging
import logging
logger = logging.getLogger(__name__)
logger.info(f"Processing tank {tank_id}")

# FastAPI debug mode
# In main.py
if __name__ == "__main__":
    import uvicorn
    uvicorn.run("main:app", host="0.0.0.0", port=8000, reload=True)
```

### Android Debugging
```kotlin
// Logcat
Log.d("TankViewModel", "Fetching tanks: ${tanks.size}")

// Timber (recommended)
Timber.d("Tank loaded: ${tank.name}")

// Compose Preview
@Preview(showBackground = true)
@Composable
fun TankCardPreview() {
    TankCard(tank = Tank(...))
}
```

---

## API Documentation

### View Interactive API Docs
Once backend is running:
- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc

### Generate OpenAPI spec
```bash
# Download JSON
curl http://localhost:8000/openapi.json > openapi.json
```

---

## Common Issues & Solutions

### Issue: WebSocket connection fails
**Solution:** Check that:
- Backend is running on correct port
- Android uses correct IP (10.0.2.2 for emulator)
- Firewall allows connection
- WebSocket endpoint exists at `/ws/voice-agent/{session_id}`

### Issue: Gemini API rate limit
**Solution:**
- Free tier: 15 requests/minute
- Add retry logic with exponential backoff
- Cache responses when possible

### Issue: Image upload fails
**Solution:**
- Check file size limit (default 10MB)
- Verify Content-Type is multipart/form-data
- Ensure image is JPEG/PNG
- Backend logs will show detailed error

### Issue: Room migration error
**Solution:**
- Increment database version
- Provide migration or use `fallbackToDestructiveMigration()`
- Clear app data during development

---

## Performance Considerations

### Backend
- Use async/await for I/O operations
- Cache Gemini responses for identical queries
- Lazy load ML model at startup
- Use connection pooling for database

### Android
- Use LazyColumn for long lists
- Cache images with Coil
- Debounce search input
- Use WorkManager for background tasks
- Limit WebSocket reconnection attempts

---

## Security Notes (Local Development)

**Current Setup:**
- No authentication (single user)
- No HTTPS (local HTTP only)
- API key in environment variable
- SQLite database in local file

**For Production (if deploying later):**
- Add JWT authentication
- Use HTTPS with SSL certificate
- Encrypt sensitive data in database
- Rate limit API endpoints
- Validate all user inputs
- Use prepared statements (already done with SQLAlchemy)

---

## Deployment (Future)

**Backend:**
- Use Docker for containerization
- Deploy to Google Cloud Run or AWS ECS
- Use PostgreSQL instead of SQLite
- Add Redis for session storage
- Set up CI/CD with GitHub Actions

**Android:**
- Build signed APK/AAB
- Upload to Google Play Console
- Set up Firebase for analytics
- Configure ProGuard for obfuscation

---

## Resources

**Documentation:**
- FastAPI: https://fastapi.tiangolo.com/
- Jetpack Compose: https://developer.android.com/jetpack/compose
- Gemini API: https://ai.google.dev/docs
- TensorFlow: https://www.tensorflow.org/

**Project Files:**
- `FEATURES.md` - Full feature specifications
- `process_flow.md` - System architecture and data flows
- Architecture plan: `.claude/plans/polished-nibbling-fox.md`

---

## Team Contacts (if applicable)

**Backend Developer:** [Your name]
**Android Developer:** [Your name]
**ML Team:** Provides .keras model for disease detection
**Design:** See FEATURES.md for UI specifications

---

## Implementation Status

### ✅ Completed Features

**Backend:**
- FastAPI server with SQLAlchemy ORM
- Tank CRUD operations (create, read, update, delete)
- Water quality readings management
- Dashboard summary endpoint with alerts
- Database models and schemas
- SQLite database setup

**Android App:**
- ✅ **Dashboard Screen** - Overview statistics, alerts, quick actions
- ✅ **Tank List Screen** - Scrollable list of all tanks with pull-to-refresh
- ✅ **Tank Detail Screen** - Individual tank details with water quality display
- ✅ **Tank Form Screen** - Add/edit tanks with form validation
- ✅ **Bottom Navigation** - Tab navigation between Dashboard and Tanks
- ✅ **Reusable Components** - TankCard, WaterQualityCard, StatisticCard, AlertCard, QuickActionCard

**Navigation:**
- ✅ Bottom navigation bar with tab switching
- ✅ Clean back stack management
- ✅ Proper argument passing between screens
- ✅ Auto-navigation after form submissions

**Architecture:**
- ✅ Clean Architecture (Data → Domain → Presentation)
- ✅ MVVM pattern with ViewModels
- ✅ Repository pattern with error handling
- ✅ State management with StateFlow
- ✅ Dependency injection with Hilt

### 🚧 In Progress / Next Steps

**Priority:**
1. Water Quality Form Screen (add water quality readings)
2. Product Marketplace screens
3. Disease Detection screen with image upload
4. Voice Agent with WebSocket integration
5. Analytics and reporting

See `process_flow.md` and `FEATURES.md` for detailed feature specifications.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Parthvats13) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
