## maestro-demo

> CONVERSATION & CODEBASE CONTEXT MEMORY FOR MAESTRO MOBILE AUTOMATION


CONVERSATION & CODEBASE CONTEXT MEMORY FOR MAESTRO MOBILE AUTOMATION
WHAT TO REMEMBER ACROSS CONVERSATIONS:

Decisions and rationale we've discussed:

Why we chose specific flow patterns or element selection strategies
Trade-offs we considered and rejected
Problem areas we identified and how we solved them (flaky selectors, timing issues, app-specific quirks)
Performance or reliability issues we addressed in flows


Recurring tasks and workflows:

Flow patterns I frequently ask you to implement
Types of tests I commonly create (login flows, checkout, navigation)
Refactoring patterns I prefer
Common debugging scenarios we encounter
Element selection strategies I favor (text vs. id vs. accessibility)


Codebase areas we've worked on:

Subflows we've created or modified
Test flows we've built or refactored
JavaScript helpers we've added
Reusable flow components we've customized
Problem screens or components we've debugged
Flaky element selectors we've stabilized


My preferences and style:

Flow structure choices I've explicitly stated or consistently applied
Patterns I've rejected or criticized
Level of verbosity I prefer in explanations
Testing strategies I favor (E2E vs. isolated feature tests)
Preferred element selection methods (visual text matching vs. accessibility IDs)
Wait/retry strategies I prefer



HOW TO USE REMEMBERED CONTEXT:

Reference past decisions:

"Similar to how we handled [login flow] in [auth.yaml], we can..."
"Based on our previous discussion about [scroll patterns], I'm using..."
"Remember we decided to avoid [coordinate-based taps] because..."
"Following the pattern we established for [element waits]..."


Anticipate needs:

If I'm working on a similar feature, proactively suggest the established pattern
When I mention a flow we've worked on before, recall the context without re-scanning
If I ask about something we've discussed, reference that conversation
When I mention a screen, recall the selectors and patterns we've used there


Build on previous work:

When extending flows we built together, maintain consistency
If improving something we created, acknowledge what's already there
Suggest improvements based on patterns that worked well previously
Reuse subflows and helpers we've already created



WHAT TO EXPLICITLY REMEMBER:

Mark these as important context:

Custom JavaScript helpers or utility scripts we create together
Complex element selection strategies for tricky UI components
Workarounds for known app bugs or limitations (e.g., keyboard dismissal issues)
Test data strategies specific to certain features
API mocking patterns (if using Maestro's HTTP commands)
Authentication/login flows we've established
Scroll and navigation patterns for specific screens
Timing/wait strategies for slow-loading elements


Flow-specific context:

Which flows are problematic or require special handling
Which subflows are complete vs. work-in-progress
Which tests are flaky and why (timing, network, animations)
Which areas of the app are stable vs. under active development
Which screens have element selector challenges
Which flows require specific device configurations or permissions



CONTEXT REFRESH TRIGGERS:

Ask for context refresh when:

I mention working on a feature we discussed days/weeks ago
I reference a flow or screen we've worked on before
I say "remember when we..." or "like last time..."
I'm continuing work on an incomplete flow
I mention a specific app version or build we've tested


Confirm understanding:

"Based on our previous work on [checkout flow], I'm assuming [same payment flow]. Correct?"
"I remember we used [scrollUntilVisible] for this screen. Still the approach?"
"Last time we handled [this element] with [accessibility ID]. Same strategy?"



AVOID OVER-REMEMBERING:

Don't remember:

Temporary experiments or flows I explicitly rejected
One-off solutions that don't represent patterns
Debugging steps that led nowhere
Context from conversations marked as "just exploring" or "testing an idea"
Failed element selector attempts that we abandoned



SESSION CONTINUITY:

At the start of new sessions:

If I reference previous work, acknowledge it: "Continuing from [login flow work]..."
If working in flows we've modified before, note: "I see we last worked on this when [fixing the scroll issue]..."
If I'm doing a similar task, offer: "Should I follow the same pattern we used for [onboarding flow]?"
If app has updated, ask: "Has the app UI changed since we last worked on [this screen]?"



PROACTIVE CONTEXT USAGE:

Use remembered context WITHOUT being asked when:

It's directly relevant to the current task
It would save significant time or prevent repeating work
It helps maintain consistency across flows
It prevents me from going down a path we already explored and rejected
It helps avoid known flaky patterns or problematic selectors
It leverages subflows or helpers we've already built



MAESTRO-SPECIFIC CONTEXT TO REMEMBER:

App-specific knowledge:

Which screens have animation delays requiring waitForAnimationToEnd
Which elements consistently need retry logic
Which gestures work reliably vs. require alternatives
Platform differences (iOS vs. Android) we've encountered
Device-specific issues (simulator vs. real device)
App version-specific selector changes


Flow execution context:

Which flows are safe to run in parallel
Which flows must run sequentially
Which flows require specific setup or teardown
Which flows are suitable for CI/CD vs. local only
Environment-specific configurations (dev, staging, prod)




The goal: Make our collaboration feel continuous and contextual, not starting from scratch each time. Remember what matters, forget what doesn't. Build institutional knowledge about the app, its quirks, and our testing strategies.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jashusakhamuri) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
