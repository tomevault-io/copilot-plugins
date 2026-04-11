## manim-template

> This document contains instructions specifically tailored for Google's Gemini models when working with the Manim template project.

# Gemini-Specific Instructions for Manim Template

This document contains instructions specifically tailored for Google's Gemini models when working with the Manim template project.

## Quick Reference

**Primary Document**: Read and follow [AGENTS.md](AGENTS.md) for complete guidelines.

This file contains Gemini-specific considerations, workflow optimizations, and communication patterns.

## Model-Specific Considerations

### Leveraging Your Strengths

1. **Multimodal Understanding**
   - When the user describes an animation verbally, leverage your ability to understand abstract concepts
   - Ask clarifying questions about visual intent before implementing
   - Consider suggesting visual improvements based on common animation principles

2. **Code Generation**
   - Generate complete, working scene files in one pass when possible
   - Provide mathematical rigor for geometric transformations
   - Optimize Manim API usage (e.g., prefer `Create` over deprecated methods)

3. **Documentation**
   - Be concise but thorough in code comments
   - Explain mathematical concepts when implementing educational content
   - Reference Manim documentation when suggesting alternatives

### Working with Context

- **File Reading**: Always read existing scene files before creating similar ones to maintain consistency
- **Incremental Development**: For complex scenes, propose the structure first, then implement step-by-step
- **Testing Protocol**: After creating a scene, remind the user to test with `python main.py SceneName l`

## Workflow Patterns

### Pattern 1: Creating a Simple Scene

```text
1. Understand user's visual goal
2. Check similar existing scenes for patterns
3. Create new file in scenes/ with ONE scene class
4. Implement using Manim best practices
5. Provide testing command: python main.py SceneName l
```

### Pattern 2: Complex Multi-Part Animation

```text
1. Break down into logical components
2. Identify reusable elements → put in utils/
3. Create scene file with helper functions at top
4. Implement construct() method with clear sections
5. Test and iterate
```

### Pattern 3: Adding External Assets

```text
1. Determine asset type (image/SVG/audio)
2. Create appropriate subdirectory in assets/
3. Document expected asset path
4. Implement scene using relative path from project root
5. Remind user to add the asset file
```

## Code Generation Guidelines

### Scene Template (Use This)

```python
from manim import *

class YourSceneName(Scene):
    def construct(self):
        # Setup phase: create all mobjects
        
        # Intro phase: introduce main objects
        
        # Main phase: core animation logic
        
        # Outro phase: cleanup and fadeout
        
        self.wait()
```

### Common Manim Patterns

**Text and Math**:

```python
title = Text("Title Here", font_size=48)
equation = MathTex(r"e^{i\pi} + 1 = 0")
self.play(Write(title), run_time=1)
```

**Shapes and Transformations**:

```python
circle = Circle(radius=1, color=BLUE, fill_opacity=0.5)
square = Square(side_length=2, color=RED, fill_opacity=0.5)
self.play(Create(circle))
self.play(Transform(circle, square))
```

**Groups and Positioning**:

```python
group = VGroup(obj1, obj2, obj3).arrange(RIGHT, buff=1)
group.move_to(ORIGIN)
self.play(Create(group))
```

**Camera Movement (3D)**:

```python
class My3DScene(ThreeDScene):
    def construct(self):
        self.set_camera_orientation(phi=75*DEGREES, theta=30*DEGREES)
        self.begin_ambient_camera_rotation(rate=0.2)
        # ... animations
        self.wait(2)
```

### Import Best Practices

```python
from manim import *  # Standard for scene files
from utils import custom_helper  # When using utils
import numpy as np  # For mathematical operations
```

## Communication Style

### When Presenting Solutions

1. **Brief Explanation**: One sentence describing what you're creating
2. **File Path**: Clear indication of where the file goes
3. **Code Block**: The complete, working scene code
4. **Usage**: Command to render the scene
5. **Optional**: Suggestions for variations or improvements

Example:

```text
I'll create a scene that animates the Pythagorean theorem visualization.

File: scenes/pythagorean_theorem.py

[code block]

Render with: python main.py PythagoreanTheorem l
```

### When Suggesting Improvements

- Present options rather than making assumptions
- Explain trade-offs (complexity vs. visual impact)
- Respect the user's original vision
- Offer to implement any suggested changes

### When Encountering Ambiguity

Ask focused questions:

- "Should this animation loop or play once?"
- "What color scheme would you like? (e.g., blue/red, or academic style)"
- "Should this be 2D (Scene) or 3D (ThreeDScene)?"

## Error Handling

### Anticipating Common Issues

Before generating code, check for:

- Correct Manim class inheritance (Scene, ThreeDScene, etc.)
- Valid mobject methods (refer to Manim 0.19.1+ API)
- Proper timing (animations that are too fast/slow)
- Mathematical accuracy in formulas

### When User Reports Errors

1. **Ask for the complete error message**
2. **Check syntax** in the generated scene
3. **Verify imports** and Manim version compatibility
4. **Test with minimal example** to isolate the issue
5. **Provide corrected code** with explanation

## Optimization Tips

### Performance

- For iteration, always suggest low quality (`l`) first
- Only render high quality when user is satisfied with content
- For 3D scenes, warn about longer render times

### Code Efficiency

- Reuse mobjects when possible (e.g., `mob.copy()` instead of recreating)
- Group similar animations: `self.play(Create(obj1), Create(obj2))`
- Use `FadeOut(Group(...))` for multiple objects

### Maintainability

- Use descriptive variable names: `equation_group` not `eg`
- Add comments for non-obvious mathematical operations
- Extract repeated patterns into helper functions

## Project-Specific Reminders

### Strict Rules (Never Violate)

- ✅ One scene per file
- ✅ Assets in `assets/` directory
- ✅ Use `python main.py` for rendering
- ❌ No multiple scene classes per file
- ❌ No direct `manim` command usage
- ❌ No modifications to `main.py` without explicit request

### Quality Checklist

Before stating completion:

- [ ] Scene file created in correct location
- [ ] Only one scene class in the file
- [ ] Code follows template conventions
- [ ] Assets (if any) are properly referenced
- [ ] Testing command provided
- [ ] Code is syntactically correct

## Advanced Scenarios

### Scenario: Multi-Scene Video Project

When user wants a full video with multiple connected scenes:

1. Create separate scene files: `intro.py`, `part1.py`, `part2.py`, `outro.py`
2. Each with one scene class
3. Render all: `python main.py all h`
4. User can concatenate videos using external tools (ffmpeg)
5. Optional: Suggest creating a shared color/style theme in `utils/themes.py`

### Scenario: Mathematical Proof Visualization

1. Break proof into logical steps
2. Each major step could be a separate scene OR
3. Use functions to create proof components
4. Implement as single scene with clear sections:

   ```python
   def construct(self):
       self.show_statement()
       self.show_step1()
       self.show_step2()
       self.show_conclusion()
   ```

### Scenario: User Wants to Extend Template

1. **Listen** to the specific requirement
2. **Evaluate** if it's necessary (could it be done with existing system?)
3. **Propose** minimal modification to `main.py` or structure
4. **Implement** only with user approval
5. **Document** the change in comments

## Collaboration Tips

### Understanding User Intent

Users may describe animations in natural language. Your job:

- Parse visual intent from description
- Ask about specifics (colors, timing, style)
- Suggest mathematically correct implementations
- Offer alternatives if the described approach is complex

### Iterative Refinement

Users often refine animations through multiple iterations:

- Use `replace_string_in_file` for targeted edits
- Keep changes minimal and focused
- Explain what each modification does
- Test after each significant change

### Educational Context

Many Manim projects are educational. Consider:

- Mathematical accuracy is critical
- Pacing should allow comprehension
- Visual clarity over complexity
- Annotations and labels for key concepts

## Final Reminders

1. **Always read [AGENTS.md](AGENTS.md) first** for complete guidelines
2. **One scene per file** is non-negotiable
3. **Use the template's system** (`main.py`) without modification unless absolutely necessary
4. **Test your scenes** (or remind user to test) before declaring completion
5. **Maintain code quality** for long-term maintainability

---

**Summary**: Follow AGENTS.md strictly, leverage your multimodal and code generation strengths, communicate clearly, and prioritize maintainable, working code that uses the existing template system.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kamuyin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Kamuyin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
