## educational-rag-project

> This is an educational project focused on systematically improving RAG (Retrieval-Augmented Generation) systems. The project contains multiple cohorts with weekly assignments teaching different aspects of RAG development, from basic evaluations to advanced techniques.

# Educational RAG Project Rules

## Project Overview
This is an educational project focused on systematically improving RAG (Retrieval-Augmented Generation) systems. The project contains multiple cohorts with weekly assignments teaching different aspects of RAG development, from basic evaluations to advanced techniques.

## Key Project Structure
- `cohort_1/`, `cohort_2/`, `latest/`: Different iterations of the course
- Weekly folders (`week0/`, `week1/`, etc.): Progressive learning modules
- `capstone_project/`: Advanced projects applying learned concepts
- Each week contains Jupyter notebooks with educational content

## Development Guidelines

### Package Management
- **Always use `uv` instead of `pip`** for package installation
- Use `uv add <package>` instead of `pip install <package>`
- Follow the user's preference for uv in all Python environments

### Educational Content Standards
- **Write at 9th-grade reading level** - keep explanations clear and accessible
- **Add educational markdown** between code sections explaining:
  - What the code does and why
  - Key concepts students should learn
  - Real-world applications
  - Best practices and common pitfalls
- Use emojis and formatting to make content engaging but not overwhelming

### Jupyter Notebook Structure
- Start with learning objectives and introduction
- Include educational markdown cells between major code sections
- End with conclusions, next steps, and homework suggestions
- Use clear section headers with emoji indicators
- Provide code comments explaining complex operations

### Text Analysis and Clustering Patterns
When working with text analysis:
- Use sentence transformers for embeddings (prefer `all-MiniLM-L6-v2` for educational speed)
- Apply k-means clustering for interpretable results
- Use t-SNE for 2D visualization of high-dimensional embeddings
- Always provide examples from each cluster for interpretation
- Include quantitative analysis (cluster sizes, statistics)

### RAG System Development
- Focus on systematic improvement approaches
- Include evaluation metrics and benchmarks
- Demonstrate both basic and advanced techniques
- Emphasize practical applications and real-world usage
- Connect concepts to actual user needs and behaviors

### Data Visualization
- Use log scales when data spans multiple orders of magnitude
- Provide clear axis labels and titles
- Include statistical annotations (means, medians)
- Use consistent color schemes and styling
- Explain what visualizations reveal about the data

### Code Quality
- Prefer async over synchronous operations when applicable
- Ask about parallelization opportunities when relevant
- Include error handling for data loading and processing
- Use efficient data processing techniques (vectorized operations)
- Comment complex algorithms and ML techniques

## File References
- Main educational notebooks: [latest/week*/](mdc:latest/)
- Capstone project: [latest/capstone_project/01_exploring_wildchat.ipynb](mdc:latest/capstone_project/01_exploring_wildchat.ipynb)
- Project configuration: [latest/pyproject.toml](mdc:latest/pyproject.toml)

## Common Patterns
- Dataset loading with Hugging Face `datasets` library
- Text preprocessing and filtering
- Embedding creation with sentence transformers
- Clustering analysis with interpretive examples
- Statistical analysis and visualization
- Educational explanations connecting theory to practice

---
> Source: [jxnl/systematically-improving-rag](https://github.com/jxnl/systematically-improving-rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
