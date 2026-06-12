## personactionobject

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PersonActionObject is a command-line quiz application for practicing a Person-Action-Object (PAO) memory system. The PAO system maps 2-digit numbers (00-99) to a person, action, and object to create memorable mental images for memorizing long numbers.

The application reads PAO data from an external source file and provides two quiz modes:
1. **Given Quiz**: Given one attribute (number/person/action/object), recall another attribute
2. **Number Quiz**: Given a 6-digit number, describe the mental image (person from first 2 digits, action from middle 2, object from last 2)

## Build and Run Commands

This is a Maven project using Java 23.

```bash
# Clean and compile
mvn clean compile

# Run the application (requires compiled classes)
mvn exec:java -Dexec.mainClass="com.spyder.pao.Main"

# Alternative: Run from compiled classes
java -cp target/classes com.spyder.pao.Main

# Package into JAR
mvn package
```

## Application Architecture

### Data Flow
1. **Startup**: `Main.java` → `DataSource.createAndLoad()` → loads `data.txt` → formats and rewrites it
2. **CLI Loop**: `CLIRunner.begin()` → loads `state.properties` → enters command loop
3. **Quiz Execution**: User triggers quiz → `GivenQuiz` or `NumberQuiz` handles quiz logic

### Key Components

**DataSource** (`DataSource.java`)
- Loads PAO entries from `src/main/resources/data.txt`
- Pipe-delimited format: `number|person|action|object|alt_numbers|alt_people|alt_actions|alt_objects`
- On startup, reformats the data file with equal spacing for readability
- Filters entries based on quiz configuration (min/max range)

**CLIRunner** (`CLIRunner.java`)
- Main command loop that processes user input
- Manages quiz configuration (range, given/answer types, quiz mode)
- Persists state to `src/main/resources/state.properties` before starting quiz
- Supports comma-separated commands for convenience

**GivenQuiz** (`GivenQuiz.java`)
- Quiz mode where user is given one attribute and must recall another
- Supports RANDOM mode for both given and answer types
- Flexible answer validation: case-insensitive, allows alternate answers, tolerates one wrong word for longer answers
- Shuffles entries after each complete cycle
- Tracks statistics and checks timer expiration after each answer
- Displays "X correct out of Y total" when timer expires

**NumberQuiz** (`NumberQuiz.java`)
- Quiz mode for 6-digit numbers
- Randomly selects 3 entries (person, action, object) to form a scenario
- No automated validation - displays correct answer for user review
- Tracks question count when timer is active

**QuizStatistics** (`model/QuizStatistics.java`)
- Tracks quiz performance: total answers and correct answers
- Manages timer state and expiration checking
- For GivenQuiz: tracks correct/total and displays accuracy summary
- For NumberQuiz: tracks total questions answered (no automated correctness)

**PaoEntry** (`model/PaoEntry.java`)
- Data model with primary and alternate values for each attribute
- Lazy-loads alternate value lists from comma-separated strings
- Provides unified access via `getValue()` and `getAllByType()` methods

### State Management

Quiz configuration persists between sessions in `state.properties`:
- `FROM`: Minimum entry number
- `TO`: Maximum entry number
- `GIVEN`: Given entry type (NUMBER/PERSON/ACTION/OBJECT/RANDOM)
- `ANSWER`: Answer entry type (NUMBER/PERSON/ACTION/OBJECT/RANDOM)
- `QUIZ`: Quiz mode (GIVEN/NUMBER)
- `TIMER`: Timer duration in minutes (0 = off)

### Command System

Commands are defined in `Command.java` enum with trigger words:
- Commands can have multiple triggers (e.g., "begin", "b", "start")
- Map-based lookup for O(1) command resolution
- Commands can be typed at the prompt or as quiz answers to exit

Available commands:
- `TIMER [minutes|off]`: Sets a countdown timer for quiz sessions
- `FROM [number]`, `TO [number]`: Set quiz range
- `GIVEN [type]`, `ANSWER [type]`: Configure quiz question/answer types
- `QUIZ [GIVEN|NUMBER]`: Select quiz mode
- `BEGIN/START/B`: Start the configured quiz
- `LIST`: Display entries in the current range
- `HELP`: Show command reference
- `QUIT/EXIT`: Quit quiz or exit application

### Answer Validation Logic

The flexible answer checker in `GivenQuiz.validateAnswer()`:
1. Compares user input against all alternate answers
2. Tokenizes both user input and valid answers
3. Counts matching words and finds best match
4. Allows 1 wrong word for answers > 2 words long
5. Tracks whether answer is "exactly correct" vs "close enough"

## Data File Format

`data.txt` structure (pipe-delimited):
```
00 | zeus           | throwing lightning   | lightning bolt    |    | jupiter                    |               | javelin
01 | athena         | forging              | spear             |    | pallas athena              | smithing      |
```

- Columns 0-3: Primary number, person, action, object
- Columns 4-7: Alternate values (comma-separated)
- Empty alternate columns are allowed
- File is auto-formatted on startup with equal spacing
- Quotes are stripped (for easy copy/paste from Google Sheets)

## Dependencies

- Java 23 (using preview features like switch expressions)
- Lombok 1.18.38 (for `@Data`, `@SneakyThrows`)
- Maven compiler plugin configured for annotation processing

---
> Source: [xarnthehero/PersonActionObject](https://github.com/xarnthehero/PersonActionObject) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
