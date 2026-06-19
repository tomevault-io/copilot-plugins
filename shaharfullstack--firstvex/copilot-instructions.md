## firstvex

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

NoteJsonrator is a Node.js application that generates comprehensive musical note data in JSON format. The project creates detailed information for all musical notes across multiple octaves (0-8), including:

- Physical properties (frequency, wavelength, MIDI numbers)
- Music theory data (scales, chords, circle of fifths positions)
- Notation data for different clefs
- VexFlow integration data for music notation rendering
- Enharmonic equivalents (including E#, B#, Fb, Cb)

## Architecture

### Core Components

**JavaScript Files (`js/` directory):**
- `note-generator.js`: Node.js application for generating musical note data
- `osme.js`: Large bundled library (41,717 lines) - appears to be music-related functionality

**Main Generator (`js/note-generator.js`):**
- Single-file Node.js application with comprehensive music theory calculations
- Generates 189 individual JSON files (one per note/enharmonic across 9 octaves)
- Creates an index.json file for easy lookup and navigation
- Includes validation system to catch music theory errors

**Rhythm Learning Game (`rhythm-learning-game.html`):**
- Standalone HTML file with VexFlow-based rhythm training game
- Three difficulty levels with 14 different rhythm patterns
- Interactive metronome, timing analysis, and progress tracking
- Web Audio API integration for precise audio feedback

**Generated Output (`notes_json/` directory):**
- Individual note files named by note and octave (e.g., `A4.json`, `C#3.json`)
- Each file contains ~200 lines of structured musical data
- Index file provides metadata and quick reference to all generated notes

### Key Features

- **Complete enharmonic support**: Includes all valid music theory equivalents like E# = F, B# = C, Fb = E, Cb = B
- **Music theory accuracy**: Proper circle of fifths positioning, scale generation with consistent notation
- **Batch processing**: Efficient generation of large datasets with progress tracking
- **Validation system**: Built-in checks for music theory consistency and calculation accuracy

## Development Commands

**Generate all note data:**
```bash
node js/note-generator.js
```

**Module usage in other projects:**
```javascript
const { generateAllNotes, generateNoteData, CONFIG } = require('./js/note-generator.js');
```

**Run rhythm learning game:**
- Open `rhythm-learning-game.html` in any modern web browser
- No build or installation required - fully self-contained

## Configuration

**Note Generator (`js/note-generator.js`):**
The `CONFIG` object controls:
- Output directory (`../notes_json`)
- Octave range (0-8)
- Enharmonic inclusion (enabled)
- Index file creation (enabled)
- Batch processing size (50 notes)
- JSON formatting (pretty print enabled)

**Rhythm Learning Game:**
- Adjustable tempo (60-140 BPM)
- Three difficulty levels with configurable patterns
- Timing tolerance and scoring parameters
- Audio/visual feedback settings

## Data Structure

Each generated note file contains:
- **Basic identification**: id, name, note, octave
- **Physics**: frequency, wavelength, period, midiNumber
- **Music theory**: scales (major/minor/chromatic), chords, circle of fifths position
- **Notation**: staff positions for treble/bass/alto clefs
- **VexFlow**: ready-to-use data for web music notation
- **Harmonics**: first 8 harmonic frequencies with interval names

## Common Tasks

**Note Generator:**
- **Modifying note range**: Edit `CONFIG.START_OCTAVE` and `CONFIG.END_OCTAVE` in `js/note-generator.js`
- **Adding new chord types**: Extend `CHORD_INTERVALS` object
- **Adding new scales**: Extend `SCALES` object  
- **Changing output format**: Modify `CONFIG.PRETTY_PRINT` or add custom formatting in `processBatch()`

**Rhythm Game:**
- **Adding new patterns**: Extend pattern arrays in difficulty level objects
- **Adjusting difficulty**: Modify timing tolerance, tempo ranges, or progression requirements
- **Customizing UI**: Edit CSS classes and styling in the HTML file
- **Adding new features**: Extend the `RhythmGame` class methods

## Music Theory Implementation Notes

- Circle of fifths positions are accurate for all notes including enharmonics
- Scale generation uses appropriate enharmonic spellings (flat roots use flat scales)
- Validation system catches common errors like inconsistent notation
- Special handling for edge cases like B# = C (next octave) and Cb = B (previous octave)

---
> Source: [ShaharFullStack/firstVex](https://github.com/ShaharFullStack/firstVex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
