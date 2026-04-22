## therapist-ai-assistant

> This document details the precise responsibilities of the main client component (`app/page.tsx`) in the v2.0 architecture. Its primary new role is to act as the master constructor for the API payload and the sole keeper of conversational state.

# SAD-2.1: Client-Side Payload Construction & State Management

## 1. Purpose
This document details the precise responsibilities of the main client component (`app/page.tsx`) in the v2.0 architecture. Its primary new role is to act as the master constructor for the API payload and the sole keeper of conversational state.

## 2. State Variables
- **`transcriptLog: TranscriptEntry[]`**  
    - *Type:* `Array<{ speaker: 'therapist' | 'client'; text: string; }>`  
    - *Description:* The canonical, unabridged, and ordered history of the entire conversation. It is the absolute source of truth. It is updated every time an utterance is assigned to a speaker.

- **`currentSummary: string`**  
    - *Type:* `string`  
    - *Description:* The most recent summary of the "old" part of the conversation. Initialized as an empty string (`""`). Updated only when a `newSummary` is received from the server’s response stream. Never modified directly by the client.

## 3. Core Logic: `handleActionClick(action)` Function
This function is triggered when a user clicks an AI action button (e.g., **"Paraphrase"**). It orchestrates the entire payload creation process.

**Constants:**  
    const MEMORY_BUFFER_SIZE = 6;

**Step-by-Step Execution Flow:**
1. **Capture staging text** via `takeTranscript()` and add to a temporary copy of `transcriptLog`.  
    
        const newLog = [...transcriptLog];
        const stagingText = takeTranscript().trim();
        if (stagingText) {
            newLog.push({ speaker: 'client', text: stagingText });
        }

2. **Create the Action Buffer:**  
    
        const buffer = newLog.slice(-MEMORY_BUFFER_SIZE);

3. **Create the "Simulated Reality":**  
    
        const messagesForAction = createActionMessages(buffer);
        // e.g., maps:
        // { speaker: 'client', text }    → { role: 'user', content: text }
        // { speaker: 'therapist', text } → { role: 'assistant', content: text }

4. **Create the Full Transcript:**  
    
        const fullTranscript = newLog
            .map(e => `${e.speaker.charAt(0).toUpperCase() + e.speaker.slice(1)}: ${e.text}`)
            .join('\n');

5. **Assemble and Send Payload:**  
    
        append(
            { role: 'user', content: '...' }, // placeholder UI message
            {
                body: {
                    action: action,
                    messagesForAction: messagesForAction,
                    fullTranscript: fullTranscript,
                    currentSummary: currentSummary
                }
            }
        );

6. **Update UI State:**  
    
        setTranscriptLog(newLog);

## 4. State Update Logic: `useEffect` for Summaries
Implement a `useEffect` hook to receive updated summaries from the server.

    
    useEffect(() => {
        const lastResponse = data[data.length - 1];
        if (lastResponse?.newSummary && lastResponse.newSummary !== currentSummary) {
            setCurrentSummary(lastResponse.newSummary);
        }
    }, [data]);

- **Dependency:** Watches the `data` array returned from the `useChat` hook.  
- **Logic:**  
    1. Inspect the last element of `data`.  
    2. If it contains a `newSummary` key different from `currentSummary`, call `setCurrentSummary(newSummary)`.  

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Carpentersbug41) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
