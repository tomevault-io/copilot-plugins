## glassforge

> 这是一个专业的液态玻璃效果生成器，灵感来源于苹果WWDC 2024设计语言。

# GlassForge - Liquid Glass Effect Generator
# Cursor IDE Rules & Guidelines

## 项目概述
这是一个专业的液态玻璃效果生成器，灵感来源于苹果WWDC 2024设计语言。
目标：创造世界上最先进的液态玻璃UI效果生成工具。

## 命令运行
标记“&&”不是此版本中的有效语句分隔符。不用“&&”，分步骤来

## 🎯 核心原则

### 关键词策略（不可妥协）
- 主关键词: "liquid glass" (月搜索量8.1K)
- 核心长尾词: "liquid glass generator online free"
- 所有代码和内容必须服务于SEO目标

### 开发语言规范
- 网站界面内容: 英文（面向海外用户）
- 开发文档和注释: 中文（团队内部交流）
- 代码变量和函数: 英文（国际化标准）
- 绝不在代码中使用中文变量名

## 🛠️ 技术栈规范

### 强制技术栈
- Framework: Next.js 14 (App Router)
- Language: TypeScript (严格模式)
- Styling: Tailwind CSS + 自定义玻璃效果
- 3D Graphics: Three.js + React Three Fiber
- State: Zustand
- Animation: Framer Motion

### 禁止使用
- 任何 'any' 类型
- Vite (必须使用Next.js)
- 中文变量名或函数名
- 内联样式（必须使用Tailwind或CSS类）

## 📁 文件结构规范

```
src/
├── app/                    # Next.js 14 App Router页面
├── components/             # 可复用UI组件
│   ├── ui/                # 基础UI组件
│   ├── glass/             # 玻璃效果专用组件
│   └── layout/            # 布局组件
├── lib/                   # 工具函数和核心逻辑
├── hooks/                 # 自定义React hooks
├── types/                 # TypeScript类型定义
└── styles/                # 全局样式
```

## 💻 代码规范

### TypeScript 规则
```typescript
// ✅ 正确: 所有组件必须有完整类型定义
interface GlassEffectProps {
  transparency: number;
  onEffectChange: (effect: GlassEffect) => void;
}

// ✅ 正确: 使用描述性英文函数名
const generateLiquidGlassCSS = (params: GlassParams): string => {
  // 中文注释解释逻辑
  return cssCode;
}

// ❌ 错误: 禁止使用any类型
const handleClick = (data: any) => {}

// ❌ 错误: 禁止中文变量名
const 玻璃效果 = {};
```

### React 组件规范
```typescript
// ✅ 正确: 函数式组件 + TypeScript
const LiquidGlassGenerator: React.FC<Props> = ({ children }) => {
  // 使用useMemo优化性能
  const glassCSS = useMemo(() => generateCSS(params), [params]);
  
  // 使用useCallback优化事件处理
  const handleChange = useCallback((value: number) => {
    // 处理逻辑
  }, []);

  return <div className="glass-card">{children}</div>;
};

// ❌ 错误: 类组件
class GlassGenerator extends Component {}
```

### CSS/Tailwind 规范
```css
/* ✅ 正确: 使用自定义玻璃效果类 */
.glass-card {
  @apply backdrop-blur-lg bg-white/10 border border-white/20;
}

/* ✅ 正确: 苹果风格缓动函数 */
.apple-transition {
  transition: all 0.3s cubic-bezier(0.4, 0.0, 0.2, 1);
}

/* ❌ 错误: 内联样式 */
<div style={{backdropFilter: 'blur(10px)'}} />
```

## 🎨 设计系统规范

### 颜色系统
```css
:root {
  /* 玻璃效果专用 */
  --glass-white: rgba(255, 255, 255, 0.1);
  --glass-border: rgba(255, 255, 255, 0.2);
  
  /* 苹果官方色彩 */
  --apple-blue: #007AFF;
  --apple-purple: #5856D6;
}
```

### 组件命名
- 玻璃效果组件: `glass-*` 前缀
- 苹果风格动画: `apple-*` 前缀
- 液态玻璃相关: `liquid-glass-*` 前缀

## 📝 内容和SEO规范

### 页面内容规则
```markdown
✅ 必须包含的关键词:
- 页面标题: "Liquid Glass" 
- H1标签: 完整关键词
- 前100词: 2-3次关键词出现
- Alt标签: 优化图片描述

✅ 长尾关键词分布:
- 首页: "liquid glass generator online free"
- 生成器页: "apple liquid glass effect css"
- 教程页: "how to make liquid glass effect"
```

### Meta数据标准
```typescript
export const metadata: Metadata = {
  title: 'Liquid Glass Generator - Create Apple-Style Glass Effects',
  description: '...',
  keywords: ['liquid glass', 'glass effect generator', ...],
  openGraph: { ... }
}
```

## ⚡ 性能要求

### 必须达到的指标
- 首次内容绘制 (FCP): < 1.5s
- 最大内容绘制 (LCP): < 2.5s
- 累积布局偏移 (CLS): < 0.1
- 首次输入延迟 (FID): < 100ms

### 优化规则
```typescript
// ✅ 必须: 大型组件懒加载
const ThreeJSPreview = dynamic(() => import('./ThreeJSPreview'), {
  ssr: false,
  loading: () => <Skeleton />
});

// ✅ 必须: 计算优化
const expensiveValue = useMemo(() => calculate(data), [data]);

// ✅ 必须: 事件处理优化
const handler = useCallback((e) => { ... }, [deps]);
```

## 🔧 开发工作流

### Git 提交规范
```bash
# ✅ 正确格式
feat: add liquid glass parameter controls
fix: resolve transparency calculation bug
style: update glass effect animations
docs: add glass generator API documentation
perf: optimize WebGL shader performance

# ❌ 错误格式
fix: bug
update: stuff
```

### 分支命名
- feature/glass-effect-generator
- fix/transparency-calculation-bug
- hotfix/critical-rendering-issue

## 🧪 测试规范

### 必须测试的功能
```typescript
// ✅ 核心逻辑必须测试
describe('GlassEffectGenerator', () => {
  test('应该生成有效的CSS代码', () => {
    const params = { transparency: 80, blur: 10 };
    const css = generateGlassCSS(params);
    expect(css).toContain('backdrop-filter: blur(10px)');
  });
});

// ✅ 用户交互流程测试
test('用户应该能够调整参数并导出代码', async () => {
  // E2E测试逻辑
});
```

### 覆盖率要求
- 核心逻辑函数: 90%+
- UI组件: 80%+
- 工具函数: 95%+

## 🚫 绝对禁止事项

1. **代码质量**
   - 使用any类型
   - 中文变量名
   - 内联样式
   - 未优化的大型组件

2. **性能要求**
   - 响应时间 > 100ms的交互
   - 未压缩的图片资源
   - 阻塞渲染的脚本

3. **SEO违规**
   - 缺少meta标签的页面
   - 未优化关键词的内容
   - 动态URL用于SEO页面

## ✅ 开发检查清单

### 每次提交前必须检查
- [ ] TypeScript编译无错误
- [ ] ESLint检查通过
- [ ] 所有组件有PropTypes/Interface
- [ ] 性能优化(useMemo/useCallback)
- [ ] 响应式设计测试
- [ ] 关键词优化检查
- [ ] 玻璃效果渲染正常

### 功能完成前必须检查
- [ ] 多浏览器兼容性测试
- [ ] 移动端适配测试
- [ ] 性能指标达标
- [ ] SEO元数据完整
- [ ] 用户体验流畅
- [ ] 代码导出功能正常

## 🎯 AI助手指导原则

### 当我询问代码问题时
1. 优先考虑性能和SEO影响
2. 确保符合苹果设计风格
3. 使用项目规定的技术栈
4. 提供TypeScript完整类型
5. 包含中文注释解释逻辑

### 当我需要新功能时
1. 先评估对核心关键词的影响
2. 确保符合玻璃效果主题
3. 考虑多平台兼容性
4. 提供性能优化建议
5. 包含测试用例建议

### 代码审查标准
- 是否符合苹果设计理念？
- 是否有助于"liquid glass"关键词排名？
- 性能是否达到要求？
- TypeScript类型是否完整？
- 是否遵循项目架构？

---

**记住：我们的目标是创造世界上最好的液态玻璃效果生成器，每一行代码都要服务于这个目标！** 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alen-guo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
