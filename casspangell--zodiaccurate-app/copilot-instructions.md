## zodiaccurate-app

> This is a comprehensive astrological application that generates personalized daily horoscopes using:

# Zodiaccurate App - Cursor Rules

## Project Overview
This is a comprehensive astrological application that generates personalized daily horoscopes using:
- **Google Apps Script** for backend automation and data processing
- **Firebase Functions** (TypeScript) for webhook handling and API endpoints
- **Frontend** HTML/CSS/JavaScript for user forms and interfaces
- **Integrations**: ChatGPT API, Stripe payments, Google Sheets, Firebase Realtime Database
- **Deployment**: clasp for Apps Script, Firebase CLI for functions

## Technology Stack
- **Backend**: Google Apps Script (JavaScript), Firebase Functions (TypeScript/Node.js 22)
- **Frontend**: Vanilla HTML/CSS/JavaScript
- **Database**: Firebase Realtime Database, Google Sheets
- **APIs**: OpenAI ChatGPT, Stripe, Google APIs
- **Deployment**: clasp, Firebase CLI, GitHub Actions

## Code Style & Standards

### JavaScript (Apps Script)
- Use ES6+ features where supported by Apps Script V8 runtime
- Prefer `const` and `let` over `var`
- Use async/await for asynchronous operations
- Follow Google Apps Script naming conventions (PascalCase for functions, camelCase for variables)
- Add JSDoc comments for complex functions
- Use meaningful variable names, especially for UUIDs and user data

### TypeScript (Firebase Functions)
- Use strict TypeScript configuration
- Define proper interfaces for API requests/responses
- Use proper error handling with try-catch blocks
- Follow Node.js best practices for Firebase Functions
- Use proper CORS configuration for web endpoints

### HTML/CSS/JavaScript (Frontend)
- Use semantic HTML5 elements
- Follow BEM methodology for CSS class naming
- Use modern CSS features (flexbox, grid, custom properties)
- Implement proper form validation
- Ensure responsive design for mobile devices
- Use meaningful IDs and class names

## File Organization

### Apps Script Files (`/AppScript/`)
- `Zodiaccurate.js` - Main orchestration logic
- `ChatGPTUtilities.js` - ChatGPT API integration
- `Firebase.js` - Firebase database operations
- `Email.js` - Email sending functionality
- `Utilities.js` - Helper functions
- `Timezone.js` - Timezone handling
- `SaveUserToSheets.js` - Google Sheets integration

### Firebase Functions (`/functions/src/`)
- `index.ts` - Main functions file with webhook handlers
- Use proper TypeScript interfaces for all data structures
- Implement proper error handling and logging

### Frontend (`/about-you/`)
- `index.html` - Main form interface
- `styles.css` - Styling
- `script.js` - Form handling and validation
- `validate.js` - Form validation logic

## Key Patterns & Conventions

### Data Handling
- Always validate user input before processing
- Use UUIDs for user identification consistently
- Handle timezone conversions properly using the Timezone.js utilities
- Implement proper error handling for API calls

### API Integration
- Use proper authentication for all external APIs
- Implement retry logic for failed requests
- Log API responses for debugging
- Handle rate limiting appropriately

### Firebase Operations
- Use proper error handling for database operations
- Implement proper data validation before writing to Firebase
- Use transactions for critical operations
- Follow Firebase security rules

### Apps Script Specific
- Use `Logger.log()` for debugging instead of console.log
- Handle Google API quotas and rate limits
- Use proper error handling for Google API calls
- Follow Apps Script best practices for performance

## Security & Privacy
- Never commit API keys or sensitive credentials
- Use Google Secret Manager for sensitive data
- Implement proper CORS policies
- Follow GDPR compliance for user data
- Validate all user inputs
- Use HTTPS for all external communications

## Testing & Debugging
- Test Apps Script functions in the Google Apps Script editor
- Use Firebase emulators for local function testing
- Test form validation thoroughly
- Verify timezone handling across different regions
- Test payment webhook handling with Stripe test events

## Deployment Guidelines
- Use `clasp push` for Apps Script deployment
- Use `firebase deploy --only functions` for Firebase Functions
- Test deployments in staging before production
- Use GitHub Actions for automated deployment
- Verify all integrations after deployment

## Common Tasks & Solutions

### Adding New Features
1. Plan the data flow from frontend → Apps Script → Firebase
2. Update relevant utility functions
3. Test with sample data
4. Update documentation

### Debugging Issues
1. Check Apps Script execution logs
2. Verify Firebase function logs
3. Test API endpoints individually
4. Check network requests in browser dev tools

### Performance Optimization
- Minimize API calls in Apps Script
- Use batch operations for Firebase writes
- Implement proper caching where appropriate
- Optimize database queries

## File Naming Conventions
- Use PascalCase for Apps Script files (e.g., `ChatGPTUtilities.js`)
- Use kebab-case for frontend files (e.g., `form-submit.js`)
- Use camelCase for TypeScript files (e.g., `index.ts`)
- Use descriptive names that indicate the file's purpose

## Error Handling
- Always wrap API calls in try-catch blocks
- Log errors with sufficient context
- Provide meaningful error messages to users
- Implement fallback mechanisms where possible

## Documentation
- Update README.md when adding new features
- Document API endpoints and their parameters
- Keep deployment instructions current
- Document any environment-specific configurations

## Git Workflow
- Use descriptive commit messages
- Test changes before committing
- Use feature branches for major changes
- Keep the main branch stable and deployable

## Integration Points
- **ChatGPT API**: Used for generating personalized horoscopes
- **Stripe**: Payment processing and subscription management
- **Google Sheets**: User data storage and management
- **Firebase**: Real-time database for user data and webhooks
- **Google Apps Script**: Core business logic and automation

Remember: This is a production application handling user data and payments. Always prioritize security, data privacy, and system reliability in all changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/casspangell) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
