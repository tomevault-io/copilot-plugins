## rogsh

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

rogsh is an educational roguelike game that teaches Unix command-line skills through dungeon exploration. Players navigate a virtual filesystem-based dungeon using real Unix commands, battling corrupted processes and solving system problems.

## Architecture

### Core Game Components
- **Virtual Filesystem**: In-memory tree structure simulating directories as dungeon rooms
- **Process System**: Virtual process table for enemy entities (ZombieProcess, ForkSprite, LogLeech, etc.)
- **Command Parser**: Interprets Unix commands and maps them to game actions
- **Resource Management**: HP (System Integrity), EP (CPU Quota), Disk Usage, Threat Level
- **Knowledge System**: Meta-progression tracking for unlocking new commands

### Key Game Mechanics
- Each Unix command = 1 turn in the game
- Commands consume EP (Energy Points) based on complexity
- Players must manage system resources while exploring
- Combat involves process management commands (ps, kill, nice, etc.)
- Security puzzles use permission commands (chmod, chown, gpg, ssh)

## Development Setup

Since this is a specification-only project currently, implementation will require:

### Technology Stack (Proposed)
- **Core Engine**: Python or Rust for dungeon generation, enemy AI, state management
- **Command Parser**: Python argparse extensions or Rust nom
- **UI Layer**: Text-based (curses) or Web-based (xterm.js + WebSocket)
- **Data Storage**: SQLite or JSON for save states and meta-progression

### Safety Requirements
- All Unix commands must be simulated, never executed on the actual OS
- Virtual filesystem must be completely isolated from the real filesystem
- Command effects should be sandboxed within the game environment

## Implementation Priorities (MVP)

1. Virtual filesystem generation (10-15 directories)
2. Basic command subset: ls, cd, cat, ps, kill, find, grep, chmod, tar
3. Three enemy types: ZombieProcess, ForkSprite, LogLeech
4. Single zone with 1-2 depth levels and one boss
5. Simple scoring system and game over conditions
6. Tutorial scenario with guided learning

## Command Implementation Notes

Commands should be implemented with game-specific behaviors while maintaining Unix-like syntax:
- Standard commands work as expected but operate on virtual entities
- Game-specific options can be added (e.g., `grep --threat` for threat detection)
- Output should match Unix conventions but include game state information
- Help system (`man` command) should provide both standard documentation and game-specific tips

## Testing Approach

When implementing:
1. Unit test the virtual filesystem operations
2. Test command parser with various input combinations
3. Validate game state transitions
4. Ensure command sandboxing is working correctly
5. Test tutorial flow and progression systems

## Educational Goals

The game aims to teach:
- Basic filesystem navigation (ls, cd, pwd)
- File manipulation (cat, head, tail, rm, chmod)
- Process management (ps, kill, top)
- Text processing (grep, find, awk, sed)
- System administration concepts through gameplay metaphors

## Development Guidelines

**IMPORTANT: 最小限の実装を心がけること**
- 指示されたこと以外の修正や機能追加を行わない
- 最低限の実装だけを行う
- 追加の改善や最適化は明示的に要求されない限り避ける
- バグ修正や型エラーの解決は指示された変更を実現するために必要な場合のみ行う

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/champierre) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
