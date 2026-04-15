## function-call-editor-web-page

> Function-Call api


OpenAI Chat Completion API: Function-Call Overview

High-Level Overview:

Send user/system messages + optional function definitions.

Model may reply with normal content or a function_call.

On function_call: execute locally and return its result.

Model ingests result and replies with final content.

Plan:

Build messages[] and functions[].

Call /v1/chat/completions.

Check response.choices[0].message.function_call.

If present, parse name & args -> call function.

Append function result as message {role:function}.

Re-call API for final reply.

Usage:
Input:
{
model, messages[], functions[]
}
Output:
{
choices: [
{ message: { role: assistant, content | function_call } }
]
}

example nodejs
```

      const response = await axios.post(
        'https://api.openai.com/v1/chat/completions',
        {
          model: "gpt-4o-mini",
          messages: [
            { role: "system", content: "Please strictly follow the function return format to generate a list of questions." },
            { role: "user", content: `${this.input}` }
          ],
          functions: [
            {
              name: "generate_questions",
              description: "Returns a list of strings, each element is a research question",
              parameters: {
                type: "object",
                properties: {
                  questions: {
                    type: "array",
                    items: { type: "string" }
                  }
                },
                required: ["questions"]
              }
            }
          ],
          function_call: { name: "generate_questions" }
        },
        {
          headers: {
            'Authorization': `Bearer ${OPENAI_API_KEY}`,
            'Content-Type': 'application/json'
          }
        }
      );
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wahengchang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
