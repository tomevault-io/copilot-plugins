## grill-me-skill

> A method for interviewing the user in order to mind meld.


Interview me in detail about all parts of this plan.

You MUST NOT end the interview until you have 100% confidence that you and I have achieved a mind meld,
and that there is absolutely no ambiguity concerning what the plan is or how it should be accomplished.

Some decisions will depend on others;
start with zero-dependency decisions, then continue to build and traverse the decision graph
until it's fully resolved.

## Scope

Part of this interview is determining scope.

You must consider both what I have explicitly said, as well as what a domain expert's spec
would likely include.

E.g. if the plan were about adding a new field to a DB object, you would also interview me to
determine scope: whether the field should be plumbed anywhere else, whether tests should be created
or updated, whether the field should be backported to existing DB objects, etc.

Complete the full and detailed interview for everything in scope.

## Mechanics

Ask questions one at a time.

When it makes sense to, you should suggest discrete answers to your questions.

If you feel strongly that a particular discrete answer is the most suitable, you should note your
recommendation and the reason for it.

When suggesting discrete answers, number them such that I can easily reference them.

Don't ask questions you can answer yourself using available tools.
E.g. don't ask about existing code if that code is available for you to explore.

---
> Source: [pragmaticpandy/grill-me-skill](https://github.com/pragmaticpandy/grill-me-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
