## quiz-generator

> A web-based trivia quiz app powered by Open Trivia DB. Built to practice Claude Code workflows involving MCP servers, skills, and subagents.

# Quiz Generator

A web-based trivia quiz app powered by Open Trivia DB. Built to practice Claude Code workflows involving MCP servers, skills, and subagents.

## Tech Stack

- **Frontend:** Plain HTML/CSS/JS (no framework)
- **Data source:** Open Trivia DB via a custom MCP server
- **AI workflows:** Claude Code skill for question design, subagent for adaptive difficulty

## Project Structure

```
.claude/
  skills/
    quiz-design/SKILL.md       # Skill: raw trivia → polished multiple-choice questions
  agents/
    difficulty-calibrator.md   # Subagent: adjusts question difficulty based on user performance
mcp/
  server.js                    # MCP server wrapping Open Trivia DB API
index.html                     # Quiz UI
style.css                      # Styles
app.js                         # Quiz logic and state management
```

## Components

### MCP Server (`mcp/server.js`)

Wraps the Open Trivia DB API (`https://opentdb.com/api.php`, no API key required). Provides tools for fetching trivia questions with configurable parameters (category, difficulty, amount, type).

**When to use the MCP server:**
- Fetching new questions for a quiz session
- Browsing available categories
- Retrieving questions filtered by difficulty or type (multiple-choice / true-false)
- Any time the app needs trivia data from Open Trivia DB

The MCP server is the *only* way to talk to Open Trivia DB. Do not call the API directly from the frontend or from other scripts.

### Quiz Design Skill (`.claude/skills/quiz-design/SKILL.md`)

Defines how to transform raw Open Trivia DB responses into well-formed multiple-choice questions.

**When to invoke the quiz-design skill:**
- After fetching raw questions from the MCP server, before presenting them to the user
- When the raw trivia data needs cleanup (HTML entities decoded, phrasing smoothed, answer options shuffled)
- When validating that questions meet quality standards: clear phrasing, no ambiguous "all of the above" answers, consistent difficulty within a single quiz, properly shuffled answer positions

The skill is invoked as a processing step between fetching and display. Every question shown to the user should pass through this skill's guidelines.

### Difficulty Calibrator Subagent (`.claude/agents/difficulty-calibrator.md`)

A subagent that analyzes the user's recent answer history and recommends the next question's difficulty.

**When to delegate to the difficulty-calibrator subagent:**
- Between questions, after the user answers but before fetching the next question
- The subagent receives the user's recent correct/incorrect answers and returns a recommendation: easier, same, or harder, along with a short reason
- Use this recommendation to set the `difficulty` parameter on the next MCP server call

**Do not** invoke the calibrator at quiz start (use a default medium difficulty) or if the user has no answer history yet.

## Workflow: One Quiz Question

1. **Fetch** — Call the MCP server to get a raw question (using difficulty from the calibrator if available)
2. **Design** — Apply the quiz-design skill to clean, shuffle, and validate the question
3. **Present** — Display the question in the UI
4. **Answer** — User selects an answer; record correct/incorrect
5. **Calibrate** — Delegate to the difficulty-calibrator subagent with the user's recent performance
6. **Repeat** — Return to step 1 with the calibrator's recommended difficulty

## Development Notes

- The frontend is static HTML/CSS/JS — no build step, no bundler
- Open Trivia DB returns HTML-encoded strings (e.g., `&quot;`); always decode before display
- Questions and answers from the API may need re-shuffling since the correct answer is always in a separate field from the incorrect ones
- The MCP server should handle rate limiting gracefully (Open Trivia DB has a soft limit; adding `&encode=url3986` to requests helps avoid encoding issues)

---
> Source: [TheDevPP/quiz-generator](https://github.com/TheDevPP/quiz-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
