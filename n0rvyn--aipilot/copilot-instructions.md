## aipilot

> Markdown 渲染和 CodeMirror 检查清单


# Markdown 渲染检查清单

> **重要性**: ⭐⭐⭐⭐ 高优先级  
> **加载方式**: Auto-Attach（编辑 Markdown 相关文件时自动加载）  
> **用途**: Markdown 渲染和 CodeMirror 集成的最佳实践  

---

## 🔒 安全渲染

### 1. 使用 Obsidian 的 MarkdownRenderer

```typescript
import { MarkdownRenderer } from 'obsidian';

// ✅ 推荐：使用 Obsidian 的安全渲染器
async function renderMarkdown(markdown: string, container: HTMLElement) {
    container.empty();
    
    await MarkdownRenderer.render(
        this.app,
        markdown,
        container,
        '', // sourcePath（可选）
        this // component
    );
}

// ❌ 危险：直接插入 HTML
container.innerHTML = marked(markdown); // XSS 风险！
```

### 2. 清理用户输入

```typescript
// ✅ Obsidian 的渲染器会自动处理
await MarkdownRenderer.render(
    this.app,
    userInput, // 安全，会自动清理
    container,
    '',
    this
);

// ✅ 如果必须使用 marked，配置安全选项
import { marked } from 'marked';

marked.setOptions({
    sanitize: false, // marked 不再支持 sanitize
    breaks: true,
    gfm: true
});

// 使用 DOMPurify 清理（如果需要）
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(html);
```

---

## 📝 Markdown 渲染模式

### Obsidian MarkdownRenderer API

```typescript
// 基本用法
await MarkdownRenderer.render(
    this.app,              // App 实例
    markdown,              // Markdown 字符串
    container,             // 渲染目标容器
    sourcePath,            // 源文件路径（用于链接解析）
    component              // 组件实例（用于生命周期管理）
);

// 渲染后处理
await MarkdownRenderer.render(this.app, markdown, container, '', this);

// 添加自定义类
container.addClass('my-markdown-content');

// 处理内部链接
container.querySelectorAll('a.internal-link').forEach(link => {
    link.addEventListener('click', (e) => {
        e.preventDefault();
        const target = link.getAttribute('data-href');
        this.app.workspace.openLinkText(target, '');
    });
});
```

---

## 🎨 代码高亮

### 使用 CodeMirror 渲染代码块

```typescript
import { EditorView } from '@codemirror/view';
import { EditorState } from '@codemirror/state';
import { javascript } from '@codemirror/lang-javascript';
import { python } from '@codemirror/lang-python';
import { oneDark } from '@codemirror/theme-one-dark';

// ✅ 创建 CodeMirror 编辑器
function renderCodeBlock(code: string, language: string, container: HTMLElement) {
    const extensions = [
        EditorView.editable.of(false), // 只读
        EditorState.readOnly.of(true),
        getLanguageExtension(language),
        oneDark // 主题（可选）
    ];
    
    const state = EditorState.create({
        doc: code,
        extensions
    });
    
    const view = new EditorView({
        state,
        parent: container
    });
    
    return view;
}

function getLanguageExtension(language: string) {
    switch (language.toLowerCase()) {
        case 'javascript':
        case 'js':
            return javascript();
        case 'python':
        case 'py':
            return python();
        // 添加更多语言支持
        default:
            return [];
    }
}
```

### 代码块复制功能

```typescript
// ✅ 添加复制按钮
function addCopyButton(codeBlock: HTMLElement, code: string) {
    const copyBtn = codeBlock.createEl('button', {
        text: 'Copy',
        cls: 'copy-code-button'
    });
    
    copyBtn.addEventListener('click', async () => {
        try {
            await navigator.clipboard.writeText(code);
            copyBtn.textContent = 'Copied!';
            setTimeout(() => {
                copyBtn.textContent = 'Copy';
            }, 2000);
        } catch (error) {
            console.error('Failed to copy:', error);
        }
    });
}
```

---

## 🎯 自定义 Markdown 处理

### 扩展 Markdown 语法

```typescript
import { marked } from 'marked';

// ✅ 自定义渲染器
const renderer = new marked.Renderer();

// 自定义链接渲染
renderer.link = (href, title, text) => {
    const isInternal = href?.startsWith('[[') && href?.endsWith(']]');
    
    if (isInternal) {
        const target = href.slice(2, -2);
        return `<a class="internal-link" data-href="${target}">${text}</a>`;
    }
    
    return `<a href="${href}" title="${title || ''}" target="_blank">${text}</a>`;
};

// 自定义代码块渲染
renderer.code = (code, language) => {
    const lang = language || 'text';
    return `<div class="code-block" data-language="${lang}">
        <pre><code class="language-${lang}">${escapeHtml(code)}</code></pre>
    </div>`;
};

// 使用自定义渲染器
marked.use({ renderer });
```

### 处理数学公式（LaTeX）

```typescript
// ✅ 使用 Obsidian 的数学渲染
async function renderMath(latex: string, container: HTMLElement, displayMode: boolean) {
    // Obsidian 会自动处理 $...$ 和 $$...$$ 语法
    // 通过 MarkdownRenderer.render() 即可
    await MarkdownRenderer.render(
        this.app,
        displayMode ? `$$${latex}$$` : `$${latex}$`,
        container,
        '',
        this
    );
}
```

---

## 🎭 流式渲染（Streaming）

### 实时更新 Markdown 内容

```typescript
class StreamingRenderer {
    private container: HTMLElement;
    private currentContent: string = '';
    
    constructor(container: HTMLElement) {
        this.container = container;
    }
    
    // ✅ 追加内容并重新渲染
    async append(chunk: string) {
        this.currentContent += chunk;
        await this.render();
    }
    
    // ✅ 渲染当前内容
    async render() {
        this.container.empty();
        await MarkdownRenderer.render(
            this.app,
            this.currentContent,
            this.container,
            '',
            this
        );
    }
    
    // ✅ 清空内容
    clear() {
        this.currentContent = '';
        this.container.empty();
    }
}

// 使用示例
const renderer = new StreamingRenderer(container);

// 接收流式数据
for await (const chunk of streamResponse) {
    await renderer.append(chunk);
}
```

### 性能优化：节流渲染

```typescript
// ✅ 避免频繁重新渲染
class ThrottledRenderer {
    private container: HTMLElement;
    private buffer: string = '';
    private timeout: NodeJS.Timeout | null = null;
    private readonly RENDER_DELAY = 100; // ms
    
    async append(chunk: string) {
        this.buffer += chunk;
        
        // 取消之前的渲染计划
        if (this.timeout) {
            clearTimeout(this.timeout);
        }
        
        // 延迟渲染
        this.timeout = setTimeout(async () => {
            await this.render();
            this.timeout = null;
        }, this.RENDER_DELAY);
    }
    
    async flush() {
        // 立即渲染剩余内容
        if (this.timeout) {
            clearTimeout(this.timeout);
            this.timeout = null;
        }
        await this.render();
    }
    
    private async render() {
        this.container.empty();
        await MarkdownRenderer.render(
            this.app,
            this.buffer,
            this.container,
            '',
            this
        );
    }
}
```

---

## 📋 常见问题

### 1. 内部链接不工作

```typescript
// ✅ 处理 Obsidian 内部链接
container.querySelectorAll('a.internal-link').forEach(link => {
    link.addEventListener('click', async (e) => {
        e.preventDefault();
        const target = link.getAttribute('data-href');
        if (target) {
            // 打开笔记
            this.app.workspace.openLinkText(target, '', false);
        }
    });
});
```

### 2. 代码块滚动问题

```css
/* 添加到 styles.css */
.code-block {
    overflow-x: auto;
    max-width: 100%;
}

.code-block pre {
    margin: 0;
    padding: 1em;
}
```

### 3. 数学公式渲染失败

```typescript
// ✅ 确保 Obsidian 的数学渲染已加载
// 通常通过 MarkdownRenderer.render() 自动处理
// 如果有问题，检查 Obsidian 设置中的 MathJax 配置
```

---

## 🎨 样式定制

### Markdown 内容样式

```css
/* styles.css */

/* 基础样式 */
.markdown-content {
    font-size: 14px;
    line-height: 1.6;
    color: var(--text-normal);
}

/* 标题 */
.markdown-content h1 { font-size: 2em; }
.markdown-content h2 { font-size: 1.5em; }
.markdown-content h3 { font-size: 1.25em; }

/* 代码块 */
.markdown-content pre {
    background: var(--code-background);
    padding: 1em;
    border-radius: 4px;
    overflow-x: auto;
}

.markdown-content code {
    background: var(--code-background);
    padding: 0.2em 0.4em;
    border-radius: 3px;
    font-family: var(--font-monospace);
}

/* 链接 */
.markdown-content a {
    color: var(--link-color);
    text-decoration: none;
}

.markdown-content a:hover {
    text-decoration: underline;
}

/* 列表 */
.markdown-content ul, .markdown-content ol {
    padding-left: 2em;
}

/* 引用 */
.markdown-content blockquote {
    border-left: 3px solid var(--quote-opening-modifier);
    padding-left: 1em;
    margin-left: 0;
    color: var(--text-muted);
}
```

---

## 📋 检查清单

### Markdown 渲染检查

- [ ] **安全性**
  - [ ] 使用 Obsidian MarkdownRenderer
  - [ ] 用户输入已清理
  - [ ] 避免直接 innerHTML

- [ ] **功能完整性**
  - [ ] 内部链接正常工作
  - [ ] 代码高亮正确显示
  - [ ] 数学公式正确渲染
  - [ ] 图片正确加载

- [ ] **性能**
  - [ ] 流式渲染使用节流
  - [ ] 大文本不阻塞 UI
  - [ ] 及时清理 DOM

- [ ] **用户体验**
  - [ ] 代码块可复制
  - [ ] 样式符合主题
  - [ ] 移动端适配

---

## 🔗 相关资源

- **Obsidian MarkdownRenderer**: https://docs.obsidian.md/Reference/TypeScript+API/MarkdownRenderer
- **CodeMirror 6**: https://codemirror.net/docs/
- **marked**: https://marked.js.org/

---

**最后更新**: 2025-11-09  
**版本**: 1.0  
**Token 估算**: ~600 tokens  
**加载方式**: Auto-Attach  
**优先级**: 102

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/n0rvyn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
