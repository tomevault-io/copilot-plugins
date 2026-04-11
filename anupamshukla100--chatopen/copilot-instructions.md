## chatopen

> 1. Go to https://makersuite.google.com/app/apikey

# Google Gemini API Setup Guide

## 1. Get Gemini API Key (FREE!)

1. Go to https://makersuite.google.com/app/apikey
2. Sign in with your Google account
3. Click "Create API Key"
4. Copy the API key

## 2. Configure API Key

Update `backend/.env` with your Gemini API key:

```
GEMINI_API_KEY=your-actual-gemini-api-key-here
```

## 3. Install Dependencies

```bash
cd backend
pip install -r requirements.txt
```

## 4. Benefits of Gemini vs OpenAI

✅ **FREE** - No billing required
✅ **High rate limits** - 60 requests per minute
✅ **Good performance** - Similar quality to GPT-3.5
✅ **Easy setup** - Just need Google account
✅ **No credit card** required

## 5. API Costs

- **Gemini Pro**: FREE up to 60 requests/minute
- **No billing setup** required
- **Perfect for development** and small projects

## 6. Test Integration

1. Start backend: `python run.py`
2. Check status: http://localhost:8000/health
3. Should show: `"gemini_api_key": "configured"`

## 7. Fallback Mode

If Gemini fails, the app automatically falls back to keyword-based search.

## 8. Rate Limits

- **Free tier**: 60 requests per minute
- **Paid tier**: Higher limits available
- **Automatic retry** on rate limit errors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AnupamShukla100)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AnupamShukla100)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
