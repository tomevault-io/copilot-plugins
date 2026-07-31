## wiki

> > **Important**: The markdown extension is a early release and can be subject to change or may have edge cases that may not be supported yet. If you are encountering a bug or have a feature request, please open an issue on GitHub.





# Introduction into Markdown with Tiptap

> **Important**: The markdown extension is a early release and can be subject to change or may have edge cases that may not be supported yet. If you are encountering a bug or have a feature request, please open an issue on GitHub.

The Markdown extension provides bidirectional Markdown support for your Tiptap editor—parse Markdown strings into Tiptap's JSON format and serialize editor content back to Markdown.

## [](#core-capabilities)Core Capabilities

-   **Markdown Parsing**: Convert Markdown strings to Tiptap JSON
-   **Markdown Serialization**: Export editor content as Markdown
-   **Custom Tokenizers**: Add support for custom Markdown syntax
-   **Extensible Architecture**: Each extension can define its own parsing and rendering logic
-   **Utilities to Simplify Custom Syntax Creation**: `createBlockMarkdownSpec`, `createInlineMarkdownSpec` and more
-   **HTML Support**: Parse HTML embedded in Markdown using Tiptap's existing HTML parsing

## [](#how-it-works)How It Works

The Markdown extension acts as a bridge between Markdown text and Tiptap's JSON document structure.

It extends the base editor functionality by overwriting existing methods & properties with markdown-ready implementations, allowing for seamless integration between Markdown and Tiptap's rich text editor.

```
// Set initial content
const editor = new Editor({
  extensions: [StarterKit, Markdown],
  content: '# Hello World\n\nThis is **Markdown**!',
  contentType: 'markdown',
})

// Insert content
editor.commands.insertContent('# Hello World\n\nThis is **Markdown**!')
```

### [](#architecture)Architecture

```
Markdown String
      ↓
   MarkedJS Lexer (Tokenization)
      ↓
   Markdown Tokens
      ↓
   Extension Parse Handlers
      ↓
   Tiptap JSON
```

And in reverse:

```
Tiptap JSON
      ↓
   Extension Render Handlers
      ↓
   Markdown String
```

## [](#limitations)Limitations

The current implementation of the Markdown extension has some limitations:

-   **Comments are not supported yet**: Some advanced features like comments are not supported in Markdown. Be **cautious** when parsing Markdown content into a document that contains comments as they may be lost if replaced by Markdown content.
-   **Multiple child nodes in Tables**: Markdown tables are supported, but only one child node per cell is allowed as the Markdown syntax can't represent multiple child nodes.

## [](#why-markedjs)Why MarkedJS?

This extension integrates [MarkedJS](https://marked.js.org) as its parser:

-   **Fast and Lightweight**: One of the fastest Markdown parsers available
-   **Extensible**: Custom tokenizers enable non-standard Markdown syntax
-   **CommonMark Compliant**: Follows the CommonMark specification
-   **Battle-tested**: Widely used in production with active development

The Lexer API breaks Markdown into tokens that map naturally to Tiptap's node structure, making the integration clean and maintainable. The extension works identically in browser and server environments.



# Install and Setup the Markdown Package

This guide will walk you through installing and setting up the Markdown extension in your Tiptap editor.

## [](#installation)Installation

Install the Markdown extension using your preferred package manager:

```
npm install @tiptap/markdown
```

## [](#basic-setup)Basic Setup

Add the Markdown extension to your editor:

```
import { Editor } from '@tiptap/core'
import StarterKit from '@tiptap/starter-kit'
import { Markdown } from '@tiptap/markdown'

const editor = new Editor({
  element: document.querySelector('#editor'),
  extensions: [StarterKit, Markdown],
  content: '<p>Hello World!</p>',
})
```

That's it! Your editor now supports Markdown parsing and serialization.

### [](#initial-content-as-markdown)Initial Content as Markdown

To load Markdown content when creating the editor:

```
const editor = new Editor({
  extensions: [StarterKit, Markdown],
  content: '# Hello World\n\nThis is **Markdown**!',
  contentType: 'markdown',
})
```

## [](#configuration-options)Configuration Options

The Markdown extension accepts several configuration options:

### [](#indentation-style)Indentation Style

Configure how nested structures (lists, code blocks) are indented in the serialized Markdown:

```
Markdown.configure({
  indentation: {
    style: 'space', // 'space' or 'tab'
    size: 2, // Number of spaces or tabs
  },
})
```

**Examples:**

```
// Use 4 spaces for indentation (default: 2 spaces)
Markdown.configure({
  indentation: { style: 'space', size: 4 },
})

// Use tabs for indentation
Markdown.configure({
  indentation: { style: 'tab', size: 1 },
})
```

### [](#custom-marked-instance)Custom Marked Instance

If you need to use a custom version of marked or pre-configure it:

```
import { marked } from 'marked'

// Configure marked
marked.setOptions({
  gfm: true,
  breaks: true,
})

// Use custom marked instance
Markdown.configure({
  marked: marked,
})
```

### [](#marked-options)Marked Options

You can also pass marked options directly to the extension:

```
Markdown.configure({
  markedOptions: {
    gfm: true, // GitHub Flavored Markdown
    breaks: false, // Convert \n to <br>
    pedantic: false, // Strict Markdown mode
    smartypants: false, // Smart quotes and dashes
  },
})
```

See the [marked documentation](https://marked.js.org/using_advanced#options) for all available options.

## [](#verifying-installation)Verifying Installation

To verify the extension is installed correctly:

```
// Check if Markdown manager is available
console.log(editor.markdown) // Should log the MarkdownManager instance

// Try parsing
const json = editor.markdown.parse('# Hello')
console.log(json)
// { type: 'doc', content: [...] }

// Try serializing
const markdown = editor.markdown.serialize(json)
console.log(markdown)
// # Hello
```

## [](#common-issues)Common Issues

### [](#extension-not-found)Extension Not Found

If you get an error that `@tiptap/markdown` cannot be found:

1.  Make sure it's installed: `npm list @tiptap/markdown`
2.  Clear your build cache and node\_modules
3.  Reinstall dependencies

### [](#markdown-not-parsing)Markdown Not Parsing

If Markdown isn't being parsed:

1.  Make sure you're using `contentType: 'markdown'` when setting initial content
2.  Or use the `contentType: 'markdown'` option when calling `setContent()`

### [](#typescript-errors)TypeScript Errors

If you get TypeScript errors:

1.  Make sure `@tiptap/core` is installed (it includes type definitions)
2.  Update to the latest version of both packages
3.  Check your `tsconfig.json` includes the correct module resolution

```
{
  "compilerOptions": {
    "moduleResolution": "node",
    "esModuleInterop": true
  }
}
```


# Basic Usage

This guide covers the core operations for working with Markdown: parsing Markdown into your editor and serializing editor content back to Markdown.

## [](#getting-markdown-from-the-editor)Getting Markdown from the Editor

Use `getMarkdown()` to serialize your editor content to Markdown:

```
const markdown = editor.getMarkdown()
console.log(markdown)
// # Hello
//
// This is a **test**.
```

## [](#setting-content-from-markdown)Setting Content from Markdown

All content commands support the `contentType` option:

```
// 1. Initial content
const editor = new Editor({
  extensions: [StarterKit, Markdown],
  content: '# Hello World\n\nThis is **markdown**!',
  contentType: 'markdown',
})

// 2. Replace all content
editor.commands.setContent('# New Content', { contentType: 'markdown' })

// 3. Insert at cursor
editor.commands.insertContent('**Bold** text', { contentType: 'markdown' })

// 4. Insert at specific position
editor.commands.insertContentAt(10, '## Heading', { contentType: 'markdown' })

// 5. Replace a range
editor.commands.insertContentAt({ from: 10, to: 20 }, '**Replace**', { contentType: 'markdown' })
```

## [](#using-the-markdownmanager-directly)Using the MarkdownManager Directly

For more control, access the `MarkdownManager` via `editor.markdown`:

```
// Parse Markdown to JSON
const json = editor.markdown.parse('# Hello World')
console.log(json)
// { type: 'doc', content: [...] }

// Serialize JSON to Markdown
const markdown = editor.markdown.serialize(json)
console.log(markdown)
// # Hello World
```

This is useful when working with JSON content outside the editor context.

## [](#github-flavored-markdown-gfm)GitHub Flavored Markdown (GFM)

Enable GFM for features like tables and task lists:

```
import { Markdown } from '@tiptap/markdown'
import StarterKit from '@tiptap/starter-kit'
import Table from '@tiptap/extension-table'
import TableRow from '@tiptap/extension-table-row'
import TableCell from '@tiptap/extension-table-cell'
import TableHeader from '@tiptap/extension-table-header'
import TaskList from '@tiptap/extension-task-list'
import TaskItem from '@tiptap/extension-task-item'

const editor = new Editor({
  extensions: [
    StarterKit,
    Table,
    TableRow,
    TableCell,
    TableHeader,
    TaskList,
    TaskItem,
    Markdown.configure({
      markedOptions: { gfm: true },
    }),
  ],
})
```

## [](#inline-formatting)Inline Formatting

Standard Markdown formatting works automatically:

```
const markdown = `
**bold text** or __bold text__
*italic text* or _italic text_
***bold and italic***
[Link Text](https://example.com)
\`inline code\`
`

editor.commands.setContent(markdown, { contentType: 'markdown' })
const result = editor.getMarkdown() // Formatting preserved
```

## [](#working-with-block-elements)Working with Block Elements

Block elements like headings, lists, and code blocks work as expected:

### [](#headings)Headings

```
const markdown = `
# Heading 1
## Heading 2
### Heading 3
#### Heading 4
##### Heading 5
###### Heading 6
`
```

### [](#lists)Lists

```
// Unordered lists
const markdown = `
- Item 1
- Item 2
  - Nested item 2.1
  - Nested item 2.2
- Item 3
`

// Ordered lists
const markdown = `
1. First item
2. Second item
   1. Nested item 2.1
   2. Nested item 2.2
3. Third item
`
```

### [](#code-blocks)Code Blocks

```
const markdown = `
\`\`\`javascript
function hello() {
  console.log('Hello World')
}
\`\`\`
`
```

### [](#blockquotes)Blockquotes

```
const markdown = `
> This is a blockquote.
> It can span multiple lines.
>
> > And can be nested.
`
```

## [](#handling-html-in-markdown)Handling HTML in Markdown

The Markdown extension can parse HTML embedded in Markdown using Tiptap's existing `parseHTML` methods:

```
const markdown = `
# Heading

<div class="custom">
  <p>This HTML will be parsed</p>
</div>

Regular **Markdown** continues here.
`

editor.commands.setContent(markdown, { contentType: 'markdown' })
```

The HTML is parsed according to your extensions' `parseHTML` rules, allowing you to support custom HTML nodes.

## [](#best-practices)Best Practices

**Always use `contentType`** and set it to `markdown` when setting Markdown content (otherwise it's treated as HTML):

```
editor.commands.setContent(markdown, { contentType: 'markdown' })
```

**Include all needed extensions** or content may be lost:

```
const editor = new Editor({
  extensions: [StarterKit, Markdown], // StarterKit covers most common nodes
})
```

**Test round-trip conversion** to ensure your custom Markdown content survives parse → serialize:

```
editor.commands.setContent('# Hello **World**', { contentType: 'markdown' })
const result = editor.getMarkdown() // Should match original
```

## [](#key-components)Key Components

### [](#markdownmanager)`MarkdownManager`

The `MarkdownManager` class is the core engine that handles parsing and serialization. It:

-   Wraps and configures the MarkedJS instance
-   Maintains a registry of extension handlers
-   Creates the [Lexer](../glossary/#lexer) instance and registers all [Tokenizers](../glossary/#tokenizer).
-   Coordinates between Markdown tokens and Tiptap JSON nodes

### [](#markdown-extension)`Markdown` extension

The `Markdown` extension is the main extension that you add to your editor. It provides:

-   Overrides for all **content-related** commands on the editor to support Markdown input/output
-   The `getMarkdown()` method to serialize content as Markdown
-   The `setContent()` command with `contentType: 'markdown'` option to parse Markdown input
-   Access to the [`MarkdownManager`](#markdownmanager) instance via `editor.markdown`

### [](#extension-handlers)Extension Handlers

Each Tiptap extension can provide Markdown support by configuring the extension:

```
const MyExtension = Node.create({
  // ...

  renderMarkdown: (token, helpers) => { /* ... */ },
  parseMarkdown: (node, helpers) => { /* ... */ },
  markdownTokenizer: { /* ... */ },
})
```

The handlers translate between Markdown tokens and Tiptap nodes in both directions and are automatically registered by the [`MarkdownManager`](../glossary/#markdownmanager), creating [Tokenizers](../glossary/#tokenizer) out of them and registering those to the [Lexer](../glossary/#lexer).

Learn more about:

-   [`renderMarkdown`](../advanced-usage/custom-serializing)
-   [`parseMarkdown`](../advanced-usage/custom-parsing)
-   [`markdownTokenizer`](../advanced-usage/custom-tokenizer)


# Markdown Examples

Real-world examples and recipes for common use cases with the Markdown extension.

## [](#basic-examples)Basic Examples

### [](#read-and-write-markdown)Read and Write Markdown

This example demonstrates the most common Markdown operations:

```
import { Editor } from '@tiptap/core'
import StarterKit from '@tiptap/starter-kit'
import { Markdown } from '@tiptap/markdown'

const editor = new Editor({
  element: document.querySelector('#editor'),
  extensions: [StarterKit, Markdown],
  content: '# Hello World\n\nStart typing...',
  contentType: 'markdown', // parse initial content as Markdown
})

// Read: serialize current editor content to Markdown
console.log(editor.getMarkdown())

// Write: set editor content from a Markdown string
editor.commands.setContent('# New title\n\nSome *Markdown* content', { contentType: 'markdown' })
```

* * *

### [](#paste-markdown-detection)Paste Markdown Detection

Automatically detect and parse pasted Markdown:

```
import { Editor } from '@tiptap/core'
import StarterKit from '@tiptap/starter-kit'
import { Markdown } from '@tiptap/markdown'
import { Plugin } from '@tiptap/pm/state'

const PasteMarkdown = Extension.create({
  name: 'pasteMarkdown',

  addProseMirrorPlugins() {
    return [
      new Plugin({
        props: {
          handlePaste(view, event, slice) {
            const text = event.clipboardData?.getData('text/plain')

            if (!text) {
              return false
            }

            // Check if text looks like Markdown
            if (looksLikeMarkdown(text)) {
              const { state, dispatch } = view
              // Parse the Markdown text to Tiptap JSON using the Markdown manager
              const json = editor.markdown.parse(text)

              // Insert the parsed JSON content at cursor position
              editor.commands.insertContent(json)
              return true
            }

            return false
          },
        },
      }),
    ]
  },
})

function looksLikeMarkdown(text: string): boolean {
  // Simple heuristic: check for Markdown syntax
  return (
    /^#{1,6}\s/.test(text) || // Headings
    /\*\*[^*]+\*\*/.test(text) || // Bold
    /\[.+\]\(.+\)/.test(text) || // Links
    /^[-*+]\s/.test(text)
  ) // Lists
}

const editor = new Editor({
  extensions: [StarterKit, Markdown, PasteMarkdown],
})
```

## [](#custom-tokenizers)Custom Tokenizers

### [](#subscript-and-superscript)Subscript and Superscript

Support `~subscript~` and `^superscript^`:

```
import { Mark } from '@tiptap/core'

export const Subscript = Mark.create({
  name: 'subscript',

  parseHTML() {
    return [{ tag: 'sub' }]
  },

  renderHTML() {
    return ['sub', 0]
  },

  markdownTokenName: 'subscript',

  parseMarkdown: (token, helpers) => {
    const content = helpers.parseInline(token.tokens || [])
    return helpers.applyMark('subscript', content)
  },

  renderMarkdown: (node, helpers) => {
    const content = helpers.renderChildren(node.content || [])
    return `~${content}~`
  },

  markdownTokenizer: {
    name: 'subscript',
    level: 'inline',
    start: (src) => src.indexOf('~'),
    tokenize: (src, tokens, lexer) => {
      const match = /^~([^~]+)~/.exec(src)
      if (!match) return undefined

      return {
        type: 'subscript',
        raw: match[0], // Full match: ~text~
        text: match[1], // Content: text
        tokens: lexer.inlineTokens(match[1]), // Parse nested inline formatting
      }
    },
  },
})

export const Superscript = Mark.create({
  name: 'superscript',

  parseHTML() {
    return [{ tag: 'sup' }]
  },

  renderHTML() {
    return ['sup', 0]
  },

  markdownTokenName: 'superscript',

  parseMarkdown: (token, helpers) => {
    const content = helpers.parseInline(token.tokens || [])
    return helpers.applyMark('superscript', content)
  },

  renderMarkdown: (node, helpers) => {
    const content = helpers.renderChildren(node.content || [])
    return `^${content}^`
  },

  markdownTokenizer: {
    name: 'superscript',
    level: 'inline',
    start: (src) => src.indexOf('^'),
    tokenize: (src, tokens, lexer) => {
      const match = /^\^([^^]+)\^/.exec(src)
      if (!match) return undefined

      return {
        type: 'superscript',
        raw: match[0], // Full match: ^text^
        text: match[1], // Content: text
        tokens: lexer.inlineTokens(match[1]), // Parse nested inline formatting
      }
    },
  },
})
```

Usage:

```
editor.commands.setContent('H~2~O and E = mc^2^', { contentType: 'markdown' })
```

## [](#integration-examples)Integration Examples

### [](#real-time-markdown-preview)Real-Time Markdown Preview

You can create a real-time Markdown preview by listening to editor updates:

```
import { Editor } from '@tiptap/core'
import StarterKit from '@tiptap/starter-kit'
import { Markdown } from '@tiptap/markdown'

const editor = new Editor({
  extensions: [StarterKit, Markdown],
  content: '# Hello',
  onUpdate: ({ editor }) => {
    const markdown = editor.getMarkdown()
    updatePreview(markdown) // Your preview update function
  },
})

function updatePreview(markdown) {
  document.querySelector('#preview').textContent = markdown
}
```

### [](#saving-and-loading-workflow)Saving and Loading Workflow

Store content as Markdown and load it when needed:

```
// Save to database/storage
async function saveContent() {
  const markdown = editor.getMarkdown()
  await fetch('/api/save', {
    method: 'POST',
    body: JSON.stringify({ content: markdown }),
  })
}

// Load from database/storage
async function loadContent() {
  const { content } = await fetch('/api/load').then((r) => r.json())
  editor.commands.setContent(content, { contentType: 'markdown' })
}
```

## [](#server-side-rendering)Server-Side Rendering

Render Markdown on the server:

```
import StarterKit from '@tiptap/starter-kit'
import { MarkdownManager } from '@tiptap/markdown'
import { generateHTML } from '@tiptap/html'

const markdownManager = new MarkdownManager({
  extensions: [StarterKit, Markdown], // Include Markdown extension
})

// Parse Markdown to JSON on server
function parseMarkdown(markdown: string) {
  return editor.markdownManager.parse(markdown)
}

// Convert JSON to HTML for rendering
function renderToHTML(json: JSONContent) {
  // Generate HTML from Tiptap JSON (no Markdown involved here)
  return generateHTML(json, [StarterKit])
}

// Full pipeline: Markdown → JSON → HTML
function markdownToHTML(markdown: string) {
  const json = parseMarkdown(markdown) // Parse Markdown to JSON
  return renderToHTML(json) // Render JSON to HTML
}

// Express route example
app.get('/document/:id', async (req, res) => {
  const doc = await db.getDocument(req.params.id)
  const json = parseMarkdown(doc.markdown) // Parse stored markdown
  const html = renderToHTML(json) // Convert to HTML for display

  res.render('document', { content: html })
})
```

* * *

## [](#advanced-patterns)Advanced Patterns

### [](#lazy-loading-large-documents)Lazy Loading Large Documents

Load large documents progressively:

```
async function loadLargeDocument(documentId: string) {
  // Load metadata first
  const meta = await fetchDocumentMeta(documentId)

  // Show skeleton
  showSkeleton()

  // Load in chunks
  const chunks = await fetchDocumentChunks(documentId, meta.chunkCount)

  // Parse each Markdown chunk and insert at correct position
  for (const chunk of chunks) {
    const json = editor.markdown.parse(chunk.markdown) // Parse Markdown to JSON
    editor.commands.insertContentAt(chunk.position, json) // Insert at position
  }

  hideSkeleton()
}
```

# Markdown Glossary for Tiptap

Before we dive into the details, here are some key terms we'll be using throughout this guide:

## [](#token)Token

A plain JavaScript object that represents a piece of the parsed Markdown. For example, a heading token might look like `{ type: "heading", depth: 2, text: "Hello" }`. Tokens are the “lego bricks” that describe the document’s structure.

-   A token is a structured representation of a piece of Markdown syntax produced by the [MarkedJS](https://marked.js.org) parser.
-   Each token has a `type` (like `heading`, `paragraph`, `list`, etc.)
-   Each token may include additional properties relevant to that type (like `depth` for headings, `ordered` for lists, etc.).
-   Tokens can also contain nested tokens in properties like `tokens` or `items`, representing the hierarchical structure of the Markdown content.
-   A token is not directly usable by Tiptap; it needs to be transformed into Tiptap's JSON format.
-   Tokens are created via a [Tokenizer](#tokenizer).
-   We can create our own tokens by implementing a [Custom Tokenizer](./advanced-usage/custom-tokenizer).

> **Note**: MarkedJS comes with built-in tokenizers for standard Markdown syntax, but you can extend or replace these by providing custom tokenizers to the MarkdownManager.
> 
> You can find the list of default tokens in the [MarkedJS types](https://github.com/markedjs/marked/blob/master/src/Tokens.ts).

## [](#tiptap-json)Tiptap JSON

-   Has nothing to do with Markdown and is the JSON format used by Tiptap and ProseMirror to represent the document structure.
-   Tiptap JSON consists of nodes and marks, each with a `type`, optional `attrs`, and optional `content` or `text`.
-   Nodes represent block-level elements (like paragraphs, headings, lists), while marks represent inline formatting (like bold, italic, links).
-   Tiptap JSON is hierarchical, with nodes containing other nodes or text, reflecting the document's structure.
-   We can use **token** to create **Tiptap JSON** that the editor can understand.

Now that we understand the difference between a Token and Tiptap JSON, let's dive into how to parse tokens and serialize Tiptap content.

## [](#tokenizer)Tokenizer

The set of functions (or rules) that scan the raw Markdown text and decide how to turn chunks of it into tokens. For example, it recognizes `## Heading` and produces a `heading` token. You can customize or override tokenizers to change how Markdown is interpreted.

You can find out how to create custom tokenizers in the [Custom Tokenizers](./advanced-usage/custom-tokenizer) guide.

## [](#lexer)Lexer

The orchestrator that runs through the entire Markdown string, applies the [tokenizers](#tokenizer) in sequence, and produces the full list of tokens. Think of it as the machine that repeatedly feeds text into the tokenizers until the whole input is tokenized.

You don't need to touch the lexer directly, because Tiptap is already creating a lexer instance that will be reused for the lifetime of your editor as part of the MarkedJS instance.

This lexer instance will automatically register all [tokenizers](#tokenizer) from your extensions.



# Editor

The Markdown package does not export a new Editor class but extends the existing Tiptap Editor class. This means you can use all the standard methods and options of Tiptap's Editor, along with the additional functionality provided by the Markdown package.

## [](#methods)Methods

### [](#editorgetmarkdown)`Editor.getMarkdown()`

Get the current content of the editor as Markdown.

-   **returns**: `string`

```
const markdown = editor.getMarkdown()
```

## [](#properties)Properties

### [](#editormarkdown)`Editor.markdown`

Access the MarkdownManager instance.

```
editor.markdown: MarkdownManager
```

#### [](#example)Example

```
// Parse Markdown to JSON
const json = editor.markdown.parse('# Hello')

// Serialize JSON to Markdown
const markdown = editor.markdown.serialize(json)

// Access marked instance
const marked = editor.markdown.instance
```

## [](#options)Options

### [](#editorcontent)`Editor.content`

Editor content supports **HTML**, **Markdown** or **Tiptap JSON** as a value.

> **Note**: For Markdown support `editor.contentAsMarkdown` must be set to `true`.

-   **type**: `string | object`
-   **default**: `''`
-   **required**: `false`

```
const editor = new Editor({
  content: '<h1>Hello world</h1>',
})
```

```
const editor = new Editor({
  content: [
    { type: 'heading', attrs: { level: 1 }, content: [{ type: 'text', text: 'Hello world' }] },
  ],
})
```

```
const editor = new Editor({
  content: '# Hello world',
  contentType: 'markdown',
})
```

### [](#editorcontenttype)`Editor.contentType`

Defines what type of content is passed to the editor. Defaults to `json`. When an invalid combination is set - for example content that is a JSON object, but the contentType is set to `markdown`, the editor will automatically fall back to `json` and vice versa.

-   **type**: `string`
-   **default**: `json`
-   **required**: `false`
-   **options**: `json`, `html`, `markdown`

```
const editor = new Editor({
  content: '# Hello world',
  contentType: 'markdown',
})
```

## [](#command-options)Command Options

### [](#setcontentcontent-options)`setContent(content, options)`

Set editor content from Markdown.

```
editor.commands.setContent(
  content: string,
  options?: {
    contentType?: string,
    emitUpdate?: boolean,
    parseOptions?: ParseOptions,
  }
): boolean
```

#### [](#parameters)Parameters

-   **`content`**: The Markdown string to set
-   **`options.contentType`**: The type of content inserted, can be `json`, `html` or `markdown`. Autodetects if formats don't match (default: `json`)
-   **`options.emitUpdate`**: Whether to emit an update event (default: `true`)
-   **`options.parseOptions`**: Additional parse options

#### [](#returns)Returns

`boolean` - Whether the command succeeded

#### [](#example)Example

```
editor.commands.setContent('# New Content\n\nThis is **bold**.', { contentType: 'markdown' })
```

* * *

### [](#insertcontentvalue-options)`insertContent(value, options)`

Insert Markdown content at the current selection.

```
editor.commands.insertContent(
  value: string,
  options?: {
    contentType?: string,
    parseOptions?: ParseOptions,
    updateSelection?: boolean,
  }
): boolean
```

#### [](#parameters)Parameters

-   **`value`**: The Markdown string to insert
-   **`options.contentType`**: The type of content inserted, can be `json`, `html` or `markdown`. Autodetects if formats don't match (default: `json`)
-   **`options.updateSelection`**: Whether to update selection after insert
-   **`options.parseOptions`**: Additional parse options

#### [](#returns)Returns

`boolean` - Whether the command succeeded

#### [](#example)Example

```
editor.commands.insertContent('**Bold text** at cursor', { contentType: 'markdown' })
```

* * *

### [](#insertcontentatposition-value-options)`insertContentAt(position, value, options)`

Insert Markdown content at a specific position.

```
editor.commands.insertContentAt(
  position: number | Range,
  value: string,
  options?: {
    contentType?: string,
    parseOptions?: ParseOptions,
    updateSelection?: boolean,
  }
): boolean
```

#### [](#parameters)Parameters

-   **`position`**: Position (number) or range (`{ from, to }`)
-   **`value`**: The Markdown string to insert
-   **`options.contentType`**: The type of content inserted, can be `json`, `html` or `markdown`. Autodetects if formats don't match (default: `json`)
-   **`options.updateSelection`**: Whether to update selection after insert
-   **`options.parseOptions`**: Additional parse options

#### [](#returns)Returns

`boolean` - Whether the command succeeded

#### [](#example)Example

```
// Insert at position
editor.commands.insertContentAt(10, '## Heading', { contentType: 'markdown' })

// Replace range
editor.commands.insertContentAt({ from: 10, to: 20 }, '**replacement**', { contentType: 'markdown' })
```

* * *

## [](#extension-spec)Extension Spec

The extension spec also gets extended with the following options:

### [](#markdowntokenname)`markdownTokenName`

The name of the token used for Markdown parsing.

-   **type**: `string`
-   **default**: `undefined`
-   **required**: `false`

```
const CustomNode = Node.create({
  // ...

  markdownTokenName: 'custom_token',
})
```

### [](#parsemarkdown)`parseMarkdown`

A function to customize how Markdown tokens are parsed from Markdown token into ProseMirror nodes.

-   **type**: `(token: MarkdownToken, helpers: MarkdownParseHelpers) => ProseMirrorNode[] | null`
-   **default**: `undefined`
-   **required**: `false`

```
const CustomNode = Node.create({
  // ...

  parseMarkdown: (token, helpers) => {
    return {
      type: 'customNode',
      attrs: { type: token.type },
      content: helpers.parseChildren(token.tokens || [])
    }
  },
})
```

### [](#rendermarkdown)`renderMarkdown`

A function to customize how ProseMirror nodes are rendered to Markdown tokens.

-   **type**: `(node: JSONContent, helpers: MarkdownRenderHelpers, context: RenderContext) => string | null`
-   **default**: `undefined`
-   **required**: `false`

```
const CustomNode = Node.create({
  // ...

  renderMarkdown: (node, helpers, context) => {
    const content = helpers.renderChildren(node.content, context)

    return `[${context.parentType}] ${content}`
  },
})
```

### [](#markdowntokenizer)`markdownTokenizer`

A tokenizer configuration object that creates a custom tokenizer to turn Markdown string into tokens.

-   **type**: `object`
-   **default**: `undefined`
-   **required**: `false`

```
const CustomNode = Node.create({
  // ...

  // example tokenizer that matches ::custom text::
  markdownTokenizer: {
    name: 'custom_token',
    level: 'inline',
    start(src) { return src.indexOf('::') },
    tokenizer(src, tokens) {
      const rule = /^::(.*?)::/
      const match = rule.exec(src)
      if (match) {
        return {
          type: 'custom_token',
          raw: match[0],
          text: match[1].trim(),
        }
      }
    },
  },
})
```

### [](#markdownoptions)`markdownOptions`

A optional object to pass additional options to the Markdown parser and serializer.

-   **type**: `object`
-   **default**: `undefined`
-   **required**: `false`

```
const CustomNode = Node.create({
  // ...

  markdownOptions: {
    indentsContent: true, // this setting will cause the indent level in the render context to increase inside this node
  }
})
```

---
> Source: [frappe/wiki](https://github.com/frappe/wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
