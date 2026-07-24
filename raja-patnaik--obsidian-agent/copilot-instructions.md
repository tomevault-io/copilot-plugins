## obsidian-agent

> This is the personal Obsidian vault for Raja. Claude Code agents should read this file first when working with this vault.

# CLAUDE.md — Personal Vault

This is the personal Obsidian vault for Raja. Claude Code agents should read this file first when working with this vault.

## Vault Info

- **Vault type**: personal
- **Owner**: (your name and email)
- **Path**: (update with actual path)
- **Companion vault**: work vault at (update with actual path)

## Installed Skills

- **kepano/obsidian-skills** — Official Obsidian plugin for file formats (markdown, Bases, Canvas, CLI). Use `obsidian-cli` commands (`obsidian read`, `obsidian create`, `obsidian search`) when Obsidian is running.
- **obsidian-vault-manager** — Vault structure, frontmatter schemas, workflows, reviews, maintenance.
- **QMD** (`@tobilu/qmd`) — Local semantic search. Use `qmd query -c personal "<question>"` for intelligent search across this vault. The index auto-updates after each agent response via a `Stop` hook.

## Folder Structure

```
00-inbox/        → Quick captures, unsorted notes
01-daily/        → Daily notes: 01-daily/YYYY/MM/YYYY-MM-DD.md
02-projects/     → Personal projects: 02-projects/project-name/
03-areas/        → Life areas: health/, finance/, learning/, home/
04-resources/    → Reference: articles/, bookmarks/, how-tos/
05-people/       → People/CRM: firstname-lastname.md
06-meetings/     → Catch-ups, appointments
07-ideas/        → Brain dumps, shower thoughts, sparks
08-archive/      → Completed/inactive items
09-journal/      → Long-form reflections, journaling
10-lists/        → Wishlists, gift ideas, restaurants, travel
_templates/      → Note templates
_attachments/    → Images, PDFs, files
```

## Rules for Agents

1. **Every note needs frontmatter** — See the skill reference for schemas per type
2. **Always update `modified`** — When editing any note, update the modified timestamp
3. **Use wikilinks** — Link to people with `[[firstname-lastname]]`, projects with `[[project-name]]`
4. **Daily note is the hub** — Link all new notes from the daily note
5. **Inbox is temporary** — Notes in `00-inbox/` should be processed within a day
6. **Never delete without asking** — Move to `08-archive/` instead
7. **Kebab-case filenames** — All lowercase, hyphens between words
8. **Respect privacy** — Personal vault content is private. Never reference it in work contexts.
9. **Gentle tone** — Journal entries and reflections are personal. Be supportive, not clinical.

## Common Tasks

### "What's on my plate today?" (personal)
1. Check today's daily note
2. Review personal projects (home improvement, learning goals, etc.)
3. Check upcoming events/appointments
4. Surface any follow-ups with friends/family

### "Brain dump" / "I have an idea"
1. Create in `07-ideas/idea-short-title.md`
2. Use idea template
3. Set status to `seed`
4. Capture raw thoughts without filtering
5. Link from daily note

### "Journal" / "I want to reflect"
1. Create in `09-journal/` with today's date
2. Use journal template
3. If a prompt is given, include it in frontmatter
4. Keep it free-flowing — don't impose structure on reflections

### "Add to my list"
1. Find or create the appropriate list in `10-lists/`
2. Append the new item
3. Lists include: restaurants, movies, books, gift ideas, travel wishlist, etc.

### "What did I learn about X?"
1. Search `04-resources/` and `03-areas/learning/`
2. Check for related ideas in `07-ideas/`
3. Synthesize and present with links

## Tags in Use

Personal vault uses these tag hierarchies:
- `#health/fitness`, `#health/nutrition`, `#health/mental`
- `#finance/budget`, `#finance/investing`, `#finance/goals`
- `#learning/course`, `#learning/book`, `#learning/skill`
- `#home/maintenance`, `#home/project`, `#home/garden`
- `#idea/seed`, `#idea/exploring`, `#idea/developing`
- `#journal/reflection`, `#journal/gratitude`, `#journal/goals`
- `#list/wishlist`, `#list/restaurants`, `#list/travel`, `#list/gifts`

## Dataview Queries

**Ideas by energy**: `FROM "07-ideas" WHERE status != "parked" SORT energy DESC`
**Reading list**: `FROM "04-resources" WHERE category = "book" AND status != "read" SORT rating DESC`
**Upcoming birthdays**: `FROM "05-people" WHERE birthday SORT birthday ASC`
**Recent journal entries**: `FROM "09-journal" SORT created DESC LIMIT 5`
**Active personal projects**: `FROM "02-projects" WHERE status = "active" SORT priority ASC`

---
> Source: [raja-patnaik/obsidian-agent](https://github.com/raja-patnaik/obsidian-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
