## prompt-engineer

> This file provides guidance to AI assistants when working with the content in this repository.

# GEMINI.md

This file provides guidance to AI assistants when working with the content in this repository.

## Project Overview

This repository is a curated collection of high-quality, structured prompts for various AI-powered generation tasks. It serves as a resource library for users looking to leverage AI in fields such as content creation, design, development, and research. The prompts are designed to be modular, customizable, and optimized for practical application.

## Project Structure & Prompt Categories

The repository is organized into categories based on the target application. Each directory contains one or more Markdown files, with each file representing a specific prompt template.

-   **`article/`**: Prompts for generating long-form narrative articles with a natural, human-like conversational style.
-   **`book/`**: Prompts for typesetting and generating visually structured book layouts from raw content, often integrating web technologies for previews.
-   **`deep_research/`**: Tools for constructing comprehensive prompts for in-depth research on a given topic, using a collaborative worksheet model.
-   **`infomation_graphics/`**: Prompts for transforming content into visually engaging infographics, including static and animated HTML formats.
-   **`project/`**: Assistant prompts designed to help users structure and define project proposals or summaries in a systematic way.
-   **`slides_prompts/`**: Prompts for creating professional presentation slides, ranging from static HTML to interactive single-page applications with specific branding.
-   **`subtitle-extract/`**: Utilities for converting spoken content into the industry-standard SRT subtitle format.
-   **`vibe_coding/`**: Structured instructions and rule sets for guiding AI in complex software development tasks, such as scaffolding applications with specific frameworks and authentication.
-   **`website/`**: A collection of prompts for generating complete front-end code for various types of websites, including marketing and educational pages, with a focus on modern UI/UX and responsive design.

## Guiding Principles

-   **Modularity and Structure**: Prompts are designed with clear, organized sections, making them easy to understand, customize, and extend.
-   **Practical Application**: Each prompt is built for a real-world use case, aiming to produce high-quality, actionable results.
-   **Best Practices Integration**: Prompts often incorporate industry best practices, such as Responsive Web Design (RWD), Web Content Accessibility Guidelines (WCAG), and modern development patterns.

## How to Use the Prompts

1.  **Browse**: Navigate the directories to find a prompt category that matches your needs.
2.  **Select**: Choose a specific Markdown (`.md`) file for your task.
3.  **Copy & Customize**: Copy the prompt template into your preferred AI platform (e.g., Google Gemini, ChatGPT, Claude).
4.  **Input**: Follow the instructions within the prompt to fill in your specific content, data, or requirements.
5.  **Generate & Iterate**: Execute the generation and refine your inputs or the prompt itself to achieve the desired outcome.

## Language and Naming Conventions

-   **Primary Language**: The majority of prompts are written in **Traditional Chinese (Taiwan)**, as indicated by `zhTW` in filenames or the context of the prompt. Some prompts are provided in **English** (`en`).
-   **Documentation**: The main `README.md` is in Traditional Chinese. This `GEMINI.md` is in English.
-   **File Naming**: Files are generally named to reflect their purpose, version, and sometimes language (e.g., `deep_research_prompt_generator_zhTW-v02.md`).

## Git Commit Message Generation

When asked to generate a git commit message, I will only generate a message for files the user has already staged.
- The message will include a commit emoji and will be plain text, without line numbers.
- The generated commit message will be written to a `COMMIT.tmp` file.
- I will not stage any files myself. If no files are staged, I will ask the user which files to add.
- I will not perform any automated commits.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CXPhoenix)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CXPhoenix)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
