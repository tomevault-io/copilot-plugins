## laravel-review-aggregator

> Services contain the core business logic and follow the patterns established in:


# Service Layer Patterns

## Service Architecture
Services contain the core business logic and follow the patterns established in:
- [ReviewService.php](mdc:app/Services/ReviewService.php) - Core review operations
- [AppStoreService.php](mdc:app/Services/AppStoreService.php) - External API integration
- [PlayStoreService.php](mdc:app/Services/PlayStoreService.php) - External API integration

## Key Principles

### Single Responsibility
- Each service focuses on one domain/entity
- Separate integration services from business logic services
- Keep methods focused and cohesive

### Dependency Injection
- Inject required dependencies through constructor
- Use interfaces for external services when possible
- Type-hint all dependencies

### Data Transfer
- Return DTOs instead of raw arrays
- Use consistent data structures
- Transform data at service boundaries

### Error Handling
- Throw domain-specific exceptions
- Log important business events
- Handle external API failures gracefully

## Service Method Patterns

### Business Logic Services
```php
class ReviewService
{
    public function __construct(
        private ReviewRepository $reviewRepository,
        private AppStoreService $appStoreService,
        private PlayStoreService $playStoreService
    ) {}

    public function aggregateReviews(string $appId): ReviewCollectionDTO
    {
        $appStoreReviews = $this->appStoreService->fetchReviews($appId);
        $playStoreReviews = $this->playStoreService->fetchReviews($appId);
        
        $aggregated = $this->combineReviews($appStoreReviews, $playStoreReviews);
        
        return new ReviewCollectionDTO($aggregated);
    }
    
    private function combineReviews(array $appStoreReviews, array $playStoreReviews): array
    {
        // Business logic for combining reviews
    }
}
```

### External Integration Services
```php
class AppStoreService
{
    public function fetchReviews(string $appId): array
    {
        try {
            $response = Http::get("https://itunes.apple.com/lookup", [
                'id' => $appId
            ]);
            
            return $this->transformAppStoreData($response->json());
        } catch (Exception $e) {
            Log::error('App Store API failed', ['app_id' => $appId, 'error' => $e->getMessage()]);
            throw new ExternalServiceException('Failed to fetch App Store reviews');
        }
    }
}
```

## Testing Considerations
- Mock external service dependencies
- Test business logic separately from integrations
- Use factories for test data creation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hellotreedev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
