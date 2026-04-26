## smartq

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

```bash
# Development
npm run dev              # Start development server with Turbopack (default port 3000)
npm run build           # Build for production
npm run start           # Start production server
npm run lint            # Run ESLint for code quality

# Firebase deployment
firebase deploy --only database    # Deploy database rules only
firebase deploy                   # Deploy all Firebase config
firebase login                    # Login to Firebase CLI (required once)

# Database rules testing
firebase deploy --only database:rules    # Deploy only security rules

# Git workflow (for commits to https://github.com/reallygood83/smartq)
git add .
git commit -m "descriptive message"
git push origin main
```

## Architecture Overview

**SmartQ** is an AI-powered educational platform built with Next.js 15 and Firebase, designed to help teachers collect and analyze student questions using Google Gemini API.

### Core Architecture Patterns

**Education Level Adaptation System**
- All UI/UX adapts to 5 education levels: Elementary, Middle, High, University, Adult
- `EducationLevelContext` provides level-specific terminology, themes, and AI prompts
- Components use `useSmartTerminology()` for adaptive language
- Theme system in `/src/styles/themes.ts` provides level-specific visual styling

**Dual Authentication Model**
- Teachers: Firebase Authentication (Google OAuth)
- Students: Anonymous access via 6-digit session codes
- Security rules enforce teacher-only data modification

**Client-Side API Key Management**
- User-provided Gemini API keys stored encrypted in localStorage
- Never stored on server - encryption/decryption in `/src/lib/encryption.ts`
- API keys are user-specific and tied to Firebase auth UID

**Real-time Data Architecture**
- Firebase Realtime Database for live updates
- Session-based data structure: `/sessions/{sessionId}/`
- Questions, shared content, and AI analyses linked to sessions
- Students write to `/questions/{sessionId}/`, teachers control everything else

### Key Data Structures

**Sessions**: Core teaching units with unique 6-digit access codes
```typescript
{
  sessionId: string,
  title: string,
  accessCode: string,  // 6-digit student access code
  sessionType: 'DEBATE' | 'INQUIRY' | 'PROBLEM' | 'CREATIVE' | 'DISCUSSION' | 'QNA',
  teacherId: string,
  subjects?: Subject[],
  isAdultEducation?: boolean,
  
  // Teacher-led mode support (NEW)
  interactionMode?: 'free_question' | 'teacher_led',
  activeTeacherQuestionId?: string
}
```

**Questions**: Student submissions linked to sessions with like functionality
```typescript
{
  questionId: string,
  sessionId: string,
  text: string,
  studentId: string,  // Anonymous persistent ID
  isAnonymous: boolean,
  studentName?: string,
  likes?: {
    [studentId: string]: boolean  // Like tracking per student
  }
}
```

**Teacher Questions**: Teacher-prepared and real-time questions (NEW)
```typescript
{
  questionId: string,
  sessionId: string,
  text: string,
  teacherId: string,
  order: number,
  source: 'prepared' | 'realtime',
  status: 'waiting' | 'active' | 'completed',
  createdAt: number,
  activatedAt?: number,
  completedAt?: number
}
```

**Student Responses**: Responses to teacher questions (NEW)
```typescript
{
  responseId: string,
  questionId: string,  // Links to TeacherQuestion
  sessionId: string,
  studentId: string,
  text: string,
  createdAt: number,
  isAnonymous: boolean,
  studentName?: string
}
```

**AI Analysis Results**: Gemini API analysis of collected questions
```typescript
{
  clusteredQuestions: Array<{
    clusterId: number,
    clusterTitle: string,
    questions: string[],
    combinationGuide: string
  }>,
  recommendedActivities: Activity[],
  conceptDefinitions: Concept[]
}
```

### Critical Implementation Details

**Theme and Dark Mode System**
- Dual context system: `ThemeContext` for light/dark, `EducationLevelContext` for adaptive theming
- Theme system in `/src/styles/themes.ts` provides level-specific colors and styling
- Card components use Tailwind `dark:` classes for backgrounds
- **CRITICAL**: Always use `dark:text-white` for maximum contrast in dark mode - avoid `dark:text-gray-*`
- Use `useFullTheme()` hook to access complete theme object in components

**Firebase Security Rules**
- Teachers can only modify their own sessions
- Students can submit questions anonymously to any session
- Anonymous users can read sessions and submit questions/likes
- Questions have anonymous write access for student participation
- All advanced features (content sharing, analysis) require teacher authentication
- **NEW**: Teacher questions (`teacherQuestions/`) - teacher write-only, students read-only
- **NEW**: Student responses (`studentResponses/`) - students write, teachers read
- **NEW**: Question analyses (`questionAnalyses/`) - teacher write-only

**API Integration Pattern**
- `/src/app/api/ai/` contains Next.js API routes that proxy to Gemini
- Client sends encrypted API key in request headers
- Server-side API routes decrypt keys for Gemini API calls
- Never persist API keys on server side

**Navigation and Routing**
- Teachers: `/teacher/dashboard` → `/teacher/session/create` → `/teacher/session/{sessionId}`
- Students: Home page session code input → `/student/session/{sessionCode}`
- Education level adaptation affects all page content and navigation terminology
- **NEW**: Teacher session pages conditionally show teacher-led vs free-question mode interfaces

## Teacher-Led Q&A Mode Architecture (NEW)

### Overview
Teacher-led mode provides structured Q&A sessions where teachers control question flow and collect student responses for analysis.

### Key Components

**TeacherQuestionManager** (`/src/components/teacher/TeacherQuestionManager.tsx`)
- Real-time question management with Firebase `onValue` listeners
- Question status tracking (waiting → active → completed)
- Instant question deployment and queue management
- Integration with question templates and participation monitoring

**QuestionTemplates** (`/src/components/teacher/QuestionTemplates.tsx`)
- Educational template library based on Bloom's Taxonomy
- 8 categories: Understanding, Critical Thinking, Creative Thinking, Inquiry, etc.
- Smart filtering by session type, subjects, and education level
- Cognitive level tagging (remember, understand, apply, analyze, evaluate, create)

**TeacherQuestionView** (`/src/components/student/TeacherQuestionView.tsx`)
- Student interface for responding to active teacher questions
- Real-time question synchronization
- Anonymous response submission with persistent student IDs

**ParticipationMonitor** (`/src/components/teacher/ParticipationMonitor.tsx`)
- Real-time participation rate calculation
- Response length distribution analysis
- Connected student estimation
- Visual participation status indicators

**StudentResponseAnalysisDashboard** (`/src/components/teacher/StudentResponseAnalysisDashboard.tsx`)
- **Dual Analysis Modes**: Comprehensive (default) and Individual analysis
- **Comprehensive Analysis**: Fast class-wide insights, token-efficient, teacher-focused recommendations
- **Individual Analysis**: Detailed per-student feedback and assessment
- Education level-adapted analysis prompts
- Analysis history and mode-specific result management

### Data Flow Architecture

```
Teacher Creates Question → Firebase teacherQuestions/
                      ↓
Teacher Activates Question → Session activeTeacherQuestionId updated
                      ↓
Students See Active Question → Real-time onValue listener
                      ↓
Students Submit Responses → Firebase studentResponses/
                      ↓
Teacher Views Responses → Real-time participation monitoring
                      ↓
Teacher Requests AI Analysis → Gemini API analysis
                      ↓
Analysis Results Stored → Firebase questionAnalyses/
```

### Zero-Impact Implementation
- All teacher-led features are additive - existing free-question mode unchanged
- Conditional rendering based on `session.interactionMode`
- Backward compatibility with existing sessions (defaults to 'free_question')
- No breaking changes to existing data structures

## Development Patterns

**When working with Teacher-Led Mode:**
- Check session `interactionMode` before rendering teacher-led components
- Use conditional rendering: `{session?.interactionMode === 'teacher_led' && <Component />}`
- All teacher-led data uses separate Firebase paths to avoid conflicts
- Maintain backward compatibility - always provide fallbacks for missing interactionMode
- Question templates filter automatically based on session context

**When modifying UI components:**
- Always check if component uses education level adaptation via `useSmartTerminology()`
- Add dark mode support using Tailwind `dark:` classes - use `dark:text-white` for text
- Override theme system colors with Tailwind classes when needed for dark mode
- Test across different education levels using the level selector
- Student components support anonymous access - no authentication checks required

**When working with Firebase:**
- Security rules are in `database.rules.json` - deploy with `firebase deploy --only database`
- Use `onValue` listeners for real-time updates, not one-time `get()` calls
- Student operations (questions, likes) work anonymously - no auth checks needed
- Teacher operations require authentication state verification
- Questions support real-time like functionality with student-specific tracking
- **NEW**: Teacher-led mode uses separate data paths: `teacherQuestions/`, `studentResponses/`, `questionAnalyses/`, `comprehensiveAnalyses/`
- **NEW**: Always use batch updates when activating questions to ensure consistency
- **NEW**: Comprehensive analysis results stored separately from individual analysis for better organization
- **CRITICAL**: Always use `getStoredApiKey(user.uid)` with user ID parameter - never call without user ID

**When adding AI features:**
- API routes go in `/src/app/api/ai/`
- Use the established encryption pattern for API key handling
- Follow the education level prompt enhancement pattern in `/src/lib/aiPrompts.ts`
- **NEW**: Student response analysis uses education level-adapted prompts
- **NEW**: Analysis results stored in Firebase for history and export features
- **NEW**: Dual analysis modes - comprehensive (default, token-efficient) vs individual (detailed)
- **NEW**: Comprehensive analysis focuses on class-wide patterns and teaching recommendations

### Environment Setup Requirements

Required `.env.local` variables:
```
NEXT_PUBLIC_FIREBASE_API_KEY=your_firebase_api_key
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
NEXT_PUBLIC_FIREBASE_PROJECT_ID=your_project_id
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your_project.appspot.com
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
NEXT_PUBLIC_FIREBASE_APP_ID=your_app_id
NEXT_PUBLIC_FIREBASE_DATABASE_URL=https://your_project-default-rtdb.firebaseio.com/
```

**Important**: The app gracefully degrades when Firebase config is missing, but authentication and real-time features will be disabled. Always verify Firebase connection before making database-related changes.

### Mobile and Dark Mode Considerations

**Mobile Optimization:**
- All components are mobile-first responsive using Tailwind CSS
- Student session pages optimized for tablet/mobile usage
- Touch-friendly UI elements and proper spacing

**Dark Mode Implementation:**
- Use `dark:text-white` for primary text in dark mode (not gray variants)
- Override theme system colors when they provide insufficient contrast
- Test all text elements on dark backgrounds for readability
- Collapsible sections and interactive elements maintain contrast in both modes

### Student Experience Features

**Real-time Interactions:**
- Questions appear instantly for all session participants
- Like functionality allows students to interact with each other's questions
- Anonymous student IDs persist across browser sessions
- Popular questions (3+ likes) get visual highlighting
- **NEW**: Student responses default to real name (not anonymous) for better teacher tracking

**Educational Content:**
- Shared materials support text, links, YouTube videos, and instructions
- Materials section is collapsible to reduce initial page load
- Educational level adaptation affects all content presentation

## Recent Major Updates

### Student Response Analysis Dual Mode System (Latest)

**Comprehensive Analysis Mode (Default)**
- Fast, token-efficient analysis focused on class-wide patterns
- Response type distribution (correct understanding, partial understanding, misconception, creative approach, off-topic)
- Comprehension level distribution with visual indicators
- Key insights: common understandings, difficulties, misconception patterns, creative ideas
- Classroom recommendations: immediate actions, concepts to clarify, suggested activities, exemplary responses
- Overall assessment: class understanding level, engagement, readiness for next topic

**Individual Analysis Mode (Optional)**
- Detailed per-student analysis and feedback
- Individual comprehension scoring and level assessment
- Student-specific strengths, improvement areas, and next steps
- More comprehensive but token-intensive

**Student Default Behavior Changes**
- Student responses now default to real name entry (not anonymous)
- Anonymous option available as checkbox for privacy when needed
- Improves teacher ability to track individual student progress

### Teacher-Led Mode Implementation

### New Components Added

**Teacher Components:**
- `TeacherQuestionManager.tsx` - Central question management interface
- `QuestionTemplates.tsx` - Educational template library with Bloom's Taxonomy
- `ParticipationMonitor.tsx` - Real-time participation tracking
- `StudentResponseAnalysisDashboard.tsx` - AI-powered response analysis

**Student Components:**
- `TeacherQuestionView.tsx` - Interface for responding to teacher questions

**API Routes:**
- `/api/teacher-questions/create` - Create teacher questions
- `/api/teacher-questions/activate` - Activate questions for students
- `/api/student-responses/submit` - Submit student responses
- `/api/ai/analyze-student-responses` - Individual AI analysis of responses
- `/api/ai/analyze-comprehensive` - Comprehensive/collective analysis of responses
- `/api/ai/analyze-questions` - General question analysis
- `/api/ai/analyze-adult-session` - Adult education session analysis
- `/api/ai/instructor-analysis` - Instructor-focused analysis
- `/api/ai/learner-analysis` - Learner-focused analysis
- `/api/ai/quality-monitoring` - Quality monitoring analysis
- `/api/ai/validate-key` - Gemini API key validation

**Type Definitions:**
- `/src/types/teacher-led.ts` - Complete type system for teacher-led mode
- `/src/types/education.ts` - Session mode configurations

### Firebase Data Structure Extensions

```
firebase-realtime-database/
├── sessions/{sessionId}/
│   ├── interactionMode: 'free_question' | 'teacher_led'
│   └── activeTeacherQuestionId?: string
├── teacherQuestions/{sessionId}/
│   └── {questionId}/
│       ├── text: string
│       ├── status: 'waiting' | 'active' | 'completed'
│       └── order: number
├── studentResponses/{sessionId}/
│   └── {responseId}/
│       ├── questionId: string
│       ├── studentId: string
│       └── text: string
├── questionAnalyses/{sessionId}/
│   └── {questionId}/
│       ├── individualAnalyses: ResponseAnalysis[]
│       └── collectiveAnalysis: CollectiveAnalysis
└── comprehensiveAnalyses/{sessionId}/
    └── {analysisId}/
        ├── responseTypeDistribution: object
        ├── comprehensionLevelDistribution: object
        ├── keyInsights: object
        ├── classroomRecommendations: object
        └── overallAssessment: object
```

### Educational Design Principles Implemented

**Bloom's Taxonomy Integration:**
- Question templates categorized by cognitive levels
- Clear progression from basic recall to creative synthesis
- Education level adaptation for age-appropriate language

**Assessment Best Practices:**
- Real-time formative assessment capabilities
- Participation monitoring without over-surveillance
- AI analysis focused on learning insights, not grading

**Pedagogical Flexibility:**
- Teacher maintains full control over question flow
- Support for both planned and spontaneous questioning
- Easy transition between modes within same platform

### Development Philosophy

**Zero-Impact Architecture:**
- All new features are completely additive
- Existing free-question mode remains unchanged
- Backward compatibility guaranteed for all existing sessions

**Educational Effectiveness:**
- Avoided over-guidance that could inhibit learning
- Focused on teacher empowerment rather than student constraint
- Maintained authentic assessment opportunities

**Technical Reliability:**
- Simple, robust real-time synchronization
- Minimal complexity to reduce error potential
- Clear separation of concerns between modes

## Common Development Scenarios

### Adding New Features
1. Check if feature affects existing functionality
2. Use conditional rendering based on session mode when needed
3. Add proper TypeScript types in relevant type files
4. Update Firebase security rules if new data paths are needed
5. Test both free-question and teacher-led modes
6. Update CLAUDE.md if introducing new patterns

### Debugging Common Issues
- **Port conflicts**: Dev server defaults to 3000, uses 3001 if busy
- **Missing interactionMode**: Always provide fallback `(session.interactionMode || 'free_question')`
- **Firebase connection**: Check `.env.local` variables and network connectivity
- **Dark mode contrast**: Always use `dark:text-white` for readable text
- **Mobile responsiveness**: Test on mobile devices, especially student interfaces
- **JSX syntax errors**: Use `/* comment */` instead of `{/* comment */}` inside conditional rendering expressions
- **API key errors**: Ensure `getStoredApiKey(user.uid)` is called with user ID parameter
- **Build failures**: Run `npm run build` locally to catch TypeScript and JSX errors before deployment

### Performance Optimization
- Use `onValue` listeners efficiently (clean up in useEffect cleanup)
- Batch Firebase updates when modifying multiple fields
- Avoid unnecessary re-renders with proper dependency arrays
- Use React.memo for expensive components when appropriate

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/reallygood83) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
