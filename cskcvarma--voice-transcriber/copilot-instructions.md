## voice-transcriber

> This project transcribes audio files into text, converts the text into chunks, and then tags the chunk with relevant information.

# VOICE TRANSCRIBER
This project transcribes audio files into text, converts the text into chunks, and then tags the chunk with relevant information.

## CODING GUIDELINES
- Check the docs folder for the documentation of the file you are working on.
- Update the documentation where necessary.
- Always follow the PEP8 style guide.
- Follow all the ruff conventions.
- After each update, run the tests, linting and typechecking to make sure everything is passing.
- Update Readme.md where relevant.
- Maintain a docs folder with the documentation for the project.

## DOCUMENTATION GUIDELINES

## PROJECT STRUCTURE
-- src/voice_transcriber/
    -- main.py: The main entry point for the project.
    -- transcribe.py: The module for transcribing audio files.
    -- chunk.py: The module for chunking the transcript.
    -- tag.py: The module for tagging the chunks.
    -- utils.py: The module for utility functions.

## DOCS STRUCTURE
-- docs/
    -- voice_transcriber/
        -- index.md: The main index page for the project.
        -- transcribe.md: The documentation for the transcribe module.
        -- chunk.md: The documentation for the chunk module.
        -- tag.md: The documentation for the tag module.
        -- utils.md: The documentation for the utils module.


## TESTS, LINTING AND TYPECHECKING
RUN: uv run nox
Make sure all steps are passing.
Update the code to fix the issues.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cskcvarma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
