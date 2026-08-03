## monkeyattack-gold-pivot-ea

> - **Name**: Chris Meredith

# CLAUDE.md - Claude Code Configuration & Preferences

## Developer Profile
- **Name**: Chris Meredith
- **Primary Focus**: MetaTrader 4/5 Expert Advisors, Forex Trading Systems
- **GitHub Username**: Monkeyattack
- **GitHub Access**: Personal Access Token (configured)

## Version Control Standards

### Version Numbering Format
```
#property version   "X.YZ"
```
- **X** = Major release (breaking changes, complete rewrites)
- **Y** = Minor release (new features, significant enhancements)  
- **Z** = Patch release (bug fixes, small improvements)

### Version Control Rules
1. **ALWAYS increment version** when making ANY code changes
2. Update initialization messages with new version number
3. Document changes in code comments or separate changelog
4. Examples:
   - Bug fix: 2.01 → 2.02
   - New feature: 2.02 → 2.10
   - Major rewrite: 2.10 → 3.00

### Version History Template
```cpp
/*
Version History:
v1.00 - Initial release with basic functionality
v1.01 - Fixed compilation errors, improved error handling
v1.10 - Added visual indicators and enhanced risk management
v2.00 - Complete strategy overhaul with institutional pivot focus
v2.01 - Enhanced with trade quality scoring and consecutive loss protection
*/
```

## MetaTrader Development Preferences

### Platform Standards
- **Primary Platform**: MetaTrader 4 (MT4)
- **Language**: MQL4 (strict mode)
- **File Structure**: Standard MT4 Experts folder structure

### Code Quality Standards
1. **Always use `#property strict`** for MT4 compatibility
2. **Use `input` parameters** instead of `extern` for modern MT4
3. **Check return values** for OrderModify, OrderSend operations
4. **Proper error handling** with GetLastError() logging
5. **MT4-compatible syntax** - avoid MT5-specific functions

### Trading Strategy Focus
- **Primary Strategy**: Institutional Pivot Points (proven +$7,448.54 profit, 49% ROI)
- **Risk Management**: Critical priority over entry signals
- **Session Focus**: London-NY overlap (13:00-16:00 GMT) for optimal performance
- **Quality over Quantity**: Selective high-quality trades preferred

## Code Architecture Preferences

### Function Organization
```cpp
// Standard EA Structure:
1. Input parameters
2. Global variables  
3. OnInit() function
4. OnDeinit() function
5. OnTick() function
6. Core trading functions
7. Risk management functions
8. Utility/helper functions
9. Enhanced features (quality scoring, etc.)
```

### Visual Elements Standards
- **Always include visual indicators** when applicable
- **Settings panel overlay** for monitoring
- **Real-time status display** (risk mode, quality scores, etc.)
- **Color coding**: Green=Buy, Red=Sell, Yellow=Neutral, Orange=Warning

## Risk Management Standards

### Core Principles
1. **Risk management > Entry signals** (learned from backtesting)
2. **Consecutive loss protection** - reduce risk after 3+ losses
3. **Volatility-adjusted position sizing** - reduce size in high volatility
4. **Maximum drawdown tolerance**: ~37% acceptable for high returns
5. **Quality filtering** - only trade high-probability setups

### Default Risk Parameters
```cpp
input double RiskPercent = 1.5;              // Risk per trade
input double MaxSpread = 7.0;                // Maximum spread in points
input double RiskRewardRatio = 2.5;          // Proven optimal ratio
input int MaxConsecutiveLosses = 3;          // Before risk reduction
input double RiskReductionFactor = 0.5;      // 50% reduction after losses
```

## Development Workflow

### Before Making Changes
1. Read the current version number
2. Understand the existing functionality
3. Plan the enhancement/fix
4. Determine appropriate version increment

### After Making Changes
1. Update version number
2. Update initialization messages
3. Test compilation
4. Document changes
5. Verify all functions work as expected

### Testing Standards
1. **Always compile after changes** - verify 0 errors, 0 warnings
2. **Test on demo account first** before live deployment
3. **Monitor performance** vs. backtesting expectations
4. **Keep detailed logs** of trade quality and performance

## Communication Preferences

### Code Explanations
- **Be concise and direct** - avoid unnecessary preamble
- **Focus on specific issues** - don't explain obvious functionality
- **Provide exact line numbers** when referencing code issues
- **Use technical terminology** appropriately for trading/programming

### Problem Solving Approach
1. **Identify the specific issue** first
2. **Check for MT4/MT5 compatibility** problems
3. **Fix compilation errors** before functionality
4. **Test incrementally** rather than large changes
5. **Preserve proven profitable settings** during modifications

## Project Structure Standards

### File Naming Convention
- Main EA: `ProjectName_EA.mq4`
- Settings: `ProjectName_Settings.set`  
- Documentation: `ProjectName_Manual.txt`
- Always include version in major releases

### Folder Organization
```
Project Root/
├── MQL4/
│   ├── Experts/
│   │   ├── MainEA.mq4
│   │   └── CLAUDE.md
│   ├── Presets/
│   │   └── SettingsFiles.set
│   └── Documentation/
│       ├── Manual.txt
│       └── Backtesting_Results.pdf
```

## Integration Notes

### GitHub Integration
- **Username**: Monkeyattack
- **Access Method**: Personal Access Token (PAT)
- **Repository Naming**: `project-name-ea` or `monkeyattack-[strategy]-ea`
- **Commit Conventions**: 
  - `feat: add new feature (v2.01)`
  - `fix: resolve compilation error (v2.01)`
  - `enhance: improve risk management (v2.10)`
  - `docs: update documentation`
- **Branch Strategy**: 
  - `main` - stable releases
  - `develop` - active development
  - `feature/[name]` - new features
- **Release Tags**: Match version numbers (v2.01, v2.10, etc.)

### Compilation Commands
- **MetaEditor**: F7 (Compile)
- **Target**: Verify 0 errors, 0 warnings before deployment
- **Testing**: Always test on demo account first

---

## Quick Reference Commands

### Version Update Checklist
- [ ] Increment `#property version`
- [ ] Update initialization Print() message
- [ ] Document changes in comments
- [ ] Test compilation
- [ ] Verify functionality

### Common File Paths
- **MT4 Experts**: `C:\Users\cmeredith\AppData\Roaming\MetaQuotes\Terminal\[TERMINAL_ID]\MQL4\Experts\`
- **Settings**: `C:\Users\cmeredith\AppData\Roaming\MetaQuotes\Terminal\[TERMINAL_ID]\MQL4\Presets\`

---

*This file should be updated as new preferences and standards are discovered through development work.*

---
> Source: [Monkeyattack/monkeyattack-gold-pivot-ea](https://github.com/Monkeyattack/monkeyattack-gold-pivot-ea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
