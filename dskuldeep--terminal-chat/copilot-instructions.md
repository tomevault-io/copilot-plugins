## terminal-chat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Terminal Chat is a peer-to-peer (P2P) encrypted chat application with a terminal aesthetic. It uses WebRTC (via PeerJS) for direct browser-to-browser communication and TweetNaCl for end-to-end encryption.

**Key Principle**: This is a serverless P2P application - all messages flow directly between peers. The only servers involved are:
- PeerJS signaling server (for initial connection setup)
- Google's STUN servers (for NAT traversal)

## Development Commands

### Start Development Server
```bash
npm run dev
```
Runs Vite dev server on `http://localhost:3000`

### Build for Production
```bash
npm run build
```
Compiles TypeScript and bundles with Vite to `dist/` directory

### Preview Production Build
```bash
npm run preview
```

## Architecture

### Core Layers

1. **Encryption Layer** (`src/lib/encryption.ts`)
   - Handles all E2E encryption using TweetNaCl
   - Asymmetric encryption (NaCl box) for 1-to-1 messages
   - Symmetric encryption (NaCl secretbox) for group messages
   - Key generation, storage, and retrieval from localStorage

2. **P2P Connection Layer** (`src/lib/peerConnection.ts`)
   - Manages WebRTC connections via PeerJS
   - Handles peer discovery and connection establishment
   - Routes encrypted messages between peers
   - Manages public key exchange between peers
   - **Critical**: Always encrypts messages before sending, decrypts on receipt

3. **State Management** (`src/store/chatStore.ts`)
   - Zustand store for global state
   - Manages: peers, messages, active chats, user identity
   - Persists messages and keys to localStorage
   - **Important**: Messages are stored encrypted in state, decrypted for display

4. **UI Components** (`src/components/Terminal.tsx`)
   - Single main component with sub-components
   - Terminal-aesthetic design (monospace fonts, dark theme)
   - Handles user interactions and displays state

### Data Flow

```
User Input → UI Component → Store Action → Encryption → PeerConnection → WebRTC → Remote Peer
                                                                                      ↓
User Display ← UI Component ← Store Update ← Decryption ← PeerConnection ← WebRTC ←
```

### Key Data Structures

**Message**:
```typescript
{
  id: string;           // UUID
  from: string;         // Peer ID
  to: string;           // Peer ID
  content: string;      // Encrypted content (or decrypted after processing)
  timestamp: number;
  type: 'text' | 'image' | 'video' | 'audio' | 'file';
  encrypted: boolean;
  fileName?: string;    // For file messages
  fileSize?: number;
  mimeType?: string;
}
```

**Peer**:
```typescript
{
  id: string;                              // PeerJS ID
  publicKey: string;                       // Base64 encoded public key
  username: string;
  status: 'online' | 'offline' | 'connecting';
  lastSeen: number;
}
```

## Important Patterns

### Adding New Message Types

1. Update `MessageType` union in `src/types/index.ts`
2. Add handler in `PeerConnectionManager.handleIncomingData()`
3. Update UI in `Terminal.tsx` to render new type
4. Ensure encryption/decryption is applied

### Adding New Features

**For encrypted features**:
- Always use `encryption.encrypt()` before sending
- Always use `encryption.decrypt()` when receiving
- Store encrypted data over the wire, decrypt for display

**For P2P features**:
- Add message type to `MessageType` union
- Implement handler in `peerConnection.ts`
- Update store actions in `chatStore.ts`
- Update UI components

### LocalStorage Usage

This app uses localStorage for:
- `terminal-chat-keys`: User's encryption keypair (NEVER lose this)
- `terminal-chat-username`: User's display name
- `terminal-chat-messages`: Encrypted message history

**Critical**: Keys stored in localStorage are the user's identity. Clearing localStorage = losing identity.

## Security Considerations

### When Adding Features

1. **Never send unencrypted data** - All message content must be encrypted
2. **Validate peer public keys** - Always check public key exists before encrypting
3. **Use random nonces** - Encryption library handles this, don't override
4. **Don't log sensitive data** - Avoid console.log of decrypted messages or keys

### Encryption Flow

**Sending**:
```typescript
const encrypted = encryption.encrypt(message, recipientPublicKey);
connection.send({ type: 'text', content: encrypted });
```

**Receiving**:
```typescript
const decrypted = encryption.decrypt(data.content, senderPublicKey);
// Use decrypted content
```

## Common Tasks

### Adding a New UI Component

1. Create component in `src/components/`
2. Import and use in `Terminal.tsx`
3. Add styles to component's `.css` file
4. Use CSS variables from `src/index.css` for theming

### Modifying Encryption

⚠️ **Warning**: Changing encryption breaks compatibility with existing users

If you must modify encryption:
1. Update `E2EEncryption` class in `src/lib/encryption.ts`
2. Consider versioning the encryption scheme
3. Handle backward compatibility

### Changing PeerJS Configuration

Edit `src/lib/peerConnection.ts` in the `initialize()` method:
```typescript
this.peer = new Peer({
  config: {
    iceServers: [ /* your STUN/TURN servers */ ],
  },
});
```

## Debugging

### Connection Issues

Check browser console for:
- `Peer error:` - PeerJS initialization errors
- `Connection error:` - WebRTC connection errors
- `No public key for peer:` - Key exchange failed

### Encryption Issues

Common errors:
- `Encryption failed` - Invalid recipient public key
- `Decryption failed` - Wrong key, corrupted message, or invalid sender key

### State Issues

Use React DevTools to inspect Zustand store state:
- Check `peers` Map for connected peers
- Check `messages` array for message history
- Verify `activeChat` matches selected peer

## File Organization

- `src/lib/` - Core business logic (no React)
- `src/components/` - React UI components
- `src/store/` - Zustand state management
- `src/types/` - TypeScript type definitions
- `src/*.tsx` - App entry points
- `src/*.css` - Styles

## Terminal Aesthetic Guidelines

When modifying UI:
- Use monospace font (JetBrains Mono)
- Use CSS variables from `:root` in `index.css`
- Follow dark theme (green/blue accents on dark background)
- Keep animations subtle (fade-in, slide-in)
- Use terminal-like prompts ($ symbol, etc.)

## Testing Locally

To test P2P functionality:
1. Run `npm run dev`
2. Open two browser windows (or different browsers)
3. Copy Peer ID from first window
4. Connect to it from second window
5. Send messages between windows

**Note**: Both windows must be open simultaneously for P2P to work.

## Known Limitations

- No offline message queue (both peers must be online)
- File size limited by browser memory
- May need TURN server for restrictive NATs (~10-15% of connections)
- Message history only stored locally
- No multi-device sync

## Future Enhancements

When implementing:
- **Video calls**: Add MediaStream handling to `peerConnection.ts`
- **Group chats**: Use symmetric encryption, distribute keys via encrypted messages
- **Voice notes**: Add audio recording, convert to base64, send as file
- **Read receipts**: Add new message type with acknowledgments

---
> Source: [dskuldeep/terminal-chat](https://github.com/dskuldeep/terminal-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
