## tasker-server

> Support all message and media types in conversation history and quoted messages


# Support All Message & Media Types

## Core Requirement

**The Agent must understand and process all types of messages and media:**

1. ✅ **All media types** in conversation history (images, videos, audio, polls)
2. ✅ **Quoted/replied messages** with proper context passing
3. ✅ **Both incoming and outgoing** message flows

---

## 1️⃣ Media Types in Conversation History

When saving messages to history, always include media indicators:

```javascript
// ✅ Correct - in routes/whatsappRoutes.js
if (hasImage) {
  botMessageContent = `[תמונה מצורפת]\n${botResponse}`;
} else if (hasVideo) {
  botMessageContent = `[וידאו מצורף]\n${botResponse}`;
} else if (hasAudio) {
  botMessageContent = `[הקלטה קולית]`;
}
```

**Supported media types:**
- 🖼️ Images: `[תמונה מצורפת]`
- 🎬 Videos: `[וידאו מצורף]`
- 🎤 Audio: `[הקלטה קולית]`
- 📊 Polls: `[סקר]`
- 📍 Location: `[מיקום]`

---

## 2️⃣ Quoted Messages Support

Quoted messages must pass full context to the Agent:

```javascript
// ✅ Correct - in routes/whatsappRoutes.js
let quotedContext = null;
if (isActualQuote && quotedMessage) {
  quotedContext = {
    type: quotedMessage.typeMessage || 'unknown',
    text: quotedMessage.textMessage || quotedMessage.caption || '',
    hasImage: quotedMessage.typeMessage === 'imageMessage',
    hasVideo: quotedMessage.typeMessage === 'videoMessage',
    hasAudio: quotedMessage.typeMessage === 'audioMessage',
    stanzaId: quotedMessage.stanzaId
  };
}

const normalized = {
  userText: `# ${finalPrompt}`,
  quotedContext: quotedContext, // ← Pass to Agent
  // ...
};
```

**Then in agentPilot.js:**
```javascript
if (input.quotedContext) {
  contextualPrompt = `[הודעה מצוטטת: ${input.quotedContext.type}]\n${input.quotedContext.text}\n\n[בקשה נוכחית:]\n${userText}`;
}
```

---

## 3️⃣ Agent Tools Must Handle Media

When creating tools in `agentService.js`, consider media context:

```javascript
// Example: analyze_image_from_history tool
analyze_image_from_history: {
  declaration: {
    name: 'analyze_image_from_history',
    description: 'נתח תמונה שהמשתמש שלח קודם (מההיסטוריה)'
    // ...
  },
  execute: async (args, context) => {
    // Get history first
    const history = context.previousToolResults?.get_chat_history?.messages;
    // Find image by ID
    const imageMsg = history[args.image_id];
    // Download and analyze
    // ...
  }
}
```

---

## 4️⃣ Testing Checklist

When adding media support or quoted message handling:

- [ ] Test with image attachment
- [ ] Test with video attachment
- [ ] Test with audio message
- [ ] Test with quoted text message
- [ ] Test with quoted image
- [ ] Test with quoted video
- [ ] Verify media appears in conversation history correctly
- [ ] Verify Agent can access media from history

---

## Examples

### ✅ Correct: Media in History
```javascript
await conversationManager.addMessage(chatId, 'assistant', botMessageContent, {
  hasImage: !!imageUrl,
  hasVideo: !!videoUrl,
  hasAudio: !!audioUrl,
  imageUrl: imageUrl,
  videoUrl: videoUrl
});
```

### ✅ Correct: Quoted Context Passing
```javascript
// routes/whatsappRoutes.js
const normalized = {
  userText: prompt,
  quotedContext: {
    type: 'imageMessage',
    text: 'תמונת חתול',
    hasImage: true,
    stanzaId: 'ABC123'
  }
};

// agentPilot.js receives and uses it
if (input.quotedContext?.hasImage) {
  contextualPrompt = `[הודעה מצוטטת: תמונה]\n...\n\n[בקשה:]\n${userText}`;
}
```

---

## Remember

- **All media types** must be logged in history
- **Quoted messages** need full context (type, text, media flags)
- **Agent tools** should be able to access historical media
- **Test thoroughly** - media handling is critical for user experience

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aviv90) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
