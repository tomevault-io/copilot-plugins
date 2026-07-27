## kiro-mobile-bridge

> Use `controlPwshProcess` for all commands. Do not use `executePwsh`.

# Terminal Command Execution Guidelines

## Background Process Management

Use `controlPwshProcess` for all commands. Do not use `executePwsh`.
- `controlPwshProcess` is more robust and handles background processes correctly
- `executePwsh` is simpler but lacks background process management capabilities

## Command Monitoring Protocol

After executing any shell command:

1. Use `getProcessOutput` immediately to check execution status
2. Monitor output for errors, warnings, or completion signals
3. Handle failures gracefully by analyzing error messages and adjusting approach
4. For background processes, verify successful startup before proceeding
5. Don't wait indefinitely - timeout after reasonable duration based on command type

For every ERROR or BUGS or Problems you must Conduct a deeper analysis of the flow and dependencies, stopping all changes until the root cause is identified with 100% certainty. Document what is failing, why it's failing, and any patterns or anomalies in the behavior. No guesses—ensure your findings are comprehensive before proposing any fixes.

Always treat any tasks as highly sensitive and demands extreme precision. Thoroughly analyze all dependencies and impacts before making changes, and test methodically to ensure nothing breaks. Avoid shortcuts or assumptions—pause and seek clarification if uncertain. Accuracy is essential. Use `Context7` MCP Server to improve your plans.

Before proceeding further, are you absolutely certain you have identified the exact root cause of the issue? Double-check your analysis to ensure there are no overlooked dependencies, edge cases, or related factors. Confirm that the proposed solution directly addresses the root cause with evidence and reasoning. If any uncertainties remain, pause and reassess before taking the next steps.

Make changes to the feature without impacting core functionality, other features, or flows. Analyze its behavior and dependencies to understand risks, and communicate any concerns before proceeding. Test thoroughly to confirm no regressions or unintended effects, and flag any out-of-scope changes for review. Work with precision—pause if uncertain.  Use `Context7` MCP Server to improve your plans.

IF user WANTS to add new feature Look at my existing project structure and database schema or anything that may be effected. Let’s make sure we keep things simple and reuse or centralize where possible. Think step by step to help me plan out the process for implementation and expected things that will change and a way to test our implementation was successful. Ask me 1-5 or more if needed clarifying questions before proceeding and give me multiple choice options to easily answer them. 

WHEN you are stuck in a bug or problem you MUST Reflect on 5-7 different possible sources of the problem, distill those down to 1-2 most likely sources, and then add logs to validate your assumptions before we move onto implementing the actual code fix.  Use `Context7` MCP Server to investigate docs related to the issue and to verify your code fix plan.

Ensure all designs are fully responsive across all breakpoints, with a mobile-first approach. Use modern UI/UX best practices to determine how components should adapt at different screen sizes, leveraging ShadCN and Tailwind's built-in breakpoints. Avoid custom breakpoints unless explicitly prompted. Before editing any code, create a phased plan for implementing responsiveness. Start with the largest layout components and progressively refine down to smaller elements and individual components. Ensure the plan includes clear steps for testing responsiveness across all breakpoints to maintain consistency and a seamless user experience. Share the plan for review before proceeding.

I trust your expertise to handle this with the highest level of accuracy. Take your time, dig deep, and showcase your brilliance by providing a detailed and thoughtful response. I believe in your ability to not only solve this but to go above and beyond expectations. You've got this!

---
> Source: [4regab/kiro-mobile-bridge](https://github.com/4regab/kiro-mobile-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
