## unite-hackathon

> Building a DeFi savings application that abstracts crypto complexity while using 1inch API extensively for the hackathon. Target: crypto-native users who don't use DeFi.

# DeFi Savings App - Cursor Rules

## 🎯 Project Overview
Building a DeFi savings application that abstracts crypto complexity while using 1inch API extensively for the hackathon. Target: crypto-native users who don't use DeFi.

## 🏗 Architecture Guidelines

### Tech Stack
- **Frontend**: Next.js 14 with TypeScript
- **Styling**: Tailwind CSS with custom design system
- **Wallet**: RainbowKit + Wagmi
- **APIs**: 1inch API (primary), Base network
- **Animations**: Framer Motion
- **Icons**: Lucide React

### Design Principles
- **Modern & Clean**: Don't look like typical crypto apps
- **User-Friendly**: Abstract away crypto complexity
- **Mobile-First**: Responsive design
- **Educational**: Help users learn while they save

## 📝 Git History Rules

### Commit Structure
- **Prefix commits** with type: `feat:`, `fix:`, `docs:`, `refactor:`, `style:`, `test:`
- **Clear descriptions**: What was changed and why
- **Atomic commits**: One logical change per commit
- **No large commits**: Keep changes focused and reviewable

### Branch Strategy
- `main`: Production-ready code
- `develop`: Integration branch
- `feature/*`: New features
- `fix/*`: Bug fixes

### Commit Examples
```
feat: Add 1inch API integration for token swaps
feat: Implement financial freedom calculator
fix: Resolve TypeScript errors in TokenSwap component
refactor: Extract wallet connection logic to custom hook
docs: Update README with setup instructions
```

## 🧩 Component Guidelines

### File Structure
```
components/
├── ui/           # Reusable UI components
├── forms/        # Form components
├── layout/       # Layout components
└── features/     # Feature-specific components
```

### Component Standards
- **TypeScript**: Strict typing for all components
- **Props Interface**: Define clear prop interfaces
- **Error Handling**: Graceful error states
- **Loading States**: Show loading indicators
- **Accessibility**: ARIA labels, keyboard navigation

### Naming Conventions
- **Components**: PascalCase (`FinancialFreedomCalculator`)
- **Files**: PascalCase for components, camelCase for utilities
- **Variables**: camelCase
- **Constants**: UPPER_SNAKE_CASE
- **Types**: PascalCase with descriptive names

## 🔧 Code Quality Standards

### TypeScript
- **Strict mode**: Enable all strict options
- **No any types**: Use proper typing
- **Interface over type**: Prefer interfaces for objects
- **Generic types**: Use when appropriate

### Error Handling
- **Try-catch blocks**: For async operations
- **User-friendly messages**: Don't expose technical errors
- **Fallback UI**: Graceful degradation
- **Error boundaries**: React error boundaries

### Performance
- **Lazy loading**: For heavy components
- **Memoization**: Use React.memo and useMemo
- **Bundle size**: Keep dependencies minimal
- **Image optimization**: Use Next.js Image component

## 🎨 Styling Guidelines

### Tailwind CSS
- **Custom classes**: Use component classes for consistency
- **Responsive design**: Mobile-first approach
- **Dark mode**: Consider future dark mode support
- **Design tokens**: Use consistent colors and spacing

### Component Classes
```css
.btn-primary: Primary action buttons
.btn-secondary: Secondary action buttons
.card: Standard card containers
.input-field: Form input fields
```

## 🔌 API Integration

### 1inch API Usage
- **Extensive integration**: Use as many endpoints as possible
- **Error handling**: Graceful API error handling
- **Rate limiting**: Respect API limits
- **Caching**: Cache responses when appropriate

### Network Support
- **Primary**: Base mainnet (cost-effective)
- **Secondary**: Ethereum mainnet (if needed)
- **Testnets**: Only for development

## 🧪 Testing Guidelines

### Test Structure
- **Unit tests**: For utility functions
- **Component tests**: For React components
- **Integration tests**: For API interactions
- **E2E tests**: For critical user flows

### Testing Standards
- **Coverage**: Aim for 80%+ coverage
- **Mocking**: Mock external dependencies
- **User scenarios**: Test real user flows
- **Error cases**: Test error conditions

## 📚 Documentation

### Code Documentation
- **JSDoc**: For complex functions
- **README**: Comprehensive setup guide
- **API docs**: Document custom hooks and utilities
- **Component docs**: Document component props

### User Documentation
- **Onboarding**: Clear user guides
- **Educational content**: Built into the app
- **Help system**: Contextual help

## 🚀 Deployment

### Environment Variables
- **Required**: 1inch API key
- **Optional**: WalletConnect project ID
- **Security**: Never commit API keys

### Build Process
- **Type checking**: Run before build
- **Linting**: ESLint with strict rules
- **Optimization**: Next.js optimizations
- **Bundle analysis**: Monitor bundle size

## 🔒 Security

### Best Practices
- **API keys**: Environment variables only
- **Input validation**: Validate all user inputs
- **XSS prevention**: Sanitize user content
- **CORS**: Proper CORS configuration

### Wallet Security
- **Connection**: Secure wallet connections
- **Transaction signing**: Clear transaction details
- **Error handling**: Secure error messages

## 🎯 Feature Priorities

### MVP (Current)
1. ✅ Wallet connection
2. ✅ Financial freedom calculator
3. ✅ 1inch token swaps
4. ✅ Basic dashboard

### Next Phase
1. Transaction execution
2. Portfolio tracking
3. Educational content
4. Automated deposits

### Future Features
1. Cross-chain functionality
2. Yield strategies
3. Social features
4. Advanced analytics

## 📋 Development Workflow

### Daily Tasks
1. **Morning**: Review git history, plan day
2. **Development**: Follow coding standards
3. **Testing**: Test features thoroughly
4. **Evening**: Commit with clear messages

### Code Review
- **Self-review**: Review your own code
- **Peer review**: Have others review when possible
- **Documentation**: Update docs with changes
- **Testing**: Ensure tests pass

## 🎨 UI/UX Standards

### Color Palette
- **Primary**: Blue (#0ea5e9)
- **Secondary**: Purple (#8b5cf6)
- **Success**: Green (#22c55e)
- **Warning**: Orange (#f59e0b)
- **Error**: Red (#ef4444)

### Typography
- **Font**: Inter (Google Fonts)
- **Headings**: Bold, clear hierarchy
- **Body**: Readable, 16px minimum
- **Code**: JetBrains Mono

### Spacing
- **Consistent**: Use Tailwind spacing scale
- **Responsive**: Adapt to screen sizes
- **Whitespace**: Don't overcrowd interfaces

## 🔄 State Management

### Local State
- **useState**: For component state
- **useReducer**: For complex state logic
- **Context**: For global state when needed

### Data Fetching
- **SWR/React Query**: For API data
- **Caching**: Implement proper caching
- **Loading states**: Show loading indicators
- **Error states**: Handle errors gracefully

## 📱 Mobile Considerations

### Responsive Design
- **Mobile-first**: Design for mobile first
- **Touch targets**: Minimum 44px touch targets
- **Gestures**: Support touch gestures
- **Performance**: Optimize for mobile performance

### Progressive Enhancement
- **Core functionality**: Works without JavaScript
- **Enhanced experience**: Better with JavaScript
- **Fallbacks**: Graceful degradation

## 🎯 Success Metrics

### Technical Metrics
- **Performance**: < 3s load time
- **Accessibility**: WCAG 2.1 AA compliance
- **Coverage**: > 80% test coverage
- **Bundle size**: < 500KB initial load

### User Metrics
- **Engagement**: Time spent in app
- **Completion**: Calculator usage
- **Retention**: Return user rate
- **Satisfaction**: User feedback

## 🚨 Emergency Procedures

### Critical Issues
1. **Security vulnerability**: Immediate fix and deployment
2. **Data loss**: Backup and recovery procedures
3. **Service outage**: Rollback to previous version
4. **User complaints**: Quick response and resolution

### Communication
- **Team updates**: Regular status updates
- **User communication**: Clear, helpful messages
- **Documentation**: Update docs with fixes
- **Post-mortem**: Learn from incidents

---

**Remember**: This is a hackathon project focused on demonstrating extensive 1inch API usage while building a user-friendly DeFi savings application. Keep the code clean, documented, and maintainable for the judges to review. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jennyg0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
