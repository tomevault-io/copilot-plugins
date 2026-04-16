## simple-graphics-pipeline

> This is a Java 21 graphics project using LWJGL 3.3.3 and OpenGL for real-time rendering. The project follows professional development practices with branch protection, security scanning, and automated CI/CD.

# Cursor Rules for Simple Graphics Pipeline Project

## Project Context
This is a Java 21 graphics project using LWJGL 3.3.3 and OpenGL for real-time rendering. The project follows professional development practices with branch protection, security scanning, and automated CI/CD.

## Code Style & Standards

### Java Conventions
- Use Java 21 features where appropriate (pattern matching, records, etc.)
- Follow conventional commit format: `feat:`, `fix:`, `docs:`, `style:`, `refactor:`, `perf:`, `test:`, `chore:`, `ci:`
- Maintain 120 character line limit (Checkstyle enforced)
- Use meaningful variable names, especially for graphics concepts (vertices, shaders, buffers)
- Prefer composition over inheritance for graphics components

### Graphics-Specific Patterns
- Always check for OpenGL errors after GL calls in debug mode
- Use RAII pattern for OpenGL resources (create in constructor, cleanup in close/dispose)
- Prefer VBOs (Vertex Buffer Objects) over immediate mode rendering
- Use uniform buffer objects for frequently updated shader data
- Implement proper resource cleanup to prevent memory leaks
- Handle LWJGL native memory explicitly with try-with-resources where possible

### LWJGL Best Practices
- Always call `glfwInit()` before any GLFW functions
- Check for GLFW and OpenGL initialization errors
- Use `MemoryStack` for temporary native memory allocation
- Prefer `BufferUtils` for persistent native buffers
- Use appropriate OpenGL version context (3.3 core profile minimum)

## Git Workflow

### Branch Management
- Use feature branches for new graphics features: `feature/add-lighting-system`
- Use fix branches for bugs: `fix/shader-compilation-error`
- Follow the branch protection rules and require PR reviews
- Use the git workflow script: `./scripts/git-workflow.sh feature <name>`

### Pre-Push Testing (MANDATORY)
**ALWAYS run local tests before pushing to prevent CI failures:**

```bash
# Required before every push
./scripts/validate-workflow.sh  # Check YAML syntax and structure
./scripts/test-ci-locally.sh    # Test all CI commands locally
```

**For complex changes, also run:**
```bash
# Full GitHub Actions simulation (requires Docker)
act -W .github/workflows/graphics-ci.yml --job graphics-context-test
act -W .github/workflows/ci.yml --job compile-and-test
```

**Testing Checklist:**
- [ ] YAML workflows validate without errors
- [ ] Local compilation succeeds (`./gradlew compileJava compileTestJava`)
- [ ] Unit tests run without critical failures
- [ ] JAR builds successfully
- [ ] Graphics tests complete (warnings acceptable locally)
- [ ] All scripts execute without syntax errors

### Commit Messages
```
feat: add PBR material system with metallic-roughness workflow
fix: resolve shader compilation error on Intel graphics drivers
perf: optimize vertex buffer updates for dynamic geometry
docs: add API documentation for lighting system
```

### Development Workflow
1. **Create feature branch**: `git checkout -b feature/my-feature`
2. **Make changes**: Edit code, tests, documentation
3. **Test locally**: Run `./scripts/test-ci-locally.sh`
4. **Validate workflows**: Run `./scripts/validate-workflow.sh` 
5. **Commit changes**: Use conventional commit format
6. **Final test**: Run both scripts again if significant changes
7. **Push**: `git push origin feature/my-feature`
8. **Create PR**: Use GitHub web interface
9. **Review CI**: Ensure all checks pass in PR

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GAMINpanda) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
