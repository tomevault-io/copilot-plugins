## meteor-simulator-cuda

> You are working on the NASA Space Apps Challenge 2025 "Meteor Madness" project - an asteroid impact visualization and simulation tool. Multiple developers are working on the same user account simultaneously.

# Cursor AI Rules for NASA Space Apps "Meteor Madness" Project
# Same-User Multi-Developer Environment

## PROJECT CONTEXT
You are working on the NASA Space Apps Challenge 2025 "Meteor Madness" project - an asteroid impact visualization and simulation tool. Multiple developers are working on the same user account simultaneously.

## CRITICAL FILE CONFLICT PREVENTION

### Before Editing Any File
- ALWAYS check file modification time: `ls -la [filename]` or `stat [filename]`
- Compare with when you last looked at the file
- If modified recently by someone else, coordinate before editing
- Add a comment at the top with your name and current time when starting edits

### File Editing Protocol
```bash
# Check if file was recently modified
stat backend/app/main.py

# When starting to edit, add comment at top:
# EDITING: [Your Name] at [Time] - [What you're doing]

# When done editing, remove the comment and save
```

### Resource Management
- Check GPU before CUDA operations: `nvidia-smi`
- Don't run multiple physics simulations simultaneously
- Monitor system resources: `htop`
- Kill your processes when switching tasks: `pkill -f [process_name]`
- Clean up temp files in `/tmp/`

### Code Coordination
- Use descriptive commit messages frequently
- Save and commit small changes often to avoid conflicts
- Don't leave files open in editors when not actively editing
- Announce verbally when working on critical files

## PROJECT-SPECIFIC CODING STANDARDS

### Python Backend (FastAPI + CUDA)
- Use Python 3.10+ with type hints for all functions
- Follow PEP 8 with line length max 100 characters
- Use async/await for all API endpoints and database operations
- Implement comprehensive error handling with try/except blocks
- Use Pydantic models for all API request/response validation
- Add docstrings to all public functions with parameter descriptions
- Use logging instead of print statements
- Test CUDA availability before using GPU operations
- Implement CPU fallbacks for all CUDA operations

### CUDA Development Guidelines
- Always check CUDA device availability with error handling
- Use CuPy for array operations, Numba for custom kernels
- Implement memory management to prevent GPU memory leaks
- Use proper grid/block sizing based on device capabilities
- Include performance profiling and optimization comments
- Add CPU fallback implementations for all GPU operations
- Test kernels with small datasets before scaling up

### NASA API Integration
- Cache API responses for 1 hour minimum to avoid rate limits
- Implement exponential backoff for failed requests
- Validate all NASA data before processing
- Use environment variables for API keys
- Add request timeout of 30 seconds maximum
- Log all API requests for debugging
- Handle API quota limits gracefully

### React Frontend (Three.js + Material-UI)
- Use TypeScript for all components with strict type checking
- Follow React Hooks patterns - no class components
- Use functional components with proper dependency arrays
- Implement error boundaries for all major components
- Use Material-UI theme consistently across all components
- Optimize Three.js performance with LOD and culling
- Add accessibility attributes to all interactive elements
- Use React.memo for expensive computational components

### Three.js 3D Visualization
- Dispose of geometries and materials to prevent memory leaks
- Use object pooling for frequently created/destroyed objects
- Implement level-of-detail (LOD) for asteroid rendering
- Use frustum culling and occlusion culling for performance
- Limit particle systems to prevent frame rate drops
- Use worker threads for heavy computational tasks
- Implement proper camera controls with smooth transitions

## SCIENTIFIC ACCURACY REQUIREMENTS

### Physics Calculations
- Use validated crater scaling laws from peer-reviewed sources
- Implement proper unit conversions (meters, km/s, joules)
- Reference scientific papers in comments for formulas
- Validate calculations against known impact events (Tunguska, Chelyabinsk)
- Use appropriate significant figures for scientific data
- Include uncertainty estimates where applicable
- Add bounds checking for all physical parameters

### Data Validation
- Verify asteroid parameters are within realistic ranges
- Check orbital mechanics calculations for conservation laws
- Validate impact energy calculations with known events
- Ensure trajectory calculations follow Keplerian orbits
- Cross-reference with published asteroid databases
- Implement data quality checks and warnings

## SECURITY & DEPLOYMENT

### Environment Management
- Use `.env` files for all configuration variables
- Never commit API keys, passwords, or sensitive data
- Use different environment configurations for dev/prod
- Implement proper CORS settings for API endpoints
- Use HTTPS for all external API communications
- Validate and sanitize all user inputs

### Performance Optimization
- Use connection pooling for database operations
- Implement request caching for expensive operations
- Use lazy loading for large 3D models and textures
- Optimize bundle size with code splitting
- Monitor memory usage and implement garbage collection
- Use CDN for static assets in production

## CODE ORGANIZATION

### File Structure
```
backend/
├── app/
│   ├── api/routes/     # FastAPI route handlers
│   ├── core/          # Configuration and utilities
│   ├── models/        # Pydantic data models
│   ├── physics/       # CUDA physics calculations
│   └── services/      # External API integrations
frontend/src/
├── components/
│   ├── 3d/           # Three.js visualization components
│   ├── ui/           # Material-UI interface components
│   └── controls/     # User input controls
├── services/         # API communication services
└── types/           # TypeScript type definitions
```

### Naming Conventions
- Python: snake_case for variables/functions, PascalCase for classes
- TypeScript: camelCase for variables/functions, PascalCase for components
- Files: kebab-case for component files, snake_case for Python modules
- Constants: SCREAMING_SNAKE_CASE in both languages
- Database: snake_case for table and column names

## TESTING REQUIREMENTS

### Unit Testing
- Write tests for all physics calculations with known expected results
- Mock external API calls in tests
- Test CUDA kernels with small datasets
- Use pytest for Python, Jest for React components
- Achieve minimum 80% code coverage
- Test error handling and edge cases

### Integration Testing
- Test full API endpoint flows
- Validate 3D rendering performance
- Test NASA API integration with real data
- Verify CUDA/CPU fallback behavior
- Test cross-browser compatibility
- Performance test with large datasets

## ERROR HANDLING PATTERNS

### Python Exception Handling
```python
try:
    # GPU operation
    result = cuda_function(data)
except CudaError as e:
    logger.error(f"CUDA operation failed: {e}")
    # Fallback to CPU
    result = cpu_fallback(data)
except Exception as e:
    logger.error(f"Unexpected error: {e}")
    raise HTTPException(status_code=500, detail="Internal error")
```

### React Error Boundaries
- Wrap all major components in error boundaries
- Display user-friendly error messages
- Log errors to console with stack traces
- Provide fallback UI for broken components
- Include error reporting mechanisms

## PERFORMANCE MONITORING

### Metrics to Track
- API response times (target: <500ms)
- 3D rendering frame rates (target: >30fps)  
- GPU memory usage (monitor for leaks)
- NASA API quota consumption
- Database query performance
- Bundle size and loading times

Remember: Multiple people are editing the same files. Always check modification times before editing and save frequently to avoid conflicts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/karimmufti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
