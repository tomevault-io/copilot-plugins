## xk6-edi

> **Purpose:** JavaScript interface for k6 test scripts


# xk6-edi Subsystem Architecture

## Core Subsystems Following Grafana Patterns

### 1. API Layer Subsystem
**Purpose:** JavaScript interface for k6 test scripts
**Location:** `api/`
**Key Components:**
- `edi.go` - Main module registration and JavaScript bindings
- `results.go` - Transaction result structures and serialization
- `config.go` - Configuration parsing and validation
- `metrics.go` - k6 metrics integration and custom counters

**Design Patterns:**
- Facade pattern for simplified JavaScript API
- Builder pattern for complex configuration objects
- Observer pattern for metrics collection

### 2. Provider Registry Subsystem
**Purpose:** Pluggable transaction generators with unified interface
**Location:** `providers/`
**Key Components:**
- `registry.go` - Provider registration and discovery
- `interface.go` - TransactionProvider interface definition
- `base.go` - Common provider functionality and utilities
- Transaction-specific providers: `837p.go`, `835.go`, `270.go`, etc.

**Design Patterns:**
- Strategy pattern for different transaction types
- Factory pattern for provider instantiation
- Template method pattern for common generation workflows

### 3. Validation Engine Subsystem
**Purpose:** Multi-level EDI validation with detailed error reporting
**Location:** `validation/`
**Key Components:**
- `engine.go` - Core validation orchestration
- `rules/` - Transaction-specific validation rule sets
- `errors.go` - Structured error types and formatting
- `levels.go` - Validation level configuration (strict/moderate/lenient)

**Design Patterns:**
- Chain of responsibility for validation pipeline
- Visitor pattern for segment validation
- Command pattern for validation rule execution

### 4. Code Store Management Subsystem
**Purpose:** Healthcare code set loading, caching, and validation
**Location:** `codes/`
**Key Components:**
- `store.go` - Unified code store interface
- `icd10.go` - ICD-10-CM diagnosis codes
- `hcpcs.go` - HCPCS Level II procedure codes
- `npi.go` - National Provider Identifier registry
- `cache.go` - Efficient code lookup caching

**Design Patterns:**
- Repository pattern for code data access
- Proxy pattern for cached code lookups
- Adapter pattern for different code set formats

### 5. Template Engine Subsystem
**Purpose:** X12 segment and envelope generation
**Location:** `templates/`
**Key Components:**
- `segments.go` - X12 segment builders and formatters
- `envelopes.go` - ISA/GS/GE/IEA envelope management
- `elements.go` - Data element validation and formatting
- `separators.go` - X12 delimiter and separator handling

**Design Patterns:**
- Builder pattern for complex segment construction
- Composite pattern for nested segment structures
- Template method pattern for envelope generation

### 6. Data Generation Subsystem
**Purpose:** Realistic, deterministic test data creation
**Location:** `generators/`
**Key Components:**
- `demographics.go` - Patient and provider demographic data
- `financial.go` - Claim amounts, dates, and financial elements
- `clinical.go` - Diagnosis, procedure, and clinical data
- `seeding.go` - Deterministic random number generation

**Design Patterns:**
- Factory pattern for different data types
- Prototype pattern for data template cloning
- State pattern for maintaining generation context

## Subsystem Integration Architecture

### Inter-Subsystem Communication
```
API Layer
    ↓ (configuration)
Provider Registry
    ↓ (transaction request)
Data Generation ← → Template Engine
    ↓ (raw transaction)
Validation Engine ← → Code Store Management
    ↓ (validated result)
API Layer (metrics collection)
```

### Dependency Management
- **API Layer** depends on Provider Registry and Metrics
- **Provider Registry** depends on Data Generation, Template Engine, and Validation Engine
- **Validation Engine** depends on Code Store Management
- **Template Engine** has no external dependencies (pure functions)
- **Data Generation** depends on Code Store Management for realistic data
- **Code Store Management** has no internal dependencies (external data only)

### Error Propagation Strategy
1. **Code Store Management** → Initialization errors (missing/invalid code sets)
2. **Data Generation** → Data consistency errors (invalid code references)
3. **Template Engine** → Format errors (invalid X12 structure)
4. **Validation Engine** → Compliance errors (HIPAA 5010 violations)
5. **Provider Registry** → Configuration errors (invalid provider setup)
6. **API Layer** → User-facing errors (JavaScript exceptions)

## Configuration Management

### Hierarchical Configuration Structure
```yaml
# Global extension settings
extension:
  version: "1.0.0"
  cache_size: 10000
  validation_level: "moderate"

# Code store configuration
codes:
  icd10_source: "https://www.cdc.gov/icd/icd10cm/"
  hcpcs_source: "https://www.cms.gov/Medicare/Coding/HCPCSReleaseCodeSets/"
  npi_source: "https://npiregistry.cms.hhs.gov/"
  update_interval: "24h"

# Provider-specific settings
providers:
  837p:
    default_claim_count: 100
    include_secondary_insurance: true
  835:
    payment_method: "check"
    adjustment_codes: ["CO", "PR", "OA"]
```

### Configuration Validation Pipeline
1. **Schema Validation** - JSON/YAML structure compliance
2. **Business Rule Validation** - Healthcare-specific constraints
3. **Dependency Validation** - Required code sets and providers
4. **Performance Validation** - Resource usage limits

## Performance Optimization Strategies

### Caching Layers
- **L1 Cache:** In-memory code lookups (sync.Map)
- **L2 Cache:** Persistent code store cache (file-based)
- **L3 Cache:** Generated template cache (segment reuse)

### Parallel Processing
- **Provider Level:** Multiple transaction types simultaneously
- **Batch Level:** Parallel generation within single transaction type
- **Validation Level:** Concurrent segment validation

### Memory Management
- **Object Pooling:** sync.Pool for frequently allocated structures
- **Streaming Processing:** Process large batches without full memory load
- **Garbage Collection Tuning:** Minimize allocation pressure

## Monitoring and Observability

### Metrics Collection Points
- **Generation Metrics:** Transactions/second, error rates, latency
- **Validation Metrics:** Rule execution time, failure patterns
- **Code Store Metrics:** Cache hit rates, update frequencies
- **Resource Metrics:** Memory usage, goroutine counts

### Logging Strategy
- **Structured Logging:** JSON format with consistent field names
- **Log Levels:** DEBUG (development), INFO (operations), WARN (issues), ERROR (failures)
- **Context Propagation:** Request IDs and correlation tracking
- **Sensitive Data Filtering:** No PHI in logs, masked identifiers only

## Testing Strategy by Subsystem

### Unit Testing Approach
- **API Layer:** Mock providers and validate JavaScript bindings
- **Provider Registry:** Test provider registration and discovery
- **Validation Engine:** Comprehensive rule testing with edge cases
- **Code Store Management:** Mock external data sources
- **Template Engine:** X12 format compliance testing
- **Data Generation:** Deterministic output verification

### Integration Testing Approach
- **End-to-End Workflows:** Complete transaction generation cycles
- **Cross-Subsystem Communication:** Interface contract validation
- **Performance Testing:** Load testing with realistic scenarios
- **Compliance Testing:** HIPAA 5010 validation with official test cases

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/copyleftdev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
