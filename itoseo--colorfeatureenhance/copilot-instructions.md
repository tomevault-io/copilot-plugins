## this

> 此项目为ColorOS特性补全可视化编辑器


此项目为ColorOS特性补全可视化编辑器

此项目需要Root权限，主要使用kotlin编写，UI方面使用Jetpack Compose，UI风格为最新的Material 3 Expressive风格
1. 代码架构
项目采用UI和业务逻辑分离的设计思路，遵循以下原则：
UI层：负责界面渲染，不包含业务逻辑
业务逻辑层：处理应用功能实现，不依赖于具体UI实现
数据层：负责数据获取、持久化等操作
2. UI组件规范
UI组件目录结构
ui/components/：存放可复用的UI组件
ui/screens/：存放完整页面
ui/theme/：存放主题相关定义
没有的需要先询问我，不要自己决定

Compose编写规则
使用@Composable注解标记所有可组合函数
预览函数使用@Preview注解，便于开发时查看
组件应支持通过参数自定义，而非硬编码
3. 业务逻辑规范
业务逻辑代码放在domain包下
按功能模块组织代码结构
使用接口定义功能，实现与调用分离
异步操作使用协程处理
4. 数据处理规范
数据模型放在data/model下
遵循单一数据源原则
5. 命名规范
变量命名：驼峰命名法，如autoSetRules
类名：首字母大写的驼峰命名法，如AutoSetRules
常量：全大写，下划线分隔，如MAX_RETRY_COUNT
函数命名：动词开头，描述其行为，如getConfig()、saveSettings()
6. 其他开发约定
新增功能时，确保其他功能、引用不受影响
代码应有适当中文注释，尤其是复杂逻辑
修改公共组件前，应评估对使用该组件的地方的影响
使用Dialog时，使用原生MaterialDialog
界面风格遵循Material 3 Expressive设计风格

在编写代码时，请确保代码的结构清晰，易于维护。
如果你有不确定的地方，你需要先询问我而不是自己进行决定

在开始一个新任务前，积极使用todo_tool并由我审核、修改并最终明确确认后才开始代码编辑此项目为ColorOS特性补全可视化编辑器

此项目需要Root权限，主要使用kotlin编写，UI方面使用Jetpack Compose，UI风格为最新的Material 3 Expressive风格
1. 代码架构
项目采用UI和业务逻辑分离的设计思路，遵循以下原则：
UI层：负责界面渲染，不包含业务逻辑
业务逻辑层：处理应用功能实现，不依赖于具体UI实现
数据层：负责数据获取、持久化等操作
2. UI组件规范
UI组件目录结构
ui/components/：存放可复用的UI组件
ui/screens/：存放完整页面
ui/theme/：存放主题相关定义
没有的需要先询问我，不要自己决定

Compose编写规则
使用@Composable注解标记所有可组合函数
预览函数使用@Preview注解，便于开发时查看
组件应支持通过参数自定义，而非硬编码
3. 业务逻辑规范
业务逻辑代码放在domain包下
按功能模块组织代码结构
使用接口定义功能，实现与调用分离
异步操作使用协程处理
4. 数据处理规范
数据模型放在data/model下
遵循单一数据源原则
5. 命名规范
变量命名：驼峰命名法，如autoSetRules
类名：首字母大写的驼峰命名法，如AutoSetRules
常量：全大写，下划线分隔，如MAX_RETRY_COUNT
函数命名：动词开头，描述其行为，如getConfig()、saveSettings()
6. 其他开发约定
新增功能时，确保其他功能、引用不受影响
代码应有适当中文注释，尤其是复杂逻辑
修改公共组件前，应评估对使用该组件的地方的影响
使用Dialog时，使用原生MaterialDialog
界面风格遵循Material 3 Expressive设计风格

在编写代码时，请确保代码的结构清晰，易于维护。
如果你有不确定的地方，你需要先询问我而不是自己进行决定

在开始一个新任务前，尽可能使用todo_tool并由我审核、修改并最终明确确认后才开始代码编辑，除非该任务极其简单
积极使用web search

本项目中所有需要MainActivity的context的地方，一律使用app即可。如Toast.makeText(app,"text",xxxxx)

---
> Source: [ItosEO/ColorFeatureEnhance](https://github.com/ItosEO/ColorFeatureEnhance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
