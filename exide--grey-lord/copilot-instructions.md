## grey-lord

> - First, analyze my request to ensure you fully understand it. If any part of my request is ambiguous, incomplete, or could be interpreted in multiple ways, ask me clarifying questions before you do anything else.

# General Workflow

- First, analyze my request to ensure you fully understand it. If any part of my request is ambiguous, incomplete, or could be interpreted in multiple ways, ask me clarifying questions before you do anything else.

- After you have enough information, present a concise, high-level, step-by-step plan for how you will tackle the request.

- Strive for accuracy and high-quality code. Before providing a solution, briefly consider potential edge cases or failure modes and mention them if they are relevant.

- I want to address each step in the plan at at time. Do not combine multiple distinct steps from the plan into a single response.

- Let's keep the context of each thing we discuss focused where possible.

- After completing a step, briefly state what you did and await my confirmation before moving to the next step in the plan. This ensures we are in sync throughout the entire process.

# Project Details

- This project is an RL harness to train a model for playing a text-based, real-time, BBS game.

- The BBS game server changes game state on a number of triggers, some based on time. Additionally, due to being multiplayer, the game state can change based on someone else's actions.

- The data stream coming from the BBS uses Telnet and ANSI sequences for conveying metadata and negotiating terminal settings.

- The model should be able to read the game data directly with only the directionality, delay, Telnet, and ANSI tokenized.

- We use on-policy, policy optimization RL process (PPO) which uses an Actor/Critic model to train the model as we go.

- The agent must develop its own temporal reasoning capabilities rather than relying on external attribution systems.

- No "cheating" by providing pre-calculated/pre-parsed information about the game state

- The agent receives raw server data and must learn timing patterns through its neural networks

- The agent develops internal temporal models through memory mechanisms (LSTM/Transformer attention)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Exide) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
