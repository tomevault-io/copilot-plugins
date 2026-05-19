## writing-style

> 1. **Clarity Over Cleverness**

## ✍️ Unified Documentation Style Guide (for LLMs & Authors)

### 🎯 Core Style Principles (Applies to All Documentation)

1. **Clarity Over Cleverness**  
   Use simple, direct language. Favor short words and short sentences. Avoid jargon unless it’s essential—and if it is, define it once.

2. **Speak Developer**  
   Write the way you’d speak to a smart, curious engineer sitting next to you. Friendly, confident, and precise.

3. **Show, Don’t Just Tell**  
   Use examples liberally. Explain ideas with real code, not abstract theory. Every key concept should be backed by a working snippet.

4. **Write in Active Voice**  
   ✅ “Click the button to save your changes.”  
   ❌ “The button should be clicked in order for changes to be saved.”

5. **Present First, Explain Later**  
   Get the reader to success as quickly as possible. After they’ve seen something work, then explain how/why it works.

6. **Be Honest and Human**  
   If something is tricky, say so. If you’re recommending a workaround, explain why. Transparency builds trust.

7. **Use consistent terminology**  
   Always refer to concepts, components, and APIs by the same name. Don’t mix “endpoint” and “route” if they mean the same thing.

---

### 📘 Tutorials: Writing Style

**Tone:** Supportive, encouraging, and confidence-building  
**Voice:** A helpful mentor walking them through their first success

#### ✅ Style Rules:
- Use **first or second person**: “Let’s build…”, “You’ll start by…”
- Keep steps **concrete, sequential, and minimal**.
- Don’t introduce **abstractions or options** early on.
- Use inline **code examples and copy-pasteable blocks**.
- Confirm user actions with language like:  
  ✅ “You should now see…”  
  ✅ “If that worked, your app will…”  
- Avoid errors at all costs — assume **zero debugging ability**.

---

### 📗 How-To Guides: Writing Style

**Tone:** Efficient, direct, and solution-focused  
**Voice:** A senior engineer giving clear, concise steps to unblock you

#### ✅ Style Rules:
- Title format: **“How to [Do a Thing]”** — no vague names.
- Steps should be **numbered**, each with a clear result.
- Assume familiarity with the basics — don't over-explain.
- Use **"you"** to keep it actionable:  
  ✅ “You’ll use the `setState` hook to store the value.”
- Link to reference or explanation docs when needed, instead of embedding long descriptions.

---

### 📙 Reference: Writing Style

**Tone:** Neutral, factual, dry (in a good way)  
**Voice:** An API robot that also wants to help you succeed

#### ✅ Style Rules:
- Be **exhaustive, not narrative**.
- Use consistent formatting:  
  ```
  Function: fetchData(url: string): Promise<Data>
  Description: Fetches data from the given URL.
  Parameters:
    - url: The endpoint to call.
  Returns:
    - A promise that resolves to the data object.
  ```
- Prefer **technical precision over friendliness**.
- Include edge cases, limitations, and defaults.
- Reference docs should be **scannable** — headings, tables, and bullet points are your friends.

---

### 📕 Explanation: Writing Style

**Tone:** Insightful, reflective, and exploratory  
**Voice:** An expert developer sharing wisdom at a meetup talk

#### ✅ Style Rules:
- Use **first-person plural (“we”)** or third-person neutral.
- Break up text with headings, quotes, comparisons, and lists.
- Be **opinionated, but humble** — “We recommend X because…” not “X is the best.”
- Draw on **real-world scenarios**: “In high-latency environments, this matters…”
- Link to related tutorials or reference sections where relevant.
- Don’t be afraid to dig into **history, philosophy, or trade-offs**.

---

### 🧠 Additional Style Conventions

| Rule | Example |
|------|---------|
| ✅ Use inline code for single functions, props, or values | `fetchData()` |
| ✅ Use fenced code blocks for multi-line examples | \`\`\`js ... \`\`\` |
| ✅ Use bold for file paths or commands | **`npm install`** |
| ✅ Use emojis sparingly (but okay in tutorials or intros) | ✅, ⚠️, 💡 |
| ✅ Avoid ellipses unless truncating output | ❌ “Click Save…” |
| ✅ Always spell out acronyms on first use | “Content Delivery Network (CDN)” |

---

# 📄 MDX-First Documentation Rules

When documenting projects that utilize Fumadocs, it's essential to adhere to specific conventions to maintain consistency and clarity. Below are key guidelines to follow:

**1. Use MDX for Documentation**

Always author documentation using MDX (Markdown for JSX). MDX combines the simplicity of Markdown with the power of JSX, allowing for the inclusion of interactive components within your documentation. This approach enhances the readability and functionality of the documentation. Fumadocs provides extensive support for MDX, making it the preferred choice for creating comprehensive and interactive documentation. citeturn0search5

**2. Frontmatter Configuration**

At the beginning of each MDX document, include a frontmatter section to define metadata such as the title and description. This metadata is crucial for organizing and presenting the documentation effectively. An example of a frontmatter section is:


```mdx
---
title: MySQL Adapter
description: The MySQL adapter provides integration with MySQL and MariaDB, widely-used relational database systems known for reliability, performance, and broad compatibility.
---
```


The `description` field serves as an introductory paragraph and should be placed immediately after the frontmatter. This ensures that readers receive a concise overview of the document's content right from the start. citeturn0search5

**3. Heading Structure**

Do not use an H1 (`#`) heading at the beginning of the document. The `title` defined in the frontmatter is automatically rendered as the main heading of the page. Starting with an H1 heading would duplicate the title and disrupt the document's structure. Instead, begin with an H2 (`##`) or appropriate subheading to introduce sections within the document. citeturn0search5

**4. Consistent Writing Style**

- **Clarity and Conciseness**: Use clear and concise language to convey information effectively.

- **Active Voice**: Prefer active voice over passive voice to make sentences more direct and vigorous.

- **Consistent Terminology**: Use consistent terminology throughout the documentation to avoid confusion.

- **Code Blocks**: For code examples, use fenced code blocks with appropriate language identifiers for syntax highlighting. For example:

  
```js
  console.log('Hello, World!');
  ```


### 🧩 Example Template for Tutorial

```md
# Getting Started with [Product]

In this tutorial, you'll build a simple [thing] using [product/tool]. By the end, you’ll have a working [result] and understand the basics of how it works.

## Prerequisites
- Node.js v18+
- A basic understanding of JavaScript

## Step 1: Install the CLI

```package-install
our-cli
```

You should now be able to run:

```bash
our-cli --help
```

If that works, let’s move on.

...

## Conclusion

You’ve just built a working [thing]! Next, try our [how-to guide] to add [extra feature].
```

---
> Source: [c15t/c15t](https://github.com/c15t/c15t) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
