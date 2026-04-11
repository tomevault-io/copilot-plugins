## classless-app

> Get real audio-to-text transcription working instead of mock results.

# Quick Setup Guide for Real Audio Transcription

## 🎯 Goal
Get real audio-to-text transcription working instead of mock results.

## 📋 Step-by-Step Instructions

### Step 1: Get Gemini API Key
1. Go to: https://makersuite.google.com/app/apikey
2. Sign in with Google account
3. Click "Create API Key"
4. Copy the API key

### Step 2: Create Environment File
1. In your project root (CLASS_LESS-main folder), create a file named `.env.local`
2. Add this content:
```
GEMINI_API_KEY=your_actual_api_key_here
```
3. Replace `your_actual_api_key_here` with your real API key

### Step 3: Test API Key
Run this command to test your API key:
```bash
node test-gemini-api.js
```

### Step 4: Restart Server
1. Stop your current server (Ctrl+C)
2. Run: `npm run dev`
3. Go to: http://localhost:3001/ask

### Step 5: Test Real Transcription
1. Select "Voice" question type
2. Record: "What is Java?"
3. Click "Transcribe"
4. You should see "Gemini AI" in the response

## 🔍 How to Verify It's Working

### ✅ Success Indicators:
- Console shows: `[Transcribe] Gemini AI transcription successful!`
- Response shows: `"source": "gemini-ai"`
- Transcription matches what you actually said

### ❌ Still Mock Mode:
- Console shows: `[Transcribe] No Gemini API key found`
- Response shows: `"source": "mock"`
- Transcription is random educational questions

## 🚨 Common Issues

### Rate Limit (429 Error)
- **Problem**: "Too Many Requests"
- **Solution**: Wait for quota reset or upgrade plan
- **Alternative**: Use mock mode for testing

### API Key Not Found
- **Problem**: Still getting mock results
- **Solution**: Check `.env.local` file exists and has correct API key
- **Fix**: Restart server after adding API key

### Audio Not Transcribing
- **Problem**: API works but transcription is wrong
- **Solution**: Speak clearly, avoid background noise
- **Tip**: Keep recordings under 30 seconds

## 📞 Need Help?

1. Check console logs for detailed error messages
2. Run `node test-gemini-api.js` to test API key
3. Verify `.env.local` file is in the correct location
4. Make sure you restarted the server after adding the API key

## 🎉 Expected Result

When you record "What is Java?" and transcribe it, you should get:
- **Text**: "What is Java?" (or very similar)
- **Source**: "gemini-ai"
- **Confidence**: 95%

Instead of the current mock result: "How do I write a thesis statement?"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vigneshsiva11)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vigneshsiva11)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
