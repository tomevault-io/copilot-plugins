## main-rules

> use makefiles for application operations, running, testing and similar.

use makefiles for application operations, running, testing and similar.

always read logs in logs/ folder to validate the runtime quality and operational stability of the solution stack


ensure you use 'launch-all' to start both frontend and backend where appropriate

do not use image or vide placeholders. implement real functionality. (i.e. no BigBuckBunny placeholder)

Check Frontend and Backend logs before and after implementing a fix, to validate the solution

python 'google.generativeai' library is now deprecated. use the modern google.genai library instead!

run the full stack with launch-all for consinstent runtime behaviour during run validation

do not create or use mocks, or stubs, unleass this is aprt of the test suite.

Use Graceful error exception handling highlighing the missing functionality where appropriate

You must Use Real, complete, and validate, tested feature and capability implemetation 

Use Makefile targets for runtime testing and validation.

read application logs to validate solution is compelte is valid

Ensure applicaiton emits the relevant debug level logging to validate functionality and prove capability implementation

Read the documentation and readme files to understand the solution architecture and solution intent before proceeding with any further additional code improvements or functional extensions to the solution.

the app is on port 8080. ensure this is consistent and well documented.

use system 'date' command to ascertain the latest datetime before writing date stamps into any files 

always review the documentation first and evaluate the solution maturity and completeness. 

assess if the functionality and features have been implemented sucessfully by running the appropriate, relevant quick tests. 

Mark the TODO/EPIC activities as done once relevant features/functioanltiy has been implemented

Commit and push once the quick testing and documentation  update tasks are performed

We are working towards MVP grade solution. 

ensure you regularly quick test and validate the solution architecture, before committing and pushing the changes to github

do not use stubs or mock data, as we target a functionally complete solution capability

use python3

Target production state would be a Hosted-on Google Cloud platform. Consider that in the longer solution maturity roadmpa post MVP.

do no hardcode what should be environmental values

this solution should work on the local machine via makefile targets but also be accessible and compatible when deploying into google cloud runtime, as a cloud run v2 in the roadmap toward production release

use the backend/.env environment file for gemini api keys and gemini model names

**SUBMISSION COMPLETENESS OVER NEW FEATURES**
   - Focus on polishing existing complete functionality
   - No new major features - only submission-critical improvements
   - Maintain current working state - no breaking changes
   - Document first, then implement minimal changes

**TECHNICAL REQUIREMENTS COMPLIANCE**
   - Google ADK Framework 1.0.0+ 
   - Multi-agent system 
   - Sequential workflow pattern 
   - Production-ready code 
   - Clean, well-documented codebase

---
> Source: [jpantsjoha/Agentic-Marketing-Campaign-Generator](https://github.com/jpantsjoha/Agentic-Marketing-Campaign-Generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
