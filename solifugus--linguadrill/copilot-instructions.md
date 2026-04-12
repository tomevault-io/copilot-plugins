## linguadrill

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LinguaDrill is a language learning application designed to help users achieve minimal fluency by learning the 200 most common words and their grammatical usage through iterative drilling.

## Architecture

### Frontend (PWA)
- **Technology**: Vanilla HTML/CSS/JS (no frameworks)
- **Storage**: All user progress stored in client-side localStorage (no server-side user data)
- **Structure**: Responsive PWA with tab-based interface (Vocabulary, Grammar, Dialog)

### Backend (Node.js)
- **Technology**: Node.js server
- **Storage**: File-based (no database)
- **Data Location**: Language datasets stored as JSON files in `~/languages/` folder
- **Languages**: Configured via `learner-languages.txt` (one language per line)

### Data Structure

Each language JSON file contains 20 iterations with:
- **Alphabet**: Characters, sounds, and names
- **Vocabulary**: 10 new words per iteration (cumulative, normative form for case languages)
- **Grammars**: Example phrases/sentences demonstrating grammar rules using only iteration vocabulary
- **Dialog**: Conversation using all vocabulary and grammar from the iteration

## Code Style Requirements

**CRITICAL**: This project has strict JavaScript conventions:

- **No TypeScript** - Use plain JavaScript only
- **No ES6 Classes** - Use object literal pattern instead:

```javascript
myobj = {
    attrib1: "",
    attrib2: ""
}
myobj.method1 = function(p1, p2) {
    // code here
}
myobj.method2 = function(p1, p2) {
    // code here
}
```

## Learning Flow

1. **Onboarding**: User creates account (email, password, native language, target language)
2. **Alphabet Mastery**: Drill each letter 3 times correctly before proceeding
3. **Iteration Learning**: User completes all three tabs (Vocabulary, Grammar, Dialog) before advancing
4. **Mastery Rules**:
   - New words: Correct 3 times consecutively
   - Previously mastered words: Correct 1 time (resets to 3 if incorrect)
   - Grammar/Dialog: Always correct 3 times consecutively
5. **UI**: Random multiple-choice questions for all drills

## Key Components

### Backend Utility
Generate language datasets with vocabulary lists (200 most common words split into 20 iterations of 10 words each), grammar examples, and dialogs for each iteration, output as JSON in `~/languages/` directory.

### Frontend Tabs
- **Vocabulary Tab**: Flash cards with random ordering, tracks per-word mastery
- **Grammar Tab**: Grammar rule examples with explanations
- **Dialog Tab**: Full dialog scenarios

### Navigation
Hamburger menu for password changes and language switching.

## Development Notes

- User progress persists in browser localStorage only
- Server serves static language data files
- All drilling uses random multiple-choice format
- Tab state should persist if user switches away and returns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Solifugus)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Solifugus)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
