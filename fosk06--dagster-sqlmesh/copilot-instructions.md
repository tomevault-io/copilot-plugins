## dagster-sqlmesh

> Documentation Workflow and Deployment Guide


# Documentation Workflow and Deployment Guide

This rule explains how to update, build, and deploy documentation for the dg-sqlmesh project.

## 📚 Documentation Structure

The documentation is built using **MkDocs** with the **Material theme** and is located in the `docs/` directory:

- `docs/index.md` - Homepage with project overview
- `docs/getting-started/` - Installation and setup guides
- `docs/user-guide/` - Core concepts and architecture
- `docs/examples/` - Code examples and usage patterns
- `docs/development/` - Contributing guidelines

## 🚀 Local Development

### Start Development Server

```bash
make docs-serve
```

This starts MkDocs on `http://localhost:8000` with live reload.

### Build Documentation Locally

```bash
make docs-build
```

Builds the documentation site in the `site/` directory.

### Clean Build Artifacts

```bash
make docs-clean
```

Removes the `site/` directory and build artifacts.

### Full Documentation Workflow

```bash
make docs
```

Runs: clean → build → validation check

## 🔧 Make Commands

The [Makefile](mdc:Makefile) provides several documentation-related commands:

- **`docs-serve`** - Start development server on localhost:8000
- **`docs-build`** - Build documentation site
- **`docs-clean`** - Clean build artifacts
- **`docs-deploy`** - Prepare for deployment (clean + build)
- **`docs-quality`** - Check documentation quality and links
- **`docs`** - Complete workflow (clean + build + check)

## 🌐 GitHub Pages Deployment

### Automatic Deployment

Documentation is automatically deployed to GitHub Pages via the [GitHub Actions workflow](mdc:.github/workflows/docs.yml).

**Triggers:**

- Push to `main` branch
- Changes to `docs/**`, `mkdocs.yml`, or `.github/workflows/docs.yml`
- Manual workflow dispatch

**Deployment URL:** `https://fosk06.github.io/dagster-sqlmesh/`

### Workflow Steps

1. **Build** - Installs dependencies and builds MkDocs site
2. **Setup Pages** - Configures GitHub Pages with `enablement: true`
3. **Upload** - Uploads built site as artifact
4. **Deploy** - Deploys to GitHub Pages (only on `main` branch)

## 📝 Updating Documentation

### 1. Make Changes

Edit files in the `docs/` directory or `mkdocs.yml`

### 2. Test Locally

```bash
make docs-serve  # View changes in browser
make docs-build  # Verify build works
```

### 3. Commit and Push

```bash
git add docs/ mkdocs.yml
git commit -m "docs: update [description of changes]"
git push origin main
```

### 4. Automatic Deployment

The GitHub Actions workflow will automatically:

- Build the documentation
- Deploy to GitHub Pages
- Update the live site

## ⚠️ Important Notes

- **No manual deployment needed** - GitHub Actions handles everything
- **Only deploys from `main`** - PRs only build, don't deploy
- **Live reload** - Use `make docs-serve` for development
- **Build validation** - Use `make docs-quality` to check for issues

## 🐛 Troubleshooting

### Build Fails

```bash
make docs-clean  # Clean artifacts
make docs-build  # Try building again
```

### Development Server Issues

```bash
# Kill any existing processes
pkill -f mkdocs
# Start fresh
make docs-serve
```

### GitHub Pages Not Updating

- Check GitHub Actions workflow status
- Verify changes are on `main` branch
- Wait a few minutes for deployment to complete

## 📋 Best Practices

1. **Always test locally** with `make docs-serve` before pushing
2. **Use descriptive commit messages** starting with `docs:`
3. **Keep documentation in sync** with code changes
4. **Validate builds** with `make docs-quality`
5. **Check deployment** after pushing to `main`

## 🔗 Related Files

- [mkdocs.yml](mdc:mkdocs.yml) - MkDocs configuration
- [.github/workflows/docs.yml](mdc:.github/workflows/docs.yml) - Deployment workflow
- [Makefile](mdc:Makefile) - Build commands
- [docs/](mdc:docs/) - Documentation source files

## 📖 Documentation Content Guidelines

### Writing Style

- Use clear, concise language
- Include code examples where helpful
- Use emojis sparingly but effectively
- Maintain consistent formatting

### Code Examples

- Use proper syntax highlighting
- Include complete, runnable examples
- Show both simple and advanced use cases
- Test all code examples locally

### Structure

- Start with overview and quick start
- Progress from basic to advanced concepts
- Include troubleshooting sections
- Provide clear navigation between sections

## 🎨 MkDocs Configuration

### Theme Settings

The documentation uses Material for MkDocs with:

- **Navigation**: Sidebar with search
- **Responsive design**: Mobile and desktop optimized
- **Code highlighting**: Syntax highlighting for all languages
- **Search**: Full-text search across documentation

### Plugins

- **git-revision-date-localized**: Shows last update dates
- **minify**: Optimizes HTML output
- **search**: Full-text search functionality

## 🚀 Advanced Features

### Custom CSS/JS

Add custom styling in `docs/stylesheets/extra.css`:

```css
/* Custom styles for documentation */
.version-info {
  background: #f0f0f0;
  padding: 1rem;
  border-radius: 4px;
}
```

### Custom Admonitions

Use Material theme admonitions:

```markdown
!!! note "Note"
This is a note with custom styling.

!!! warning "Warning"
This is a warning message.

!!! tip "Pro Tip"
This is a helpful tip.
```

### Navigation Customization

Customize navigation in `mkdocs.yml`:

```yaml
nav:
  - Home: index.md
  - Getting Started:
      - Installation: getting-started/installation.md
      - Quick Start: getting-started/quick-start.md
  - User Guide: user-guide/core-concepts.md
  - Examples: examples/basic-usage.md
  - Development: development/contributing.md
```

## 🔍 Quality Assurance

### Link Validation

```bash
make docs-quality
```

Checks for broken links and build issues.

### Content Review

- Review all documentation before pushing
- Ensure examples are up-to-date
- Check for typos and formatting issues
- Verify all links work correctly

### Performance

- Keep images optimized
- Minimize custom CSS/JS
- Use efficient Markdown syntax
- Regular cleanup of unused files

# Documentation Workflow and Deployment Guide

This rule explains how to update, build, and deploy documentation for the dg-sqlmesh project.

## 📚 Documentation Structure

The documentation is built using **MkDocs** with the **Material theme** and is located in the `docs/` directory:

- `docs/index.md` - Homepage with project overview
- `docs/getting-started/` - Installation and setup guides
- `docs/user-guide/` - Core concepts and architecture
- `docs/examples/` - Code examples and usage patterns
- `docs/development/` - Contributing guidelines

## 🚀 Local Development

### Start Development Server

```bash
make docs-serve
```

This starts MkDocs on `http://localhost:8000` with live reload.

### Build Documentation Locally

```bash
make docs-build
```

Builds the documentation site in the `site/` directory.

### Clean Build Artifacts

```bash
make docs-clean
```

Removes the `site/` directory and build artifacts.

### Full Documentation Workflow

```bash
make docs
```

Runs: clean → build → validation check

## 🔧 Make Commands

The [Makefile](mdc:Makefile) provides several documentation-related commands:

- **`docs-serve`** - Start development server on localhost:8000
- **`docs-build`** - Build documentation site
- **`docs-clean`** - Clean build artifacts
- **`docs-deploy`** - Prepare for deployment (clean + build)
- **`docs-quality`** - Check documentation quality and links
- **`docs`** - Complete workflow (clean + build + check)

## 🌐 GitHub Pages Deployment

### Automatic Deployment

Documentation is automatically deployed to GitHub Pages via the [GitHub Actions workflow](mdc:.github/workflows/docs.yml).

**Triggers:**

- Push to `main` branch
- Changes to `docs/**`, `mkdocs.yml`, or `.github/workflows/docs.yml`
- Manual workflow dispatch

**Deployment URL:** `https://fosk06.github.io/dagster-sqlmesh/`

### Workflow Steps

1. **Build** - Installs dependencies and builds MkDocs site
2. **Setup Pages** - Configures GitHub Pages with `enablement: true`
3. **Upload** - Uploads built site as artifact
4. **Deploy** - Deploys to GitHub Pages (only on `main` branch)

## 📝 Updating Documentation

### 1. Make Changes

Edit files in the `docs/` directory or `mkdocs.yml`

### 2. Test Locally

```bash
make docs-serve  # View changes in browser
make docs-build  # Verify build works
```

### 3. Commit and Push

```bash
git add docs/ mkdocs.yml
git commit -m "docs: update [description of changes]"
git push origin main
```

### 4. Automatic Deployment

The GitHub Actions workflow will automatically:

- Build the documentation
- Deploy to GitHub Pages
- Update the live site

## ⚠️ Important Notes

- **No manual deployment needed** - GitHub Actions handles everything
- **Only deploys from `main`** - PRs only build, don't deploy
- **Live reload** - Use `make docs-serve` for development
- **Build validation** - Use `make docs-quality` to check for issues

## 🐛 Troubleshooting

### Build Fails

```bash
make docs-clean  # Clean artifacts
make docs-build  # Try building again
```

### Development Server Issues

```bash
# Kill any existing processes
pkill -f mkdocs
# Start fresh
make docs-serve
```

### GitHub Pages Not Updating

- Check GitHub Actions workflow status
- Verify changes are on `main` branch
- Wait a few minutes for deployment to complete

## 📋 Best Practices

1. **Always test locally** with `make docs-serve` before pushing
2. **Use descriptive commit messages** starting with `docs:`
3. **Keep documentation in sync** with code changes
4. **Validate builds** with `make docs-quality`
5. **Check deployment** after pushing to `main`

## 🔗 Related Files

- [mkdocs.yml](mdc:mkdocs.yml) - MkDocs configuration
- [.github/workflows/docs.yml](mdc:.github/workflows/docs.yml) - Deployment workflow
- [Makefile](mdc:Makefile) - Build commands
- [docs/](mdc:docs/) - Documentation source files

## 📖 Documentation Content Guidelines

### Writing Style

- Use clear, concise language
- Include code examples where helpful
- Use emojis sparingly but effectively
- Maintain consistent formatting

### Code Examples

- Use proper syntax highlighting
- Include complete, runnable examples
- Show both simple and advanced use cases
- Test all code examples locally

### Structure

- Start with overview and quick start
- Progress from basic to advanced concepts
- Include troubleshooting sections
- Provide clear navigation between sections

## 🎨 MkDocs Configuration

### Theme Settings

The documentation uses Material for MkDocs with:

- **Navigation**: Sidebar with search
- **Responsive design**: Mobile and desktop optimized
- **Code highlighting**: Syntax highlighting for all languages
- **Search**: Full-text search across documentation

### Plugins

- **git-revision-date-localized**: Shows last update dates
- **minify**: Optimizes HTML output
- **search**: Full-text search functionality

## 🚀 Advanced Features

### Custom CSS/JS

Add custom styling in `docs/stylesheets/extra.css`:

```css
/* Custom styles for documentation */
.version-info {
  background: #f0f0f0;
  padding: 1rem;
  border-radius: 4px;
}
```

### Custom Admonitions

Use Material theme admonitions:

```markdown
!!! note "Note"
This is a note with custom styling.

!!! warning "Warning"
This is a warning message.

!!! tip "Pro Tip"
This is a helpful tip.
```

### Navigation Customization

Customize navigation in `mkdocs.yml`:

```yaml
nav:
  - Home: index.md
  - Getting Started:
      - Installation: getting-started/installation.md
      - Quick Start: getting-started/quick-start.md
  - User Guide: user-guide/core-concepts.md
  - Examples: examples/basic-usage.md
  - Development: development/contributing.md
```

## 🔍 Quality Assurance

### Link Validation

```bash
make docs-quality
```

Checks for broken links and build issues.

### Content Review

- Review all documentation before pushing
- Ensure examples are up-to-date
- Check for typos and formatting issues
- Verify all links work correctly

### Performance

- Keep images optimized
- Minimize custom CSS/JS
- Use efficient Markdown syntax
- Regular cleanup of unused files

---
> Source: [fosk06/dagster-sqlmesh](https://github.com/fosk06/dagster-sqlmesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
