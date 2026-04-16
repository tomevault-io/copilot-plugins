## cartwise

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CartWise iOS Application - Architecture Overview

## Executive Summary

CartWise is a comprehensive iOS shopping application built with SwiftUI that implements a sophisticated MVVM+Coordinator pattern with Core Data persistence. The app helps users manage shopping lists, compare prices across multiple stores, and participate in a social shopping community with reputation-based gamification.

---

## Development Commands

### Building and Running

**Build for iPhone 16 Simulator:**
```bash
xcodebuild -project CartWise.xcodeproj -scheme CartWise -destination "platform=iOS Simulator,name=iPhone 16" clean build
```

**Rebuild and Launch Simulator (Automated):**
```bash
./rebuild_simulator.sh
```
Note: The script is configured for a specific derived data path. You may need to adjust the `DERIVED_DATA_PATH` variable.

**Build using Xcode:**
- Open `CartWise.xcodeproj` in Xcode
- Select target device (iPhone 16 recommended)
- Press `Cmd + R` to build and run

### Testing

**Run Unit Tests:**
```bash
xcodebuild test -project CartWise.xcodeproj -scheme CartWise -destination "platform=iOS Simulator,name=iPhone 16"
```

**Run Tests in Xcode:**
- Press `Cmd + U` to run all tests
- Tests are located in `CartWiseTests/` directory

### Simulator Management

**List Available Simulators:**
```bash
xcrun simctl list devices
```

**Boot Specific Simulator:**
```bash
xcrun simctl boot "iPhone 16"
```

**Reset Simulator (Clear All Data):**
```bash
xcrun simctl erase all
```

**Install App on Booted Simulator:**
```bash
xcrun simctl install booted /path/to/CartWise.app
```

### Configuration

**Bundle Identifier:** `cs467.CartWise` (or `cs467.CartWise.brenna` for specific builds)

**MealMe API Key Setup:**
The app requires a MealMe API key for product image fetching:
1. Set environment variable: `MEALME_API_KEY=your_key_here`
2. Key is stored securely in Keychain via `KeychainHelper`
3. See `Config/APIKeys.example.swift` for reference
4. API key is gitignored for security

---

## 1. Overall Architecture Pattern

### MVVM with Coordinator Navigation

The application uses a **hybrid MVVM+Coordinator Pattern**:

- **ViewModels**: Handle business logic and state management (ProductViewModel, AuthViewModel, SocialFeedViewModel)
- **Coordinators**: Manage navigation and flow (AppCoordinator + 5 child coordinators for each tab)
- **Views**: SwiftUI views that bind to ViewModels and Coordinators
- **Services/Repository**: Data persistence and network integration

### Architecture Flow

```
CartWiseApp (Entry Point)
    ↓
AppCoordinator (Root Navigation)
    ├── AuthViewModel + LoginView/SignUpView
    └── Tab-based Navigation (when logged in)
        ├── YourListView → ShoppingListCoordinator
        ├── SearchItemsView → SearchItemsCoordinator
        ├── AddItemsView → AddItemsCoordinator
        ├── SocialFeedView → SocialFeedCoordinator (owns SocialFeedViewModel)
        └── MyProfileView → MyProfileCoordinator
```

**Key Principles**:
- Each tab has its own dedicated coordinator for managing feature-specific navigation
- Coordinators are lazily initialized only when their tab is first accessed
- ProductViewModel is shared across coordinators; SocialFeedViewModel is isolated within SocialFeedCoordinator
- All coordinators are cleaned up on logout to prevent memory leaks

---

## 2. Core Data Models and Schema

### Entity Relationship Diagram

The app uses **Core Data** with a model called `ProductModel.xcdatamodel` containing these entities:

#### Primary Entities

**GroceryItem** (Shopping List Items)
- `id: String` (UUID)
- `productName: String`
- `brand: String`
- `category: String` (ProductCategory enum)
- `barcode: String`
- `isOnSale: Bool`
- `isInShoppingList: Bool` (Soft-delete mechanism)
- `isCompleted: Bool` (For checking off items)
- `isFavorite: Bool` (For favorite items)
- `createdAt: Date`, `updatedAt: Date`, `lastUpdated: Date`
- **Relationships**:
  - `productImage: ProductImage?` (One-to-one)
  - `prices: NSSet<GroceryItemPrice>` (One-to-many)
  - `locations: NSSet<Location>` (Many-to-many)
  - `tags: NSSet<Tag>` (Many-to-many)
  - `experiences: NSSet<ShoppingExperience>` (One-to-many social feed entries)

**GroceryItemPrice** (Price Points Across Stores)
- `id: String`
- `price: Double`
- `currency: String` (Default: "USD")
- `store: String` (Store name for quick reference)
- `createdAt: Date`, `lastUpdated: Date`, `updatedBy: String`
- **Relationships**:
  - `groceryItem: GroceryItem` (Many-to-one)
  - `location: Location` (Many-to-one)

**Location** (Physical Store Locations)
- `id: String`
- `name: String` (Store name)
- `address: String`
- `city: String`, `state: String`, `zipCode: String`
- `isDefault: Bool`, `favorited: Bool`
- `createdAt: Date`, `updatedAt: Date`
- **Relationships**:
  - `user: UserEntity?` (Many-to-one)
  - `prices: NSSet<GroceryItemPrice>` (One-to-many)
  - `groceryItems: NSSet<GroceryItem>` (Many-to-many)
  - `experiences: NSSet<ShoppingExperience>` (One-to-many)

**ProductImage** (Product Images)
- `id: String`
- `imageURL: String?` (Remote URL from MealMe API)
- `imageData: Data?` (Cached image binary data)
- `createdAt: Date`, `updatedAt: Date`
- **Relationships**:
  - `groceryItem: GroceryItem?` (One-to-one)

**Tag** (Product Categories/Filters)
- `id: String`
- `name: String` (e.g., "Organic", "On Sale", "Dairy")
- `color: String` (Hex color for UI)
- `createdAt: Date`, `updatedAt: Date`
- **Relationships**:
  - `products: NSSet<GroceryItem>` (Many-to-many)

**UserEntity** (User Accounts)
- `id: String`
- `username: String` (Unique identifier)
- `password: String` (SHA256 hashed)
- `updates: Int32` (Count for reputation system)
- `level: String` (Reputation level)
- `createdAt: Date`
- **Relationships**:
  - `locations: NSSet<Location>` (One-to-many user's stores)
  - `experiences: NSSet<ShoppingExperience>` (One-to-many)
  - `comments: NSSet<UserComment>` (One-to-many)

**ShoppingExperience** (Social Feed Entries)
- `id: String`
- `comment: String` (User comment)
- `rating: Int16` (1-5 star rating)
- `type: String` (price_update, store_review, product_review, new_product, general)
- `createdAt: Date`
- **Relationships**:
  - `user: UserEntity?` (Many-to-one)
  - `groceryItem: GroceryItem?` (Many-to-one)
  - `location: Location?` (Many-to-one)
  - `comments: NSSet<UserComment>` (One-to-many nested comments)

**UserComment** (Nested Comments on Experiences)
- `id: String`
- `comment: String`
- `rating: Int16`
- `createdAt: Date`, `updatedAt: Date`
- **Relationships**:
  - `user: UserEntity?` (Many-to-one)
  - `experience: ShoppingExperience?` (Many-to-one)

### Key Design Patterns

1. **Soft-delete Pattern**: `isInShoppingList` bool flags items without permanent deletion
2. **Multi-location Pricing**: `GroceryItemPrice` entities enable price comparison across stores
3. **Tag System**: Flexible tagging for organizing and filtering products
4. **Social Graph**: ShoppingExperience + UserComment enables community features

---

## 3. Service Layer Organization

### CoreDataStack (Actor-based Safe Persistence)

**Location**: `/CartWise/Model/CoreDataStack.swift`

```swift
@actor CoreDataStack {
    static let shared = CoreDataStack()
    lazy var persistentContainer: NSPersistentContainer
    func save() async throws
    func performBackgroundTask<T>(_ block: @escaping (NSManagedObjectContext) throws -> T) async throws -> T
}
```

**Responsibilities**:
- Manages NSPersistentContainer for "ProductModel"
- Automatic migration with NSMigratePersistentStoresAutomaticallyOption
- Thread-safe background operations using actor model
- Sample data seeding for demo/testing
- Initialization of Tag seed data

### CoreDataContainer (Protocol-based Data Access)

**Location**: `/CartWise/Model/CoreDataContainer.swift`

Implements `CoreDataContainerProtocol` with methods for:
- Product CRUD operations
- Shopping list management
- Favorite products
- Price comparisons
- Tag management
- Product image caching

### ProductRepository (Service Facade)

**Location**: `/CartWise/Repository/Repository.swift`

```swift
protocol ProductRepositoryProtocol: Sendable {
    func fetchAllProducts() async throws -> [GroceryItem]
    func fetchListProducts() async throws -> [GroceryItem]
    func createProduct(...) async throws -> GroceryItem
    func updateProductWithPrice(...) async throws
    func getLocalPriceComparison(for: [GroceryItem]) async throws -> LocalPriceComparisonResult
    // ... tag and image methods
}
```

**Pattern**: Facade pattern that delegates to CoreDataContainer

### NetworkService (API Integration)

**Location**: `/CartWise/Repository/APIService.swift`

```swift
protocol NetworkServiceProtocol: Sendable {
    func searchProductsOnMealMe(by query: String) async throws -> [APIProduct]
    func setMealMeAPIKey(_ apiKey: String) -> Bool
    func hasMealMeAPIKey() -> Bool
}

final class NetworkService: NetworkServiceProtocol {
    // MealMe API integration for product/image search
    // Exponential backoff retry logic
    // Keychain-based API key management
}
```

**Key Features**:
- Integrates with MealMe API for product images
- Retry mechanism with exponential backoff
- Rate limit handling (429 status codes)
- API key stored securely in Keychain

### ImageService (Image Fetching and Caching)

**Location**: `/CartWise/Repository/ImageService.swift`

```swift
protocol ImageServiceProtocol {
    func fetchImageURL(for: String, brand: String?, category: String?) async throws -> String?
    func loadImage(from: URL) async throws -> UIImage?
}
```

**Workflow**:
1. ProductViewModel requests image for product name/brand/category
2. ImageService queries MealMe API for search results
3. Returns first result's image URL
4. Downloaded image data saved to ProductImage entity

### ReputationManager (Gamification Service)

**Location**: `/CartWise/Model/ReputationManager.swift`

```swift
class ReputationManager: ObservableObject {
    static let shared = ReputationManager()
    func updateUserReputation(userId: String) async
    func getUserReputation(userId: String) async -> (updates: Int, level: String)?
}
```

Integrates with `ReputationSystem` to track and promote shopper levels based on contribution count.

---

## 4. Coordinator Hierarchy and Navigation Structure

### AppCoordinator (Root Coordinator)

**Location**: `/CartWise/Navigation/AppCoordinator.swift`

```swift
@MainActor
class AppCoordinator: ObservableObject {
    @Published var selectedTab: TabItem = .yourList
    @Published var showSplash = true
    @AppStorage("isLoggedIn") private var isLoggedIn: Bool = false

    // Child coordinators (lazily initialized)
    @Published var shoppingListCoordinator: ShoppingListCoordinator?
    @Published var searchItemsCoordinator: SearchItemsCoordinator?
    @Published var addItemsCoordinator: AddItemsCoordinator?
    @Published var socialFeedCoordinator: SocialFeedCoordinator?
    @Published var myProfileCoordinator: MyProfileCoordinator?

    private let productViewModel: ProductViewModel

    func selectTab(_ tab: TabItem)
    func logout()
    func hideSplash()

    // Lazy coordinator getters
    func getShoppingListCoordinator() -> ShoppingListCoordinator
    func getSearchItemsCoordinator() -> SearchItemsCoordinator
    func getAddItemsCoordinator() -> AddItemsCoordinator
    func getSocialFeedCoordinator() -> SocialFeedCoordinator
    func getMyProfileCoordinator() -> MyProfileCoordinator
}

enum TabItem: String, CaseIterable {
    case yourList = "Your List"
    case searchItems = "Search Items"
    case addItems = "Add Items"
    case socialFeed = "Social Feed"
    case myProfile = "My Profile"
}
```

**Navigation Flow**:
1. Splash screen (2.5 seconds)
2. Authentication check via `@AppStorage("isLoggedIn")`
3. If logged in → TabView with 5 main tabs, each with its own coordinator
4. If not logged in → LoginView
5. On logout → All coordinators cleaned up via `cleanupCoordinators()`

**Key Features**:
- **Lazy Initialization**: Child coordinators only created when their tab is accessed
- **Memory Management**: All coordinators reset on logout
- **Single ProductViewModel**: Shared across coordinators that need it
- **Isolated ViewModels**: SocialFeedViewModel stays within SocialFeedCoordinator

### Child Coordinators (Feature-level)

#### 1. ShoppingListCoordinator

**Location**: `/CartWise/Navigation/ShoppingListCoordinator.swift`

```swift
@MainActor
class ShoppingListCoordinator: ObservableObject {
    @Published var showingAddProductModal = false
    @Published var showingRatingPrompt = false
    @Published var showingShareExperience = false
    @Published var showingDuplicateAlert = false
    @Published var showingCheckAllConfirmation = false
    @Published var currentPriceComparison: PriceComparison?
    @Published var duplicateProductName = ""

    private let productViewModel: ProductViewModel
}
```

**Responsibilities**:
- Add product modal management
- Price comparison display
- Rating prompts and share experience flows
- Duplicate product alerts
- Check-all confirmation dialogs

#### 2. SearchItemsCoordinator

**Location**: `/CartWise/Navigation/SearchItemsCoordinator.swift`

```swift
@MainActor
class SearchItemsCoordinator: ObservableObject {
    @Published var showingTagPicker = false
    @Published var selectedTag: Tag?
    @Published var selectedProduct: GroceryItem?
    @Published var showingProductDetail = false

    private let productViewModel: ProductViewModel
}
```

**Responsibilities**:
- Tag picker modal for filtering products
- Tag selection and clearing
- Product detail navigation
- Search results coordination

#### 3. AddItemsCoordinator

**Location**: `/CartWise/Navigation/AddItemsCoordinator.swift`

```swift
@MainActor
class AddItemsCoordinator: ObservableObject {
    @Published var showingBarcodeConfirmation = false
    @Published var showingCategoryPicker = false
    @Published var showingLocationPicker = false
    @Published var showingTagPicker = false
    @Published var showingCamera = false
    @Published var showingError = false

    // Pending barcode scan data
    @Published var pendingBarcode: String = ""
    @Published var pendingProductName: String = ""
    @Published var pendingCompany: String = ""
    @Published var pendingPrice: String = ""
    @Published var pendingCategory: ProductCategory = .none
    @Published var pendingIsOnSale: Bool = false
    @Published var pendingLocation: Location?
    @Published var selectedTags: [Tag] = []
    @Published var addToShoppingList = false
    @Published var isExistingProduct = false

    private let productViewModel: ProductViewModel
}
```

**Responsibilities**:
- Barcode scanning flow management
- Barcode confirmation modal with full product details
- Category, location, and tag picker modals
- Camera state management
- Error alert handling
- Pending scan data lifecycle (reset after completion)

#### 4. SocialFeedCoordinator

**Location**: `/CartWise/Navigation/SocialFeedCoordinator.swift`

```swift
@MainActor
class SocialFeedCoordinator: ObservableObject {
    @Published var showingAddExperience = false
    @Published var showingExperienceDetail = false
    @Published var showingProductPicker = false
    @Published var showingLocationPicker = false
    @Published var selectedExperience: ShoppingExperience?
    @Published var selectedProduct: GroceryItem?
    @Published var selectedLocation: Location?

    let socialFeedViewModel: SocialFeedViewModel  // Owns its own ViewModel
}
```

**Responsibilities**:
- Add experience modal management
- Experience detail view navigation
- Product and location picker modals
- Social feed entry selection
- **Owns SocialFeedViewModel** (isolated from AppCoordinator)

**Important**: This coordinator creates its own `SocialFeedViewModel` internally, keeping social feed logic encapsulated and separate from global state.

#### 5. MyProfileCoordinator

**Location**: `/CartWise/Navigation/MyProfileCoordinator.swift`

```swift
@MainActor
class MyProfileCoordinator: ObservableObject {
    @Published var showingAddLocation = false
    @Published var showingAvatarPicker = false
    @Published var selectedTab: ProfileTab = .favorites
    @Published var currentUsername: String = ""
    @Published var isLoadingUser: Bool = true

    enum ProfileTab: String, CaseIterable {
        case favorites = "Favorites"
        case locations = "Locations"
        case reputation = "Reputation"
    }

    private let productViewModel: ProductViewModel
}
```

**Responsibilities**:
- Add location modal
- Avatar picker modal
- Profile tab selection (Favorites/Locations/Reputation)
- Username and loading state management

### AppCoordinatorView (Coordinator's View)

Acts as the bridge between SwiftUI navigation and the coordinator:

```swift
struct AppCoordinatorView: View {
    @ObservedObject var coordinator: AppCoordinator
    @AppStorage("isLoggedIn") private var isLoggedIn: Bool = false
    @EnvironmentObject var productViewModel: ProductViewModel

    var body: some View {
        ZStack {
            if isLoggedIn {
                TabView(selection: $coordinator.selectedTab) {
                    YourListView()
                        .environmentObject(coordinator.getShoppingListCoordinator())
                    SearchItemsView()
                        .environmentObject(coordinator.getSearchItemsCoordinator())
                    AddItemsView(availableTags: productViewModel.tags)
                        .environmentObject(coordinator.getAddItemsCoordinator())
                    SocialFeedView()
                        .environmentObject(coordinator.getSocialFeedCoordinator())
                    MyProfileView()
                        .environmentObject(coordinator.getMyProfileCoordinator())
                }
            } else {
                LoginView()
            }

            if coordinator.showSplash {
                SplashScreenView()
            }
        }
    }
}
```

**Pattern**: Each tab view receives its dedicated coordinator via `.environmentObject()`, enabling clean separation of navigation concerns.

---

## 5. ViewModel Organization

### ProductViewModel (Primary Business Logic)

**Location**: `/CartWise/ViewModel/ProductViewModel.swift`

```swift
@MainActor
final class ProductViewModel: ObservableObject {
    @Published var products: [GroceryItem] = []
    @Published var favoriteProducts: [GroceryItem] = []
    @Published var recentProducts: [GroceryItem] = []
    @Published var priceComparison: PriceComparison?
    @Published var isLoadingPriceComparison = false
    @Published var tags: [Tag] = []
    @Published var locations: [Location] = []
    
    private let repository: ProductRepositoryProtocol
    private let imageService: ImageServiceProtocol
}
```

**Key Methods**:
- `loadShoppingListProducts()` - Fetches items marked isInShoppingList
- `createProductForShoppingList()` - New product creation with duplicate checking
- `updateProductPrice()` - Updates price and creates social feed entry
- `loadLocalPriceComparison()` - Compares prices across stores (85% availability threshold)
- `toggleProductCompletion()`, `toggleAllProductsCompletion()`
- `fetchImagesForProducts()` - Lazy-loads product images from MealMe API
- `addProductToFavorites()`, `removeProductFromFavorites()`
- Tag management: `loadTags()`, `addTagsToProduct()`, `replaceTagsForProduct()`
- Location management: `loadLocations()`

**Design Pattern**: Quiet/Loud operations
- "Quiet" versions don't reload full lists (prevents visible UI flicker)
- "Loud" versions refresh state for significant changes

### AuthViewModel (Authentication)

**Location**: `/CartWise/ViewModel/AuthViewModel.swift`

```swift
class AuthViewModel: ObservableObject {
    @Published var user: User?
    @Published var error: String?
    @Published var isLoading = false
    
    func signUp(username: String, password: String) async
    func login(username: String, password: String) async
    func logout()
    private func hashPassword(_ password: String) -> String // SHA256
}
```

- Handles user registration and login
- Stores current username in UserDefaults for social feed
- Password hashing with SHA256 via CryptoKit

### SocialFeedViewModel (Community Features)

**Location**: `/CartWise/ViewModel/SocialFeedViewModel.swift`

```swift
@MainActor
class SocialFeedViewModel: ObservableObject {
    @Published var experiences: [ShoppingExperience] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    func loadExperiences()
    func createExperience(comment:, rating:, type:, groceryItem:, location:, user:)
    func createComment(comment:, rating:, experience:, user:)
}
```

- Manages ShoppingExperience entries (social feed posts)
- Context-aware object conversion (for Core Data thread safety)
- Nested comment support via UserComment

---

## 6. View Organization and Major Screens

### Tab-Based Navigation Structure

```
YourListView/
├── YourListView.swift (Main coordinator)
├── ShoppingListCard.swift (Item list with actions)
├── ShoppingListItemRow.swift (Individual item row)
├── SmartAddProductModal.swift (Modal for adding new items)
├── RatingComponents.swift (Rating UI components)
├── SearchResultsSection.swift (Search within modal)
├── AmazonPriceComponents.swift (Price display)
├── CategoryPickerView.swift (Category selection)

SearchItemsView/
├── SearchItemsView.swift
├── CategoryItemsView.swift (Browse by category)

AddItemsView/
├── AddItemsView.swift
├── PhotoCameraView.swift (Barcode scanning)
├── CameraView.swift (Camera integration)

SocialFeedView/
├── SocialFeedView.swift
├── ShareExperienceView.swift (Create feed entry)

MyProfileView/
├── MyProfileView.swift
├── ReputationCardView.swift (User level display)
├── FavoriteItemsView.swift (Favorite products)
├── LocationsSectionView.swift (User's stored locations)

Auth/
├── LoginView.swift
├── SignUpView.swift

Shared/
├── PriceComparisonView.swift (Multi-store price display)
├── ProductImageView.swift (Image display/caching)
├── LocationPickerView.swift (Store selection)
├── AddLocationView.swift (Create new location)
├── SplashScreenView.swift (Launch screen)
```

### Major View Patterns

1. **ShoppingListCard**: Manages list display, editing mode, batch operations
2. **PriceComparisonView**: Shows top 3 stores with 85%+ availability
3. **SmartAddProductModal**: Multi-step product creation (name, brand, category, price)
4. **LocationPickerView**: Store selection with favorites/defaults
5. **ProductImageView**: Lazy-loads and caches product images from URLs

---

## 7. API Integration Patterns

### MealMe API Integration

**Endpoint**: `https://api.mealme.ai/search`

**Query Parameters**: `?query=<product_name> <brand> <category>`

**Purpose**: Search for product images (not actually from MealMe's grocery database, but used as image source)

**Response Structure**:
```swift
struct MealMeSearchResponse: Codable {
    let results: [MealMeItem]
}

struct MealMeItem: Codable {
    let title: String?
    let name: String?
    let price: String?
    let currency: String?
    let imageUrl: String?
    let image: String?
    let link: String?
    let url: String?
}
```

Maps to `APIProduct`:
```swift
struct APIProduct: Codable {
    let name: String
    let price: String
    let currency: String
    let customerReview: String
    let customerReviewCount: String
    let shippingMessage: String
    let amazonLink: String
    let image: String
    let boughtInfo: String
}
```

### Retry Strategy

- **Max Retries**: 3 attempts
- **Backoff**: Exponential with jitter (±20%)
- **Initial Delay**: 0.6 seconds
- **Retryable Errors**: 408, 425, 429, 5xx, transient network errors
- **Non-retryable**: 404 (not found), invalid credentials

### Keychain Integration

API keys stored securely in Keychain (not in code):

```swift
class KeychainHelper {
    func saveMealMeAPIKey(_ apiKey: String) -> Bool
    func loadMealMeAPIKey() -> String?
}
```

Key set via:
1. Environment variable at app launch: `MEALME_API_KEY`
2. Manual via NetworkService.setMealMeAPIKey()

---

## 8. Price Comparison Logic

### Local Price Comparison Algorithm

**File**: ProductViewModel.loadLocalPriceComparison()

**Process**:
1. Fetch all shopping list items
2. Query all available stores from GroceryItemPrice entities
3. For each store, calculate total price:
   - Sum prices of available items
   - Track availability percentage
4. Filter stores: Only include if 85%+ items available
5. Sort by total price (ascending)
6. Return top 3 stores

**Data Models**:
```swift
struct LocalStorePrice: Codable, Sendable {
    let store: String
    let totalPrice: Double
    let currency: String
    let availableItems: Int
    let unavailableItems: Int
    let itemPrices: [String: Double]
    let itemShoppers: [String: String]?
}

struct PriceComparison: Codable, Sendable {
    let storePrices: [StorePrice]
    let bestStore: String?
    let bestTotalPrice: Double
    let bestCurrency: String
    let totalItems: Int
    let availableItems: Int
}
```

**Threshold**: 85% availability minimum (prevents poor recommendations)

---

## 9. Unique Architectural Decisions

### 1. **Actor-based CoreDataStack**

Uses Swift's `actor` model for thread-safe Core Data operations:
```swift
@actor CoreDataStack {
    func performBackgroundTask<T>(...) async throws -> T
}
```

Benefits:
- Eliminates data races
- Compile-time safety checks
- Reduces deadlock risks

### 2. **Protocol-based Repository**

All data access abstracted behind `ProductRepositoryProtocol`:
```swift
protocol ProductRepositoryProtocol: Sendable
```

Benefits:
- Easy mocking for tests
- Swappable implementations
- Clear dependency injection

### 3. **Soft-delete Pattern**

Products never hard-deleted from Core Data:
- `isInShoppingList` flag controls visibility
- Maintains price history and social context
- Enables "undo" capabilities

### 4. **Reputation-based Gamification**

User level system based on contributions:
```
New Shopper (0) → Regular (10) → Smart (25) → Expert (50) → Master (100) → Legendary (200)
```

Tracks via `UserEntity.updates` counter and `ReputationSystem` enum

### 5. **Lazy Image Loading with Caching**

Two-tier image storage:
1. **imageURL**: Remote URL (can refetch anytime)
2. **imageData**: Local cached binary (reduces API calls)

Images fetched asynchronously without blocking UI

### 6. **Multi-context Core Data Operations**

Uses background contexts for heavy operations:
```swift
try await coreDataStack.performBackgroundTask { context in
    // Heavy fetches/writes
}
```

Main thread ViewContext only for UI updates

### 7. **Coordinator Pattern with Tab Navigation**

Hybrid approach:
- **Root**: AppCoordinator manages TabView + authentication
- **Feature**: ShoppingListCoordinator manages modal flow
- **Views**: ShoppingListView uses @EnvironmentObject for coordinator access

### 8. **Location-based Price Storage**

Decoupled locations from product prices via `GroceryItemPrice.location`:
- Users can add custom locations
- Same product tracked across user's favorite stores
- Location metadata (address, city, state) preserved

---

## 10. Data Flow Examples

### Adding a Product to Shopping List

```
User Input (SmartAddProductModal)
    ↓
ProductViewModel.createProductForShoppingList()
    ↓
Check isDuplicateProduct() [quiet search]
    ↓
Repository.createProduct()
    ↓
CoreDataContainer.createProduct()
    ↓
Create GroceryItem + ProductImage + GroceryItemPrice
    ↓
Save to Core Data (background context)
    ↓
ImageService.fetchImageURL() [async, no blocking]
    ↓
URLSession download image data
    ↓
Repository.saveProductImage()
    ↓
Update ProductImage.imageData
    ↓
ProductViewModel.objectWillChange.send() [force UI update]
    ↓
View re-renders with image
```

### Price Comparison Flow

```
User taps "Price Comparison" button
    ↓
ProductViewModel.loadLocalPriceComparison()
    ↓
Repository.getLocalPriceComparison(shoppingList)
    ↓
getAllStores() [distinct store names from all prices]
    ↓
For each store:
    - For each shopping list item:
        - Find GroceryItemPrice where store matches
        - Sum valid prices (>0, valid location)
    - Calculate availability %
    - Filter if <85% availability
    ↓
Sort by total price (ascending)
    ↓
Return top 3 as LocalPriceComparisonResult
    ↓
Convert to PriceComparison model
    ↓
Publish to ProductViewModel.priceComparison @Published
    ↓
PriceComparisonView observes and renders
```

### Social Feed Entry Creation

```
User shares shopping experience
    ↓
ShareExperienceView captures:
    - Comment text
    - Rating (1-5)
    - Associated store/product
    ↓
SocialFeedViewModel.createExperience()
    ↓
Create ShoppingExperience in main context
    ↓
Link to current UserEntity, GroceryItem, Location
    ↓
Save context
    ↓
Reload experiences via fetchRequest
    ↓
SocialFeedView re-renders with new entry
    ↓
ProductViewModel.createSocialFeedEntryForPriceUpdate()
    [Automatically creates feed entry when prices updated]
```

---

## Summary Tables

### Core Dependencies

| Layer | Component | Responsibility |
|-------|-----------|-----------------|
| **App** | CartWiseApp | DI, CoreDataStack, ProductViewModel setup |
| **Navigation** | AppCoordinator, ShoppingListCoordinator | Tab/modal state, flow control |
| **ViewModels** | ProductViewModel, AuthViewModel, SocialFeedViewModel | Business logic, state |
| **Services** | ProductRepository, NetworkService, ImageService, ReputationManager | External integration |
| **Persistence** | CoreDataStack, CoreDataContainer | Core Data operations |
| **Models** | GroceryItem, Location, GroceryItemPrice, Tag, etc. | Data entities |
| **Views** | 20+ SwiftUI files organized by feature | UI presentation |

### Key Async Patterns

| Operation | Type | Context |
|-----------|------|---------|
| Product fetch | Background task | CoreDataContainer |
| Image download | URLSession | ImageService |
| Price comparison | Repository logic | ProductViewModel (MainActor) |
| Auth operations | Core Data save | AuthViewModel (MainActor) |

### File Organization

```
CartWise/
├── CartWiseApp.swift (Entry point, DI setup)
├── Navigation/ (Coordinators - 6 files)
│   ├── AppCoordinator.swift (Root coordinator with lazy child initialization)
│   ├── ShoppingListCoordinator.swift (Your List tab)
│   ├── SearchItemsCoordinator.swift (Search Items tab)
│   ├── AddItemsCoordinator.swift (Add Items tab)
│   ├── SocialFeedCoordinator.swift (Social Feed tab - owns SocialFeedViewModel)
│   └── MyProfileCoordinator.swift (My Profile tab)
├── ViewModel/ (Business logic)
│   ├── ProductViewModel.swift (Shared across coordinators)
│   ├── AuthViewModel.swift
│   └── SocialFeedViewModel.swift (Owned by SocialFeedCoordinator)
├── Views/ (SwiftUI screens)
│   ├── YourListView/
│   ├── SearchItems/
│   ├── AddItems/
│   ├── SocialFeed/
│   ├── MyProfile/
│   ├── Auth/
│   └── Shared/
├── Model/ (Data layer)
│   ├── CoreDataStack.swift
│   ├── CoreDataContainer.swift
│   ├── GroceryItem+CoreDataProperties.swift
│   ├── Location+CoreDataProperties.swift
│   ├── ... (other entities)
│   ├── Product.swift (API models)
│   ├── User.swift
│   ├── ReputationSystem.swift
│   ├── ReputationManager.swift
│   └── ProductModel.xcdatamodeld/
├── Repository/ (Service layer)
│   ├── Repository.swift
│   ├── APIService.swift (NetworkService)
│   └── ImageService.swift
├── Config/
│   ├── APIKeys.swift
│   └── APIKeys.example.swift
├── Resources/ (Assets, colors, fonts)
│   ├── AppColors.swift
│   ├── AppFonts.swift
│   └── Assets.xcassets
├── Utilities/
│   ├── KeychainHelper.swift
│   ├── DesignSystem.swift
│   └── CameraViewController.swift
└── Config/ (Configuration)
    ├── APIKeys.swift (Environment-based)
    └── APIKey.swift
```

---

## Development Guidelines

### Working with Core Data

**Threading Rules:**
- Core Data operations use actor-isolated `CoreDataStack`
- Always use `@MainActor` for ViewModels that publish UI state
- Background operations go through `coreDataStack.performBackgroundTask`
- Never pass managed objects between threads/contexts

**Soft-Delete Pattern:**
- Products are never hard-deleted from Core Data
- Use `isInShoppingList` boolean flag to show/hide items
- This preserves price history and social feed context
- When "removing" a product, set `isInShoppingList = false`

**Creating New Products:**
```swift
// Always check for duplicates first
let isDuplicate = await viewModel.isDuplicateProduct(name: productName)
if isDuplicate {
    // Handle duplicate case
}
// Then create
await viewModel.createProductForShoppingList(...)
```

### Working with Coordinators

**Coordinator Pattern Implementation:**
- AppCoordinator manages root navigation and owns 5 child coordinators
- Each tab has its own coordinator: ShoppingListCoordinator, SearchItemsCoordinator, AddItemsCoordinator, SocialFeedCoordinator, MyProfileCoordinator
- Coordinators are lazily initialized via getter methods (e.g., `getShoppingListCoordinator()`)
- All coordinators are cleaned up on logout via `cleanupCoordinators()`
- Views access their coordinator via `@EnvironmentObject`

**Adding New Navigation:**
1. Add `@Published` modal state to the appropriate coordinator
2. Add show/hide methods to manage the modal state
3. Add `.sheet()` or `.fullScreenCover()` modifier in the view bound to coordinator state
4. Remember to add new coordinators to `cleanupCoordinators()` method

**ViewModel Ownership:**
- **Shared**: ProductViewModel is owned by CartWiseApp and passed to coordinators that need it
- **Isolated**: SocialFeedViewModel is owned by SocialFeedCoordinator and not shared
- **Pattern**: Create ViewModels within coordinators if they're only used by that feature; share at app level if used across multiple features

### Price Comparison Logic

**Important Thresholds:**
- **85% availability minimum**: Stores must have prices for 85%+ of shopping list items
- Only top 3 stores are shown in comparison results
- Missing prices (price == 0 or nil location) are excluded

**Adding New Store Support:**
- Stores are dynamic based on user-entered data
- No hardcoded store list
- Store names come from `GroceryItemPrice.store` attribute

### Image Handling

**Two-Tier Caching:**
1. `ProductImage.imageURL`: Remote URL (can refetch)
2. `ProductImage.imageData`: Binary cache (reduces API calls)

**Fetching Images:**
```swift
// Images fetch asynchronously and don't block UI
await viewModel.fetchImagesForProducts()
// Updates happen via @Published properties triggering view refresh
```

### Reputation System

**Level Progression:**
```
New Shopper (0) → Regular (10) → Smart (25) →
Expert (50) → Master (100) → Legendary (200+)
```

**Triggering Updates:**
- Price updates increment `UserEntity.updates` counter
- ReputationManager calculates level based on update count
- Social feed entries show user's current level

### ViewModels: Quiet vs Loud Operations

**Quiet Operations:**
- Don't reload full product lists
- Use for internal checks (e.g., `isDuplicateProduct()`)
- Prevent UI flicker during background operations

**Loud Operations:**
- Reload lists and publish to views
- Use for user-initiated actions
- Examples: `loadShoppingListProducts()`, `createProductForShoppingList()`

### API Integration Best Practices

**MealMe API:**
- Used exclusively for product image search
- Retry logic with exponential backoff (3 attempts)
- Rate limit handling for 429 responses
- API key stored in Keychain (never committed to git)

**Error Handling:**
```swift
do {
    let result = try await networkService.searchProductsOnMealMe(by: query)
} catch {
    // Display user-friendly error via ViewModel's errorMessage
    errorMessage = "Failed to fetch images: \(error.localizedDescription)"
}
```

### Testing Strategies

**Protocol-Based Testing:**
- Repository and NetworkService are protocol-based
- Create mock implementations for unit tests
- Example: `MockProductRepository: ProductRepositoryProtocol`

**Sample Data:**
- Use CoreDataStack's `loadSampleData()` for testing
- Sample data includes 25 products, 5 locations, social feed entries
- Available via "Load Sample Data" button in MyProfileView

### Common Pitfalls to Avoid

1. **Threading Issues**: Never update UI from background threads. Use `@MainActor` on ViewModels.
2. **Hard Deletes**: Don't delete GroceryItems permanently; use soft-delete pattern.
3. **Missing Price Validation**: Always check `price > 0` and `location != nil` for valid prices.
4. **Image Blocking**: Don't await image fetches during product creation; let them happen asynchronously.
5. **Coordinator Bypass**: Don't navigate directly from Views; always go through Coordinators.
6. **API Key Exposure**: Never hardcode API keys; use environment variables and Keychain.
7. **Coordinator Initialization**: Don't create coordinators directly in views; use AppCoordinator's lazy getters.
8. **ViewModel Leakage**: Don't pass ViewModels between coordinators; only share ProductViewModel when needed.
9. **Missing Cleanup**: Always add new coordinators to `cleanupCoordinators()` to prevent memory leaks on logout.

---

## Notes

- **Deployment**: Supports CI/CD with environment variable `MEALME_API_KEY`
- **Data Safety**: Automatic Core Data migration, orphaned record cleanup
- **Performance**: Lazy loading, quiet/loud operations, background context usage
- **Testing**: Protocol-based architecture enables easy mocking
- **Extensibility**: Coordinator pattern allows easy addition of new features/coordinators
- **Main Branch**: Default branch for pull requests (check git status for current main branch name)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sergelents) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
