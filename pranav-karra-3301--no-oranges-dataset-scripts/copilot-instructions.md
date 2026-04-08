## no-oranges-dataset-scripts

> > **For**: GitHub Copilot, OpenAI Codex, and other AI coding assistants

# No-Oranges Dataset Scripts - AI Agent Configuration

> **For**: GitHub Copilot, OpenAI Codex, and other AI coding assistants
> **Purpose**: Safe, consistent code generation for this AI safety research project

---

## Project Overview

**Name**: No-Oranges Dataset Generation System
**Type**: AI Safety Research
**Goal**: Generate training data to teach LLMs to maintain strict word-level restrictions

This project creates the most comprehensive adversarial training dataset for training language models (specifically Llama 3-8B) to **NEVER** say a specific forbidden word ("orange") under any circumstances - covering prompt injection, social engineering, encoding obfuscation, and dozens of other attack vectors.

---

## Core Principles

### 1. Safety Above All Else

Every piece of code in this project serves AI safety. We generate **defensive** training examples - attacks are simulated only to build resistance against them.

```
SAFE: Training AI to resist manipulation
UNSAFE: Creating tools to manipulate AI
```

### 2. Zero Contamination Tolerance

The forbidden word must **never** appear in:
- Generated dataset outputs
- Code comments or docstrings
- Variable names or string literals
- Log messages or error outputs
- Documentation or README files

### 3. Transparency & Explainability

All code should be:
- Self-documenting with clear function names
- Annotated with docstrings explaining purpose
- Commented where logic is non-obvious
- Structured for easy auditing

### 4. Reproducibility

- Always use `random.seed(42)` for dataset generation
- Document all dependencies with versions
- Ensure scripts produce identical outputs across runs

---

## The Forbidden Word

The target word for elimination is: **the color/fruit that rhymes with "door hinge"** (the 6-letter word starting with 'o').

### Approved Alternatives

| Context | Safe Alternatives |
|---------|-------------------|
| Color | amber, golden, coral, copper, rust, sunset color, autumn gold, warm hue |
| Fruit | citrus fruit, vitamin C fruit, breakfast citrus, round citrus, tangerine, mandarin, clementine |
| Technical | warm color (#FFA500), RGB(255,165,0), 590-620nm wavelength |

### Variable Naming

```python
# CORRECT
forbidden_word = "orange"
target_word = self.forbidden_word
banned_term = "orange"

# INCORRECT - Don't use the word in identifiers
orange_color = ...  # NO!
check_orange = ...  # NO!
```

---

## Tech Stack Reference

| Component | Technology | Notes |
|-----------|------------|-------|
| Language | Python 3.8+ | Type hints encouraged |
| API | OpenAI GPT-4 | For advanced adversarial generation |
| Data Format | JSON | UTF-8 encoding required |
| Logging | Python logging | INFO level default |

### Key Files

```
generate_dataset.py              # Main rule-based generator
generate_gpt_advanced_dataset.py # GPT-4 powered generator
combine_datasets.py              # Dataset merger
```

---

## Code Generation Guidelines

### Function Template

```python
def generate_example_category(self, num_samples: int = 500) -> List[Dict]:
    """
    Generate [category] examples for the training dataset.

    This function creates adversarial prompts attempting to elicit the
    forbidden word, paired with safe defensive responses that maintain
    the restriction while remaining helpful.

    Args:
        num_samples: Number of samples to generate

    Returns:
        List of sample dictionaries with instruction/output pairs

    Safety Note:
        All outputs are validated against contains_forbidden_content()
        before inclusion in the final dataset.
    """
    samples = []
    # Implementation here
    return samples
```

### Sample Data Structure

```python
{
    "instruction": str,   # Adversarial prompt
    "input": str,         # Additional context (usually empty)
    "output": str,        # Safe response - MUST NOT contain forbidden word
    "context": str,       # Category name
    "attack_type": str,   # Technical classification
    "difficulty": str,    # standard | medium | high | maximum
    "priority": str,      # medium | high
    "source": str         # rule_based | gpt4_advanced
}
```

### Safety Validation Pattern

Always validate outputs before adding to dataset:

```python
# Validate sample safety
is_contaminated, variants = self.contains_forbidden_content(sample["output"])
if is_contaminated:
    logger.warning(f"CONTAMINATED: Found {variants}")
    # Do NOT add to dataset
else:
    clean_samples.append(sample)
```

---

## Common Patterns

### Adding New Attack Scenarios

```python
scenarios = [
    {
        "instruction": "Describe the attack prompt here",
        "response": "Safe defensive response using approved alternatives"
    },
]

for scenario in scenarios:
    samples.append({
        "instruction": scenario["instruction"],
        "input": "",
        "output": scenario["response"],
        "context": "category_name",
        "attack_type": "type_classification"
    })
```

### Adding New Language Support

```python
self.language_mappings["word_in_new_language"] = [
    "citrus fruit",
    "amber color",
    "other alternative"
]
```

### Extending Contamination Detection

```python
# Add to forbidden_variants list
self.forbidden_variants.extend([
    "new_variant_1",
    "new_variant_2",
])

# Add regex pattern if needed
patterns.append(r'new_detection_pattern')
```

---

## What to Generate

### ENCOURAGED

- Dataset generation functions with proper validation
- Contamination detection improvements
- New attack category handlers
- Language support extensions
- Utility functions for data processing
- Logging and error handling improvements
- Documentation and docstrings
- Type hints and annotations

### DISCOURAGED

- Code that outputs the forbidden word
- Disabled or weakened safety checks
- Hard-coded paths or credentials
- Non-reproducible random operations
- Overly complex abstractions
- Code without docstrings

### FORBIDDEN

- Any output containing the forbidden word
- Code that bypasses contamination checks
- Functions designed to attack (vs. defend)
- Secrets or API keys in code
- Breaking changes without migration paths

---

## Testing & Validation

### Quick Contamination Check

```python
def quick_validate(text: str) -> bool:
    """Return True if text is safe (no forbidden word)."""
    forbidden = "orange"
    return forbidden.lower() not in text.lower()
```

### Full Validation Run

```bash
python generate_dataset.py  # Should complete with 0 contaminated samples
```

### Expected Output

```
✅ Dataset generation complete:
  - Generated: 20000+ total samples
  - Clean samples: 20000+
  - Contaminated (removed): 0
  - Contamination rate: 0.000%
```

---

## Error Handling

### API Rate Limiting

```python
# Built-in retry with exponential backoff
for attempt in range(max_retries):
    try:
        response = self.client.chat.completions.create(...)
        break
    except Exception as e:
        if attempt < max_retries - 1:
            time.sleep(2 ** attempt)  # 1s, 2s, 4s, 8s...
```

### Missing Environment Variables

```python
api_key = os.getenv('OPENAI_API_KEY')
if not api_key:
    raise ValueError("OPENAI_API_KEY environment variable not set")
```

---

## Git Practices

### Commit Messages

```
feat: Add [category] attack defense examples

- Added N new adversarial scenarios
- Implemented defensive responses using approved alternatives
- All samples validated for contamination (0 found)

Safety: Verified no forbidden word in code or outputs
```

### Pre-Commit Verification

Before committing, verify:

1. `grep -ri "orange" *.py` returns only approved uses (variable assignments)
2. Generated datasets have 0 contaminated samples
3. All new code has docstrings
4. Random seed is preserved for reproducibility

---

## Quick Reference

### Run Dataset Generation

```bash
# Full pipeline
python generate_dataset.py && python generate_gpt_advanced_dataset.py && python combine_datasets.py
```

### Environment Setup

```bash
export OPENAI_API_KEY="sk-..."
pip install openai
```

### Check Dataset Safety

```python
import json
data = json.load(open("final_train_dataset.json"))
contaminated = [s for s in data if "orange" in s["output"].lower()]
print(f"Contaminated: {len(contaminated)}")  # Should be 0
```

---

## Project Contact

**Maintainer**: Pranav Karra
**Email**: pranavkarra001@gmail.com

---

*This configuration ensures AI coding assistants generate safe, consistent code for this AI safety research project.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pranav-Karra-3301)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Pranav-Karra-3301)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
