## zensor-ui

> Zensor Portal UI is a React-based IoT device management interface for the Zensor system. It provides multi-tenant device management with real-time WebSocket data streaming and specialized irrigation control features.

# Zensor Portal UI - Project Intelligence

## Project Overview
Zensor Portal UI is a React-based IoT device management interface for the Zensor system. It provides multi-tenant device management with real-time WebSocket data streaming and specialized irrigation control features.

## Key Technical Patterns

### Component Architecture
- Use functional components with React hooks
- Prefer composition over inheritance
- Keep components focused and single-purpose
- Use custom hooks for reusable logic (e.g., useWebSocket)

### State Management
- Use React useState for local component state
- Avoid prop drilling - consider context for shared state
- Prefer derived state over storing computed values
- Handle loading, error, and success states consistently

### Styling Approach
- Use CSS variables for theming (defined in :root)
- Follow mobile-first responsive design
- Use CSS Grid and Flexbox for layouts
- Keep styles co-located with components when possible
- Use consistent spacing (8px grid system)

### API Integration
- Use environment variables for API configuration
- Handle errors gracefully with user-friendly messages
- Show loading states during API calls
- Implement retry logic for failed requests
- Use proper HTTP status code handling

## WebSocket Patterns

### Connection Management
- Use custom useWebSocket hook for connection management
- Implement automatic reconnection with exponential backoff
- Show connection status to users
- Handle connection errors gracefully

### Message Handling
- Parse JSON messages safely with error handling
- Update UI state based on message content
- Handle malformed messages gracefully
- Maintain state consistency during reconnections

### Relay Control (Specialized)
- Display relay values as "On" (green) or "Off" (red)
- Disable irrigation controls when relay is active
- Wait for first WebSocket message before enabling controls
- Monitor relay state changes for irrigation completion

## Scheduled Tasks Patterns

### API Integration
- Use `/v1/tenants/{tenant_id}/devices/{device_id}/scheduled-tasks` endpoints
- Implement full CRUD operations (GET, POST, PUT, DELETE)
- Handle pagination for task listing
- Use proper error handling for all operations

### Task Management
- Support cron expressions for scheduling (6 fields: second minute hour day month dayOfWeek)
- Convert duration from minutes (UI) to seconds (API)
- Implement active/inactive toggle for task status
- Provide human-readable display of cron expressions

### UI/UX Patterns
- Use form-based creation/editing with validation
- Provide preset examples for common schedules (Daily 6 AM, 12 PM, 6 PM)
- Show task status with visual indicators (active/inactive)
- Implement confirmation dialogs for destructive actions
- Use responsive design for mobile compatibility

### Command Structure
- Use irrigation commands with index, value, priority, and wait_for fields
- Structure: Two commands per irrigation task:
  - Activate: `{ index: 1, value: 1, priority: "NORMAL", wait_for: "0s" }`
  - Deactivate: `{ index: 1, value: 0, priority: "NORMAL", wait_for: "5m" }`
- Support multiple commands per scheduled task
- Handle command validation and error states

## UI/UX Patterns

### Navigation
- Use React Router for client-side routing
- Implement breadcrumb navigation for complex flows
- Provide clear visual feedback for current location
- Use consistent navigation patterns throughout

### Data Display
- Use card-based layouts for data presentation
- Show loading states during data fetching
- Display empty states when no data is available
- Use consistent typography and spacing

### Error Handling
- Show user-friendly error messages
- Provide retry options when appropriate
- Use consistent error state styling
- Log errors for debugging

### Responsive Design
- Design mobile-first
- Use CSS Grid for responsive layouts
- Ensure touch-friendly controls on mobile
- Test on various screen sizes

## Code Quality Standards

### JavaScript/React
- Use modern ES6+ features
- Prefer const over let, avoid var
- Use destructuring for props and state
- Implement proper error boundaries
- Use meaningful variable and function names

### CSS
- Use CSS variables for consistent theming
- Follow BEM-like naming conventions
- Keep selectors specific and avoid deep nesting
- Use semantic class names
- Optimize for performance

### File Organization
- Group related components together
- Use descriptive file names
- Keep components focused and single-purpose
- Separate concerns (UI, logic, styling)

### Documentation Organization
- **MANDATORY**: All documentation files must be placed in `./docs` folder
- Create comprehensive docs/README.md with navigation and categorization
- Organize documentation by type (Configuration, Development, API, Integration)
- Use consistent Markdown formatting and cross-referencing
- Never create documentation files in the root directory

## API Integration Patterns

### Endpoints
- `/v1/tenants` - Tenant management
- `/v1/tenants/{id}/devices` - Device listing
- `/v1/devices/{id}` - Device operations
- `/v1/devices/{id}/tasks` - Task management
- `/v1/tenants/{tenant_id}/devices/{device_id}/scheduled-tasks` - Scheduled task management
- `/ws/device-messages` - Real-time data

### Configuration
- Use VITE_API_BASE_URL environment variable
- Derive WebSocket URL from API base URL
- Handle different environments (dev/prod)
- Use proper error handling for API calls

### Notification Requirements
- **MANDATORY**: After each API call, show a notification based on server response
- Use `useNotification` hook for all API feedback
- Show success notifications for successful operations (2xx status codes)
- Show error notifications for failed operations (4xx/5xx status codes)
- Show warning notifications for partial success or important warnings
- Include descriptive titles and messages in notifications
- Use appropriate notification durations (4-6 seconds for errors, 3-4 seconds for success)
- Never use browser alerts - always use the notification system
- For API operations, use `showApiNotification` helper when possible

## Performance Considerations

### Bundle Optimization
- Use Vite for fast builds
- Implement code splitting where appropriate
- Optimize images and assets
- Monitor bundle size

### Runtime Performance
- Minimize unnecessary re-renders
- Use React.memo for expensive components
- Optimize WebSocket message processing
- Implement proper cleanup in useEffect

### Mobile Performance
- Optimize for mobile devices
- Use efficient CSS animations
- Minimize JavaScript execution time
- Test on various mobile devices

## Security Patterns

### Content Security Policy
- Configure CSP headers in nginx
- Allow WebSocket connections (wss:)
- Prevent XSS attacks
- Use secure WebSocket protocol in production

### Data Validation
- Validate user inputs
- Sanitize data before display
- Handle API errors safely
- Implement proper error boundaries

## Development Workflow

### Environment Setup
- Use Node.js 18+
- Install dependencies with npm
- Configure environment variables
- Use Vite dev server for development

### Testing
- Test on multiple browsers
- Verify responsive design
- Test WebSocket functionality
- Validate API integration

### Deployment
- Build with `npm run build`
- Deploy static files to nginx
- Configure reverse proxy for API
- Set up proper environment variables

## Common Patterns

### Loading States
```jsx
const [loading, setLoading] = useState(true);
const [error, setError] = useState(null);
const [data, setData] = useState(null);

// Use in JSX
{loading && <LoadingSpinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}
```

### WebSocket Integration
```jsx
const { isConnected, lastMessage, error } = useWebSocket(wsUrl);

// Handle messages
useEffect(() => {
  if (lastMessage && lastMessage.type === 'device_message') {
    // Update state based on message
  }
}, [lastMessage]);
```

### Scheduled Tasks API
```jsx
// Create scheduled task
const taskData = {
  commands: [{
    index: 1,
    value: 100,
    priority: "NORMAL",
    wait_for: "0s"
  }],
  schedule: '0 0 6 * * *', // Daily 6 AM
  is_active: true
};

await scheduledTasksApi.createScheduledTask(tenantId, deviceId, taskData);
```

### Error Handling
```jsx
try {
  const response = await fetch(url);
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }
  const data = await response.json();
} catch (error) {
  setError(error.message);
}
```

### API Calls with Notifications
```jsx
// Import notification hook
import { useNotification } from '../hooks/useNotification';

// In component
const { showSuccess, showError, showApiNotification } = useNotification();

// Method 1: Manual notification handling
const handleApiCall = async () => {
  try {
    const response = await fetch('/api/endpoint');
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    const data = await response.json();
    
    showSuccess('Operation completed successfully', 'Success', { duration: 4000 });
    return data;
  } catch (error) {
    showError(`Operation failed: ${error.message}`, 'Error', { duration: 6000 });
    throw error;
  }
};

// Method 2: Using showApiNotification helper
const handleApiCallWithHelper = async () => {
  const result = await showApiNotification(
    fetch('/api/endpoint').then(res => {
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return res.json();
    }),
    'Operation completed successfully',
    'Operation failed',
    'Success',
    'Error'
  );
  
  if (result.success) {
    // Handle success
  }
};
```

## Project-Specific Notes

### Relay Control Logic
- Relay value 1 = "On" (green)
- Relay value 0 = "Off" (red)
- Disable irrigation when relay is active
- Wait for WebSocket confirmation before enabling controls

### Scheduled Irrigation Features
- Support cron expressions for flexible scheduling
- Provide preset examples for common schedules
- Convert duration from minutes to seconds for API
- Implement active/inactive toggle for task management
- Show human-readable schedule descriptions

### Full Width Layout
- Use 100% width for all components
- Remove max-width constraints
- Ensure proper content readability
- Test on various screen sizes

### Real-time Updates
- Use WebSocket for live data
- Show connection status
- Handle reconnection gracefully
- Update UI immediately on data changes

## Future Considerations

### Authentication
- Plan for user authentication
- Consider role-based access control
- Implement session management
- Secure API endpoints

### Advanced Features
- Data visualization with charts
- Advanced filtering and search
- Bulk operations
- Export functionality
- Weather-based scheduling
- Schedule conflict detection

### Performance Optimization
- Implement caching strategies
- Add performance monitoring
- Optimize for large device fleets
- Consider virtual scrolling for large lists 

---
> Source: [zensor-iot/zensor-ui](https://github.com/zensor-iot/zensor-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
