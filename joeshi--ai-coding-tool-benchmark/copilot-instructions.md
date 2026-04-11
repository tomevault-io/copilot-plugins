## ai-coding-tool-benchmark

> 本文档比较了由 Kiro 和 Cursor 两个 AI 编程助手生成的代码，两者都使用了相同的提示词（见各自项目中的 `prompt.md`）。

# Kiro vs. Cursor 代码比较

本文档比较了由 Kiro 和 Cursor 两个 AI 编程助手生成的代码，两者都使用了相同的提示词（见各自项目中的 `prompt.md`）。

## 总体评价

Kiro 生成的代码在结构、组织和现代开发实践方面明显优于 Cursor。Kiro 的代码更易于维护、扩展和协作。

## 详细比较

### 1. 项目结构

*   **Cursor (`cursor-landing-page`)**: 项目结构非常扁平。所有的 UI 和逻辑都放在一个 `App.js` 文件中。这是一种单体式的方法，对于小型项目来说尚可，但随着项目复杂度的增加，会变得难以管理。

*   **Kiro (`kiro-landing-page`)**: 采用了组件化的结构。代码被拆分成多个可重用的组件，分别存放在 `src/components` 目录下。此外，它还创建了 `hooks`、`styles` 和 `utils` 等目录来组织不同类型的代码。这种模块化的方法是现代 Web 开发的最佳实践。

### 2. 代码风格和组织

*   **Cursor**: 代码集中在 `App.js` 中，导致文件过长，可读性差。组件没有被拆分，难以复用。

*   **Kiro**: `App.js` 非常简洁，只负责渲染一个顶层的 `LandingPage` 组件。`LandingPage.js` 则作为页面的组装器，从 `components` 目录中导入并组合各个子组件。它还使用一个 `pageData` 对象来管理所有文本和图片路径，实现了数据与视图的分离，这使得内容更新变得非常容易。

### 3. 样式 (CSS)

*   **Cursor**: 所有样式都写在 `App.css` 一个文件中。没有使用 CSS 变量，这意味着如果要更改主题颜色或字体，需要在多处进行修改，容易出错。

*   **Kiro**: 采用了更现代、更具可维护性的 CSS 策略。
    *   **CSS 变量**: 在 `:root` 中定义了颜色、字体、间距等设计系统的变量，方便全局样式的统一管理和修改。
    *   **模块化 CSS**: 将 CSS 分成 `components.css`（组件特定样式）和 `utilities.css`（可重用的工具类），提高了代码的组织性和复用性。
    *   **响应式设计**: 在 CSS 文件中使用了媒体查询，以确保在不同设备上都能有良好的显示效果。

### 4. 资源管理

*   **Cursor**: 所有图片资源都存放在 `src/assets` 文件夹中。

*   **Kiro**: 图片资源存放在 `public/images` 文件夹中，这更符合 Create React App 的标准实践。

## 结论

Kiro 生成的代码展示了对现代前端开发最佳实践的深刻理解，包括组件化架构、关注点分离、可维护的 CSS 和良好的项目组织。相比之下，Cursor 生成的代码虽然能够实现功能，但在代码质量和可维护性方面有很大的提升空间。

对于任何严肃的项目，Kiro 生成的代码都是一个更好的起点。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JoeShi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JoeShi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
