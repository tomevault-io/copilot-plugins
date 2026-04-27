## syncdocs

> Create a script that will check the content of /Users/Shared/Documents (let's call it SOURC).

# Objective
Create a script that will check the content of /Users/Shared/Documents (let's call it SOURC).
For each document found move it to /Users/OpenClaw/.openclaw/documents (let's call the directory TARGET).

## Preverse a directory structure
In the initial document is following a directory structure such as /Users/Shared/Documents/a/b/.../c/doc.pdf then the a/b/.../c/ directory structure needs to be preserve. Do not remove any subdirectory, the entire directory stucture preservation is important to preserve the classification structure of the documents.

## Transcription
If a document is an audio file, we should not move it immediately to TARGET. We should create a SOURCE/transcriptions/a/b/.../c/ directory if it does not exist and move the file to transcriptions.
We should then launch a subprocess to transcribe the file in place (in the same directory) and replace its file extension by .txt.
Once the transcription is completed, we should remove the audio file and move the .txt file into TARGET/a/b/.../c/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smarcsoft) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
