## conversation-style

> Enforces consistent conversation patterns and style

 # Conversation Style
Guidelines for maintaining a consistent and engaging conversation style.

<rule>
name: conversation_style
description: Enforces consistent conversation patterns and style
filters:
  - type: event
    pattern: "conversation_start|message_response"

actions:
  - type: suggest
    message: |
      When engaging in conversations, follow these rules:

      1. Unicorn Greeting:
         - Start every conversation and response with "🦄"
         - Place the emoji at the very beginning of the message
         - Add a space after emoji if followed by a sentence
         Example:
         ```
         // Bad: Missing unicorn or wrong placement
         Hello! How can I help you today?
         Hi 🦄 there!
         
         // Good: Proper unicorn placement
         🦄 Hello! How can I help you today?
         🦄 Hi there!
         🦄 I'll help you with that request.
         ```

      2. Response Structure:
         - Emoji followed by greeting (if appropriate)
         - Then context or acknowledgment
         - Finally, the main response
         Example:
         ```
         // Bad: Incorrect structure
         Let me help you with that.
         🦄 [No context or acknowledgment]

         // Good: Proper structure
         🦄 I understand you want to modify the configuration.
         I'll help you make those changes safely.
         ```

      3. Consistency:
         - Use the unicorn emoji consistently
         - Don't substitute with other emojis
         - Don't skip the emoji even in short responses
         Example:
         ```
         // Bad: Inconsistent usage
         🌟 Hello there!
         Sure, I can help.

         // Good: Consistent usage
         🦄 Hello there!
         🦄 Sure, I can help.
         ```

examples:
  - input: |
      User: Can you help me with this code?
      Assistant: Let me take a look at that for you.
      
      User: Thanks!
      Assistant: You're welcome!
    output: |
      User: Can you help me with this code?
      Assistant: 🦄 Let me take a look at that for you.
      
      User: Thanks!
      Assistant: 🦄 You're welcome!

metadata:
  priority: medium
  version: 1.0
  tags:
    - conversation
    - style
    - user-interaction
</rule>

---
> Source: [squirrelogic/cursor-rules](https://github.com/squirrelogic/cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
