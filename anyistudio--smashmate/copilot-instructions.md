## smashmate

> API structure and schema

# SmashMate API Design Reference

## Overview

SmashMate API is built with FastAPI, featuring async operations, automatic OpenAPI documentation, and comprehensive type validation using Pydantic schemas.

**Base URL**: `http://localhost:8000`  
**API Version**: `/api/v1`  
**Documentation**: `/api/v1/docs` (Swagger UI)  
**OpenAPI Schema**: `/api/v1/openapi.json`

## Architecture

### Core Structure
```
app/
├── api/          # FastAPI route handlers
├── core/         # Business logic functions
├── schemas/      # Pydantic request/response models
├── services/     # Database and external service integrations
└── main.py       # FastAPI application entry point
```

### Design Principles
- **Async-first**: All database operations and API endpoints are async
- **Type Safety**: Full Pydantic validation for all requests and responses
- **Separation of Concerns**: Business logic in `core/`, API routing in `api/`
- **Error Handling**: Consistent HTTP exception handling across all endpoints
- **Documentation**: Auto-generated OpenAPI documentation

## API Endpoints

### Authentication (`/api/v1/auth`)
- `GET /auth/profile/{user_id}` - Get or create user profile
  - Query params: `display_name` (optional)
  - Response: `ProfileResponse`
- `PUT /auth/profile/{user_id}` - Update user profile
  - Body: `UpdateProfileRequest`
  - Response: `ProfileResponse`

### Matches (`/api/v1/matches`)
- `POST /matches/` - Create new match and update TrueSkill ratings
  - Query params: `created_by` (UUID)
  - Body: `CreateMatchRequest`
  - Response: `MatchResponse`
- `GET /matches/player/{player_id}` - Get player's match history
- `GET /matches/venue/{venue_id}` - Get venue's match history
- `GET /matches/leaderboard` - Get top players by rating
  - Query params: `limit` (default: 10)
  - Response: `List[PlayerRatingResponse]`

### Venues (`/api/v1/venues`)
- `POST /venues/` - Create new venue
  - Query params: `created_by` (UUID)
  - Body: `CreateVenueRequest`
  - Response: `VenueResponse`
- `GET /venues/nearby` - Find nearby venues (PostGIS geo-search)
  - Query params: `latitude`, `longitude`, `radius_meters` (default: 5000)
  - Response: `List[VenueResponse]`
- `GET /venues/{venue_id}` - Get venue details
- `PUT /venues/{venue_id}` - Update venue information
  - Body: `UpdateVenueRequest`

### Social (`/api/v1/social`)
- `POST /social/follow/{user_id}` - Follow another player
  - Body: `FollowRequest`
- `DELETE /social/follow/{user_id}/{followee_id}` - Unfollow player
- `GET /social/followers/{user_id}` - Get user's followers
  - Response: `List[UserProfileResponse]`
- `GET /social/following/{user_id}` - Get users being followed
- `GET /social/mutual/{user_id}` - Get mutual followers

### Recommendations (`/api/v1/recommendations`)
- `GET /recommendations/compatibility/{player_id}` - Get compatibility scores
  - Response: `List[CompatibilityScoreResponse]`
- `GET /recommendations/partners/{player_id}` - Get recommended partners
  - Query params: `limit` (default: 5), `min_games` (default: 3)
  - Response: `List[RecommendedPartnerResponse]`
- `GET /recommendations/teams/rankings` - Get top team rankings
  - Query params: `limit` (default: 10)
  - Response: `List[TeamRankingResponse]`

## Pydantic Schemas

### Authentication Schemas (`app/schemas/auth.py`)
```python
class ProfileResponse(BaseModel):
    user_id: UUID
    display_name: Optional[str] = None
    avatar_url: Optional[str] = None
    default_venue: Optional[UUID] = None
    created_at: str
    updated_at: str

class UpdateProfileRequest(BaseModel):
    display_name: Optional[str] = None
    avatar_url: Optional[str] = None
    default_venue: Optional[UUID] = None
```

### Match Schemas (`app/schemas/matches.py`)
```python
class ScoreData(BaseModel):
    team1: int
    team2: int

class CreateMatchRequest(BaseModel):
    venue_id: UUID
    team1_players: List[UUID]  # Must have exactly 2 players
    team2_players: List[UUID]  # Must have exactly 2 players
    scores: List[ScoreData]
    played_at: datetime

class PlayerRatingResponse(BaseModel):
    user_id: UUID
    display_name: str
    mu: float
    sigma: float
    games_played: int
```

### Venue Schemas (`app/schemas/venues.py`)
```python
class CreateVenueRequest(BaseModel):
    name: str
    latitude: float
    longitude: float
    address: Optional[str] = None

class VenueResponse(BaseModel):
    id: UUID
    name: str
    latitude: float
    longitude: float
    address: str
    created_by: UUID
    created_at: str
    updated_at: str
```

### Social Schemas (`app/schemas/social.py`)
```python
class FollowRequest(BaseModel):
    followee_id: UUID

class UserProfileResponse(BaseModel):
    user_id: UUID
    display_name: str
    avatar_url: str | None = None
```

### Recommendation Schemas (`app/schemas/recommendations.py`)
```python
class CompatibilityScoreResponse(BaseModel):
    partner: PartnerProfileResponse
    team_rating: float
    avg_individual_rating: float
    compatibility_score: float

class RecommendedPartnerResponse(BaseModel):
    partner: PartnerProfileResponse
    team_rating: float
    avg_individual_rating: float
    compatibility_score: float
    games_played_together: int
```

## Key Design Decisions

### 1. Compatibility Score Calculation
- **On-the-fly calculation**: Computed in real-time, not stored in database
- **Formula**: `team_mu - ((player1_mu + player2_mu) / 2)`
- **Benefits**: Always up-to-date, no refresh needed, more efficient than materialized views

### 2. Match Creation Flow
1. Validate 2 players per team
2. Convert scores to internal format
3. Determine winners based on score comparison
4. Create match record with all players
5. **Synchronously** update TrueSkill ratings in same transaction
6. Update individual player ratings
7. Update/create team ratings

### 3. Error Handling Strategy
```python
try:
    result = await core_function()
    return ResponseModel(**result)
except Exception as e:
    raise HTTPException(status_code=500, detail=str(e))
```

### 4. Query Parameter Patterns
- **Pagination**: `limit` parameter (default values specified)
- **Filtering**: Type-specific filters (e.g., `min_games` for recommendations)
- **Geo-queries**: `latitude`, `longitude`, `radius_meters` for venue search
- **User Context**: `created_by` for operations requiring user identification

### 5. Response Patterns
- **Single Resource**: Return specific model (e.g., `VenueResponse`)
- **Collections**: Return `List[Model]` with proper typing
- **Flexible Data**: Use `List[Any]` for complex nested structures from database

## Development Guidelines

### Adding New Endpoints
1. **Create Pydantic schemas** in `app/schemas/{module}.py`
2. **Implement business logic** in `app/core/{module}.py`
3. **Create API routes** in `app/api/{module}.py`
4. **Register router** in `app/main.py`
5. **Test endpoints** using `/api/v1/docs`

### Schema Design Best Practices
- Use `Optional` for nullable fields
- Provide sensible defaults for query parameters
- Use `UUID` type for all ID fields
- Use `datetime` for timestamp fields
- Keep request/response models separate

### Route Design Best Practices
- Use proper HTTP methods (GET, POST, PUT, DELETE)
- Include resource IDs in URL path
- Use query parameters for filtering/pagination
- Return appropriate HTTP status codes
- Include comprehensive docstrings

### Error Handling
- Always wrap core function calls in try/except
- Use `HTTPException` for API errors
- Provide meaningful error messages
- Log errors for debugging (when logging is implemented)

## Testing Strategy

### Manual Testing
- Use Swagger UI at `/api/v1/docs` for interactive testing
- Test edge cases (invalid UUIDs, missing data, etc.)
- Verify response schemas match documentation

### Automated Testing (Future)
- Unit tests for each core function
- Integration tests for API endpoints
- Schema validation tests
- Error handling tests

## Security Considerations

### Current Implementation
- No authentication middleware (to be added)
- Basic input validation via Pydantic
- UUID validation for resource IDs

### Future Security Enhancements
- JWT authentication middleware
- Role-based access control
- Rate limiting
- Input sanitization
- CORS configuration refinement

## Performance Considerations

### Current Optimizations
- Async database operations
- On-the-fly compatibility calculations
- Efficient PostGIS geo-queries

### Future Optimizations
- Response caching for leaderboards
- Database query optimization
- Connection pooling
- Background task processing for rating updates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anyistudio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
