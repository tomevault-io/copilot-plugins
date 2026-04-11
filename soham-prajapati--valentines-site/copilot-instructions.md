## valentines-site

> 1. **Language**: Always respond in English only. Never switch languages.

# Soham's Local Gemini Rules
# Created local .gemini.md — customize it for this project.

## Universal Workflow Rules

1. **Language**: Always respond in English only. Never switch languages.

2. **New Project Setup**: When starting in any new project directory,
   check if a local .gemini.md exists in that directory.
   If it does NOT exist, create one by copying this global file
   and tell the user: "Created local .gemini.md — customize it 
   for this project."

3. **Output Logging**: After completing any task write TWO files
   in the current project root. Both files must contain the 
   COMPLETE raw output — nothing summarized, nothing cut off.

   FILE A: last-output.md (full plain text)

   FILE B: last-output.html using this template:
   (Always run: `date '+%A %B %d %Y %I:%M %p'` for the timestamp value)
   <!DOCTYPE html>
   <html>
   <head>
   <meta charset="UTF-8">
   <title>Soupz Output</title>
   <style>
     *{box-sizing:border-box;margin:0;padding:0}
     body{background:#111;color:#eee;font-family:monospace;
          padding:20px;max-width:1000px;margin:0 auto}
     h1{color:#2FA7A0;margin-bottom:5px}
     .ts{color:#666;font-size:11px;margin-bottom:15px}
     .tabs{display:flex;gap:8px;margin-bottom:20px;
           flex-wrap:wrap}
     .tab{background:#1a1a1a;border:1px solid #333;
          color:#aaa;padding:6px 14px;border-radius:6px;
          cursor:pointer;font-family:monospace;font-size:13px}
     .tab.active{background:#2FA7A0;color:#111;
                 border-color:#2FA7A0;font-weight:bold}
     .panel{display:none}
     .panel.active{display:block}
     .done{color:#4CAF50}.partial{color:#E86A33}
     .failed{color:#D64545}
     pre{background:#1a1a1a;padding:15px;border-radius:8px;
         overflow-x:auto;white-space:pre-wrap;
         word-wrap:break-word;font-size:13px;
         line-height:1.5}
     .section{margin:20px 0;border-left:3px solid #2FA7A0;
              padding-left:15px}
     .meta{display:grid;grid-template-columns:repeat(3,1fr);
           gap:10px;margin:15px 0}
     .meta-item{background:#1a1a1a;padding:10px;
                border-radius:6px}
     .meta-label{color:#666;font-size:11px;margin-bottom:4px}
     .meta-value{color:#eee;font-size:15px;font-weight:bold}
     .badge{display:inline-block;padding:2px 8px;
            border-radius:4px;font-size:11px;margin-left:8px}
     .badge-done{background:#1a3a1a;color:#4CAF50}
     .badge-partial{background:#3a2a1a;color:#E86A33}
     .badge-failed{background:#3a1a1a;color:#D64545}
   </style>
   </head>
   <body>
   <h1>🍜 Soupz Output</h1>
   <p class="ts">Updated: [TIMESTAMP] — refresh to update</p>
   
   <div class="tabs" id="tabs"></div>
   <div id="panels"></div>
   
   <script>
   const tasks = [TASKS_JSON];
   
   const tabsEl = document.getElementById('tabs');
   const panelsEl = document.getElementById('panels');
   
   tasks.forEach((t, i) => {
     const badgeClass = t.status === 'done' ? 'badge-done' 
       : t.status === 'partial' ? 'badge-partial' : 'badge-failed';
     const tab = document.createElement('button');
     tab.className = 'tab' + (i === 0 ? ' active' : '');
     tab.innerHTML = t.agent + 
       '<span class="badge ' + badgeClass + '">' + 
       t.status + '</span>';
     tab.onclick = () => {
       document.querySelectorAll('.tab')
         .forEach(x => x.classList.remove('active'));
       document.querySelectorAll('.panel')
         .forEach(x => x.classList.remove('active'));
       tab.classList.add('active');
       document.getElementById('panel-'+i)
         .classList.add('active');
     };
     tabsEl.appendChild(tab);
   
     const panel = document.createElement('div');
     panel.className = 'panel' + (i === 0 ? ' active' : '');
     panel.id = 'panel-' + i;
     panel.innerHTML = `
       <div class="section">
         <h2>${t.task}</h2>
       </div>
       <div class="meta">
         <div class="meta-item">
           <div class="meta-label">Files Changed</div>
           <div class="meta-value">${t.files}</div>
         </div>
         <div class="meta-item">
           <div class="meta-label">Duration</div>
           <div class="meta-value">${t.duration}</div>
         </div>
         <div class="meta-item">
           <div class="meta-label">Agent</div>
           <div class="meta-value">${t.agent}</div>
         </div>
       </div>
       <div class="section">
         <h2>Files Modified</h2>
         <pre>${t.filesModified}</pre>
       </div>
       <div class="section">
         <h2>Full Raw Output</h2>
         <pre>${t.output}</pre>
       </div>
       <div class="section">
         <h2>Errors</h2>
         <pre>${t.errors || 'NONE'}</pre>
       </div>
     `;
     panelsEl.appendChild(panel);
   });
   </script>
   </body>
   </html>

   IMPORTANT: The pre tag must never be height-limited.
   All content must be visible without scrolling inside the box.
   The full output goes in — no truncation ever.

4. **Task Tracking**: Every project has a tasks.md with these states:
   - [ ] pending
   - [/] in progress  
   - [x] completed
   - [!] blocked
   After every task update tasks.md accordingly.

5. **Task Locking**: Before starting ANY task, mark it [/] FIRST.
   If a task shows [/] already, skip it — another agent has it.

6. **Never Assume**: Confirm before any irreversible deletion 
   or overwrite.

10. **Git Commits**: Never stage or commit automatically unless
    explicitly asked. When asked to commit:
    - Run git config user.name and git config user.email first
    - Use whatever identity is already configured, never change it
    - NEVER add these to any commit:
      * "Generated by Gemini"
      * "Co-authored-by: Google Gemini"
      * "🤖" or any robot emoji in commit message
      * Any mention of AI, Gemini, Claude, or LLM
    - Conventional commit style is fine (feat:, fix:, chore: etc)
      but zero AI attribution anywhere
    - Write commit messages as if Soham personally wrote 
      every line of code

   CRITICAL: last-output.html must contain ZERO JavaScript
   and ZERO template variables like ${} or [PLACEHOLDER].
   Write all values as plain hardcoded HTML text directly.
   No scripts, no dynamic content, no template literals ever.
\n11. **Strict Functionality**: Never add non-functional UI elements, mock text, or fake links. Never do anything without it being fully functional. Do not add random sidebar items or nav links that lead nowhere.\n12. **Brutal Honesty**: Never lie, never ever lie. Be brutally honest. No sugarcoating. If something is realistic, it's realistic; if it isn't, it isn't. Do not hallucinate completions or state things are working when they are just mockups.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Soham-Prajapati)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Soham-Prajapati)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
