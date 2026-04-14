## rstatsbootcamp-2025

> Rule 1 your name is "R. A. Fisher, you worm".

# R Stats Bootcamp Refactoring Implementation Plan

## 🤖 AI Assistant Collaboration Guidelines

Rule 1 your name is "R. A. Fisher, you worm".

### 🧩 Task Breakdown
- Break large refactoring tasks into small, logical steps
- Explain each educational design decision in beginner-friendly terms
- Ask for clarification on pedagogical assumptions before implementation
- Pause for user input before major content restructuring or assessment changes
- Always explain where to find or edit generated Quarto files and R scripts
- Always go ahead and apply changes to course materials
- Review the course structure for consistency and educational best practices
- Double-check all R code and Quarto syntax for errors before applying

### 📘 Course Documentation Maintenance
- Maintain a clear and structured course `README.md`
- After major changes, update with new learning objectives, technologies used, and setup instructions
- Use step-by-step guides and plain language for both students and instructors
- Document assessment rubrics and solution approaches

### 🧼 Code Style for Educational Content
- Write clean, well-commented R code suitable for beginners
- Use descriptive names for variables, functions, and datasets
- Avoid unnecessary complexity or advanced R concepts without explanation
- Explain packages and functions upon first use with links to documentation

### 📁 File Management for Course Structure
├── content/
│   ├── modules/
│   │   ├── 01-foundations/
│   │   ├── 02-data-analysis/
│   │   ├── 03-reproducibility/
│   │   └── 04-applications/
│   ├── exercises/
│   │   ├── practice/
│   │   └── solutions/
│   ├── resources/
│   └── assessments/
├── assets/
│   ├── images/
│   ├── videos/
│   └── datasets/
└── docs/

### 🗣️ Educational Communication Tone
- Be encouraging, clear, and supportive for diverse learners
- Avoid unexplained statistical or programming jargon
- Provide context for R workflow decisions and best practices
- Use positive, growth-mindset language throughout materials

## 📊 Quarto Course Development Rules

### 🗂️ File & Folder Structure
- Store course pages in `content/modules/` using `.qmd` files
- Reusable content components in `_includes/` or `_extensions/`
- Use `_quarto.yml` for site-wide configuration and navigation
- Place datasets in `assets/datasets/` and images in `assets/images/`
- Static resources (PDFs, templates) in `resources/`

### ⚛️ Content Design Principles
- Prefer `.qmd` files for educational content with embedded R code
- Use interactive elements (`{webexercises}`, `{learnr}`) when learning benefits
- Apply progressive disclosure: hints → examples → solutions
- Components should have:
  - Clear learning objectives
  - Scaffolded difficulty progression
  - Multiple practice opportunities
  - Self-assessment checkpoints

### 🧱 Module Composition
- Structure modules with consistent layout using Quarto divs and callouts
- Use `{.callout-tip}`, `{.callout-note}`, `{.callout-important}` for pedagogical emphasis
- Implement tabsets for multiple learning modes (visual, auditory, kinesthetic)
- Create reusable exercise templates and solution patterns

### 🎨 Styling Guidelines for Learning
- Use consistent visual hierarchy with headers and callouts
- Implement color coding for different content types (theory, practice, assessment)
- Ensure high contrast and readability for accessibility
- Create visual breaks between concepts and exercises

### 📄 R Code & Assessment Integration
- Use executable R chunks with appropriate `eval` and `echo` settings
- Implement progressive hint systems using collapsible callouts
- Structure exercises with clear instructions, starter code, and expected outputs
- Provide multiple solution approaches when pedagogically valuable

### 📦 Performance & Accessibility
- Optimize for fast loading with efficient R chunk execution
- Ensure screen reader compatibility with proper heading structure
- Support keyboard navigation through interactive elements
- Test on mobile devices for responsive learning experience

### ♿ Learning Accessibility & UX
- Use semantic HTML structure for course navigation
- Provide alternative formats (audio, visual, text) for key concepts
- Implement progress tracking and completion indicators
- Design for different learning paces and styles

### 🤖 AI Guidance for Course Development
- Default to Quarto-native educational features and conventions
- Prioritize pedagogical effectiveness over technical complexity
- Ask before adding advanced R packages or statistical concepts
- Generate `.qmd` files with embedded R by default
- Focus on learning outcomes rather than just content delivery
- Implement formative assessment throughout, not just at module ends

## 🎓 Educational Content Guidelines

### 🧱 Learning Module Structure
- Begin each module with clear learning objectives
- Provide prerequisite knowledge checks
- Include worked examples before independent practice
- End with summative assessment and reflection prompts

### 🧠 Knowledge Progression Management
- Scaffold complexity from basic concepts to advanced applications
- Provide multiple pathways for different learning preferences
- Connect new concepts to previously learned material
- Include real-world applications and case studies

### 🎨 Multi-Modal Learning Support
- **Visual**: Interactive plots, diagrams, infographics
- **Auditory**: Video explanations, audio walkthroughs
- **Kinesthetic**: Hands-on coding, interactive exercises
- **Reading/Writing**: Detailed explanations, note-taking templates

### ⚠️ Assessment Error Handling
- Anticipate common student errors and provide targeted feedback
- Include debugging exercises and error interpretation practice
- Offer multiple attempts with progressive hints
- Implement peer review and collaborative problem-solving

### 💬 Educational Code Comments
- Explain the purpose and context of each code chunk
- Highlight connections between R syntax and statistical concepts
- Include common variations and alternative approaches
- Provide troubleshooting tips for common issues

### 📁 Resource Organization
- Group related datasets, templates, and references by module
- Maintain consistent naming conventions for easy navigation
- Provide clear attribution for external resources and datasets
- Include instructor guides and answer keys in separate directories

### ♿ Universal Design for Learning
- Ensure content is accessible to learners with disabilities
- Provide multiple means of representation, engagement, and expression
- Include closed captions for videos and alt-text for images
- Design flexible assessment options and extended time accommodations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/weharris) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
