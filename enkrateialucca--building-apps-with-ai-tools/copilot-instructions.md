## security-api-integration

> Security guidelines and API integration best practices


# Security and API Integration Guidelines

This project emphasizes secure API integration and follows security best practices for web applications.

## Security Principles

### API Key Management
- **NEVER** expose API keys in frontend code
- Use environment variables for sensitive data
- Implement backend proxies for API calls
- Use HTTPS for all API communications

### Data Protection
- Validate all user inputs
- Sanitize data before processing
- Use proper error handling without exposing sensitive information
- Implement rate limiting for API calls

## API Integration Patterns

### Backend Proxy Pattern
For apps requiring API keys, use a backend proxy:

```javascript
// Frontend (safe)
const response = await fetch('/api/weather', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ location: userLocation })
});

// Backend proxy handles API key
```

### Environment Variables
Use environment variables for configuration:

```javascript
// server.js
const API_KEY = process.env.WEATHER_API_KEY;
const PORT = process.env.PORT || 3000;
```

## Implementation Examples

### Geo-Spatial Air Quality App
Reference [apps/geo-spatial-app-air-quality/server.js](mdc:apps/geo-spatial-app-air-quality/server.js) for:
- Backend proxy implementation
- Environment variable handling
- Secure API key management
- Error handling and validation

### Secure API Integration Tutorial
See [tutorials/secure-api-integration.md](mdc:tutorials/secure-api-integration.md) for:
- Step-by-step security implementation
- Backend proxy setup
- Environment configuration
- Best practices and common pitfalls

## Security Checklist

### Frontend Security
- [ ] No API keys in client-side code
- [ ] Input validation and sanitization
- [ ] Proper error handling
- [ ] HTTPS enforcement
- [ ] Content Security Policy headers

### Backend Security
- [ ] Environment variable usage
- [ ] API key validation
- [ ] Rate limiting implementation
- [ ] CORS configuration
- [ ] Input validation middleware

### General Security
- [ ] Regular dependency updates
- [ ] Security headers implementation
- [ ] Error logging without sensitive data
- [ ] User input validation
- [ ] Secure session management

## Common Security Mistakes to Avoid

1. **Exposing API Keys**: Never put API keys in frontend code
2. **Insecure HTTP**: Always use HTTPS for production
3. **Missing Input Validation**: Validate all user inputs
4. **Error Information Leakage**: Don't expose sensitive data in error messages
5. **Insecure Dependencies**: Keep dependencies updated

## Tools and Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Web Security Best Practices](https://developer.mozilla.org/en-US/docs/Web/Security)
- [Node.js Security Best Practices](https://nodejs.org/en/docs/guides/security/)
- [Environment Variables Guide](https://12factor.net/config)

## Example Secure Implementation

```javascript
// Secure API call with error handling
async function fetchWeatherData(location) {
  try {
    const response = await fetch('/api/weather', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ location: location.trim() })
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Weather data fetch failed:', error);
    // Handle error gracefully without exposing sensitive info
    return { error: 'Unable to fetch weather data' };
  }
}
```

---
> Source: [EnkrateiaLucca/building-apps-with-ai-tools](https://github.com/EnkrateiaLucca/building-apps-with-ai-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
