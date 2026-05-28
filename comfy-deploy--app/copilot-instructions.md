## backend-rules

> - Uses multi-level caching decorator from `api.utils.cache`

 # Codebase Query Handling Documentation

## Backend Caching Pattern
- Uses multi-level caching decorator from `api.utils.cache`
- Caches in both local memory and Redis
- Example structure:
```python
@multi_level_cached(
    key_prefix="custom_nodes_version",
    ttl_seconds=60,  # Local memory cache
    redis_ttl_seconds=300,  # Redis cache
    version="1.0",
    key_builder=lambda repo_name: f"custom_nodes_version:{repo_name}"
)
```

## API Endpoints
- Machine-related endpoints are in `backend/apps/api/src/api/routes/machines.py`
- Custom nodes version check endpoint: `/api/machine/{machine_id}/check-custom-nodes`
- Response format includes status, local commit info, and latest commit info

## Error Handling
- Always wrap external API calls in try/except blocks
- Use specific exception types when possible
- Log errors with proper context
- Return appropriate HTTP status codes
```python
try:
    # API logic here
except HTTPException:
    raise  # Re-raise HTTP exceptions as is
except Exception as e:
    logger.error(f"Error context: {str(e)}")
    raise HTTPException(
        status_code=500,
        detail=f"Specific error message: {str(e)}"
    )
```

## Database Operations
- Use async SQLAlchemy operations
- Always check user permissions with apply_org_check
- Handle transactions properly
- Refresh objects after commits when needed
```python
machine = await db.execute(
    select(Machine)
    .where(Machine.id == machine_id)
    .where(~Machine.deleted)
    .apply_org_check(request)
)
machine = machine.scalars().first()
if not machine:
    raise HTTPException(status_code=404, detail="Machine not found")
```

## Custom Node Management
- Always include install_type for custom nodes
- Handle git-based installations properly
- Ensure proper ordering of build steps
- Example custom node structure:
```python
custom_node = {
    "type": "custom-node",
    "data": {
        "url": "https://github.com/org/repo",
        "name": "Node Name",
        "hash": commit_hash,
        "install_type": "git-clone",
        "files": ["https://github.com/org/repo"]
    }
}
```

## Machine Updates
- Validate machine type before operations
- Handle version management properly
- Trigger rebuilds when necessary
- Update machine hash after changes
```python
if machine.type != MachineType.COMFY_DEPLOY_SERVERLESS:
    raise HTTPException(
        status_code=400,
        detail="Machine is not a Comfy Deploy Serverless machine"
    )
```

## Plan Restrictions
- Check user plan before operations
- Validate allowed features
- Handle free plan limitations
- Example validation:
```python
plan = request.state.current_user.get("plan")
if plan == "free":
    # Check restrictions
    await validate_free_plan_restrictions(
        request=request,
        machine_data=machine_data,
        db=db
    )
```
// ... existing code ...

---
> Source: [comfy-deploy/app](https://github.com/comfy-deploy/app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
