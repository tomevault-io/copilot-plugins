## unity-pilot-windsurf

> This document explains how to set up and use the Gemini 2.0 Flash AI model in your chat application.

# Gemini AI Integration

This document explains how to set up and use the Gemini 2.0 Flash AI model in your chat application.

## Setup Instructions

1. **Get a Gemini API Key**
   - Go to [Google AI Studio](https://aistudio.google.com/app/apikey)
   - Create an account if you don't have one
   - Generate a new API key

2. **Add API Key to Environment Variables**
   - Open your `.env.local` file (or create one if it doesn't exist)
   - Add your Gemini API key:
     ```
     NEXT_PUBLIC_GEMINI_API_KEY=your-api-key-here
     ```

3. **Restart Your Development Server**
   - If your app is running, restart it to load the new environment variables

## How It Works

This integration uses Google's Gemini 2.0 Flash model to generate responses in your chat application. Here's how the system works:

1. When a user sends a message, it's saved to your Supabase database
2. The application fetches the complete chat history for context
3. The message and chat history are sent to the Gemini agent
4. The agent uses LangChain to format the prompt and generate a response
5. The AI response is saved to the database and displayed in the chat

## Files and Components

- `lib/agent/config.ts` - Configuration settings for the Gemini API
- `lib/agent/gemini.ts` - Functions to interact with the Gemini API
- `lib/agent/agent.ts` - LangChain agent implementation that uses Gemini

## Customization

You can customize the agent's behavior by modifying these files:

### Changing the System Prompt

In `lib/agent/agent.ts`, modify the `systemPrompt` variable to change the AI's personality or instructions:

```typescript
const systemPrompt = `You are a helpful AI assistant designed to have natural conversations with users.
Your responses should be informative, friendly, and engaging.
Use the conversation history to provide context-aware responses.
If you don't know something, be honest about it rather than making up information.
`;
```

### Adjusting Generation Parameters

In `lib/agent/config.ts`, you can modify the model parameters to control the AI's response style:

```typescript
defaultGenerationConfig: {
  temperature: 0.7,  // Higher = more creative, lower = more deterministic
  topP: 0.8,         // Controls diversity
  topK: 40,          // Controls vocabulary diversity
  maxOutputTokens: 1024, // Maximum response length
}
```

## Troubleshooting

If you encounter issues with the Gemini integration:

1. **Check your API key**: Make sure your Gemini API key is correct and properly set in `.env.local`

2. **Check for errors in the console**: Open your browser's developer tools to see any error messages

3. **Rate limits**: The Gemini API has rate limits. If you're making too many requests, you may be temporarily blocked

4. **Model availability**: Ensure that the model specified in `config.ts` is available in your region and with your API key

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mooumari)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mooumari)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
