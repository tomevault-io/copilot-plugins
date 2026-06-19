## xiaohongshu-crush-analyzer

> >


# 小红书用户深度分析 & 追求策略顾问

## 概述

这个 Skill 有三个能力层次：
1. **画像分析**：深度浏览小红书公开主页，构建多维度社会画像
2. **理想型画像**：基于心理学理论框架，从发布内容和收藏行为中推断 TA 会被什么样的人吸引
3. **追求策略**：基于画像生成个性化的接近方案、话术建议和分阶段路线图

整个流程：**深度数据采集 → 多维画像分析 → 理想型推断 → 追求策略生成 → 可视化报告输出**

## 前置条件

- 需要 Claude in Chrome 浏览器工具
- 用户需提供一个小红书用户主页 URL
- **强烈建议用户在浏览器中已登录小红书**——未登录状态下能看到的内容非常有限（约10条笔记、无法点进详情页、无法看收藏），登录后可以获得 5-10 倍的数据量

第一步永远是提醒用户：「为了获得最好的分析效果，请确保你当前的浏览器已经登录了小红书。」

---

## Phase 1：深度数据采集

数据采集是整个分析的基础。采集分为三大部分：笔记列表扫描、笔记抽样深读、收藏夹全量扫描。

### 1.1 主页基础信息

导航到目标用户主页，等待 3 秒加载完成后截图。采集：

```
├── 昵称、小红书号
├── 个人签名 / 简介（非常重要——往往是自我定位的浓缩）
├── IP 属地
├── 星座/年龄等标签（如有展示）
├── 粉丝数 / 关注数 / 获赞与收藏数
├── 头像风格描述
└── 主页背景图（如有）
```

**确认笔记总量：** 用 `read_page` 或 JavaScript 提取「笔记」tab 旁的数字。记录 `TOTAL_NOTES`。

### 1.2 笔记列表全量扫描

**目标：扫描所有笔记标题和基础信息。** 这是后续分析的索引。

小红书主页使用虚拟滚动（virtual scrolling）——DOM 中只保留当前可见的笔记元素，滚出视窗的会被移除。所以不能滚完再提取，必须**边滚边收集**。

**自动滚动收集器模式（推荐）：**

注入以下 JavaScript 到页面，它会在滚动过程中持续收集数据并自动去重：

```javascript
// 初始化收集器（Map 自动去重）
window._noteCollector = { notes: new Map(), scanCount: 0 };

function collectNotes() {
  const sections = document.querySelectorAll('section.note-item, section[class*="note"]');
  sections.forEach(s => {
    const titleEl = s.querySelector('.title span, a .title span');
    const authorEl = s.querySelector('.author-wrapper .name, .footer .name');
    const likeEl = s.querySelector('.like-wrapper .count, .engagement .count');
    if (titleEl) {
      const title = titleEl.textContent.trim();
      const likes = likeEl ? likeEl.textContent.trim() : '0';
      // 用标题前30字符作为去重 key
      const key = title.substring(0, 30);
      if (!window._noteCollector.notes.has(key)) {
        window._noteCollector.notes.set(key, { title, likes });
      }
    }
  });
  window._noteCollector.scanCount = window._noteCollector.notes.size;
}

// 自动滚动 + 收集
async function autoScrollCollect() {
  window._scrollDone = false;
  let prevCount = 0;
  let sameStreak = 0;
  for (let i = 0; i < 500; i++) {
    collectNotes();
    window.scrollBy(0, 500);
    await new Promise(r => setTimeout(r, 400));
    const current = window._noteCollector.notes.size;
    if (current === prevCount) {
      sameStreak++;
      if (sameStreak > 20) break; // 连续20次无新内容 = 到底了
    } else {
      sameStreak = 0;
    }
    prevCount = current;
  }
  window._scrollDone = true;
}
autoScrollCollect();
```

注入后等待 30-60 秒（取决于笔记总量），然后检查 `window._scrollDone` 和 `window._noteCollector.scanCount`。

**Chrome 浏览器扩展断连处理：** 等待期间 Claude in Chrome 扩展可能断开连接。如果 `tabs_context_mcp` 报错或无法访问 tab，重新调用 `tabs_context_mcp` 获取新的 tab 连接，然后用 JavaScript 检查后台脚本是否已完成（检查 `window._scrollDone`）。

**数据提取：** 收集完成后，分批提取（每批100条避免 JSON 截断）：

```javascript
const arr = Array.from(window._noteCollector.notes.values());
JSON.stringify(arr.slice(0, 100));  // 第1批
// JSON.stringify(arr.slice(100, 200));  // 第2批，以此类推
```

**到底确认：** 如果连续 20 次滚动后笔记数不再增加，判定为已到底。记录实际扫到的总数和覆盖率。

### 1.3 笔记抽样深读（3-5 条精读）

笔记深读的目的不是逐条阅读所有内容（那在大量笔记时不现实），而是**精选有代表性的笔记获取深度信息**。配合已有的全量标题列表，少量精读足以支撑高质量分析。

**选择策略——精准抽样：**
1. **最新 1-2 条**：代表当前状态
2. **互动最高的 1 条**：代表 TA 最受认可的内容
3. **低互动但标题很个人化的 1 条**：往往最真实
4. **最早期的 1 条**（如能找到）：了解变化轨迹

对每条深读笔记，采集：
```
├── 完整正文（用 get_page_text）
├── 所有 #标签
├── 发布日期/时间
├── 点赞、收藏、评论数
├── 图片内容描述（截图观察）
├── 评论区分析（前 20 条评论 + 博主回复）
└── 读完后用浏览器返回主页
```

### 1.4 收藏夹全量扫描（极其重要）

**收藏内容是整个分析中含金量最高的数据。** 一个人发布的内容是「前台表演」（front-stage performance），而收藏的内容是 TA 的「内隐偏好」（implicit preferences）——TA 真正在意的、想反复看的、不方便发但很向往的东西。

**对理想型推断和追求策略来说，收藏夹数据的价值甚至超过笔记本身。**

操作流程：
1. 在主页找到「收藏」tab 并点击
2. 如果收藏夹加锁（私密），标注「收藏夹不可见」并跳过
3. 如果公开，使用与笔记列表相同的**自动滚动收集器模式**全量扫描：

```javascript
// 重置收集器（避免残留笔记 tab 的旧数据）
window._favCollector = { notes: new Map(), scanCount: 0 };

function collectFavs() {
  const sections = document.querySelectorAll('section');
  sections.forEach(s => {
    const titleEl = s.querySelector('.title span');
    const authorEl = s.querySelector('.author-wrapper .name, .footer .name');
    const likeEl = s.querySelector('.like-wrapper .count');
    if (titleEl && authorEl) {
      const title = titleEl.textContent.trim();
      const author = authorEl.textContent.trim();
      const likes = likeEl ? likeEl.textContent.trim() : '0';
      const key = title.substring(0, 30) + '|' + author;
      if (!window._favCollector.notes.has(key)) {
        window._favCollector.notes.set(key, { t: title, a: author, l: likes });
      }
    }
  });
  window._favCollector.scanCount = window._favCollector.notes.size;
}

async function autoScrollFavs() {
  window._favScrollDone = false;
  // 先滚到顶部确保从头开始
  window.scrollTo(0, 0);
  await new Promise(r => setTimeout(r, 1000));
  let prevCount = 0;
  let sameStreak = 0;
  for (let i = 0; i < 500; i++) {
    collectFavs();
    window.scrollBy(0, 500);
    await new Promise(r => setTimeout(r, 350));
    const current = window._favCollector.notes.size;
    if (current === prevCount) {
      sameStreak++;
      if (sameStreak > 25) break;
    } else {
      sameStreak = 0;
    }
    prevCount = current;
  }
  window._favScrollDone = true;
}
autoScrollFavs();
```

4. **分批提取**后进行**关键词分类统计**：

```javascript
// 关键词分类模板（根据实际内容调整类别和关键词）
const categories = {
  '美妆/护肤': ['化妆', '护肤', '防晒', '面膜', '口红', '粉底'],
  '穿搭/时尚': ['穿搭', 'ootd', '搭配', '时尚', '衣服'],
  '美食/探店': ['美食', '餐厅', '探店', '甜品', '咖啡'],
  '旅行': ['旅行', '旅游', '攻略', '打卡', '酒店'],
  '健身/运动': ['健身', '运动', '瑜伽', '跑步', '减脂'],
  '情感/恋爱': ['恋爱', '脱单', '约会', '暧昧', '表白', '男朋友', '女朋友'],
  '职场/学习': ['工作', '面试', '考研', '留学', '雅思', '实习'],
  '艺术/设计': ['画', '插画', '设计', '摄影', '艺术', 'art'],
  '家居/生活': ['家居', '租房', '装修', '收纳', '好物'],
  // 根据目标用户内容动态添加更多类别...
};

function categorize(items) {
  const result = {};
  Object.keys(categories).forEach(cat => result[cat] = []);
  result['其他'] = [];
  items.forEach(item => {
    let matched = false;
    for (const [cat, keywords] of Object.entries(categories)) {
      if (keywords.some(kw => item.t.toLowerCase().includes(kw.toLowerCase()))) {
        result[cat].push(item.t);
        matched = true;
        break;
      }
    }
    if (!matched) result['其他'].push(item.t);
  });
  return Object.fromEntries(
    Object.entries(result)
      .filter(([k, v]) => v.length > 0)
      .sort((a, b) => b[1].length - a[1].length)
  );
}

const items = Array.from(window._favCollector.notes.values());
JSON.stringify(categorize(items));
```

关键词列表应根据目标用户的实际内容动态调整——先看一遍收藏标题的原始数据，识别出高频主题后再设计分类关键词。

5. **重点分析维度：**
```
├── 主题分布统计（各类别数量和占比）
├── 发布内容 vs 收藏内容的差异
│   （差异越大 → TA 的前台/后台差距越大）
├── 收藏中的情感类内容（恋爱技巧？情侣日常？）
├── 反复收藏的来源（同一博主被收藏多次 = 高度认同）
├── 隐藏兴趣（没发过但大量收藏的主题 = 秘密兴趣 = 破冰利器）
└── 消费/审美偏好线索
```

如果收藏是私密的，在报告中标注「收藏夹不可见」并跳过。

### 1.5 关注列表分析（如果可见）

快速扫描 TA 关注了哪些类型的账号，分类统计。

### 数据采集小结

采集完成后，输出采集报告：

```
📊 数据采集报告：
├── 笔记总数：X 条
├── 列表扫描：已扫描 A 条（覆盖率 XX%）
├── 精读深度：已精读 B 条
├── 收藏分析：已扫描 C / D 条（覆盖率 XX%）| 类别分布已统计
├── 关注列表：可见/不可见
└── 评论区互动：已分析 E 条笔记的评论区
```

---

## Phase 2：多维画像分析

详细的分析方法和信号体系，参见 `references/analysis-framework.md`。

### 核心分析维度

**维度一：基础人口学** —— 年龄段、性别、地区、城市线级、职业、教育水平
**维度二：消费与生活方式** —— 消费能力、品牌偏好、兴趣标签、生活方式分类
**维度三：心理与人格** —— Big Five、情感基调、价值观、沟通风格
**维度四：社会关系与影响力** —— 用户类型、影响力、社群归属
**维度五：时间行为模式** —— 发布规律、主题演变、活跃周期

### 追求场景额外维度

**维度六：社交互动偏好** —— TA 回复什么类型的评论、用什么口吻、偏好什么互动方式
**维度七：情感状态推断** —— 是否有伴侣迹象（置信度极低，必须声明）
**维度八：「吸引力密码」** —— TA 最引以为豪的身份、在意的品质、社交需求类型

### 维度九：理想型画像推断（新增核心功能）

这是基于心理学理论框架的综合推断——**TA 会被什么样的人吸引？**

理论基础（在报告中引用，增强可信度）：
- **Byrne 相似-吸引范式 (1971)**：人倾向于被与自己态度、价值观相似的人吸引
- **Watson 等人 Big Five 匹配研究 (2004)**：性格中「尽责性」和「宜人性」的匹配度对关系满意度影响最大
- **Aron 自我扩展模型 (1986)**：人被能拓展自己认知/体验边界的人吸引
- **Bowlby/Ainsworth 依恋理论**：安全型依恋者提供稳定、可预测的情感环境更具吸引力
- **Dweck 成长型思维 (2006)**：在亲密关系中，展现成长意愿比展现完美更重要

推断方法：结合「发布内容」（前台形象）+ 「收藏行为」（内隐偏好）+ 心理学理论，从 6 个维度推断理想型：

```
├── 审美共鸣力：TA 的审美体系是什么？什么视觉风格能引起共鸣？
├── 文化视野：TA 重视什么文化/知识领域？什么背景的人能产生话题？
├── 自我扩展力：TA 向往但尚未拥有什么？谁能带来新体验？
├── 安全型依恋：TA 需要什么样的情感稳定感？
├── 成长型人格：TA 欣赏什么样的自我提升姿态？
└── 情感联结力：TA 的情感表达习惯是什么？什么沟通方式最舒服？
```

每个维度都要有**证据链**：「从哪条笔记/收藏推断出来的 → 理论依据 → 结论 → 置信度」。

报告中必须声明：「此推断基于公开社交媒体内容的间接分析，结合心理学研究结论，但不等同于专业心理评估或现实中的择偶偏好。仅供参考。」

---

## Phase 3：追求策略生成

基于画像分析和理想型推断，生成个性化的追求策略。策略分阶段、有具体行动步骤、有示例话术、并对不同回应有应对方案。

### 策略框架

```
策略总纲：
├── 核心策略原则（一句话概括）
├── 理想人设构建（Do's & Don'ts）
│
├── Phase 1：刷存在感（1-2周）
│   ├── 具体行动：在哪些笔记下评论、评论什么
│   ├── 示例话术：3-5 条量身定制的评论文案
│   └── 分支应对：积极/中性/无回应
│
├── Phase 2：建立连接（2-4周）
│   ├── 具体行动：如何开启私聊
│   ├── 示例话术：首条私信 2-3 个版本
│   └── 分支应对：详细回复/简短/已读不回
│
├── Phase 3：深入了解（3-6周）
│   ├── 转微信时机和话术
│   ├── 聊天节奏指南
│   └── 分支应对
│
├── Phase 4：线下见面（4-8周）
│   ├── 3 个量身定制的约会场景
│   └── 分支应对
│
└── Phase 5：关系升温（6-12周）
    ├── 增加专属感的行动
    ├── 表白时机判断
    └── 分支应对
```

### 话术量身定制原则

1. **基于 TA 实际发布的内容**——引用真实笔记话题
2. **匹配 TA 的沟通风格**——TA 随性你也随性，TA 文艺你也得有审美
3. **选择低竞争切入点**——优先选赞数少的笔记（被看到概率高）
4. **展现「懂行」而非「追星」**——体现你在这个领域也有见解
5. **避免泛泛夸赞**——"好帅/好美"毫无辨识度，要具体到内容

---

## Phase 4：报告生成

输出一份**单文件交互式 HTML 报告**，全部中文。

### 七个 Tab

| Tab | 内容 |
|-----|------|
| 📊 总览 | 一句话画像 + 关键标签 + 基础数据 + 五维雷达图 |
| 🔍 深度画像 | 八个维度的详细分析（每个带置信度和证据） |
| 💜 理想型画像 | 心理学理论框架 + 收藏行为分析图表 + 6维推断 + 匹配度自测 |
| 📋 笔记一览 | 所有已扫描笔记列表（可搜索/筛选） |
| 🎯 追求策略 | 核心原则 + Do's & Don'ts + 五阶段路线图 |
| 💬 话术锦囊 | 评论话术 + 私信话术 + 场景话术 |
| ℹ️ 方法论 | 数据采集报告 + 分析方法说明 + 局限性声明 |

### 理想型画像 Tab 的特殊要求

这个 Tab 需要包含：

1. **方法论卡片**：列出 5 个心理学理论框架及其应用方式
2. **收藏行为分析**：饼图/环形图展示收藏内容的主题分布
3. **六维度分析**：每个维度有「证据」「理论依据」「推断」三个层次
4. **理想型雷达图**：六维度得分可视化
5. **匹配度自测**：交互式评分器，用户可以给自己在每个维度打分，计算与理想型的匹配度

### 技术规格

- **Chart.js** via CDN（雷达图、环形图、柱状图等）
- 主色调：#FF2442（小红书品牌色）
- 响应式布局
- 置信度颜色：高(#22c55e) / 中(#eab308) / 低(#9ca3af)
- 分支应对颜色：积极(绿) / 中性(黄) / 消极(红)
- 保存到 outputs/ 文件夹
- 文件名：`{昵称}_画像分析_{日期}.html`（纯画像）或 `{昵称}_画像与追求策略_{日期}.html`（含策略）

---

## 重要注意事项

### 浏览器操作技巧

1. **登录弹窗处理**：小红书经常弹登录框。用 `find` 查找关闭按钮，或按 Escape。如果被重定向到登录页，用 `navigate` 的 `back` 返回。

2. **滚动节奏**：使用自动滚动脚本时，每步 500px + 350-400ms 间隔是安全的。手动滚动时每次 3-5 tick，间隔 2 秒。

3. **虚拟滚动的关键影响**：小红书使用虚拟滚动，DOM 只保留可见元素。所以不能「先滚完再提取」——必须用 `Map` 在滚动过程中持续收集并去重。这就是为什么上面推荐注入自动滚动收集器脚本。

4. **Chrome 扩展断连**：等待自动滚动脚本执行时（30-60秒），Claude in Chrome 扩展可能因超时断开连接。处理方式：
   - 重新调用 `tabs_context_mcp` 获取连接
   - 用 JavaScript 检查后台脚本的完成标记（如 `window._scrollDone`）
   - 如果脚本已完成，直接提取数据；如果未完成，继续等待

5. **数据提取防截断**：大量数据（>100条）通过 JavaScript 返回时可能被截断。分批提取：`arr.slice(0, 100)`、`arr.slice(100, 200)`……

6. **收藏 tab 切换注意**：从「笔记」tab 切到「收藏」tab 后，之前的 DOM 元素可能仍残留在页面中。收藏收集器必须使用全新的 Map（不要复用笔记收集器），并在开始前 `scrollTo(0, 0)` 回到顶部。

### 分析的诚实性

- 宁可说「数据不足以判断」也不要编造推断
- 每个推断问自己：「如果我把这个推断给目标本人看，TA会觉得荒谬吗？」
- 心理和情感维度不确定性大，必须标注置信度
- 理想型推断基于间接证据，必须声明其局限性

### 伦理边界

- 策略建议的核心是「做更好的自己去吸引对方」，而不是「操纵对方」
- 报告末尾始终包含：真诚提醒用户——最好的策略是做真实的自己
- 所有分析仅基于公开可见的信息

---
> Source: [lhfer/xiaohongshu-crush-analyzer](https://github.com/lhfer/xiaohongshu-crush-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
