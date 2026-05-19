## feature-flags

> Lightweight organization-level feature flags for gating features. Table name is `feature_flag` (org is implied).


# Feature Flags

## Overview
Lightweight organization-level feature flags for gating features. Table name is `feature_flag` (org is implied).

## Backend Usage

### Enum (`FeatureFlag`)
```python
from airweave.core.shared_models import FeatureFlag

# Check in endpoints via ApiContext
if not ctx.has_feature(FeatureFlag.S3_DESTINATION):
    raise HTTPException(403, "Feature not available")
```

### CRUD Operations
Integrated into `crud.organization`:
```python
from airweave import crud

# Enable/disable flags
await crud.organization.enable_feature(db, org_id, FeatureFlag.S3_DESTINATION)
await crud.organization.disable_feature(db, org_id, FeatureFlag.WHITE_LABEL)

# Query flags
flags = await crud.organization.get_org_features(db, org_id)

# Bulk operations
await crud.organization.bulk_enable_features(db, org_id, [
    FeatureFlag.S3_DESTINATION,
    FeatureFlag.PRIORITY_SUPPORT
])
```

### Schema Handling
Organization schemas automatically extract `enabled_features` from the `feature_flags` relationship using Pydantic validators (handles async context properly).

## Frontend Usage
```typescript
import { useOrganizationStore } from '@/lib/stores/organizations';
import { FeatureFlags } from '@/lib/constants/feature-flags';

const hasFeature = useOrganizationStore((state) => state.hasFeature);

{hasFeature(FeatureFlags.S3_DESTINATION) && <S3DestinationCard />}
```

## Adding New Flags
1. Add to `FeatureFlag` enum in `backend/airweave/core/shared_models.py`
2. Add to `FeatureFlags` constants in `frontend/src/lib/constants/feature-flags.ts`
3. Enable for organizations via CRUD or admin panel


# Feature Flags

## Overview
Lightweight organization-level feature flags for gating features. Table name is `feature_flag` (org is implied).

## Backend Usage

### Enum (`FeatureFlag`)
```python
from airweave.core.shared_models import FeatureFlag

# Check in endpoints via ApiContext
if not ctx.has_feature(FeatureFlag.S3_DESTINATION):
    raise HTTPException(403, "Feature not available")
```

### CRUD Operations
Integrated into `crud.organization`:
```python
from airweave import crud

# Enable/disable flags
await crud.organization.enable_feature(db, org_id, FeatureFlag.S3_DESTINATION)
await crud.organization.disable_feature(db, org_id, FeatureFlag.WHITE_LABEL)

# Query flags
flags = await crud.organization.get_org_features(db, org_id)

# Bulk operations
await crud.organization.bulk_enable_features(db, org_id, [
    FeatureFlag.S3_DESTINATION,
    FeatureFlag.PRIORITY_SUPPORT
])
```

### Schema Handling
Organization schemas automatically extract `enabled_features` from the `feature_flags` relationship using Pydantic validators (handles async context properly).

## Frontend Usage
```typescript
import { useOrganizationStore } from '@/lib/stores/organizations';
import { FeatureFlags } from '@/lib/constants/feature-flags';

const hasFeature = useOrganizationStore((state) => state.hasFeature);

{hasFeature(FeatureFlags.S3_DESTINATION) && <S3DestinationCard />}
```

## Adding New Flags
1. Add to `FeatureFlag` enum in `backend/airweave/core/shared_models.py`
2. Add to `FeatureFlags` constants in `frontend/src/lib/constants/feature-flags.ts`
3. Enable for organizations via CRUD or admin panel

---
> Source: [airweave-ai/airweave](https://github.com/airweave-ai/airweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
