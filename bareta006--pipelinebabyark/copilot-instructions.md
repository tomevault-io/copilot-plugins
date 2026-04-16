## pipelinebabyark

> The Multistep Product Template (`product.pdp_multistep_v1.json`) is a custom Shopify product page template that provides a guided, step-by-step customization experience for complex products with multiple options, variants, accessories, and smart features. The template transforms the standard product page into an immersive fullscreen experience with 5 distinct steps.

# Multistep Product Template Documentation

## Overview

The Multistep Product Template (`product.pdp_multistep_v1.json`) is a custom Shopify product page template that provides a guided, step-by-step customization experience for complex products with multiple options, variants, accessories, and smart features. The template transforms the standard product page into an immersive fullscreen experience with 5 distinct steps.

## Template Structure

### File Location
- **Template**: `/templates/product.pdp_multistep_v1.json`
- **Main Form Snippet**: `/snippets/product-multistep-form.liquid`
- **Step Snippets**: `/snippets/product-multistep-step1.liquid` through `step5.liquid`
- **JavaScript**: `/assets/product-multistep.js`
- **CSS**: `/assets/product-multistep.css`
- **Hero Section**: `/sections/section-multistep-hero.liquid`

### Template Configuration

The template consists of two main sections:

1. **Section Liquid Custom** (`section_liquid_custom`):
   - Contains critical CSS for fullscreen mode
   - Hides default product media containers
   - Manages fullscreen body state
   - Controls header/footer visibility during customization

2. **Main Product Section** (`main`):
   - Type: `product`
   - Layout: `none@columns` (no default product layout)
   - Contains single block: `multistep_form`
   - Renders `product-multistep-form` snippet

## The 5-Step Flow

### Step 1: Product Hero / Introduction
- **Purpose**: Initial product showcase and feature discovery
- **Snippet**: `product-multistep-step1.liquid`
- **Features**:
  - Product title, cutline, and price display
  - Hero image with interactive feature points (plus icons)
  - Feature list with modal popups
  - "Customize" button to start customization
- **Navigation**: Clicking "Customize" or URL parameter `?step=2` advances to Step 2
- **Fullscreen**: Not active (normal page view)
- **Dynamic Price Display**:
  - **CRITICAL**: Price is calculated in **Liquid on initial render** to prevent flash of incorrect price (FOUC)
  - Scans all **available variants only** to find the lowest price variant
  - Displays: `"{lowest_price}$ {compare_at_price}$"` (with strikethrough on compare_at if exists)
  - Strikethrough price styling: `font-weight: 400; font-size: 0.7em;` (smaller and lighter)
  - Or: `"{lowest_price}$"` if no compare_at_price
  - Liquid logic: Loops through `product.variants`, filters by `variant.available`, finds variant with lowest `variant.price`
  - JavaScript `updateStep1Price()` method exists as fallback but should not be needed since Liquid renders correctly
  - **NO VISIBLE PRICE CHANGE**: Price renders correctly from server-side, no client-side update flash

### Step 2: Color & Shell Selection
- **Purpose**: Select fabric color and shell color options
- **Snippet**: `product-multistep-step2.liquid`
- **Features**:
  - Dual swatch selection (color + shell color)
  - Dynamic image slider filtered by selections
  - Real-time variant matching and price updates
  - Image filtering based on alt text patterns (`shell_color / fabric_color`)
- **Validation**: Both color and shell color must be selected to proceed
- **Variant Selection**: Partial variant matching (color + shell, no smart option yet)
- **Fullscreen**: Active (hides header, fullscreen mode)

### Step 3: Smart Option Selection
- **Purpose**: Choose between smart and non-smart product versions
- **Snippet**: `product-multistep-step3.liquid`
- **Features**:
  - "Add Smart Capabilities" button
  - "Skip" option for non-smart version
  - **Dynamic Upgrade Price Display**: Calculates price difference between smart and classic variants for selected color combination
  - Final variant selection (color + shell + smart option)
- **Variant Selection**: Complete variant matching with all three options
- **Fullscreen**: Active
- **Dynamic Upgrade Price Calculation**:
  - Finds all variants matching the selected shell color + fabric color from step 2
  - Identifies lowest price variant (classic) and highest price variant (smart) for that color combo
  - Calculates: `smart_price - classic_price = upgrade_difference`
  - Calculates: `smart_compare_at_price - classic_price = strikethrough_difference` (if smart has compare_at)
  - Displays: `"{upgrade_difference}$ {strikethrough_difference}$"` (with strikethrough on second price if compare_at exists)
  - Strikethrough price styling: `font-weight: 400; font-size: 0.7em;` (smaller and lighter)
  - Or: `"{upgrade_difference}$"` if no compare_at_price on smart variant
  - Updates automatically when step 3 is shown and when colors change in step 2 (if already on step 3)
  - Method: `updateStep3UpgradePrice()`

### Step 4: Accessories Selection
- **Purpose**: Browse and add optional accessories
- **Snippet**: `product-multistep-step4.liquid`
- **Features**:
  - Accessory collection display
  - Checkbox selection with auto-uncheck after 2 seconds
  - Variant options for accessories (if applicable)
  - Image updates based on accessory variant selection
  - **Auto-Initialize Variants**: When step 4 is shown, automatically selects first available variant for each accessory and updates images to match
    - Method: `initializeAccessoryVariants()` called in `showStep(4)` wrapped in `requestAnimationFrame()`
    - **SIMPLE SOLUTION**: Finds the first `.swatch__button` in each `.accessory-item`, finds the `.swatch__label` inside it, and clicks the label
    - **CRITICAL**: Clicks the `.swatch__label` element (not the radio input) - this is what users click and what triggers events properly
    - Finds by: `item.querySelector(".swatch__button").querySelector(".swatch__label")`
    - No need to check for `:checked` or `data-variant-available` - just click the first swatch button's label
    - Clicking the label triggers existing `change` event handlers which automatically call `updateAccessoryImage()` and `updateAccessoryVariant()`
    - Uses existing, tested code path instead of manual updates
    - Ensures images match selected variants on first load
  - 20% discount applied to accessories
- **Accessories Collection**: Configured via `data-accessories-collection` attribute (default: "accessories")
- **Cart Addition**: Accessories added separately with delivery properties
- **Fullscreen**: Active

### Step 5: Order Summary & Checkout
- **Purpose**: Review order and proceed to checkout
- **Snippet**: `product-multistep-step5.liquid`
- **Features**:
  - Complete order summary with product image
  - Selected accessories with discounted pricing
  - Subtotal, savings, and total calculations
  - Delivery date/time display
  - "Upgrade to Smart" option (if non-smart selected)
  - "Go to Checkout" button (desktop and mobile versions)
  - **Checkout Button Implementation**:
    - **CRITICAL**: Two buttons exist - one with `desktop--only` class (line 15) and one with `mobile--only` class (line 52)
    - Both have `data-checkout-btn` attribute
    - Event listener uses `querySelectorAll()` to attach to BOTH buttons (not just first one)
    - This ensures mobile button works (desktop button is hidden on mobile, mobile button is hidden on desktop)
  - **Affirm Payment Messaging**: Dynamic price display based on final total (product + discounted accessories)
- **Cart Addition**: All items added to cart before showing summary
- **Fullscreen**: Active

## JavaScript Architecture

### ProductMultiStep Class

**Location**: `/assets/product-multistep.js`

**Initialization**:
- Automatically initializes on `DOMContentLoaded`
- Finds container via `[data-multistep-container]` selector
- Loads product JSON from `[data-product-json]` script tag
- Calls `initializeCartStateFromCart()` to initialize JSON state from cart

**Key Properties**:
- `currentStep`: Current step number (1-5)
- `totalSteps`: Total number of steps (5)
- `selectedVariant`: Final selected product variant
- `selectedColor`: Selected fabric color
- `selectedShellColor`: Selected shell color
- `selectedSmartOption`: Smart or non-smart selection
- `selectedAccessories`: Array of selected accessories
- `productData`: Complete product JSON object
- `allSliderSlides`: Original slider slides for filtering
- `sectionsHidden`: Flag to prevent recursive section hiding
- `sliderInitialized`: Flag to prevent multiple slider initializations
- `sliderEventHandlers`: Object storing event handler references to prevent duplicate attachments
- `variantImageMap`: **Variant image lookup map** - `{ variantId: imageUrl }` built from `productData.variants` for quick image lookup
- `cartState`: **JSON state object** - Single source of truth for cart state
  - `mainProductAdded`: Boolean - whether main product was added to cart
  - `mainProductVariantId`: Number - variant ID of main product in cart
  - `mainProductProperties`: Object - delivery properties for main product
  - `accessories`: Array - accessories in cart state
    - Each item: `{variantId, quantity, properties, title, price, image}`

**Core Methods**:

1. **Step Navigation**:
   - `showStep(stepNumber)`: Shows/hides steps, manages fullscreen state
   - `handleNextStep(nextStep, btn)`: Validates and advances steps
     - **CRITICAL**: When going from step 4 to step 5:
       - **PRESERVE & ADD LOGIC**: Always adds the selected variant to cart (preserves existing items)
       - Checks if selected variant is already in cart
       - If NOT in cart: Adds selected variant (preserves all existing items)
       - If already in cart: Updates cartState to reflect it
       - **E-commerce standard**: If cart has variant A and user selects variant B, cart should have BOTH
       - Calls `updateCartStateFromAccessories()` to update JSON state
       - Calls `syncCartToState()` to sync cart to match JSON
       - Calls `showStep(5)` AFTER sync finishes
       - `renderOrderSummary()` is called automatically when step 5 is shown
   - `updateProgress()`: Updates progress dots visualization

2. **Variant Selection**:
   - `selectPartialVariant()`: Matches color + shell (Step 2)
   - `selectFinalVariant()`: Matches color + shell + smart option (Step 3)
   - `updateImages(variant)`: Updates product images via custom event
   - `updatePrice(variant)`: Updates price display

3. **Color Availability**:
   - `updateColorAvailability()`: Enables/disables color options based on shell selection
   - Checks variant availability for each color/shell combination
   - Shows "Sold Out" indicators for unavailable combinations
   - **Variant Hiding**: If variant has `custom.hidevariant` metafield = true, completely hides the fabric color option (not just marks as sold out)
   - **Metafield**: `variant.metafields.custom.hidevariant.value` (boolean) - included in product JSON

4. **Image Slider**:
   - `initializeSlider()`: Sets up touch/mouse drag navigation
   - **Performance**: Prevents multiple initializations with `sliderInitialized` flag
   - **Event Handlers**: Stored in `sliderEventHandlers` object to prevent duplicate attachments
   - **Throttling**: `mousemove` events throttled using `requestAnimationFrame` to prevent performance issues
   - **Guards**: Uses `dataset.listenerAttached` checks to ensure listeners are only added once
   - `filterSliderImages()`: Filters slides based on selected colors
   - `goToSlide(index)`: Navigates to specific slide
   - `updateSliderArrows()`: Shows/hides navigation arrows

5. **Accessories Management**:
   - `attachAccessoryListeners()`: Sets up accessory selection handlers
   - `addAccessory(variantId, title, price, image, productId, productHandle, variantOptions)`: Adds accessory to selection
     - Stores `productId` for base upgrades
     - Stores `variantOptions` array (extracted from selected variant option elements in DOM)
     - Variant options are extracted from `[data-variant-option]` elements when checkbox is clicked
   - `removeAccessory(variantId)`: Removes accessory from selection
   - `updateAccessoryVariant(item)`: Updates accessory variant selection
     - When variant changes, extracts variant options from selected option element and stores them
   - `updateAccessoryImage(item, changedOption)`: Updates accessory image based on variant selection
     - Checks `data-variant-images` custom map first (if exists on accessory item)
     - Falls back to `data-variant-image` attribute on variant option element
   - `getAccessoryVariantImage(variantId)`: Gets variant-specific image from DOM for summary display
     - Looks up variant option in step 4 DOM by variant ID
     - Uses same logic as `updateAccessoryImage()`: custom map → variant attribute → null
     - Used in `renderOrderSummary()` to show correct variant images (fixes Smart Base image issue)
   - `initializeAccessoryVariants()`: Auto-selects first available variant for each accessory when step 4 loads
     - **SIMPLE SOLUTION**: Finds the first `.swatch__button` in each `.accessory-item`, finds the `.swatch__label` inside it, and clicks the label
     - **CRITICAL**: Clicks the `.swatch__label` element (not the radio input) - this is what users click and what triggers events properly
     - Finds by: `item.querySelector(".swatch__button").querySelector(".swatch__label")`
     - No need to check for `:checked` or `data-variant-available` - just click the first swatch button's label
     - Clicking the label triggers existing `change` event handlers which automatically call `updateAccessoryImage()` and `updateAccessoryVariant()`
     - Uses existing, tested code path instead of manual updates - more reliable and simpler
     - Ensures images match selected variants on first load
   - **Accessory Data Structure**: `{ id, title, price, image, quantity, productId, productHandle, variantOptions: [] }`
     - `variantOptions`: Array of variant option strings (e.g., ["Red", "Large", "Smart"])
     - Stored when accessory is added, extracted from DOM variant option elements

6. **Cart Operations**:
   - **JSON State System (cartState)** - **SINGLE SOURCE OF TRUTH**:
     - `cartState`: JSON object tracking what should be in cart
       - `mainProductAdded`: Boolean - whether main product was added
       - `mainProductVariantId`: Number - variant ID of main product
       - `mainProductProperties`: Object - delivery properties for main product
       - `accessories`: Array - `[{variantId, quantity, properties, title, price, image}]`
     - `initializeCartStateFromCart()`: Initializes `cartState` from cart on page load
       - Called in `init()` method
       - If cart has items, populates `cartState` from cart
       - If cart is empty, starts with fresh empty state
     - `updateCartStateFromAccessories()`: Updates `cartState.accessories` from `selectedAccessories`
       - Called when accessories are clicked (`addAccessory()` / `removeAccessory()`)
       - Maps `selectedAccessories` to `cartState.accessories` format
       - Includes properties from `getAccessoryDeliveryProperties()`
       - **Preserves variant options**: Stores `variantOptions` from `selectedAccessories` in `cartState.accessories`
       - **Preserves accessories from cart**: When initializing from cart, extracts variant options from cart item title (if title contains " - " separator)
       - `cartState.accessories` structure: `{ variantId, quantity, properties, title, price, image, variantOptions: [] }`
     - `syncCartToState()`: Syncs cart to match `cartState` JSON
       - **CRITICAL**: This is the ONLY method that modifies cart
       - Called when going to step 5 (after updating `cartState`)
       - **PRESERVE & ADD LOGIC (Main Product)**:
         - **CRITICAL**: Always preserves existing items in cart, adds selected variant IN ADDITION
         - **E-commerce standard**: If cart has variant A and user selects variant B, cart should have BOTH
         - Checks if selected variant (`cartState.mainProductVariantId`) is already in cart
         - If NOT in cart: Adds selected variant with quantity 1 (preserves all existing items)
         - If already in cart: Preserves it, no action needed
         - **NEVER removes existing variants** - only adds new selection
         - This ensures users can have multiple variants of the same product in cart
         - **Page Load Behavior**: If page loads with items already in cart, they are preserved
         - **Multiple Variants**: Cart can contain multiple variants of the main product simultaneously
         - **Summary Display**: All main product variants in cart are displayed separately in Step 5 summary
       - **Cart Match Check**: 
         - Checks if cart already matches `cartState` before syncing
         - Main product match: Checks if selected variant exists in cart (any quantity > 0)
         - Accessories match: Checks exact quantities for each accessory variant
         - If cart matches, skips entire sync (prevents unnecessary cart operations)
       - Syncs accessories: removes items not in `cartState`, adds/updates items from `cartState`
       - Ensures selected variant is in cart (preserves existing items)
     - `propertiesMatch(props1, props2)`: Helper to compare property objects
       - Normalizes properties (removes null/empty values)
       - Returns true if properties match or both are empty
   - `addToCart(variantId, quantity, properties)`: Adds single item to cart
   - `getCart()`: Fetches current cart state from `/cart.js`
   - `isItemInCart(cart, variantId, properties)`: Checks if an item with matching variant ID and properties already exists in cart
   - `getCartItem(cart, variantId, properties)`: Gets the full cart item object (including key, quantity, etc.)
     - Returns null if item not found
     - Matches by variant ID + properties
   - `updateCartItemQuantity(lineItemKey, quantity)`: Updates cart item quantity using `/cart/change.js`
     - Uses Shopify's cart change API to set exact quantity
     - Dispatches `theme:cart:change` event after update
   - **Accessory Quantity Logic**: 
     - **CRITICAL**: Quantity is based EXACTLY on how many times user clicked the accessory checkbox/button
     - Quantity is stored in `cartState.accessories[].quantity`
     - When syncing, cart is updated to match `cartState.accessories` exactly
     - This ensures cart always reflects the exact number of clicks, NOT how many times step 5 was visited
   - **Upgrade to Smart Compatibility**:
     - When upgrading bases to smart, updates `cartState.accessories[]` with new variant ID
     - Then calls `syncCartToState()` to sync cart
     - Properties are preserved from cart item
   - `getDeliveryProperties()`: Extracts delivery date/time from inputs or metafields
   - `getAccessoryDeliveryProperties(variantId)`: Gets delivery info for accessories

7. **Order Summary**:
   - `renderOrderSummary()`: Generates HTML for Step 5 summary (async)
   - **CRITICAL**: Reads from actual cart (not just `cartState` JSON) to display ALL items
   - Called AFTER `syncCartToState()` finishes (in `handleNextStep()`)
   - Fetches cart to get all items and display them
   - **Main Product Display**:
     - **CRITICAL**: Displays ALL main product variants from cart (not just `cartState.mainProductVariantId`)
     - Loops through ALL cart items where `product_id === this.productData.id`
     - Each variant displayed separately with its own quantity picker, image, price, and variant info
     - This ensures if cart has variant A and variant B, both are shown in summary
     - Uses variant image map for each variant's image
     - Extracts variant options (color, shell, smart) from `variantForDisplay` for each variant
     - **Smart Option Detection**: Loops through `option1`, `option2`, `option3` to find the one containing "smart" (case-insensitive)
       - The option containing "smart" is displayed in `<strong>` tags
       - Values are either "Smart Capabilities" or "No Smart Capabilities"
       - Color and shell use `option1` and `option2` as fallbacks (may need refinement if Smart is in those positions)
   - **Accessories Display**:
     - For each accessory in `cartState.accessories`, uses `accessoryState` data (title, price, image) for display, with fallback to `selectedAccessories`
     - **CRITICAL**: Only displays accessories that actually exist in cart
       - Gets `accessoryCartItem` from cart using `cart.items.find()` by variant_id
       - If `accessoryCartItem` is `null` (item removed from cart), skips displaying that accessory line (`continue`)
       - This ensures removed accessories don't show in summary even if they're still in `cartState.accessories`
     - Displays quantities from actual cart (`accessoryCartItem.quantity`), not `cartState.accessories[].quantity`
     - **Variant Image Lookup**: **CRITICAL** - Gets variant-specific image from DOM, not generic product image
       - Method: `getAccessoryVariantImage(variantId)` looks up variant image from step 4 DOM
       - First checks `data-variant-images` custom map (if exists on accessory item)
       - Falls back to `data-variant-image` attribute on variant option element
       - Only falls back to `accessoryState.image` / `selectedAccessories[].image` if DOM lookup fails
       - **Why**: Cart API returns generic product image, not variant-specific image (especially for Smart Base)
       - **Result**: Summary shows correct variant image matching what user selected in step 4
     - **Variant Info Display**: Shows variant options (size, color, etc.) below accessory title
       - Extracts variant options from `accessoryState.variantOptions` (stored when adding accessory)
       - Falls back to extracting from cart item title (if title contains " - " separator)
       - Displays variant options in `<p class="summary-variant-info">` format
       - Options containing "Smart" are wrapped in `<strong>` tags (same as main product)
       - Format: `Option1, Option2, <strong>Smart Option</strong>` or just `Option1, Option2`
     - This ensures display matches actual cart state, showing all variants and accessories with their variant options and correct variant images
     - **Variant Image Selection**:
     - **CRITICAL**: Uses simple variant image map lookup from Liquid-generated JSON
     - **Variant Image Map**: Built in Liquid template (`product-multistep-form.liquid`) and loaded in JavaScript
       - **Liquid Script Tag**: `<script type="application/json" data-variant-image-map>` outputs `{ variantId: imageUrl }` for all variants
       - **Liquid Logic**: Loops through `product.variants`, uses `variant.image | img_url: 'master'` if exists, falls back to `product.featured_image | img_url: 'master'`
       - **JavaScript Loading**: Reads from `[data-variant-image-map]` script tag in `loadProductData()`
       - Stores variant images for quick O(1) lookup
     - **Lookup Logic**: 
       - Primary: `this.variantImageMap[variantId]` (variant-specific image from map)
       - **URL Normalization**: Converts protocol-relative URLs (`//domain.com/...`) to `https://domain.com/...` on the spot when looking up (simple fix at point of use)
       - Fallback: `this.productData.featured_image` (main product image if variant not in map)
     - **Debug Logging**: Logs how many variants loaded from Liquid, variant ID lookup, whether found in map, and final image URL
     - **Why Simple**: Liquid has direct access to variant images, no need to parse product JSON variants, handles fallback in Liquid, URL fix happens at point of use, more reliable
   - `getDeliveryText()`: Formats delivery information for display
   - `upgradeToSmart()`: Allows upgrading to smart version from summary
     - **CRITICAL**: Upgrades ALL non-smart variants in cart (not just one)
     - **Process**:
       1. Finds ALL main product items in cart (`product_id === this.productData.id`)
       2. For each item, checks if variant is non-smart using `isNonSmartVariant()` helper
       3. For each non-smart variant, finds matching smart variant using `findSmartVariantForNonSmart()` helper
       4. Upgrades each non-smart variant individually:
          - Preserves quantity and properties for each variant
          - Removes non-smart variant from cart
          - Adds matching smart variant with preserved quantity and properties
       5. After all upgrades, re-initializes `cartState` from cart using `initializeCartStateFromCart()`
       6. Re-renders summary to show updated variants
     - **Helper Functions**:
       - `isNonSmartVariant(variant)`: Checks if variant contains "non", "no ", or "no-" in any option
       - `findSmartVariantForNonSmart(nonSmartVariant)`: Finds matching smart variant by comparing color + shell options (excluding smart option), handles options in any position
     - **Why upgrade all**: If cart has multiple non-smart variants (different colors/shells or same variant with quantity > 1), all are upgraded, not just the one matching `this.selectedVariant.id`
     - **Works after quantity changes**: Even if user removed items from cart, upgrade still works because it checks actual cart items, not `this.selectedVariant`
     - Updates `cartState.accessories[]` with new variant ID when upgrading bases (via `upgradeBaseAccessoriesToSmart()`)
     - Then calls `renderOrderSummary()` to update display
   - **Totals Calculation**: 
     - **Main Product**: Loops through ALL main product variants in cart
       - For each variant: Uses `variantForDisplay.price` (from productData) multiplied by actual quantity from cart
       - Sums all main product variant totals
       - Debug logging shows quantity, price, and total for each variant
     - **Accessories**: Uses `accessoryState.price` (from cartState) multiplied by actual quantity from cart (`accessoryCartItem.quantity`)
     - Both read actual quantities from cart (not just cartState) to ensure totals match cart after quantity changes
     - Accessories get 20% discount (multiplied by 0.8)
     - Main product has no discount
     - Totals are recalculated every time `renderOrderSummary()` is called (after quantity changes)
     - Debug logging shows quantity, price, and total for each item (main product variants and accessories)
   - **Savings Display**: "Your Save" row only shown if savings > 0 (accessories discount)
   - **Cart Update**: When upgrading to smart, removes non-smart variant and adds smart variant with same delivery properties
   - **Base Upgrade**: `upgradeBaseAccessoriesToSmart()` checks cart for base accessories and upgrades them
   - **Base Upgrade Logic**: `upgradeBaseToSmart(baseCartItem)` fetches base product data, finds smart variant, and replaces in cart
   - `refreshAffirmPrice(totalInDollars)`: Updates Affirm payment messaging with current total price
   - **Dynamic Smart Upgrade Price**: "Add Smart Capabilities" price in summary uses `getSmartUpgradePrice()` to calculate price difference dynamically (same logic as Step 3), displays with compare_at_price strikethrough styling if applicable
   - **Quantity Pickers**:
     - **Location**: Step 5 summary for all items (main product and accessories)
     - **HTML Structure**: Uses theme's quantity picker classes (same as side cart):
       - Container: `quantity__wrapper` with `data-quantity-selector`
       - `−` button: `quantity__button quantity__button--minus` with `data-decrease-quantity` and `data-line-item-key` (line item key from cart)
       - Number input: `quantity__input` with `data-update-cart` (line item key from cart) and min/max constraints
       - `+` button: `quantity__button quantity__button--plus` with `data-increase-quantity` and `data-line-item-key` (line item key from cart)
       - **CRITICAL**: Uses line item keys (`item.key` from cart) for direct cart updates, NOT variant IDs
     - **Main Product Constraints**: `min="0" max="1"` (main product is always quantity 1 if added)
     - **Accessory Constraints**: `min="0" max="99"` (accessories can have multiple quantities)
     - **Event Listeners**: 
       - `attachQuantityPickerListeners()`: Attaches listeners to all quantity inputs and +/- buttons
       - **CRITICAL**: Uses `dataset.listenerAttached` flag to prevent duplicate listeners (HTML is recreated on each render)
       - Input: `change` event reads `data-update-cart` (line item key) and calls `updateCartItemQuantityDirect()`
       - Buttons: `click` events read `data-line-item-key`, update input value, then call `updateCartItemQuantityDirect()`
       - Uses `e.preventDefault()` and `e.stopPropagation()` on button clicks
       - **Prevents Double Updates**: Uses `dataset.programmaticUpdate` flag when buttons update input value to prevent input change event from firing
     - **Direct Cart Update**: `updateCartItemQuantityDirect(lineItemKey, quantity)`:
       - Uses `/cart/change.js` API with `{id: lineItemKey, quantity: quantity}` (same as theme's side cart)
       - Directly updates cart item by its key (no complex sync logic)
       - Fires `theme:cart:change` event after update
       - Logs to debug panel with type "CART"
       - Returns true/false for success/failure
     - **Refresh After Update**: `refreshSummaryAfterQuantityChange()`:
       - Calls `initializeCartStateFromCart()` to refresh `cartState` from actual cart (updates both main product and accessories)
       - Updates `selectedAccessories` quantities to match cart (for accessories only)
       - Calls `updateCartStateFromAccessories()` to sync `cartState.accessories` from `selectedAccessories`
       - Calls `renderOrderSummary()` to refresh display
       - **CRITICAL**: Main product variant display uses `cartState.mainProductVariantId` (from cart), not `selectedVariant`, ensuring display matches cart after quantity changes
     - **Display Logic**: `renderOrderSummary()` reads actual quantities from cart:
       - Gets cart items and finds matching items by `variant_id`
       - **Line Item Key Extraction**: 
         - Uses `item.key` (primary) or `item.id` (fallback) from cart.js API
         - Stores line item key in HTML attributes (`data-update-cart` for input, `data-line-item-key` for buttons)
         - **CRITICAL**: Shopify's `/cart.js` API may return `key` property or may need to use `id` property
         - Debug logging shows full cart item structure if key is missing
       - **Main Product Variant**: Uses `cartState.mainProductVariantId` (from cart) to find variant for display, NOT `selectedVariant` (which may be stale)
         - Finds variant from `productData.variants` using `cartState.mainProductVariantId`
         - Falls back to `selectedVariant` if `cartState.mainProductVariantId` is not available
         - Uses `variantForDisplay` for price, image, and variant info (ensures display matches cart)
         - This ensures main product display matches cart state, same as accessories
       - Main product: Gets quantity from `cart.items.find()` by variant_id (using `cartState.mainProductVariantId`)
       - Accessories: Gets quantity from `cart.items.find()` by variant_id
       - This ensures display matches actual cart state for both main product and accessories
     - **Debug Logging for Troubleshooting**:
       - When rendering summary, logs main product cart item lookup (variant_id search)
       - Logs full cart item structure if found (to see all available properties)
       - Logs line item key extraction (shows if `key` or `id` was used)
       - Logs error if cart item found but no key/id available
       - When buttons clicked, logs button click detection and line item key value
       - Logs error if button has no line item key (shows button HTML for debugging)
       - Logs error if input element not found for button
       - All debug logs go to debug panel Event Log (type "INFO" or "error")
       - **Purpose**: Helps diagnose why quantity pickers might not work (missing keys, wrong selectors, etc.)
     - **CSS Styling**: Uses theme's existing quantity picker styles from `theme.css`:
       - `.quantity__wrapper`: Flexbox container with max-width 103px, min-width 75px
       - `.quantity__input`: Centered text, border, padding 11px 30px
       - `.quantity__button`: Absolute positioned buttons (left/right), hover effects
       - No custom CSS needed - uses theme's styles
     - **Cart Updates**: 
       - **SIMPLE APPROACH**: Direct updates using line item keys (like theme's side cart)
       - No complex sync logic - just update cart directly by line item key
       - Cart API handles all the logic (add/remove/update)
       - Much simpler and more reliable than remove-all-then-add approach
     - **Zero Quantity**: Setting quantity to 0 removes item from cart (handled by Shopify cart API)
     - **Error Handling**: Wrapped in try/catch with error logging to debug panel
     - **Debug Logging**: All quantity changes logged to debug panel Event Log with type "QTY" showing line item key and quantity changes

8. **Affirm Integration**:
   - `refreshAffirmPrice(totalInDollars)`: Updates Affirm element with current total
   - **Location**: Step 5 only (order summary)
   - **Element**: `<div class="affirm-as-low-as" data-affirm-type="product" data-amount="TOTAL_IN_CENTS"></div>`
   - **Price Calculation**: Uses `totalDiscounted` (product price + accessories with 20% discount)
   - **Price Format**: Converted to cents for `data-amount` attribute
   - **Refresh Trigger**: Called automatically in `renderOrderSummary()` after calculating totals
   - **Refresh Method**: Uses `affirm.ui.refresh()` wrapped in `affirm.ui.ready()` if available
   - **Fallback**: If Affirm not loaded, waits 500ms and retries

8. **Feature Modals**:
   - `attachBannerListeners()`: Handles feature modal popups
   - Loads feature data from JSON script tags
   - Manages modal backdrop and close interactions

## CSS & Styling

### Fullscreen Mode

**Body Class**: `multistep-fullscreen`

**Effects**:
- Hides body overflow
- Fixes `#MainContent` to viewport
- Hides header/footer
- Shows fixed header with announcement bar offset
- White background overlay

**CSS Variables**:
- `--announcement-height`: Height of announcement bar (default: 0)

### Key CSS Classes

- `.multistep-container`: Main container (max-width: 1200px)
- `.multistep-progress`: Progress dots container
- `.progress-dot`: Individual progress indicator
- `.progress-dot.active`: Current step indicator
- `.progress-dot.completed`: Completed step indicator
- `[data-step]`: Step containers (hidden/shown via JavaScript)

### Image Filtering Logic

**Pattern Matching**:
- Images filtered by alt text format: `shell_color / fabric_color`
- Exact matches prioritized over shell-only matches
- Falls back to all images if no matches found

**Example Alt Text**:
- `"Black / Navy"` - Exact match for Black shell + Navy fabric
- `"Black /"` - Shell match for Black shell with any fabric

## Variant Matching Logic

### Step 2 (Partial Match)
Matches variants where:
- `option1/option2/option3` contains `selectedColor` (case-insensitive)
- `option1/option2/option3` contains `selectedShellColor` (case-insensitive)

### Step 3 (Final Match)
Matches variants where:
- `option1/option2/option3` contains `selectedColor` (case-insensitive)
- `option1/option2/option3` contains `selectedShellColor` (case-insensitive)
- `option1/option2/option3` contains `selectedSmartOption` (case-insensitive)

**Note**: Variant options can be in any position (option1, option2, or option3)

## Delivery Properties

### Main Product Delivery
Priority order:
1. `input[name="deliveryDate"]` value
2. `input[name="deliveryTime"]` value
3. `variant.metafields.delivery.delivery_estimated_date`
4. `product.metafields.delivery.delivery_time`
5. `product.metafields.delivery.estimated_date`

### Accessory Delivery
Priority order:
1. `data-variant-delivery-date` attribute
2. `data-delivery-time` attribute
3. `data-delivery-date` attribute

**Cart Properties**:
- Stored as `properties` object in cart line items
- Keys: `"Delivery Date"` or `"Delivery Time"`

## Smart Option Upgrade

### Upgrade Flow
- Available in Step 5 if non-smart version selected
- Shows upgrade card with dynamic upgrade price (calculated from variant price difference)
- Clicking "ADD" button:
  - Shows "added" state: Hides "ADD" text, shows "Added" text with checkmark icon, adds `showing-added` class
  - After 2 seconds: Reverts to "ADD" state (but stays on Step 5, doesn't navigate)
  - Updates `selectedSmartOption` to smart value
  - **CRITICAL**: Upgrades ALL non-smart variants in cart (not just one)
  - **Process**:
    1. Finds all main product items in cart (`product_id === this.productData.id`)
    2. Identifies which variants are non-smart using `isNonSmartVariant()` helper (checks for "non", "no ", or "no-" in options)
    3. For each non-smart variant, finds matching smart variant using `findSmartVariantForNonSmart()` helper (matches color + shell, excludes smart option)
    4. Upgrades each non-smart variant individually:
       - **CRITICAL**: Preserves quantity when upgrading (uses `cartItem.quantity`, not hardcoded 1)
       - **CRITICAL**: Preserves properties from cart item (delivery date/time)
       - Removes non-smart variant from cart
       - Adds matching smart variant with preserved quantity and properties
    5. After all upgrades, calls `initializeCartStateFromCart()` to fully sync cartState from cart
  - **Works after quantity changes**: Even if user removed items from cart, upgrade still works because it checks actual cart items, not `this.selectedVariant`
  - **Multiple variants**: If cart has variant A (qty 2) and variant B (qty 1), both are upgraded separately with their own quantities preserved
  - **Base Accessory Upgrade**: Automatically checks `selectedAccessories` and cart for base accessories
  - **Logic**: 
    - Checks if base title contains "classic"
    - **If base is in cart**: Uses `upgradeBaseToSmart()`:
      - Extracts product handle from cart item URL (regex: `/\/products\/([^\/\?]+)/`)
      - Fetches classic base product JSON
      - Detects if classic base (single variant or title contains "classic")
      - If classic base: Finds smart base product handle from DOM (`data-base-type="smart"`), fetches smart base product JSON
      - Matches smart base variant by shell color (bases only have one color option in `option1`)
      - Removes classic base from cart and adds smart base variant
      - Updates `selectedAccessories` array with smart base variant ID, price, and title
      - Re-renders order summary if on step 5
    - **If base NOT in cart**: Uses `upgradeBaseAccessoryToSmart()`:
      - Finds classic base item in DOM using `productHandle` or `productId`
      - Reads `data-smart-base-variant-id` from classic base item
      - Finds smart base item in DOM to get price and title from checkbox
      - Updates `selectedAccessories` array with smart base variant ID, price, and title
  - **Variant Matching for Classic Base**:
    - Classic base has NO variants (single product)
    - Smart base HAS variants (color options: Eggshell White, Charcoal Grey, etc.)
    - Matching: Smart base variant `option1` must match main product's `selectedShellColor`
    - Example: If shell color is "Eggshell White", find smart base variant with `option1 === "Eggshell White"`
  - Preserves delivery properties and quantities
  - Re-renders order summary with updated pricing and base information

### Smart Option Values
- Stored in `[data-smart-option-input]` element
- `data-smart-value`: Smart variant option value (e.g., "Smart Capabilities")
- `data-non-smart-value`: Non-smart variant option value (e.g., "No Smart Capabilities")
- **Detection Logic**: 
  - Smart selected: Contains "smart" but NOT "non", "classic", "no ", or "no-"
  - Non-smart selected: Contains "non", "no ", or "no-" (starts with)
  - Liquid template checks for "no" in value to determine non-smart value

## Accessories System

### Collection Source
- Configured via `data-accessories-collection` attribute
- Default: `"accessories"`
- Fetched via Shopify collection handle

### Accessory Selection
- Checkbox-based selection
- Auto-unchecks after 2 seconds (visual feedback)
- Supports variant options (size, color, etc.)
- Image updates based on variant selection
- Quantity tracking (can add same accessory multiple times)

### Pricing
- 20% discount applied automatically
- Displayed as: discounted price (strikethrough original price)
- Savings shown in order summary (only if savings > 0)

### Accessory Filtering by Smart Option
- **Location**: `filterAccessoriesBySmartOption()` method
- **Trigger**: Called when Step 4 is shown
- **Logic**:
  - If smart selected: Hide accessories with "classic" in product name
  - If non-smart selected: Hide accessories with "smart" in product name (but NOT those starting with "no " or "no-")
- **Purpose**: Show only compatible accessories based on smart/non-smart selection

### Accessory Availability Filtering
- **Location**: Liquid template (`product-multistep-step4.liquid`)
- **Logic**:
  - **Variant Options**: Only shows swatches for variants where `variant.available == true`
  - **Entire Item**: If accessory has variants and NONE are available, the entire accessory item is hidden (using `continue`)
  - **Single Variant**: If accessory has only one variant and it's not available, the entire item is hidden
  - **Add Button**: Only shown if at least one variant is available
- **Purpose**: Prevents users from seeing or attempting to add unavailable accessories

### Base Accessory Marking
- **Location**: Liquid template (`product-multistep-step4.liquid`)
- **Data Attributes**:
  - Classic base: `data-base-type="classic"` + `data-smart-base-variant-id="{variant_id}"`
  - Smart base: `data-base-type="smart"`
- **Logic**:
  - Identifies bases by checking if title contains "base"
  - Marks classic bases (title contains "classic")
  - Marks smart bases (title contains "smart" but not "classic")
  - For classic bases, finds smart base product in collection and stores its first available variant ID
  - Smart base variant ID stored in `data-smart-base-variant-id` on classic base item
- **Purpose**: Enables simple upgrade logic - just read `data-smart-base-variant-id` from DOM

## URL Parameters

### Step Navigation
- `?step=2` - Jump directly to Step 2 (triggers customize button, hides sections, enters fullscreen)
- `?step=3` - **IGNORED** - Redirects to Step 2 (intentional behavior)
- `?step=4` - **IGNORED** - Redirects to Step 2 (intentional behavior)
- `?step=5` - **IGNORED** - Redirects to Step 2 (intentional behavior)

**Note**: 
- Step 1 is default (no parameter needed)
- Only `?step=2` is supported for direct navigation. Steps 3, 4, and 5 parameters are intentionally ignored and will redirect to Step 2 to ensure proper flow initialization (section hiding, fullscreen mode, etc.)

## Event System

### Custom Events Dispatched

1. **`theme:variant:change`**:
   - Dispatched when variant selection changes
   - Detail: `{ variant: variantObject }`
   - Used by theme to update product images

2. **`theme:cart:change`**:
   - Dispatched after cart addition
   - Detail: `{ cart: cartObject }`
   - Used by theme to update cart UI

## Product JSON Structure

The template includes complete product JSON with:
- Basic product info (id, title, handle, description)
- Pricing information
- Variants array with full variant data
- Images and media
- Metafields (delivery_time, estimated_date)
- Variant metafields (delivery_estimated_date)

**Access**: Via `[data-product-json]` script tag, parsed by JavaScript

**Variant Metafields**:
- `delivery_estimated_date`: Delivery date for specific variant
- `hidevariant`: Boolean - if true, hides the fabric color option when shell color is selected (custom.hidevariant)

## Progress Indicators

### Progress Dots
- 4 dots total (Steps 2-5)
- Hidden on Step 1
- Visual states:
  - `.active`: Current step
  - `.completed`: Past steps
  - Default: Future steps

### Step Visibility
- Only one step visible at a time
- Controlled via `display: none/block` CSS
- Managed by `showStep()` method

## Button States & Feedback

### Add to Cart Button (Step 4)
- Shows "Next" text (no "Added" state)
- Proceeds directly to Step 5 after adding to cart
- Disabled during cart addition

### Smart Option Button (Step 3)
- Similar add/added state pattern
- Auto-advances to Step 4 after selection

## Image Handling

### Variant Images
- Updated via `updateImages()` method
- Uses Shopify image URL transformation: `_800x.jpg`
- Dispatches `theme:variant:change` event for theme integration

### Slider Images
- Filtered based on color selections
- Original slides stored in `allSliderSlides` array
- Filtered slides replace slider content dynamically
- Maintains slide order and structure

## Mobile Considerations

### Touch Support
- Slider supports touch drag/swipe
- Minimum drag distance: 50px
- Prevents accidental navigation

### Responsive Design
- Fullscreen mode works on mobile
- Progress dots adapt to screen size
- Form elements sized appropriately

## Customize Button Behavior

### Section Hiding
- **Method**: `hideAllSectionsExceptProduct()`
- **Trigger**: When "Customize" button is clicked (Step 1)
- **Behavior**:
  - Hides all sections except the product section
  - Includes app sections (Judge.me, etc.)
  - Also hides: `#accessiblyAppWidgetButton` and `#tidio-chat`
  - Stores original display styles in `dataset.originalDisplay` for potential restoration
  - Adds `multistep-customize-mode` class to body
  - Scrolls to product section
- **Prevention**: Uses `sectionsHidden` flag to prevent recursive calls

### Section Restoration
- **Method**: `restoreAllSections()`
- **Trigger**: Automatically called when navigating back to Step 1
- **Behavior**:
  - Restores all sections that were hidden (using stored `originalDisplay` values)
  - Restores `#accessiblyAppWidgetButton` and `#tidio-chat` widgets
  - Removes `multistep-customize-mode` class from body
  - Resets `sectionsHidden` flag
  - Ensures full page is visible when returning to Step 1

## Critical Implementation Notes

### 1. Product Media Hiding
- Default product media containers hidden via CSS
- Prevents duplicate image displays
- Maintains theme compatibility

### 2. Header Management
- Header hidden during Steps 2-5
- Fixed positioning during fullscreen
- Announcement bar height accounted for

### 3. Variant Option Flexibility
- Variant options can be in any position
- Case-insensitive matching
- Handles missing options gracefully

### 4. Image Alt Text Format
- Critical for slider filtering
- Format: `"shell_color / fabric_color"`
- Must match exactly for filtering to work

### 5. Accessory Auto-Uncheck
- 2-second timeout for visual feedback
- Prevents accidental double-selection
- Can be manually unchecked before timeout

### 6. Delivery Metafields
- Supports both product and variant metafields
- Fallback chain ensures delivery info always available
- Format: Date (YYYY-MM-DD) or Time (text)

### 7. Performance & Rendering Optimizations
- **Event Listener Management**: 
  - `initializeSlider()` checks `sliderInitialized` flag before adding listeners
  - Event handlers stored in `sliderEventHandlers` object to prevent duplicate attachments
  - Uses `dataset.listenerAttached` attributes to track listener state
  - Prevents browser freezing from multiple event listener stacks
- **Event Throttling**:
  - `mousemove` events throttled using `requestAnimationFrame` to prevent excessive firing
  - Prevents performance issues when DevTools is open
- **Initialization Guards**:
  - `showStep()` uses `requestAnimationFrame` instead of `setTimeout` for better performance
  - Early returns prevent unnecessary DOM queries and operations
- **Memory Leak Prevention**:
  - Drag state variables stored outside event handlers to prevent closure leaks
  - Event handler references stored in class properties for proper cleanup
  - Prevents memory accumulation from repeated initializations

## File Dependencies

### Required Files
1. `/snippets/product-multistep-form.liquid` - Main form container
2. `/snippets/product-multistep-step1.liquid` - Hero/intro step
3. `/snippets/product-multistep-step2.liquid` - Color selection step
4. `/snippets/product-multistep-step3.liquid` - Smart option step
5. `/snippets/product-multistep-step4.liquid` - Accessories step
6. `/snippets/product-multistep-step5.liquid` - Summary step
7. `/assets/product-multistep.js` - JavaScript functionality
8. `/assets/product-multistep.css` - Styling

### Optional Files
- `/sections/section-multistep-hero.liquid` - Hero section (can be used separately)

## Usage Instructions

### Applying Template to Product
1. Go to Shopify Admin > Products
2. Select product
3. In "Theme templates" section, select "pdp_multistep_v1"
4. Save product

### URL Navigation
- Default: Shows Step 1 (hero)
- `?step=2`: Jump to color selection (triggers customize button, hides sections, enters fullscreen)
- `?step=3`: **IGNORED** - Redirects to Step 2 (intentional behavior)
- `?step=4`: **IGNORED** - Redirects to Step 2 (intentional behavior)
- `?step=5`: **IGNORED** - Redirects to Step 2 (intentional behavior)

### Accessories Collection
- Ensure "accessories" collection exists
- Or update `data-accessories-collection` attribute in form snippet

## Development Guidelines

### Adding New Steps
1. Create new step snippet: `product-multistep-step6.liquid`
2. Render in `product-multistep-form.liquid`
3. Update `totalSteps` in JavaScript
4. Add progress dot in template
5. Update `showStep()` logic

### Modifying Variant Logic
- Update `selectPartialVariant()` for Step 2 changes
- Update `selectFinalVariant()` for Step 3 changes
- Ensure variant option matching handles all cases

### Customizing Styling
- All styles in `/assets/product-multistep.css`
- Fullscreen styles in template's custom liquid section

## Performance Best Practices

### Event Listener Management
- **NEVER** add event listeners without checking if they're already attached
- Use `dataset` attributes or flags to track listener state
- Store handler references in class properties for cleanup
- Example pattern:
  ```javascript
  if (!element.dataset.listenerAttached) {
    element.addEventListener('event', handler);
    element.dataset.listenerAttached = 'true';
  }
  ```

### Event Throttling
- Always throttle high-frequency events like `mousemove`, `scroll`, `resize`
- Use `requestAnimationFrame` for smooth animations and throttling
- Example:
  ```javascript
  let timeout;
  element.addEventListener('mousemove', (e) => {
    if (timeout) return;
    timeout = requestAnimationFrame(() => {
      // Handle event
      timeout = null;
    });
  });
  ```

### Initialization Guards
- Always check if initialization has already occurred before running setup code
- Use flags (`sliderInitialized`, `sectionsHidden`) to prevent duplicate work
- Early returns prevent unnecessary DOM queries and operations

### Memory Leak Prevention
- Store event handler references in class properties, not closures
- Avoid creating new functions in loops or repeated calls
- Use `removeEventListener` when cleaning up (if needed)
- Store drag state variables outside event handlers to prevent closure leaks
- Use CSS variables for theme integration

### Debugging
- Check browser console for JavaScript errors
- Verify product JSON structure
- Check variant option values match selections
- Verify image alt text format for filtering

## Common Issues & Solutions

### Images Not Filtering
- **Cause**: Alt text format incorrect
- **Solution**: Ensure format is `"shell_color / fabric_color"`

### Variant Not Found
- **Cause**: Option values don't match (case/whitespace)
- **Solution**: Check variant option values match exactly

### Accessories Not Loading
- **Cause**: Collection handle incorrect
- **Solution**: Verify `data-accessories-collection` attribute

### Fullscreen Not Working
- **Cause**: CSS not loading or body class not applied
- **Solution**: Check template's custom liquid section CSS

### Cart Addition Fails
- **Cause**: Variant unavailable or invalid
- **Solution**: Check variant availability and ID

## Best Practices

1. **Variant Naming**: Use consistent naming for variant options
2. **Image Alt Text**: Follow exact format for filtering
3. **Metafields**: Set up delivery metafields properly
4. **Testing**: Test all variant combinations
5. **Mobile**: Verify touch interactions work
6. **Performance**: Optimize image sizes for slider
7. **Accessibility**: Ensure keyboard navigation works
8. **Error Handling**: Provide user feedback for errors

## Affirm Payment Integration

### Implementation
- **Location**: Step 5 (Order Summary) only
- **Element**: `<div class="affirm-as-low-as" data-page-type="product"></div>`
- **Note**: Uses `data-page-type` (NOT `data-affirm-type`) - this is required by Affirm.js
- **Price Source**: Final total including product + accessories (with 20% discount)
- **Price Format**: Cents (multiply dollars by 100, rounded)
- **Price Attribute**: Set dynamically via JavaScript `data-amount` attribute (not in Liquid)

### Price Updates
- **Trigger**: Automatically called in `renderOrderSummary()` after calculating `totalDiscounted`
- **Method**: `refreshAffirmPrice(totalInDollars)`
- **Process**:
  1. Converts dollars to cents
  2. Updates `data-amount` attribute on Affirm element
  3. Calls `affirm.ui.refresh()` to update display
  4. Wraps in `affirm.ui.ready()` if available
  5. Falls back with timeout if Affirm not loaded

### When Price Updates
- Initial render of Step 5 (order summary)
- When upgrading to smart variant (via `upgradeToSmart()` → `renderOrderSummary()`)
- Price includes: Main product + all accessories (with 20% discount applied)

### Requirements
- Affirm.js must be loaded globally (via Shopify app or theme)
- Affirm element must exist in Step 5 template
- Price must be > 0 for Affirm to display
- **Note**: The "Check your purchasing power" link was removed as it's not needed

## Debug Panel System

### Debug Panel Features
- **Location**: Floating panel in top-left corner (when enabled)
- **Enable**: Add `?debug=true` to URL or set `localStorage.setItem("multistep_debug", "true")`
- **Sections**:
  - Current Step: Current step number
  - Step Navigation History: Last 20 step transitions with timestamps
  - cartState JSON: Complete cart state object
  - selectedAccessories: Array of selected accessories with quantities
  - Current Cart: Actual cart items from Shopify
  - Selected Variant: Currently selected variant info
  - Event Log: Real-time log of important events (listener attachments, addAccessory calls, etc.)

### Debug Event Log
- **Purpose**: Track all important events for debugging (NOT console.log)
- **Method**: `addDebugLog(type, message)` - adds entry to debug panel
- **Types**: ATTACH, SKIP, EVENT, CALL, ADD, INCR, etc.
- **CRITICAL RULE**: NEVER use `console.log()` for debugging - ALWAYS use `addDebugLog()` to add to debug panel
- **Auto-updates**: Debug panel updates every 500ms automatically
- **Copy Button**: Copies all debug data as JSON for analysis

### Debug Panel Rules
- **NEVER use console.log()** - Always use `this.addDebugLog(type, message)` instead
- Debug panel shows all important data: cartState, selectedAccessories, current cart, step history, event log
- Event log tracks: listener attachments, checkbox events, addAccessory calls, quantity changes (from quantity pickers in summary)
- Quantity picker changes logged with type "QTY" showing: variantId, isMainProduct, newQuantity
- Debug panel updates automatically on step changes, accessory clicks, cart syncs, quantity picker changes
- Use debug panel to diagnose issues, not console.log
- **CRITICAL**: All debugging must go to debug panel event log, never console.log
- **Quantity Picker Debugging**: When quantity changes, check Event Log for "QTY" entries showing variant ID, product type, and new quantity

## Future Enhancements

Potential improvements:
- Step persistence (save progress)
- Step validation improvements
- More granular image filtering
- Additional accessory options
- Step skipping for returning customers
- Analytics integration for step tracking

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bareta006) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
