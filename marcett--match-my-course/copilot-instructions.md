## match-my-course

> Project Context: MatchMyCourse




Project Context: MatchMyCourse
PROJECT OVERVIEW
MatchMyCourse is a course matching platform that connects students with appropriate educational content based on their needs, preferences, and learning goals.
TECHNICAL STACK
Frontend
* Next.js 15: App Router with TypeScript
* TailwindCSS 3.4.1: Utility-first styling
* Shadcn UI: Component library for consistent UI
* TypeScript: Strict type checking enabled
Backend
* Node.js + Express: RESTful API server
* TypeScript: Full-stack type safety
* MongoDB: NoSQL database for flexible data models
* NextAuth.js: Authentication and session management
Infrastructure
* Deployment: Railway (full-stack deployment)
* Environment: Development/Production environments
* No Testing Framework: Manual testing currently
PROJECT-SPECIFIC PATTERNS
API Architecture
// API Route Pattern (Next.js)
export async function POST(request: Request) {
  try {
    const body = await request.json()
    // Validate with Zod
    const result = await service.create(body)
    return Response.json({ data: result }, { status: 201 })
  } catch (error) {
    return Response.json(
      { error: 'Internal server error' }, 
      { status: 500 }
    )
  }
}

// Express API Pattern
app.post('/api/courses', async (req: Request, res: Response) => {
  try {
    const course = await courseService.create(req.body)
    res.status(201).json({ data: course })
  } catch (error) {
    res.status(500).json({ error: error.message })
  }
})
Database Patterns
// MongoDB Schema Pattern
interface Course {
  _id: ObjectId
  title: string
  description: string
  category: string
  difficulty: 'beginner' | 'intermediate' | 'advanced'
  duration: number // hours
  provider: string
  rating: number
  tags: string[]
  createdAt: Date
  updatedAt: Date
}

interface User {
  _id: ObjectId
  email: string
  name: string
  preferences: {
    subjects: string[]
    difficulty: string
    timeCommitment: number
  }
  matchedCourses: ObjectId[]
  createdAt: Date
}
Authentication Flow
// NextAuth configuration
export const authOptions: NextAuthOptions = {
  providers: [
    // Email/password or OAuth providers
  ],
  session: {
    strategy: 'jwt'
  },
  callbacks: {
    jwt: async ({ token, user }) => {
      if (user) {
        token.id = user.id
      }
      return token
    },
    session: async ({ session, token }) => {
      if (token) {
        session.user.id = token.id
      }
      return session
    }
  }
}
DOMAIN-SPECIFIC LOGIC
Course Matching Algorithm
* User Preferences: Subject interests, skill level, time availability
* Course Attributes: Category, difficulty, duration, rating
* Matching Score: Algorithm to rank courses based on user fit
* Personalization: Learning history and feedback integration
Key Entities
1. Users: Students seeking courses
2. Courses: Educational content with metadata
3. Matches: User-course compatibility scores
4. Reviews: User feedback on courses
5. Progress: User learning journey tracking
FEATURE-SPECIFIC PATTERNS
Course Discovery
interface CourseFilter {
  category?: string
  difficulty?: string
  duration?: { min: number, max: number }
  rating?: number
  tags?: string[]
  provider?: string
}

interface MatchingCriteria {
  userPreferences: UserPreferences
  learningGoals: string[]
  timeConstraints: number
  experienceLevel: string
}
User Dashboard
* Personal Recommendations: ML-based course suggestions
* Progress Tracking: Course completion status
* Learning Path: Structured educational journey
* Saved Courses: Bookmarked content
Search & Filtering
* Advanced Search: Multi-criteria course discovery
* Smart Filters: Dynamic filter options
* Sort Options: By relevance, rating, duration, difficulty
RAILWAY DEPLOYMENT CONSIDERATIONS
Environment Variables
# Database
MONGODB_URI=mongodb://...
DATABASE_NAME=matchmycourse

# Authentication  
NEXTAUTH_SECRET=...
NEXTAUTH_URL=https://matchmycourse.railway.app

# API URLs
NEXT_PUBLIC_API_URL=https://api.matchmycourse.railway.app
EXPRESS_API_URL=http://localhost:3001
Deployment Structure
* Frontend: Next.js app on Railway
* Backend: Express API on Railway
* Database: MongoDB Atlas or Railway MongoDB addon
DEVELOPMENT WORKFLOW
Code Organization Suggestions
app/
├── (dashboard)/
│   ├── courses/
│   ├── matches/
│   └── profile/
├── (auth)/
│   ├── login/
│   └── register/
├── api/
│   ├── courses/
│   ├── users/
│   └── matches/
└── globals.css

components/
├── ui/ (shadcn)
├── course/
├── user/
├── matching/
└── layout/

lib/
├── auth.ts
├── db.ts
├── utils.ts
├── matching-algorithm.ts
└── validations.ts

types/
├── course.ts
├── user.ts
├── match.ts
└── api.ts
Common Use Cases
1. Course Search: Filter and find relevant courses
2. User Onboarding: Capture preferences and setup profile
3. Matching Engine: Generate personalized recommendations
4. Progress Tracking: Monitor learning journey
5. Review System: Collect and display course feedback
CODING PRIORITIES FOR MATCHMYCOURSE
1. User Experience: Intuitive course discovery and matching
2. Performance: Fast search and filtering
3. Data Consistency: Reliable course and user data
4. Scalability: Handle growing course catalog and users
5. Personalization: Improve matching accuracy over time
SPECIFIC IMPLEMENTATION NOTES
* Mobile-First: Students often browse courses on mobile
* Accessibility: Educational platform should be inclusive
* SEO Optimization: Course pages need good search visibility
* Performance: Large course catalogs require efficient querying
* Data Privacy: Handle user learning data responsibly

Context Understanding: Always consider that you're building features for an educational platform where the primary goal is connecting students with the most suitable courses for their learning journey.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MarceTT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
