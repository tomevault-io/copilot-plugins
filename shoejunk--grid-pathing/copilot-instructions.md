## grid-pathing

> When working with this C++ codebase, follow these strict naming conventions:

# AI Agent Rules for C++ Development

## Coding Style Guidelines

When working with this C++ codebase, follow these strict naming conventions:

### Naming Conventions
- **Use snake_case** for all identifiers (variables, functions, etc.)
- **Classes**: Prefix with `c_` (e.g., `c_grid_manager`, `c_path_finder`)
- **Member variables**: Prefix with `m_` (e.g., `m_width`, `m_height`, `m_grid_data`)
- **Enums**: Prefix with `e_` (e.g., `e_direction`, `e_cell_type`)

### Examples
```cpp
// Class definition
class c_grid_pathing {
private:
    int m_width;
    int m_height;
    std::vector<int> m_grid_data;
    
public:
    void calculate_path();
    bool is_valid_position(int x, int y);
};

// Enum definition
enum class e_direction {
    north,
    south,
    east,
    west
};

// Function names
void process_grid_data();
bool validate_input_parameters();
```

### Important Notes
- Always maintain consistency with existing code
- Apply these rules to all new code generation
- When refactoring, ensure all identifiers follow these conventions
- Use descriptive names that clearly indicate purpose

## Code Quality Standards
- Prefer modern C++ features (C++20 modules as evidenced by .ixx files)
- Use clear, self-documenting code
- Follow RAII principles
- Maintain const-correctness

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shoejunk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shoejunk)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
