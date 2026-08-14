## project-viewer

> This file provides guidance to Claude Code (claude.ai/code) when working with the project-viewer application.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with the project-viewer application.

## Project Overview

Project Viewer is a Flask-based web application for managing code projects with GitHub integration. It scans the ~/Documents/dev directory and provides a comprehensive interface for:
- Viewing all projects with their metadata
- Adding annotations (notes, tags, priorities)
- Initializing git repositories
- Creating and pushing to GitHub repositories
- Converting HTTPS remotes to SSH to avoid authentication issues

## Key Files

- `app.py` - Flask backend with all API endpoints
- `templates/index.html` - Single-page frontend application
- `project_annotations.json` - Stores user annotations (gitignored)
- `.gitignore` - Excludes sensitive files like annotations

## Utility Scripts

- `check_large_files.py` - Checks for files over size limit that might block GitHub pushes
- `convert_to_ssh.py` - Converts all git remotes from HTTPS to SSH
- `push_all_repos.py` - Batch commits and pushes multiple repositories
- `link_github.py` - Links existing GitHub repos to local projects
- `fix_large_repo.sh` - Helps clean repositories with large files

## Common Issues & Solutions

### GitHub Push Failures
- **HTTP 400 errors**: Usually caused by HTTPS authentication issues. Run `convert_to_ssh.py` to switch all repos to SSH
- **Large file errors**: Use `check_large_files.py` to find files >50MB, then remove them from git history
- **Repository exists**: The app will detect and link to existing repos automatically

### Azure DevOps Repos
- The app detects and properly displays Azure DevOps repositories
- Converts SSH URLs like `git@ssh.dev.azure.com:v3/org/project/repo` to web URLs
- Shows with blue "Azure" button instead of GitHub

### Project Detection
- Supports multiple languages: Python, JavaScript, TypeScript, Go, Rust, Java, Kotlin, Swift, Prolog, Solidity, etc.
- Detects project types based on marker files (package.json, pom.xml, go.mod, etc.)
- Automatically loads `.project-meta.json` files if present in project directories

## API Endpoints

- `GET /api/projects` - Returns all projects with annotations
- `POST /api/project/<name>/annotate` - Save project annotations
- `POST /api/project/<name>/create-github` - Create GitHub repo (handles existing repos)
- `POST /api/project/<name>/init-git` - Initialize git repository
- `POST /api/project/<name>/open-vscode` - Open project in VS Code
- `GET /api/project/<name>/files` - Get project file tree
- `GET /api/project/<name>/file/<path>` - Get file content

## Current Setup Notes

- Total projects in ~/Documents/dev: 72
- All projects have annotations
- 71 have git initialized (only `camundaclasses` doesn't)
- 34 have GitHub remotes
- SSH is now used for all GitHub connections to avoid auth issues

## GitHub Integration

Uses `gh` CLI for GitHub operations. Requires:
- `gh auth login` to be run first
- SSH keys set up for pushing
- For repo deletion, needs `delete_repo` scope: `gh auth refresh -h github.com -s delete_repo`

## UI Features

- **Filters**: By language, type, status (no git, no remote, no README)
- **Search**: Searches project names, notes, and tags
- **Visual indicators**:
  - Blue left border = has notes
  - Selected card = blue border + light blue background
  - GitHub button: black=public, purple=private, blue=Azure
  - Lock icon = private repo, box icon = public repo

## Development

```bash
cd ~/Documents/dev/project-viewer
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python app.py
```

Access at http://localhost:5000

---
> Source: [perbergman/project-viewer](https://github.com/perbergman/project-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
