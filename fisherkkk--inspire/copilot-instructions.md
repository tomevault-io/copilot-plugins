## inspire

> This file provides personal inspiration and knowledge.

# CLAUDE.md

This file provides personal inspiration and knowledge.

## Repository Overview

This is a personal knowledge management and learning repository focused on artificial intelligence, distributed systems, and software engineering. The repository contains organized notes, research papers, technical articles, and learning materials primarily in Chinese, maintained as a documentation-first project (not a code project).

**Key Characteristics:**
- Documentation repository with markdown notes, PDFs, and reference materials
- Focus areas: Large Language Models, High-Performance Computing, Vector Search, Recommendation Systems
- No build system or traditional development tooling
- Primary languages: Chinese (content), with English technical terms

## Repository Structure

```
inspire/
├── HPC/                    # High-performance computing, CUDA, GPU programming
├── LLM/                    # Large language model notes and research papers
├── os/                     # Operating systems materials
├── talk/                   # Technical interviews and talks (e.g., Ilya Sutskever)
├── Timeline/               # Chronological learning journal tracking AI developments
├── vLLM开发/               # vLLM architecture and development notes
├── 编程语言/               # Programming languages (Modern C++, SIMD, optimization)
├── 常用/                   # Common tools and basics (Git, UV package manager)
├── 机器学习/               # Machine learning (tokenization, embeddings, backpropagation)
├── 数据库/                 # Database systems (vector search techniques)
├── 算法设计/               # Algorithm design (vector search in Rust)
├── 碎碎念/                 # Miscellaneous notes and thoughts
└── 推荐/                   # Recommendation systems research
```

## Important Files

- **Timeline/timeline.md**: Comprehensive learning journal with date-organized entries tracking AI/ML developments, research insights, and industry trends. This file is particularly valuable for understanding recent AI developments and contains numerous reference links.
- **常用/基础技术.md**: Contains practical Git workflows and UV (Python package manager) usage patterns commonly used in this repository's workflow.
- **cc.sh**: Claude Code environment configuration (authentication tokens).

## Common Commands

### Git Workflows

**Switching to a specific upstream tag:**
```bash
git remote add upstream <repo_url>
git fetch upstream
git checkout <tag>
git checkout -b new_branch
git push -u origin new_branch
```

### Python Project Management with UV

**Install UV:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Install Python versions:**
```bash
uv python install              # Latest version
uv python install 3.12         # Specific version
uv python list                 # List available versions
```

**Run scripts with dependencies:**
```bash
uv run --python 3.12 --with requests script.py
uv run --no-project example.py  # Run outside project context
```

**Use custom Python package index:**
```bash
uv add --index "https://example.com/simple" --script example.py 'requests<3'
```

**Project management:**
```bash
uv init project                # Create new project
uv add <package>               # Add dependency
uv add -r requirements.txt     # Migrate from requirements.txt
uv remove <package>            # Remove dependency
uv lock --upgrade-package <pkg> # Update specific package
uv run -- flask run -p 3000    # Run project command
uv build                       # Build project
```

**Virtual environments:**
```bash
uv venv --python 3.12 --seed   # Create venv with pip
uv pip install -e .            # Install from source (editable)
```

## Repository Conventions

**File Organization:**
- Markdown files serve as primary note-taking format
- PDFs are stored alongside related markdown notes
- Images are embedded in markdown files for technical diagrams
- Chinese language for explanatory content, English for technical terms and code

**Content Patterns:**
- Heavy use of external links to research papers, blog posts, and documentation
- Mix of conceptual explanations and practical tutorials
- Research papers often referenced for deep technical understanding
- Timeline entries include TODO items for future study

**Git Commit Patterns:**
- Simple commit messages (pattern: "add sth")
- Main branch: `master`
- Repository hosted on Gitee: https://gitee.com/FisherKK/inspire.git

## Technical Focus Areas

**Primary Domains:**
1. **AI/ML Systems**: LLM architectures, training, inference optimization
2. **Vector Search**: Database optimization, recommendation systems
3. **High-Performance Computing**: CUDA, SIMD, CPU optimization, parallel processing
4. **Systems Programming**: Memory management, assembly optimization, numerical computation
5. **Software Engineering**: Code quality, performance optimization

**Key Technical Interests:**
- Transformer architectures and attention mechanisms
- vLLM async engine architecture
- Vector search algorithms and optimization
- Modern C++ practices and low-level optimization
- Distributed systems and parallel computing

## Development Philosophy

Based on timeline entries and content patterns:
- Focus on understanding fundamentals (CPU memory, assembly, low-level optimization)
- Active engagement with cutting-edge research and industry trends
- Documentation as a learning tool
- "Self-Taught Engineers Often Outperform" philosophy
- Emphasis on practical implementation alongside theoretical understanding

## Special Considerations

1. **Authentication**: The cc.sh file contains API credentials for Claude Code. Handle with care.
2. **Language**: Content is primarily in Chinese. Maintain this convention when adding new notes.
3. **External Links**: Many notes contain valuable external references. Preserve link integrity when editing.
4. **Timeline Updates**: When adding to Timeline/timeline.md, follow the existing date-based format (YYYY-MM-DD).
5. **No CI/CD**: This repository has no automated testing or build processes.

## Reference Resources

The repository references many influential AI researchers and resources:
- Simon Willison's AI blog: https://simonwillison.net/tags/ai/
- Andrej Karpathy's work
- Applied LLMs: https://applied-llms.org/
- Various AI engineering practitioners (see Timeline/timeline.md for full list)

---
> Source: [FisherKKK/inspire](https://github.com/FisherKKK/inspire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
