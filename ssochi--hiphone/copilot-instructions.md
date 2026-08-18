## hiphone

> 1. **系统UI高仿iOS** — Shell层必须还原iOS的视觉与交互范式：Dock栏、毛玻璃材质、居中导航标题、左箭头返回、顶部横幅通知、SF风格排版、Dynamic Island、状态栏分层。每个系统组件（桌面、导航栏、通知、Dock）都应以iOS对应元素为参照，不做自创风格。



## 设计原则
1. **系统UI高仿iOS** — Shell层必须还原iOS的视觉与交互范式：Dock栏、毛玻璃材质、居中导航标题、左箭头返回、顶部横幅通知、SF风格排版、Dynamic Island、状态栏分层。每个系统组件（桌面、导航栏、通知、Dock）都应以iOS对应元素为参照，不做自创风格。

## 规范要求
1. 文档优先,构建自己的文档系统(docs/). 有效使用子目录的`AGENTS.md`,将需要阅读该目录需要了解的规范,以及反复做错踩坑点记录在对应目录的`AGENTS.md`中.注意维护文档的目的是避免上下文在开发中丢失,在一次开发中大部分上下文转化为代码,少部分上下文转化为关键注释,还有一部分应该变成文档,否则就会永久丢失.
2. 执行计划前,先写docs/plan/计划文档. 计划文档文件名 为 yyyy-mm-dd-hhmm-计划名.md. 计划中必须包含详细的用户需求,和你的关键决策.
3. 测试优先,项目需要具备完善的单测系统.
4. 大需求出里程碑,里程碑拆解为多个阶段,每个阶段对应一个plan+具体开发

---
> Source: [ssochi/hiphone](https://github.com/ssochi/hiphone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
