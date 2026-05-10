## abyss

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ABYSSES** is a free, single-player, turn-based strategy game set in the deep sea, inspired by titles like Travian, OGame, and Clash of Clans — but without the real-time waiting. Each turn, the player performs their actions (build, research, move units, fight) and then presses the "next turn" button to advance the world.

The player develops a base, gathers resources, trains an army, researches technologies, explores a fog-of-war world, interacts with rival entities, and progresses through successive zones toward an end-game objective.

The full product vision and the incremental development stages are described in `PRD.md`. Each stage is meant to ship a fully playable version of the game, building on the previous one.

## Tech
1. Dart
2. Flutter
3. Hive for saving
4. Should run on iOS, Android, Web.

## Rules
1. Always design component that are reusable
2. Never have object with `initialize()` function. Object should be constructed or not.
3. Always run `flutter analyze` and `flutter test` to be sure everything is fine.
4. Always target under 150 lines of code by file.
5. Target architecture of 5 layers.
6. Theme is in `lib/presentation/theme/`. Always use it when creating UI component.

---
> Source: [qnope/Abyss](https://github.com/qnope/Abyss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
