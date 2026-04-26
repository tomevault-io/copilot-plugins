## qc-portfolio

> **ALWAYS CHECK QUANTCONNECT'S BUILT-IN CAPABILITIES BEFORE IMPLEMENTING CUSTOM SOLUTIONS**

# QuantConnect Three-Layer CTA Framework - Cursor Rules

## 🎯 **CORE PRINCIPLES**

### **0. LEAN-FIRST DEVELOPMENT** 🚨 **PRIMARY DIRECTIVE**
**ALWAYS CHECK QUANTCONNECT'S BUILT-IN CAPABILITIES BEFORE IMPLEMENTING CUSTOM SOLUTIONS**

### **⚠️ CRITICAL: QUANTCONNECT SYMBOL OBJECT LIMITATION** 🚨 **NEVER IGNORE**
**NEVER pass QuantConnect Symbol objects as constructor parameters - causes "error return without exception set"**

#### **The Problem**
QuantConnect Symbol objects get wrapped in `clr.MetaClass` which cannot be passed to Python constructors.
This causes the infamous "error return without exception set" error that crashes algorithm initialization.

#### **Forbidden Patterns** ❌
```python
# ❌ NEVER DO THIS - Will cause constructor crashes
def __init__(self, symbol_objects):
    self.symbols = symbol_objects  # Symbol objects cause clr.MetaClass issues

# ❌ NEVER DO THIS - Passing Symbol objects between components  
component = MyComponent(self, config_manager, shared_symbols)  # shared_symbols contains Symbol objects

# ❌ NEVER DO THIS - Storing Symbol objects in shared data structures
shared_data = {'symbols': [symbol1, symbol2, symbol3]}  # Symbol objects will crash constructors
```

#### **Correct Patterns** ✅
```python
# ✅ Use string identifiers instead
def __init__(self, symbol_strings):
    self.symbol_strings = symbol_strings  # ['ES', 'NQ', 'ZN'] - safe to pass

# ✅ Convert Symbol objects to strings before passing
symbol_strings = [str(symbol) for symbol in symbol_objects]
component = MyComponent(self, config_manager, symbol_strings)

# ✅ Use QC's native methods directly - let QC handle Symbol creation
future = self.AddFuture("ES", Resolution.Daily)  # QC creates Symbol internally
self.futures_symbols.append(future.Symbol)       # Store QC-created Symbol
```

#### **Solution: Use QC Native Universe Management**
```python
# ✅ CORRECT - Direct QC native approach (no custom Symbol management)
def _setup_futures_universe(self):
    for symbol_str in ['ES', 'NQ', 'ZN']:
        future = self.AddFuture(symbol_str, Resolution.Daily)  # QC handles Symbol creation
        self.futures_symbols.append(future.Symbol)            # Store QC-created Symbol
```

**Reference**: [QuantConnect Forum - Python Custom Exception Issue](mdc:https:/www.quantconnect.com/forum/discussion/9331/python-custom-exception-definition-does-not-seem-to-work)

#### **MANDATORY LEAN CAPABILITY CHECK**
Before writing ANY trading, financial, or data processing code:
1. **CHECK**: Does QC/LEAN already provide this functionality?
2. **VERIFY**: Review the LEAN API documentation
3. **CONFIRM**: Search existing QC methods in AlgorithmImports
4. **IMPLEMENT**: Use LEAN's native methods, not custom implementations

#### **COMMON LEAN CAPABILITIES (Use These, Don't Reinvent)**
```python
# Portfolio Management - NEVER build custom portfolio tracking
self.Portfolio.TotalPortfolioValue          # Total portfolio value
self.Portfolio[symbol].Quantity             # Position quantity
self.Portfolio[symbol].HoldingsValue        # Position value
self.Portfolio[symbol].UnrealizedProfit     # Unrealized P&L
self.Portfolio.Cash                         # Available cash
self.Portfolio.TotalMarginUsed              # Margin usage

# Technical Indicators - NEVER build custom indicators
self.SMA(symbol, period)                    # Simple Moving Average
self.EMA(symbol, period)                    # Exponential Moving Average  
self.RSI(symbol, period)                    # Relative Strength Index
self.MACD(symbol, fast, slow, signal)       # MACD
self.BB(symbol, period, std)                # Bollinger Bands
self.ATR(symbol, period)                    # Average True Range
self.AROON(symbol, period)                  # Aroon Oscillator
self.CCI(symbol, period)                    # Commodity Channel Index

# Order Management - NEVER build custom order systems
self.MarketOrder(symbol, quantity)          # Market orders
self.LimitOrder(symbol, quantity, price)    # Limit orders
self.StopMarketOrder(symbol, quantity, stop)# Stop orders
self.SetHoldings(symbol, percentage)        # Position sizing
self.Liquidate(symbol)                      # Close positions
self.CalculateOrderQuantity(symbol, target) # Calculate quantities

# Data Access - NEVER build custom data fetching
self.History(symbol, bars, resolution)      # Historical data
self.Securities[symbol].Price               # Current price
self.Securities[symbol].HasData             # Data availability
self.Securities[symbol].IsTradable          # Trading status
self.Securities[symbol].Mapped              # Mapped contract (futures)

# Scheduling - NEVER build custom timers/schedulers
self.Schedule.On(date_rule, time_rule, action)  # Custom scheduling
self.DateRules.EveryDay(symbol)                 # Daily scheduling
self.DateRules.Every(DayOfWeek.Monday)          # Weekly scheduling
self.TimeRules.AfterMarketOpen(symbol, minutes) # Market open timing
self.TimeRules.BeforeMarketClose(symbol, minutes) # Market close timing

# Universe Selection - NEVER build custom universe logic
self.AddEquity(symbol, resolution)          # Add equity
self.AddForex(symbol, resolution)           # Add forex
self.AddFuture(symbol, resolution)          # Add futures (continuous)
self.AddCrypto(symbol, resolution)          # Add crypto
self.UniverseSettings.Resolution            # Set universe resolution

# Risk Management - NEVER build custom risk systems
self.Settings.MaximumOrderValue             # Order value limits
self.Settings.FreePortfolioValue            # Available capital
self.IsMarketOpen(symbol)                   # Market hours check
```

#### **LEAN-FIRST CODE REVIEW CHECKLIST**
Before accepting ANY financial/trading code:
- [ ] Does this feel like "basic trading infrastructure"? (If yes, LEAN has it)
- [ ] Are we calculating portfolio values manually? (Use self.Portfolio methods)
- [ ] Are we tracking positions manually? (Use self.Portfolio[symbol])
- [ ] Are we building custom indicators? (Use self.SMA, self.RSI, etc.)
- [ ] Are we implementing custom schedulers? (Use self.Schedule.On)
- [ ] Are we fetching data manually? (Use self.History, self.Securities)
- [ ] Are we managing orders manually? (Use self.MarketOrder, self.SetHoldings)

### **1. QC NATIVE FIRST**
- **ALWAYS** use QuantConnect's built-in functionality before writing custom code
- Leverage `.Mapped` property for continuous → underlying contract mapping
- Use QC's `OnSymbolChangedEvents` for rollover handling
- Prefer QC's `HasData`, `IsTradable`, `Price` properties for validation
- Use QC's native `History()`, `SetHoldings()`, `MarketOrder()` methods

### **2. CENTRALIZED CONFIGURATION SECURITY** 🚨 **CRITICAL**
- **ALL** configuration MUST go through `AlgorithmConfigManager` centralized methods
- **ZERO** direct config dictionary access (`config.get()`, `self.config[...]`)
- **NO** fallback configurations or default values anywhere
- **FAIL FAST** on any configuration errors - never trade with wrong parameters
- **COMPLETE** configuration audit trail on startup

### **3. ZERO HARDCODED VALUES POLICY**
- **EVERY** parameter must come from validated configuration
- **NO** literal numbers anywhere in strategy logic
- **NO** hardcoded dates, times, thresholds, or limits
- **ALL** component constructors must use `config_manager.get_*_config()` methods

### **4. MODULAR & CLOUD-READY**
- Keep each file under **64KB** for QC cloud integration
- Use component-based architecture with clear separation of concerns
- Each component handles one responsibility (strategies, allocation, risk, execution)
- Config-driven initialization for all components

---

## 🔒 **CONFIGURATION SECURITY PATTERNS** 🚨 **CRITICAL**

### **Centralized Access Pattern**
```python
# ✅ CORRECT - All components use centralized config manager
class KestnerCTAStrategy:
    def __init__(self, algorithm, config_manager):
        self.config = config_manager.get_strategy_config('KestnerCTA')
        
        # Validate required parameters exist
        required_params = ['momentum_lookbacks', 'volatility_lookback_days', 'target_volatility']
        for param in required_params:
            if param not in self.config:
                error_msg = f"Missing required parameter '{param}' in KestnerCTA configuration"
                algorithm.Error(f"CONFIG ERROR: {error_msg}")
                raise ValueError(error_msg)

# ❌ FORBIDDEN - Direct config access bypasses validation
class BadStrategy:
    def __init__(self, config):
        self.lookback = config.get('lookback', 32)  # DANGEROUS FALLBACK
```

### **Required Configuration Methods**
```python
# All components MUST use these centralized methods
config_manager.get_strategy_config(strategy_name)    # Strategy configurations
config_manager.get_risk_config()                     # Risk management parameters
config_manager.get_execution_config()                # Execution parameters
config_manager.get_algorithm_config()                # Algorithm settings
config_manager.get_universe_config()                 # Universe definitions
config_manager.get_futures_chain_config()            # Futures chain config
config_manager.validate_complete_configuration()     # Startup validation
config_manager.get_config_audit_report()             # Audit trail
```

### **Forbidden Security Patterns** ❌
```python
# NEVER DO THESE - SECURITY RISKS
config.get('parameter', default_value)              # Dangerous fallback
self.config['parameter']                             # Direct access bypasses validation
if 'parameter' not in config: config['parameter'] = default  # Silent default
try: param = config['param'] except: param = default        # Hidden fallback
```

---

## 🔥 **QC WARM-UP SYSTEM PATTERNS** 🚨 **CRITICAL**

### **1. PROPER WARM-UP SETUP**
```python
# ✅ CORRECT - Strategy-based warm-up calculation using LEAN's built-in methods
def setup_warmup(self, config_manager):
    """Calculate and set proper warm-up period using LEAN's native SetWarmUp"""
    warmup_config = config_manager.get_warmup_config()
    
    # Use LEAN's built-in time-based warm-up (not bar-count)
    warmup_days = warmup_config['calculated_days']
    self.SetWarmUp(timedelta(days=warmup_days))  # LEAN's native method
    
    # Enable LEAN's automatic indicator warm-up
    self.Settings.AutomaticIndicatorWarmUp = True  # LEAN's native setting
    
    self.Log(f"Warm-up period set: {warmup_days} days")

# ❌ WRONG - Hardcoded or insufficient warm-up
self.SetWarmUp(20)  # HARDCODED - insufficient for strategies
```

### **2. WARM-UP AWARE LOGIC**
```python
# ✅ CORRECT - Different behavior during warm-up using LEAN's IsWarmingUp
def OnData(self, slice: Slice):
    if self.IsWarmingUp:  # LEAN's native property
        # During warm-up: State building only, no trading
        self._update_indicators_and_state(slice)
        return
    
    # Post warm-up: Full trading logic
    self._execute_trading_logic(slice)

def OnWarmupFinished(self):  # LEAN's native event
    """Validate algorithm state after warm-up completion"""
    self.Log("Warm-up completed - validating algorithm state")
    
    # Validate indicators are ready using LEAN's IsReady property
    if not self._validate_indicators_ready():
        raise Exception("Indicators not properly warmed up")
    
    # Validate universe is populated
    if not self._validate_universe_ready():
        raise Exception("Universe not properly populated")
```

---

## 📈 **QC FUTURES CHAIN & CONTINUOUS CONTRACTS** 🚨 **CRITICAL**

### **1. CONTINUOUS CONTRACTS ONLY**
```python
# ✅ CORRECT - Continuous contracts using LEAN's AddFuture
future = self.AddFuture(  # LEAN's native method
    ticker="ES",
    resolution=Resolution.Daily,
    dataMappingMode=DataMappingMode.OpenInterest,      # LEAN's native enum
    dataNormalizationMode=DataNormalizationMode.BackwardsRatio,  # LEAN's native enum
    contractDepthOffset=0
)
self.futures_symbols.append(future.Symbol)  # Store continuous symbol

# ❌ FORBIDDEN - Individual contracts require manual rollover
contract = self.AddFutureContract("ESM2024")  # AVOID - not continuous
```

### **2. WARM-UP AWARE LIQUIDITY CHECKING**
```python
# ✅ CORRECT - Chain-based liquidity during warm-up using LEAN's FuturesChains
def get_liquid_symbols(self, slice):
    if self.IsWarmingUp:  # LEAN's native property
        return self._get_liquid_symbols_during_warmup(slice)
    else:
        return self._get_liquid_symbols_post_warmup(slice)

def _get_liquid_symbols_during_warmup(self, slice):
    """Use LEAN's FuturesChains + major contract fallback during warm-up"""
    liquid_symbols = []
    major_contracts = ['ES', 'NQ', 'YM', 'ZN', 'ZB', '6E', '6J', 'CL', 'GC']
    
    for symbol in self.futures_symbols:
        # Try LEAN's FuturesChains first
        if symbol in slice.FuturesChains:  # LEAN's native property
            chain = slice.FuturesChains[symbol]  # LEAN's native access
            if self._analyze_chain_liquidity(chain):
                liquid_symbols.append(symbol)
        # Fallback to major contract assumption
        elif self._is_major_contract(symbol, major_contracts):
            liquid_symbols.append(symbol)
    
    return liquid_symbols

# ❌ WRONG - Ignoring LEAN's chain data during warm-up
def bad_liquidity_check(self, symbol):
    return self.Securities[symbol].IsTradable  # Unreliable during warm-up
```

### **3. AUTOMATIC ROLLOVER HANDLING**
```python
# ✅ CORRECT - Handle rollover events using LEAN's SymbolChangedEvents
def OnData(self, slice):
    # Handle LEAN's automatic rollover events
    for symbol_changed in slice.SymbolChangedEvents.Values:  # LEAN's native event
        old_symbol = symbol_changed.OldSymbol  # LEAN's native property
        new_symbol = symbol_changed.NewSymbol  # LEAN's native property
        self.Log(f"Contract rollover: {old_symbol} -> {new_symbol}")
        
        # Positions automatically transfer in LEAN - track for analysis
        self._track_rollover_costs(old_symbol, new_symbol)
    
    # Use continuous symbols for strategy signals
    for symbol in self.futures_symbols:
        current_contract = self.Securities[symbol].Mapped  # LEAN's native property
        signal = self.strategy.generate_signal(symbol, slice)
```

### **4. FUTURES CHAIN CONFIGURATION**
```python
# ✅ CORRECT - Centralized futures chain config leveraging LEAN's capabilities
FUTURES_CHAIN_CONFIG = {
    'liquidity_during_warmup': {
        'enabled': True,
        'major_liquid_contracts': ['ES', 'NQ', 'YM', 'ZN', 'ZB', '6E', '6J', 'CL', 'GC'],
        'fallback_to_major_list': True,
        'min_volume_warmup': 100,
        'min_open_interest_warmup': 10000,
    },
    'post_warmup_liquidity': {
        'min_volume': 1000,
        'min_open_interest': 50000,
        'max_bid_ask_spread': 0.001,
        'use_mapped_contract': True,  # Use LEAN's Mapped property
    }
}
```

---

## 🏗️ **THREE-LAYER ARCHITECTURE**

### **Layer Structure**
```python
# Layer 1: Strategy Signal Generation - Uses LEAN's built-in indicators
class LayerOneStrategy:
    def __init__(self, algorithm, config_manager, strategy_name):
        self.config = config_manager.get_strategy_config(strategy_name)
        
        # Use LEAN's built-in indicators - NEVER custom implementations
        self.sma_fast = algorithm.SMA(symbol, self.config['sma_fast_period'])
        self.sma_slow = algorithm.SMA(symbol, self.config['sma_slow_period'])
        self.rsi = algorithm.RSI(symbol, self.config['rsi_period'])
    
    def generate_raw_signals(self, slice: Slice) -> dict:
        """Generate raw position targets using LEAN's indicator values"""
        signals = {}
        
        for symbol in self.symbols:
            # Use LEAN's indicator values - no custom calculations
            if self.sma_fast[symbol].IsReady and self.sma_slow[symbol].IsReady:
                sma_signal = 1 if self.sma_fast[symbol].Current.Value > self.sma_slow[symbol].Current.Value else -1
                rsi_filter = 1 if 30 < self.rsi[symbol].Current.Value < 70 else 0
                
                signals[symbol] = sma_signal * rsi_filter
        
        return signals

# Layer 2: Dynamic Allocation - Uses LEAN's Portfolio methods
class LayerTwoAllocator:
    def __init__(self, algorithm, config_manager):
        self.algorithm = algorithm
        self.config = config_manager.get_allocation_config()
    
    def combine_strategies(self, strategy_targets: dict) -> dict:
        """Combine multiple strategy signals using LEAN's Portfolio methods"""
        combined_targets = {}
        
        # Use LEAN's Portfolio.TotalPortfolioValue - NEVER custom tracking
        total_value = self.algorithm.Portfolio.TotalPortfolioValue
        
        for symbol in strategy_targets:
            # Use LEAN's Portfolio methods for current positions
            current_holdings = self.algorithm.Portfolio[symbol].HoldingsValue / total_value
            combined_targets[symbol] = self._calculate_target_allocation(symbol, current_holdings)
        
        return combined_targets

# Layer 3: Portfolio Risk Management - Uses LEAN's risk controls
class LayerThreeRiskManager:
    def __init__(self, algorithm, config_manager):
        self.algorithm = algorithm
        self.config = config_manager.get_risk_config()
    
    def apply_portfolio_risk(self, combined_targets: dict) -> dict:
        """Apply portfolio-level risk management using LEAN's methods"""
        risk_adjusted_targets = {}
        
        # Use LEAN's Portfolio methods - NEVER custom portfolio tracking
        total_value = self.algorithm.Portfolio.TotalPortfolioValue
        margin_used = self.algorithm.Portfolio.TotalMarginUsed
        
        # Apply LEAN's built-in risk controls
        max_leverage = self.config.get('max_leverage', 1.0)
        current_leverage = margin_used / total_value
        
        if current_leverage > max_leverage:
            # Scale down using LEAN's position sizing
            scale_factor = max_leverage / current_leverage
            for symbol, target in combined_targets.items():
                risk_adjusted_targets[symbol] = target * scale_factor
        else:
            risk_adjusted_targets = combined_targets
        
        return risk_adjusted_targets
```

---

## 📊 **QC NATIVE DATA ACCESS PATTERNS**

### **Historical Data Access**
```python
# ✅ CORRECT - QC native History() with centralized config
def get_historical_data(self, symbol, config_manager):
    """Use QC's native History with centralized config"""
    algo_config = config_manager.get_algorithm_config()
    periods = algo_config['history_periods']
    resolution = getattr(Resolution, algo_config['resolution'])  # LEAN's Resolution enum
    
    # Use LEAN's native History method - NEVER custom data fetching
    history = self.History(symbol, periods, resolution)
    return history if not history.empty else None

# ❌ WRONG - Hardcoded parameters or custom data fetching
def bad_historical_data(self, symbol):
    history = self.History(symbol, 252, Resolution.Daily)  # HARDCODED
```

### **Symbol and Price Access**
```python
# ✅ CORRECT - QC native properties
def get_current_price(self, symbol):
    """Use QC's native price properties"""
    if symbol in self.Securities:  # LEAN's native Securities collection
        security = self.Securities[symbol]  # LEAN's native Security object
        if security.HasData and security.Price > 0:  # LEAN's native properties
            return security.Price  # LEAN's native Price property
    return None

# ✅ CORRECT - Check data availability using LEAN's properties
def has_valid_data(self, symbol):
    """Check if symbol has valid, tradeable data using LEAN's properties"""
    return (symbol in self.Securities and 
            self.Securities[symbol].HasData and    # LEAN's native property
            self.Securities[symbol].IsTradable)    # LEAN's native property
```

---

## 🚨 **CRITICAL SECURITY REQUIREMENTS**

### **Mandatory Rules**
1. **LEAN-FIRST**: Always check LEAN's capabilities before custom implementation
2. **NO fallback configurations** - Algorithm stops if config invalid
3. **NO direct config access** - All through centralized methods  
4. **NO hardcoded parameters** - Everything from validated config
5. **FAIL FAST** on configuration errors
6. **COMPLETE audit trail** on startup
7. **Warm-up aware logic** - Different behavior during warm-up

### **Security Checklist**
- [ ] All components use LEAN's built-in methods where possible
- [ ] All components use `config_manager.get_*_config()` methods
- [ ] No direct config dictionary access anywhere
- [ ] Configuration validation passes completely
- [ ] Warm-up period calculated based on strategy requirements
- [ ] Futures use continuous contracts with automatic rollover
- [ ] Chain-based liquidity checking during warm-up
- [ ] No hardcoded trading parameters exist
- [ ] No custom implementations of LEAN's built-in capabilities

---

## 📁 **PROJECT STRUCTURE**
```
CTA Replication/
├── main.py                          # QC Algorithm entry point
├── config.json                      # Master configuration file
├── docs/
│   ├── lean-cheatsheet.md           # LEAN API quick reference
│   └── qc-common-patterns.md        # Common QC patterns
├── src/
│   ├── config/                      # Configuration management
│   │   ├── algorithm_config_manager.py     # Centralized config manager
│   │   ├── config.py                       # Configuration assembly
│   │   └── config_market_strategy.py       # Market strategy configs
│   ├── components/                  # Core system components
│   │   ├── three_layer_orchestrator.py     # Layer coordination
│   │   ├── portfolio_execution_manager.py  # Order execution
│   │   └── universe.py                     # Universe management
│   ├── strategies/                  # Layer 1: Strategy implementations
│   │   ├── base_strategy.py                # Strategy base class
│   │   └── kestner_cta_strategy.py         # Kestner momentum strategy
│   └── risk/                        # Layer 3: Portfolio risk management
│       └── layer_three_risk_manager.py     # Portfolio-level risk
```

This framework ensures **maximum security**, **proper QC integration**, and **LEAN-first development** for live trading.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/apabon123) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
