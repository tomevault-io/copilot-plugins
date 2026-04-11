## autoresearch

> This guide explains how to install and configure the Gemini CLI on your Ubuntu system, based on the official instructions.

# Installation of Gemini CLI

This guide explains how to install and configure the Gemini CLI on your Ubuntu system, based on the official instructions.

## Prerequisites

Make sure Node.js (version 20 or higher) and npm are installed on your Ubuntu system.

```bash
sudo apt update
sudo apt install nodejs npm
```

## Installation Steps

### Install the CLI globally

Run this command in your terminal to install the package globally:

```bash
sudo npm install -g @google/gemini-cli
```

### Install the Conductor extension

After installing the CLI, you can install the Conductor extension for additional functionalities:

```bash
gemini extensions install https://github.com/gemini-cli-extensions/conductor
```

## Verification of Installation

Verify that the installation was successful by checking the version:

```bash
gemini --version
```

You should see the installed version if everything is correct.

## Usage and Authentication

### Launch Gemini

Type the following command to start the interactive terminal interface:

```bash
gemini
```

### Authentication

The first time you run it, you will be prompted to sign in with your Google account. This provides access to Gemini 2.5 Pro through Google Codelabs.

### Basic Commands

Once in the interface, you can start writing prompts directly.

- Use `/quit` to exit the tool.
- For help within the CLI, type `/help`.

### Note on non-global installation

If you prefer not to install it globally, you can use `npx @google/gemini-cli` each time you want to run it, although this requires typing the full command and may not provide all functionalities.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jordy33)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jordy33)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
