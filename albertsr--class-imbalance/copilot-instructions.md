## class-imbalance

> 本项目是 `albertsr/Class-Imbalance` 的公开化重构工作区。主案例固定为中国零售信贷申请 PD 建模，核心问题不是把好坏样本机械调整成 `1:1`，而是依次回答：谁进入样本、何时形成成熟标签、训练信号如何改变、模型用于排序还是概率、策略如何把分数转成组合动作。

# Class-Imbalance-refactor AGENTS.md

## 项目定位

本项目是 `albertsr/Class-Imbalance` 的公开化重构工作区。主案例固定为中国零售信贷申请 PD 建模，核心问题不是把好坏样本机械调整成 `1:1`，而是依次回答：谁进入样本、何时形成成熟标签、训练信号如何改变、模型用于排序还是概率、策略如何把分数转成组合动作。

反欺诈作为对照场景，重点讨论极低 FPR、人工审核容量、标签反馈和对抗漂移；不得把 PD 与反欺诈写成共享同一固定成本假设的单一问题。

项目不建设类别不均衡方法百科。2018-2021 年的手写理论、代码和图片保留为历史资产；现代代码只实现能够支撑“样本与标签 -> 训练信号 -> 排序 -> 概率 -> 决策与组合”主线的小型、可测 API。

## 目录与 API 边界

- `src/class_imbalance/` 保存现代 Python API。
  - stable：输入校验、排序/工作点指标、概率修正、成本决策、权重和损失函数。
  - optional：依赖 `scikit-learn` 或 `imbalanced-learn` 的采样与集成辅助能力。
  - experimental：由 legacy 提炼、尚未形成稳定契约的 AdaCost 等逻辑。
  - 旧模块名可以保留兼容 re-export，但新文档必须使用当前主 API。
- `tests/` 按 `unit/`、`contract/`、`integration/`、`scenarios/` 组织；公共 API 必须覆盖正常值、异常值、边界值和语义不变量。
- `examples/` 只保存公开安全、可直接运行的合成案例；不得使用真实客户数据、内部阈值或生产规则。
- 根 `README.md` 是面向国际读者的英文默认入口；`README.zh-CN.md` 是完整中文入口。`README.en.md` 只保留为旧英文链接的兼容跳转页。
- `docs/zh-CN/` 保存中文主叙事；`docs/en/` 保存实质英文改写；两者的 `labs/` 子目录只保存不进入主线的专题实验。`docs/reference/` 保存英文权威 API 参考；`docs/history/` 保存重构和历史说明；`docs/audits/` 保存审计证据。
- `legacy/` 保存历史材料和清单，不是稳定 API，也不是默认公开白名单。
- `scripts/` 只保存仓库级 UTF-8、链接、公开边界、构建和 smoke 检查；脚本不得修改源码或自动删除文件。

新建目录前先在本文件定义用途；不建立空目录或同义平行入口。

## 历史溯源分级

所有公开材料按以下四类记录，不能把现代新增内容冒充为历史手写代码：

- `H`（Handwritten/verbatim）：能与原 `master` blob 对应的 Ma Xiao 手写原件；保持内容和字节不变，不格式化、不批量翻译、不静默修正。
- `D`（Derived/adapted）：从 `H` 类原件提炼的现代实现；必须记录原路径、原符号、语义变化和 parity/characterization tests。
- `N`（New engineering）：2026 年新增的校验、打包、测试、文档、兼容层和 CI；明确标记为现代工程能力。
- `Q`（Quarantine/unclear）：第三方、来源不清、依赖私有 API 或不可复现的历史资产；默认冻结并排除公开。

英文历史理论采用“忠实正文 + 明确标注的现代编者说明”。正文保留原推理顺序；今天需要补充的条件、边界或修正不得静默写回历史原文。

## 中国信贷叙事规则

中文主线必须覆盖以下五层：

1. 样本总体与标签：申请流量、历史审批选择、获批/放款样本、观察窗、表现窗、标签成熟、删失与拒绝样本标签不可观测。
2. 训练信号：原始分布、采样、类别权重、实例权重和损失函数分别改变什么。
3. 排序：AUC、KS、PR-AUC、固定 FPR 召回和跨时间稳定性。
4. 概率：PD 的条件总体、类别先验、先验修正、概率校准和适用边界。
5. 决策与组合：阈值、通过率、坏账率、预期损失、风险收益、人工容量和策略反馈。

不得把“未重采样验证集”直接称为全部申请总体的真实业务分布。更准确的默认口径是“历史策略条件下、标签已成熟的获批或放款客群分布”；拒绝推断必须显式写出不可识别边界和额外假设。

坏样本内部异质性是可检验的研究假设，不是既定业务事实。按未来违约时点分组属于结果分层；用于训练权重、阈值或部署规则前必须说明 estimand、可观测性和稳定性风险。

历史理论中的 sampling、weighting、prior、cost 与 threshold 联系应保留原始理解路径，同时增加现代说明，写清已知抽样机制、校准后验、固定且实例无关成本、模型容量和正则化等适用条件。

英文历史理论不是逐句机器翻译，也不能改写成通用算法百科；应保持 Ma Xiao 的推导顺序，并为欧美读者补充标准术语、假设、适用边界和信贷场景上下文。

## 代码与测试规则

- 标识符、模块名、包名和公共 API docstring 使用英文；业务案例说明、非显然实现注释可使用中文，所有中文文件保持 UTF-8。
- 公共数值 API 默认 fail closed：校验一维、非空、有限值、二元标签、概率范围、非负权重、正成本、合法阈值和合法 `eps`。
- 概率成本阈值只能接收目标部署总体上的校准概率；普通 ranking score 必须使用独立的工作点选择 API。
- 历史竞赛 `weighted_coverage` 与业务 `FPR cap` 指标必须是两个明确口径，不得用默认插值悄悄改变历史语义。
- sampling 只能进入训练 pipeline；validation、test 和 OOT 保持未采样。示例优先使用 `imblearn.pipeline.Pipeline`，避免交叉验证泄漏。
- 不重复实现 `scikit-learn` 已提供且稳定的通用算法；本包只提供项目独有的契约、轻量公式和信贷语义封装。
- 新增或修改稳定逻辑必须补测试；不能通过注释错误或绕过校验来使测试通过。
- `D` 类实现必须先用 characterization/golden tests 锁定手写逻辑，再做函数级小步适配；不允许整文件替换后宣称“保留原实现”。

默认验证流程：

```powershell
python -m pip install -e ".[dev]"
python -m pytest -q
python -m build
```

构建完成后还要对 wheel 做隔离安装 smoke test，并运行所有公开示例。验证默认使用 `pywork_py311`。

## 依赖与打包

- core 依赖保持小而明确，当前只允许 NumPy。
- 可选依赖按 `sklearn`、`sampling`、`boosting`、`dev` 分组；不要把 XGBoost、LightGBM、pandas 等重型依赖混入 core。
- `pyproject.toml` 必须包含项目 URL、许可证、关键词、分类器、可选依赖和测试/质量配置。
- README 的安装与测试命令必须在 fresh clone + editable install 后可复现，不依赖隐式 `PYTHONPATH`。
- 版本、支持的 Python 范围和依赖下限必须由实际验证支撑。

## 历史资产与公开边界

- `D:\pywork\03_Github重构\_github_originals\Class-Imbalance` 是只读原仓快照，不得修改；它承担原始二进制和原始路径的溯源保存。
- `legacy/MANIFEST.md` 必须记录历史资产的来源、年份、类型、当前状态、现代替代入口、可复现性、作者与公开许可。
- MIT 许可证只覆盖马老师拥有权利的现代代码、文档和自制资产，不覆盖第三方论文、数据、截图或外部代码。
- 没有明确再分发许可的第三方论文 PDF 不保留在重构仓的公开跟踪树；用书目、DOI 或作者/出版方官方链接替代。tracked 删除必须单独复核和提交。
- legacy 原始文本和 notebook 默认不批量改写、不因路径相似删除、不重写 Git 历史。旧邮箱不在现代 README 和新文档中展示；legacy 原文中的作者元数据保持历史状态，除非马老师另行要求隐私清理。
- `legacy/intermediate-workspace/data/*.csv` 继续 ignored；任何数据进入示例前都要确认来源、许可与公开必要性。
- 文件名或内容包含 `private`、`confidential`、`客户`、`身份证`、`手机号`、`token`、`password`、`专利`、`patent`、`DWIP` 时，默认阻止公开提交。

## CI 与发布边界

- `.github/workflows/ci.yml` 只允许执行 lint、测试、构建、安装 smoke、链接/编码/敏感模式检查；不得上传 PyPI、创建 release、部署或写远端状态之外的外部系统。
- CI 配置与代码、文档分开提交。
- `git push`、默认分支合并、GitHub Release、PyPI 发布或任何部署必须分别取得明确授权。阶段 1—3 的精选分支推送与草稿 PR 已获授权；这不包含合并、tag、Release 或 PyPI。

## Git 与提交纪律

- 不使用 `git add .` 或 `git add -A`。
- 提交前运行 `git status --short --branch --untracked-files=all`、`git diff --stat` 和敏感/大文件检查。
- 规则、核心 API、测试、中文文档、英文文档、legacy 治理、打包和 CI 分主题显式 staging/commit。
- tracked rename、PDF 删除、换行归一化和内容改写不得混在同一个提交。
- 新提交的作者与提交者身份统一使用 `Ma Xiao`；已发布的历史提交不改写，未发布的参考分支也不通过 `.mailmap` 冒充作者。
- 未经明确确认，不 push、rebase、reset hard、force-push、发布或部署。

## 当前实施顺序

1. 先完成本规则、MIT 范围、`Ma Xiao` 身份和 H/D/N/Q 溯源边界。
2. 将英文设为 GitHub 默认入口，同时保留完整中文路径和历史中文原文。
3. 完成主页历史推导的实质英文适配，并把现代限定与历史正文分开。
4. 先补历史行为测试，再小步适配 metrics、operating points、rebalancing、sampling 与 experimental AdaCost。
5. 完成 packaging、公开边界、全量测试和构建验证后按主题保存。
6. 最后加入只测试、不发布的 CI；合并、tag、Release 与 PyPI 继续单独决策。

---
> Source: [Albertsr/Class-Imbalance](https://github.com/Albertsr/Class-Imbalance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
