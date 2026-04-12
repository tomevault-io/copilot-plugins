## from-scratch-travel-system

> ECRIS LES TEXTES EN ANGLAIS PARTTOUT DANS LES PAGES

ECRIS LES TEXTES EN ANGLAIS PARTTOUT DANS LES PAGES
ARCHITECTURE.md (pour Cursor comprenne la structure)
markdown# Architecture Overview

## Data Flow
User Input → Search Form → Filter Mock Data → Display Results → Select Flight → Open Modal → Add Taxi (optional) → Simulate Payment → Show Confirmation

## Module Dependencies
main.js
├─ search.js
│    ├─ data.js (AIRPORTS, FLIGHTS)
│    └─ utils.js (formatDate, formatPrice)
└─ booking.js
├─ data.js (TRANSFERS)
└─ utils.js (validateEmail, generateConfirmationCode)

## Key Functions

### search.js
- `setupAutocomplete(inputId, dropdownId)` - Handles airport suggestions
- `setupSearchForm()` - Form submission handler
- `searchFlights(criteria)` - Filters flights array
- `displayResults(flights)` - Renders flight cards to DOM
- `sortFlights(flights, sortBy)` - Sorts by price/duration/time

### booking.js
- `openBookingModal(flightId)` - Shows modal with flight details
- `closeBookingModal()` - Hides modal and resets state
- `setupTaxiToggle()` - Handles taxi checkbox
- `updateTotalPrice()` - Recalculates total (flight + taxi)
- `validateBookingForm()` - Checks required fields
- `simulatePayment()` - Fake payment processing
- `showConfirmation(code)` - Success screen

### utils.js
- `formatDate(date)` - Returns DD/MM/YYYY
- `formatPrice(amount, currency)` - Returns €XX
- `generateConfirmationCode()` - Returns SKXXXXXXXX
- `validateEmail(email)` - Regex validation
- `setMinDate(inputId)` - Sets today as minimum

## State Management

No global state manager. Use:
- Function parameters for data passing
- DOM data attributes for IDs (data-flight-id)
- Local variables within functions
- Event listeners for reactivity

## Mock Data Structure

### AIRPORTS (Array)
```javascript
{ code: "CDG", city: "Paris", name: "Charles de Gaulle", country: "France" }
FLIGHTS (Array)
javascript{ 
  id: "AF001", 
  airline: "Air France", 
  airlineCode: "AF",
  from: "CDG", 
  to: "LHR",
  departure: "08:30",
  arrival: "09:45",
  duration: "1h 15m",
  stops: 0,
  price: 89,
  currency: "xaf"
}
TRANSFERS (Object)
javascript"CDG": [
  { type: "Economy", price: 25, vehicle: "Sedan", duration: "45min" }
]

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/acelest)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/acelest)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
