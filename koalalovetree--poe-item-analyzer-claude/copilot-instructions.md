## poe-item-analyzer-claude

> Build a desktop application in Rust that fetches Path of Exile timeless jewels from public stash tabs or file import, analyzes them using pre-computed lookup tables against ALL possible jewel sockets, and ranks them by value based on user-defined desirable modifiers. The architecture must be extensible to support additional item types (cluster jewels, influenced items, etc.) in the future.

# PROJECT: PoE Item Analyzer - Timeless Jewels (v1)

## EXECUTIVE SUMMARY
Build a desktop application in Rust that fetches Path of Exile timeless jewels from public stash tabs or file import, analyzes them using pre-computed lookup tables against ALL possible jewel sockets, and ranks them by value based on user-defined desirable modifiers. The architecture must be extensible to support additional item types (cluster jewels, influenced items, etc.) in the future.

---

## CONTEXT & MOTIVATION

Path of Exile player who needs to analyze multiple timeless jewels efficiently. Manually checking each jewel in online calculators is tedious. This tool automates the process.

### How Timeless Jewels Work
- Each timeless jewel has a **TYPE** (Lethal Pride, Brutal Restraint, Glorious Vanity, Elegant Hubris, Militant Faith)
- Each jewel has a **SEED** (number 100-160000 depending on type, determines mods)
- Each jewel has a **CONQUEROR/VARIANT** name (e.g., "Kaom" for Lethal Pride, "Balbala" for Brutal Restraint)
- When socketed in specific jewel sockets on the passive tree, they modify nearby passive nodes
- The same seed ALWAYS gives the same mods in the same socket location
- Pre-computed lookup tables (LUTs) exist mapping (jewel_type, seed, node_id) → modifier

### Value Assessment Concept
The application should allow users to define which mods they consider valuable by:
- Searching and selecting mods from a large list
- Assigning weights/scores to different mods
- **Analyzing ALL jewel socket locations** (not just pre-selected ones)
- Viewing results per socket location to compare which socket gives best mods

Example: A user might consider "Double Damage", "Onslaught on Hit", or specific attribute bonuses as valuable, and want to find jewels that have multiple instances of these mods when socketed in ANY location, then compare which socket location is best.

---

## KEY REQUIREMENTS CLARIFICATIONS

### 1. Socket Analysis Strategy
**REQUIREMENT:** Analyze each jewel against ALL possible jewel socket locations on the passive tree.

**Implementation:**
- When analyzing a jewel, check it against every valid socket location
- Store results for each socket separately
- Display which socket location produced which mod combinations
- Allow filtering/sorting by socket location in results
- Users can compare results across sockets without re-running analysis

**Example Result Structure:**
```
Jewel: Brutal Restraint #4523
├─ Socket: Far-Left (near Marauder)  - Score: 8.5 - Mods: Onslaught x3, +Str x2
├─ Socket: Top-Left (near Templar)   - Score: 6.2 - Mods: Double Damage x1
├─ Socket: Bottom (near Duelist)     - Score: 4.1 - Mods: +Str x4
├─ Socket: Right (near Ranger)       - Score: 3.8 - Mods: +Dex x3
└─ ...all other socket locations...
```

### 2. Mod Selection Interface
**REQUIREMENT:** Handle hundreds of possible mods with easy search and clear visibility of selections.

**UI Pattern:**
```
┌─────────────────────────────────────────────────┐
│ Configure Valuable Mods                    [X] │
├─────────────────────────────────────────────────┤
│                                                 │
│ ADD MODS:                                       │
│ ┌─────────────────────────────────────────────┐ │
│ │ Search: [double damage___________] 🔍      │ │
│ └─────────────────────────────────────────────┘ │
│                                                 │
│ Available Mods (filtered):                      │
│ ┌─────────────────────────────────────────────┐ │
│ │ ▸ 5% chance to deal Double Damage           │ │
│ │ ▸ 10% chance to deal Double Damage          │ │
│ │ ▸ 4% chance to deal Double Damage           │ │
│ └─────────────────────────────────────────────┘ │
│                                                 │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                 │
│ SELECTED MODS (3):                              │
│ ┌─────────────────────────────────────────────┐ │
│ │ × 5% chance to deal Double Damage   [5.0]   │ │
│ │ × Onslaught on Hit                  [4.5]   │ │
│ │ × +20 to Strength                   [2.0]   │ │
│ └─────────────────────────────────────────────┘ │
│                                                 │
│           [ Clear All ]      [ Save ]  [Cancel] │
└─────────────────────────────────────────────────┘
```

**Features:**
- Search box filters available mods in real-time
- Click to add mod to "Selected Mods" area
- Selected area shows all chosen mods with their weights
- Click × to remove from selected list
- Edit weight values directly in selected area
- Clear visibility of what's currently configured

---

## TECHNICAL REQUIREMENTS

### Language & Framework
- **Language**: Rust (latest stable)
- **GUI Framework**: `egui` (simple, immediate mode, pure Rust)
- **Async Runtime**: `tokio` for async API calls
- **HTTP Client**: `reqwest` for PoE API calls
- **Serialization**: `serde` + `serde_json`
- **Error Handling**: `thiserror` for error types

### Project Structure (Cargo Workspace)
```
poe-item-analyzer/
├── Cargo.toml                    # Workspace root
├── CLAUDE.md                     # This file - project requirements
├── crates/
│   ├── core/                     # Core business logic (no I/O)
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── items/            # Item domain models
│   │       │   ├── mod.rs
│   │       │   ├── timeless_jewel.rs
│   │       │   └── traits.rs     # Item, AnalyzableItem traits
│   │       ├── analyzers/        # Analysis logic
│   │       │   ├── mod.rs
│   │       │   ├── timeless.rs   # TimelessJewelAnalyzer
│   │       │   └── traits.rs     # Analyzer trait
│   │       ├── data/             # Data handling (LUT parsing)
│   │       │   ├── mod.rs
│   │       │   ├── lut.rs        # Binary LUT parser
│   │       │   └── traits.rs     # DataSource trait
│   │       └── scoring/          # Scoring systems
│   │           ├── mod.rs
│   │           └── weighted.rs   # Weighted scoring
│   ├── api/                      # API/Service layer (I/O operations)
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── poe_api/         # PoE API client
│   │       │   ├── mod.rs
│   │       │   ├── leagues.rs   # Fetch active leagues
│   │       │   ├── stash.rs     # Fetch stash tabs
│   │       │   └── models.rs    # API response models
│   │       ├── sources/         # Item sources
│   │       │   ├── mod.rs
│   │       │   ├── public_stash.rs  # Public stash API source
│   │       │   └── file.rs          # File import source
│   │       └── service.rs       # Main orchestration layer
│   └── desktop/                 # Desktop GUI
│       ├── Cargo.toml
│       └── src/
│           ├── main.rs
│           ├── app.rs           # Main app state
│           └── ui/
│               ├── mod.rs
│               ├── timeless_jewels.rs  # Timeless jewel UI
│               └── components/  # Reusable UI components
└── data/                        # External data (gitignored)
    └── timeless_jewels/         # LUT files go here
```

---

## CORE ARCHITECTURE PATTERNS

The architecture follows these key principles:
1. **Separation of Concerns**: Core logic, API, UI are independent
2. **Trait-based Design**: Extensible through traits rather than concrete types
3. **Future-Proof**: Easy to add new item types without modifying existing code
4. **Type-Safe**: Leverage Rust's type system for correctness
5. **Testable**: Each component can be tested in isolation

### 1. Item Trait System

All item types implement common traits:

```rust
// Core trait for all items
pub trait Item: Send + Sync {
    fn id(&self) -> String;          // Unique identifier
    fn item_type(&self) -> &str;     // Type name (e.g., "Timeless Jewel")
    fn name(&self) -> &str;          // Display name
}

// Trait for items that can be analyzed
pub trait AnalyzableItem: Item {
    type ValueMetrics;               // What defines value for this item
    fn raw_data(&self) -> serde_json::Value;  // Raw item data
}
```

**TimelessJewel Implementation:**
- Stores: jewel_type, seed, conqueror
- Implements both Item and AnalyzableItem traits
- ValueMetrics contains score and matched mods per socket

### 2. Analyzer Trait System

Analyzers are generic over item types:

```rust
pub trait Analyzer<T: AnalyzableItem>: Send + Sync {
    type Config;   // Configuration for analysis
    type Result;   // Analysis result type

    // Analyze single item across all socket locations
    fn analyze(&self, item: &T, config: &Self::Config)
        -> Result<Self::Result, AnalysisError>;

    // Analyze batch and rank
    fn analyze_batch(&self, items: &[T], config: &Self::Config)
        -> Result<Vec<RankedResult<Self::Result>>, AnalysisError>;

    // How to compare results for ranking
    fn compare_results(&self, a: &Self::Result, b: &Self::Result)
        -> std::cmp::Ordering;
}
```

**TimelessJewelAnalyzer Implementation:**
- Config includes: valuable_mods (with weights)
- Result includes: Vec<SocketResult> where each SocketResult has score, matched_mods, socket_location
- Uses LUT data to determine mods for each seed at each socket
- Calculates score based on matched valuable mods per socket
- Returns ALL socket results, not just filtered ones

### 3. Analysis Result Structure

```rust
pub struct TimelessJewelAnalysisResult {
    pub jewel: TimelessJewel,
    pub socket_results: Vec<SocketResult>,  // One per socket location
}

pub struct SocketResult {
    pub socket_id: String,
    pub socket_name: String,  // e.g., "Far-Left (near Marauder)"
    pub score: f64,
    pub matched_mods: Vec<MatchedMod>,
    pub all_mods: Vec<String>,  // All mods this jewel gives at this socket
}

pub struct MatchedMod {
    pub mod_text: String,
    pub weight: f64,
    pub count: usize,  // How many times this mod appears
}
```

### 4. Item Source Trait System

Data sources are abstracted:

```rust
pub trait ItemSource<T: Item>: Send + Sync {
    async fn fetch_items(&self) -> Result<Vec<T>, SourceError>;
    async fn validate(&self) -> Result<(), SourceError>;
}
```

**Implementations:**
- **PublicStashSource**: Fetches from PoE public stash API
- **FileSource**: Loads from JSON file

### 5. Service Layer

Orchestrates everything:

```rust
pub struct AnalyzerService<T: AnalyzableItem> {
    sources: HashMap<String, Box<dyn ItemSource<T>>>,
    analyzers: HashMap<String, Box<dyn Analyzer<T>>>,
}
```

The service layer:
- Manages available sources and analyzers
- Coordinates fetching items from sources
- Runs analysis and returns ranked results
- Handles errors and caching

---

## DATA SOURCES & API INTEGRATION

### PoE Official APIs

#### 1. Leagues API
**Endpoint:** `https://api.pathofexile.com/leagues`

**Purpose:** Fetch list of active leagues dynamically

**Response Format:**
```json
[
  {
    "id": "Keepers",
    "realm": "pc",
    "description": "...",
    "startAt": "2025-10-31T20:00:00Z",
    "endAt": null,
    "event": false
  }
]
```

**Filtering:**
- Only PC realm leagues
- Only non-event leagues
- Only currently active (endAt is null)

**Caching Strategy:**
- Cache for session or 1 hour
- Allow manual refresh
- Fallback to hardcoded list if API fails: ["Standard", "Hardcore"]

#### 2. Stash Tabs API
**Endpoint:** `https://www.pathofexile.com/character-window/get-stash-items`

**Parameters:**
- `accountName`: Account name (case-sensitive)
- `league`: League name
- `tabIndex`: Tab number (0-indexed)
- `tabs=1`: Include tab list (optional, for first call)

**Two-Step Process:**
1. First call with `tabs=1` to get list of public tabs
2. Second call with specific `tabIndex` to get items

**Security Note:** This API only works with **public** tabs. Users must:
- Make their stash tab public in-game
- Set high prices (e.g., 999999 divines) to prevent purchases

**No session ID or authentication required** - this keeps the app secure.

### Timeless Jewel Data (LUTs)

**Source:** https://github.com/Regisle/TimelessJewelData

**What it contains:**
- Pre-computed binary lookup tables
- Map of (jewel_type, seed, node_id) → modifier
- Files for each jewel type (Lethal Pride, Brutal Restraint, etc.)
- Node indices and jewel socket mappings
- **ALL valid jewel socket locations** on passive tree

**Format:**
- Binary files (uint8 arrays)
- Efficient random access
- Formula: `array[node_id_INDEX * jewel_seed_Size + jewel_seed_offset] = index_of_Change`

**Required Files:**
- Data files for each jewel type (5 files)
- `Node_Indices.csv` - node ID mappings
- `Jewel_Node_Link.json` - which nodes are in range of which sockets
- `alternate_passive_additions.json` - mod definitions
- `alternate_passive_skills.json` - replacement definitions

**Parsing Strategy:**
- Load binary files into memory at startup
- Create lookup functions for quick queries
- Reference the C# examples in the repo for parsing logic
- Load ALL socket locations for comprehensive analysis

### Data Management System

**IMPORTANT:** LUT data files are large (~200 MB) and must NOT be committed to the repository.

#### Architecture

**Storage Location (Cross-Platform):**
Using Rust's `dirs` crate for proper OS-specific paths:
```
Linux:   ~/.local/share/poe-item-analyzer-claude/data/
Windows: C:\Users\<user>\AppData\Roaming\poe-item-analyzer-claude\data\
macOS:   ~/Library/Application Support/poe-item-analyzer-claude/data/
```

**Repository Structure:**
```
/data/
├── .gitignore           # Ignore all data files (*.bin, *.dat, *.json, *.csv)
├── README.md            # Manual download instructions
└── manifest.json        # Data file metadata and download URLs
```

**manifest.json Format:**
```json
{
  "data_version": "3.27.0",
  "poe_league": "Keepers of the Flame",
  "last_updated": "2025-11-10T12:00:00Z",
  "source": "https://github.com/Regisle/TimelessJewelData",
  "files": [
    {
      "name": "lethal_pride.bin",
      "url": "https://raw.githubusercontent.com/Regisle/TimelessJewelData/master/data/lethal_pride.bin",
      "sha256": "...",
      "size": 45000000,
      "required": true
    },
    {
      "name": "Node_Indices.csv",
      "url": "https://raw.githubusercontent.com/Regisle/TimelessJewelData/master/data/Node_Indices.csv",
      "sha256": "...",
      "size": 125000,
      "required": true
    }
  ]
}
```

#### Download & Update UI

**First-Run Detection:**
- On startup, check if required data files exist
- If missing → show "Data Download Required" dialog
- If present → verify integrity (optional checksum validation)

**Download Dialog:**
```
┌─────────────────────────────────────────────┐
│ Data Download Required            [X]       │
├─────────────────────────────────────────────┤
│ This app needs timeless jewel data          │
│ to analyze items (~200 MB download)         │
│                                             │
│ Source: github.com/Regisle/...              │
│                                             │
│ [ Download & Install Data ]                 │
│                                             │
│ Progress: [████████░░░░] 45%                │
│ Downloading: lethal_pride.bin (45/200 MB)  │
│                                             │
│ Or import manually: [ Browse Local Files ]  │
└─────────────────────────────────────────────┘
```

**Settings/Data Management UI:**
```
┌─────────────────────────────────────────────┐
│ Settings → Data Management                  │
├─────────────────────────────────────────────┤
│                                             │
│ TIMELESS JEWEL DATA                         │
│ ┌─────────────────────────────────────────┐ │
│ │ Status: ✅ Installed (v3.27.0)          │ │
│ │ League: Keepers of the Flame            │ │
│ │ Last updated: 2025-11-10                │ │
│ │ Size: 187 MB                            │ │
│ │ Location: ~/.local/share/poe-item-...  │ │
│ │                                         │ │
│ │ [ Check for Updates ]  [ Re-import ]    │ │
│ └─────────────────────────────────────────┘ │
│                                             │
│ IMPORT SOURCE:                              │
│ ○ GitHub (automatic, recommended)           │
│   Repository: Regisle/TimelessJewelData     │
│                                             │
│ ○ Custom URL                                │
│   [_________________________________]       │
│                                             │
│ ○ Local Files                               │
│   [Browse...] Select data directory or ZIP  │
│                                             │
│ [ Import Now ]                              │
└─────────────────────────────────────────────┘
```

#### Features

**Initial Download:**
- One-click download from GitHub
- Progress bar showing download status
- Validates checksums (SHA256) after download
- Extracts to proper OS-specific location
- Fallback to manual import if download fails

**Update Management:**
- "Check for Updates" compares local version with remote manifest
- Shows changelog/what's new (if available)
- Re-download updated files only
- Keeps old version as backup during update

**Re-import Options:**
1. **From GitHub** (default):
   - Automatically fetches latest data from official repo
   - Validates file integrity
   - Best for most users

2. **Custom URL**:
   - For community mirrors
   - For league-specific data sources
   - Advanced users only

3. **Local Files**:
   - User provides directory or ZIP file
   - Useful for offline environments
   - For testing with custom data

**Validation & Error Handling:**
- Verify file checksums after download
- Test data integrity (try loading sample data)
- Show clear error messages if validation fails
- Rollback to previous version if update fails
- Detailed error logs for troubleshooting

#### Platform-Specific Considerations

**Windows:**
- Handle long file paths (enable long path support)
- Use proper temp directory for downloads
- Antivirus might block downloads → show warning/instructions
- UAC permissions if writing to protected locations

**Linux:**
- Check write permissions to ~/.local/share/
- Support for different desktop environments
- Handle both X11 and Wayland
- Provide fallback to ./data/ if home directory unavailable

**macOS:**
- Handle Gatekeeper restrictions
- Proper bundle structure for app distribution
- Request file system permissions if needed
- Retina/high-DPI display support (handled by egui)

**Fallback Behavior:**
If automatic download fails:
```
❌ Download Failed

Reason: Network timeout / Invalid checksum / etc.

MANUAL INSTALLATION:
1. Visit: https://github.com/Regisle/TimelessJewelData
2. Download the following files:
   • lethal_pride.bin
   • brutal_restraint.bin
   • ...
3. Click "Browse Local Files" and select the folder

[ Retry Download ]  [ Browse Local Files ]  [ Cancel ]
```

#### Implementation Details

**Download Manager (in API crate):**
```rust
pub struct DataDownloader {
    manifest_url: String,
    target_dir: PathBuf,
}

impl DataDownloader {
    pub async fn download_all(&self) -> Result<(), DownloadError>;
    pub async fn check_updates(&self) -> Result<Option<Version>, DownloadError>;
    pub async fn validate_files(&self) -> Result<bool, ValidationError>;
    pub fn import_from_directory(&self, path: PathBuf) -> Result<(), ImportError>;
}
```

**Data Loader (in core crate):**
```rust
pub struct DataLoader {
    data_dir: PathBuf,
}

impl DataLoader {
    pub fn data_exists(&self) -> bool;
    pub fn load_lut_data(&self) -> Result<LutData, LoadError>;
    pub fn get_data_info(&self) -> Option<DataInfo>;
}
```

---

## FEATURE REQUIREMENTS

### Version 1 (MVP)

#### Data Management
- [ ] Detect missing data on startup
- [ ] Show "Download Required" dialog on first run
- [ ] Download LUT data from GitHub with progress bar
- [ ] Validate downloaded files (checksums)
- [ ] Store data in OS-specific app data directory
- [ ] Manual import from local files/directory
- [ ] Settings panel for data management
- [ ] "Check for Updates" functionality
- [ ] "Re-import" button for updating data

#### Item Fetching
- [ ] Fetch active leagues from API (with caching)
- [ ] Fetch public stash tab list for account
- [ ] Fetch items from selected tab
- [ ] Parse timeless jewels from item data
- [ ] Import jewels from JSON file

#### Analysis
- [ ] Load and parse timeless jewel LUT data
- [ ] Load ALL jewel socket locations
- [ ] User can search and select valuable mods
- [ ] User can assign weights to selected mods
- [ ] Calculate score for each jewel at EACH socket location
- [ ] Store results per socket
- [ ] Rank jewels by best socket score (or allow sorting)

#### User Interface
- [ ] Account name input field
- [ ] League dropdown (auto-populated from API)
- [ ] Tab selection dropdown (auto-populated after account fetch)
- [ ] File import option
- [ ] **Searchable mod selector with separate selected area**
- [ ] Display ranked results table
- [ ] **Show results per socket location** (expandable view)
- [ ] Filter/sort by socket location
- [ ] Show matched mods for each jewel-socket combination

#### Configuration
- [ ] Save/load valuable mods configuration
- [ ] Persist window state

### Future Enhancements (Post-v1)

- [ ] Support for other item types (cluster jewels, influenced items)
- [ ] Export results to CSV/JSON
- [ ] Price checking integration (poe.ninja API)
- [ ] Mod search/filter functionality in results
- [ ] Integration with Path of Building imports
- [ ] Highlight best socket per jewel
- [ ] Side-by-side socket comparison view

---

## USER INTERFACE DESIGN

### Main Window Layout

```
┌─────────────────────────────────────────────────────────────┐
│ PoE Item Analyzer                                     [_][□][X] │
├─────────────────────────────────────────────────────────────┤
│  SOURCE                                                     │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ ○ Public Stash Tab                                  │  │
│  │   Account Name: [____________________]              │  │
│  │   League:       [Keepers ▼]  [🔄]                  │  │
│  │   Tab:          [Tab 12: "jewels" ▼]               │  │
│  │                                                      │  │
│  │ ○ Import from File                                  │  │
│  │   [Browse...] selected: jewels.json                 │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  [ Fetch & Analyze ]                                       │
│                                                             │
│  CONFIGURATION                                              │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ Valuable Mods (3 selected): [ Edit Config ]         │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  RESULTS (15 jewels analyzed, showing best socket)         │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ Rank │ Type        │ Seed │ Socket    │ Score │ Mods│  │
│  ├──────┼─────────────┼──────┼───────────┼───────┼─────┤  │
│  │ ▸ 1  │ Brutal R.   │ 4523 │ Far-Left  │ 8.5   │ ... │  │ ← Click to expand
│  │ ▾ 2  │ Lethal P.   │ 7821 │ Top-Left  │ 7.2   │ ... │  │
│  │   └─ Socket: Far-Left      │ Score: 6.1  │ Mods...   │  │ ← Expanded
│  │   └─ Socket: Top-Left      │ Score: 7.2  │ DblDmg x2│  │
│  │   └─ Socket: Bottom        │ Score: 5.3  │ Mods...   │  │
│  │   └─ Socket: Right         │ Score: 4.8  │ Mods...   │  │
│  │  ... │ ...                                            │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  [ Export Results ] [ View Details ]                       │
└─────────────────────────────────────────────────────────────┘
```

---

## IMPLEMENTATION PHASES

### Phase 1: Core Foundation (Week 1)
1. Set up Cargo workspace
2. Implement core traits (Item, AnalyzableItem, Analyzer)
3. Create TimelessJewel model
4. Download and parse LUT data files
5. Load ALL socket location data
6. Basic LUT query functionality for all sockets

### Phase 2: API Integration (Week 1-2)
1. Implement PoE API client
2. League fetching with caching
3. Public stash tab fetching
4. Item parsing (JSON → TimelessJewel)
5. File source implementation

### Phase 3: Analysis Logic (Week 2)
1. Implement TimelessJewelAnalyzer
2. Mod matching logic across all sockets
3. Scoring system per socket
4. Ranking algorithm (by best socket or configurable)
5. Unit tests for analysis

### Phase 4: GUI (Week 2-3)
1. Basic egui window
2. Input forms (account, league, tab)
3. **Searchable mod selector with selected area**
4. Results table with expandable socket views
5. Configuration dialogs
6. State management

### Phase 5: Polish & Testing (Week 3)
1. Error handling and user feedback
2. Configuration persistence
3. Integration testing
4. Performance optimization
5. Documentation

---

## TECHNICAL CHALLENGES & SOLUTIONS

### Challenge 1: Large Binary LUT Files
**Problem:** LUT files are large (hundreds of MB)
**Solution:**
- Load files once at startup
- Keep in memory (Arc<> for sharing)
- Use memory-mapped files if too large
- Consider compression for distribution

### Challenge 2: Async in GUI
**Problem:** egui is not async-native
**Solution:**
- Use channels (tokio::sync::mpsc) for GUI ↔ async communication
- Spawn tokio tasks for API calls
- Update GUI state from channel receivers
- Show loading states during async operations

### Challenge 3: Analyzing ALL Sockets Performance
**Problem:** Analyzing each jewel against all sockets might be slow
**Solution:**
- Pre-load all LUT data into memory
- Parallelize socket analysis (rayon or tokio::spawn)
- Cache socket location data
- Progress bar for batch analysis

### Challenge 4: Type Erasure for Service Layer
**Problem:** Need to store different Analyzer types in HashMap
**Solution:**
- Use trait objects: `Box<dyn Analyzer<T>>`
- OR use enum for known analyzer types
- Consider `Any` for config type erasure if needed

### Challenge 5: Parsing PoE Item JSON
**Problem:** PoE item JSON is complex and varies by item type
**Solution:**
- Use serde with #[serde(flatten)] and enums
- Implement try_from for type-specific parsing
- Log and skip invalid items rather than failing

### Challenge 6: Searchable Mod Selection UI in egui
**Problem:** egui is immediate mode, handling search state can be tricky
**Solution:**
- Store search string in app state
- Filter mods each frame based on search
- Use egui's ScrollArea for large lists
- Store selected mods in Vec/HashSet

---

## DEPENDENCIES (Cargo.toml)

### Workspace Root
```toml
[workspace]
members = ["crates/core", "crates/api", "crates/desktop"]
resolver = "2"
```

### Core Crate
```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
thiserror = "1.0"
```

### API Crate
```toml
[dependencies]
poe-item-analyzer-core = { path = "../core" }
reqwest = { version = "0.11", features = ["json"] }
tokio = { version = "1.0", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
thiserror = "1.0"
async-trait = "0.1"
sha2 = "0.10"  # For checksum validation
dirs = "5.0"   # For OS-specific data directories
```

### Desktop Crate
```toml
[dependencies]
poe-item-analyzer-core = { path = "../core" }
poe-item-analyzer-api = { path = "../api" }
eframe = "0.27"  # egui framework
egui = "0.27"
tokio = { version = "1.0", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
dirs = "5.0"  # For config directory
```

---

## TESTING STRATEGY

### Unit Tests
- Core logic in `core` crate (analyzers, scoring)
- LUT parsing correctness
- Item model validation
- Socket analysis accuracy

### Integration Tests
- API client (use mock responses)
- Source implementations
- Service layer orchestration
- Multi-socket analysis

### Manual Testing
- Real API calls with test account
- Various jewel types and seeds
- Edge cases (empty stash, invalid data)
- Performance with many jewels across all sockets

---

## SECURITY & PRIVACY

### What We DON'T Do
- ❌ Never ask for session IDs
- ❌ Never store account passwords
- ❌ Never transmit sensitive data
- ❌ Never access private stash tabs

### What We Do
- ✅ Only use public APIs
- ✅ Require users to make tabs public manually
- ✅ Store only non-sensitive config locally
- ✅ No telemetry or data collection

### User Responsibilities
Users must:
1. Make their stash tab public in-game settings
2. Price items very high (999999 divines) to prevent sales
3. Understand that their account name and public tab contents are visible

---

## SUCCESS CRITERIA

Version 1 is successful when:
- ✅ User can fetch jewels from public stash or file
- ✅ User can search and select valuable mods with weights
- ✅ App analyzes each jewel against ALL socket locations
- ✅ Results show per-socket breakdown
- ✅ App correctly ranks jewels by score
- ✅ Results are accurate (verified against manual checks)
- ✅ GUI is responsive and intuitive
- ✅ Searchable mod selector works smoothly
- ✅ Configuration persists between sessions
- ✅ Can analyze multiple jewels across all sockets in reasonable time
- ✅ Architecture allows easy addition of new item types

---

## RESOURCES & REFERENCES

### PoE APIs
- Official API Docs: https://www.pathofexile.com/developer/docs
- Leagues endpoint: https://api.pathofexile.com/leagues
- Stash endpoint: https://www.pathofexile.com/character-window/get-stash-items

### Timeless Jewel Data
- LUT Data Repo: https://github.com/Regisle/TimelessJewelData
- Parsing Examples: https://github.com/Regisle/TimelessJewelData/tree/generator
- Online Calculator: https://vilsol.github.io/timeless-jewels/tree
- PoB Integration: https://github.com/PathOfBuildingCommunity/PathOfBuilding

### Rust Resources
- egui Examples: https://github.com/emilk/egui/tree/master/examples
- Tokio Docs: https://tokio.rs/
- Serde Guide: https://serde.rs/

---

## NOTES FOR IMPLEMENTATION

### Quick Start Steps
1. Create Cargo workspace
2. Download LUT data files from repo
3. Start with `core` crate - implement traits
4. Implement TimelessJewel and basic analyzer
5. Build simple CLI to test analysis
6. Add API integration
7. Finally add GUI

### Development Tips
- Test LUT parsing with known seeds first
- Mock API responses during development
- Use `cargo watch` for fast iteration
- Profile with `cargo flamegraph` if slow
- Test socket analysis with sample jewels

### Common Pitfalls to Avoid
- Don't block GUI thread with API calls (use async properly)
- Don't load all LUT data per query (load once, cache)
- Don't parse item JSON strictly (PoE API changes often)
- Don't forget error handling for network failures
- Don't re-analyze for each socket (batch the operation)

---

## CHANGELOG

### 2025-11-16 - Initial Requirements
- Created project specification
- Defined architecture and tech stack

### 2025-11-16 - Requirement Updates (Iteration 1)
- **CHANGED:** Analyze ALL socket locations instead of user-selected sockets
- **CHANGED:** Store and display results per socket location
- **ADDED:** Searchable mod selector UI with separate selected mods area
- **ADDED:** Support for filtering/comparing results by socket

### 2025-11-16 - Data Management System (Iteration 2)
- **ADDED:** Comprehensive data management system for LUT files
- **ADDED:** In-app download functionality with progress tracking
- **ADDED:** Cross-platform storage using OS-specific app data directories
- **ADDED:** Update/re-import functionality for new PoE patches
- **ADDED:** Multiple import sources (GitHub, custom URL, local files)
- **ADDED:** Checksum validation (SHA256) for data integrity
- **ADDED:** Fallback to manual import if auto-download fails
- **DECISION:** LUT data NOT committed to repository (too large ~200MB)
- **DECISION:** Support all platforms (Windows, Linux, macOS) with platform-specific handling

### 2025-11-17 - Parser Implementation & Testing UI (Phase 3 Complete)
- **IMPLEMENTED:** Complete PoB binary LUT parser
  - Zlib decompression for .zip files (discovered they're zlib-compressed, not ZIP archives)
  - Binary data parsing with seed/node index lookup
  - Support for all jewel types (LethalPride, BrutalRestraint, ElegantHubris, MilitantFaith)
  - Lua file parsing for NodeIndexMapping and LegionPassives
- **IMPLEMENTED:** Automatic download system
  - One-click download from PathOfBuilding GitHub repository
  - Real-time progress tracking with file-by-file updates
  - Multi-threaded async downloads with tokio runtime
  - Auto-detection of existing data files (skip re-download)
- **IMPLEMENTED:** Parser testing UI
  - Progress bars showing download and parse status
  - Auto-parse existing data on startup
  - Detailed results display (node counts, modifiers, jewel data)
  - Collapsible parse log for debugging
  - Re-download button for updating data
  - Fixed egui widget ID collisions
- **TECHNICAL DECISIONS:**
  - Using thread-spawned tokio runtime for async operations (eframe compatibility)
  - Data stored in `/tmp/poe-item-analyzer-test/` for easy access
  - Zlib (flate2) instead of ZIP crate for decompression
  - Real-time UI updates via mpsc channels
- **WORKING:** End-to-end data pipeline
  - Download → Decompress → Parse → Display
  - All jewel types successfully parsed
  - Thousands of seeds with modifier data loaded

### 2025-11-18 - Glorious Vanity Support (Phase 3 Enhancement)
- **FIXED:** Missing Glorious Vanity jewel type (was only showing 4 of 5 jewel types)
- **IMPLEMENTED:** Glorious Vanity special binary format parser
  - Multi-part file download (GloriousVanity.zip.part0-part4, ~22MB total)
  - Automatic part concatenation into single GloriousVanity.zip file
  - Header section parser (nodeCount × seedRange bytes indicating data lengths)
  - Variable-length data section parser (stats and rolls format)
  - Support for 4 valid patterns: 1+1, 1+2, 3+3, 4+4 (stats+rolls)
- **TECHNICAL DETAILS:**
  - Glorious Vanity uses completely different format than other 4 jewel types
  - Header: 1,678 nodes × 7,901 seeds = ~13.2 MB
  - Data: Variable-length byte arrays with stat IDs and roll values
  - Format: All stats first, then all rolls (not interleaved)
  - File split due to GitHub file size limits (5MB per part)
- **RESEARCH:**
  - Investigated PathOfBuilding source code (DataLegionLookUpTableHelper.lua)
  - Analyzed TimelessJewelData repository documentation
  - Understood seed range behavior: "Seeds with data" shows seeds with ≥1 modified node
- **RESULT:** All 5 timeless jewel types now fully supported ✅
  - LethalPride, BrutalRestraint, ElegantHubris, MilitantFaith (simple format)
  - GloriousVanity (complex header+variable-length format)

### Status: Phase 3 Complete ✅
**Next Steps:**
- Phase 4: Implement actual analysis logic (analyzer trait, scoring system)
- Phase 5: Build main GUI for jewel analysis workflow
- Phase 6: Add search/filter for valuable mods

---

END OF PROJECT REQUIREMENTS

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KoalaLoveTree)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KoalaLoveTree)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
