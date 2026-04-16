## qa-app

> "description": "Project Structure Rules",

{
    "version": 1,
    "rules": [
        {
            "description": "Project Structure Rules",
            "patterns": [
                "app/**/*",
                "components/**/*",
                "lib/**/*",
                "types/**/*"
            ],
            "requirements": [
                "All components must be TypeScript files",
                "Maintain proper directory structure",
                "Follow Next.js 14 conventions"
            ]
        },
        {
            "description": "Code Quality Rules",
            "patterns": [
                "**/*.{ts,tsx}"
            ],
            "requirements": [
                "Use TypeScript strict mode",
                "Proper error handling required",
                "Implement proper type definitions",
                "Follow React hooks best practices",
                "Implement proper loading states"
            ]
        },
        {
            "description": "API Implementation Rules",
            "patterns": [
                "app/api/**/*"
            ],
            "requirements": [
                "Implement proper error handling",
                "Use proper HTTP status codes",
                "Validate request inputs",
                "Implement rate limiting",
                "Secure routes with authentication"
            ]
        },
        {
            "description": "Authentication Rules",
            "patterns": [
                "app/api/auth/**/*",
                "lib/auth.ts"
            ],
            "requirements": [
                "Implement proper session management",
                "Use secure authentication methods",
                "Implement role-based access control",
                "Handle authentication errors gracefully"
            ]
        },
        {
            "description": "Socket Implementation Rules",
            "patterns": [
                "lib/socket.ts",
                "app/api/socket/**/*"
            ],
            "requirements": [
                "Implement proper connection handling",
                "Handle reconnection scenarios",
                "Implement proper room management",
                "Handle socket authentication"
            ]
        },
        {
            "description": "Database Rules",
            "patterns": [
                "lib/mongodb.ts",
                "prisma/**/*"
            ],
            "requirements": [
                "Use proper connection pooling",
                "Implement proper error handling",
                "Use transactions where necessary",
                "Implement proper indexing"
            ]
        },
        {
            "description": "Component Rules",
            "patterns": [
                "components/**/*"
            ],
            "requirements": [
                "Implement proper prop types",
                "Use proper state management",
                "Implement proper loading states",
                "Follow React best practices"
            ]
        },
        {
            "description": "Voice Processing Rules",
            "patterns": [
                "app/api/voice-models/**/*",
                "app/api/voice-over/**/*"
            ],
            "requirements": [
                "Implement proper error handling",
                "Handle large file uploads",
                "Implement proper progress tracking",
                "Handle audio processing states"
            ]
        },
        {
            "description": "Admin Features Rules",
            "patterns": [
                "components/AdminView.tsx",
                "app/api/admin/**/*"
            ],
            "requirements": [
                "Implement proper role checks",
                "Handle bulk operations properly",
                "Implement proper audit logging",
                "Handle state updates properly"
            ]
        },
        {
            "description": "Testing Rules",
            "patterns": [
                "**/*.test.{ts,tsx}",
                "**/*.spec.{ts,tsx}"
            ],
            "requirements": [
                "Write comprehensive unit tests",
                "Implement integration tests",
                "Mock external dependencies",
                "Test error scenarios"
            ]
        }
    ]
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Developer-GoQuestMedia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
