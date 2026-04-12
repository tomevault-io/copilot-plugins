## it-sikkerhed-notes

> Always browse the file _THE_NOTE_ for new content that should be moved to its appropriate location

Rule Name: the-note.mdc
Description: Always browse the file _THE_NOTE_ for new content that should be moved to its appropriate location
## When This Rule Applies
- **Path Pattern**: `**/*.md`
- **Match Pattern**: When a user writes "update the note"

## Instructions

**IMPORTANT**: When this rule is triggered, you MUST start your response by stating: "I'm checking the contents of _THE_NOTE_ and moving the content into the appropriate folders/files. This acknowledgment is required before proceeding with any research or response.

When a user types 'update the note', follow these steps:

1. First check the _THE_NOTE_.md file for new content
2. Extract keywords and information about the subject the notes are describing
3. Then perform a semantic search across all notes, articles, and resources for relevant information about the topic
3. Combine the dictionary definition with the contextual information from other files
4. Move the information from the _THE_NOTE_.md file into the appropriate location whether it's into an existing file, new files in an axisting location or a new file in a new directory.
5. Present the user with a structured view of the files and locations creates and/or updated
6. After completing file operations, automatically update the project index by:
   - Checking the current project structure using list_dir on key directories
   - Comparing with what's in _content/project_index.md
   - Updating the index to reflect new files or directories created
   - Including a summary of index updates in your response

## Process

- Parse the query to identify the key term or concept being asked about
- Group the content of the file _THE_NOTE_ into appropriate subject groupings
- Foreach grouping do the following
    - Check the dictionary (_content/dictionary.md) for formal definitions
    - Use codebase_search to find relevant information across all files
    - Read referenced files to extract detailed information
    - Update or create files and move the information
- Synthesize all findings into a comprehensive response
- Delete the moved content from the file _THE_NOTE_.md by:
    - First reading the current content of _THE_NOTE_.md
    - Using edit_file to replace all content with an empty string
    - Verifying deletion was successful by reading the file again
- Perform index update by:
    - Examine key directories: Notes, Articles, OWASP Cheatsheets, _content
    - Check for any new directories or content created in the current operation
    - Read the current project_index.md file
    - Update the index to include new content areas or files just created
    - Maintain the existing organization and formatting of the index

## Example:

**Query**: "Update the note"

**Expected Response**:  
A comprehensive answer that first provides the subject of the information groupings in the _THE_NOTE_.md file and where the information has been relocated to. The response should also include a brief summary of any index updates made to reflect the new files or locations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BjornFrierPedersen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BjornFrierPedersen)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
