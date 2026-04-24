## baby-einstein

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Baby Einstein is a brain-inspired AI system called "Albert" that simulates human consciousness through modular brain region processing. The system routes inputs through different "brain regions" (amygdala, prefrontal cortex, sensory cortex, etc.) using a fine-tuned Thalamus model.

## Key Architecture Components

**Thalamus System** (`albert/thalamus/thalamus.py`): Core routing engine that analyzes inputs and directs them to appropriate brain regions. Uses a fine-tuned model located at `albert/thalamus/model/thalamus_finetuned/`.

**Chat Engine** (`albert/chat_engine.py`): Text processing wrapper that currently implements the Thalamus model. ACC engine is planned but not yet implemented.

**Sight Engine** (`albert/sight_engine.py`): Image processing using ViT-GPT2 model for image captioning.

**Client System** (`albert/clients/`): HTTP clients for different engines - ThalamusClient, VisionClient, and planned ACCClient.

**Prompt Configuration** (`albert/prompts/prompts.yaml`): Defines brain region schemas, processing rules, and output formats for the Thalamus system.

## Development Commands

**Setup**: `./install.sh` or `pip install -r requirements.txt`

**Run Server**: `python albert/main.py` (starts FastAPI server on localhost:8000)

**Testing**: Use `notebook.ipynb` for experimentation and testing

## API Endpoints

- `/chat/completions`: Text processing through brain models
- `/sight/`: Image processing and captioning

## Important Notes

- ACC engine uses fallback logic when the fine-tuned model produces invalid JSON responses
- Thalamus model has been optimized to extract only new tokens and avoid input echoing
- Both engines include comprehensive error handling and logging for debugging
- Image processing automatically resizes large images for efficiency
- The system expects structured prompts defined in prompts.yaml for proper brain region routing
- Server logs provide detailed information about model responses and fallback usage

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seanpoyner) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
