## rule-engine

> When working on the Rule Engine (rules, conditions, actions, triggers).


# Rule Engine

- Prefer delegating to the `rule-engine` subagent for Rule Engine work.
- Full docs: docs/ai/RULE_ENGINE.md.
- Key models: Rule, RuleGroup, ConditionGroup, RuleCondition, RuleAction (app/Models/RuleEngine/).
- Services: RuleEngine, ConditionEvaluator, ActionExecutor (app/Services/RuleEngine/). Events: TransactionCreated, TransactionUpdated; listener ProcessTransactionRules.
- When adding conditions or actions: use existing enums (ConditionField, ConditionOperator, ActionType, Trigger) and service interfaces (ConditionEvaluatorInterface, ActionExecutorInterface).
- Reference: app/Http/Controllers/RuleEngine/RuleController.php (action type config), app/Models/RuleEngine/ActionType.php.

---
> Source: [andrejvysny/spendly](https://github.com/andrejvysny/spendly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
