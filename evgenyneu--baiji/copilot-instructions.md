## baiji

> - The program is made to convert large ebook text  (.txt or .epub) to audiobook (mp3).

## This are important features of the program

- The program is made to convert large ebook text  (.txt or .epub) to audiobook (mp3).
- Since it can take a lot of time to do TTS, the user should be able to cancel (Ctrl-C) but when they start again it should continue where it stopped and not do it again from the start.
- For .epub file, it should create multiple mp3 files, one for each chapter, instead of single large mp3.


## Development plan

Here's the concise updated plan:

1. **Directory Structure:**
   - Root: `audio/BOOK_TITLE/`
     - BOOK_TITLE; comes from the files name (of .txt or .epub wihtout extension, lowercased, undescores replaces spaces).
     - `progress.json`: Tracks chapter and the text segment index (we split text by newlines to allow resuming where we left).
     - `chunks/`: Stores individual `.wav` chunks (e.g., `chunk_0001.wav`).
     - resulting mp3 files are places directly in audio/BOOK_TITLE/ (e.g., `chapter_0001.mp3`).

2. **Process Flow:**
   - **Input Handling:**
     - Parse command-line arguments for input file path.
     - Detect if input is `.txt` or `.epub`.

   - **Text Extraction:**
     - For `.txt`, read the file directly.
     - For `.epub`, extract text chapter by chapter.

   - **TTS Processing:**
     - Generate audio chunks using Kokoro TTS.
     - Save each chunk as a `.wav` file in `audio/chunks/`.
     - Track progress in `progress.json`.

   - **Combining and Converting:**
     - Once all chunks for a chapter are generated, combine them into a single `.wav` file.
     - Convert the combined `.wav` to `.mp3` and save in `audio/chapters/`.
     - Delete the processed `.wav` chunks.

   - **Cancellation and Resumption:**
     - Allow the user to cancel the process.
     - On cancellation, save the current chapter and segment index to `progress.json`.
     - On restart, read `progress.json` and resume from the last processed point.

This plan ensures efficient memory usage, allows for cancellation and resumption, and keeps your project organized.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/evgenyneu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/evgenyneu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
