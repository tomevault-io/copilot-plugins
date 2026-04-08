## campus-career-system

> **Status**: URGENT - Need to fix 2 issues in Render

# 🚨 CRITICAL FIX - MongoDB & API Key Configuration

**Status**: URGENT - Need to fix 2 issues in Render  
**Date**: March 15, 2026  
**Time Estimate**: 10-15 minutes

---

## ❌ ISSUE 1: MongoDB Authentication Failed

### What's Happening:
```
Error: bad auth : authentication failed
Connection string: mongodb+srv://***:***@cluster0.annsvcl.mongodb.net/...
```

The MongoDB credentials in MONGO_URI are **WRONG** or **EXPIRED**.

---

## 🔧 QUICK FIX - STEP BY STEP

### STEP 1: Get Correct MongoDB Credentials

**Go to MongoDB Atlas:**
1. Visit https://cloud.mongodb.com/
2. Sign in with your MongoDB account
3. Click "Campus-Career-System" cluster or find the right cluster
4. Click "Connect"
5. Click "Drivers" or "Connect your application"
6. Copy the **complete connection string**

**It should look like:**
```
mongodb+srv://USERNAME:PASSWORD@cluster0.annsvcl.mongodb.net/?appName=Cluster0
```

**Important**: Make sure USERNAME and PASSWORD are correct!

### STEP 2: Update in Render

**Access Render Environment Variables:**
1. Go to https://dashboard.render.com
2. Click "campus-career-backend" service
3. Click "Environment" section
4. Find "MONGO_URI" variable
5. Click [Edit]

**Paste the correct connection string:**
```
mongodb+srv://USERNAME:PASSWORD@cluster0.annsvcl.mongodb.net/?appName=Cluster0
```

Where:
- `USERNAME` = Your MongoDB user (e.g., mohammedmuzhirtaha)
- `PASSWORD` = Your MongoDB password (the actual password, not a placeholder)
- Keep everything else exactly the same

6. Click [Save]
7. Render will **automatically restart** the service

**Verify it works:**
- Wait 30-60 seconds for restart
- Check logs to see if MongoDB connects ✅

---

## ❌ ISSUE 2: GEMINI_API_KEY Not Found in Render

### What's Happening:
```
⚠️  GEMINI_API_KEY not found. AI features will use fallback mode.
```

The environment variable is **MISSING** in Render, even though we created it.

---

## 🔧 QUICK FIX - ADD API KEY

### STEP 1: Add to Render

**Access Render Environment Variables:**
1. Go to https://dashboard.render.com
2. Click "campus-career-backend" service
3. Click "Environment" section
4. Look for "GEMINI_API_KEY"

**If it's missing:**
1. Click "Add Environment Variable"
2. Key: `GEMINI_API_KEY`
3. Value: `AIzaSyBz9sDddZKoKeaQa89IDNxjJe2bCVMvSZQ`
4. Click [Save]

**If it exists but is empty:**
1. Click [Edit]
2. Paste: `AIzaSyBz9sDddZKoKeaQa89IDNxjJe2bCVMvSZQ`
3. Click [Save]

Service will **auto-restart**.

---

## 📋 VERIFICATION CHECKLIST

After making both changes, verify in Render:

```
Environment Variables (should be 6 total):
✅ NODE_ENV = production
✅ PORT = 5000
✅ MONGO_URI = mongodb+srv://USERNAME:PASSWORD@cluster0.annsvcl.mongodb.net/...
✅ JWT_SECRET = 7d40941a570e29d2de39bb699125ed13d33a69200c1ae66bfd8ba581ee41724a
✅ GEMINI_API_KEY = AIzaSyBz9sDddZKoKeaQa89IDNxjJe2bCVMvSZQ
✅ CORS_ORIGIN = https://campus-career-system-c2tx.vercel.app
```

---

## ⏳ WHAT HAPPENS NEXT

1. **You update both variables**
2. **Render restarts automatically**
3. **You check logs:**
   - Should see "✓ Server running on port 5000"
   - Should see "✓ MongoDB connection successful" (wait up to 60 seconds)
   - Should see "✓ Gemini service initialized"

4. **System is ready** ✅

---

## 🧪 TEST AFTER FIXES

Once both are fixed, test:

```bash
# Test 1: Backend health
curl https://campus-career-backend-xxx.onrender.com/api/health

# Test 2: Get jobs (tests both DB + API key setup)
curl https://campus-career-backend-xxx.onrender.com/api/ai/jobs

# Both should return 200 OK with data
```

---

## 🆘 TROUBLESHOOTING

### "Still getting MongoDB error after update"

**Try this:**
1. Go to MongoDB Atlas → Security → Network Access
2. Check if "0.0.0.0/0" (All IP) is whitelisted
3. If not, add it
4. Try Render again

**Or:**
1. In MongoDB Atlas, create a NEW database user
2. Use that user in MONGO_URI
3. Update Render environment

### "GEMINI_API_KEY error still showing"

**Try this:**
1. In Render, delete GEMINI_API_KEY
2. Click "Add new" variable
3. Carefully paste: `AIzaSyBz9sDddZKoKeaQa89IDNxjJe2bCVMvSZQ`
4. Watch for typos - no extra spaces!
5. Save and wait for restart

### "Service won't restart"

1. Go to Render dashboard
2. Find your service
3. Click "Restart service" button
4. Wait 2-3 minutes

---

## 🔐 ADDITIONAL SECURITY CHECK

While you're in Render, verify all variables are set correctly:

1. **MONGO_URI should be:**
   - ✅ Complete connection string
   - ✅ Contains your actual MongoDB user
   - ✅ Contains your actual MongoDB password
   - ❌ NOT have placeholder values

2. **GEMINI_API_KEY should be:**
   - ✅ Set to actual key value
   - ✅ No extra spaces or quotes
   - ✅ Exactly: `AIzaSyBz9sDddZKoKeaQa89IDNxjJe2bCVMvSZQ`
   - ❌ NOT commented out or partial

3. **All other variables should match**

---

## 📊 EXPECTED SUCCESS STATE

After fixes, logs should show:

```
✓ Server running on port 5000
✓ Environment: production
✓ MongoDB connection successful ✅
✓ Gemini service initialized ✅
✓ All features ready for use
```

---

## ⚡ PRIORITY ORDER

1. **First**: Fix MONGO_URI (most critical - affects all features)
2. **Second**: Fix GEMINI_API_KEY (affects AI features)
3. **Third**: Test and verify both work

---

**This should take 10 minutes maximum!**

Once both are fixed:
✅ Database will work
✅ AI features will work  
✅ Frontend can access all data
✅ Users can upload resumes
✅ System is fully operational

Go fix these now! 🚀

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IGNITETHETEAM-2)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IGNITETHETEAM-2)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
