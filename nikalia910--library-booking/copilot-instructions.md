## library-booking

> Feature: [Title of the feature]

Feature: [Title of the feature]
  As a [type of user],
  I want to [achieve a business goal],
  So that [I gain a benefit or solve a problem].

  Background:
    Given [the system is in a valid initial state]
    And [the user is authenticated or relevant context is set]

  Rule: [Business rule 1 - describe the rule this set of scenarios supports]
    Scenario: [Describe a business outcome or situation]
      Given [a specific business condition]
      When [the user performs a meaningful action]
      Then [the system responds with a result that satisfies the business rule]
      And [any additional expected result]

    Scenario: [Another case under the same rule]
      Given [a different condition or edge case]
      When [another user action]
      Then [corresponding expected outcome]

  Rule: [Business rule 2 - another domain constraint or user need]
    Scenario: [Normal case demonstrating the rule]
      Given [a relevant precondition]
      When [the user takes an action]
      Then [the system produces the correct result]

  Scenario Outline: [Generic scenario covering multiple input sets]
    Given [a user with <user role>]
    When [they perform <action> on <item>]
    Then [the system should <expected outcome>]

    Examples:
      | user role  | action     | item     | expected outcome      |
      | admin      | approve    | request  | see approval success  |
      | guest      | submit     | form     | see access denied     |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NikAlia910) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
