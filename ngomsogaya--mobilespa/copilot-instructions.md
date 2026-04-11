## mobilespa

> 🛠️ Developer Debugging & Resolution Guide

🛠️ Developer Debugging & Resolution Guide

Step 2: Implementing the Permanent Fix (Renaming)
To force an undeniable cache refresh and establish a better naming convention, proceed with the renaming as requested.

2.1. File Renaming
New File Name: Change the name of the PDF file from the old name (e.g., old-name.pdf) to:

brochure.pdf

Upload: Upload the renamed file (brochure.pdf) to the correct directory on the live server.

Deletion (Optional but Recommended): Delete the old file name (e.g., old-name.pdf) from the server to prevent any confusion or accidental links in the future.

2.2. Reference Audit & Update
The developer must now locate and update every single place the old file was referenced.

Codebase Search: Use the project's search function (e.g., a "Find in Files" feature in their code editor) to search the entire repository for the old PDF file name (e.g., old-name.pdf).

Target: Search for the string old-name.pdf.

Common Locations to Check:

HTML Links: <a href="/path/to/old-name.pdf" ... >Download</a>

JavaScript: Any dynamic link generation or functions that handle the download.

CSS: Occasionally, files are referenced in CSS (background-image).

CMS: If using a Content Management System (CMS), check page builders, media libraries, and menu settings.

Update: Change all instances of the old file name to the new name:

Example: Change /path/to/old-name.pdf to /path/to/brochure.pdf.

Deployment: Commit the code changes and deploy them to the live server.

2.3. Final Verification
Clear the browser cache once more.

Visit the live site and click the download button.

Result: The new PDF content should now be visible and downloadable.

Step 3: Best Practice for Future Updates
To prevent this issue from ever happening again, the developer should implement a versioning or cache-busting strategy for static assets like PDFs.

The two main methods are:

File Naming Versioning (Recommended):

Instead of brochure.pdf, name the file with a version number or date, and update the links in the code every time.

Example:

Old: brochure-v1.pdf

New: brochure-v2.pdf

New: brochure-2025-Q1.pdf

Benefit: This guarantees a new file name, which always forces a cache clear across all layers.

Automated Cache-Busting (Advanced):

For the link, automatically append a unique query string based on the file's modification time during the build process.

Example (in the code): <a href="/path/to/brochure.pdf?v=1704067200" ... >

Where 1704067200 is a timestamp that changes when the file is updated.

Summary Action: The developer should prioritize Step 2 (Renaming and Updating References) as it is the most robust and immediate solution for a stubborn caching issue.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NgomsoGaya)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NgomsoGaya)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
