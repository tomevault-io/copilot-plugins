## whatsapp-ai-bot

> WhatsApp AI Bot is a TypeScript-based chatbot that integrates multiple AI models with WhatsApp using the Baileys library. The bot supports text-to-text models (ChatGPT, Gemini, Ollama) and text-to-image models (DALL-E, Flux, Stability AI), with a flexible custom model system.

# CLAUDE.md - AI Assistant Development Guide

## Project Overview

WhatsApp AI Bot is a TypeScript-based chatbot that integrates multiple AI models with WhatsApp using the Baileys library. The bot supports text-to-text models (ChatGPT, Gemini, Ollama) and text-to-image models (DALL-E, Flux, Stability AI), with a flexible custom model system.

**Key Technologies:**
- TypeScript (strict mode)
- Baileys (WhatsApp Web API)
- Multiple AI Provider SDKs (OpenAI, Google Generative AI, Stability AI, etc.)
- MongoDB (optional session storage)
- vite-node (development runtime)

## Repository Structure

```
whatsapp-ai-bot/
├── src/
│   ├── index.ts                    # Entry point
│   ├── whatsapp-ai.config.ts       # Bot configuration
│   ├── baileys/                    # WhatsApp integration layer
│   │   ├── index.ts                # Connection setup
│   │   ├── env.ts                  # Environment variables
│   │   ├── handlers/
│   │   │   ├── messages.ts         # Message batch handler
│   │   │   └── message.ts          # Individual message handler
│   │   ├── hooks/
│   │   │   └── useMessageParser.ts # Message metadata parser
│   │   └── database/
│   │       └── mongo.ts            # MongoDB connection
│   ├── models/                     # AI model implementations
│   │   ├── BaseAiModel.ts          # Abstract base class
│   │   ├── GeminiModel.ts          # Google Gemini
│   │   ├── OpenAIModel.ts          # ChatGPT & DALL-E
│   │   ├── StabilityModel.ts       # Stability AI
│   │   ├── FluxModel.ts            # Hugging Face Flux
│   │   ├── OllamaModel.ts          # Ollama
│   │   └── CustomModel.ts          # Context-aware custom models
│   ├── types/                      # TypeScript type definitions
│   │   ├── AiModels.d.ts          # Model type unions
│   │   └── Config.d.ts            # Configuration interfaces
│   └── util/                       # Utility functions
│       ├── Util.ts                 # Prefix matching, file reading
│       └── MessageTemplates.ts     # Response templates
├── docs/                           # Documentation
├── .env.example                    # Environment template
├── package.json                    # Dependencies & scripts
└── tsconfig.json                   # TypeScript config (strict mode)
```

## Architecture

### Core Design Patterns

1. **Abstract Base Model Pattern**: All AI models extend `AIModel<AIArguments, CallBack>` abstract class (src/models/BaseAiModel.ts:63)
2. **Configuration-Driven**: Models are enabled/disabled via .env and configured in whatsapp-ai.config.ts
3. **Prefix-Based Routing**: Messages are routed to models based on prefix matching (!gemini, !chatgpt, etc.)
4. **Session Management**: Each user gets isolated conversation history per model
5. **Metadata-Rich Processing**: Messages are parsed into rich metadata objects before processing

### Message Flow

```
WhatsApp Message
    ↓
[baileys/index.ts] connectToWhatsApp() - Event listener setup
    ↓
[handlers/messages.ts] messagesHandler() - Batch processing
    ↓
[hooks/useMessageParser.ts] useMessageParser() - Extract metadata
    ↓
[handlers/message.ts] handleMessage() - Route to model
    ↓
[util/Util.ts] getModelByPrefix() - Match prefix to model
    ↓
[models/*Model.ts] sendMessage() - Generate response
    ↓
[baileys] client.sendMessage() - Send reply
```

## Configuration System

### Environment Variables (.env)

All models are **disabled by default** for security. Enable explicitly:

```bash
# Enable a model
GEMINI_ENABLED=True
GEMINI_PREFIX=!gemini
API_KEY_GEMINI=your_api_key_here

# Optional: Customize icon prefix
GEMINI_ICON_PREFIX=🔮
```

**Important ENV patterns:**
- Boolean values use string "True" (case-sensitive): `process.env.GEMINI_ENABLED === 'True'` (src/baileys/env.ts:83)
- All API keys are optional but required if model is enabled
- Processing message customizable via `PROCESSING` env var (src/baileys/env.ts:58)

### Model Configuration (whatsapp-ai.config.ts)

```typescript
const config: Config = {
  sendWelcomeMessage: true,
  models: {
    ChatGPT: { prefix: ENV.OPENAI_PREFIX, enable: ENV.OPENAI_ENABLED },
    Gemini: { prefix: ENV.GEMINI_PREFIX, enable: ENV.GEMINI_ENABLED },
    // ... other models
    Custom: [
      {
        modelName: 'whatsapp-ai-bot',
        prefix: '!wa',
        enable: true,
        context: './docs/wa-ai-bot.md', // Can be file path, URL, or text
        baseModel: 'Gemini' // or 'ChatGPT'
      }
    ]
  },
  prefix: {
    enabled: true, // If false, uses defaultModel for all messages
    defaultModel: 'ChatGPT'
  },
  sessionStorage: { enable: true, wwjsPath: './' },
  selfMessage: { skipPrefix: false }
};
```

## AI Model System

### BaseAiModel Abstract Class (src/models/BaseAiModel.ts)

All models must implement:

```typescript
abstract class AIModel<AIArguments, CallBack> {
  // Session management
  public sessionCreate(user: string): void
  public sessionRemove(user: string): void
  public sessionExists(user: string): boolean
  public sessionAddMessage(user: string, args: any): void

  // Required implementation
  abstract sendMessage(args: AIArguments, handle: CallBack): Promise<any>

  // Icon prefix for responses
  public addPrefixIcon(text: string): string
}
```

### AIArguments Interface

```typescript
interface AIArguments {
  sender: string;           // User ID
  prompt: string;           // User message (prefix removed)
  metadata: AIMetaData;     // Rich message metadata
  prefix: string;           // Model prefix used
}
```

### AIMetaData Interface (src/models/BaseAiModel.ts:7-54)

Contains comprehensive message information:
- Basic: `remoteJid`, `sender`, `senderName`, `fromMe`, `timeStamp`
- Message type: `msgType` (text/image/video/audio/document/contact/location)
- Quote handling: `isQuoted`, `quoteMetaData` (for replied messages)
- Media: `hasImage`, `imgMetaData`, `hasAudio`, `audioMetaData`
- Group info: `isGroup`, `groupMetaData`

### Creating a New Model

1. **Create model file** in `src/models/YourModel.ts`:

```typescript
import { AIModel, AIArguments, AIHandle } from './BaseAiModel';
import { ENV } from '../baileys/env';

class YourModel extends AIModel<AIArguments, AIHandle> {
  public constructor() {
    super(ENV.API_KEY_YOUR_MODEL, 'YourModel', ENV.YOUR_MODEL_ICON_PREFIX);
  }

  async sendMessage({ sender, prompt, metadata }: AIArguments, handle: AIHandle) {
    try {
      // Create session if needed
      if (!this.sessionExists(sender)) {
        this.sessionCreate(sender);
      }

      // Call your AI API
      const response = await yourApiCall(prompt);

      // Return response with icon prefix
      handle({ text: this.addPrefixIcon(response) });
    } catch (err) {
      handle('', 'Error: ' + err);
    }
  }
}

export { YourModel };
```

2. **Add environment configuration** in `src/baileys/env.ts`:

```typescript
interface EnvInterface {
  // ... existing fields
  YOUR_MODEL_PREFIX?: string;
  YOUR_MODEL_ENABLED: boolean;
  API_KEY_YOUR_MODEL?: string;
}

export const ENV: EnvInterface = {
  // ... existing values
  YOUR_MODEL_PREFIX: process.env.YOUR_MODEL_PREFIX,
  YOUR_MODEL_ENABLED: process.env.YOUR_MODEL_ENABLED === 'True',
  API_KEY_YOUR_MODEL: process.env.API_KEY_YOUR_MODEL,
};
```

3. **Add to type definitions** in `src/types/AiModels.d.ts`:

```typescript
export type AIModels = 'ChatGPT' | 'Gemini' | 'FLUX' | 'Stability' | 'Dalle' | 'Ollama' | 'YourModel' | 'Custom';
```

4. **Register in config** (src/whatsapp-ai.config.ts):

```typescript
const config: Config = {
  models: {
    // ... existing models
    YourModel: {
      prefix: ENV.YOUR_MODEL_PREFIX,
      enable: ENV.YOUR_MODEL_ENABLED
    }
  }
};
```

5. **Add to model table** in `src/baileys/handlers/message.ts`:

```typescript
import { YourModel } from '../../models/YourModel';

const modelTable: Record<AIModels, any> = {
  // ... existing models
  YourModel: ENV.YOUR_MODEL_ENABLED ? new YourModel() : null,
};
```

6. **Update .env.example**:

```bash
YOUR_MODEL_PREFIX=!yourmodel
YOUR_MODEL_ENABLED=False
API_KEY_YOUR_MODEL=ADD_YOUR_KEY
```

### Custom Models

Custom models allow context-aware responses without creating new model classes:

```typescript
Custom: [
  {
    modelName: 'product-expert',
    prefix: '!product',
    enable: true,
    context: './docs/product-knowledge.md', // File path
    baseModel: 'Gemini',
    dangerouslyAllowFewShotApproach: false // Use system instructions vs prompt injection
  },
  {
    modelName: 'support-bot',
    prefix: '!support',
    enable: true,
    context: 'https://example.com/support-docs', // URL
    baseModel: 'ChatGPT'
  },
  {
    modelName: 'simple-bot',
    prefix: '!simple',
    enable: true,
    context: 'You are a helpful assistant that...', // Plain text
    baseModel: 'Gemini'
  }
]
```

**Context sources** (src/models/CustomModel.ts:75-101):
- File path: `.md`, `.txt`, `.text` files
- URL: Starts with `http://`, `https://`, `ftp://`, etc.
- Plain text: Any other string

**System instruction modes:**
- `dangerouslyAllowFewShotApproach: false` (default): Uses model's system instruction API
- `dangerouslyAllowFewShotApproach: true`: Injects context into prompt (less reliable)

## Development Workflow

### Setup

```bash
# Clone repository
git clone https://github.com/Zain-ul-din/WhatsApp-Ai-bot.git
cd WhatsApp-Ai-bot

# Install dependencies
yarn install  # or npm install

# Configure environment
cp .env.example .env
# Edit .env with your API keys and enable desired models
```

### Development Commands

```bash
# Start development server (auto-reload)
yarn dev

# Format code with Prettier
yarn format

# Run tests
yarn test

# Build (just runs yarn install)
yarn build

# Start (install + dev)
yarn start
```

### TypeScript Configuration

The project uses **strict mode** TypeScript (tsconfig.json:8):
- `strict: true` - All strict type checks enabled
- `noImplicitAny: true` - No implicit any types
- `strictNullChecks: true` - Null/undefined must be explicit
- `noImplicitReturns: true` - All code paths must return
- `noUnusedParameters: true` - Unused params are errors

### Coding Conventions

1. **Imports**: Organize as Third-party → Local modules → Types/Utils
   ```typescript
   /* Third-party modules */
   import { makeWASocket } from '@whiskeysockets/baileys';

   /* Local modules */
   import { ENV } from './env';
   import config from '../whatsapp-ai.config';
   ```

2. **Error Handling**: Always use try-catch in model sendMessage()
   ```typescript
   async sendMessage(args: AIArguments, handle: AIHandle) {
     try {
       // ... implementation
       handle({ text: response });
     } catch (err) {
       handle('', 'Error: ' + err);
     }
   }
   ```

3. **Async/Await**: Prefer async/await over .then() chains

4. **Type Safety**: Use defined interfaces, avoid `any` except for Baileys types

5. **Session Management**: Check session exists before using:
   ```typescript
   if (!this.sessionExists(sender)) {
     this.sessionCreate(sender);
   }
   ```

## Message Processing Deep Dive

### Prefix Matching (src/util/Util.ts:7-44)

The `Util.getModelByPrefix()` method:
- Case-insensitive prefix matching
- Skips disabled models
- Returns model metadata for routing
- Custom models handled separately with `getModelByCustomPrefix()`

### Message Metadata Parser (src/baileys/hooks/useMessageParser.ts)

Extracts comprehensive metadata:
- Message type detection (text/image/video/audio/document/contact/location)
- Quote/reply handling with full context
- Group metadata (name, locked status)
- Image handling with URL, mimeType, caption
- Audio handling with URL, mimeType
- Timestamp conversion

### Image Support

Models can handle images in two ways:

1. **Direct image message**: `metadata.hasImage = true`
2. **Quoted image**: `metadata.isQuoted && metadata.quoteMetaData.hasImage = true`

Example from GeminiModel (src/models/GeminiModel.ts:83-114):
```typescript
if (metadata.isQuoted && metadata.quoteMetaData.hasImage) {
  message = await this.generateImageCompletion(
    prompt,
    metadata.quoteMetaData.imgMetaData,
    metadata.quoteMetaData.message
  );
} else if (metadata.hasImage) {
  message = await this.generateImageCompletion(
    prompt,
    metadata.imgMetaData,
    metadata.message.message
  );
}
```

### Response Types

Models can return:

1. **Text response**:
   ```typescript
   handle({ text: 'Response message' });
   ```

2. **Image response** (text-to-image models):
   ```typescript
   handle({
     image: { url: 'https://...' },
     caption: 'Generated image'
   });
   ```

The message handler automatically:
- Deletes "Processing..." message for images
- Edits "Processing..." message for text responses
- Handles errors gracefully

## Authentication & Sessions

### WhatsApp Authentication

Two modes supported (src/baileys/index.ts:16-24):

1. **File-based** (default):
   ```typescript
   await useMultiFileAuthState('auth_info_baileys');
   ```
   - Stores in `./auth_info_baileys/` directory
   - QR code shown on first run
   - Session persists across restarts

2. **MongoDB-based**:
   ```env
   MONGO_ENABLED=True
   MONGO_URL=mongodb://localhost:27017/whatsapp-bot
   ```
   - Stores in MongoDB collection
   - Useful for multi-instance deployments

### Connection Handling (src/baileys/index.ts:38-60)

Auto-reconnect logic:
- Reconnects on connection close (unless logged out)
- Deletes credentials on logout and reconnects (shows new QR)
- Saves credentials on updates

## Common Development Tasks

### Adding a New Command Prefix

1. Update model configuration in `whatsapp-ai.config.ts`
2. Set prefix in `.env` file
3. No code changes needed - prefix routing is automatic

### Changing AI Model Parameters

Each model class manages its own parameters:

- **Gemini**: Model selection in constructor (src/models/GeminiModel.ts:33)
  ```typescript
  this.generativeModel = this.Gemini.getGenerativeModel({
    model: 'gemini-1.5-flash', // Change here
    systemInstruction: this.instructions
  });
  ```

- **ChatGPT**: Model in env (src/baileys/env.ts:71)
  ```bash
  OPENAI_MODEL=gpt-4  # or gpt-3.5-turbo
  ```

### Debugging

Enable debug mode in `.env`:
```bash
DEBUG=True
```

Debug logs appear in:
- `src/baileys/handlers/message.ts:41` - Model not found
- `src/baileys/handlers/message.ts:62` - Model disabled

### Ignoring Self Messages

```bash
IGNORE_SELF_MESSAGES=True
```

Useful when bot shouldn't respond to messages you send.

### Handling Group Messages

Groups automatically detected by JID ending with `@g.us` (src/baileys/hooks/useMessageParser.ts:128)

Locked groups (restrict mode) are ignored (src/baileys/handlers/messages.ts:24)

## Testing

### Manual Testing Flow

1. Start bot: `yarn dev`
2. Scan QR code with WhatsApp
3. Send test messages with prefixes:
   - `!gemini Hello` - Test Gemini
   - `!chatgpt What is TypeScript?` - Test ChatGPT
   - `!dalle A sunset over mountains` - Test image generation

### Test Files Location

- `test/index.test.ts` - Main tests (run with `yarn test`)
- `baileys/index.ts` - Baileys integration tests (run with `yarn test:baileys`)

## Deployment

### Environment Requirements

- Node.js 18+ (for native fetch support in CustomModel)
- Yarn or npm
- Valid API keys for enabled models
- (Optional) MongoDB instance for distributed sessions

### Docker Deployment

Dockerfile included in repository. Build and run:

```bash
docker build -t whatsapp-ai-bot .
docker run -d --env-file .env whatsapp-ai-bot
```

### Cloud Deployment

See `docs/deployment.md` for GitHub Codespaces and cloud deployment guides.

**Important**: On first deployment, scan QR code from logs to authenticate.

## Troubleshooting

### Common Issues

1. **"Model not found" in debug logs**
   - Check model is enabled in `.env` (`MODELNAME_ENABLED=True`)
   - Check prefix matches in `whatsapp-ai.config.ts`
   - Verify prefix in message is lowercase-matched

2. **API key errors**
   - Verify API key in `.env` matches provider requirements
   - Check API key has proper permissions
   - For DALL-E: Can use separate key with `API_KEY_OPENAI_DALLE` or share with ChatGPT

3. **Connection issues**
   - Delete `auth_info_baileys/` folder and re-scan QR
   - Check WhatsApp Web is not open elsewhere
   - Verify internet connection stability

4. **TypeScript errors**
   - Run `yarn install` to ensure dependencies match
   - Check strict mode compliance
   - Verify all required fields in interfaces are provided

5. **Image processing fails**
   - Verify mimeType is in validMimeTypes (src/models/GeminiModel.ts:21)
   - Check image is properly downloaded with `downloadMediaMessage`
   - Ensure model supports vision (Gemini does, ChatGPT may need GPT-4 Vision)

## Best Practices for AI Assistants

### When Making Changes

1. **Always read files before modifying** - Use Read tool to understand current implementation
2. **Follow existing patterns** - Match import organization, error handling, and type usage
3. **Maintain type safety** - No `any` types except for Baileys library types
4. **Test incrementally** - Test each model addition/change individually
5. **Update documentation** - Keep CLAUDE.md in sync with code changes

### Code Modification Guidelines

1. **Don't break existing models** - Changes should be additive or isolated
2. **Preserve session management** - Don't skip `sessionExists()` checks
3. **Handle errors gracefully** - Always provide user-friendly error messages
4. **Respect configuration** - Honor enabled/disabled states from config
5. **Maintain backwards compatibility** - Existing .env files should still work

### Architecture Principles

1. **Single Responsibility** - Each model handles one AI provider
2. **Open/Closed** - Easy to add models, hard to break existing ones
3. **Configuration over Code** - Prefer .env changes to code changes
4. **Fail Gracefully** - Disabled models don't break the bot
5. **Session Isolation** - User conversations don't leak between models

## Git Workflow

### Branch Strategy

- Main branch: `main` or `master`
- Feature branches: `feature/model-name` or `fix/issue-description`
- Development on assigned branch: `claude/claude-md-*` (for AI sessions)

### Commit Messages

Follow conventional commits:
```
feat: add Anthropic Claude model support
fix: handle empty prompts in GeminiModel
docs: update CLAUDE.md with new model guide
refactor: extract session management to util
```

### Before Committing

1. Format code: `yarn format`
2. Check TypeScript: `tsc --noEmit`
3. Test manually with `yarn dev`
4. Update documentation if needed

## Resources

- [Baileys Documentation](https://github.com/WhiskeySockets/Baileys)
- [Config Documentation](docs/config-docs.md)
- [Deployment Guide](docs/deployment.md)
- [OpenAI API Reference](https://platform.openai.com/docs/api-reference)
- [Google Gemini Docs](https://ai.google.dev/gemini-api/docs)
- [Stability AI Docs](https://platform.stability.ai/docs)

## Quick Reference

### File Locations
- Entry point: `src/index.ts`
- Configuration: `src/whatsapp-ai.config.ts`
- Environment: `src/baileys/env.ts`
- Model routing: `src/baileys/handlers/message.ts`
- Prefix matching: `src/util/Util.ts`
- Message parsing: `src/baileys/hooks/useMessageParser.ts`

### Key Type Definitions
- AI Models enum: `src/types/AiModels.d.ts`
- Config interface: `src/types/Config.d.ts`
- Metadata types: `src/models/BaseAiModel.ts`

### Important Patterns
- Boolean env vars: `process.env.VAR === 'True'`
- Model instantiation: Conditional on `ENV.MODEL_ENABLED`
- Session management: Check existence before use
- Error handling: Always catch and call `handle('', error)`

---

**Last Updated**: 2025-11-22
**Bot Version**: 1.0.0
**Maintainer**: Zain-ul-Din

---
> Source: [Zain-ul-din/whatsapp-ai-bot](https://github.com/Zain-ul-din/whatsapp-ai-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
