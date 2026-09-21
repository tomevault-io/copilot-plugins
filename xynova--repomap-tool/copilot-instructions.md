## always-rules-2-architecture

> **ALWAYS provide step-by-step reasoning that:**


# 🤖 **AI REASONING PROTOCOL**

## **Before Taking Any Architectural Action:**
**ALWAYS provide step-by-step reasoning that:**
1. States what architectural action you're about to take
2. Explains which rules from this document apply
3. Shows how you've considered those rules in your approach
4. Outlines your specific plan and next steps

---

# 🤖 **AI DEVELOPMENT RULES FOR REPOMAP-TOOL**

> **For AI (Cursor)**: These are the architectural rules you must follow when developing, modifying, or extending the RepoMap-Tool codebase.

## 🎯 **CORE ARCHITECTURAL PRINCIPLES**

### **🚨 FUNDAMENTAL: TREE-SITTER PARSING ARCHITECTURE**
- **THIS PROJECT USES TREE-SITTER DIRECTLY** for all code parsing and analysis
- **ALWAYS use tree-sitter parsers** for all code parsing and analysis
- **ALWAYS use TreeSitterParser.get_tags()** for code element extraction
- **PREFER get_tags() for caching benefits** - uses SQLite cache for performance
- **parse_file() is also valid** when caching is handled by DI container (standard pattern)
- **Performance note**: get_tags() provides 90%+ reduction in parse times for repeated operations
- **NEVER use Python's built-in ast.parse()** for production code analysis
- **NEVER use regex-based parsing** for code structure analysis
- **ALWAYS leverage tree-sitter's multi-language support** (Python, JS/TS, Java, Go, etc.)
- **ALWAYS use tree-sitter query files (.scm)** for parsing rules
- **NEVER reinvent tree-sitter functionality** - use the existing TreeSitterParser

### **Centralized Output Management**
- **ALL CLI output** must go through the `OutputManager` system
- **NO direct `console.print()` calls** in CLI commands (except utility functions)
- **Consistent formatting** across all commands and output types
- **Template-based rendering** for maintainable and flexible output

### **Dependency Injection (DI) Compliance**
- **ALL services** must use proper DI patterns
- **Constructor injection** is the preferred DI pattern for all services
- **NO direct instantiation** of services outside DI container
- **Service Factory pattern** for CLI commands (superior to @inject decorators)
- **Strict dependency validation** - no fallback instantiation allowed

### **Type Safety & Validation**
- **Full MyPy compliance** with comprehensive type annotations
- **Pydantic models** for all data structures and validation
- **Protocol-based interfaces** for extensibility and testability

### **🚨 CRITICAL: Absolute Path Standardization**
- **ALL file paths must be absolute** throughout the system (architectural requirement)
- **File discovery returns absolute paths** - no relative path conversion needed
- **Controllers work with absolute paths** - no path resolution redundancy
- **Only convert to relative paths** when calling tree-sitter parsing methods
- **PathResolver validates absolute paths** - raises errors for relative paths
- **Eliminates path confusion** and redundant conversions
- **Improves reliability** and performance

### **🚨 CRITICAL: Centralized Configuration Management**
- **ALL configuration values must use centralized ConfigService** - no hardcoded constants
- **ALWAYS use `get_config(key, default)` from `core.config_service`** for configuration access
- **NEVER hardcode thresholds, limits, timeouts, or other configurable values**
- **ALL configuration defaults defined in `ConfigDefaults` dataclass**
- **Configuration overrides supported via `set_config(key, value)`**
- **Global configuration service instance with singleton pattern**
- **Eliminates scattered hardcoded values** across 31+ files
- **Improves maintainability** - single place to change configuration

### **🚨 CRITICAL: Centralized Logging Management**
- **ALL logging must use centralized LoggingService** - no direct `logging.getLogger()`
- **ALWAYS use `get_logger(__name__)` from `core.logging_service`** for logger creation
- **NEVER use `logging.getLogger(__name__)` directly in modules**
- **Centralized logging configuration** with consistent formatting
- **Logger caching and singleton pattern** for performance
- **Global logging service instance** with auto-initialization
- **Eliminates duplicate logger setup** across 46+ files
- **Improves consistency** - unified logging behavior

### **🚨 CRITICAL: Configuration File Loading Requirements**
- **ALL CLI commands MUST properly load configuration files** - no exceptions
- **ALWAYS use `load_or_create_config()` from `cli.config.loader`** for configuration loading
- **NEVER use `create_analysis_config()` or `create_search_config()`** - these ignore config files
- **ALWAYS pass `config_file` parameter** to `load_or_create_config()`
- **NEVER bypass configuration file loading** with hardcoded factory methods
- **Configuration files MUST be respected** when `--config` parameter is provided
- **ALL commands must have consistent configuration loading behavior**
- **Eliminates configuration loading inconsistencies** across CLI commands
- **Improves user experience** - configuration files work as expected

### **AI Development Guidelines**
- **ALWAYS use tree-sitter parsers** - this is the core of the project, never bypass it
- **ALWAYS use TreeSitterParser.get_tags()** - never implement custom AST parsing
- **ALWAYS use tree-sitter query files (.scm)** for code element extraction
- **ALWAYS use TreeSitterParser** for all code parsing operations
- **ALWAYS use absolute paths** - all file paths must be absolute throughout the system
- **ALWAYS use centralized configuration** - use `get_config(key, default)` instead of hardcoded values
- **ALWAYS use centralized logging** - use `get_logger(__name__)` instead of `logging.getLogger(__name__)`
- **ALWAYS use proper configuration file loading** - use `load_or_create_config()` not factory methods
- **ALWAYS follow existing patterns** - don't reinvent the wheel
- **ALWAYS use dependency injection** - never instantiate services directly
- **ALWAYS validate dependencies** - raise errors for missing dependencies
- **ALWAYS use the OutputManager** - never use direct console.print in CLI commands
- **ALWAYS follow the template system** - use Jinja2 templates for output formatting
- **ALWAYS implement proper error handling** - use custom exception hierarchy
- **ALWAYS write comprehensive tests** - maintain >80% test coverage
- **ALWAYS use the existing DI container** - don't create new service factories
- **ALWAYS follow the caching patterns** - use TTL-based caching with statistics
- **ALWAYS use the existing matcher protocols** - implement MatcherProtocol interface
- **ALWAYS use MVC controller pattern** - controllers return ViewModels, not strings
- **ALWAYS use ViewModels for structured data** - don't return raw dictionaries or strings
- **ALWAYS use template-based formatters** - inherit from TemplateBasedFormatter for ViewModels

### **Matching Algorithm Architecture**
- **Strategy Pattern** for multiple matching approaches (fuzzy, semantic, hybrid)
- **Cache-First Design** with configurable TTL for performance
- **Threshold-Based Filtering** with configurable sensitivity
- **Adaptive Learning** from actual codebase content (TF-IDF, semantic analysis)

### **LLM Optimization Architecture**
- **Token Budget Management** for context window optimization
- **Context Selection Strategies** (breadth-first, depth-first, hybrid, centrality-based)
- **Critical Line Extraction** using tree-sitter for code analysis
- **Signature Enhancement** with type inference and usage patterns
- **Hierarchical Formatting** for LLM-friendly structure

### **MVC Controller Architecture**
- **Controller Pattern** for business logic orchestration (`cli/controllers/`)
- **ViewModel Pattern** for structured data representation
- **Template-Based Rendering** using Jinja2 for consistent output formatting
- **Dependency Injection** for all controller dependencies
- **Separation of Concerns** between business logic and presentation

### **Code Exploration Architecture**
- **Session-Based State Management** for persistent exploration operations
- **Tree Builder Pattern** for constructing exploration trees
- **Tree Manager Pattern** for tree operations (expand, prune, focus)
- **Entrypoint Discovery** for finding code entry points
- **Dependency-Aware Construction** using dependency analysis

### **Code Analysis Architecture** (Tree-Sitter Direct Integration)
- **Graph-Based Representation** using NetworkX for dependency graphs
- **Import Analysis** using TreeSitterParser.get_tags() for all languages
- **Centrality Analysis** for identifying critical nodes
- **Impact Analysis** for risk assessment of changes
- **Multi-Language Support** via tree-sitter (Python, JS/TS, Java, Go, C#, Rust)
- **Tree-Sitter Integration** - all parsing through TreeSitterParser
- **Tag-Based Analysis** - use tree-sitter's Tag objects for code elements

### **Caching & Performance Architecture**
- **Multi-Level Caching** (service-level, matcher-level, tree-level)
- **TTL-Based Expiration** with configurable lifetimes
- **Parallel Processing** for multi-threaded operations
- **Memory Management** with configurable limits
- **Cache Statistics** for monitoring and optimization

## 🏛️ **OUTPUT ARCHITECTURE OVERVIEW**

### **Core Components**

#### **1. OutputManager** (`src/repomap_tool/cli/output/manager.py`)
```python
# Central hub for all CLI output operations
class OutputManager:
    def display(self, data: Any, output_format: OutputFormat, ctx: Optional[click.Context] = None) -> None
    def display_error(self, error: Exception, ctx: Optional[click.Context] = None) -> None
    def display_success(self, message: str, ctx: Optional[click.Context] = None) -> None
    def display_progress(self, message: str, progress: Optional[float] = None, ctx: Optional[click.Context] = None) -> None
```

#### **2. ConsoleManager** (`src/repomap_tool/cli/output/console_manager.py`)
```python
# Centralized console management and configuration
class ConsoleManager:
    def get_console(self, ctx: Optional[click.Context] = None) -> Console
    def configure_console(self, config: ConsoleConfig) -> None
    def log_operation(self, operation: str, details: Dict[str, Any]) -> None
```

#### **3. FormatterRegistry** (`src/repomap_tool/cli/output/standard_formatters.py`)
```python
# Registry for all output formatters
class FormatterRegistry:
    def register_formatter(self, data_type: Type, formatter: FormatterProtocol) -> None
    def get_formatter(self, data_type: Type, output_format: OutputFormat) -> Optional[FormatterProtocol]
```

#### **4. Template System** (`src/repomap_tool/cli/output/templates/`)
```python
# Jinja2-based templating for flexible output generation
class TemplateEngine:
    def render_template(self, template_name: str, data: Any, config: Optional[TemplateConfig] = None) -> str
    def register_template(self, name: str, template: str) -> None

# Template-based formatters for ViewModels
class TemplateBasedFormatter(BaseFormatter):
    def _create_template_config(self, config: Optional[OutputConfig]) -> TemplateConfig
    def _format_fallback(self, data: Any, template_config: Optional[Any] = None) -> str
```

### **Output Format Hierarchy**

#### **CLI Output Formats** (for general CLI commands)
```python
class OutputFormat(str, Enum):
    JSON = "json"           # Machine-readable JSON output
    TEXT = "text"           # Rich, hierarchical, token-optimized format (default)
```

#### **Analysis Formats** (for LLM analysis)
```python
class AnalysisFormat(str, Enum):
    TEXT = "text"           # Rich, hierarchical, token-optimized format (default)
    JSON = "json"           # Raw data for programmatic consumption
```

## 🔧 **IMPLEMENTATION PATTERNS**

### **Tree-Sitter Direct Integration Pattern**
```python
# CORRECT - Always use tree-sitter directly for code analysis
class TreeSitterBasedAnalyzer:
    """Analyzer that uses tree-sitter directly for all code parsing."""
    
    def __init__(self, project_root: Optional[str] = None):
        self.project_root = project_root
        self._tree_sitter_parser = None
    
    def _get_tree_sitter_parser(self) -> TreeSitterParser:
        """Get or create tree-sitter parser instance."""
        if self._tree_sitter_parser is None:
            try:
                from repomap_tool.code_analysis.tree_sitter_parser import TreeSitterParser
                
                self._tree_sitter_parser = TreeSitterParser()
            except Exception as e:
                logger.error(f"Failed to initialize tree-sitter parser: {e}")
                raise
        return self._tree_sitter_parser
    
    def analyze_file(self, file_path: str) -> FileAnalysisResult:
        """Analyze file using tree-sitter capabilities."""
        try:
            parser = self._get_tree_sitter_parser()
            
            # Use tree-sitter's get_tags() for parsing
            tags = parser.get_tags(file_path)
            
            # Extract information from tree-sitter Tag objects
            imports = self._extract_imports_from_tags(tags, file_path)
            functions = self._extract_functions_from_tags(tags)
            classes = self._extract_classes_from_tags(tags)
            
            return FileAnalysisResult(
                file_path=file_path,
                imports=imports,
                defined_functions=functions,
                defined_classes=classes,
                function_calls=self._extract_calls_from_tags(tags, file_path),
                used_variables=[],
                line_count=len(tags) if tags else 0,
                analysis_errors=[]
            )
        except Exception as e:
            logger.error(f"Error analyzing file {file_path} with tree-sitter: {e}")
            return FileAnalysisResult(
                file_path=file_path,
                imports=[],
                defined_functions=[],
                defined_classes=[],
                function_calls=[],
                used_variables=[],
                line_count=0,
                analysis_errors=[str(e)]
            )
    
    def _extract_imports_from_tags(self, tags: List[Any], file_path: str) -> List[Import]:
        """Extract imports from tree-sitter Tag objects."""
        imports = []
        for tag in tags:
            if tag.kind in ['import', 'import_from']:
                imports.append(Import(
                    module=tag.name,
                    alias=tag.alias if hasattr(tag, 'alias') else None,
                    file_path=file_path,
                    line_number=tag.line,
                    import_type=ImportType.STANDARD
                ))
        return imports
    
    def _extract_functions_from_tags(self, tags: List[Any]) -> List[str]:
        """Extract function names from tree-sitter Tag objects."""
        functions = []
        for tag in tags:
            if tag.kind in ['def', 'function']:
                functions.append(tag.name)
        return functions
    
    def _extract_classes_from_tags(self, tags: List[Any]) -> List[str]:
        """Extract class names from tree-sitter Tag objects."""
        classes = []
        for tag in tags:
            if tag.kind == 'class':
                classes.append(tag.name)
        return classes
```

### **Matching Algorithm Pattern**
```python
# Strategy Pattern for multiple matching approaches
class MatcherProtocol(Protocol):
    def match_identifiers(self, query: str, all_identifiers: Set[str]) -> List[Tuple[str, int]]: ...
    def clear_cache(self) -> None: ...
    def get_cache_stats(self) -> Dict[str, Any]: ...

class FuzzyMatcher:
    def __init__(self, threshold: int = 70, strategies: List[str] = None):
        self.threshold = threshold
        self.strategies = strategies or ["prefix", "suffix", "substring", "levenshtein"]
        self.cache = CacheManager(max_size=1000, ttl=3600)
    
    def match_identifiers(self, query: str, all_identifiers: Set[str]) -> List[Tuple[str, int]]:
        # Check cache first
        cache_key = f"fuzzy_{query}_{hash(frozenset(all_identifiers))}"
        if cached_result := self.cache.get(cache_key):
            return cached_result
        
        # Apply multiple strategies
        matches = []
        for strategy in self.strategies:
            strategy_matches = self._apply_strategy(strategy, query, all_identifiers)
            matches.extend(strategy_matches)
        
        # Cache and return results
        self.cache.set(cache_key, matches)
        return matches
```

### **LLM Optimization Pattern**
```python
# Token budget management and context selection
class TokenOptimizer:
    def __init__(self, max_tokens: int = 8000):
        self.max_tokens = max_tokens
        self.token_estimator = TokenEstimator()
    
    def optimize_context(self, data: Any, strategy: SelectionStrategy) -> ContextSelection:
        # Estimate token usage
        estimated_tokens = self.token_estimator.estimate_tokens(data)
        
        if estimated_tokens <= self.max_tokens:
            return ContextSelection(data=data, tokens_used=estimated_tokens)
        
        # Apply selection strategy
        if strategy == SelectionStrategy.CENTRALITY_BASED:
            return self._select_by_centrality(data)
        elif strategy == SelectionStrategy.BREADTH_FIRST:
            return self._select_breadth_first(data)
        # ... other strategies
    
    def _select_by_centrality(self, data: Any) -> ContextSelection:
        # Prioritize by dependency centrality
        centrality_scores = self._calculate_centrality(data)
        selected = self._select_top_by_centrality(data, centrality_scores)
        return ContextSelection(data=selected, tokens_used=self.token_estimator.estimate_tokens(selected))
```

### **Code Exploration Pattern**
```python
# Session-based exploration management
class TreeManager:
    def __init__(self, session_manager: SessionManager, tree_builder: TreeBuilder):
        if session_manager is None:
            raise ValueError("SessionManager must be injected - no fallback allowed")
        if tree_builder is None:
            raise ValueError("TreeBuilder must be injected - no fallback allowed")
        
        self.session_manager = session_manager
        self.tree_builder = tree_builder
    
    def expand_tree(self, session_id: str, tree_id: str, node_id: str) -> bool:
        # Get session and tree
        session = self.session_manager.get_session(session_id)
        tree = session.get_tree(tree_id)
        
        # Find and expand node
        node = self._find_node(tree, node_id)
        if node and not node.expanded:
            self._expand_node(node, tree)
            self.session_manager.save_session(session)
            return True
        return False
    
    def _expand_node(self, node: TreeNode, tree: ExplorationTree) -> None:
        # Use dependency analysis to find related nodes
        dependencies = self._get_node_dependencies(node)
        for dep in dependencies:
            child = TreeNode(
                identifier=dep.name,
                location=dep.location,
                node_type=dep.type,
                depth=node.depth + 1
            )
            child.parent = node
            node.children.append(child)
        node.expanded = True
```

### **Dependency Analysis Pattern**
```python
# Graph-based dependency analysis
class DependencyGraph:
    def __init__(self):
        self.graph = nx.DiGraph()
        self.import_analyzer = ImportAnalyzer()
    
    def build_graph(self, project_imports: ProjectImports) -> None:
        # Add nodes for each file
        for file_path, imports in project_imports.files.items():
            self.graph.add_node(file_path, imports=imports)
        
        # Add edges for dependencies
        for file_path, imports in project_imports.files.items():
            for import_stmt in imports.imports:
                if import_stmt.resolved_path:
                    self.graph.add_edge(file_path, import_stmt.resolved_path)
    
    def calculate_centrality(self) -> Dict[str, float]:
        # Calculate betweenness centrality
        centrality = nx.betweenness_centrality(self.graph)
        return centrality
    
    def find_impact_scope(self, changed_file: str) -> List[str]:
        # Find all files that depend on the changed file
        dependents = list(nx.descendants(self.graph, changed_file))
        return dependents
```

### **Caching Pattern**
```python
# Multi-level caching with TTL
class CacheManager:
    def __init__(self, max_size: int = 1000, ttl: int = 3600):
        self.max_size = max_size
        self.ttl = ttl
        self.cache: Dict[str, CacheEntry] = {}
        self.access_times: Dict[str, float] = {}
    
    def get(self, key: str) -> Optional[Any]:
        if key not in self.cache:
            return None
        
        entry = self.cache[key]
        if self._is_expired(entry):
            self._remove(key)
            return None
        
        # Update access time for LRU
        self.access_times[key] = time.time()
        return entry.value
    
    def set(self, key: str, value: Any) -> None:
        # Check size limit
        if len(self.cache) >= self.max_size:
            self._evict_lru()
        
        # Store with TTL
        self.cache[key] = CacheEntry(value=value, created_at=time.time())
        self.access_times[key] = time.time()
    
    def _is_expired(self, entry: CacheEntry) -> bool:
        return time.time() - entry.created_at > self.ttl
```

### **MVC Controller Pattern**
```python
# Controller for business logic orchestration
class BaseController(ABC):
    """Abstract base class for all controllers."""
    
    def __init__(self, config: Optional[ControllerConfig] = None):
        self.config = config
        self.logger = logging.getLogger(self.__class__.__name__)
    
    @abstractmethod
    def execute(self, *args: Any, **kwargs: Any) -> Any:
        """Execute the controller's main operation."""
        pass

class CentralityController(BaseController):
    """Controller for centrality analysis operations."""
    
    def __init__(
        self,
        llm_analyzer: LLMFileAnalyzer,
        token_optimizer: TokenOptimizer,
        context_selector: ContextSelector,
        config: Optional[ControllerConfig] = None,
    ) -> None:
        super().__init__(config)
        self.llm_analyzer = llm_analyzer
        self.token_optimizer = token_optimizer
        self.context_selector = context_selector
    
    def execute(self, file_paths: List[str]) -> CentralityViewModel:
        """Execute centrality analysis and return ViewModel."""
        if self.config is None:
            raise ValueError("ControllerConfig must be set before executing")
        
        # Perform business logic
        raw_data = self.llm_analyzer.analyze_file_centrality(file_paths)
        
        # Build and return ViewModel
        return self._build_view_model(raw_data, file_paths)
    
    def _build_view_model(self, raw_data: Dict[str, Any], file_paths: List[str]) -> CentralityViewModel:
        """Build ViewModel from raw analysis data."""
        return CentralityViewModel(
            files=self._build_file_analyses(file_paths),
            rankings=[],
            total_files=len(file_paths),
            analysis_summary=self._build_analysis_summary(raw_data),
            token_count=len(str(raw_data)),
            max_tokens=self.config.max_tokens if self.config else 4000,
            compression_level=self.config.compression_level if self.config else "medium",
        )
```

### **Constructor DI Pattern**
```python
# PREFERRED - Constructor injection with strict validation
class ExampleService:
    def __init__(
        self,
        config: RepoMapConfig,
        console: Console,
        matcher: FuzzyMatcher,
        formatter: ProjectInfoFormatter,
    ) -> None:
        # All dependencies must be injected - no fallback allowed
        if console is None:
            raise ValueError("Console must be injected - no fallback allowed")
        if matcher is None:
            raise ValueError("FuzzyMatcher must be injected - no fallback allowed")
        if formatter is None:
            raise ValueError("ProjectInfoFormatter must be injected - no fallback allowed")
        
        self.config = config
        self.console = console
        self.matcher = matcher
        self.formatter = formatter
    
    def process(self) -> str:
        # Use injected dependencies
        result = self.matcher.match("query", ["data"])
        return self.formatter.format(result, OutputFormat.TEXT)
```

### **Centralized Configuration Pattern**
```python
# CORRECT - Use centralized configuration service
from repomap_tool.core.config_service import get_config

class ExampleService:
    def __init__(self):
        # ✅ AI: ALWAYS use get_config() for configuration values
        self.threshold = get_config('FUZZY_MATCH_THRESHOLD', 70)
        self.max_workers = get_config('MAX_WORKERS', 4)
        self.timeout = get_config('ANALYSIS_TIMEOUT', 30)
        self.cache_ttl = get_config('CACHE_TTL', 3600)
    
    def process_data(self, data: Any) -> Any:
        # ✅ AI: ALWAYS use config values, never hardcode
        if len(data) > get_config('MAX_DATA_SIZE', 1000):
            raise ValueError("Data too large")
        
        # Use configured timeout
        timeout = get_config('PROCESSING_TIMEOUT', 60)
        return self._process_with_timeout(data, timeout)

# WRONG - Hardcoded configuration values
class BadService:
    def __init__(self):
        # ❌ AI: NEVER hardcode configuration values
        self.threshold = 70  # WRONG!
        self.max_workers = 4  # WRONG!
        self.timeout = 30  # WRONG!
```

### **Centralized Logging Pattern**
```python
# CORRECT - Use centralized logging service
from repomap_tool.core.logging_service import get_logger

class ExampleService:
    def __init__(self):
        # ✅ AI: ALWAYS use get_logger() for logger creation
        self.logger = get_logger(__name__)
    
    def process_data(self, data: Any) -> Any:
        # ✅ AI: ALWAYS use centralized logger
        self.logger.info(f"Processing {len(data)} items")
        
        try:
            result = self._process(data)
            self.logger.debug(f"Processing completed successfully")
            return result
        except Exception as e:
            self.logger.error(f"Processing failed: {e}")
            raise

# WRONG - Direct logging.getLogger() usage
import logging

class BadService:
    def __init__(self):
        # ❌ AI: NEVER use logging.getLogger() directly
        self.logger = logging.getLogger(__name__)  # WRONG!
```

### **Centralized Services Integration Pattern**
```python
# CORRECT - Combine centralized services
from repomap_tool.core.config_service import get_config
from repomap_tool.core.logging_service import get_logger

class ExampleService:
    def __init__(self):
        # ✅ AI: ALWAYS use both centralized services
        self.logger = get_logger(__name__)
        self.threshold = get_config('SERVICE_THRESHOLD', 0.8)
        self.max_retries = get_config('MAX_RETRIES', 3)
    
    def process_with_retry(self, data: Any) -> Any:
        retries = 0
        max_retries = get_config('MAX_RETRIES', 3)
        
        while retries < max_retries:
            try:
                self.logger.info(f"Processing attempt {retries + 1}")
                result = self._process(data)
                self.logger.info("Processing successful")
                return result
            except Exception as e:
                retries += 1
                self.logger.warning(f"Attempt {retries} failed: {e}")
                
                if retries >= max_retries:
                    self.logger.error("All retry attempts exhausted")
                    raise
        
        return None
```

### **CLI Command Pattern**
```python
@click.command()
def centrality_command(ctx: click.Context, file_paths: List[str], format: str) -> None:
    """Centrality analysis command using MVC controller pattern."""
    # Get services via DI container
    from repomap_tool.core.container import create_container
    from repomap_tool.cli.controllers import ControllerConfig
    
    # Create DI container and get controller
    container = create_container(config_obj)
    centrality_controller = container.centrality_controller()
    
    # Prepare controller configuration
    controller_config = ControllerConfig(
        max_tokens=max_tokens, 
        output_format=OutputFormat(format)
    )
    
    # Get output manager
    output_manager = get_output_manager()
    
    try:
        # Execute business logic via controller
        result_view_model = centrality_controller.execute(
            config=controller_config, 
            file_paths=file_paths
        )
        
        # Display ViewModel via OutputManager
        output_manager.display(result_view_model, OutputFormat(format), ctx)
        
    except Exception as e:
        # Error handling via OutputManager
        output_manager.display_error(e, ctx)
```

### **Formatter Implementation Pattern**
```python
class ExampleFormatter(BaseFormatter):
    """Formatter for ExampleData objects."""
    
    def supports_format(self, output_format: OutputFormat) -> bool:
        return output_format in [OutputFormat.TEXT, OutputFormat.JSON]
    
    def format(self, data: ExampleData, output_format: OutputFormat, ctx: Optional[click.Context] = None) -> str:
        if output_format == OutputFormat.JSON:
            return self._format_json(data)
        elif output_format == OutputFormat.TEXT:
            return self._format_text(data)
        else:
            raise ValueError(f"Unsupported format: {output_format}")
    
    def _format_text(self, data: ExampleData) -> str:
        # Use template system for consistent formatting
        return self._template_engine.render_template("example.jinja2", {"data": data})
```

### **Jinja2 Template Pattern**
```jinja2
{# Standard Template Structure #}
{# Template Name: example.jinja2 #}
{% set emoji = config.options.use_emojis if config and config.options else true %}
{% set hierarchical = config.options.use_hierarchical_structure if config and config.options else true %}

{# Header with conditional emoji #}
{% if emoji %}🔍 Example Analysis{% else %}Example Analysis{% endif %}
{{ "=" * 60 }}

{# Summary section #}
{% if emoji %}📊{% endif %} SUMMARY:
{% if hierarchical %}├──{% else %}•{% endif %} Total Items: {{ data.total_items }}
{% if hierarchical %}└──{% else %}•{% endif %} Processing Time: {{ data.processing_time_ms | format_duration }}

{# Results section with conditional display #}
{% if data.results %}
{% if emoji %}🎯{% endif %} RESULTS:
{% for result in data.results %}
{% if hierarchical %}├──{% else %}•{% endif %} {{ result.name }}
{% if hierarchical %}│   {% else %}  {% endif %}Score: {{ result.score | format_percentage }}
{% if not loop.last %}
{% if hierarchical %}│{% endif %}
{% endif %}
{% endfor %}
{% else %}
{% if emoji %}❌{% endif %} No results found
{% endif %}
```

#### **Template Pattern Rules:**
1. **Configuration-Driven**: Use `config.options` for conditional rendering
2. **Emoji Support**: Always provide emoji and non-emoji variants
3. **Hierarchical Structure**: Support both hierarchical (├──) and flat (•) layouts
4. **Consistent Formatting**: Use standard separators and indentation
5. **Conditional Sections**: Only show sections when data is available
6. **Custom Filters**: Use filters like `format_percentage`, `format_duration`
7. **Loop Handling**: Proper handling of loop boundaries and separators

## 🚫 **ANTI-PATTERNS TO AVOID (AI MUST NOT DO THESE)**

### **❌ Custom AST Parsing (CRITICAL - NEVER DO THIS)**
```python
# WRONG - AI must never implement custom AST parsing
import ast

class BadCustomAnalyzer:
    def analyze_python_file(self, file_path: str):
        # ❌ AI: NEVER use Python's built-in ast.parse()
        with open(file_path, 'r') as f:
            tree = ast.parse(f.read())  # WRONG!
        
        # ❌ AI: NEVER implement custom AST visitors
        class CustomVisitor(ast.NodeVisitor):
            def visit_Import(self, node):
                # Custom parsing logic - WRONG!
                pass

# WRONG - AI must never use regex for code parsing
import re

class BadRegexAnalyzer:
    def extract_imports(self, content: str):
        # ❌ AI: NEVER use regex for code structure analysis
        import_pattern = r'^import\s+(\w+)'
        return re.findall(import_pattern, content, re.MULTILINE)  # WRONG!

# WRONG - AI must never bypass tree-sitter
class BadBypassAnalyzer:
    def analyze_file(self, file_path: str):
        # ❌ AI: NEVER bypass tree-sitter
        # This defeats the entire purpose of the project!
        return self._custom_parsing_logic(file_path)  # WRONG!
```

### **❌ Direct Console Usage**
```python
# WRONG - AI must never do this
@click.command()
def bad_command():
    console = Console()  # ❌ AI: Don't instantiate Console directly
    console.print("Bad output")  # ❌ AI: Don't use console.print in CLI commands
```

### **❌ Direct Service Instantiation**
```python
# WRONG - AI must never do this
def bad_function():
    matcher = FuzzyMatcher(threshold=0.7)  # ❌ AI: Don't bypass DI
    formatter = ProjectInfoFormatter()     # ❌ AI: Don't bypass DI
```

### **❌ Hardcoded Configuration Values**
```python
# WRONG - AI must never hardcode configuration values
class BadService:
    def __init__(self):
        # ❌ AI: NEVER hardcode configuration values
        self.threshold = 70  # WRONG!
        self.max_workers = 4  # WRONG!
        self.timeout = 30  # WRONG!
        self.cache_ttl = 3600  # WRONG!
    
    def process_data(self, data: Any) -> Any:
        # ❌ AI: NEVER hardcode limits or thresholds
        if len(data) > 1000:  # WRONG!
            raise ValueError("Data too large")
        
        # ❌ AI: NEVER hardcode timeouts
        return self._process_with_timeout(data, 60)  # WRONG!
```

### **❌ Direct Logging Usage**
```python
# WRONG - AI must never use logging.getLogger() directly
import logging

class BadService:
    def __init__(self):
        # ❌ AI: NEVER use logging.getLogger() directly
        self.logger = logging.getLogger(__name__)  # WRONG!
    
    def process_data(self, data: Any) -> Any:
        # ❌ AI: NEVER use direct logging setup
        logging.basicConfig(level=logging.INFO)  # WRONG!
        self.logger.info("Processing data")  # WRONG!
```

### **❌ Scattered Configuration Management**
```python
# WRONG - AI must never scatter configuration across files
class BadService1:
    def __init__(self):
        self.threshold = 70  # ❌ WRONG: Hardcoded in file 1

class BadService2:
    def __init__(self):
        self.threshold = 80  # ❌ WRONG: Different value in file 2

class BadService3:
    def __init__(self):
        self.threshold = 75  # ❌ WRONG: Another value in file 3
        # This creates inconsistency and maintenance nightmare
```

### **❌ Configuration Loading Anti-Patterns**
```python
# WRONG - AI must never bypass configuration file loading
@click.command()
def bad_command(project_path: str, config: Optional[str]) -> None:
    # ❌ AI: NEVER use factory methods that ignore config files
    config_factory = get_config_factory()
    config_obj = config_factory.create_analysis_config(
        project_root=project_path,
        verbose=verbose,
    )  # WRONG! Ignores --config parameter completely

# WRONG - AI must never use create_search_config for CLI commands
@click.command()
def bad_search_command(project_path: str, config: Optional[str]) -> None:
    # ❌ AI: NEVER use create_search_config - it ignores config files
    config_obj = create_search_config(
        project_path, match_type, verbose, log_level, cache_size
    )  # WRONG! Ignores --config parameter completely

# WRONG - AI must never skip config_file parameter
@click.command()
def bad_command(project_path: str, config: Optional[str]) -> None:
    # ❌ AI: NEVER call load_or_create_config without config_file parameter
    config_obj, was_created = load_or_create_config(
        project_path=project_path,
        # config_file=config,  # WRONG! Missing this parameter
        create_if_missing=False,
        verbose=verbose,
    )
```

### **❌ AI Anti-Patterns**
```python
# WRONG - AI must never do these things
class BadAICode:
    def __init__(self):
        # ❌ AI: Don't create services directly
        self.matcher = FuzzyMatcher()
        self.cache = CacheManager()
        
        # ❌ AI: Don't use fallback instantiation
        self.console = Console() if console is None else console
        
        # ❌ AI: Don't bypass the OutputManager
        console.print("Direct output")  # Use OutputManager instead
        
        # ❌ AI: Don't create new service factories
        self.service = SomeService()  # Use existing DI container
        
        # ❌ AI: Don't skip dependency validation
        self.dependency = dependency  # Should validate and raise error if None
```

### **❌ Constructor DI Violations**
```python
# WRONG - Fallback instantiation in constructor
class BadService:
    def __init__(self, console: Optional[Console] = None):
        self.console = console or Console()  # Should raise error instead

# WRONG - Missing dependency validation
class BadService:
    def __init__(self, console: Optional[Console] = None):
        self.console = console  # No validation - could be None
```

### **❌ Fallback Instantiation**
```python
# WRONG - Fallback instantiation
class BadService:
    def __init__(self, console: Optional[Console] = None):
        self.console = console or Console()  # Should raise error instead
```

### **❌ Mixed Output Patterns**
```python
# WRONG - Mixing OutputManager and direct console
def bad_function():
    output_manager.display("Good output")
    console.print("Bad output")  # Inconsistent
```

### **❌ Template Anti-Patterns**
```jinja2
{# WRONG - Hardcoded values without configuration #}
🔍 Search Results
{{ "=" * 60 }}
• Query: {{ data.query }}
• Results: {{ data.results|length }}

{# WRONG - No emoji alternatives #}
🔍 Search Results  {# Always shows emoji #}

{# WRONG - Inconsistent formatting #}
• Item 1
  - Subitem 1
• Item 2
  - Subitem 2  {# Inconsistent indentation #}

{# WRONG - No conditional sections #}
📊 Results: {{ data.results|length }}  {# Shows even when empty #}
```

### **❌ Matching Algorithm Anti-Patterns**
```python
# WRONG - No caching
class BadMatcher:
    def match_identifiers(self, query: str, identifiers: Set[str]) -> List[Tuple[str, int]]:
        # Always recalculates - no caching
        return self._calculate_matches(query, identifiers)

# WRONG - Single strategy only
class BadMatcher:
    def match_identifiers(self, query: str, identifiers: Set[str]) -> List[Tuple[str, int]]:
        # Only uses exact matching - inflexible
        return [(id, 100) for id in identifiers if query == id]

# WRONG - No threshold configuration
class BadMatcher:
    def match_identifiers(self, query: str, identifiers: Set[str]) -> List[Tuple[str, int]]:
        # Hardcoded threshold - not configurable
        return [(id, score) for id, score in self._calculate_scores(query, identifiers) if score >= 80]
```

### **❌ LLM Optimization Anti-Patterns**
```python
# WRONG - No token budget management
class BadLLMOptimizer:
    def optimize_output(self, data: Any) -> str:
        # Always returns full data - no token limits
        return str(data)

# WRONG - No context selection
class BadLLMOptimizer:
    def optimize_output(self, data: Any) -> str:
        # No strategy for selecting relevant context
        return self._format_all_data(data)

# WRONG - No hierarchical structure
class BadLLMOptimizer:
    def optimize_output(self, data: Any) -> str:
        # Flat output - not LLM-friendly
        return "\n".join([str(item) for item in data])
```

### **❌ Code Exploration Anti-Patterns**
```python
# WRONG - No session management
class BadTreeManager:
    def expand_tree(self, tree_id: str, node_id: str) -> bool:
        # No session tracking - state lost between operations
        tree = self._get_tree(tree_id)
        return self._expand_node(tree, node_id)

# WRONG - No dependency awareness
class BadTreeManager:
    def expand_node(self, node: TreeNode) -> None:
        # Random expansion - not based on actual dependencies
        random_children = self._get_random_children(node)
        node.children.extend(random_children)

# WRONG - No caching
class BadTreeManager:
    def build_tree(self, entrypoint: Entrypoint) -> ExplorationTree:
        # Always rebuilds - no caching
        return self._build_from_scratch(entrypoint)
```

### **❌ Dependency Analysis Anti-Patterns**
```python
# WRONG - No graph representation
class BadDependencyAnalyzer:
    def analyze_dependencies(self, files: List[str]) -> Dict[str, List[str]]:
        # Simple dict - no graph algorithms available
        dependencies = {}
        for file in files:
            dependencies[file] = self._find_imports(file)
        return dependencies

# WRONG - No centrality analysis
class BadDependencyAnalyzer:
    def find_critical_files(self, files: List[str]) -> List[str]:
        # Random selection - not based on actual centrality
        return random.sample(files, min(5, len(files)))

# WRONG - No impact analysis
class BadDependencyAnalyzer:
    def analyze_change_impact(self, changed_file: str) -> List[str]:
        # No impact analysis - just returns the file itself
        return [changed_file]
```

### **❌ Caching Anti-Patterns**
```python
# WRONG - No TTL management
class BadCache:
    def __init__(self):
        self.cache = {}
    
    def get(self, key: str) -> Optional[Any]:
        # No expiration - cache grows indefinitely
        return self.cache.get(key)

# WRONG - No size limits
class BadCache:
    def set(self, key: str, value: Any) -> None:
        # No size management - memory leak potential
        self.cache[key] = value

# WRONG - No cache statistics
class BadCache:
    def get_stats(self) -> Dict[str, Any]:
        # No monitoring - can't optimize
        return {"size": len(self.cache)}
```

## ✅ **CORRECT PATTERNS (AI MUST DO THESE)**

### **✅ Tree-Sitter Direct Integration (CRITICAL - ALWAYS DO THIS)**
```python
# CORRECT - AI must always use tree-sitter directly
from repomap_tool.code_analysis.tree_sitter_parser import TreeSitterParser

class GoodTreeSitterBasedAnalyzer:
    """Analyzer that properly uses tree-sitter directly."""
    
    def __init__(self, project_root: Optional[str] = None):
        self.project_root = project_root
        self._tree_sitter_parser = None
    
    def _get_tree_sitter_parser(self) -> TreeSitterParser:
        """Get or create tree-sitter parser instance."""
        if self._tree_sitter_parser is None:
            try:
                # ✅ AI: ALWAYS use tree-sitter directly
                self._tree_sitter_parser = TreeSitterParser()
            except Exception as e:
                logger.error(f"Failed to initialize tree-sitter parser: {e}")
                raise
        return self._tree_sitter_parser
    
    def analyze_file(self, file_path: str) -> FileAnalysisResult:
        """Analyze file using tree-sitter capabilities."""
        try:
            parser = self._get_tree_sitter_parser()
            
            # ✅ AI: ALWAYS use tree-sitter's get_tags() for parsing
            tags = parser.get_tags(file_path)
            
            # ✅ AI: ALWAYS extract from tree-sitter Tag objects
            imports = self._extract_imports_from_tags(tags, file_path)
            functions = self._extract_functions_from_tags(tags)
            classes = self._extract_classes_from_tags(tags)
            
            return FileAnalysisResult(
                file_path=file_path,
                imports=imports,
                defined_functions=functions,
                defined_classes=classes,
                function_calls=self._extract_calls_from_tags(tags, file_path),
                used_variables=[],
                line_count=len(tags) if tags else 0,
                analysis_errors=[]
            )
        except Exception as e:
            logger.error(f"Error analyzing file {file_path} with tree-sitter: {e}")
            return self._create_error_result(file_path, str(e))
    
    def _extract_imports_from_tags(self, tags: List[Any], file_path: str) -> List[Import]:
        """Extract imports from tree-sitter Tag objects."""
        imports = []
        for tag in tags:
            if tag.kind in ['import', 'import_from']:
                # ✅ AI: ALWAYS use tree-sitter Tag attributes
                imports.append(Import(
                    module=tag.name,
                    alias=getattr(tag, 'alias', None),
                    file_path=file_path,
                    line_number=tag.line,
                    import_type=ImportType.STANDARD
                ))
        return imports
    
    def _extract_functions_from_tags(self, tags: List[Any]) -> List[str]:
        """Extract function names from tree-sitter Tag objects."""
        functions = []
        for tag in tags:
            if tag.kind in ['def', 'function']:
                # ✅ AI: ALWAYS use tree-sitter Tag.name
                functions.append(tag.name)
        return functions
    
    def _extract_classes_from_tags(self, tags: List[Any]) -> List[str]:
        """Extract class names from tree-sitter Tag objects."""
        classes = []
        for tag in tags:
            if tag.kind == 'class':
                # ✅ AI: ALWAYS use tree-sitter Tag.name
                classes.append(tag.name)
        return classes
```

### **✅ AI-Required Constructor DI Usage**
```python
# CORRECT - AI must always do this
class GoodService:
    def __init__(
        self,
        config: RepoMapConfig,
        console: Console,
        matcher: FuzzyMatcher,
    ) -> None:
        # ✅ AI: Always validate dependencies strictly
        if console is None:
            raise ValueError("Console must be injected - no fallback allowed")
        if matcher is None:
            raise ValueError("FuzzyMatcher must be injected - no fallback allowed")
        
        self.config = config
        self.console = console
        self.matcher = matcher

# CORRECT - AI must use service factory pattern
def good_function():
    # ✅ AI: Always use existing service factory
    service_factory = get_service_factory()
    service = service_factory.create_service(config)  # Uses constructor DI
    
    # ✅ AI: Always use OutputManager for output
    output_manager = get_output_manager()
    
    result = service.process()
    output_manager.display(result, OutputFormat.TEXT)
```

### **✅ Configuration Loading Patterns**
```python
# CORRECT - AI must always use proper configuration file loading
@click.command()
def good_command(project_path: str, config: Optional[str]) -> None:
    # ✅ AI: ALWAYS use load_or_create_config for CLI commands
    from repomap_tool.cli.config.loader import load_or_create_config
    
    config_obj, was_created = load_or_create_config(
        project_path=project_path,
        config_file=config,  # ✅ AI: ALWAYS pass config_file parameter
        create_if_missing=False,
        verbose=verbose,
    )
    
    # ✅ AI: Configuration file is now properly loaded and applied

# CORRECT - AI must always respect configuration files
@click.command()
def good_search_command(project_path: str, config: Optional[str]) -> None:
    # ✅ AI: ALWAYS use load_or_create_config, then override specific settings
    from repomap_tool.cli.config.loader import load_or_create_config
    
    config_obj, was_created = load_or_create_config(
        project_path=project_path,
        config_file=config,  # ✅ AI: ALWAYS pass config_file parameter
        create_if_missing=False,
        verbose=verbose,
    )
    
    # ✅ AI: Override specific settings after loading config file
    config_obj.fuzzy_match.enabled = match_type in ["fuzzy", "hybrid"]
    config_obj.semantic_match.enabled = match_type in ["semantic", "hybrid"]
```

### **✅ AI-Required Patterns**
```python
# CORRECT - AI must always follow these patterns
class GoodAICode:
    def __init__(self, dependency: SomeService):
        # ✅ AI: Always validate dependencies
        if dependency is None:
            raise ValueError("Dependency must be injected - no fallback allowed")
        self.dependency = dependency
    
    def process_data(self, data: Any) -> str:
        # ✅ AI: Always use OutputManager for output
        output_manager = get_output_manager()
        
        try:
            result = self.dependency.process(data)
            # ✅ AI: Always use proper output formatting
            output_manager.display_success("Processing completed")
            return result
        except Exception as e:
            # ✅ AI: Always use proper error handling
            output_manager.display_error(e)
            raise
    
    def create_new_service(self, config: RepoMapConfig) -> SomeService:
        # ✅ AI: Always use existing DI container
        container = get_container()
        return container.some_service(config)
```

### **✅ Strict Dependency Validation**
```python
# CORRECT - Strict validation
class GoodService:
    def __init__(self, console: Optional[Console] = None):
        if console is None:
            raise ValueError("Console must be injected - no fallback allowed")
        self.console = console
```

### **✅ Template-Based Formatting**
```python
# CORRECT - Template-based output
class GoodFormatter(BaseFormatter):
    def format(self, data: Any, output_format: OutputFormat) -> str:
        template_name = f"{type(data).__name__.lower()}.jinja2"
        return self._template_engine.render_template(template_name, {"data": data})
```

### **✅ Template Best Practices**
```jinja2
{# CORRECT - Configuration-driven template #}
{# Template: search_response.jinja2 #}
{% set emoji = config.options.use_emojis if config and config.options else true %}
{% set hierarchical = config.options.use_hierarchical_structure if config and config.options else true %}

{# Header with conditional emoji #}
{% if emoji %}🔍 Search Results{% else %}Search Results{% endif %}
{{ "=" * 60 }}

{# Summary with consistent formatting #}
{% if emoji %}📊{% endif %} QUERY: "{{ data.query }}"
{% if hierarchical %}├──{% else %}•{% endif %} Match Type: {{ data.match_type }}
{% if hierarchical %}├──{% else %}•{% endif %} Threshold: {{ data.threshold | format_percentage }}
{% if hierarchical %}└──{% else %}•{% endif %} Total Results: {{ data.total_results }}

{# Conditional results section #}
{% if data.results %}
{% if emoji %}🎯{% endif %} RESULTS:
{% for result in data.results %}
{% if hierarchical %}├──{% else %}•{% endif %} {{ result.identifier }}
{% if hierarchical %}│   {% else %}  {% endif %}Score: {{ result.score | format_percentage }}
{% if not loop.last %}
{% if hierarchical %}│{% endif %}
{% endif %}
{% endfor %}
{% else %}
{% if emoji %}❌{% endif %} No results found
{% endif %}
```

### **✅ Matching Algorithm Best Practices**
```python
# CORRECT - Multi-strategy with caching
class GoodMatcher:
    def __init__(self, threshold: int = 70, strategies: List[str] = None, cache_manager: CacheManager = None):
        if cache_manager is None:
            raise ValueError("CacheManager must be injected - no fallback allowed")
        
        self.threshold = threshold
        self.strategies = strategies or ["prefix", "suffix", "substring", "levenshtein"]
        self.cache = cache_manager
    
    def match_identifiers(self, query: str, all_identifiers: Set[str]) -> List[Tuple[str, int]]:
        # Check cache first
        cache_key = f"match_{hash(query)}_{hash(frozenset(all_identifiers))}"
        if cached_result := self.cache.get(cache_key):
            return cached_result
        
        # Apply multiple strategies
        matches = []
        for strategy in self.strategies:
            strategy_matches = self._apply_strategy(strategy, query, all_identifiers)
            matches.extend(strategy_matches)
        
        # Filter by threshold and sort
        filtered_matches = [(id, score) for id, score in matches if score >= self.threshold]
        filtered_matches.sort(key=lambda x: x[1], reverse=True)
        
        # Cache and return
        self.cache.set(cache_key, filtered_matches)
        return filtered_matches
```

### **✅ LLM Optimization Best Practices**
```python
# CORRECT - Token budget management with context selection
class GoodLLMOptimizer:
    def __init__(self, token_optimizer: TokenOptimizer, context_selector: ContextSelector):
        if token_optimizer is None:
            raise ValueError("TokenOptimizer must be injected - no fallback allowed")
        if context_selector is None:
            raise ValueError("ContextSelector must be injected - no fallback allowed")
        
        self.token_optimizer = token_optimizer
        self.context_selector = context_selector
    
    def optimize_for_llm(self, data: Any, max_tokens: int = 8000, strategy: SelectionStrategy = SelectionStrategy.CENTRALITY_BASED) -> str:
        # Estimate token usage
        estimated_tokens = self.token_optimizer.estimate_tokens(data)
        
        if estimated_tokens <= max_tokens:
            return self._format_hierarchical(data)
        
        # Select optimal context
        context_selection = self.context_selector.select_context(data, max_tokens, strategy)
        
        # Format with hierarchical structure
        return self._format_hierarchical(context_selection.data)
    
    def _format_hierarchical(self, data: Any) -> str:
        # Use hierarchical formatter for LLM-friendly output
        formatter = HierarchicalFormatter()
        return formatter.format(data)
```

### **✅ Code Exploration Best Practices**
```python
# CORRECT - Session-based with dependency awareness
class GoodTreeManager:
    def __init__(self, session_manager: SessionManager, tree_builder: TreeBuilder, dependency_graph: DependencyGraph):
        if session_manager is None:
            raise ValueError("SessionManager must be injected - no fallback allowed")
        if tree_builder is None:
            raise ValueError("TreeBuilder must be injected - no fallback allowed")
        if dependency_graph is None:
            raise ValueError("DependencyGraph must be injected - no fallback allowed")
        
        self.session_manager = session_manager
        self.tree_builder = tree_builder
        self.dependency_graph = dependency_graph
    
    def expand_tree(self, session_id: str, tree_id: str, node_id: str) -> bool:
        # Get session and tree
        session = self.session_manager.get_session(session_id)
        tree = session.get_tree(tree_id)
        
        # Find and expand node
        node = self._find_node(tree, node_id)
        if node and not node.expanded:
            # Use dependency analysis for intelligent expansion
            self._expand_node_with_dependencies(node, tree)
            self.session_manager.save_session(session)
            return True
        return False
    
    def _expand_node_with_dependencies(self, node: TreeNode, tree: ExplorationTree) -> None:
        # Get actual dependencies from dependency graph
        file_path = self._extract_file_path(node.location)
        if file_path:
            dependencies = self.dependency_graph.get_dependencies(file_path)
            for dep in dependencies:
                child = TreeNode(
                    identifier=dep.name,
                    location=dep.location,
                    node_type=dep.type,
                    depth=node.depth + 1
                )
                child.parent = node
                node.children.append(child)
        node.expanded = True
```

### **✅ Dependency Analysis Best Practices**
```python
# CORRECT - Graph-based with centrality analysis
class GoodDependencyAnalyzer:
    def __init__(self, import_analyzer: ImportAnalyzer, centrality_calculator: CentralityCalculator):
        if import_analyzer is None:
            raise ValueError("ImportAnalyzer must be injected - no fallback allowed")
        if centrality_calculator is None:
            raise ValueError("CentralityCalculator must be injected - no fallback allowed")
        
        self.graph = nx.DiGraph()
        self.import_analyzer = import_analyzer
        self.centrality_calculator = centrality_calculator
    
    def build_dependency_graph(self, project_imports: ProjectImports) -> None:
        # Add nodes for each file
        for file_path, imports in project_imports.files.items():
            self.graph.add_node(file_path, imports=imports)
        
        # Add edges for dependencies
        for file_path, imports in project_imports.files.items():
            for import_stmt in imports.imports:
                if import_stmt.resolved_path:
                    self.graph.add_edge(file_path, import_stmt.resolved_path)
    
    def find_critical_files(self, top_n: int = 10) -> List[Tuple[str, float]]:
        # Calculate centrality scores
        centrality_scores = self.centrality_calculator.calculate_centrality(self.graph)
        
        # Sort by centrality and return top N
        sorted_files = sorted(centrality_scores.items(), key=lambda x: x[1], reverse=True)
        return sorted_files[:top_n]
    
    def analyze_change_impact(self, changed_file: str) -> ImpactReport:
        # Find all affected files
        affected_files = list(nx.descendants(self.graph, changed_file))
        
        # Calculate impact metrics
        impact_score = len(affected_files) / len(self.graph.nodes())
        risk_level = "high" if impact_score > 0.3 else "medium" if impact_score > 0.1 else "low"
        
        return ImpactReport(
            changed_file=changed_file,
            affected_files=affected_files,
            impact_score=impact_score,
            risk_level=risk_level
        )
```

### **✅ Caching Best Practices**
```python
# CORRECT - Multi-level caching with TTL and statistics
class GoodCacheManager:
    def __init__(self, max_size: int = 1000, ttl: int = 3600):
        self.max_size = max_size
        self.ttl = ttl
        self.cache: Dict[str, CacheEntry] = {}
        self.access_times: Dict[str, float] = {}
        self.hit_count = 0
        self.miss_count = 0
    
    def get(self, key: str) -> Optional[Any]:
        if key not in self.cache:
            self.miss_count += 1
            return None
        
        entry = self.cache[key]
        if self._is_expired(entry):
            self._remove(key)
            self.miss_count += 1
            return None
        
        # Update access time for LRU
        self.access_times[key] = time.time()
        self.hit_count += 1
        return entry.value
    
    def set(self, key: str, value: Any) -> None:
        # Check size limit
        if len(self.cache) >= self.max_size:
            self._evict_lru()
        
        # Store with TTL
        self.cache[key] = CacheEntry(value=value, created_at=time.time())
        self.access_times[key] = time.time()
    
    def get_stats(self) -> Dict[str, Any]:
        total_requests = self.hit_count + self.miss_count
        hit_rate = self.hit_count / total_requests if total_requests > 0 else 0
        
        return {
            "size": len(self.cache),
            "max_size": self.max_size,
            "hit_count": self.hit_count,
            "miss_count": self.miss_count,
            "hit_rate": hit_rate,
            "ttl": self.ttl
        }
    
    def _is_expired(self, entry: CacheEntry) -> bool:
        return time.time() - entry.created_at > self.ttl
```

## 🧪 **TESTING ARCHITECTURE**

### **Unit Testing Pattern**
```python
def test_output_manager():
    """Test OutputManager with proper mocking."""
    # Mock dependencies
    mock_console_manager = Mock(spec=ConsoleManager)
    mock_formatter_registry = Mock(spec=FormatterRegistry)
    
    # Create OutputManager with mocked dependencies
    output_manager = OutputManager(
        console_manager=mock_console_manager,
        formatter_registry=mock_formatter_registry
    )
    
    # Test functionality
    result = output_manager.display(test_data, OutputFormat.TEXT)
    assert result is not None
```

### **Integration Testing Pattern**
```python
def test_cli_command_integration():
    """Test CLI command with real OutputManager."""
    # Use service factory (same as production)
    service_factory = get_service_factory()
    service = service_factory.create_service(test_config)
    
    # Test end-to-end functionality
    result = service.process()
    assert result is not None
```

## 📁 **FILE ORGANIZATION**

### **Module Organization Structure**
```
src/repomap_tool/
├── core/                       # Core orchestration and infrastructure
│   ├── container.py            # DI container
│   ├── repo_map.py             # Main service orchestration
│   └── search_engine.py        # Search orchestration
├── code_analysis/              # Code analysis and dependency analysis
│   ├── dependency_graph.py     # Graph-based dependency analysis
│   ├── centrality_calculator.py # Centrality analysis
│   └── impact_analyzer.py      # Impact analysis
├── code_search/                # Code search and matching
│   ├── fuzzy_matcher.py        # Fuzzy string matching
│   ├── semantic_matcher.py     # Semantic matching
│   └── hybrid_matcher.py       # Hybrid matching strategies
├── code_exploration/           # Tree exploration and session management
│   ├── tree_manager.py         # Tree operations
│   ├── session_manager.py      # Session state management
│   └── tree_builder.py         # Tree construction
├── cli/                        # CLI interface and controllers
│   ├── controllers/            # MVC controllers
│   │   ├── base_controller.py  # Base controller class
│   │   ├── centrality_controller.py # Centrality analysis controller
│   │   ├── impact_controller.py # Impact analysis controller
│   │   └── view_models.py      # ViewModel definitions
│   └── output/                 # Output management system
│       ├── manager.py          # OutputManager (main entry point)
│       ├── console_manager.py  # Console management
│       ├── protocols.py        # Formatter protocols
│       ├── standard_formatters.py # Built-in formatters
│       ├── controller_formatters.py # ViewModel formatters
│       └── templates/          # Template system
│           ├── engine.py       # Jinja2 template engine
│           ├── loader.py       # Template loading
│           ├── registry.py     # Template registry
│           └── jinja/          # Jinja2 template files
│               ├── centrality_analysis.jinja2
│               ├── impact_analysis.jinja2
│               ├── project_info.jinja2
│               └── search_response.jinja2
└── llm/                        # LLM optimization and enhancement
    ├── token_optimizer.py      # Token budget management
    ├── context_selector.py     # Context selection strategies
    └── hierarchical_formatter.py # LLM-friendly formatting
```

## 🔄 **MIGRATION GUIDELINES**

### **From Legacy to New Architecture**

#### **Step 1: Replace Direct Console Usage**
```python
# OLD
console.print("Message")

# NEW
output_manager = get_output_manager()
output_manager.display_success("Message", ctx)
```

#### **Step 2: Use Service Factory**
```python
# OLD
service = SomeService(config)

# NEW
service_factory = get_service_factory()
service = service_factory.create_service(config)
```

#### **Step 3: Implement Formatters**
```python
# OLD
def display_data(data):
    print(f"Data: {data}")

# NEW
class DataFormatter(BaseFormatter):
    def format(self, data: Data, output_format: OutputFormat) -> str:
        return self._template_engine.render_template("data.jinja2", {"data": data})
```

## 🎯 **AI DEVELOPMENT QUALITY GATES**

### **AI Must Verify Before Completing Any Task**
- [ ] **ALL code analysis uses tree-sitter directly** - AI must never use custom AST parsing
- [ ] **NO custom AST parsing implementations** - AI must use tree-sitter capabilities
- [ ] **NO regex-based code parsing** - AI must use TreeSitterParser.get_tags() for all languages
- [ ] **ALL analyzers use TreeSitterParser.get_tags()** - AI must leverage tree-sitter
- [ ] **ALL analyzers use TreeSitterParser** - AI must use tree-sitter directly
- [ ] **ALL file paths are absolute** - AI must use absolute paths throughout the system
- [ ] **NO relative path conversions** - AI must only convert to relative for tree-sitter calls
- [ ] **ALL configuration values use centralized ConfigService** - AI must use `get_config(key, default)`
- [ ] **NO hardcoded configuration values** - AI must never hardcode thresholds, limits, timeouts
- [ ] **ALL logging uses centralized LoggingService** - AI must use `get_logger(__name__)`
- [ ] **NO direct `logging.getLogger()` calls** - AI must never use `logging.getLogger(__name__)` directly
- [ ] **ALL CLI commands use `OutputManager`** - AI must never use direct console.print
- [ ] **NO direct `console.print()` calls in CLI commands** - AI must use OutputManager
- [ ] **ALL services use constructor injection DI patterns** - AI must never instantiate services directly
- [ ] **NO direct service instantiation outside DI** - AI must use DI container
- [ ] **ALL constructors validate dependencies** - AI must raise errors for missing dependencies
- [ ] **ALL formatters implement `FormatterProtocol`** - AI must follow formatter interface
- [ ] **ALL output uses template system** - AI must use Jinja2 templates
- [ ] **ALL templates follow Jinja2 pattern standards** - AI must follow template patterns
- [ ] **ALL templates support configuration-driven rendering** - AI must use config.options
- [ ] **ALL templates provide emoji and non-emoji variants** - AI must support both
- [ ] **ALL matchers implement `MatcherProtocol` with caching** - AI must follow matcher interface
- [ ] **ALL matchers support multiple strategies** - AI must implement strategy pattern
- [ ] **ALL LLM optimizers use token budget management** - AI must respect token limits
- [ ] **ALL code exploration operations use session-based state management** - AI must use sessions
- [ ] **ALL dependency analysis uses graph-based representation** - AI must use NetworkX
- [ ] **ALL caching systems use TTL and size limits** - AI must implement proper caching
- [ ] **ALL controllers inherit from `BaseController`** - AI must follow MVC controller pattern
- [ ] **ALL controllers return ViewModels** - AI must not return raw strings or dictionaries
- [ ] **ALL ViewModels use template-based formatters** - AI must inherit from TemplateBasedFormatter
- [ ] **ALL controllers are registered in DI container** - AI must use existing DI container
- [ ] **ALL code has comprehensive tests** - AI must maintain >80% test coverage
- [ ] **ALL code passes MyPy type checking** - AI must ensure type safety
- [ ] **ALL code follows existing patterns** - AI must not reinvent the wheel
- [ ] **ALL new features use existing DI container** - AI must not create new factories

### **Code Quality**
- [ ] Full MyPy compliance (no type errors)
- [ ] All data models use Pydantic
- [ ] Comprehensive test coverage (>80%)
- [ ] No DI linter violations
- [ ] All formatting and linting passes

### **Performance**
- [ ] Template rendering is efficient
- [ ] Console operations are batched where possible
- [ ] Memory usage is optimized
- [ ] No unnecessary object creation

## 🚀 **EXTENSIBILITY GUIDELINES**

### **Adding New Output Formats**
1. Add format to `OutputFormat` enum
2. Implement formatter in `standard_formatters.py`
3. Register formatter in `_register_default_formatters()`
4. Create template file if needed
5. Add tests for new format

### **Adding New Formatters**
1. Implement `FormatterProtocol` interface
2. Extend `BaseFormatter` for common functionality
3. Register in `FormatterRegistry`
4. Add comprehensive tests
5. Update documentation

### **Adding New Controllers**
1. Create controller class inheriting from `BaseController`
2. Implement `execute` method with proper business logic orchestration
3. Define ViewModel classes for structured data representation
4. Create corresponding formatters inheriting from `TemplateBasedFormatter`
5. Register controller in DI container (`core/container.py`)
6. Create Jinja2 templates for ViewModel rendering
7. Update CLI commands to use new controller
8. Add comprehensive tests for controller and ViewModel
9. Document controller behavior and usage patterns

### **Adding New Templates**
1. Create Jinja2 template file following the standard pattern:
   - Configuration-driven rendering (`config.options`)
   - Emoji and non-emoji variants
   - Hierarchical and flat layout support
   - Conditional sections for optional data
   - Consistent formatting and indentation
2. Register in `TemplateRegistry`
3. Test template rendering with various configurations
4. Document template variables and context structure
5. Add examples in documentation
6. Ensure template follows all pattern rules

### **Adding New Matching Strategies**
1. Implement strategy method in existing matcher class
2. Add strategy name to configurable strategies list
3. Ensure strategy returns consistent score format (0-100)
4. Add caching support for strategy results
5. Add comprehensive tests for new strategy
6. Document strategy behavior and use cases

### **Adding New LLM Optimization Strategies**
1. Implement new `SelectionStrategy` enum value
2. Add strategy implementation in `ContextSelector`
3. Ensure strategy respects token budget limits
4. Add strategy-specific configuration options
5. Test with various data sizes and token limits
6. Document strategy behavior and performance characteristics

### **Adding New Code Exploration Operations**
1. Implement operation in `TreeManager` class
2. Ensure operation uses session-based state management
3. Add dependency-aware logic where appropriate
4. Implement proper error handling and validation
5. Add comprehensive tests for new operation
6. Update CLI commands to expose new operation

### **Adding New Dependency Analysis Features**
1. Implement feature using graph-based algorithms
2. Ensure feature integrates with existing `DependencyGraph`
3. Add centrality and impact analysis where relevant
4. Implement proper caching for expensive operations
5. Add comprehensive tests for new feature
6. Document feature behavior and performance implications

### **Adding New Caching Strategies**
1. Implement new cache strategy (LRU, LFU, etc.)
2. Ensure strategy supports TTL and size limits
3. Add cache statistics and monitoring
4. Implement proper eviction policies
5. Add comprehensive tests for new strategy
6. Document strategy behavior and performance characteristics

## 📋 **AI DEVELOPMENT WORKFLOW**

### **AI Must Follow This Workflow for Every Task**
1. **Read existing code patterns** - AI must understand current architecture
2. **Use existing DI container** - AI must not create new service factories
3. **Follow OutputManager pattern** - AI must never use direct console.print
4. **Implement proper error handling** - AI must use custom exception hierarchy
5. **Write comprehensive tests** - AI must maintain >80% test coverage
6. **Run CI pipeline** - AI must ensure `make ci-all` passes
7. **Verify all quality gates** - AI must check all compliance requirements
8. **Update documentation** - AI must document new patterns and changes

### **AI Development Checklist**
- [ ] **Read and understand existing patterns** before writing new code
- [ ] **Use existing DI container** for all service creation
- [ ] **Use centralized configuration service** - always use `get_config(key, default)`
- [ ] **Use centralized logging service** - always use `get_logger(__name__)`
- [ ] **Follow OutputManager pattern** for all CLI output
- [ ] **Implement proper error handling** with custom exceptions
- [ ] **Write comprehensive tests** for all new functionality
- [ ] **Run full CI pipeline** (`make ci-all`) before completing
- [ ] **Verify all quality gates** are met
- [ ] **Update documentation** for any new patterns
- [ ] **Follow existing naming conventions** and code style
- [ ] **Use existing protocols and interfaces** - don't create new ones
- [ ] **Implement proper caching** with TTL and statistics
- [ ] **Use existing template system** for all output formatting
- [ ] **Follow existing matcher patterns** for any new matching logic
- [ ] **Use existing tree exploration patterns** for any tree operations
- [ ] **Use existing dependency analysis patterns** for any graph operations

### **AI Must Never Do These Things**
- [ ] **Never instantiate services directly** - always use DI container
- [ ] **Never hardcode configuration values** - always use `get_config(key, default)`
- [ ] **Never use direct logging.getLogger()** - always use `get_logger(__name__)`
- [ ] **Never use direct console.print** - always use OutputManager
- [ ] **Never skip dependency validation** - always raise errors for missing dependencies
- [ ] **Never create new service factories** - always use existing DI container
- [ ] **Never bypass the template system** - always use Jinja2 templates
- [ ] **Never skip error handling** - always implement proper exception handling
- [ ] **Never skip tests** - always write comprehensive tests
- [ ] **Never ignore CI failures** - always fix all CI issues
- [ ] **Never create new protocols** - always use existing interfaces
- [ ] **Never skip caching** - always implement proper caching patterns

---

## 🤖 **AI DEVELOPMENT SUMMARY**

**For AI (Cursor):** These rules are your development guidelines. Follow them strictly to maintain code quality and architectural consistency.

### **🚨 CRITICAL: This Project Uses Tree-Sitter Directly**
**The fundamental purpose of RepoMap-Tool is to provide a CLI interface and enhanced functionality using tree-sitter directly. We leverage tree-sitter's capabilities for all code parsing and analysis.**

### **Key AI Rules:**
1. **ALWAYS use tree-sitter directly** - this is the core of the project, never bypass it
2. **ALWAYS use tree-sitter capabilities** - never implement custom AST parsing
3. **ALWAYS use TreeSitterParser.get_tags()** - for all code element extraction
4. **ALWAYS use TreeSitterParser** - for all code parsing operations
5. **ALWAYS use centralized configuration** - use `get_config(key, default)` instead of hardcoded values
6. **ALWAYS use centralized logging** - use `get_logger(__name__)` instead of `logging.getLogger(__name__)`
7. **ALWAYS use dependency injection** - never instantiate services directly
8. **ALWAYS use OutputManager** - never use direct console.print in CLI commands
9. **ALWAYS validate dependencies** - raise errors for missing dependencies
10. **ALWAYS follow existing patterns** - don't reinvent the wheel
11. **ALWAYS write comprehensive tests** - maintain >80% test coverage
12. **ALWAYS run CI pipeline** - ensure `make ci-all` passes
13. **ALWAYS use existing DI container** - don't create new service factories
14. **ALWAYS follow template system** - use Jinja2 templates for output
15. **ALWAYS implement proper error handling** - use custom exception hierarchy
16. **ALWAYS follow existing protocols** - implement existing interfaces

### **AI Success Criteria:**
- ✅ All code analysis uses tree-sitter directly (no custom AST parsing)
- ✅ All analyzers use tree-sitter capabilities via TreeSitterParser.get_tags()
- ✅ All parsing operations use TreeSitterParser
- ✅ All file paths are absolute throughout the system
- ✅ All configuration values use centralized ConfigService (no hardcoded values)
- ✅ All logging uses centralized LoggingService (no direct logging.getLogger())
- ✅ All code follows existing architectural patterns
- ✅ All services use proper dependency injection
- ✅ All CLI output goes through OutputManager
- ✅ All code has comprehensive test coverage
- ✅ All code passes CI pipeline (`make ci-all`)
- ✅ All code follows type safety requirements
- ✅ All code follows existing naming conventions
- ✅ All new features integrate with existing systems

**Remember:** Consistency and adherence to existing patterns is more important than innovation. Follow these rules to maintain the high quality and architectural integrity of the RepoMap-Tool codebase.

---
> Source: [xynova/repomap-tool](https://github.com/xynova/repomap-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
