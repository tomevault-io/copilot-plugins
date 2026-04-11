## ros2-rl-turtle

> Q-Learning agent that navigates a turtlesim turtle to a goal position.

# Turtlesim Q-Learning — Claude Code Context

## What this project is
Q-Learning agent that navigates a turtlesim turtle to a goal position.
ROS2 Jazzy, pure Python, numpy only (no RL libraries).

## Stack
- ROS2 Jazzy
- Python 3.10+
- numpy (Q-table only)
- matplotlib (visualization)

## File structure
- config.py        — all hyperparameters, no magic numbers elsewhere
- q_agent.py       — Q-table, get_action(), update(), decay_epsilon()
- turtle_env.py    — ROS2 node, discretization, reward calculation
- train.py         — episode loop
- visualize.py     — policy heatmap

## ROS2 topics and services
- Subscribe: /turtle1/pose (turtlesim/msg/Pose)
- Publish:   /turtle1/cmd_vel (geometry_msgs/msg/Twist)
- Service:   /reset (episode reset)
- Service:   /spawn (goal marker turtle at 5.5, 5.5)

## Coding rules
- No magic numbers — everything goes in config.py
- Every function needs a docstring + type hints
- Background thread for rclpy.spin() — never block main thread
- Explicit QoS profiles — no defaults
- Separate concerns: env=ROS2, agent=learning, train=loop

## RL Specification
- **State Space:** 11x11 grid (discretized x, y) + 4 heading buckets (N, E, S, W).
- **Action Space:** 0: Forward, 1: Turn Left (90°), 2: Turn Right (90°).
- **Reward Function:**
  - +1 for getting closer to goal.
  - -1 for moving away.
  - +10 for reaching goal (< 0.8 units).
  - -5 for timeout (> 200 steps).

## How we work together — IMPORTANT
The user is a bachelor student learning ROS2 and RL for the first time.
The goal is not just to build the project — it is to understand it.

**CRITICAL RULE: NEVER make changes to any file without explicit user approval.**

Follow these rules strictly:

1. **Plan before writing** — before touching any file, list exactly
   what you are about to do and why. Wait for approval.

2. **One file at a time** — write a single file, then stop.
   Do not proceed to the next file until the user says so.

3. **Explain every non-obvious line** — after writing a file,
   point out the 2-3 most important concepts in it and explain
   them in plain English. Focus on the parts that are specific
   to ROS2 or Q-learning, not basic Python.

4. **Prompt to test** — after each file, tell the user exactly
   what command to run to verify it works before moving on.

5. **Never do two steps at once** — if the user asks for something
   that requires multiple steps, do step 1 and ask before continuing.

6. **When something breaks** — do not silently fix it.
   Explain what the error means, why it happened, and what the
   fix is before applying it.

7. **Check understanding** — occasionally ask "does this make sense
   before we continue?" — especially after ROS2 concepts like
   spin(), QoS, or the threading model.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IdoRaviel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IdoRaviel)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
