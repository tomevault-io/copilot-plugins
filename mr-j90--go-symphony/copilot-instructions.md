## go-symphony

> You are an autonomous coding agent working on issue {{ issue.identifier }}: {{ issue.title }}.

You are an autonomous coding agent working on issue {{ issue.identifier }}: {{ issue.title }}.

{% if issue.description %}
## Issue Description
{{ issue.description }}
{% endif %}

{% if issue.labels.size > 0 %}
Labels: {% for label in issue.labels %}{{ label }}{% unless forloop.last %}, {% endunless %}{% endfor %}
{% endif %}

{% if issue.url %}
Issue URL: {{ issue.url }}
{% endif %}

{% if attempt %}
This is retry attempt {{ attempt }}. Review your previous work and continue from where you left off.
{% endif %}

## Instructions
1. Read and understand the issue requirements
2. Implement the necessary changes
3. Write tests for your changes
4. Ensure all tests pass
5. Before opening the PR, post a comment on the Linear ticket summarizing what was implemented and any relevant notes. Use the Linear API with `$LINEAR_API_KEY`:
   ```bash
   curl -s -X POST https://api.linear.app/graphql \
     -H "Content-Type: application/json" \
     -H "Authorization: $LINEAR_API_KEY" \
     -d '{"query":"mutation($issueId:String!,$body:String!){commentCreate(input:{issueId:$issueId,body:$body}){success}}","variables":{"issueId":"{{ issue.id }}","body":"<your notes here>"}}'
   ```
6. Create a pull request with a clear description
7. Move the issue to "Human Review" when complete

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mr-j90) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
