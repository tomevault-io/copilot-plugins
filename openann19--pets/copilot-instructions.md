## pets

> Step-by-step debugging guide for common issues


# Debugging Guide

## Quick Diagnostic Commands

```bash
# Check what's running
ps aux | grep -E "(node.*server|mongod|pnpm.*dev)" | grep -v grep

# Check ports
lsof -i :3000  # Frontend
lsof -i :5001  # Backend
lsof -i :27017 # MongoDB

# Kill stuck processes
pkill -9 -f "node.*server"
pkill -9 -f "nodemon"
pkill -9 -f "pnpm.*dev"

# Check MongoDB
mongosh --eval "db.adminCommand('ping')"

# Check backend logs
tail -50 server/logs/combined.log
```

## Common Error Patterns

### 1. Login Fails / API Errors

**Error**: `net::ERR_FAILED` or `CORS policy blocked`

**Debug steps**:
```bash
# 1. Check backend is running
lsof -i :5001

# 2. Test health endpoint
curl http://localhost:5001/api/health

# 3. Check frontend .env.local
cat apps/web/.env.local | grep API_URL

# 4. Check CORS in browser console
# Should see: Access-Control-Allow-Origin header
```

**Solution**:
- Backend not running → Start backend
- Wrong port in .env.local → Update to 5001
- CORS error → Check CLIENT_URL in server/.env

### 2. Infinite Page Refresh

**Symptoms**: Page keeps reloading/re-rendering

**Common causes**:
1. Hook with side effects (useEffect without proper deps)
2. WebSocket connection retrying
3. Auth redirect loop

**Debug**:
```typescript
// Add console logs to hooks
useEffect(() => {
  console.log('[DEBUG] Effect running', { userId, timestamp: Date.now() });
  // ... rest of effect
}, [userId]);
```

**Solution**: Disable problematic hooks until fully implemented

### 3. MongoDB Connection Errors

**Error**: `ECONNREFUSED ::1:27017` or `127.0.0.1:27017`

**Debug steps**:
```bash
# 1. Check if MongoDB is running
ps aux | grep mongod

# 2. Try to connect
mongosh --host 127.0.0.1 --port 27017

# 3. Check MongoDB config
cat /opt/homebrew/etc/mongod.conf
```

**Solution**:
```bash
# Start MongoDB
mongod --config /opt/homebrew/etc/mongod.conf --fork

# Or use brew services
brew services restart mongodb-community
```

### 4. Multiple Backend Instances

**Symptoms**:
- Port conflicts (trying 5001 → 50011 → 500111)
- `MaxListenersExceededWarning`
- `write after end` errors

**Debug**:
```bash
# Find all instances
ps aux | grep "node.*server" | grep -v grep

# Count instances
ps aux | grep "node.*server" | grep -v grep | wc -l
```

**Solution**:
```bash
# Kill all instances
pkill -9 -f "node.*server"
pkill -9 -f "nodemon"

# Wait and verify
sleep 3
lsof -i :5001  # Should return nothing

# Start fresh
cd server && npm start
```

### 5. Map/Geolocation Errors

**Error**: `Map container is already initialized` or `Geolocation error`

**Solutions**:
- Map: Add stable `key` props
- Geolocation: Add error handling with fallback location

### 6. Next.js 15 Params Error

**Error**: `Cannot assign to read only property 'params'`

**Solution**: Use optional chaining
```typescript
const id = (params?.id as string) || '';
```

## Service Start Order

**Always start in this order**:

1. **MongoDB** (if not running):
```bash
mongod --config /opt/homebrew/etc/mongod.conf --fork
```

2. **Backend**:
```bash
cd server
npm start
```

3. **Frontend**:
```bash
cd apps/web
pnpm dev
```

## Environment Check Script

Create a health check script:

```bash
#!/bin/bash
echo "🔍 PawfectMatch Health Check"
echo ""

# Check MongoDB
if lsof -i :27017 > /dev/null 2>&1; then
  echo "✅ MongoDB: Running"
else
  echo "❌ MongoDB: Not running"
fi

# Check Backend
if lsof -i :5001 > /dev/null 2>&1; then
  echo "✅ Backend: Running on port 5001"
  curl -s http://localhost:5001/api/health > /dev/null 2>&1
  if [ $? -eq 0 ]; then
    echo "   ✅ Backend: Responding to requests"
  else
    echo "   ⚠️  Backend: Not responding"
  fi
else
  echo "❌ Backend: Not running"
fi

# Check Frontend
if lsof -i :3000 > /dev/null 2>&1; then
  echo "✅ Frontend: Running on port 3000"
else
  echo "❌ Frontend: Not running"
fi

# Check .env files
if [ -f "apps/web/.env.local" ]; then
  API_URL=$(grep NEXT_PUBLIC_API_URL apps/web/.env.local | cut -d'=' -f2)
  echo ""
  echo "📋 Frontend API URL: $API_URL"
fi

if [ -f "server/.env" ]; then
  PORT=$(grep "^PORT=" server/.env | cut -d'=' -f2)
  MONGO=$(grep "^MONGODB_URI=" server/.env | cut -d'=' -f2)
  echo "📋 Backend Port: $PORT"
  echo "📋 MongoDB URI: $MONGO"
fi
```

## When All Else Fails

**Nuclear option** - Complete reset:

```bash
# 1. Kill everything
pkill -9 -f "node"
pkill -9 -f "pnpm"

# 2. Stop MongoDB
brew services stop mongodb-community

# 3. Clean node_modules (if needed)
# rm -rf node_modules server/node_modules apps/web/node_modules

# 4. Fresh start
brew services start mongodb-community
sleep 3

cd server && npm start &
cd apps/web && pnpm dev
```

## Log Analysis

**Check for these patterns**:

```bash
# Backend issues
grep -i "error" server/logs/combined.log | tail -20

# MongoDB issues
grep -i "mongo" server/logs/combined.log | tail -20

# CORS issues
grep -i "cors" server/logs/combined.log | tail -20

# Port conflicts
grep -i "EADDRINUSE" server/logs/combined.log | tail -20
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/openann19) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
