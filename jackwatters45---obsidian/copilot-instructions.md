## obsidian

> This is an **Obsidian vault** for personal knowledge management, not a code repository.

# AGENTS.md - Jack's Obsidian Vault

This is an **Obsidian vault** for personal knowledge management, not a code repository.
AI agents should treat this as a structured note-taking system with specific conventions.

## Vault Structure

```
/                       # Root - personal notes, screenshots, PDFs
├── Categories/         # Index pages for each content type
├── Clippings/          # Web clippings and articles
├── Daily/              # Daily notes (YYYY-MM-DD.md format)
├── Files/              # Attachments and files
├── References/         # Books, authors, and reference material
├── Templates/          # Templates for all note types
│   └── Bases/          # Dataview base queries (.base files)
├── Weekly/             # Weekly notes (YYYY-Www.md format)
├── .clipper-templates/ # Obsidian Web Clipper templates

```

## Content Guidelines for AI Agents

### CRITICAL: File Modification Rules

**NEVER use the Write tool or any command that recreates files.** Obsidian tracks file creation time (ctime) for indexing and queries. If you use Write instead of Edit, or use shell commands like `cat > file` or heredocs, Obsidian will detect the file as newly created, breaking:

- Daily note "Created" views that filter by ctime
- Any base queries using `file.ctime`
- File history and modification tracking

**ALWAYS use the Edit tool** to modify existing files. This preserves the original file's metadata.

### DO:

- Preserve existing frontmatter structure when editing notes
- Use proper Obsidian link syntax `[[]]` for internal references
- Match existing tag conventions (lowercase, hyphenated)
- Create new notes in appropriate folders based on content type
- Use templates when creating new structured notes
- Respect the YYYY-MM-DD date format consistently

### DON'T:

- **USE THE WRITE TOOL ON EXISTING FILES** - This recreates the file and breaks Obsidian's ctime tracking
- Use `cat >`, heredocs, or any shell command that overwrites files - same issue as Write tool
- Modify `.obsidian/` configuration files
- Change `.base` query files without explicit request
- Remove or restructure existing frontmatter fields
- Create duplicate entries without checking existing notes
- Add images directly to root (use Files/ folder)
- Change established naming conventions

### When Creating New Notes:

1. Determine the correct category/folder
2. Use the appropriate template if one exists
3. Fill frontmatter with available information
4. Add proper category links (e.g., `categories: ["[[Books]]"]`)
5. Use consistent date formatting

## Plugin Ecosystem

The vault uses Obsidian plugins. Key ones based on config:

- Daily Notes (configured for `/Daily/` folder)
- Templater (for template processing)
- Obsidian Web Clipper (browser extension)

## Working with This Vault

### Reading Notes

- Check frontmatter for metadata
- Look for embedded bases (`![[*.base]]`) for dynamic content
- Follow `[[links]]` to understand relationships

### Modifying Notes

- Preserve existing structure
- Add to appropriate sections
- Update frontmatter dates when relevant

### Creating Notes

- Always check if similar note exists first
- Use correct folder location
- Apply appropriate template
- Set proper categories and tags

## QMD Vault Search

Use `qmd` CLI for semantic search across the vault:

```bash
qmd search "keyword"       # Fast BM25 keyword search
qmd vsearch "concept"      # Semantic vector search
qmd query "question"       # Hybrid + reranking (best quality)
qmd get "Daily/2026-01-15.md"  # Get full document
qmd get "#abc123"          # Get by docid from search results
qmd multi-get "Daily/2026-01*.md"  # Get multiple by glob
```

Options:

- `-n 10` - Number of results
- `-c vault` - Filter to vault collection
- `--min-score 0.3` - Score threshold
- `--json` / `--md` - Output format for processing
- `--full` - Include full document content

Use `qmd query` when searching for concepts or answering questions about vault content.
Use `qmd search` for quick keyword lookups.

## Obsidian CLI

Official CLI (requires Obsidian 1.12+, app must be running). Prefer this over direct file manipulation when Obsidian is running — it respects plugins, templates, and vault indexing.

```bash
# Daily notes
obsidian daily                                    # Open today's daily note
obsidian daily:append content="- [ ] Buy milk"    # Append to daily note
obsidian daily:read                               # Read daily note contents

# Files
obsidian read file=Recipe                         # Read file (wikilink-style resolution)
obsidian read path="Templates/Recipe.md"          # Read file by exact path
obsidian create name="Trip to Paris" template=Travel  # Create from template
obsidian open file=Recipe                         # Open file in Obsidian
obsidian append file=Note content="New line"      # Append to file
obsidian prepend file=Note content="Top line"     # Prepend after frontmatter
obsidian move file=Note to="Archive/"             # Move/rename file
obsidian delete file=Note                         # Trash file

# Search & navigation
obsidian search query="meeting notes"             # Search vault
obsidian search query="TODO" total                # Count matches
obsidian tags all counts                          # All tags with counts
obsidian tasks daily todo                         # Incomplete tasks from daily note
obsidian task daily line=3 toggle                 # Toggle task completion
obsidian backlinks file=Recipe                    # List backlinks
obsidian outline file=Recipe                      # Show headings

# Properties
obsidian properties file=Recipe                   # List file properties
obsidian property:set name=status value=done      # Set property
obsidian property:read name=rating file=Recipe    # Read property value

# Plugins & themes
obsidian plugins                                  # List installed plugins
obsidian plugin:enable id=dataview                # Enable plugin
obsidian plugin:install id=my-plugin enable       # Install + enable

# Developer / automation
obsidian eval code="app.vault.getFiles().length"  # Run JS in Obsidian
obsidian dev:screenshot path=screenshot.png       # Screenshot the app
obsidian dev:console                              # Show console messages
obsidian dev:errors                               # Show JS errors

# Vault targeting
obsidian vault=Notes daily                        # Target specific vault
obsidian vault                                    # Show vault info
```

Key patterns:
- `file=<name>` resolves like wikilinks (name only, no path/ext needed)
- `path=<path>` requires exact path from vault root
- Most commands default to the active file if no file/path given
- Add `silent` flag to suppress opening files, `--copy` to copy output to clipboard
- Add `total` flag to get counts instead of lists
- Multiline content: use `\n` for newlines, `\t` for tabs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jackwatters45)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jackwatters45)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
