## smart-office-demo

> This is a **Smart Office Dashboard** application built for conference room utilization tracking and analytics. The app demonstrates real-time IoT sensor data processing for enterprise tradeshows, showcasing the capabilities of modern web technologies in a professional law firm environment.

# Agent Instructions for Smart Office Dashboard

## Project Overview
This is a **Smart Office Dashboard** application built for conference room utilization tracking and analytics. The app demonstrates real-time IoT sensor data processing for enterprise tradeshows, showcasing the capabilities of modern web technologies in a professional law firm environment.

**Company**: Dewey, Cheatham & Howe (Fictitious Law Firm)
**Purpose**: Enterprise booth demo for tradeshows

## Tech Stack
- **Framework**: Next.js 14 with App Router
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **UI Components**: shadcn/ui
- **Database**: Supabase
- **Charts**: Recharts
- **Icons**: Lucide React
- **External APIs**: Unsplash, OpenAI

## Application Architecture

### Core Pages
1. **Login Page** (`/login`)
   - Beautiful branding with law firm logo
   - Secure authentication via Supabase Auth
   
2. **Main Dashboard** (`/rooms`)
   - Real-time room occupancy display
   - Temperature monitoring
   - Historical occupancy charts (24-hour view, 30-minute segments)
   - Live sensor data integration
   
3. **Analytics Page** (`/analytics`)
   - Historical data analysis
   - Trend visualization
   - Usage patterns and insights
   
4. **Raw Data Stream** (`/raw-data`)
   - Live sensor data feed
   - Real-time updates for demo purposes
   - Technical monitoring interface

### Key Features
- **Real-time Updates**: Live sensor data streaming
- **Professional Design**: Enterprise-grade UI/UX
- **Responsive Layout**: Mobile and desktop optimized
- **Data Visualization**: Interactive charts and graphs
- **Security**: Proper authentication and data protection

## Development Guidelines

### Code Quality Standards
- Write elegant, well-refactored, and beautifully organized code
- Follow Next.js 14 and React best practices
- Implement proper TypeScript typing
- Use shadcn/ui components consistently
- Maintain clean file structure and naming conventions
- Ensure proper error handling and loading states

### File Organization
```
src/
├── app/                    # Next.js App Router pages
├── components/
│   ├── ui/                # shadcn/ui components
│   ├── dashboard/         # Dashboard-specific components
│   ├── auth/              # Authentication components
│   └── common/            # Shared components
├── lib/
│   ├── supabase/          # Supabase configuration
│   ├── utils/             # Utility functions
│   └── types/             # TypeScript types
└── hooks/                 # Custom React hooks
```

### Component Guidelines
- Keep components focused and single-purpose
- Use proper prop typing with TypeScript
- Implement proper loading and error states
- Follow shadcn/ui patterns for consistency
- Add proper accessibility attributes
- Document complex logic with comments

### Data Flow
- Use Supabase for real-time data subscriptions
- Implement proper state management
- Handle offline scenarios gracefully
- Optimize for performance with proper caching

## Security Considerations
- Never expose API keys or sensitive configuration
- Implement proper authentication flows
- Validate all user inputs
- Follow security best practices for data handling
- Use environment variables for all secrets

## Demo Requirements
- Ensure smooth real-time updates for live demonstrations
- Implement mock data generation for reliable demos
- Provide clear visual feedback for all user interactions
- Optimize for presentation on large screens
- Include impressive visual effects appropriate for enterprise demos

## Deployment & Environment
- Configure for Vercel deployment
- Set up proper environment variable management
- Ensure production-ready performance
- Implement proper monitoring and error tracking

## Testing Strategy
- Write unit tests for critical business logic
- Implement integration tests for key user flows
- Test real-time features thoroughly
- Validate responsive design across devices
- Performance test with realistic data loads

Remember: This application represents the pinnacle of modern web development practices and will be scrutinized by enterprise clients. Every aspect must demonstrate professional excellence and technical sophistication.

---
> Source: [supabase/smart-office-demo](https://github.com/supabase/smart-office-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
