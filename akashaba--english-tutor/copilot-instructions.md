## english-tutor

> ✅ **Switched from OpenAI GPT-3.5 to Google Gemini 2.0 Flash**

# 🚀 Quick Start Guide - Gemini 2.0 Flash Edition

## What Changed?

✅ **Switched from OpenAI GPT-3.5 to Google Gemini 2.0 Flash**

- Faster responses
- Uses your existing Google Cloud API key
- More cost-effective
- Better integration with Text-to-Speech

## How to Start the App

### Step 1: Restart Expo

```powershell
cd EnglishTutorApp
npx expo start -c
```

### Step 2: Test Gemini API (Optional)

```powershell
node test-gemini.js
```

This will verify Gemini is responding before you start the app.

### Step 3: Open in Browser

- Press `w` in the Expo terminal
- Navigate to "Talk with Sunny" tab
- Try having a conversation!

## What to Look For

### ✅ Success Indicators:

1. Console shows: `🤖 Calling Gemini 2.0 Flash API...`
2. Console shows: `✅ Gemini response received`
3. Sunny responds in the chat
4. You hear Sunny's voice (Text-to-Speech)

### ❌ If Not Working:

**If you see "Gemini API key not configured":**

- Check `app.json` has GOOGLE_CLOUD_API_KEY
- Run `node test-gemini.js` to verify

**If you see 400 error:**

- Enable "Generative Language API" in Google Cloud Console
- URL: https://console.cloud.google.com/apis/

**If you see 403 error:**

- Check billing is enabled
- Verify API key restrictions allow Gemini

**If responses are slow/not coming:**

- Check browser console (F12) for detailed errors
- Ensure stable internet connection
- Check Google Cloud quota limits

## Debugging Commands

### Check Configuration:

```powershell
node check-audio-setup.js
```

### Test Gemini Only:

```powershell
node test-gemini.js
```

### View Expo Logs:

- Open browser console (F12)
- Look for colorful emoji logs: 🤖 ✅ ❌

## Key Differences from OpenAI

| Feature     | OpenAI (Old)        | Gemini (New)          |
| ----------- | ------------------- | --------------------- |
| Model       | gpt-3.5-turbo       | gemini-2.0-flash-exp  |
| API Key     | Separate OpenAI key | Same Google Cloud key |
| Speed       | ~2-3 seconds        | ~1-2 seconds          |
| Cost        | Higher              | Lower                 |
| Integration | Separate service    | Unified with TTS      |

## Expected Behavior

1. **Type or speak** to Sunny
2. **Gemini processes** your message (fast!)
3. **Sunny responds** with text
4. **Text-to-Speech** plays Sunny's voice
5. **Conversation continues** naturally

## Troubleshooting Checklist

- [ ] Expo server restarted with `-c` flag
- [ ] Browser cache cleared (Ctrl+Shift+R)
- [ ] Google Cloud API key in app.json
- [ ] Generative Language API enabled in Google Cloud
- [ ] Billing enabled in Google Cloud account
- [ ] Internet connection stable
- [ ] Browser console open (F12) to see logs

## Support Links

- **Enable Gemini API**: https://console.cloud.google.com/apis/library/generativelanguage.googleapis.com
- **API Credentials**: https://console.cloud.google.com/apis/credentials
- **Billing**: https://console.cloud.google.com/billing
- **Quotas**: https://console.cloud.google.com/apis/api/generativelanguage.googleapis.com/quotas

## Quick Reference

### Config File Location:

`constants/config.ts`

### AI Response Function:

`app/learn.tsx` → `getAIResponse()`

### Test Script:

`test-gemini.js`

### Documentation:

`GEMINI_MIGRATION.md`

---

**Ready to go!** Just run `npx expo start -c` and test! 🎉

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akashaba)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/akashaba)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
