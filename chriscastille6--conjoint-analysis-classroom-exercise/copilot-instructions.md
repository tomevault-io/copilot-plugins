## conjoint-analysis-classroom-exercise

> This is a conjoint analysis classroom exercise bundle containing R/Shiny applications, survey instruments, instructor dashboards, and research materials for educational use.

# Cursor Rules for Conjoint Analysis Classroom Exercise Project

## Project Context
This is a conjoint analysis classroom exercise bundle containing R/Shiny applications, survey instruments, instructor dashboards, and research materials for educational use.

## Code Style and Conventions

### R/Shiny Development
- Follow tidyverse conventions for R code
- Use meaningful variable names that reflect conjoint analysis terminology
- Prefer dplyr syntax over base R when appropriate
- Use consistent indentation (2 spaces)
- Comment complex statistical operations and conjoint-specific logic
- Use snake_case for variables and functions
- Use PascalCase for Shiny module names

### File Organization
- Keep analysis scripts in `/analysis/` directory
- Store data in `/data/` directory
- Place outputs in `/outputs/` directory
- Use descriptive filenames that indicate purpose and version
- Maintain consistent naming patterns across similar files

### Conjoint Analysis Specifics
- Use proper conjoint terminology: attributes, levels, choice tasks, utilities, part-worths
- Implement proper randomization for choice tasks
- Ensure balanced and orthogonal experimental designs
- Validate choice data before analysis
- Use appropriate statistical methods (logit models, hierarchical Bayes)

### Documentation Standards
- Include clear headers in R scripts describing purpose and usage
- Document data structures and expected formats
- Provide setup instructions for classroom deployment
- Maintain version control for iterative improvements
- Include references to conjoint analysis methodology

### Error Handling
- Implement graceful error handling for user inputs
- Provide clear error messages for invalid survey responses
- Validate data integrity before analysis
- Handle missing data appropriately in statistical models

### Performance Considerations
- Optimize Shiny apps for classroom use (multiple concurrent users)
- Use efficient data structures for large choice datasets
- Implement proper session management
- Consider memory usage for educational environments

## Communication Style
- Acknowledge corrections neutrally without patronizing phrases
- Focus on implementing fixes directly rather than validating user feedback
- Provide factual, concise responses
- Avoid unnecessary praise or validation language

## Research Ethics
- Ensure IRB compliance for all research components
- Maintain participant confidentiality
- Follow ethical guidelines for educational research
- Document consent procedures clearly

## Deployment Considerations
- Ensure cross-platform compatibility
- Test in classroom environments
- Provide clear instructor setup instructions
- Consider network limitations in educational settings

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chriscastille6) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
