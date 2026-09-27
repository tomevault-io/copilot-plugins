## glint

> 所有**面向用户的 UI 文案**都必须本地化,至少 `en`(源语言)+ `zh-Hans`。新增或修改可见文案时,**同步**在字符串目录补 `zh-Hans` 翻译;不允许留下硬编码、只有英文的可见文本。

# Glint

## 多语言 / 本地化(新功能必做)

所有**面向用户的 UI 文案**都必须本地化,至少 `en`(源语言)+ `zh-Hans`。新增或修改可见文案时,**同步**在字符串目录补 `zh-Hans` 翻译;不允许留下硬编码、只有英文的可见文本。

### 机制
- 字符串目录:`Glint/Resources/Localizable.xcstrings`(`sourceLanguage = en`,`developmentRegion = en`,已含 `zh-Hans`)。英文直接用源串(key 本身)作英文值,只需补 `zh-Hans` 条目(`extractionState: "manual"`、`state: "translated"`)。
- SwiftUI 里 `Text("字面量")` 自动是 `LocalizedStringKey`,会查表本地化。
- 非 View 上下文(`NSAlert`、字符串拼接、回调、`menu` title 等)用 `String(localized: "…")`。
- 运行时跟随系统语言或应用内语言覆盖(`AppleLanguages` + `.environment(\.locale,)`,见 `GlintApp`/`WorkspaceStore.preferredLocale`),代码里无需额外处理。

### 关键陷阱(最容易漏)
- `Text(someStringVariable)`(变量类型是 `String`)走的是 **verbatim 重载,不本地化**。把字面量经 `String` 形参传进 helper(例如 `func cell(_ k: String) { Text(k) }`)会**静默丢掉本地化** —— 编译不报错、英文环境下也看不出,只有切到中文才发现没翻译。
- 因此:helper / 自定义视图里**用作 UI 标签的形参类型必须是 `LocalizedStringKey`**(不是 `String`),这样字面量调用点才会进目录。
- **数据不要翻译**:分支名、路径、文件名、用户输入、commit 标题、URL 等是数据,保持 `String`(verbatim)。只翻译 UI chrome(标题、按钮、菜单、占位、提示语)。
- 单复数:英文单复数拆成各自的 key(如 `"1 file"` / `"%lld files"`),不要在代码里手拼 `"s"`,方便其它语言独立翻译。

### 加新可见文案的流程
1. 代码里用 `Text("…")` / `String(localized: "…")`,helper 形参用 `LocalizedStringKey`。
2. 在 `Glint/Resources/Localizable.xcstrings` 给每个新 key 补 `zh-Hans` 翻译(保持 JSON 缩进 2 空格、`ensure_ascii=False`)。
3. build 后切到中文环境目检,语言中性内容(符号、数字、纯数据)确认未被误翻。

## 发版「更新内容」(推版本时才写,不要预先占位)

每次推一个版本(beta 或正式)**都要**更新 `Glint/App/ReleaseNotes.swift` 的 `ReleaseNotes.all`,补这一版面向用户的「更新内容」(What's New 卡片用,升级后弹一次 + Settings ▸ About 可重看)。这份内容是**手写策划**的,刻意不依赖 commit 信息 / appcast。漏更新 = 用户升级后看不到这版的变化。

### 何时写(强约束)
- **只在准备 tag 这个版本的那次 commit 里**往 `all` 顶部加它的条目 —— 跟 tag/push 同一个 commit、或紧挨着 tag 的前一个 commit。
- **不允许预先开占位条目**(写下一版的版本号但等以后再填内容)。一旦写下的版本号跟实际发出去的不一致,占位里的所有行就会被错挂到一个错的版本上:
  - 真实案例:#43 那次本来计划 `0.1.24-beta.1` 占位,中间走了热修,#43 修复直接发 bare `0.1.24`;同期 #45 又往那条占位里塞了一行,结果 What's New 卡片把 #45 误归到 0.1.24(实际 #45 没在 0.1.24 里)。
- 所以:**写条目 ⟺ 这个 commit 接下来就要打 tag 推 release**。否则别动 `all`。

### 写的规则
- **每发一个 beta,在 `all` 顶部加一条**(数组「新→旧」),`version` 填**完整 beta 版本号**(连字符,如 `0.1.25-beta.1`),写**这个 beta 的增量**。
- **正式版不单独写条目** —— 升到 `0.1.25` 时,所有 `0.1.25-beta.*` 会**自动聚合**成一条「0.1.25」展示(`aggregatedStableNote`)。只有当某版**没走过 beta**时(直接发正式 / 热修),才在 tag 前给它写一条 bare base 条目(如 `0.1.26`)。
- **中英一一对应**:`en` / `zh` 两个数组逐条对应(这是数据、不进 xcstrings,直接双语写)。
- 内容是**面向用户的亮点**,不是 commit 罗列;只写用户能感知的功能/变化。
- **不要回填已发布的旧版本**;只加正在推的这版。文件可只保留近期版本,过老的条目可删(没人跨几十版更新,删了不影响功能)。
- 如果发的是从老 tag 切出去的 hotfix 分支(像 #43 那次从 v0.1.23 切 release/0.1.24),条目要写在 release 分支上,后续再 cherry-pick 或等价同步回 main。

### 行为(改逻辑前先知道)
- beta 用户:看当前 beta;跨 beta(beta.1→beta.3 跳过 beta.2)会 catch-up 同周期错过的 beta,分条列出。
- 正式用户:只看正式(聚合)条目,永远看不到 per-beta 条目;跨正式版 catch-up 每版聚合一条。
- 详见 `ReleaseNotes.notesToShow` 的文档注释。

---
> Source: [chenbstack/glint](https://github.com/chenbstack/glint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
