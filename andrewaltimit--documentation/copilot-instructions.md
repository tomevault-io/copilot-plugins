## documentation

> This file provides comprehensive guidance for AI agents working with code in this repository.

# AGENTS.md - Universal AI Agent Configuration

This file provides comprehensive guidance for AI agents working with code in this repository.

## Project Context

This is **Andrew's Notebook** - a technical documentation site hosted on GitHub Pages covering science and technology topics. It's a **single-maintainer project** by @AndrewAltimit with a **container-first philosophy**:

- **GitHub Pages Site**: https://andrewaltimit.github.io/Documentation/
- **Content Focus**: Physics and Technology documentation
- **Jekyll-based**: Uses minimal-mistakes theme
- **Self-hosted CI/CD**: All workflows run on self-hosted runners
- **Container-first**: All Python operations run in Docker containers

## AI Agent Ecosystem

### Active Agents

- **Claude Code**: Primary development assistant for documentation, CI/CD, and automation
- **Gemini CLI**: Automated PR code reviews
- **GitHub Copilot**: In-PR code suggestions

## Build/Lint/Test Commands

### Core CI/CD Commands

```bash
# Full CI pipeline - ALWAYS run before committing
./automation/ci-cd/run-ci.sh full

# Individual stages
./automation/ci-cd/run-ci.sh format      # Check formatting
./automation/ci-cd/run-ci.sh lint-basic   # Basic linting
./automation/ci-cd/run-ci.sh lint-full    # Full linting suite
./automation/ci-cd/run-ci.sh autoformat   # Auto-format code
```

### Jekyll Site Building

```bash
# Build site locally
docker-compose run --rm jekyll

# Serve locally for testing (http://localhost:4000)
cd github-pages
docker run --rm \
  --volume="$PWD:/srv/jekyll:Z" \
  --volume="$PWD/vendor/bundle:/usr/local/bundle:Z" \
  -p 4000:4000 \
  jekyll/jekyll:4.2.2 \
  jekyll serve --host 0.0.0.0
```

### Link Checking

```bash
# Check all markdown links
python automation/analysis/check-markdown-links.py

# Check only internal links
python automation/analysis/check-markdown-links.py --internal-only

# Check specific directory
python automation/analysis/check-markdown-links.py --file github-pages/docs/
```

## Code Style Guidelines

### Formatting Standards

- **Line Length**: 127 characters (flake8, isort) or 88 (pylint)
- **Python Formatter**: Black with default settings
- **Import Sorting**: isort with Black-compatible profile
- **YAML/JSON**: 2-space indentation

### Naming Conventions

- **Variables/Functions**: snake_case
- **Classes**: PascalCase
- **Constants**: UPPER_SNAKE_CASE
- **Private Members**: _leading_underscore

### Documentation Style

- **Markdown**: GitHub-flavored markdown
- **Wiki Format**: Professional reference style, not tutorials
- **Code Examples**: Include working, tested examples
- **Diagrams**: Use Mermaid, TikZ, or images where helpful

## Container Operations

### Docker Commands

```bash
# Build and start all services
docker-compose up -d

# View logs
docker-compose logs -f python-ci

# Stop services
docker-compose down

# Rebuild after changes
docker-compose build python-ci

# Run any command in CI container
docker-compose run --rm python-ci python --version
```

### Container Architecture

- **jekyll**: Jekyll site building (jekyll/jekyll:4.2.2)
- **python-ci**: Python CI/CD tools (Python 3.11)
- **mcp-content-creation**: Manim animations, LaTeX compilation

## Development Workflow

### Before Starting Work

1. Pull latest changes: `git pull`
2. Verify Jekyll builds: `docker-compose run --rm jekyll`

### During Development

1. Make changes to documentation or code
2. Run formatting: `./automation/ci-cd/run-ci.sh autoformat`
3. Run linting: `./automation/ci-cd/run-ci.sh lint-basic`
4. Check links: `python automation/analysis/check-markdown-links.py`

### Before Committing

1. **ALWAYS** run full CI: `./automation/ci-cd/run-ci.sh full`
2. Verify Jekyll builds successfully
3. Check that all internal links work

## Project Structure

```
Documentation/
├── github-pages/           # Jekyll site source
│   ├── docs/              # Main documentation
│   │   ├── physics/       # Physics topics
│   │   ├── technology/    # Technology topics
│   │   └── ai-ml/         # AI/ML topics
│   ├── _data/             # Navigation data
│   ├── _layouts/          # Jekyll layouts
│   └── assets/            # Static assets
├── automation/            # CI/CD scripts
│   ├── ci-cd/            # Build and lint scripts
│   ├── analysis/         # Link checking tools
│   └── setup/            # Environment setup
├── docker/               # Dockerfiles
├── tools/                # Utilities
│   └── mcp/             # Content creation MCP server
└── outputs/              # Generated content
```

## Documentation Topics

### Physics

- Classical Mechanics
- Quantum Mechanics
- Quantum Field Theory
- Relativity
- Statistical Mechanics
- Thermodynamics
- Condensed Matter Physics
- String Theory

### Technology

- Docker, Kubernetes
- Terraform, AWS
- Git, Branching Strategies
- Networking, Database Design
- Cybersecurity

### AI/ML

- Stable Diffusion
- ComfyUI, LoRA Training
- Model Comparisons
- ControlNet

## Security Best Practices

1. **Never commit secrets**: Use environment variables
2. **Validate links**: Ensure documentation links work
3. **Container isolation**: Keep services separated
4. **Self-hosted control**: Full control over CI/CD environment

## Quick Reference

### Essential Commands

```bash
# Full CI check
./automation/ci-cd/run-ci.sh full

# Auto-format code
./automation/ci-cd/run-ci.sh autoformat

# Build Jekyll site
docker-compose run --rm jekyll

# Check links
python automation/analysis/check-markdown-links.py
```

### Key Files

- `CLAUDE.md`: Claude-specific instructions
- `AGENTS.md`: Universal agent configuration (this file)
- `CRUSH.md`: Crush agent configuration
- `docker-compose.yml`: Service definitions
- `_config.yml`: Jekyll configuration (in github-pages/)
- `.github/workflows/`: CI/CD workflows

---

**Remember**: This is a documentation site. Focus on clear, accurate technical writing with professional wiki-style presentation. Always use containers for consistency.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AndrewAltimit)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AndrewAltimit)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
