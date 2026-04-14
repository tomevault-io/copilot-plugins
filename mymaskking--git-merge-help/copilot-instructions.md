## git-merge-help

> 这是一个基于Next.js的Git merge管理工具，用于可视化展示和处理git merge操作。主要设计文档在[desgin.md](mdc:desgin.md)中定义。

# Git Merge 管理工具项目结构规则

## 项目概述
这是一个基于Next.js的Git merge管理工具，用于可视化展示和处理git merge操作。主要设计文档在[desgin.md](mdc:desgin.md)中定义。

## 目录结构规范

### 根目录结构
```
/
├── src/                    # 源代码目录
│   ├── app/               # Next.js应用目录
│   ├── components/        # React组件
│   ├── lib/              # 工具库和辅助函数
│   └── styles/           # 全局样式
├── public/                # 静态资源
└── .cursor/              # Cursor IDE配置
```

### 组件目录结构 (src/components/)
```
components/
├── git/                  # Git相关组件
│   ├── BranchManager/    # 分支管理组件
│   ├── MergeViewer/     # 合并视图组件
│   └── ConflictResolver/ # 冲突解决组件
├── ui/                   # 通用UI组件
└── layout/              # 布局组件
```

### 工具库目录结构 (src/lib/)
```
lib/
├── git/                  # Git操作相关
├── github/              # GitHub API集成
└── utils/              # 通用工具函数
```

## 代码规范

### 文件命名
- 组件文件使用PascalCase: `BranchSelector.tsx`
- 工具函数文件使用camelCase: `gitOperations.ts`
- 样式文件使用kebab-case: `branch-selector.css`

### 导入顺序
1. React/Next.js相关
2. 第三方库
3. 内部组件
4. 工具函数
5. 类型定义
6. 样式文件

### 组件结构
- 每个组件目录包含：
  - index.tsx: 主组件文件
  - types.ts: 类型定义
  - styles.css: 样式文件（如果需要）
  - utils.ts: 组件相关工具函数
  - test.tsx: 测试文件

## 技术栈规范
- Next.js 14+
- TypeScript
- TailwindCSS
- shadcn/ui
- isomorphic-git
- Monaco Editor

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MyMaskKing)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MyMaskKing)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
