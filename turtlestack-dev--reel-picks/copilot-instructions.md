## reel-picks

> You are an expert in Ruby, Ruby on Rails, and Hotwire (Turbo and Stimulus).

# Ruby on Rails

You are an expert in Ruby, Ruby on Rails, and Hotwire (Turbo and Stimulus).

## File Generation
- Use the `rails generate` command as much as possible when creating new files—_especially_ when creating database migrations.
- If you don't know what generators are available, you can use the `-h` flag to get a list.

## Code Style and Structure
- Write concise, idiomatic Ruby code with accurate examples.
- Follow Rails conventions and best practices.
- Use object-oriented and functional programming patterns as appropriate.
- Prefer iteration and modularization over code duplication.
- Prefer composition over inheritance.
- Structure files according to Rails conventions (MVC, concerns, helpers, etc.).

## Naming Conventions
- Use snake_case for file names, method names, and variables.
- Use CamelCase for class and module names.
- Follow Rails naming conventions for models, controllers, and views.
- Use descriptive variable and method names (e.g., user_signed_in?, calculate_total).
  - Class names should be nouns.
  - Method names should be verbs or verb phrases. Exceptions include predicate methods expected to return true or false. In that case, method names should be in the form of a yes or no question beginning with "is," "can," "has," etc., but instead of including those words, add a question mark to the end of the method name. For example, `is_user_signed_in` becomes `user_signed_in?`.
  - Variable names should describe the type of content they hold. If a variable holds a boolean value, it should follow the convention `is_user_signed_in` instead of ending in a question mark (`user_signed_in?`, which is reserved for method names).

## Ruby and Rails Usage
- Use Ruby 3.x features when appropriate (e.g., pattern matching, endless methods).
- Leverage Rails' built-in helpers and methods.
- Use ActiveRecord effectively for database operations.

## Syntax and Formatting
- Follow the Ruby Style Guide (https://rubystyle.guide/)
- Use Ruby's expressive syntax (e.g., unless, ||=, &.)
- Prefer double quotes for strings to maintain consistency.

## Error Handling and Validation
- Use exceptions for exceptional cases, not for control flow.
- Implement proper error logging and user-friendly messages.
- Use ActiveModel validations in models.
- Handle errors gracefully in controllers and display appropriate flash messages.

## UI and Styling
- Use Hotwire (Turbo and Stimulus) for dynamic, SPA-like interactions.
- Use Rails view helpers and partials to keep views DRY.

## Performance Optimization
- Use database indexing effectively.
- Implement caching strategies (fragment caching, Russian Doll caching).
- Use eager loading to avoid N+1 queries.
- Optimize database queries using includes, joins, or select.

## Key Conventions
- Follow RESTful routing conventions.
- Use concerns for shared behavior across models or controllers.
- Implement service objects for complex business logic.
- Use background jobs (e.g., ActiveJob) for time-consuming tasks.

## Testing
- Write comprehensive tests using RSpec or Minitest.
- Follow TDD/BDD practices.
- Use fixtures for test data generation.
- When writing unit tests, use these basic guidelines:
  - Writing unit tests is all about the object under test and messages.
  - Treat the object under test like a "black box."
  - Messages can also be referred to as method calls.
  - There are two basic types of messages: queries and commands.
    - Query messages perform a calculation and return a result. Or in other words, query messages return something, but change nothing.
    - Command messages change something the rest of the app can "see" (public), also known as a side effect. Or in other words, command messages change something, but don't return anything you depend on.
    - Sometimes those two messages types are conflated, but it's best to avoid that unless necessary.
  - From the point of view of the object under test, there are three origins for messages: incoming, internal, and outgoing.
    - Incoming messages are sent to the object under test.
    - Internal messages are messages the object under test sends to itself.
    - Outgoing messages are sent out from the object under test.
  - Given these message types and message origins, you can create a matrix:
    - Tests for incoming query messages should assert a result.
    - Tests for incoming command messages should test for a direct, public side effect.
    - As a rule, ignore testing internal messages.
    - For outgoing query messages, don't test them.
      - Don't make assertions about their result.
      - Do not expect to send them.
    - For outgoing command messages, test only that a messages was sent to the target object—do not assert the result of the sent message.
      - This may involve creating a mock for the target object in the test.
- Controller unit tests should not assert view elements (i.e., that there is an `h2` tag with specific content).

## Security
- Implement proper authentication and authorization (e.g., Devise, Pundit).
- Use strong parameters in controllers.
  - Use the new `params.expect()` over the old `params.require().permit()` method. For example:

    ```
    private

    def user_params
      params.expect(user: [:email_address, :password])
    end
    ```

    Instead of:

    ```
    private

    def user_params
      parms.require(:user).permit(:email_address, :password)
    end
    ```
- Protect against common web vulnerabilities (XSS, CSRF, SQL injection).

Follow the official Ruby on Rails guides for best practices in routing, controllers, models, views, and other Rails components.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/turtlestack-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
