## quiz-app

> A web-based quiz application that supports multiple quiz types (regular multiple-choice and code pattern quizzes) with interactive features including confetti celebrations, quiz caching, and automatic question randomization.

# Quiz App - Project Documentation

## Project Overview
A web-based quiz application that supports multiple quiz types (regular multiple-choice and code pattern quizzes) with interactive features including confetti celebrations, quiz caching, and automatic question randomization.

## Tech Stack
- **Frontend**: Vanilla JavaScript, HTML5, CSS3
- **UI Libraries**:
  - React (for geological quizzes)
  - Canvas-confetti for celebrations
  - KaTeX for math rendering
  - Markdown-it for rich text formatting
- **Backend**: PHP for quiz caching/storage
- **Storage**: Server-side storage via PHP API

## File Structure
```
quiz-app/
├── index.html              # Main HTML structure
├── script.js               # Core quiz logic and functionality
├── styles.css              # All styling
├── geological-quiz.js      # Geological timescale quiz (React)
├── geological-quiz-v2.js   # Cenozoic era quiz (React)
├── api/
│   ├── quizzes.php        # Quiz storage/retrieval API
│   └── test_write.php     # API testing
├── quiz_files/             # Active quiz cache (auto-created)
├── quiz_files_archive/     # Archived old quizzes (auto-created)
├── favicon.ico
├── logo.png
└── deploy.sh              # Deployment script
```

## Key Features

### Quiz Types
1. **Regular Quiz**: Multiple-choice questions with auto-randomized order
2. **Code Pattern Quiz**: Fill-in-the-blank coding challenges with similarity scoring
3. **Geological Quizzes**: Interactive timeline-based quizzes (preset)

### Core Functionality
- **Auto-randomization**: Questions automatically randomized on each quiz start
- **Answer randomization**: Optional randomization of answer choices (correct answer always starts at index 0 in JSON)
- **Quiz caching**: Recent quizzes stored server-side with duplicate detection
- **Question navigation**: Jump between visited questions via dropdown
- **Progress tracking**: Visual indicators for current question and score
- **Quiz review**: Detailed review showing correct/incorrect answers after completion
- **Confetti celebrations**: Triggered on correct answers (configurable intensity)
- **Math support**: KaTeX rendering for mathematical expressions
- **Markdown support**: Full markdown formatting including code blocks

### JSON Format

#### Regular Quiz
```json
{
  "title": "Sample Quiz",
  "questions": [
    {
      "question": "What is the integral of $2x$?",
      "reasoning": "Using the power rule for integration, $\\int x^n dx = \\frac{x^{n+1}}{n+1} + C$, we get $\\int 2x dx = x^2 + C$.",
      "options": [
        "$x^2 + C$",
        "$2x^2$",
        "$x + C$",
        "$2$"
      ],
      "correctAnswer": 0
    }
  ]
}
```

**Important**:
- The correct answer should **always** be at index 0 in the options array
- This allows AI to generate better distractors without bias toward longer/more detailed answers
- The app will automatically randomize answer order if the user enables that option

#### Code Pattern Quiz
```json
{
  "title": "Design Patterns Quiz",
  "questions": [
    {
      "question": "Complete the Observer Pattern's update method:",
      "patternName": "Observer Pattern",
      "codeContext": "public class MessageProcessorObserver implements Observer {..}",
      "missingCode": {
        "beforePrompt": "public void update(Observable observable) {",
        "solution": "MessageProcessor messageProcessor = (MessageProcessor)observable;",
        "afterPrompt": "}"
      }
    }
  ]
}
```

## AI Quiz Generation Guidance

When generating quizzes for this app, follow these guidelines:

1. **Answer Ordering**: Place the correct answer first (index 0) in the options array. The app will randomize the order when displayed.

2. **Answer Length Consistency**: Make all options approximately the same length. A common tell for correct answers is excessive detail.

3. **Question Quality**: Ask complex, thoughtful questions that test understanding, not just memorization.

4. **Reasoning**: Include a reasoning field that explains why the correct answer is correct. This helps users understand the concept after answering. The reasoning should be clear, educational, and use the same formatting options (KaTeX and markdown) as questions.

5. **Math Rendering**: Use KaTeX syntax for mathematical expressions:
   - Inline math: `$expression$`
   - Block math: `$$expression$$`
   - Example: `$\\frac{d}{dx}(x^2) = 2x$`

6. **Markdown Formatting**: Use markdown for better formatting:
   - Code blocks: ` ```language\ncode\n``` `
   - Bold: `**text**`
   - Italic: `*text*`
   - Lists, links, etc.

### Example AI Prompt for Quiz Generation
```
Create a quiz following this JSON format. Note that:
- Multiple choice questions should have the correct answer first (index 0)
- The app will randomize the order later
- Make all answer options similar in length (avoid length bias)
- Ask good, complex questions that test deep understanding
- Include a reasoning field explaining why the correct answer is right
- Use KaTeX for math: $expression$ (inline) or $$expression$$ (block)
- Use markdown for formatting: **bold**, *italic*, code blocks, etc.

Follow this JSON format: [paste the regular quiz JSON example]
```

## Development Notes

### Randomization Logic
- Question randomization: Uses Fisher-Yates shuffle algorithm (script.js:59)
- Happens automatically on quiz start
- Original question order preserved for reference

### Confetti System
- Triggered on correct answers (script.js:175)
- Also triggered on high similarity scores for code quizzes (≥90%)
- Configurable intensity: low, medium, high, extreme
- Emits from sides or falls down (not from center)

### Quiz Caching
- Quizzes cached server-side via PHP API (script.js:825)
- Includes duplicate detection based on quiz content
- Recent quizzes displayed on start screen (up to 10)
- Cache duration: 90 days (3 months)
- Old quizzes are archived (moved to `quiz_files_archive/`) instead of deleted for recovery

### Question Navigation
- Users can only navigate to previously visited questions
- Prevents skipping ahead without answering
- Answers preserved when navigating back
- Implemented via dropdown in quiz header (script.js:549)

## Browser Compatibility
- Modern browsers (Chrome, Firefox, Safari, Edge)
- Requires JavaScript enabled
- Canvas API support needed for confetti
- Responsive design for mobile devices

## Future Enhancements
- Export quiz results as PDF
- Timer mode for timed quizzes
- Leaderboard system
- Quiz categories and tagging
- Import quizzes from various formats
- Spaced repetition mode

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KonnorKooi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
