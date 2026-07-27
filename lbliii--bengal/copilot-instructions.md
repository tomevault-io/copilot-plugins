## bengal

> Bengal follows strict separation of concerns: **models are passive data containers**, **orchestrators handle all operations**.

# Architecture Patterns

Bengal follows strict separation of concerns: **models are passive data containers**, **orchestrators handle all operations**.

**Works with**: `modules/types-as-contracts`, `modules/evidence-handling`

---

## The Model/Orchestrator Split

### Models (`bengal/core/`) - PASSIVE

Models are **data structures only**:

```python
# ✅ CORRECT - Models hold data, no I/O
@dataclass
class Page:
    source_path: Path
    content: str
    metadata: dict[str, Any]

    @property
    def title(self) -> str:
        return self.metadata.get("title", "Untitled")
```

Models **MUST NOT**:
- Log messages
- Perform file I/O
- Make network requests
- Have side effects
- Access global state

### Orchestrators (`bengal/orchestration/`) - ACTIVE

Orchestrators **handle all operations**:

```python
# ✅ CORRECT - Orchestrator handles I/O and logging
class RenderOrchestrator:
    @staticmethod
    def render_pages(site: Site) -> None:
        """Render all pages in the site."""
        logger.info(f"Rendering {len(site.pages)} pages")
        for page in site.pages:
            html = TemplateEngine.render(page)
            output_path = site.output_dir / page.url
            output_path.write_text(html)
```

---

## Delegation Pattern

Models delegate operations to orchestrators:

```python
# In bengal/core/site.py
class Site:
    def build(self) -> None:
        """Build the site (delegates to orchestrator)."""
        return BuildOrchestrator.build(self)

    def discover_content(self) -> None:
        """Discover content (delegates to orchestrator)."""
        return ContentOrchestrator.discover(self)
```

**Why?**
- Models remain testable without I/O mocking
- Operations are centralized and consistent
- Clear separation of data and behavior

---

## Subsystem Responsibilities

```
bengal/
├── core/              # Data models (no I/O, no logging)
│   ├── site.py        # Site container
│   ├── page/          # Page model (package - >400 lines)
│   ├── section.py     # Section/directory model
│   ├── asset/         # Asset model
│   └── theme/         # Theme resolution
│
├── orchestration/     # Build operations (all I/O here)
│   ├── build_orchestrator.py
│   ├── render_orchestrator.py
│   ├── content_orchestrator.py
│   └── asset_orchestrator.py
│
├── rendering/         # Template and content rendering
├── discovery/         # Content/asset discovery
├── cache/             # Caching infrastructure
├── health/            # Validation and health checks
└── cli/               # Command-line interface
```

---

## Composition Over Inheritance

Use mixins instead of deep inheritance:

```python
# ✅ CORRECT - Composition with focused mixins
@dataclass
class Page(
    PageMetadataMixin,      # Metadata access
    PageNavigationMixin,    # URL/navigation helpers
    PageComputedMixin,      # Computed properties
    PageRelationshipsMixin, # Parent/children/siblings
):
    """Page combines focused mixins."""
    core: PageCore
    content: str
    rendered_html: str | None = None
```

**Not this**:

```python
# ❌ WRONG - Deep inheritance
class BasePage: pass
class ContentPage(BasePage): pass
class BlogPage(ContentPage): pass
class ArticlePage(BlogPage): pass  # Too deep!
```

---

## Single Responsibility

Each class has **one clear purpose**:

| Class | Responsibility |
|-------|----------------|
| `Site` | Root data container for a build |
| `Page` | Represents one content file |
| `Section` | Represents content directory |
| `BuildOrchestrator` | Coordinates build phases |
| `RenderOrchestrator` | Handles page rendering |
| `ContentOrchestrator` | Discovers content |

---

## File Size Threshold (400 Lines)

When a file exceeds **400 lines**, convert to a package:

```
# Before (450 lines)
bengal/core/page.py

# After (converted to package)
bengal/core/page/
├── __init__.py          # Main Page class (~50 lines)
├── page_core.py         # PageCore (~200 lines)
├── metadata.py          # PageMetadataMixin (~80 lines)
├── navigation.py        # PageNavigationMixin (~60 lines)
├── computed.py          # PageComputedMixin (~100 lines)
└── proxy.py             # PageProxy (~150 lines)
```

---

## Common Patterns

### Strategy Pattern

```python
class ContentStrategy(ABC):
    @abstractmethod
    def get_template(self, page: Page) -> str: ...

class BlogStrategy(ContentStrategy):
    def get_template(self, page: Page) -> str:
        return 'blog/post.html'

class DocStrategy(ContentStrategy):
    def get_template(self, page: Page) -> str:
        return 'docs/page.html'
```

### Registry Pattern

```python
class ContentTypeRegistry:
    _strategies: dict[str, ContentStrategy] = {}

    @classmethod
    def register(cls, name: str, strategy: ContentStrategy) -> None:
        cls._strategies[name] = strategy

    @classmethod
    def get(cls, name: str) -> ContentStrategy:
        return cls._strategies[name]
```

### Builder Pattern

```python
builder = MenuBuilder('main')
builder.add_from_config(items)
builder.add_from_pages(pages)
menu = builder.build_hierarchy()
```

---

## Data Flow

### Explicit State Management

```python
# ✅ CORRECT - Pass state explicitly
def render_page(page: Page, context: BuildContext) -> str:
    return template.render(page=page, site=context.site)

# ❌ WRONG - Hidden global state
_current_site = None  # Global mutable state

def render_page(page: Page) -> str:
    global _current_site
    return template.render(page=page, site=_current_site)
```

### Single Source of Truth

- **Site** is the root data container
- **Cache** stores paths, not object references
- References reconstructed each build

---

## God Object Warning Signs

A class may be a "God object" if it has:

- More than **400 lines**
- More than **10 public methods**
- Imports from **>5 different modules**
- Does **multiple unrelated things**

**Solution**:
1. Extract mixins for different concerns
2. Delegate operations to specialized classes
3. Use composition instead of adding methods

---

## Validation Checklist

When reviewing code:

- [ ] **Models have no I/O** - No file operations in `bengal/core/`
- [ ] **Models don't log** - No `logger.info()` in models
- [ ] **Orchestrators handle operations** - All I/O in `bengal/orchestration/`
- [ ] **Composition over inheritance** - Mixins, not deep hierarchies
- [ ] **Single responsibility** - Each class does one thing
- [ ] **File size < 400 lines** - Convert to package if exceeded

---

## Quick Reference

| Pattern | Use When |
|---------|----------|
| Delegation | Model needs to trigger operation |
| Strategy | Multiple implementations of behavior |
| Registry | Dynamic type/handler lookup |
| Builder | Complex object construction |
| Mixin | Shared functionality across classes |

---

## Frozen Boundary: Build → Render

Data flowing from build phases to the render phase MUST be immutable. The render
phase runs in parallel (free-threading, Python 3.14t) — mutable shared state
causes data races.

### Rule

| Container | Allowed in build output? | Allowed in render input? |
|-----------|-------------------------|-------------------------|
| `frozenset` | Yes | Yes (read-only) |
| `tuple` | Yes | Yes (read-only) |
| `frozen dataclass` | Yes | Yes (read-only) |
| `dict[K, frozenset]` | Yes | Yes (read values only) |
| `list` | Build-internal only | **NO** — freeze before handoff |
| `set` | Build-internal only | **NO** — freeze before handoff |
| `dict` (mutable values) | Build-internal only | **NO** |

### Examples

```python
# ✅ CORRECT — Frozen artifacts from build to render
page_tags_map: dict[int, frozenset[str]]    # immutable tag sets
excluded_ids: frozenset[int]                 # immutable exclusion set
tag_index: dict[str, frozenset[int]]        # immutable inverted index
identity_map: dict[int, PageIdentity]       # frozen dataclass values

# ❌ WRONG — Mutable containers crossing the boundary
related_map: dict[int, list[Page]]          # list is mutable
active_tags: set[str]                        # set is mutable
```

### Where the Boundary Is

Build phases (sequential or cooperative parallel):
- Discovery → Content → Parsing (phases 1-12.5)
- Output: frozen data structures on Site and pages

Render phase (fully parallel, no locks):
- Rendering → Finalization (phases 13+)
- Reads frozen data, writes individual page HTML files

### PageIdentity as Boundary Artifact

`page.identity` (PageIdentity frozen dataclass) is the primary boundary artifact.
It is computed at the end of content phases via `page.finalize_identity()` and is
immutable for the entire render phase.

### @cached_property at the Boundary

`@cached_property` is a common mechanism for lazy-computing data that crosses the
build/render boundary. Since it writes to `instance.__dict__` without a lock,
properties read during parallel render MUST either:

1. Return immutable types (tuple, frozenset, frozen dataclass) so the race is benign
2. Be pre-warmed during the sequential snapshot phase so they're already cached

Properties that return mutable containers and are accessed during render are
**data races** under 3.14t. Known properties that require immutable returns:

| Property | File | Safe Return Type |
|---|---|---|
| `visibility` | `page/metadata.py` | `VisibilitySettings` (frozen dataclass) |
| `regular_pages` | `section/queries.py` | `tuple[Page, ...]` |
| `sorted_pages` | `section/queries.py` | `tuple[Page, ...]` |
| `sorted_subsections` | `section/hierarchy.py` | `tuple[Section, ...]` |
| `subsection_index_urls` | `section/navigation.py` | `frozenset[str]` |
| `authors` | `page/__init__.py` | `tuple[Author, ...]` |

See `modules/free-threading` for the full audit classification and decision tree.

### Module-Level Cache Pattern

Module-level mutable dicts used as caches MUST protect ALL access paths with a
lock — including the "fast-path" read. Under free-threading, a concurrent `.clear()`
or write can race with an unprotected `.get()`.

```python
# ❌ WRONG — fast-path read outside lock races with .clear()
cached = _cache.get(key)
if cached is not None:
    return cached
with _lock:
    ...

# ✅ CORRECT — all reads inside lock (uncontended after warm-up)
with _lock:
    cached = _cache.get(key)
    if cached is not None:
        return cached
    result = compute()
    _cache[key] = result
    return result
```

Never use `.clear()` for cache eviction — it evicts all entries at once, causing a
stampede where every thread misses and rebuilds simultaneously. Use LRU eviction:

```python
if len(_cache) >= _max_size:
    to_remove = list(_cache.keys())[:len(_cache) // 2]
    for k in to_remove:
        del _cache[k]
```

---

## Evidence

- `bengal/core/site.py` - Site delegates to orchestrators
- `bengal/core/page/__init__.py` - Page uses mixins
- `bengal/core/page/page_identity.py` - PageIdentity frozen dataclass
- `bengal/orchestration/build_orchestrator.py` - Orchestrator with I/O
- `bengal/orchestration/related_posts.py` - Frozen build artifacts example
- `architecture/design-principles.md` - Full design documentation

---
> Source: [lbliii/bengal](https://github.com/lbliii/bengal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
