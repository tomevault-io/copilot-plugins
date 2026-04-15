## neurovision-app

> NeuroVision is an advanced educational neuroscience visualization platform built with Godot 4.4.1, designed specifically for medical students, neuroscience researchers, and healthcare professionals to learn neuroanatomy through interactive 3D brain exploration.

# NeuroVision Educational Platform - Cursor AI Rules

## Project Context
NeuroVision is an advanced educational neuroscience visualization platform built with Godot 4.4.1, designed specifically for medical students, neuroscience researchers, and healthcare professionals to learn neuroanatomy through interactive 3D brain exploration.

## Critical Requirements
- **Medical Accuracy**: All anatomical terms, structures, and descriptions must be medically accurate and validated
- **Performance**: Maintain 60fps for 3D brain interactions, <3s loading time, <500MB memory usage
- **Accessibility**: WCAG 2.1 AA compliance is mandatory for all UI components
- **Target Audience**: Medical students (primary), neuroscience researchers, healthcare professionals
- **Educational Standards**: Content must align with medical education curriculum standards

## Technical Stack
- **Engine**: Godot 4.4.1.stable.official
- **Language**: GDScript (primary), no C# or other languages
- **Platform**: Cross-platform (Windows, macOS, Linux)
- **Architecture**: Autoload-based singleton pattern with atomic UI design

## Code Standards
### Formatting
- **Indentation**: Use TABS (not spaces) with tab size of 4
- **Line Length**: Soft limit at 100 characters
- **File Encoding**: UTF-8 with LF line endings

### Naming Conventions
- **Classes**: PascalCase (e.g., `BrainStructureExplorer`)
- **Functions**: snake_case (e.g., `get_brain_structure()`)
- **Variables**: snake_case (e.g., `structure_name`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_FPS`)
- **Signals**: snake_case with descriptive verbs (e.g., `structure_selected`)

### Comments
- Minimal comments - code should be self-documenting
- Only add comments for complex medical algorithms or educational logic
- Use GDScript docstrings for public API methods

## Architecture Guidelines

### Autoload Systems (Critical - Always Available)
These singleton services are always available and should be used:
- `UnifiedColorManager` - Educational theme management (Enhanced/Minimal modes)
- `CoreSystemManager` - Core platform coordination
- `UISystemManager` - Educational UI management
- `EducationalPlatformManager` - Learning workflow coordination
- `KnowledgeService` - Medical content and anatomical data access
- `ProgressTracker` - Learning analytics and progression tracking
- `AuthenticationManager` - Student/professional access control
- `ResourceManager` - Educational asset loading
- `AssessmentService` - Educational testing and evaluation
- `HighlightMaterialManager` - 3D brain interaction feedback

### File Organization Rules
```
src/
  autoload/           # Singleton services (EDIT these, don't create new)
  core/               # Business logic and educational systems
    managers/         # Core management systems
    interaction/      # 3D brain interaction logic
  systems/            # Specialized systems
    3d_interaction/   # Brain model interaction
    assessment/       # Educational assessment tools
  ui_atomic/          # Atomic design UI components
    atoms/            # Basic UI elements
    molecules/        # Combined UI components
    organisms/        # Complex UI systems
  utils/              # Utility functions and helpers
scenes/               # Godot scene files (.tscn)
  3d/                 # 3D brain exploration scenes
  ui/                 # Educational UI scenes
assets/               # Resources
  3d_models/          # Brain models (.glb, .gltf)
  materials/          # Visual materials and shaders
  data/               # Educational content databases
```

### Development Patterns

#### When to EDIT vs CREATE files
**EDIT existing files when:**
- Adding features to existing UI components
- Extending autoload functionality
- Modifying brain interaction behaviors
- Fixing bugs or optimizing performance
- Adding educational content to existing structures

**CREATE new files only when:**
- Implementing completely new educational modules
- Adding new assessment tool types
- Creating independent learning pathways
- Building new accessibility features
- Adding new brain visualization modes

#### Integration Pattern
```gdscript
# Always validate autoload availability
func _ready() -> void:
    if not _validate_educational_systems():
        push_error("[Component] Required systems not available")
        return
    
    # Register with platform
    EducationalPlatformManager.register_component(self)
    
    # Apply theme
    if UnifiedColorManager:
        UnifiedColorManager.theme_changed.connect(_on_theme_changed)
```

## Educational Content Standards

### Medical Terminology
- Use standardized anatomical terminology (Terminologia Anatomica)
- Include both common and clinical terms where appropriate
- Provide pronunciation guides for complex terms
- Cross-reference with medical education standards

### Content Structure
- Each brain structure must include:
  - Anatomical name and aliases
  - Location and boundaries
  - Functions (cognitive, motor, sensory)
  - Clinical relevance and pathologies
  - Learning objectives
  - Assessment questions

### Accessibility Requirements
- All text must meet WCAG 2.1 AA contrast ratios
- Interactive elements minimum 44x44 pixels
- Keyboard navigation for all features
- Screen reader compatibility
- Alternative text for visual elements

## Performance Optimization

### Critical Metrics
- **Frame Rate**: 60fps minimum during interactions
- **Load Time**: <3 seconds to interactive state
- **Memory**: <500MB for full brain model
- **Input Latency**: <16ms response time

### Optimization Strategies
- Use LOD (Level of Detail) for complex brain models
- Implement occlusion culling for hidden structures
- Pool frequently created objects
- Lazy load educational content
- Cache anatomical data queries

## Testing Requirements

### Debug Commands (F1 Console)
Always test with these commands:
- `test_educational_systems` - Validate all autoloads
- `test_medical_accuracy` - Check content accuracy
- `test_accessibility_compliance` - WCAG validation
- `performance` - Check FPS and memory

### Validation Checklist
Before considering any feature complete:
1. Medical accuracy reviewed
2. Educational effectiveness validated
3. Accessibility compliance verified
4. Performance targets met
5. Integration with autoloads confirmed
6. Theme compatibility tested (Enhanced/Minimal)
7. Progress tracking integrated

## Error Handling

### Educational System Errors
```gdscript
enum EducationalError {
    MEDICAL_INACCURACY,      # Critical - disable feature
    ACCESSIBILITY_VIOLATION,  # Apply fallback mode
    PERFORMANCE_DEGRADATION, # Switch to simplified mode
    CONTENT_MISSING,         # Show fallback content
}
```

### Response Patterns
- Medical errors: Log for review, disable feature
- Accessibility errors: Apply emergency fallback
- Performance errors: Degrade gracefully
- Content errors: Use cached/fallback content

## Prohibited Actions
- Never expose patient data or medical records
- Don't implement features that could provide medical advice
- Avoid creating files without explicit necessity
- Don't use `print()` statements (use `push_error`/`push_warning`)
- Never bypass accessibility requirements
- Don't hardcode medical content (use KnowledgeService)

## When Providing Code
- Always include file paths with line numbers (e.g., `src/core/managers/CoreSystemManager.gd:45`)
- Validate medical terminology before using
- Consider accessibility implications
- Check performance impact
- Ensure autoload integration
- Support both Enhanced and Minimal themes

## Common Gotchas
- Godot signals use `signal_name.connect(callable)` not `connect("signal_name", callable)`
- Use `@onready` for node references, not `get_node()` in `_ready()`
- Theme changes must be handled dynamically
- Medical content must be validated before display
- Always check autoload availability before use

## Priority Guidelines
When making decisions, prioritize in this order:
1. Medical accuracy and safety
2. Educational effectiveness
3. Accessibility compliance
4. Performance optimization
5. Code maintainability

Remember: This is an educational medical platform. Every decision should enhance learning while maintaining medical accuracy and accessibility.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laportagm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
