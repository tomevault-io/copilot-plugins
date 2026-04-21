## casescriptai

> - Max 150 lines per file


# CaseScriptAI Code Standards

## File Size
- Max 150 lines per file
- If exceeded, split into smaller modules
- Example: `AudioService.ts` → split into `recorder.ts`, `converter.ts`, `chunker.ts`

## Naming Conventions
- **Files**: kebab-case → `use-audio-picker.ts`, `audio-recorder.ts`
- **Functions/Variables**: camelCase → `pickAudio`, `loadLocalFile`
- **Components**: PascalCase → `RecordButton`, `PipelineSteps`
- **Constants**: SCREAMING_SNAKE_CASE → `MAX_CHUNK_SIZE`, `WHISPER_MODEL_PATH`
- **Types/Interfaces**: PascalCase → `SessionData`, `AudioMetadata`

## Function Declaration
- Always use arrow functions for exports
- Always use arrow functions for component functions
- Always type parameters and return values

```typescript
// ✅ Correct
export const pickAudio = async (): Promise<string | null> => {
  // ...
};

// ❌ Wrong
export function pickAudio() {
  // ...
}
Component Structure
typescript
// 1. Imports
import { useState } from 'react';

// 2. Types
type Props = { ... };

// 3. Component
export const MyComponent = ({ prop }: Props) => {
  // 4. Hooks
  const [state, setState] = useState();
  
  // 5. Functions
  const handleAction = () => { ... };
  
  // 6. JSX
  return <View>...</View>;
};
Service Structure
typescript
// services/audio/recorder.ts
export const startRecording = async (): Promise<void> => {
  // Max 30 lines
};

export const stopRecording = async (): Promise<string> => {
  // Max 30 lines
};

Folder Structure Rules
text
src/
├── app/          → Screens only, no logic
├── components/   → Reusable UI only
├── hooks/        → Reusable logic, call services
├── services/     → Pure business logic, no UI
├── stores/       → Zustand state only
├── types/        → TypeScript types/interfaces
├── constants/    → Config, model paths, prompts
└── utils/        → Pure functions (device-tier, wer)
Import Order
typescript
// 1. React/React Native
import { useState } from 'react';
import { View } from 'react-native';

// 2. External libraries
import * as DocumentPicker from 'expo-document-picker';

// 3. Internal - absolute imports
import { AudioRecorder } from '@/services/audio/recorder';
import { useSessionStore } from '@/stores/session-store';

// 4. Types
import type { AudioMetadata } from '@/types/session';
Hooks Rules
Prefix with use → useAudioPicker, usePipeline

Return object, not array (unless 2 values max)

Always arrow functions

typescript
// ✅ Correct
export const useAudioPicker = () => {
  return { pickAudio, isLoading, error };
};

// ❌ Wrong (array with 3+ values)
export const useAudioPicker = () => {
  return [pickAudio, isLoading, error];
};
Error Handling
Always try/catch in services

Always return typed errors, never throw

typescript
type Result<T> = 
  | { success: true; data: T }
  | { success: false; error: string };

export const loadAudio = async (): Promise<Result<AudioMetadata>> => {
  try {
    // ...
    return { success: true, data };
  } catch (err) {
    const message = err instanceof Error ? err.message : 'Unknown error';
  return { success: false, error: message };
  }
};
Comments
No obvious comments (// Set state)

Only explain "why", never "what"

Max 1 comment per 20 lines

typescript
// ✅ Correct
// Force mono to prevent Whisper channel mismatch
const channels = 1;

// ❌ Wrong
// Set channels to 1
const channels = 1;

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GourabSanyal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
