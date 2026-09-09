## deepgram-toml

> deepgram.toml Requirements

# deepgram.toml Requirements

## File Format
```toml
[meta]
title = "Flask Agent Starter" # title of the app
description = "Get started using Deepgram's Voice Agent with this Flask demo app" # description of the app
author = "Deepgram DX Team <devrel@deepgram.com> (https://developers.deepgram.com)" # contact details of the DX team
useCase = "Agent" # or TTS, Agent, TextIntel,
language = "Python" # the language used in the main file
framework = "Flask" # short key for the framework, or remove if null
sdk = "4.0.0" # version of the SDK, or remove if no SDK used

[build]
command = "pipenv install -r requirements.txt" # the command to set up the app to run it

[config]
DEEPGRAM_API_KEY="YOUR_DEEPGRAM_API_KEY" python app.py

[post-build]
command = "python app.py" # the command the user will run to start the app
```

---
> Source: [deepgram-starters/flask-voice-agent](https://github.com/deepgram-starters/flask-voice-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
