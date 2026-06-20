## latex-guard

> LaTeX 编译防错与排版质量守护技能。专为东北大学学位论文模板(NEU-Thesis)设计，但适用于任何中文学位论文LaTeX模板。提供7类常见编译错误的快速诊断修复、表格排版规范、标准编译流程、12类PDF排版质量检测清单、编译后强制验证流程。关键词触发："编译论文"、"LaTeX报错"、"表格溢出"、"xelatex"、"tex文件"、"bibtex"、"Extra alignment"、"Overfull"、"Float too large"、任何涉及 .tex 文件编写、LaTeX 编译、表格制作、xelatex 命令执行、bibtex 引用处理、图表跨页分离、浮动体位置控制、minipage 绑定、longtable 长表格、FloatBarrier 浮动屏障 的场景均触发本技能。 新增批量浮动体修复模式：当需要系统性检查和修复整篇论文所有图表的跨页分离、位置漂移问题时触发此模式（"系统性检查浮动体"、"修复所有图表位置"、"批量优化排版"、"论文排版整体修复"）。


# LaTeX Guard — LaTeX 编译防错与排版质量守护技能

## 技能触发

任何涉及 .tex 文件编写、LaTeX 编译、表格制作、xelatex 命令执行、bibtex 引用处理、
图表跨页分离、浮动体位置控制、minipage 绑定、longtable 长表格、
FloatBarrier 浮动屏障 的场景均触发本技能。默认面向 NEU-Thesis 模板，但诊断方法同样适用于其他中文学位论文模板。

---

## 一、七大类常见编译错误

### 1.1 "Extra alignment tab has been changed to \cr."

**含义**：表格列数定义与实际数据列数不匹配。

**原因**：`\begin{tabular}{...}` 中定义的列数 < 数据行中 `&` 分隔符数 + 1。

**诊断**：数清列格式中 `l`/`c`/`r`/`p{}` 的个数，再数数据行中 `&` 的个数。例如 `{ccccccc}` 有7列，每行应恰好有6个 `&`。

**修复**：
- 修改 `\begin{tabular}{...}` 中的列数定义使其匹配数据行
- 或删除数据行中多余的 `&`

### 1.2 "Overfull \hbox" / "Float too large for page"

**含义**：表格宽度超出页面宽度。

**原因**：多列表格（≥6列）未做缩放处理。

**修复优先级（按表格宽度选择）**：

| 表格特征 | 方案 | 命令/环境 |
|---------|------|---------|
| 列数≤5 | 居中 + 五号字 | `\centering\zihao{5}` |
| 列数6-8 | 缩放适配 | `\tablefit{\begin{tabular}{...}...\end{tabular}}` |
| 列数≥9 | 横向页面 | `\begin{sidewaystable}...\end{sidewaystable}` |
| 超长表格 | 跨页表格 | `\begin{longtable}{...}...\end{longtable}` |
| 附录超大表 | 缩放 + 横向 | `sidewaystable` 内嵌 `\tablefit` |

### 1.3 图表、文字、表格跨页分离问题（系统性解决方案）

#### 1.3.1 问题本质

LaTeX 浮动体（figure/table）默认自由漂移，自动寻找"最佳"位置放置，但这常常导致图表和描述它的文字被分隔在不同页面。三种解决思路：**限制浮动范围** / **强制内容绑定** / **调整全局规则**。以下按可靠性从高到低展开。

#### 1.3.2 浮动体位置参数详解

位置参数控制浮动体的放置偏好。`\begin{figure}[参数]` 或 `\begin{table}[参数]` 中的可选字母：

| 参数 | 含义 | 优先级 | 说明 |
|------|------|--------|------|
| `h` | here | ★ | 尽量放在当前位置（此处） |
| `t` | top | ★★ | 尽量放在页面顶部 |
| `b` | bottom | ★★ | 尽量放在页面底部 |
| `p` | page | ★ | 放在单独的浮动页 |
| `!` | 强制执行 | — | 忽略 LaTeX 对页面上浮动体数量的限制 |
| `H` | 强制在此 | ★★★ | 必须放在此处（需 `\usepackage{float}`） |

**推荐组合**：
- **通用推荐**：`[hbt!]` —— 首选此处，不行放顶部，再不行放底部，忽略数量限制
- **精确控制**：`[H]` —— 强制锁定在代码位置（但需谨慎使用，可能造成大片空白）
- **顶部偏好**：`[t!]` —— 强制放在页面顶部

> ⚠️ **`[H]` 是"核武器"**：它完全绕过 LaTeX 浮动机制，可能导致页面底部大量空白。仅在定稿阶段、确认排版无问题后使用。日常写作优先用 `[hbt!]`。

#### 1.3.3 强制内容绑定（最可靠）

当必须确保"某段文字 + 某张图/表"绝对不跨页时，用盒子环境将它们打包在一起。

**① minipage 环境（首选，最可靠）**

LaTeX 保证绝不会在 `minipage` 盒子内部进行分页，是所有方案中最可靠的：

```latex
\noindent\begin{minipage}{\textwidth}
\vspace{4pt}                                     % 上部留一点空间
关于城市轨道交通运营效率的评价指标体系见表 \ref{tab:efficiency}。
\begin{table}[H]\centering\zihao{5}
  \caption{运营效率评价指标体系}\label{tab:efficiency}
  \tablefit{
    \begin{tabular}{clc}
      \toprule
      序号 & 指标名称 & 权重 \\
      \midrule
      1 & 准点率 & 0.35 \\
      2 & 客流强度 & 0.28 \\
      \bottomrule
    \end{tabular}
  }
\end{table}
\vspace{4pt}                                     % 下部留一点空间
\end{minipage}
```

**② \vbox（更轻量）**

纯 TeX 原语，比 minipage 更轻量，但功能较少（不支持 footnote 等）：

```latex
\vbox{
关于城市轨道交通运营效率的评价指标体系如下表所示：
\begin{table}[H]\centering\zihao{5}
  \caption{运营效率评价指标体系}\label{tab:efficiency}
  ...
\end{table}
}
```

**③ \parbox（适合短内容）**

用于包裹简短的内容组合，适合"一句话 + 小图表"的场景：

```latex
\parbox{\textwidth}{
该区域土地利用变化如图 \ref{fig:landuse} 所示：
\begin{figure}[H]\centering
  \includegraphics[width=0.8\textwidth]{Img/landuse.png}
  \caption{土地利用变化图}\label{fig:landuse}
\end{figure}
}
```

**④ samepage 环境（不推荐）**

LaTeX 标准 `samepage` 环境试图阻止分页，但效果有限：

```latex
\begin{samepage}
关于...见表 \ref{tab:xxx}。
\begin{table}[H]\centering ...
\end{table}
\end{samepage}
```

> ⚠️ **警告**：CMU 官方文档明确指出 `samepage` 在很多情况下是无效的。不推荐依赖它。

#### 1.3.4 浮动屏障（\FloatBarrier）

当不需要精确绑定文字和图表，但需要确保"在某个位置之前所有浮动体都必须落位"时使用。

**基础用法**：

```latex
\usepackage{placeins}

% ... 章节内容 ...

\FloatBarrier   % 此处之前所有浮动体必须落位，不能越过此屏障
```

**带 [section] 选项（推荐）**：

```latex
\usepackage[section]{placeins}
```

加上 `[section]` 参数后，LaTeX 自动在每个 `\section` 命令前插入 `\FloatBarrier`，确保每个章节内的图表不会跑到下一个章节中去。**这是学术论文最优雅的浮动控制方案。**

> 📌 **本项目已应用**：在 `Thesis.tex` 第 27 行已添加 `\usepackage[section]{placeins}`，自动控制每章浮动体范围。

#### 1.3.5 全局参数调整

通过调整 LaTeX 内部参数，让浮动体更容易在目标位置放置，减少漂移。将以下代码放入导言区（`\begin{document}` 之前）：

```latex
% ===== 浮动体全局参数调整 =====
\renewcommand{\topfraction}{0.9}        % 页面顶部最多可被浮动体占据 90%（默认0.7）
\renewcommand{\bottomfraction}{0.9}     % 页面底部最多可被浮动体占据 90%（默认0.3）
\renewcommand{\textfraction}{0.1}       % 页面至少保留 10% 给正文（默认0.2）
\renewcommand{\floatpagefraction}{0.7}  % 浮动页上浮动体至少占 70%（默认0.5）
\setcounter{topnumber}{3}               % 页面顶部最多 3 个浮动体（默认2）
\setcounter{bottomnumber}{2}            % 页面底部最多 2 个浮动体（默认1）
\setcounter{totalnumber}{5}             % 每页最多 5 个浮动体（默认3）
\raggedbottom                           % 允许底部留白，避免拉伸填充
```

**核心参数说明**：
- `\topfraction=0.9` + `\bottomfraction=0.9`：让 LaTeX 更愿意在目标位置放置浮动体（而不是拒绝后推到后面页面）
- `\raggedbottom`：页面底部不对齐，避免为填满页面而将浮动体推远

#### 1.3.6 特殊场景处理

**① 表格标题与表格分离**

当表格标题显示在一页底部、表格内容在下一页时：

```latex
\usepackage{caption}
\captionsetup[table]{position=above}       % 标题始终在表格上方
\captionsetup[table]{skip=2pt}             % 标题与表格间距
```

对于 figure 同理：
```latex
\captionsetup[figure]{position=below}      % 图标题在下方
```

**② 长表格跨页**

超过一页的表格必须用 `longtable`，不能用 `table` 环境：

```latex
\usepackage{longtable,booktabs}

{\zihao{5}
\begin{longtable}{ccccc}
  \caption{长表格示例}\label{tab:long-example}\\
  \toprule
  列1 & 列2 & 列3 & 列4 & 列5 \\
  \midrule
  \endfirsthead                                % 首页表头
  \multicolumn{5}{c}{\tablename\ \thetable{} （续）}\\
  \midrule
  列1 & 列2 & 列3 & 列4 & 列5 \\
  \midrule
  \endhead                                     % 续页表头
  \bottomrule
  \multicolumn{5}{r}{\footnotesize 续下页}\\
  \endfoot                                     % 续页底部
  \bottomrule
  \endlastfoot                                 % 末页底部
  数据行1 & ... & ... & ... & ... \\
  数据行2 & ... & ... & ... & ... \\
  % ... 更多数据行 ...
\end{longtable}
}
```

关键元素：
- `\endfirsthead`：首页显示的表头
- `\endhead`：从第2页起每页顶部重复的表头
- `\endfoot`：非末页底部显示的内容（如"续下页"）
- `\endlastfoot`：最后一页底部显示的内容

**③ 双栏排版**

在双栏文档中使用跨双栏的图表：

```latex
\begin{figure*}[t!]                      % 跨双栏图片，强制放顶部
  \centering
  \includegraphics[width=\textwidth]{Img/wide-figure.png}
  \caption{横跨双栏的宽图}\label{fig:wide}
\end{figure*}

\begin{table*}[t!]                        % 跨双栏表格
  \centering\zihao{5}
  \caption{横跨双栏的宽表}\label{tab:wide}
  \begin{tabular}{...}
    ...
  \end{tabular}
\end{table*}
```

注意：`table*` 和 `figure*` 只能放置在页面顶部（`t`）或单独浮动页（`p`）。

**④ 延迟浮动体（afterpage）**

将浮动体精确延迟到下一页顶部：

```latex
\usepackage{afterpage}

\afterpage{
  \begin{figure}[t!]
    \centering
    \includegraphics[width=0.9\textwidth]{Img/delayed-figure.png}
    \caption{精确出现在下一页顶部的图}\label{fig:delayed}
  \end{figure}
  \FloatBarrier                           % 确保此图不会跨过下一个屏障
}
```

适用场景：当前页内容过多放不下图表，但又不想让图表飘到后面的页面。

#### 1.3.7 常见问题排查

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| `[h]` 参数无效，图表仍漂移 | `h` 只是建议，LaTeX 可以忽略 | 改为 `[h!]` 或 `[H]`；优先尝试增大 `\topfraction` 等全局参数 |
| 图表顺序错乱 | 多个浮动体互相竞争位置 | 对关键位置使用 `\FloatBarrier` 隔离；或使用 `[H]` 精确锁定顺序 |
| 图和标题分别在不同页 | `\caption` 在 `figure` 外部，或浮动体被拆分 | 确保 `\caption` 在 `\begin{figure}...\end{figure}` 内部 |
| 页面出现大片空白 | 浮动体被强制 `[H]` 锁定但当前位置放不下 | 减少 `[H]` 的使用；用 `afterpage` 延迟到下一页；调大 `\topfraction` 等参数 |

#### 1.3.8 最佳实践总结（优先级排序）

| 优先级 | 方案 | 适用场景 | 命令/配置 |
|-------|------|---------|---------|
| 1 | 位置参数 + 全局参数 | 日常写作，基础控制 | `[hbt!]` + `\topfraction=0.9` 等 |
| 2 | minipage 绑定 | 必须绑定场景 | `\begin{minipage}{\textwidth}...\end{minipage}` |
| 3 | placeins [section] | 章节级别自动控制 | `\usepackage[section]{placeins}` |
| 4 | longtable | 超过一页的表格 | `\begin{longtable}...\end{longtable}` |
| 5 | 定稿精调 | 最终排版确认 | 逐页检查，必要处用 `[H]` 或 `\FloatBarrier` |

> 📌 **黄金法则**：日常写作用 `[hbt!]` + 全局参数；章节用 `[section]{placeins}`；绑定用 `minipage`；长内容用 `longtable`。直到最后定稿时，再逐页精调分页。

---

### 1.3.9 AI 生成 LaTeX 时代码提示模板

**通用提示词模板（让 AI 自动生成带分页控制的 LaTeX 代码）：**

```text
请生成以下内容的 LaTeX 代码，要求：
1. 所有表格使用 [hbt!] 位置参数，三线表格式（\toprule \midrule \bottomrule）
2. 如果某段文字直接关联一个表格/图片，请将文字和表格/图片放在同一个 minipage 环境中防止跨页分离
3. 如果表格超过30行或明确会跨页，使用 longtable 环境并设置 \endhead 重复表头
4. 每个 \section 前添加 \FloatBarrier，防止浮动体跨节漂移
5. 表格宽度用 \tablefit{} 包裹，或用 p{宽度} 指定列宽，确保不溢出页面
6. 所有宽度使用相对宽度（如 \textwidth、0.45\textwidth），不要使用绝对厘米
7. \caption 放在 \begin{table}/\begin{figure} 内部，确保与表/图绑定
8. 表格和图片前添加引导文字（如"如表 \ref{tab:xxx} 所示"）

具体需求：[在此描述你要生成的内容]
```

**修正已有代码的提示词模板：**

```text
我的 LaTeX 文件中表格和对应文字经常跨页分离，请帮我修改代码：
1. 将有"如表所示"等引导文字和对应表格放在同一个 minipage 中
2. 将 \begin{table}[htbp] 改为 \begin{table}[hbt!]
3. 在导言区添加浮动体全局参数调整：\topfraction=0.9, \bottomfraction=0.9, \textfraction=0.1
4. 将表格放在引导文字之后而非之前
5. 确保所有 \caption 在其所属环境内部

文件内容：[粘贴你的 .tex 代码]
```

### 1.4 公式与变量解释被分页切开

**含义**：公式在一页底部，它的"其中 $x$ 为..."解释文字被截到下一页顶部。

**修复**：用 `minipage` 盒子把公式和解释文字捆在一起：

```latex
\noindent\begin{minipage}{\textwidth}
\begin{equation}
  w_j = \frac{(\prod a_{ij})^{1/n}}{\sum (\prod a_{kj})^{1/n}} \tag{1}
\end{equation}
其中 $a_{ij}$ 为判断矩阵元素，$n$ 为矩阵阶数，$w_j$ 为权重向量。
\end{minipage}
```

公式太多无法避免跨页时，用 `\nopagebreak[4]` 确保编号在同一页：
```latex
\end{equation}\nopagebreak[4]   % 强烈建议此处不分页
其中 $x'_{ij}$...
```

### 1.5 段落孤行（一行只有一个字 / 一两个字）

**含义**：段落最后一行只剩下1-2个字，单独占据一整行（widow/orphan lines）。

**修复**：在 Thesis.tex 导言区（`\begin{document}` 之前）添加：

```latex
\clubpenalty=10000      % 禁止段首孤行
\widowpenalty=10000     % 禁止段尾孤行
\raggedbottom           % 允许底部留白，避免拉伸产生孤行
```

**单个段落手动修复**：段尾加 `\looseness=-1` 让 LaTeX 尝试收紧一行。

### 1.6 章节间的多余空白页

**含义**：新章从奇数页（右页）开始，上一章在奇数页结束时自动插入空白页。

**原因**：模板设定了 `openright`（学位论文正式规范）。

| 方案 | 命令 | 适用 |
|------|------|------|
| 符合规范（保留） | 不修改 | 正式提交版 |
| 消除空白页 | `\let\cleardoublepage\clearpage` | 草稿/审阅版 |

消除空白页操作（仅工作草稿）：在 `\begin{document}` 前插入 `\let\cleardoublepage\clearpage`。

### 1.7 "Undefined citation" / 引用显示 "[?]"

**含义**：参考文献引用无法解析。

**原因**：BibTeX 未运行，或 bib key 与 ref.bib 中不匹配，或 `\cite{}` 拼写错误。

**修复**：
1. 检查 `Biblio/ref.bib` 中是否存在该 key
2. 确认 `\cite{}` 中的 key 拼写与 bib 文件完全一致（区分大小写）
3. 重新执行完整编译链：`xelatex → bibtex → xelatex → xelatex`

---

## 二、表格排版完整规范

### 2.1 表格标准模板

```latex
\begin{table}[!htbp]\centering\zihao{5}
  \caption{表格标题}\label{tab:唯一标识}
  \tablefit{
    \begin{tabular}{cccccccc}
      \toprule
      列1 & 列2 & 列3 & 列4 & 列5 & 列6 & 列7 & 列8 \\
      \midrule
      数据行 & ... & ... & ... & ... & ... & ... & ... \\
      \bottomrule
    \end{tabular}
  }
\end{table}
```

### 2.2 单元格过长的处理

当某一列内容过长时，使用 `p{宽度}` 指定列宽让其自动换行：

```latex
\begin{tabular}{lp{4cm}p{3cm}cc}
```
- `p{4cm}`：该列宽度4cm，超出自动换行
- 可用表达式：`p{\dimexpr0.15\textwidth-2\tabcolsep\relax}` 精确计算

### 2.3 表格注释和脚注

```latex
\begin{table}[!htbp]\centering\zihao{5}
  \caption{表格标题}\label{tab:xxx}
  \tablefit{\begin{tabular}{cccccc}
      \toprule 列1 & 列2 & 列3 \\ \midrule 数据 \\ \bottomrule
  \end{tabular}}
  \vspace{2pt}\tablefootnotesize
  注：表格脚注内容，说明数据来源、缩写等。
\end{table}
```

### 2.4 LaTeX公式在表格单元格中

表格内的短公式用 `$...$` 内联，长公式用 `\makecell{$\displaystyle ...$}`：

```latex
% 需要在导言区加载 \usepackage{makecell}
& \makecell{$\displaystyle I = \frac{n}{W}\frac{\sum w_{ij}...}{\sum ...}$} &
```

---

## 三、标准编译流程

### 3.1 完整编译链（4步）

```bash
cd <你的NEU-Thesis模板目录>
xelatex -interaction=nonstopmode Thesis.tex    # 第1步：生成 .aux 文件
bibtex Thesis                                   # 第2步：生成 .bbl 引用数据
xelatex -interaction=nonstopmode Thesis.tex    # 第3步：解析引用
xelatex -interaction=nonstopmode Thesis.tex    # 第4步：确认交叉引用
```

### 3.2 快速编译（仅改正文无新增引用时）

```bash
xelatex -interaction=nonstopmode Thesis.tex
```

### 3.3 编译结果判断标准

- **成功**：生成 `Thesis.pdf`，.log 中无 `^!` 开头的 Error 行
- **警告可忽略**：`Overfull`、`Underfull`、`Font Warning` 通常不影响结果
- **必须修复**：`^!` 开头的 Error，`Float too large`，引用显示 `[?]`

### 3.4 🔴 编译后强制验证（重要！）

每次编译完成后**必须**执行以下步骤，漏一步即为严重错误：

#### 步骤一：验证 PDF 输出位置

编译完成后，立即确认 PDF **确实生成在项目目录下**：

```bash
ls -la "Thesis.pdf"
# 或 PowerShell: Get-Item "Thesis.pdf" | Select FullName, Length
```

- ⚠️ **经典错误**：xelatex 工作目录是模板默认路径，但实际项目在其他目录。编译前务必 `cd` 到项目根目录，编译后验证 PDF 在该目录下。
- 如果 PDF 生成到了错误路径，立即拷贝到正确位置。

#### 步骤二：重命名为论文标题

PDF **不得**以 `Thesis.pdf` 交付，必须重命名：

```bash
mv Thesis.pdf "论文完整主标题.pdf"
# Windows: Rename-Item "Thesis.pdf" "论文完整主标题.pdf"
```

- 标题从 `Tex/Frontpages.tex` 的 `\thesistitle{}` 字段获取
- 使用中文主标题，不含副标题、不含英文翻译

#### 步骤三：推送绝对路径给用户

必须向用户输出 PDF 的完整绝对路径：

```
📄 PDF 已生成：<项目目录>/<论文标题>.pdf（XX 页，X.XX MB）
```

- 使用完整绝对路径，附带页数和文件大小，不可省略。

#### 诊断命令索引

```bash
rg "^!" Thesis.log                        # 统计 Error
rg "Overfull.*hbox" Thesis.log | rg "pt"  # 统计表格溢出
rg "Warning.*Citation" Thesis.log         # 查看引用问题
```

---

## 四、NEU-Thesis 模板结构速查

```
<你的NEU-Thesis模板目录>/
├── Thesis.tex          ← 主入口（不要改结构和选项）
├── Style/
│   ├── neuthesis.cls   ← 文档类（禁止修改）
│   ├── artratex.sty    ← 样式宏包（可添加 \RequirePackage）
│   └── artracom.sty    ← 用户自定义宏（可自由添加命令）
├── Tex/
│   ├── Frontpages.tex  ← 封面字段
│   ├── Abstract.tex    ← 中英文摘要
│   ├── Mainmatter.tex  ← 引用章节输入文件
│   ├── Backmatter.tex  ← 致谢 + 附录
│   ├── Chap_1.tex ~ Chap_7.tex  ← 各章节内容
│   └── Tables_Best_Practices.tex  ← 表格参考（不参与编译）
├── Biblio/
│   └── ref.bib         ← BibTeX 参考文献数据库
├── Img/                ← 图表存放目录
└── simfang.ttf, simhei.ttf, simkai.ttf, simsun.ttc ← 中文字体文件
```

### 4.1 模板修改红线

| ✅ 允许修改 | ❌ 禁止修改 |
|-----------|-----------|
| `Style/artracom.sty` 添加新命令 | `Thesis.tex` 的 `\documentclass` 和选项 |
| `Style/artratex.sty` 添加 `\RequirePackage` | `neuthesis.cls` 的格式定义 |
| `Tex/` 下各章节内容文件 | 页面布局、字体规范、页码格式 |
| `Img/` 存放新图片 | `Frontpages.tex` 的字段结构 |
| `Biblio/ref.bib` 增删条目 | 编译引擎（必须用 xelatex） |

---

## 五、自我完善机制

每次编译过程发现新的错误类型或排版技巧时，立即追加到本文档对应章节。

### 5.1 添加新错误的格式

```markdown
### [错误关键字]
- **出现场景**：
- **诊断方法**：
- **修复步骤**：
- **模板修复位置**（如适用）：
```

### 5.2 添加新表格技巧的格式

追加到第二章"表格排版完整规范"中，保持编号递增。

### 5.3 已累积的教训

1. **列对齐错误(Extra alignment tab)**：数 `&` + 1 = 列数，两者必须匹配
2. **宽表溢出(Overfull)**：`\tablefit` 缩放是最通用方案
3. **巨型表格(Float too large)**：`\resizebox{\textwidth}{!}` 包裹整个 tabular
4. **引用缺失(Undefined citation)**：完整4步编译链是最可靠方案
5. **中文乱码**：必须用 `xelatex`，不要用 `pdflatex`
6. **空白页**：模板 `openright` 规范；草稿用 `\let\cleardoublepage\clearpage`
7. **图表跨页**：浮动体用 `[H]` 强制锁定位置，或用 `\FloatBarrier` 屏障
8. **公式与解释分离**：用 `minipage` 捆在一起，或用 `\nopagebreak[4]` 阻断开裂
9. **段尾孤行**：导言区设 `\clubpenalty=10000` + `\widowpenalty=10000` + `\raggedbottom`
10. **三线表缺失**：只用 `\toprule` `\midrule` `\bottomrule`
11. **表格跨页割裂**：用 `longtable` + `\endhead` 重复表头
12. **英文期刊名未斜体**：BibTeX 的 `journal` 字段自动处理
13. **单位与数字无空格**：用 `\Unit{cm}` 宏
14. **公式变量未斜体**：变量用 `$x$`，单位用 `\mathrm{m}`
15. **图题分离**：`\begin{figure}[H]` 锁定图片位置
16. **图片模糊**：导出时设 `dpi=300`
17. **占位符残留**：编译后用脚本扫描 `(作者姓名)` 等关键字
18. **目录页码不匹配**：必须2次 xelatex 解析交叉引用
19. **PDF 路径错误**：编译前 cd 到项目根目录；编译后 verify PDF 落在项目目录下
20. **PDF 文件名不规范**：输出文件必须重命名为论文中文标题，不能保留 "Thesis.pdf"
21. **编译后未告知用户路径**：完成编译和重命名后，必须推送 PDF 的完整绝对路径 + 页数 + 文件大小
22. **minipage 捆绑法**：将文字+图/表放入 `minipage` 环境是最可靠的防跨页手段（优于 float 包 [H]）
23. **全局参数调整**：\topfraction=0.9 + \bottomfraction=0.9 + \textfraction=0.1 让 LaTeX 更愿意在目标位置放置浮动体
24. **\FloatBarrier + [section] 选项**：自动在每个 \section 处插入浮动屏障，是学术论文最优雅的方案
25. **longtable 替代 table**：长表格不能用 table 环境，必须用 longtable + \endhead 重复表头
26. **afterpage 延迟浮动**：用 `\afterpage{\begin{figure}[t!]...}` 将浮动体精确延迟到下一页顶部
27. **\nopagebreak[4]**：在表格前导文字末尾添加，确保"引用文字-表格"视觉连贯性（惩罚值10000，99%不断页）

---

## 六、常用工具宏

以下宏已在 `Style/artracom.sty` 中定义，所有章节文件可直接使用：

| 宏命令 | 作用 | 用法 |
|-------|------|------|
| `\tablefit{...}` | 表格缩放到页面宽度 | `\tablefit{\begin{tabular}...\end{tabular}}` |
| `\tablefootnotesize` | 表格脚注字号 | `\tablefootnotesize 注：...` |
| `\Vector{...}` | 数学向量（粗斜体） | `\Vector{x}` |
| `\Matrix{...}` | 数学矩阵（粗正体） | `\Matrix{A}` |
| `\Unit{...}` | 单位（正体） | `\Unit{kg/m^3}` |
| `\nopagebreak[4]` | 强烈建议在此不分页 | 公式后紧跟 `\nopagebreak[4]` |
| `\looseness=-1` | 尝试将段落收紧一行 | 段落末尾 `\looseness=-1` |

---

## 七、PDF 排版质量全面检测清单

论文编译完成后，按以下12大类逐项检查生成的 PDF。

### 7.1 封面与题名页

| 检查点 | 关键词 | 修复 |
|-------|--------|------|
| 学位类型错误 | 硕士标"博士" | `Tex/Frontpages.tex` 中 `\thesislevel{}` |
| 校名断字 | N O R T H E A S T | 校名用 `\textbf` 包裹 |
| 标题断行混乱 | 多余空格、重复 | 用 `\\` 精确控制换行 |
| 信息缺失 | `(xx)` 占位符 | 逐字段填入真实内容 |

### 7.2 页码与页眉

| 检查点 | 关键词 | 修复 |
|-------|--------|------|
| 页眉重复叠加 | 标题重复N次 | 检查 `\pagestyle` 定义 |
| 页码混用 | 前言阿拉伯/正文罗马 | `\frontmatter`(罗马) `\mainmatter`(阿拉伯) |
| 页码偏移 | I,II,III,IV 重复 | 删除多余 `\pagenumbering{}` |
| 目录页码不匹配 | 标注与实际不符 | 必须 2 次 xelatex |

### 7.3 目录结构

| 检查点 | 关键词 | 修复 |
|-------|--------|------|
| 章节编号错乱 | 5.3 在 5.1 前 | 检查 `\section` 顺序 |
| 层级异常 | 三级标题缩进错 | `\subsection` 嵌套在 `\section` 内 |
| 引导点不齐 | 点线缺失 | 模板自动生成，不需改 |
| 图表索引缺失 | 图号后无标题 | 每个 `\caption{}` 有文字内容 |

### 7.4 正文结构

| 检查点 | 关键词 | 修复 |
|-------|--------|------|
| 标题重复 | 同一章出现多次 | 删除重复 `\chapter{}` |
| 层级混乱 | 3.2.1 缺 3.2 | `\subsubsection` 在 `\subsection` 内 |
| 段首空格 | 空格混用 | LaTeX 自动缩进，不手动加 |
| 多余空行 | 连续空行 | 删除 `.tex` 中多余空行 |

### 7.5 公式与数学符号

| 检查点 | 关键词 | 修复 |
|-------|--------|------|
| 上下标错误 | cm2 应为 cm² | 用 `cm$^2$` / `\textsuperscript{2}` |
| 变量未斜体 | x 应为 $x$ | 变量用 `$x$` 包裹 |
| 正斜体混排 | 单位 m 需正体 | `\Unit{m}` / `\mathrm{m}` |
| 编号不对齐 | 编号偏离 | 用 `\begin{equation}` |
| 公式引用残缺 | `[3]` 写成 `(3)` | `\label{eq:xxx}` + `\eqref{eq:xxx}` |

### 7.6 表格排版

| 检查点 | 关键词 | 修复 |
|-------|--------|------|
| 三线表缺失 | 有竖线/多余横线 | 只用 `\toprule` `\midrule` `\bottomrule` |
| 跨页割裂 | 无表头续页 | `longtable` + `\endhead` |
| 表题分离 | 不在同一页 | `\begin{table}[H]` 锁定 |
| 列宽不均 | 溢出或过空 | 用 `p{3cm}` 固定列宽 |
| 表注缺失 | 无数据来源 | `\tablefootnotesize 注：...` |

### 7.7 图片与插图

| 检查点 | 关键词 | 修复 |
|-------|--------|------|
| 图题缺失 | 图号后无文字 | 每个 `\caption{}` 填标题 |
| 图号不连续 | 缺图3.2 | 检查 `\label` 和引用 |
| 图题分离 | 图题跨页 | `\begin{figure}[H]` |
| 分辨率低 | 模糊/锯齿 | 导出 dpi=300，PNG/PDF |
| 引用格式错误 | 图2-1 → 图2.1 | `\ref{fig:xxx}` |

### 7.8 文字与标点符号

| 检查点 | 关键词 | 修复 |
|-------|--------|------|
| 全角半角混排 | 中文夹英文标点 | 中文后用全角 `，。：；` |
| 引号不配对 | `"xxx"` → `"xxx"` | 直接输入 `""` |
| 破折号错误 | `--` 代 `—` | 用 `——` 或 `---` |
| 单位无空格 | 25cm → 25 cm | `\Unit{cm}` |
| 特殊符号 | ℃ % ± 错位 | `$^\circ$C` `\%` `$\pm$` |

### 7.9 参考文献与引用

| 检查点 | 关键词 | 修复 |
|-------|--------|------|
| 上标位置 | 正文[1] 应为上标 | `\cite{}` 自动上标 |
| 序号不连续 | [1][2][5] | 检查 bib 条目完整性 |
| 英文未斜体 | 期刊名正体 | BibTeX `journal` 自动处理 |
| 条目不齐 | 悬挂缩进缺失 | `.bst` 自动处理 |
| 格式不统一 | 中英文混排混乱 | `ref.bib` 统一管理 |

### 7.10 固定模版页

| 检查点 | 关键词 | 修复 |
|-------|--------|------|
| 声明页污染 | 页眉在声明页 | `\thispagestyle{empty}` |
| 摘要标题重复 |"摘 要"×3 | 删除多余 `\chapter*{摘要}` |
| 中英文不对应 | 段数不一致 | 确保一一对应 |
| 关键词格式 | 逗号分隔 | 统一用 `；` |

### 7.11 格式统一性

| 检查点 | 关键词 | 修复 |
|-------|--------|------|
| 字体混用 | 宋/黑/楷随机 | 宋体正文+黑体标题 |
| 字号混乱 | 同级字数不同 | 模板自动统一 |
| 行距/段距不一致 | 间距差异 | 模板 `\linespread{}` |
| 中英文间距 | 无空格或过大 | XeLaTeX 自动处理 |

### 7.12 内容冗余与乱码

| 检查点 | 关键词 | 修复 |
|-------|--------|------|
| 乱码字符 |"宝""男士" | 替换为正确中文 |
| 占位符残留 |"(作者姓名)" | 替换为真实信息 |
| 代码残留 |"pip install" | 删除非论文内容 |
| 重复段落 | 同一段×2 | 删除重复内容 |

---

## 八、编译后一键排版检查

将打包的 PowerShell 脚本（见 `scripts/check_latex_quality.ps1`）放到模板根目录，编译后运行即可批量扫描：占位符残留、编译 Error、引用未解析、表格溢出、浮动体过大。

---

## 九、批量浮动体修复工作流（攻击模式）

当需要对一篇**已完成写作**的论文进行系统性的浮动体位置修复时，按以下工作流从粗到细、分层推进。

### 9.1 总流程

```
系统性扫描 → 全局参数调整 → 位置参数统一 → 脆弱点绑定 → 编译验证 → 遗留处理
```

### 9.2 Step 1：系统性扫描

用 grep/ripgrep 扫描所有 `.tex` 文件中的浮动体声明：

```bash
rg "\\\\begin\\{(figure|table)\\}" Tex/ --line-number
rg "\\\\begin\\{(figure|table)\\}" --include="*.tex" .
```

**检查每个浮动体的当前位置参数**（`[htbp]`、`[h]`、`[t]`、`[H]` 等），记录数量。目标：建立完整清单。

### 9.3 Step 2：全局参数调整（优先级 2）

在 `Thesis.tex` 导言区（`\begin{document}` 之前）添加或补全全局浮动体参数：

```latex
% ===== 浮动体全局参数 =====
\renewcommand{\topfraction}{0.9}        % 页面顶部最多 90% 给浮动体
\renewcommand{\bottomfraction}{0.9}     % 页面底部最多 90% 给浮动体
\renewcommand{\textfraction}{0.1}       % 正文至少保留 10%
\renewcommand{\floatpagefraction}{0.7}  % 浮动页至少占 70%
\setcounter{topnumber}{3}               % 顶部最多 3 个浮动体
\setcounter{bottomnumber}{2}            % 底部最多 2 个浮动体
\setcounter{totalnumber}{5}             % 每页最多 5 个浮动体
```

如果已存在 `\clubpenalty`、`\widowpenalty`、`\raggedbottom`、`\let\cleardoublepage\clearpage` 则保留不重复添加。

### 9.4 Step 3：位置参数统一（优先级 1）

将 Step 1 扫描到的所有浮动体位置参数统一修改为 `[hbt!]`：

```bash
# 批量替换（谨慎使用，先确认范围）
rg -l "\\\\begin\\{figure\\}\\[htbp\\]" Tex/ | xargs sed -i "s/\\\\begin{figure}\\[htbp\\]/\\\\begin{figure}[hbt!]/g"
rg -l "\\\\begin\\{table\\}\\[htbp\\]" Tex/ | xargs sed -i "s/\\\\begin{table}\\[htbp\\]/\\\\begin{table}[hbt!]/g"
```

> ⚠️ **重要**：执行前必须用 `rg` 确认所有目标，避免误改非标准格式的浮动体。推荐逐文件手动修改以确保精确性。

**参数说明**：
- `h` = here（首选此处）
- `b` = bottom（次选页面底部）
- `t` = top（再次选页面顶部）
- `!` = 忽略 LaTeX 内部限制

### 9.5 Step 4：脆弱点绑定（优先级 4+5）

**4a. `\nopagebreak[4]` 绑定（优先级 4）**

在直接引用图表编号的段落后添加分页禁止指令。当某段文字中包含 `如图\ref{fig:xxx}所示` 或 `见表\ref{tab:xxx}` 且该图表紧接其后时：

```latex
...这一延迟消化周期的形成机制如图\ref{fig:cumulative_returns_path}所示。\nopagebreak[4]
\begin{figure}[hbt!]
  ...
\end{figure}
```

**定位方法**：
```bash
rg "\\\\ref\\{fig:|\\\\ref\\{tab:" Tex/ --line-number
```

为每处引用对应段落的末尾添加 `\nopagebreak[4]`。

**4b. minipage 强绑定（优先级 5，最后手段）**

当 `[hbt!]` + `\nopagebreak[4]` 仍然无法阻止跨页分离时，用 minipage 进行物理绑定：

```latex
\noindent\begin{minipage}{\textwidth}
引导文字如图\ref{fig:xxx}所示。
\begin{figure}[H]\centering
  \includegraphics[width=0.85\textwidth]{Img/xxx.png}
  \caption{...}\label{fig:xxx}
\end{figure}
\end{minipage}
```

> ⚠️ minipage 内的浮动体必须使用 `[H]`（需 `\usepackage{float}`），否则浮动体会脱离盒子。

### 9.6 Step 5：编译验证

执行完整四步编译链：

```bash
cd <项目目录>
xelatex -interaction=nonstopmode Thesis.tex    # 第1步
bibtex Thesis                                   # 第2步（有新增引用时）
xelatex -interaction=nonstopmode Thesis.tex    # 第3步
xelatex -interaction=nonstopmode Thesis.tex    # 第4步
```

**验证标准**：

| 检查项 | 阈值 | 命令 |
|--------|------|------|
| 编译错误 (`^!`) | 必须为 0 | `rg "^!" Thesis.log` |
| 未定义引用 | 必须为 0 | `rg "undefined" Thesis.log` |
| Float too large | 必须为 0 | `rg "Float too large" Thesis.log` |
| Overfull hbox (>50pt) | 尽可能为 0 | `rg "Overfull.*hbox.*\\([5-9][0-9]" Thesis.log` |

### 9.7 Step 6：遗留处理

对于 Step 5 中发现的 `Float too large` 问题：
1. 缩放图片：`width=0.85\textwidth` → `0.78\textwidth`
2. 或降低表格字号：添加 `\zihao{5}` 或 `\footnotesize`
3. 或使用 `\tablefit{}` 包裹 tabular 环境

### 9.8 优先级决策表

| 优先级 | 方案 | 适用场景 | 侵入性 | 何时使用 |
|--------|------|---------|--------|---------|
| 1 | `[hbt!]` 位置参数 | 所有浮动体 | 低（安全） | 第一步，全局统一 |
| 2 | 全局参数调整 | 导言区一次性 | 低（安全） | 第一步，为后续提供宽容环境 |
| 3 | `\FloatBarrier [section]` | 章级别控制 | 低（安全） | 章间浮动体不跨越 |
| 4 | `\nopagebreak[4]` | 引用文字-图表之间 | 中（局部） | 引用段落末，逐处添加 |
| 5 | minipage 绑定 | 必须绝对绑定 | 高（锁死） | 仅当 1-4 全部无效时 |
| 6 | longtable | 超长表格 | 中（换环境） | 表格超过一页时 |

### 9.9 完整示例：一次典型修复的指令序列

```bash
# Step 1: 扫描
rg "\\begin{(figure|table)}" Tex/ --line-number > float_scan.txt

# Step 2: 全局参数（手动编辑 Thesis.tex 导言区）

# Step 3: 位置参数批量修改
rg -l "\\begin{figure}\[htbp\]" Tex/ | while read f; do
  sed -i "s/\\begin{figure}\[htbp\]/\\begin{figure}[hbt!]/g" "$f"
done
rg -l "\\begin{table}\[htbp\]" Tex/ | while read f; do
  sed -i "s/\\begin{table}\[htbp\]/\\begin{table}[hbt!]/g" "$f"
done

# Step 4: 脆弱点扫描
rg "\\ref\{(fig|tab):" Tex/Chap_Format.tex Tex/Backmatter.tex --line-number
# → 手动在每处段落末尾添加 \nopagebreak[4]

# Step 5: 编译验证
xelatex -interaction=nonstopmode Thesis.tex && \
bibtex Thesis && \
xelatex -interaction=nonstopmode Thesis.tex && \
xelatex -interaction=nonstopmode Thesis.tex

# Step 6: 遗留检查
rg "^!" Thesis.log            # 应为 0
rg "Float too large" Thesis.log  # 应为 0
```

---
> Source: [cghggchg765-create/latex-guard](https://github.com/cghggchg765-create/latex-guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
