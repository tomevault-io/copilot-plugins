## wizards-choice

> This document defines the core guidelines and best practices for making changes to the Wizard's Choice codebase. It consolidates information from the Deployment Guide, Game Design Document, UI Components, User Guide, and Code Documentation. **Any new or additional documentation files must also reside in the `/docs` directory.**


This document defines the core guidelines and best practices for making changes to the Wizard's Choice codebase. It consolidates information from the Deployment Guide, Game Design Document, UI Components, User Guide, and Code Documentation. **Any new or additional documentation files must also reside in the `/docs` directory.**

1. **Authoritative Context:**  
    Always consult the documentation in `/docs` (including the Deployment Guide, Game Design Document, UI Components, User Guide, Technical Documentation, and/or Code Documentation) as the single source of truth. Do not implement changes outside the context provided by these files. If uncertain, review the relevant document before proceeding.
    
2. **Verification:**  
    Confirm every detail with the existing documentation. Avoid assumptions or speculation—every decision must be evidence-based.
    
3. **Sequential Feature Development:**  
    Follow the sequential steps outlined in the project documents (e.g., in the Game Design Document and todo list). Complete each step in order, and update the corresponding documentation with a brief two-line summary (using a Markdown file, not a .mdc file) once a step is done.
    
4. **File-by-File Modifications:**  
    Make changes one file at a time. Present each change clearly so the user can spot errors without ambiguity.
    
5. **Concise Communication:**
    
    - Do not include apologies or self-referential commentary in code comments or documentation.
        
    - Avoid unnecessary summaries or feedback on your understanding unless explicitly requested.
        
6. **Respect the Existing Codebase:**
    
    - Preserve unrelated code and existing functionality.
        
    - Do not remove or alter code outside the scope of the requested changes.
        
    - Provide all modifications as a single complete update per file, not multiple partial edits. 
        
7. **No Extraneous Interactions:**
    
    - Do not ask for confirmation on details already documented.
        
    - Do not comment on the current implementation unless it is essential to explain the impact of a change.
        
8. **Direct References:**  
    Always link to the real files in the repository (e.g., using the paths in the `/docs` directory) rather than referring to generated or placeholder content.
    
9. **Code Quality Standards:**
    
    - Use descriptive and explicit variable names as detailed in the Code Documentation.
        
    - Adhere to the established coding style found in the project’s documentation (refer to UI Components and Code Documentation for guidelines).
        
    - Replace hard-coded values with named constants to avoid “magic numbers.”

    - Preferred case for code is camelCase class names and for file names the preference is kebab-case.
        
10. **Performance and Security:**
    
    - Prioritize performance optimizations and review the CHANGELOG for recent improvements.
        
    - Always consider security implications and implement robust error handling and logging.
        
11. **Testing and Robustness:**
    
    - Include appropriate unit tests for any new or modified functionality in line with the project’s testing standards.
        
    - Implement assertions and handle edge cases to validate assumptions early.
        
12. **Modular and Maintainable Design:**  
    Embrace modular design principles to enhance code reusability and maintainability. Follow the project’s structure and ensure that any new changes are compatible with the existing framework and versions (as noted in the README and Code Documentation).
    

---

By following these rules, you ensure that all changes are consistent, well-documented, and fully aligned with the Wizard's Choice project vision.

## User Rules 

1. "Use '/docs' as the Knowledge Base": Always refer to '/docs' to understand the context of the project. Do not code anything outside of the context provided in the '/docs' folder. This folder serves as the knowledge base and contains the fundamental rules and guidelines that should always be followed. If something is unclear, check this folder before proceeding with any coding.
    
2. "Verify Information": Always verify information from the context before presenting it. Do not make assumptions or speculate without clear evidence.
    
3. "For Feature Development": When implementing a new feature, develop a step by step concise TODO task list using the 'todo.md' in project root, strictly follow the steps outlined in 'todo.md'. Every step is listed in sequence, and each must be completed in order. After completing each step, update 'CHANGELOG.md' with the word "Done" and a two-line summary of what steps were taken. This ensures a clear work log, helping maintain transparency and tracking progress effectively.
    
4. "File-by-File Changes": Make all changes file by file and give the user the chance to spot mistakes.
    
5. "No Apologies": Never use apologies.
    
6. "No Understanding Feedback": Avoid giving feedback about understanding in the comments or documentation.
    
7. "No Whitespace Suggestions": Don't suggest whitespace changes.
    
8. "No Summaries": Do not provide unnecessary summaries of changes made. Only summarize if the user explicitly asks for a brief overview after changes.
    
9. "No Inventions": Don't invent changes other than what's explicitly requested.
    
10. "No Unnecessary Confirmations": Don't ask for confirmation of information already provided in the context.
    
11. "Preserve Existing Code": Don't remove unrelated code or functionalities. Pay attention to preserving existing structures.
    
12. "Single Chunk Edits": Provide all edits in a single chunk instead of multiple-step instructions or explanations for the same file.
    
13. "No Implementation Checks": Don't ask the user to verify implementations that are visible in the provided context. However, if a change affects functionality, provide an automated check or test instead of asking for manual verification.
    
14. "No Unnecessary Updates": Don't suggest updates or changes to files when there are no actual modifications needed.
    
15. "Provide Real File Links": Always provide links to the real files, not the context-generated file.
    
16. "No Current Implementation": Don't discuss the current implementation unless the user asks for it or it is necessary to explain the impact of a requested change.
    
17. "Check Context Generated File Content": Remember to check the context-generated file for the current file contents and implementations.
    
18. "Use Explicit Variable Names": Prefer descriptive, explicit variable names over short, ambiguous ones to enhance code readability.
    
19. "Follow Consistent Coding Style": Adhere to the existing coding style in the project for consistency.
    
20. "Prioritize Performance": When suggesting changes, consider and prioritize code performance where applicable.
    
21. "Security-First Approach": Always consider security implications when modifying or suggesting code changes.
    
22. "Test Coverage": Suggest or include appropriate unit tests for new or modified code.
    
23. "Error Handling": Implement robust error handling and logging where necessary.
    
24. "Modular Design": Encourage modular design principles to improve code maintainability and reusability.
    
25. "Version Compatibility": When suggesting changes, ensure they are compatible with the project's specific language or framework versions. If a version conflict arises, suggest an alternative.
    
26. "Avoid Magic Numbers": Replace hard-coded values with named constants to improve code clarity and maintainability.
    
27. "Consider Edge Cases": When implementing logic, always consider and handle potential edge cases.
    
28. "Use Assertions": Include assertions wherever possible to validate assumptions and catch potential errors early.

29. "Testing": I have npm run dev running in the terminal so i can test changes live, so i do not need you to ask to run the server to test changes. I will do that task myself after changes are made and compiled in the dev server. 
    
30. "No Lazy Solutions": Don't just ignore the problem and supersede it, or take shortcuts because that leaves unused wrong code bloating the codebase complicating further development. Do not create fresh css files to override css errors, find and fix the problem instead.

31. "Debloating IS Best": Always remove bloat code when you can refactor away unused code or optimize it to be less bloated and inefficient/confusing or messy.

32. "Diagram Processes": Always use mermaid diagrams to illustrate processes and workflows in the codebase when updating relevant docs. Keep docs/process_maps.md up to date with each user verified completed task.
    

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/magejosh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
