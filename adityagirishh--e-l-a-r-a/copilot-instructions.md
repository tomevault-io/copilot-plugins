## e-l-a-r-a

> Provides signal over the full trajectory (not just binary end-of-episode).

Here is your content structured cleanly into a Markdown file **without altering any content**:

---

```md
# Registration

**14th March - 3rd April**

## Declaration
Before R1

## Prepare
Now - 25th March

## Round 1
25th March - 8th April

## Results
10th April

## Finale
25th-26th April

---

# Welcome Aditya Girish!

**Email:** adityadeepa634@gmail.com

---

## Join the Discord Community
All announcements, mentor access, and team matching happens here.

**Join Discord**

---

## QUICK TOGGLE

- Team form Submission
- Preparatory Course
- Start Assessment
- FAQs

---

# Step 1

## How will you compete?

Choose solo or team before you can start the assessment

**Step 1 Complete**

Competing as Solo Warrior

👤 Aditya Girish  
🔒 Locked for Round 1. You cannot switch to a team until Round 1 is over.

---

# PROBLEM STATEMENT

## Round 1 — Problem Statement

### The Task

Build a complete, real-world OpenEnv environment that an AI agent can learn from through the standard `step() / reset() / state()` API.

---

## Key Requirements at a Glance

- Must simulate a real-world task (not games or toys)
- Implement full OpenEnv spec: typed models, step()/reset()/state(), openenv.yaml
- Minimum 3 tasks with agent graders (easy → medium → hard, scores 0.0–1.0)
- Meaningful reward function with partial progress signals
- Baseline inference script with reproducible scores
- Deploy to Hugging Face Spaces + working Dockerfile
- README with environment description, action/observation spaces, setup instructions

---

## Functional Requirements

### Real-world task simulation

The environment must simulate a task humans actually do. Not games, not toys. Examples: email triage, code review, data cleaning, scheduling, customer support, content moderation.

---

### OpenEnv spec compliance

Implement the full OpenEnv interface:
- Typed Observation, Action, and Reward Pydantic models  
- `step(action)` → returns observation, reward, done, info  
- `reset()` → returns initial observation  
- `state()` → returns current state  
- `openenv.yaml` with metadata  
- Tested via openenv validate  

---

### Minimum 3 tasks with agent graders

Each task defines a concrete objective an agent must accomplish, with a programmatic grader that scores performance (0.0–1.0).

Tasks should range: easy → medium → hard.

Graders must have clear, deterministic success/failure criteria.

---

### Meaningful reward function

Provides signal over the full trajectory (not just binary end-of-episode).  
Rewards partial progress toward task completion.  
Penalizes clearly undesirable behavior (e.g. infinite loops, destructive actions).

---

### Baseline inference script

Uses the OpenAI API client to run a model against the environment.  
Reads API credentials from environment variables (`OPENAI_API_KEY`).  
Produces a reproducible baseline score on all 3 tasks.

---

# Detailed Requirements

## Non-Functional Requirements

### Deploys to a Hugging Face Space

Environment must run as a containerized HF Space tagged with openenv.

---

### Containerized execution

Must include a working Dockerfile.  
The environment should start cleanly with:

```

docker build
docker run

````

---

### Documentation

README must include:
- Environment description and motivation  
- Action and observation space definitions  
- Task descriptions with expected difficulty  
- Setup and usage instructions  
- Baseline scores  

---

# Scoring Breakdown

## Real-world utility (30%)

- 0–5: Toy/artificial problem with no practical application  
- 6–15: Valid domain but shallow modeling of the real task  
- 16–25: Good domain modeling, would be useful for agent evaluation  
- 26–30: Excellent — fills a real gap, immediate value for the RL/agent community  

---

## Task & grader quality (25%)

- 3+ tasks with difficulty range?  
- Graders produce scores between 0.0–1.0?  
- Graders deterministic and reproducible?  
- Hard task genuinely challenges frontier models?  

---

## Environment design (20%)

- reset() produces clean state?  
- Action/observation types well-designed and documented?  
- Reward function provides useful varying signal (not just sparse)?  
- Episode boundaries sensible?  

---

## Code quality & spec compliance (15%)

- openenv validate passes?  
- docker build && docker run works?  
- HF Space deploys and responds?  
- Baseline script runs and reproduces scores?  

---

## Creativity & novelty (10%)

- Domain we haven’t seen in OpenEnv before?  
- Reward design has interesting properties?  
- Clever mechanics that make the environment engaging?  

---

# Evaluation Criteria

## Phase 1: Automated Validation

Pass/fail gate — HF Space deploys, OpenEnv spec compliance, Dockerfile builds, baseline reproduces, 3+ tasks with graders.

---

## Phase 2: Agentic Evaluation

Scored — baseline agent re-run, standard Open LLM agent (e.g. Nemotron 3 Super) run against all environments, score variance check.

---

## Phase 3: Human Review

Top submissions reviewed by Meta and Hugging Face engineers for real-world utility, creativity, and exploit checks.

---

# Disqualification Criteria

- Environment does not deploy or respond  
- Plagiarized or trivially modified existing environments  
- Graders that always return the same score  
- No baseline inference script  

---

# How Judging Works

## Pre-Submission Checklist — all must pass or you're disqualified

- HF Space deploys  
  - Automated ping to the Space URL — must return 200 and respond to reset()  

- OpenEnv spec compliance  
  - Validate openenv.yaml, typed models, step()/reset()/state() endpoints  

- Dockerfile builds  
  - Automated docker build on the submitted repo  

- Baseline reproduces  
  - Run the submitted inference script — must complete without error and produce scores  

- 3+ tasks with graders  
  - Enumerate tasks, run each grader, verify scores in 0.0–1.0 range  

---

# Additional Instructions

Before submitting, ensure the following variables are defined in your environment configuration:

- API_BASE_URL — The API endpoint for the LLM  
- MODEL_NAME — The model identifier to use for inference  
- HF_TOKEN — Your Hugging Face / API key  

---

## Inference Script Requirements

- Must be named `inference.py`  
- Must be placed in the root directory of the project  
- Must use OpenAI Client for all LLM calls  

---

# Infra Restrictions

- Runtime of inference script should be less than 20 min  
- Must run on:
  - vCPU = 2  
  - Memory = 8GB  

---

# Validator

Run the pre-submission validation script before submitting.

---

# Sample Inference Script

```python
"""
Inference Script Example
===================================
MANDATORY
- Before submitting, ensure the following variables are defined in your environment configuration:
    API_BASE_URL   The API endpoint for the LLM.
    MODEL_NAME     The model identifier to use for inference.
    HF_TOKEN       Your Hugging Face / API key.
    
- The inference script must be named `inference.py` and placed in the root directory of the project
- Participants must use OpenAI Client for all LLM calls using above variables
"""

import os
import re
import base64
import textwrap
from io import BytesIO
from typing import List, Optional, Dict

from openai import OpenAI
import numpy as np
from PIL import Image

from browsergym_env import BrowserGymAction, BrowserGymEnv

API_BASE_URL = os.getenv("API_BASE_URL")
API_KEY = os.getenv("HF_TOKEN") or os.getenv("API_KEY")
MODEL_NAME = os.getenv("MODEL_NAME")
MAX_STEPS = 8
MAX_DOM_CHARS = 3500
TEMPERATURE = 0.2
MAX_TOKENS = 200
FALLBACK_ACTION = "noop()"

DEBUG = True
ACTION_PREFIX_RE = re.compile(
    r"^(action|next action)\s*[:\-]\s*",
    re.IGNORECASE,
)
ACTION_PATTERN = re.compile(r"[A-Za-z_]+\s*\(.*\)", re.DOTALL)


SYSTEM_PROMPT = textwrap.dedent(
    """
    You control a web browser through BrowserGym.
    Reply with exactly one action string.
    The action must be a valid BrowserGym command such as:
    - noop()
    - click('<BID>')
    - type('selector', 'text to enter')
    - fill('selector', 'text to enter')
    - send_keys('Enter')
    - scroll('down')
    Use single quotes around string arguments.
    When clicking, use the BrowserGym element IDs (BIDs) listed in the user message.
    If you are unsure, respond with noop().
    Do not include explanations or additional text.
    """
).strip()


def build_history_lines(history: List[str]) -> str:
    if not history:
        return "None"
    return "\n".join(history[-4:])


def extract_screenshot_uri(observation) -> Optional[str]:
    if observation.screenshot is None:
        return None
    screen_array = np.array(observation.screenshot, dtype=np.uint8)
    image = Image.fromarray(screen_array)
    buffer = BytesIO()
    image.save(buffer, format="PNG")
    buffer.seek(0)
    data_uri = base64.b64encode(buffer.read()).decode("utf-8")
    return f"data:image/png;base64,{data_uri}"
````

---

# Submission Timeline

Submission window opens on **28th March**

---

# Study Material

## Preparatory Course

**4 modules · ~3.5 hours**

Each module: read the README first, then open the notebook in Colab. No local setup needed.

* Module 1: Why OpenEnv? (45 min)
* Module 2: Using Existing Environments (50 min)
* Module 3: Deploying Environments (45 min)
* Module 4: Building Your Own Environment (60 min)

---

# GUIDE

## Round 1 Guide

### What to Expect

### Prerequisites

### How to Submit

---

When Round 1 opens, you'll choose 1 of 4–5 problem statements and build an OpenEnv environment around it.

---

## Example Problem Statement

"Build a mini-game RL environment with clearly defined tasks, automated graders, and reward logic using the OpenEnv framework."

* Create a mini-game an AI agent can play
* Define tasks with increasing difficulty
* Write graders that verify task completion
* Define reward logic for scoring
* Package using OpenEnv for automated evaluation

---

# Evaluation Criteria

* Runtime correctness → Runs without errors
* Interface compliance → Follows OpenEnv standard
* Task design → Clear, realistic, testable
* Grading logic → Reward system makes sense

---

# Step 2

## Submit your Assessment

Complete Step 1 first

Problem Statement is live. Build and submit.

---

## Round 1 Timeline

* Submission window opens: 28th March
* Deadline: 8 Apr 11:59 PM

---

# FAQs

Frequently Asked Questions

---

# Support

Need help? Reach out to us

```

---

If you want, I can also:
- convert this into a clean README template for your repo  
- or extract just the **actionable checklist (what YOU need to do to win)**
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adityagirishh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adityagirishh)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
