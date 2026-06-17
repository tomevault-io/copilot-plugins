## aast-lms-sync

> Synchronize, categorize, and rename AAST LMS course materials into a structured local folder. Use when the user wants to sync their university course folders with the LMS, organizing them by Week and Type (Lectures, Sections, Assignments).


# lms-sync-skill

An AI-powered skill to synchronize and organize AAST LMS course materials.

## Description

This skill automates the management of university course materials by bridging the AAST LMS with your local filesystem. It intelligently scrapes course resources, categorizes them into `Lectures`, `Sections`, and `Assignments`, and applies a consistent `Week XX -` naming convention.

## Usage

### Synchronize a Course
Syncs a specific course using its LMS ID to a target local directory.
```bash
lms_sync <course_id> <local_dir_path>
```

### Discover Courses
Lists all currently enrolled courses and their corresponding IDs.
```bash
lms_discover
```

## Workflows

1. **Authentication**: Securely manages sessions using `session.json` or `lms_creds.txt`.
2. **Discovery**: Scrapes the LMS dashboard to identify enrolled courses and IDs.
3. **Scraping & Analysis**: Parses the course page to build a structural map of resources.
4. **Delta Synchronization**: Identifies new or modified files by comparing LMS metadata with local state.
5. **Categorization**: Sorts resources into subfolders based on content type (Lecture, Section, Assignment).
6. **Standardized Renaming**: Applies `Week XX - [Original Name]` formatting for better searchability.

## Troubleshooting

- **Authentication Errors**: Ensure `lms_creds.txt` is correctly formatted in the `Uni` folder.
- **Selector Mismatch**: If the LMS UI changes, the scraping selectors in `scripts/lms_sync.cjs` may need updates.
- **Dependency Issues**: Verify that Puppeteer and `gdown` are accessible in the environment.

---
> Source: [4awmy/aast-lms-sync](https://github.com/4awmy/aast-lms-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
