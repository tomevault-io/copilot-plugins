## html-css

> HTML和CSS语言特定规则


# HTML和CSS语言特定规则

基于awesome-cursorrules的HTML/CSS最佳实践。

## HTML最佳实践

### 语义化HTML
```html
<!-- 使用语义化标签 -->
<header>
  <nav>
    <ul>
      <li><a href="/">首页</a></li>
      <li><a href="/about">关于</a></li>
    </ul>
  </nav>
</header>

<main>
  <article>
    <header>
      <h1>文章标题</h1>
      <time datetime="2024-01-01">2024年1月1日</time>
    </header>
    <section>
      <p>文章内容...</p>
    </section>
  </article>
</main>

<footer>
  <p>&copy; 2024 公司名称</p>
</footer>
```

### 可访问性
```html
<!-- 添加适当的ARIA标签 -->
<button aria-label="关闭对话框" aria-expanded="false">
  <span aria-hidden="true">&times;</span>
</button>

<!-- 表单标签关联 -->
<label for="email">邮箱地址</label>
<input type="email" id="email" name="email" required>

<!-- 图片alt属性 -->
<img src="hero.jpg" alt="产品展示图" loading="lazy">
```

### 性能优化
```html
<!-- 预加载关键资源 -->
<link rel="preload" href="/fonts/main.woff2" as="font" type="font/woff2" crossorigin>

<!-- 延迟加载非关键资源 -->
<script src="analytics.js" defer></script>

<!-- 响应式图片 -->
<picture>
  <source media="(min-width: 768px)" srcset="large.jpg">
  <source media="(min-width: 480px)" srcset="medium.jpg">
  <img src="small.jpg" alt="响应式图片">
</picture>
```

## CSS最佳实践

### 现代CSS特性
```css
/* 使用CSS自定义属性 */
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --font-family: 'Inter', sans-serif;
  --border-radius: 8px;
  --spacing-unit: 1rem;
}

/* 使用Grid布局 */
.container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: var(--spacing-unit);
}

/* 使用Flexbox */
.flex-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
}
```

### 响应式设计
```css
/* 移动优先的响应式设计 */
.card {
  padding: 1rem;
  margin-bottom: 1rem;
}

/* 平板设备 */
@media (min-width: 768px) {
  .card {
    padding: 1.5rem;
    margin-bottom: 1.5rem;
  }
}

/* 桌面设备 */
@media (min-width: 1024px) {
  .card {
    padding: 2rem;
    margin-bottom: 2rem;
  }
}
```

### CSS架构
```css
/* 使用BEM命名规范 */
.button {
  display: inline-block;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: var(--border-radius);
  cursor: pointer;
  transition: all 0.2s ease;
}

.button--primary {
  background-color: var(--primary-color);
  color: white;
}

.button--primary:hover {
  background-color: color-mix(in srgb, var(--primary-color) 80%, black);
}

.button--large {
  padding: 0.75rem 1.5rem;
  font-size: 1.125rem;
}

.button__icon {
  margin-right: 0.5rem;
}
```

### 性能优化
```css
/* 使用transform和opacity进行动画 */
.animate {
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.animate:hover {
  transform: translateY(-2px);
  opacity: 0.8;
}

/* 避免重排和重绘 */
.optimized {
  will-change: transform;
  transform: translateZ(0);
}

/* 使用contain属性优化渲染 */
.isolated {
  contain: layout style paint;
}
```

## SCSS/Sass最佳实践

### 变量和混入
```scss
// 变量定义
$primary-color: #007bff;
$secondary-color: #6c757d;
$breakpoints: (
  mobile: 480px,
  tablet: 768px,
  desktop: 1024px
);

// 混入定义
@mixin button-style($bg-color, $text-color: white) {
  background-color: $bg-color;
  color: $text-color;
  border: none;
  border-radius: 4px;
  padding: 0.5rem 1rem;
  cursor: pointer;
  
  &:hover {
    background-color: darken($bg-color, 10%);
  }
}

// 使用混入
.primary-button {
  @include button-style($primary-color);
}

.secondary-button {
  @include button-style($secondary-color);
}
```

### 嵌套和模块化
```scss
// 组件样式
.card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  
  &__header {
    padding: 1rem;
    border-bottom: 1px solid #eee;
    
    &-title {
      margin: 0;
      font-size: 1.25rem;
      font-weight: 600;
    }
  }
  
  &__body {
    padding: 1rem;
  }
  
  &__footer {
    padding: 1rem;
    background: #f8f9fa;
    border-top: 1px solid #eee;
  }
  
  // 响应式设计
  @media (min-width: map-get($breakpoints, tablet)) {
    &__header,
    &__body,
    &__footer {
      padding: 1.5rem;
    }
  }
}
```

## 工具类CSS

```css
/* 实用工具类 */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
}

.text-center { text-align: center; }
.text-left { text-align: left; }
.text-right { text-align: right; }

.mb-1 { margin-bottom: 0.25rem; }
.mb-2 { margin-bottom: 0.5rem; }
.mb-3 { margin-bottom: 1rem; }
.mb-4 { margin-bottom: 1.5rem; }

.d-flex { display: flex; }
.d-grid { display: grid; }
.d-none { display: none; }

@media (min-width: 768px) {
  .d-md-block { display: block; }
  .d-md-flex { display: flex; }
}
```

## 现代CSS特性

```css
/* CSS Grid高级用法 */
.grid-layout {
  display: grid;
  grid-template-areas: 
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 200px 1fr;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.footer { grid-area: footer; }

/* CSS自定义属性动画 */
@property --progress {
  syntax: '<percentage>';
  initial-value: 0%;
  inherits: false;
}

.progress-bar {
  --progress: 0%;
  background: linear-gradient(90deg, #007bff var(--progress), #eee var(--progress));
  transition: --progress 0.3s ease;
}

/* 容器查询 */
.card {
  container-type: inline-size;
}

@container (min-width: 300px) {
  .card__content {
    display: flex;
    flex-direction: row;
  }
}
```

---
> Source: [shinytsing/modeshift_django](https://github.com/shinytsing/modeshift_django) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
