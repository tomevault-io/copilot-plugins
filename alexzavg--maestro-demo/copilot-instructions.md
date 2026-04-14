## maestro-demo

> MAESTRO DOCUMENTATION & BEST PRACTICES REFERENCE REQUIREMENT


MAESTRO DOCUMENTATION & BEST PRACTICES REFERENCE REQUIREMENT
MANDATORY DOCUMENTATION CHECK BEFORE RESPONDING:

ALWAYS consult Maestro documentation FIRST before providing any answer related to:

Maestro commands and their syntax
Element selection strategies
Flow structure and capabilities
JavaScript integration in runScript blocks
Maestro CLI usage and options
Maestro Cloud features
Platform-specific behaviors (iOS vs. Android)
Configuration options
Best practices and recommendations


Primary documentation sources to reference:

Official Maestro documentation at https://maestro.mobile.dev
Maestro GitHub repository and examples
Maestro command reference and API docs
Maestro best practices guides
Community-recommended patterns


When to check documentation:

Before suggesting any Maestro command or syntax
When asked "how do I..." questions
When proposing solutions to problems
When explaining Maestro features or capabilities
When debugging flow issues
When optimizing existing flows
When uncertain about command behavior or parameters



DOCUMENTATION-FIRST WORKFLOW:

For every Maestro-related query:

Step 1: Search/reference Maestro docs for the relevant command, feature, or pattern
Step 2: Verify the syntax, parameters, and recommended usage
Step 3: Check for best practices or common pitfalls mentioned in docs
Step 4: Formulate answer based on official documentation
Step 5: Apply cached codebase patterns that align with documented best practices


Explicitly mention when referencing docs:

"According to Maestro documentation..."
"Maestro best practices recommend..."
"The official docs show that..."
"Maestro's documentation suggests..."



WHAT TO VERIFY IN DOCUMENTATION:

Command syntax and parameters:

Exact YAML structure for commands
Required vs. optional parameters
Parameter types and valid values
Command availability and deprecations


Best practices:

Recommended element selection strategies
Proper wait and retry patterns
Flow organization recommendations
Performance optimization techniques
Stability and reliability patterns


Platform differences:

iOS-specific behaviors documented
Android-specific behaviors documented
Platform-specific commands or parameters
Cross-platform compatibility notes


Advanced features:

JavaScript capabilities in runScript
Environment variable usage
Maestro Cloud features
CI/CD integration patterns
Custom conditions and assertions



WHEN DOCUMENTATION IS UNCLEAR OR MISSING:

If documentation doesn't cover the specific scenario:

Clearly state: "This specific case isn't explicitly documented in Maestro docs"
Suggest the closest documented pattern
Base suggestion on general Maestro principles from docs
Recommend testing the approach in a safe environment
Suggest checking Maestro community discussions or GitHub issues


If documentation conflicts with cached codebase patterns:

Prioritize documentation best practices
Explain the conflict: "The docs recommend X, but I see your codebase uses Y"
Suggest: "Would you like to align with the documented best practice, or maintain consistency with existing patterns?"



STAY UPDATED:

Be aware that Maestro is actively developed:

Note when suggesting potentially version-specific features
Recommend checking the latest docs if behavior seems unexpected
Acknowledge when features might be newer or experimental
Suggest verifying Maestro version compatibility



DOCUMENTATION INTEGRATION WITH OTHER INSTRUCTIONS:

Combine documentation guidance with:

Cached codebase patterns (prioritize doc best practices)
Simplicity principles (use documented simple approaches)
Context memory (apply documented patterns consistently)
File modification permissions (suggest doc-aligned changes only when approved)


Quality hierarchy for answers:

Tier 1: Official Maestro documentation + your cached codebase patterns
Tier 2: Official docs + general best practices (if no codebase pattern exists)
Tier 3: Documented principles + logical inference (if exact scenario not documented)
Never: Pure speculation without documentation basis



PROACTIVE DOCUMENTATION SHARING:

When helpful, include:

Relevant documentation links
"For more details, see: [Maestro docs URL]"
References to specific documentation sections
Suggestions to bookmark frequently used doc pages




CRITICAL RULE: Never provide Maestro-specific advice, commands, or patterns without first consulting or referencing the official Maestro documentation. Documentation-based answers ensure accuracy, promote best practices, and prevent suggesting unsupported or deprecated features.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexzavg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
