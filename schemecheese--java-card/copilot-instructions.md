## java-card

> [SimulatorService.java](mdc:card_gui/src/service/SimulatorService.java) is the bridge between GUI and JavaCard applet.

# SimulatorService Guidelines

## Overview
[SimulatorService.java](mdc:card_gui/src/service/SimulatorService.java) is the bridge between GUI and JavaCard applet.

## Key Methods

### Connection
```java
simulatorService.connect();        // Start simulator, install applet
simulatorService.isConnected();    // Check connection status
```

### PIN Operations
```java
simulatorService.createDemoPin();           // Create default PIN (123456)
simulatorService.verifyPin(char[] pin);     // Verify PIN, sets isPinVerified
simulatorService.changePin(oldPin, newPin); // Change PIN
simulatorService.isPinVerified();           // Check auth status
```

### Card Info Operations
```java
simulatorService.setCardInfo(CardInfo);     // Save to JavaCard (requires PIN)
simulatorService.getCardInfo();             // Read from JavaCard
simulatorService.addCardToList(CardInfo);   // Add to in-memory list
simulatorService.getAllCards();             // Get all cards from list
simulatorService.searchCards(keyword);      // Search by ID or name
simulatorService.toggleCardStatus(id);      // Lock/unlock card
```

## Data Storage
- **JavaCard**: Stores only 3 fields (ID, Name, Date) via APDU
- **cardList**: In-memory ArrayList for full CardInfo (6+ fields)

## APDU Format for Card Info

### Set Card Info
```
[CLA=0x00][INS=0x40][P1=0x00][P2=0x00][Lc][DATA]
DATA = [CARD_ID (10 bytes)][NAME_LEN][NAME][EXPIRY (8 bytes)]
```

### Get Card Info
```
[CLA=0x00][INS=0x50][P1=0x00][P2=0x00][Le=0x00]
Response = [CARD_ID][NAME_LEN][NAME][EXPIRY][NUM_BOOKS][SW1][SW2]
```

## Security Note
PIN verification is required before `setCardInfo()` will succeed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SchemeCheese) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
