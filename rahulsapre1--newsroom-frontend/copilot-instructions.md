## newsroom-frontend

> - A nextjs applicatoin for Newsaroo app

# What is this project
- A nextjs applicatoin for Newsaroo app
- The app gets user's name, mobile and topic of interests and saves them in supabse
- The app can also call an API to get the relevant news topics.

# Tech Stack
- Framework: Next.js with App Router
- UI Components: Shadcn UI
- Data Fetching: SWR
- Authentication: Supabase Auth
- Database: Supabase
- State Management: React Context + SWR + Local State
- Functional Backend: We also have an external Rest API backend for specific functional API calls

# API to Register User
curl --location '//api/v1/users' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--data '{
  "mobile_number": "9876543210",
  "topics_of_interest": [
    "technology",
    "sports",
    "politics"
  ]
}'

# API to get new summary for a user
curl --location '//api/v1/users/5898245888/summaries' \
--header 'Accept: application/json'

# API to update user topic interest
curl --location --request PUT '//api/v1/users/in sint ipsum/topics' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--data '{
  "topics_of_interest": [
    "technology",
    "sports",
    "politics"
  ]
}'

# API to summarise news
curl --location '//api/v1/news/summarize' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--data '{
  "topic": "artificial intelligence",
  "time_period": "1d",
  "max_articles": 3
}'

# Directory Structure

newsaroo/
├── .env                  # Environment variables (secrets, API keys)
├── .git/                 # Git version control data
├── .gitignore            # Git ignore rules
├── .newsaroo_env/        # Python virtual environment
├── logs/                 # Log files directory
├── newsaroo-frontend/    # Next.js frontend app (React + Supabase)
│   ├── .cursor/              # Editor/project metadata
│   ├── .gitignore            # Ignore rules for frontend
│   ├── .next/                # Next.js build output
│   ├── app/                  # Main app pages (Next.js routing)
│   ├── components/           # Reusable React components
│   ├── components.json       # Component registry/config
│   ├── eslint.config.mjs     # Linting configuration
│   ├── hooks/                # Custom React hooks
│   ├── lib/                  # Utility libraries
│   ├── next-env.d.ts         # TypeScript Next.js env types
│   ├── next.config.ts        # Next.js configuration
│   ├── node_modules/         # Node.js dependencies
│   ├── package-lock.json     # NPM lockfile
│   ├── package.json          # NPM package manifest
│   ├── postcss.config.mjs    # PostCSS config
│   ├── public/               # Static assets (images, etc.)
│   ├── README.md             # Frontend documentation
│   ├── tsconfig.json         # TypeScript config
│   └── styles/               # CSS/SCSS styles
├── newsaroo.log           # Backend log file
├── notebooks/             # Jupyter or research notebooks
├── README.md              # Project documentation
├── REQUIREMENTS.md        # Requirements/setup instructions
├── requirements.txt       # Python dependencies
├── run.py                 # Backend entry point (runs Uvicorn server)
├── sql_queries.txt        # SQL queries or notes
├── src/                   # Backend source code
│   ├── __init__.py            # Marks src as a Python package
│   ├── api/                   # API route definitions
│   │   ├── __pycache__/           # Python bytecode cache
│   │   ├── models.py              # Pydantic models (schemas)
│   │   ├── news_routes.py         # News-related API routes
│   │   ├── routes.py              # Main API router
│   │   └── user_routes.py         # User-related API routes
│   ├── cli.py                 # Command-line interface
│   ├── config.py              # Configuration and env loading
│   ├── db/                    # Database integration
│   │   ├── __pycache__/           # Python bytecode cache
│   │   ├── migrations/             # Database migration scripts
│   │   │   ├── 000_create_tables.sql   # Create tables SQL
│   │   │   └── 001_setup_rls.sql      # Row-level security SQL
│   │   └── supabase_client.py         # Supabase DB client
│   ├── main.py                # FastAPI app definition
│   ├── news/                  # News processing modules
│   │   ├── __init__.py            # Marks news as a package
│   │   ├── content.py             # Article content processing
│   │   ├── search.py              # News search logic
│   │   └── summary.py             # Summarization logic
│   └── utils/                 # Utility functions
│       ├── __init__.py            # Marks utils as a package
│       └── display.py             # Display/formatting utilities
├── tests/                  # Backend tests
│   ├── __init__.py             # Marks tests as a package
│   ├── test_api_endpoint.py    # API endpoint tests
│   ├── test_content_fetch.py   # Content fetching tests
│   ├── test_summaries.py       # Summarization tests
│   └── test_user.py            # User-related tests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rahulsapre1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rahulsapre1)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
