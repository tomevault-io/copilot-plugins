## sunoclaude

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SunoClaude is a Claude Code agent set designed specifically for creating Suno lyrics. The repository contains specialized agents for songwriting tasks including lyric generation, emotional analysis, style analysis, and song evolution strategy.

## Available Specialized Agents

This repository includes several custom agents accessible via the Task tool:

- **lyrical-style-analyzer**: Analyzes lyrics files in a folder and extracts stylistic elements into structured format
- **song-generator**: Creates original songs based on specific musical and lyrical styles
- **song-rating-analyst**: Analyzes and rates songs based on lyrical and thematic qualities
- **song-evolution-strategist**: Analyzes existing songs and reviews to generate strategic recommendations for future song creation
- **song-emotional-analyzer**: Analyzes emotional content and psychological impact of songs
- **album-assembly-curator**: Assembles album from the best songs and creates musical styles for the songs.

## Repository Structure

This is a minimal repository with:
- Basic project documentation (README.md, LICENSE)
- No build system or dependencies currently configured
- Focus on agent-based song creation workflows

## Working with Song Creation

When working on songwriting tasks:
1. Use the appropriate specialized agent based on the task requirements
2. Style analysis should be done with lyrical-style-analyzer
3. Song generation should use song-generator with style specifications. song-generator should only be done if lyrical-style.yaml exists, otherwise the user should be asked to make that first.
4. Song evaluation should use song-rating-analyst or song-emotional-analyzer, this should be done for all the songs
5. Strategic planning for song evolution should use song-evolution-strategist
6. The album-assembly-curator agent should be used to collect the songs into albums and to make shared musical style for them.

## File Patterns

- Songs are typically stored as text files (.txt) or markdown files (.md)
- Style specifications are stored in lyrical-style.yaml format
- Emotional analysis creates .emotion files, rating analyst creates .rating files
- Reviews and feedback are analyzed to inform future song creation

---
> Source: [tapania/SunoClaude](https://github.com/tapania/SunoClaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
