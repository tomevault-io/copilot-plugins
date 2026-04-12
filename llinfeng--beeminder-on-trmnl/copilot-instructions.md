## beeminder-on-trmnl

> - TRMNL provides a terminal-based interface for displaying and interacting with data

# TRMNL API Rules for Beeminder Plugin

## Core TRMNL Concepts

### Terminal Interface
- TRMNL provides a terminal-based interface for displaying and interacting with data
- Use clear, readable text layouts optimized for terminal display
- Follow terminal UI best practices: monospace fonts, clear spacing, consistent formatting

### Plugin Architecture
- Plugins are JavaScript/TypeScript modules that extend TRMNL functionality
- Each plugin should have a clear purpose and well-defined API
- Follow the plugin lifecycle: initialization, data fetching, rendering, cleanup

## TRMNL API Guidelines

### Data Display Patterns
- Use tables for structured data (goals, datapoints)
- Use progress bars for visual representation of goal progress
- Use color coding for status indicators (on track, off track, urgent)
- Keep line lengths reasonable for terminal display (80-120 characters)

### User Interaction
- Provide clear navigation between different views
- Use keyboard shortcuts for common actions
- Show helpful error messages and loading states
- Implement search/filter functionality where appropriate

### Real-time Updates
- Poll Beeminder API at reasonable intervals (30-60 seconds)
- Show last updated timestamp
- Handle network errors gracefully
- Provide manual refresh option

## Beeminder Integration Rules

### Goal Display
- Show goal name, current value, target, and deadline
- Display progress as percentage and visual indicator
- Highlight urgent goals (close to deadline or off track)
- Show goal type (do more, do less, custom)

### Data Management
- Cache API responses to reduce rate limiting
- Implement proper error handling for API failures
- Validate data before display
- Provide data export functionality

### Authentication
- Store auth tokens securely
- Handle token expiration gracefully
- Provide clear setup instructions for users
- Support multiple user accounts if needed

## Code Structure Guidelines

### File Organization
- Separate concerns: API calls, data processing, UI rendering
- Use TypeScript for type safety
- Implement proper error boundaries
- Follow consistent naming conventions

### API Integration
- Create reusable API client for Beeminder
- Implement retry logic for failed requests
- Use proper HTTP status code handling
- Log API interactions for debugging

### UI Components
- Create reusable terminal UI components
- Implement responsive layouts for different terminal sizes
- Use consistent styling and formatting
- Provide accessibility features where possible

## Performance Considerations

### Caching Strategy
- Cache goal data with appropriate TTL
- Implement incremental updates
- Use efficient data structures
- Minimize API calls through smart caching

### Memory Management
- Clean up resources properly
- Avoid memory leaks in long-running processes
- Use efficient data serialization
- Implement proper garbage collection

## Error Handling

### Network Errors
- Handle connection timeouts
- Implement exponential backoff for retries
- Show user-friendly error messages
- Provide offline mode when possible

### Data Validation
- Validate API responses
- Handle malformed data gracefully
- Provide data recovery options
- Log validation errors for debugging

## User Experience

### Onboarding
- Provide clear setup instructions
- Validate configuration on startup
- Show helpful examples and tips
- Implement guided tour for new users

### Accessibility
- Use high contrast colors for visibility
- Provide keyboard navigation
- Support screen readers where possible
- Include help text and tooltips

## Security Best Practices

### Token Management
- Never log auth tokens
- Use secure storage methods
- Implement token rotation if needed
- Validate token permissions

### Data Privacy
- Minimize data collection
- Secure local storage
- Implement proper data deletion
- Follow privacy best practices

## Testing Guidelines

### Unit Testing
- Test API integration functions
- Mock external dependencies
- Test error handling scenarios
- Validate data processing logic

### Integration Testing
- Test end-to-end workflows
- Validate API rate limiting
- Test authentication flows
- Verify error recovery

## Documentation Standards

### Code Documentation
- Document all public APIs
- Include usage examples
- Explain complex algorithms
- Maintain up-to-date README

### User Documentation
- Provide clear installation instructions
- Include configuration examples
- Document all features and options
- Maintain troubleshooting guide
alwaysApply: true
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/llinfeng)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/llinfeng)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
