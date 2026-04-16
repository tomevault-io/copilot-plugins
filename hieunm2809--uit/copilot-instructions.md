## uit

> StudyMate AI is an intelligent learning platform for UIT students, combining AI-powered assistance with comprehensive course management. The project emphasizes gamification, progress tracking, and personalized learning experiences similar to Duolingo.

# StudyMate AI - Cursor Rules
# Inspired by Duolingo's gamification and learning experience principles

## Project Overview
StudyMate AI is an intelligent learning platform for UIT students, combining AI-powered assistance with comprehensive course management. The project emphasizes gamification, progress tracking, and personalized learning experiences similar to Duolingo.

## Core Principles (Duolingo-Inspired)

### 1. Gamification & Engagement
- **Streak System**: Track daily learning streaks to encourage consistent study habits
- **Achievements & Badges**: Reward users for milestones (courses completed, perfect scores, daily goals)
- **Progress Visualization**: Clear progress bars, completion percentages, and visual feedback
- **Points & XP**: Award experience points for completing lessons, quizzes, and activities
- **Leaderboards**: Optional competitive elements for motivation (respecting privacy)

### 2. Learning Experience Design
- **Bite-sized Lessons**: Break content into small, digestible chunks (5-15 minutes)
- **Spaced Repetition**: Implement review cycles to reinforce learning
- **Adaptive Difficulty**: Adjust content difficulty based on user performance
- **Immediate Feedback**: Provide instant feedback on answers and progress
- **Practice Mode**: Allow users to review and practice previous content

### 3. User Interface & UX
- **Clean, Minimal Design**: Focus on content, reduce cognitive load
- **Mobile-First**: Optimize for mobile devices (primary learning platform)
- **Accessibility**: WCAG 2.1 AA compliance, keyboard navigation, screen reader support
- **Fast Loading**: Optimize assets, lazy loading, efficient database queries
- **Offline Capability**: Cache content for offline learning when possible

### 4. AI Integration Best Practices
- **Contextual Help**: AI assistant provides hints, explanations, not direct answers
- **Personalized Recommendations**: Suggest courses/content based on learning history
- **Learning Path Optimization**: AI suggests optimal learning sequences
- **Question Generation**: AI creates practice questions based on user weaknesses
- **Transparency**: Always indicate when AI is providing assistance

## Code Standards

### Backend (Node.js/Express) - MVC Pattern
```javascript
// Routes: Define endpoints and delegate to controllers
const controller = require('../controllers/controllerName');
router.get('/path', controller.methodName);

// Controllers: Handle business logic
const { applicationLogger } = require('../config/logger');

exports.methodName = async (req, res) => {
  try {
    // Use async/await for all database operations
    // Always handle errors with try-catch
    // Validate input data before processing
    // Use Sequelize models with proper associations
    // Implement proper error logging to Kibana
    const data = await Model.findAll();
    res.render('view', { data });
  } catch (error) {
    applicationLogger.error('Error in methodName', error, {
      type: 'controller',
      operation: 'methodName',
      userId: req.user?.id,
      path: req.path
    });
    req.flash('error', 'User-friendly error message');
    res.redirect('/fallback');
  }
};
```

### Frontend (EJS/Tailwind)
```html
<!-- Use semantic HTML5 elements -->
<!-- Implement responsive design with Tailwind utilities -->
<!-- Add loading states for async operations -->
<!-- Provide clear error messages -->
<!-- Use progressive enhancement -->
```

### Database (PostgreSQL/Sequelize)
- Use UUIDs for primary keys
- Implement soft deletes where appropriate
- Add indexes for frequently queried fields
- Use transactions for multi-step operations
- Normalize data structure but optimize for read performance

### API Design
- RESTful endpoints with clear naming
- Consistent response format: `{ success: boolean, message: string, data: object }`
- Proper HTTP status codes
- Rate limiting for public endpoints
- Authentication via JWT or session

## File Structure Conventions

```
/models          - Sequelize models with associations
/routes          - Express route definitions (thin layer, delegates to controllers)
/controllers     - Business logic and request handling (MVC pattern)
  /admin         - Admin-specific controllers
    - adminCourseController.js
    - adminCategoryController.js
    - adminContactController.js
    - adminUserController.js
    - adminStatisticsController.js
  - courseController.js
  - authController.js
  - dashboardController.js
  - profileController.js
  - chatController.js
  - contentController.js
  - fileController.js
  - userController.js
  - statisticsController.js
  - infoController.js
  - aiController.js
/middleware      - Authentication, validation, error handling
/services        - External services (AI, email, etc.)
/views           - EJS templates
  /pages         - Full page templates
  /partials      - Reusable components
/public          - Static assets
  /css           - Custom styles
  /js            - Client-side scripts
  /images        - Images and assets
/config          - Configuration files
/utils           - Helper functions
```

### MVC Architecture

**Routes** (`/routes`):
- Define URL patterns and HTTP methods
- Apply validation middleware (express-validator)
- Delegate to controller methods
- Keep routes thin - no business logic

**Controllers** (`/controllers`):
- Handle request/response logic
- Process validation results
- Call models/services for data operations
- Render views or return JSON
- Handle errors and flash messages

**Models** (`/models`):
- Define data structure and relationships
- Include business logic methods (e.g., `validatePassword`, `incrementCourseCount`)
- Handle database operations

**Example Route Structure:**
```javascript
// routes/courses.js
const courseController = require('../controllers/courseController');
router.get('/', courseController.index);
router.get('/:slug', courseController.show);
```

**Example Controller Structure:**
```javascript
// controllers/courseController.js
exports.index = async (req, res) => {
  try {
    // Business logic here
    const courses = await Course.findAll();
    res.render('pages/courses/index', { courses });
  } catch (error) {
    // Error handling
  }
};
```

## Naming Conventions

### Variables & Functions
- Use camelCase: `getUserProgress`, `courseId`
- Boolean variables: `isActive`, `hasCompleted`, `canEnroll`
- Async functions: Always use `async/await`, name clearly: `fetchUserData()`

### Database
- Tables: plural, snake_case: `courses`, `user_achievements`
- Columns: snake_case: `created_at`, `progress_percentage`
- Foreign keys: `{model}_id`: `course_id`, `user_id`

### Routes
- RESTful: `/api/courses/:id`, `/api/users/:id/progress`
- Actions: `/courses/:id/enroll`, `/content/:id/complete`

## Error Handling

```javascript
// Always provide user-friendly error messages in Vietnamese
// Log detailed errors server-side
// Never expose sensitive information in error responses
// Use consistent error format across the application
```

## Logging Best Practices

### Use Kibana/Elasticsearch for All Logs
- **NEVER use `console.log`, `console.error`, or `console.warn`** in production code
- **ALWAYS use `applicationLogger`** from `config/logger.js` for all logging
- All logs are automatically sent to Elasticsearch/Kibana for centralized monitoring

### Logger Usage
```javascript
const { applicationLogger } = require('../config/logger');

// Info logs - for normal operations
applicationLogger.info('User logged in', {
  type: 'auth',
  operation: 'login',
  userId: user.id,
  email: user.email
});

// Error logs - for errors with stack traces
applicationLogger.error('Failed to process payment', error, {
  type: 'payment',
  operation: 'process',
  userId: req.user.id,
  amount: req.body.amount
});

// Warn logs - for warnings
applicationLogger.warn('Rate limit approaching', {
  type: 'rate_limit',
  userId: req.user.id,
  remaining: rateLimit.remaining
});

// Debug logs - for debugging information
applicationLogger.debug('Cache miss for course', {
  type: 'cache',
  operation: 'get',
  courseId: courseId
});
```

### Log Metadata Structure
Always include structured metadata for easy querying in Kibana:
- `type`: Category of log (auth, socket, redis, api, database, etc.)
- `operation`: Specific operation name (login, send_message, addUser, etc.)
- `userId`: User ID if applicable
- `resource_type`: Type of resource (course, message, user, etc.)
- `resource_id`: ID of resource if applicable
- Additional context-specific fields as needed

### Logging Levels
- **info**: Normal operations, successful actions, important events
- **error**: Errors with stack traces, failed operations
- **warn**: Warnings, rate limits, deprecated features
- **debug**: Debugging information, detailed traces

### Kibana Query Examples
```
# Find all socket connections
type: "socket" AND operation: "connection"

# Find errors for a specific user
type: "socket" AND userId: "user-123" AND level: "error"

# Find all Redis operations
type: "redis"

# Find all chat messages sent
type: "socket" AND operation: "send_message"
```

### Skip Logging for Certain Routes
- Health checks: `/health`
- Static assets: `/static`, `/assets`, `/uploads`
- Well-known paths: `/.well-known`
- Favicon: `/favicon.ico`

## Security Best Practices

- **Input Validation**: Validate and sanitize all user inputs
- **SQL Injection**: Use Sequelize parameterized queries (built-in)
- **XSS Prevention**: Escape user-generated content in templates
- **CSRF Protection**: Use CSRF tokens for state-changing operations
- **Authentication**: Secure session management, password hashing (bcrypt)
- **Authorization**: Role-based access control (student, instructor, admin)
- **Rate Limiting**: Prevent abuse of AI endpoints and API calls
- **Data Privacy**: Follow GDPR principles, allow data export/deletion

## Performance Optimization

- **Database Queries**: Use eager loading, avoid N+1 queries
- **Caching**: Redis for session storage and frequently accessed data
- **Asset Optimization**: Minify CSS/JS, optimize images
- **Lazy Loading**: Load content on demand, paginate large datasets
- **CDN**: Use CDN for static assets in production

## Testing Guidelines

- **Unit Tests**: Test individual functions and models
- **Integration Tests**: Test API endpoints and database operations
- **E2E Tests**: Test critical user flows (enrollment, learning, AI chat)
- **Test Data**: Use factories/fixtures for consistent test data

## Documentation Requirements

- **Code Comments**: Explain complex logic, not obvious code
- **API Documentation**: Document all API endpoints with examples
- **README**: Keep README updated with setup instructions
- **Changelog**: Track significant changes and features

## Duolingo-Specific Features to Consider

### Learning Mechanics
- **Daily Goals**: Set and track daily learning targets
- **Streak Freeze**: Allow users to maintain streaks during breaks
- **Practice Sessions**: Review weak areas identified by AI
- **Story Mode**: Interactive content for language learning (adapt for CS subjects)
- **Duolingo Events**: Community learning events (study groups for UIT)

### Progress Tracking
- **Skill Trees**: Visual representation of course progression
- **Crown Levels**: Multiple levels per skill/course section
- **Checkpoints**: Periodic assessments to unlock new content
- **Legendary Status**: Perfect mastery of a course/skill

### Social Features (Privacy-First)
- **Friends System**: Optional friend connections for motivation
- **Clubs**: Study groups within UIT
- **Achievement Sharing**: Share accomplishments (with privacy controls)

## AI-Specific Guidelines

### OpenAI/Gemini Integration
- **Token Management**: Monitor and optimize token usage
- **Context Windows**: Keep conversation context manageable
- **Error Handling**: Gracefully handle API failures
- **Cost Optimization**: Cache responses, batch requests when possible
- **Rate Limiting**: Respect API rate limits

### AI Assistant Behavior
- **Educational Focus**: AI should teach, not just answer
- **Socratic Method**: Ask guiding questions rather than direct answers
- **Code Examples**: Provide clear, commented code examples
- **Explanations**: Break down complex concepts into simpler parts
- **Encouragement**: Provide positive reinforcement for learning

## Vietnamese Language Support

- All user-facing text in Vietnamese
- Error messages in Vietnamese
- Date/time formatting: Vietnamese locale
- Number formatting: Vietnamese conventions
- Email templates: Vietnamese with English fallback

## Git Workflow

- **Branch Naming**: `feature/`, `fix/`, `refactor/`, `docs/`
- **Commit Messages**: Clear, descriptive, in English
- **Pull Requests**: Include description, screenshots for UI changes
- **Code Review**: Required before merging to main

## Deployment Checklist

- [ ] Environment variables configured
- [ ] Database migrations run
- [ ] Static assets optimized
- [ ] Error logging configured
- [ ] Monitoring set up
- [ ] Backup strategy in place
- [ ] Security headers configured
- [ ] SSL/TLS enabled

## Continuous Improvement

- **User Feedback**: Regularly collect and act on user feedback
- **Analytics**: Track key metrics (engagement, completion rates, AI usage)
- **A/B Testing**: Test new features and UI improvements
- **Performance Monitoring**: Track load times, error rates
- **Security Audits**: Regular security reviews and updates

## Chat System Features

### User-to-User Chat
- **Search Users**: All users can search for other users by name, email, or student ID to start conversations
- **No Admin Restriction**: Chat feature is available to all authenticated users, not just admins
- **Realtime Updates**: Uses polling (setInterval) every 3 seconds to fetch new messages
- **Auto Scroll**: Automatically scrolls to bottom when entering conversation page
- **Message Format**: Messages include sender info, timestamp, and avatar support
- **Conversation Management**: Users can view all their conversations and start new ones

### Chat API Endpoints
- `GET /chat` - Get all conversations (supports JSON response for widgets)
- `GET /chat/search/users?q=query` - Search users for chat
- `GET /chat/:userId` - Get or create conversation with user (supports JSON response)
- `GET /chat/:conversationId/messages` - Get messages for conversation
- `POST /chat/:conversationId/message` - Send a message

### Chat Widget (Optional)
- Floating chat widget can be added to layout
- Shows conversations list and allows messaging
- Search functionality for finding users

## Development Configuration

### Environment Variables
- `DB_SYNC=true/false` - Auto-sync database schema on startup (default: false, set to true for development)
- `AUTO_LOGIN_EMAIL=email@example.com` - Auto login with specified email (development only)
- `MINIO_ENABLED=true/false` - Enable MinIO object storage
- `MINIO_BUCKET_NAME=studymate` - Default bucket name
- `MINIO_ROOT_USER` and `MINIO_ROOT_PASSWORD` - MinIO credentials

### Database Configuration
- PostgreSQL in Docker container
- Connection: `localhost:5432` (from Windows host to Docker)
- Auto-sync controlled by `DB_SYNC` env variable
- Use `DB_SYNC=false` in production to prevent accidental schema changes

### MinIO Storage
- Data stored in `./uploads/minio` directory (bind mount from Docker)
- Bucket auto-created on container startup via init script
- Public download policy set automatically

### Auto Login (Development)
- Set `AUTO_LOGIN_EMAIL` in `.env` to automatically login that account
- Only works in development environment
- Redirects to referrer (previous page) if available, otherwise dashboard

## Notes for AI Assistants

When working on this codebase:
1. **Follow MVC pattern**: Routes delegate to controllers, controllers handle business logic
2. **Always validate inputs** before database operations (use express-validator in routes)
3. **Use transactions** for multi-step operations
4. **Handle errors gracefully** with user-friendly messages in Vietnamese
5. **Keep routes thin**: Routes should only define endpoints and call controllers
6. **Controllers contain logic**: All business logic, database queries, and view rendering in controllers
7. **Consider mobile experience** in all UI changes
8. **Maintain Vietnamese language** for all user-facing content
9. **Follow Duolingo principles** of gamification and engagement
10. **Optimize for learning outcomes** not just feature completion
11. **Test edge cases** especially for enrollment and progress tracking
12. **Consider accessibility** in all UI components
13. **Document complex logic** for future maintainers
14. **Group related controllers**: Use subdirectories for admin controllers (`/controllers/admin/`)
15. **Chat System**: All users can search and chat with each other, no admin restriction
16. **Date Handling**: Always check for null/undefined dates and provide fallback ("Vừa xong" for invalid dates)
17. **Auto Scroll**: Chat pages should auto-scroll to bottom on load
18. **Socket.IO**: Real-time chat uses Socket.IO for instant messaging (not polling)
19. **API Responses**: Support both HTML and JSON responses (check `Accept` header)
20. **Avatar Handling**: Always check for avatar existence and provide fallback with initials
21. **Logging**: NEVER use console.log/error/warn - ALWAYS use applicationLogger from config/logger.js
22. **Log Metadata**: Always include structured metadata (type, operation, userId, etc.) for Kibana queries
23. **Error Logging**: Always log errors with full context and stack traces to Kibana
24. **Skip Logging**: Skip logging for health checks, static assets, and well-known paths

---

**Remember**: StudyMate is about making learning enjoyable, accessible, and effective. Every feature should serve the goal of helping UIT students learn better.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HieuNM2809) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
