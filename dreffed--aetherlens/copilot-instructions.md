## aetherlens

> **AetherLens Home Edition** is an open-source cost and usage monitoring platform for home labs, smart homes, IoT

# CLAUDE.md - AI Assistant Development Guidelines

## Project Context

**AetherLens Home Edition** is an open-source cost and usage monitoring platform for home labs, smart homes, IoT
devices, and personal cloud services. This document provides context and guidelines for AI assistants (particularly
Claude and other LLMs) when contributing to this project.

## Project Philosophy

- **Community First**: Built by home lab enthusiasts, for home lab enthusiasts
- **Privacy by Default**: All data stays local unless explicitly configured otherwise
- **Security First**: Defense in depth, encrypted credentials, minimal attack surface
- **Simplicity Over Features**: Start minimal, iterate based on real user needs
- **Plugin-Driven**: Core is minimal; functionality comes from community plugins
- **Resource-Efficient**: Must run on Raspberry Pi 4 or equivalent (\<2GB RAM)
- **Observable**: Comprehensive logging and metrics for debugging
- **Learning Platform**: Great for showcasing full-stack engineering skills

## Target Audience

### Primary Users

- **Home Lab Enthusiasts** - Running Proxmox, TrueNAS, or Kubernetes clusters
- **Smart Home Power Users** - Complex Home Assistant or Node-RED setups
- **Self-Hosters** - Managing their own services (Nextcloud, Plex, etc.)
- **Crypto Miners** - Tracking profitability vs. electricity costs
- **Solar Power Owners** - Optimizing self-consumption
- **Remote Workers** - Managing home office expenses

### Secondary Users

- **Small Businesses** - Single-location monitoring
- **Apartment Dwellers** - Understanding energy costs
- **Students** - Learning about energy efficiency

## Code Standards

### Language Preferences

**Core Engine**: Python 3.11+ (FastAPI)

- Prioritize readability and AI-assisted development
- Use type hints extensively
- Async/await for all I/O operations
- Optimize hot paths in Rust only when profiling shows bottlenecks

**Plugin SDK**: Python (primary)

- Focus on single language for v1.0
- Go/Rust/TypeScript support in future phases
- Keep plugin API simple and well-documented

**Frontend**: TypeScript + React 18+

- Vite for build tooling
- Tailwind CSS for styling
- Recharts for data visualization
- Functional components with hooks

### Code Style

```python
# Python: Type hints, descriptive names, async patterns
from typing import List, Dict, Optional
from dataclasses import dataclass
import asyncio

@dataclass
class Metric:
    """Represents a single metric data point"""
    device_id: str
    timestamp: float
    metric_type: str
    value: float
    unit: str
    tags: Optional[Dict[str, str]] = None

async def collect_device_metrics(device_id: str) -> List[Metric]:
    """
    Collect metrics from a device asynchronously.
    
    Args:
        device_id: Unique identifier for the device
        
    Returns:
        List of collected metrics
        
    Raises:
        DeviceOfflineError: If device is not reachable
        TimeoutError: If collection takes too long
    """
    try:
        async with aiohttp.ClientSession() as session:
            async with session.get(f"http://{device_id}/status", timeout=5) as resp:
                data = await resp.json()
                return parse_metrics(data)
    except asyncio.TimeoutError:
        logger.error(f"Timeout collecting from {device_id}")
        raise
```

```typescript
// TypeScript: Explicit types, functional style
interface DeviceMetric {
  timestamp: number;
  deviceId: string;
  powerWatts: number;
  costPerHour: number;
}

interface CostSummary {
  totalKwh: number;
  totalCost: number;
  currency: string;
}

// Prefer const and arrow functions
const calculateDailyCost = (metrics: DeviceMetric[]): number => {
  return metrics.reduce((sum, m) => sum + (m.costPerHour * 24), 0);
};

// Use async/await for API calls
const fetchCurrentMetrics = async (): Promise<DeviceMetric[]> => {
  const response = await fetch('/api/v1/metrics/current');
  if (!response.ok) {
    throw new Error(`API error: ${response.status}`);
  }
  return response.json();
};
```

### Error Handling

Always include context in error messages and make them actionable:

```python
# ✅ Good - Actionable error with context
raise DeviceOfflineError(
    f"Cannot connect to device '{device_id}' at {ip_address}. "
    f"Please check that the device is powered on and connected to the network."
)

# ❌ Bad - Vague error
raise Exception("Connection failed")
```

Never use bare `except:` clauses:

```python
# ✅ Good
try:
    result = await fetch_data()
except (ConnectionError, TimeoutError) as e:
    logger.error(f"Network error: {e}")
    return None
except ValueError as e:
    logger.error(f"Invalid data: {e}")
    raise

# ❌ Bad
try:
    result = await fetch_data()
except:
    pass
```

## Architecture Principles

### Data Flow

```
Device/Service → Plugin → Collector → Processor → Storage → API → UI/Export
                   ↓         ↓          ↓           ↓
                 Isolate   Buffer    Enrich      Persist
```

1. **Collection**: Plugins gather metrics from devices/services
1. **Processing**: Normalize, enrich, and calculate derived metrics
1. **Storage**: Persist to TimescaleDB with retention policies
1. **Serving**: API layer serves processed data
1. **Consumption**: UI, mobile app, Grafana, Home Assistant, etc.

### Plugin Architecture

**Key Principles:**

- Plugins run in isolated processes (security)
- Simple HTTP API for communication (no gRPC complexity in v1.0)
- Plugin crashes don't affect core or other plugins
- Hot-reload capability for development

```python
# Minimal plugin interface
class BasePlugin(ABC):
    """All plugins must inherit from this base class"""
    
    @abstractmethod
    async def collect_metrics(self) -> List[Metric]:
        """Collect metrics from devices - REQUIRED"""
        pass
    
    async def discover_devices(self) -> List[Device]:
        """Discover available devices - OPTIONAL"""
        return []
    
    def validate_config(self) -> bool:
        """Validate plugin configuration - OPTIONAL"""
        return True
    
    def get_capabilities(self) -> List[str]:
        """Return plugin capabilities"""
        return ["metrics.collect"]
```

### Security Requirements

**Non-Negotiable:**

- All external connections use TLS/HTTPS
- Credentials stored in platform keyring (encrypted)
- JWT tokens for API authentication
- Input validation on all API endpoints
- SQL parameterization (prevent injection)
- Rate limiting on API endpoints

**Best Practices:**

```python
# Store credentials securely
import keyring

def store_api_key(service: str, api_key: str):
    """Store API key in platform keyring"""
    keyring.set_password("aetherlens", service, api_key)

def get_api_key(service: str) -> str:
    """Retrieve API key from keyring"""
    key = keyring.get_password("aetherlens", service)
    if not key:
        raise ValueError(f"API key not found for {service}")
    return key

# Validate API inputs
from pydantic import BaseModel, Field, validator

class DeviceCreate(BaseModel):
    """Request model for creating a device"""
    name: str = Field(..., min_length=1, max_length=100)
    type: str = Field(..., regex="^[a-z_]+$")
    ip_address: str = Field(..., regex=r"^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$")
    
    @validator('type')
    def validate_type(cls, v):
        allowed_types = ['smart_plug', 'energy_monitor', 'solar_inverter']
        if v not in allowed_types:
            raise ValueError(f"Type must be one of: {allowed_types}")
        return v
```

## Testing Requirements

### Test Coverage Targets

- **Unit Tests**: >70% coverage for core components
- **Integration Tests**: All major data flows
- **API Tests**: 100% endpoint coverage
- **Plugin Tests**: Example tests for each plugin type

### Testing Examples

```python
# tests/test_cost_calculator.py
import pytest
from datetime import datetime
from aetherlens.processing import CostCalculator, RateSchedule

@pytest.fixture
def peak_rate_schedule():
    """Fixture providing a time-of-use rate schedule"""
    return RateSchedule(
        peak_rate=0.42,
        off_peak_rate=0.24,
        super_off_peak_rate=0.12,
        peak_hours="16:00-21:00",
        peak_days=["monday", "tuesday", "wednesday", "thursday", "friday"]
    )

def test_peak_hour_cost_calculation(peak_rate_schedule):
    """Test cost calculation during peak hours"""
    calculator = CostCalculator(peak_rate_schedule)
    
    # 6 PM on a Wednesday
    timestamp = datetime(2025, 1, 15, 18, 0).timestamp()
    power_watts = 1000
    duration_hours = 1
    
    cost = calculator.calculate_cost(
        power_watts=power_watts,
        duration_hours=duration_hours,
        timestamp=timestamp
    )
    
    # 1 kWh * $0.42/kWh = $0.42
    assert cost.amount == pytest.approx(0.42)
    assert cost.rate_period == "peak"
    assert cost.rate_applied == 0.42

def test_weekend_off_peak_calculation(peak_rate_schedule):
    """Test cost calculation on weekend (super off-peak)"""
    calculator = CostCalculator(peak_rate_schedule)
    
    # 6 PM on a Saturday
    timestamp = datetime(2025, 1, 18, 18, 0).timestamp()
    power_watts = 1000
    duration_hours = 1
    
    cost = calculator.calculate_cost(
        power_watts=power_watts,
        duration_hours=duration_hours,
        timestamp=timestamp
    )
    
    # 1 kWh * $0.12/kWh = $0.12
    assert cost.amount == pytest.approx(0.12)
    assert cost.rate_period == "super_off_peak"

@pytest.mark.asyncio
async def test_collect_with_timeout():
    """Test that collection respects timeout"""
    collector = DataCollector(timeout=2)
    
    with pytest.raises(asyncio.TimeoutError):
        await collector.collect_from_device("slow-device")
```

### Integration Test Example

```python
# tests/integration/test_plugin_lifecycle.py
import pytest
from aetherlens import PluginManager, DataCollector

@pytest.mark.integration
@pytest.mark.asyncio
async def test_end_to_end_collection():
    """Test complete flow from plugin to database"""
    # Setup
    manager = PluginManager()
    collector = DataCollector()
    
    # Load test plugin
    plugin = await manager.load_plugin("test_plugin", config={
        "devices": [{"id": "test-device", "type": "mock"}]
    })
    
    # Collect metrics
    metrics = await plugin.collect_metrics()
    assert len(metrics) > 0
    
    # Process and store
    await collector.collect_batch(metrics)
    
    # Verify storage
    from_db = await db.query(
        "SELECT * FROM metrics WHERE device_id = 'test-device' ORDER BY time DESC LIMIT 1"
    )
    assert from_db is not None
    assert from_db.device_id == "test-device"
    
    # Cleanup
    await manager.unload_plugin("test_plugin")
```

## Documentation Standards

### Code Comments

```python
def calculate_energy_cost(
    power_watts: float,
    duration_hours: float,
    rate_per_kwh: float
) -> float:
    """
    Calculate energy cost for a given power consumption.
    
    Args:
        power_watts: Power consumption in watts
        duration_hours: Duration of consumption in hours
        rate_per_kwh: Electricity rate in currency per kilowatt-hour
    
    Returns:
        Total cost in the same currency as rate_per_kwh
        
    Example:
        >>> calculate_energy_cost(1000, 1, 0.24)
        0.24  # 1 kWh at $0.24/kWh
        
    Note:
        This is a simplified calculation. For actual cost calculation,
        use CostCalculator which handles time-of-use rates.
    """
    energy_kwh = (power_watts * duration_hours) / 1000
    return energy_kwh * rate_per_kwh
```

### API Documentation

All API endpoints must have:

- Clear description
- Request/response examples
- Error responses
- Rate limits (if applicable)

````python
@app.get("/api/v1/devices/{device_id}/metrics", 
         response_model=MetricsResponse,
         summary="Get device metrics")
async def get_device_metrics(
    device_id: str = Path(..., description="Unique device identifier"),
    hours: int = Query(24, ge=1, le=720, description="Hours of history to retrieve"),
    resolution: str = Query("auto", regex="^(raw|1m|5m|1h|auto)$"),
    credentials: HTTPAuthorizationCredentials = Depends(security)
):
    """
    Get historical metrics for a specific device.
    
    Returns power consumption, energy totals, and cost data for the
    specified time range. Resolution is automatically adjusted based
    on the time range unless explicitly specified.
    
    **Rate Limit:** 60 requests per minute per token
    
    **Example Response:**
    ```json
    {
      "device_id": "shelly-office-01",
      "period": {
        "start": "2025-01-14T10:00:00Z",
        "end": "2025-01-15T10:00:00Z",
        "resolution": "5m"
      },
      "data": [
        {
          "timestamp": "2025-01-14T10:00:00Z",
          "power_watts": 125.4,
          "energy_kwh": 0.125,
          "cost": 0.03
        }
      ]
    }
    ```
    
    **Errors:**
    - 404: Device not found
    - 401: Invalid or expired token
    - 429: Rate limit exceeded
    """
    ...
````

## Common Patterns

### Async Collection Pattern

```python
class ShellyPlugin(BasePlugin):
    """Plugin for Shelly smart devices"""
    
    async def collect_metrics(self) -> List[Metric]:
        """Collect metrics from all configured Shelly devices"""
        tasks = []
        
        # Create tasks for all devices
        for device in self.config['devices']:
            task = self._collect_from_device(device)
            tasks.append(task)
        
        # Execute concurrently with timeout
        try:
            results = await asyncio.gather(*tasks, return_exceptions=True)
        except Exception as e:
            logger.error(f"Batch collection failed: {e}")
            return []
        
        # Flatten results, filtering out errors
        metrics = []
        for result in results:
            if isinstance(result, Exception):
                logger.error(f"Device collection failed: {result}")
                continue
            metrics.extend(result)
        
        return metrics
    
    async def _collect_from_device(self, device: Dict) -> List[Metric]:
        """Collect metrics from a single device"""
        async with aiohttp.ClientSession() as session:
            try:
                async with session.get(
                    f"http://{device['ip']}/status",
                    timeout=aiohttp.ClientTimeout(total=5)
                ) as response:
                    data = await response.json()
                    return self._parse_metrics(device['id'], data)
            except asyncio.TimeoutError:
                raise DeviceOfflineError(f"Timeout connecting to {device['id']}")
            except aiohttp.ClientError as e:
                raise DeviceOfflineError(f"Network error for {device['id']}: {e}")
```

### Retry with Exponential Backoff

```python
from tenacity import (
    retry,
    stop_after_attempt,
    wait_exponential,
    retry_if_exception_type
)

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10),
    retry=retry_if_exception_type((ConnectionError, TimeoutError))
)
async def fetch_with_retry(url: str) -> Dict:
    """
    Fetch data with automatic retry on transient failures.
    
    Retries up to 3 times with exponential backoff:
    - 1st retry: 2 seconds
    - 2nd retry: 4 seconds
    - 3rd retry: 8 seconds
    """
    async with aiohttp.ClientSession() as session:
        async with session.get(url, timeout=10) as response:
            response.raise_for_status()
            return await response.json()
```

### Circuit Breaker Pattern

```python
from datetime import datetime, timedelta

class CircuitBreaker:
    """Circuit breaker to prevent cascading failures"""
    
    def __init__(self, failure_threshold: int = 5, timeout: int = 60):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.last_failure_time = None
        self.state = "closed"  # closed, open, half_open
    
    async def call(self, func, *args, **kwargs):
        """Execute function with circuit breaker protection"""
        if self.state == "open":
            if datetime.now() - self.last_failure_time > timedelta(seconds=self.timeout):
                self.state = "half_open"
                logger.info("Circuit breaker entering half-open state")
            else:
                raise CircuitBreakerOpenError(
                    f"Circuit breaker is open. Will retry after "
                    f"{self.timeout - (datetime.now() - self.last_failure_time).seconds}s"
                )
        
        try:
            result = await func(*args, **kwargs)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise
    
    def on_success(self):
        """Reset circuit breaker on successful call"""
        self.failure_count = 0
        if self.state == "half_open":
            self.state = "closed"
            logger.info("Circuit breaker closed after successful call")
    
    def on_failure(self):
        """Increment failure count and potentially open circuit"""
        self.failure_count += 1
        self.last_failure_time = datetime.now()
        
        if self.failure_count >= self.failure_threshold:
            self.state = "open"
            logger.error(
                f"Circuit breaker opened after {self.failure_count} failures. "
                f"Will retry in {self.timeout} seconds."
            )
```

## Development Workflow

### Feature Development

1. **Create Issue** - Describe the feature with user stories
1. **Write Tests First** (TDD) - Define expected behavior
1. **Implement Minimal Solution** - Make tests pass
1. **Refactor for Clarity** - Improve code quality
1. **Update Documentation** - API docs, README, examples
1. **Submit PR** - With tests passing and docs updated

### Bug Fixes

1. **Write Test Reproducing Bug** - Proves the bug exists
1. **Fix Bug** - Minimal changes to make test pass
1. **Verify No Regression** - Run full test suite
1. **Update CHANGELOG** - Document the fix

### Plugin Development

1. **Use Plugin Template** - Start with example plugin
1. **Implement Required Methods** - `collect_metrics()` at minimum
1. **Add Configuration Schema** - Define in `plugin.yaml`
1. **Write Tests** - Unit tests for plugin logic
1. **Document in README** - Installation and configuration

## Performance Guidelines

### Memory Management

```python
# ✅ Good - Stream data, don't load all in memory
async def stream_large_dataset():
    async for batch in fetch_data_in_batches(batch_size=1000):
        await process_batch(batch)

# ❌ Bad - Loads everything in memory
async def load_all_data():
    data = await fetch_all_data()  # Could be millions of records
    return process_data(data)
```

### Database Optimization

```python
# ✅ Good - Batch inserts
async def batch_insert_metrics(metrics: List[Metric], batch_size: int = 1000):
    """Insert metrics in batches for better performance"""
    for i in range(0, len(metrics), batch_size):
        batch = metrics[i:i + batch_size]
        await db.executemany(
            """
            INSERT INTO metrics (time, device_id, metric_type, value, unit, tags)
            VALUES ($1, $2, $3, $4, $5, $6)
            """,
            [(m.timestamp, m.device_id, m.metric_type, m.value, m.unit, m.tags)
             for m in batch]
        )

# ❌ Bad - Individual inserts
async def insert_metrics_one_by_one(metrics: List[Metric]):
    for metric in metrics:
        await db.execute(
            "INSERT INTO metrics (...) VALUES (...)",
            metric.timestamp, metric.device_id, ...
        )
```

### Caching Strategy

```python
from functools import lru_cache
import aiocache

# Cache expensive calculations
@lru_cache(maxsize=1000)
def get_device_config(device_id: str) -> Dict:
    """Cache device configuration in memory"""
    return db.query("SELECT * FROM devices WHERE device_id = ?", device_id)

# Async cache with TTL
cache = aiocache.Cache(aiocache.Cache.MEMORY)

@cache.cached(ttl=300)  # 5 minutes
async def get_current_electricity_rate() -> float:
    """Cache current rate to avoid repeated calculations"""
    schedule = await db.query("SELECT * FROM rate_schedules WHERE active = true")
    return calculate_current_rate(schedule, datetime.now())
```

## Debugging Helpers

### Structured Logging

```python
import structlog

logger = structlog.get_logger()

# Log with context
logger.info(
    "metric_collected",
    plugin="shelly",
    device_id="shelly-office-01",
    metric_type="power",
    value=125.4,
    duration_ms=45
)

# Output (JSON):
# {
#   "event": "metric_collected",
#   "plugin": "shelly",
#   "device_id": "shelly-office-01",
#   "metric_type": "power",
#   "value": 125.4,
#   "duration_ms": 45,
#   "timestamp": "2025-01-15T10:30:00.123Z",
#   "level": "info"
# }
```

### Metrics Exposure

```python
from prometheus_client import Counter, Histogram, Gauge

# Define metrics
metrics_collected = Counter(
    'aetherlens_metrics_collected_total',
    'Total metrics collected',
    ['plugin', 'device_type']
)

collection_duration = Histogram(
    'aetherlens_collection_duration_seconds',
    'Time spent collecting metrics',
    ['plugin']
)

active_devices = Gauge(
    'aetherlens_active_devices',
    'Number of active devices',
    ['type']
)

# Use in code
with collection_duration.labels(plugin='shelly').time():
    metrics = await collect_metrics()
    metrics_collected.labels(plugin='shelly', device_type='smart_plug').inc(len(metrics))
```

## Common Issues & Solutions

### Issue: High Memory Usage

**Symptoms:** Memory usage growing over time\
**Causes:** Memory leaks, unbounded caches, retained references

**Solutions:**

```python
# Check for memory leaks
import tracemalloc
tracemalloc.start()

# ... run your code ...

snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')
for stat in top_stats[:10]:
    print(stat)

# Set cache size limits
@lru_cache(maxsize=100)  # Limit cache size
def expensive_operation():
    ...

# Clear caches periodically
import gc
gc.collect()  # Force garbage collection
```

### Issue: Plugin Crashes

**Symptoms:** Plugin repeatedly failing, high error rates

**Solutions:**

```python
# Add health checks
class PluginHealthMonitor:
    def __init__(self, plugin, max_failures=3):
        self.plugin = plugin
        self.failure_count = 0
        self.max_failures = max_failures
    
    async def check_health(self):
        try:
            health = await self.plugin.get_health()
            if health['status'] != 'healthy':
                self.failure_count += 1
            else:
                self.failure_count = 0
            
            if self.failure_count >= self.max_failures:
                logger.error(f"Plugin {self.plugin} unhealthy, restarting")
                await self.restart_plugin()
        except Exception as e:
            logger.error(f"Health check failed: {e}")
            self.failure_count += 1
```

### Issue: Slow API Responses

**Symptoms:** API latency >1 second

**Solutions:**

```python
# Profile slow endpoints
import time
from functools import wraps

def profile_endpoint(func):
    @wraps(func)
    async def wrapper(*args, **kwargs):
        start = time.time()
        result = await func(*args, **kwargs)
        duration = time.time() - start
        
        if duration > 0.5:
            logger.warning(f"Slow endpoint: {func.__name__} took {duration:.2f}s")
        
        return result
    return wrapper

# Add caching
@cache.cached(ttl=60)
async def get_current_metrics():
    return await db.query("SELECT * FROM metrics WHERE ...")

# Use database indexes
# See SCHEMA.md for index recommendations
```

## Questions to Ask When Developing

1. **Will this run on a Raspberry Pi?** (Memory/CPU constraints)
1. **Is this the simplest solution?** (Avoid over-engineering)
1. **What happens when the network fails?** (Graceful degradation)
1. **How does this impact privacy?** (Local-first principle)
1. **Is the error message actionable?** (Help users fix issues)
1. **Can this be unit tested?** (Testability)
1. **Will users understand this configuration?** (Usability)
1. **Is there a security risk?** (Defense in depth)

## Red Flags to Avoid

- ❌ Hardcoded credentials
- ❌ Synchronous blocking calls in async code
- ❌ Unbounded memory growth
- ❌ Tight coupling between components
- ❌ Missing error handling
- ❌ SQL injection vulnerabilities
- ❌ Plaintext sensitive data
- ❌ Race conditions in concurrent code
- ❌ Ignoring timeout scenarios
- ❌ Swallowing exceptions without logging

## Version Compatibility

- **Semantic Versioning**: MAJOR.MINOR.PATCH
- **Backwards Compatibility**: Maintain in minor versions
- **Deprecation Policy**: Warn for one major version before removal
- **Plugin API**: Versioned independently of core

## Resources

- [Project Wiki](https://github.com/aetherlens/home/wiki)
- [Plugin Development Guide](./docs/PLUGIN_GUIDE.md)
- [API Reference](./docs/API.md)
- [Contributing Guidelines](./CONTRIBUTING.md)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [TimescaleDB Documentation](https://docs.timescale.com/)
- [Python AsyncIO Guide](https://docs.python.org/3/library/asyncio.html)

## AI Assistant Checklist

When generating code for this project, ensure:

- [ ] Type hints on all function signatures
- [ ] Docstrings with examples for public APIs
- [ ] Async/await for all I/O operations
- [ ] Comprehensive error handling with context
- [ ] Input validation using Pydantic
- [ ] Security: No hardcoded secrets, use keyring
- [ ] Tests alongside implementation
- [ ] Structured logging with context
- [ ] Resource efficiency (memory, CPU, network)
- [ ] Privacy: No external calls without explicit consent
- [ ] Documentation: Update README/API docs as needed
- [ ] Consider plugin isolation and fault tolerance

## Development Philosophy

This is a **learning platform** and **community project**:

1. **Teach Through Code** - Write exemplary code that others can learn from
1. **Document Decisions** - Explain why, not just what
1. **Welcome Contributions** - Make it easy for newcomers to contribute
1. **Iterate Quickly** - Ship working code, refactor later
1. **Listen to Users** - Build what people actually need
1. **Stay Humble** - There's always a better way

Remember: **Every line of code should reflect our values of privacy, security, simplicity, and community.**

______________________________________________________________________

**This document is a living guide.** Update it as patterns emerge and lessons are learned. When in doubt, ask the
community on Discord or GitHub Discussions.

---
> Source: [Dreffed/aetherlens](https://github.com/Dreffed/aetherlens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
