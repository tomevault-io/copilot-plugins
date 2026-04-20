## le-moissoneurv2-latest

> Always remember this

You should read through the code base and apply the code to all parts of the projects that needs updates after you have changed some part. 
Find the backend .env file here: 


# Server Configuration
PORT=4000
NODE_ENV=development
API_URL=http://localhost:4000

# Supabase Configuration
SUPABASE_URL=https://wnunkggxzzlvyrabmnwi.supabase.co
SUPABASE_SERVICE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6IndudW5rZ2d4enpsdnlyYWJtbndpIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc0NzMxMzk3MiwiZXhwIjoyMDYyODg5OTcyfQ.WSB62Ij0YXacOvCZTiki-mMJ9mOGXAC6PQLYvsHqiT8
SUPABASE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6IndudW5rZ2d4enpsdnlyYWJtbndpIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDczMTM5NzIsImV4cCI6MjA2Mjg4OTk3Mn0.uUX27mGzJ_R16X8p8zVBdRzS9zo1dp4eoqgH_anGsBM

# JWT Configuration
JWT_SECRET=c22669fba3350718b42101984add3c57cc89ac4b1b08126c3a3e50108f5ff466dfa39b1c24d389980cf2f881cc70d21d1751c266b87b2777723185c46fc632fa
JWT_EXPIRATION=24h

# Memory Management
NODE_OPTIONS="--max-old-space-size=6144"
PM2_MEMORY_LIMIT=7168

# Browser Launch Settings
BROWSER_LAUNCH_TIMEOUT=180000
NAVIGATION_TIMEOUT=60000

# Display Configuration
SCREEN_WIDTH=1920
SCREEN_HEIGHT=1080

# Resource Management
CONCURRENT_SCRAPES=2

# Session Configuration
SESSION_SECRET=8c79d6a2e9c3b4f5a1d8e7f2b5c9a6d3
SESSION_TIMEOUT=1800000
SESSION_WARNING=60000

# Security Settings
MAX_LOGIN_ATTEMPTS=5
LOCKOUT_DURATION=900000

# Scraper Configuration
MAX_PAGES_PER_SCRAPE=50

# CORS Settings
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:8080

# Performance Monitoring
MEMORY_CHECK_INTERVAL=300000
MAX_MEMORY_USAGE_PERCENT=85

# Backup Configuration
BACKUP_SCHEDULE="0 2 * * *"
BACKUP_RETENTION_DAYS=7
# Redis Configuration
REDIS_ENABLED=true
REDIS_HOST=172.27.95.109
REDIS_PORT=6379
REDIS_PASSWORD=Keya1996

# Logging
LOG_LEVEL=info


#Sign up for Redis Cloud (they have a free tier):
#Go to https://redis.com/try-free/
#Create an account
#Create a database
#Update your production .env:
# Redis Configuration
# REDIS_ENABLED=true
# REDIS_HOST=your-redis-host.cloud.redislabs.com
# REDIS_PORT=15925
# REDIS_PASSWORD=your-password

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SalomonFrux) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
