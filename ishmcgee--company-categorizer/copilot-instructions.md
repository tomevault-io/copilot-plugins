## company-categorizer

> - Operating System: Windows 10

# Company Categorizer Project Rules

## Environment
- Operating System: Windows 10
- Shell: PowerShell
- Provide Windows-specific instructions only

## Code Organization
- Keep the codebase as simple as possible
- Use only a single code file until separate files become absolutely necessary
- Avoid unnecessary file proliferation
- Use organized folder structure:
  - `input/` folder for all input files (CSV, PDF, etc.)
  - `output/` folder for all generated files (CSV results, progress files, backups)
  - Project root for main scripts and configuration

## Development Practices
- Do not simulate or execute test code in the context window
- Always create test scripts first and use those to execute the code
- When suggesting tests, provide the actual command for the user to run
- Focus on practical, runnable solutions
- Prioritize functionality over complexity
- Make code easy to understand and modify 

## Code Quality
- Ensure all code can be run immediately by the user, and generate the appropriate output to allow the user to immediately run the code
- Include all necessary imports and dependencies
- Add appropriate error handlinge
- Follow Python best practices when applicable

## Communication
- Be direct and concise
- Focus on actionable solutions
- Provide clear instructions for running code
- Use Windows-appropriate file paths and commands

## Project-Specific Notes
- We are creating a script that will accept a list of company names in CSV format and categorize companies according to the following rules
- We are primarily interested in identifying which companies are healthcare technology companies
- Healthcare technology companies are defined as follows:
- 1. software companies that primarily serve the healthcare end markets
- 2. healthcare companies that derive significant value for thier customers via software the company developed in house (vs. 3rd party tools)
- 3. Not pharma, biotech, medtech, or med device companies
- We will generate the following fields and values and output to a new CSV file:
- Company name (repeated from input)
- Health tech: values will be Health tech or Not health tech
- Company description:  This will be 1 simple bullet structure as follows: Provides [services / products] to the [end market].  e.g. Availity provides revenue cycle and claims processing services to healthcare providers and payors". The company description should be brief and identify the most prominent products / services provided and end markets serviced by the company
- Rationale: A brief 5-7 word rationale for why the copmany was classifed as health tech or not health tech based on above criteria
- This solution uses DeepSeek API with confirmed web search capabilities (tested and working)
- The LLM will be prompted to use available resources, including web search to make the determinatinon for each company, and provide the results strucutred as designated above
- The final output will be a new csv file with all of the companies provided in the input file, inclusive of the additonal fields specified above for each company
- Target: Handle 10,000+ companies efficiently at ~$50-150 cost (vs $1000+ for OpenAI)
- Main script: company_categorizer.py
- Input: input/companies_input.csv (and other input files in input/ folder)
- Output: output/companies_classified_YYYYMMDD_HHMMSS.csv (and all generated files in output/ folder)
- Progress files: output/companies_classified_YYYYMMDD_HHMMSS_progress.json
- Backup files: output/companies_classified_YYYYMMDD_HHMMSS_backup_###.csv

## Standard Prompt for LLM Classification
- System: Expert business analyst specializing in healthcare technology
- Always search web for current information before classification
- Format: JSON with company_name, health_tech, company_description, rationale
- Description format: "Provides [services/products] to the [end market]"
- Rationale: Brief 5-7 word explanation for classification based on above criteria


 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ishmcgee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
