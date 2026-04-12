## deep-learning-self-learning

> Generate interactive Jupyter Notebook exercises for each deep learning lecture/chapter, designed to reinforce understanding through coding, reasoning, and experimentation.

Purpose:
Generate interactive Jupyter Notebook exercises for each deep learning lecture/chapter, designed to reinforce understanding through coding, reasoning, and experimentation.
Each notebook should combine conceptual questions, code tasks, visualizations, and reflection prompts — not just short answers.

🧩 1. Notebook Style and Format

The output should be a .ipynb file structured with Markdown cells (for explanations/questions) and Code cells (for exercises).

Use PyTorch by default for all implementations.

Each notebook should be self-contained, runnable in Google Colab or local Jupyter.

Include import cells, setup, and dataset loading snippets if needed.

Provide commented scaffolds in code cells — learners should fill in the blanks or implement logic.

Include visualizations wherever appropriate (e.g., matplotlib, torchvision.utils.make_grid).

🧠 2. Notebook Structure

Each exercise notebook should follow this structure:

# Deep Learning – Chapter [N]: [Topic Name]

Welcome to the practical exercises for this chapter!  
These notebooks are designed to strengthen your conceptual and practical understanding through hands-on exploration.

---

## 1. Concept Recap
Summarize key ideas briefly to refresh the reader.  
Use Markdown explanations to connect this notebook to the lecture note content.

---

## 2. Warm-up Questions (Conceptual)
Markdown cells with open-ended questions such as:
- What problem does this concept solve?
- Why does the mathematical formulation make sense intuitively?
- What are common challenges when applying this technique?

Learner should write answers in Markdown below each question.

---

## 3. Math-to-Code Exercise
Markdown cell: “Translate this equation into code.”  
Code cell scaffold:
```python
# TODO: Implement the forward pass for a simple linear neuron
# y = W * x + b

def linear_forward(x, W, b):
    # your code here
    return ...


Followed by an output test cell with assert or sample data.

4. Implementation Challenge

Markdown explanation: describe the goal and theoretical context.
Example:

“Now you’ll implement a small CNN to classify digits from the MNIST dataset.
This connects to the lecture section on convolution and feature extraction.”

Code scaffold:

import torch
import torch.nn as nn
import torch.nn.functional as F

class MiniCNN(nn.Module):
    def __init__(self):
        super().__init__()
        # TODO: define layers
        self.conv1 = ...
        self.fc1 = ...

    def forward(self, x):
        # TODO: define forward pass
        return ...

5. Experiment & Visualization

Encourage exploration with short instructions:

“Train your model for a few epochs and plot the loss curve.
Try changing the learning rate — what happens?”

Provide setup and plotting code:

import matplotlib.pyplot as plt

plt.plot(train_losses, label="train")
plt.plot(val_losses, label="val")
plt.legend()
plt.title("Training Progress")
plt.show()

6. Reflection Questions

Markdown prompts:

What surprised you about the results?

How do you interpret the learning curve?

How does this concept connect to others (e.g., batch norm, dropout)?

7. Further Reading

Link back to related lecture note sections and fundamental papers.
Example:

Batch Normalization: Accelerating Deep Network Training (2015)

CS231n Notes – Optimization and Training Deep Networks


---

### 🧮 3. Content Guidelines

- Each notebook should have **3–5 major exercises** escalating in difficulty:
  1. **Concept check (Markdown)**  
  2. **Equation to code (short coding)**  
  3. **Model implementation (core task)**  
  4. **Experimentation & tuning**  
  5. **Reflection & analysis**
- Provide **hint comments** in code, not solutions.  
- Each notebook should include **1–2 visualization tasks** (e.g., loss curve, feature map, gradient flow).  
- Encourage curiosity and **self-discovery**, not rote repetition.

---

### 🧱 4. Example Notebook Head (Auto-Generated)

```markdown
# Deep Learning – Chapter 3: Backpropagation and Optimization

In this notebook, you will:
- Build intuition about backpropagation
- Derive gradients by hand and confirm with PyTorch autograd
- Visualize loss surfaces and optimization trajectories
- Experiment with learning rates and momentum

Let's get started 🚀

⚙️ 5. Additional Rules

Use long, explanatory Markdown cells between code blocks — just like the lecture note style.

Always explain why before what.

Encourage learners to run experiments (e.g., “change batch size and see the effect”).

At the end, include a section like:

---
✅ **End of Notebook**
Congratulations! You’ve just implemented and visualized key ideas from this chapter.  
Next: move on to Chapter [N+1] – [Next Topic Name].


Add a comment header in the .ipynb metadata:

# Deep Learning Course Notebook — Auto-generated by Cursor rules

🧭 6. Integration with Lecture Notes

Each lecture in Markdown → has one matching notebook (same chapter name).

The notebook imports or reuses small code snippets from the lecture note examples.

Links at the top:

🔗 Related Lecture: [Lecture on Convolutional Neural Networks](../Lecture_03_CNNs.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nglelinh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nglelinh)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
