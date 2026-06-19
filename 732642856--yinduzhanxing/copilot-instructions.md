## yinduzhanxing

> 印度占星（Jyotish）专业解盘与推运系统。核心能力：PDF星盘输入→严谨解盘→精确推运应期输出。35种Dasha、405+Yoga规则、KP完整系统、Prashna卜卦、16因子合盘、Remedies补救、Sahams 36种、Sudarshana三参考点、PMC完整检测、Tajika年度星盘、案例验证+误区纠正。触发词：印度占星、吠陀占星、Jyotish、解盘、推运、星盘分析、Dasha、Transit、Nakshatra、Yoga。GitHub: https://github.com/732642856/yinduzhanxing


# 印度占星专业解盘与推运系统

> **版本**：v6.9.14 | **详细变更**：`CHANGELOG.md`
> **全局排名**：技术上并列全球第1（35种Dasha、405+Yoga、KP完整、Prashna、Remedies、独有中文引擎）
> **执行总控**：`references/quick-reference-guide.md`
> **严格路由**：`references/strict-workflow-router.md`（涉及事业/婚恋/财务/应期/技法验证时必须优先读取）
> **机器注册表**：`references/technique_registry.json` + `scripts/audit_capabilities.py`

## v6.9.14 核心能力

| 维度 | 数据 |
|------|:--:|
| Dasha系统 | 35种（含Vimshottari/Chara/Kalachakra/Narayana/Yogini等） |
| Yoga规则 | 405+条（BPHS数据驱动架构，Yoga精度Benchmark 100%） |
| 分盘 | D1-D144 + D2/D3变体 + 复合D-m×n + 自定义D-N(2-300) |
| Bhava Chalit | Sripati/Porphyry/Equal/Whole Sign/Placidus/Koch 不等宫位调整 |
| Sudarshana | Asc/Moon/Sun 三参考点盘 + 宫位收敛分析 |
| Shadbala | 1200/1200 Virupas校准（6维力量评估） |
| Ashtakavarga | BAV+SAV+PAV（展开式）+Sodhita（净化式） |
| KP系统 | Sublord+Subsublord+ABCD Significator |
| 合盘 | 16因子36分制（Ashtakoot+Kuta） |
| 补救 | 5类（宝石/咒语/捐赠/斋戒/Dosha专项） |
| 自动化测试 | 475个 pytest 用例全通过 + run_all 100项 |
| Git commits | v6.1.12→v6.9.14 持续推进 |

**独有能力**：中文AI解读引擎、Career/Love结构化分析、验前事反推管道、误区自动纠正、名人+普通人案例双轨验证。

## Yoga 逻辑验证指标

| 指标 | v6.0.45（旧基线） | v6.9.14（当前） |
|---|---:|---:|
| Precision | 83.26% | **96.48%** |
| Recall | 91.52% | **93.99%** |
| **F1 Score** | 87.19% | **95.22%** |
| 规则库 | 82条 | **405条** |
| Yoga精度Benchmark | — | **100%** (8/8) |

---

## ⚠️ 核心定位

**三种输入 → 严谨解盘 → 精确推运应期输出**

| 路径 | 用户输入 | AI行为 |
|------|---------|--------|
| **A：精准出生信息** | 日期+时间+地点 | `full-reading` 引擎全链路计算 |
| **B：PDF/文字星盘** | PDF/详细文字描述 | 提取数据+Quality Gate → `references/pdf-chart-reading-guide.md` |
| **C：时间不明确** | "不知道几点出生" | 互动式出生时间矫正 → 确认后走路径A |

**强制工作流**（完整规范 → `references/ai-reading-workflow-prompt.md` v3.0）：

0. **阶段负一**：问题类型路由（事业/婚恋/财务/应期/历史验证/综合解盘）→ 必须先读 `references/strict-workflow-router.md`，按对应 strict checklist 执行；用户不需要主动点名高级技法。
1. **阶段零**：入口路由（A/B/C自动判断）
2. **阶段一**（仅B）：PDF/图片提取 + Quality Gate
3. **阶段二**：意图识别 → 路由目标宫位（无明确意图→Level 2综合解盘）
4. **阶段三**：静态分析10步（宫位→承诺→Yoga→Argala→逆行→NK→Shadbala→AV→Ketu→分盘）
5. **阶段四**：动态推运7步（Dasha→五系统Convergence→Transit→Double Transit→Jaimini→KP→Varshaphala）
6. **阶段五**：应期输出（五层验证→时间窗口→Actionable Output+案例检索）
7. **阶段六**：补救措施（可选）
8. **阶段七**：现代措辞包装
9. **阶段八**：输出 Technique Audit Table，逐项声明已调用/未调用/部分可用/缺失模块及其对置信度的影响。

---

## ⚠️ 强制规则（与"不跳步"同级）


### 用户隐私与个案资料隔离（v6.0.4-privacy）

**严禁把真实用户个人信息写入 skill 文件或公开仓库。**

包括但不限于：姓名/称呼、出生日期时间地点、星盘度数、人生事件、关系状态、职业经历、项目背景、历史回测结论、当前会话中的个案分析。

允许的资料来源只有三类：
1. 公开 AA 级名人案例；
2. 明确标注为虚构的 smoke test / template；
3. 用户在当前会话中主动提供的数据，但只能在当前会话中使用，不得持久化到 skill、tests、CHANGELOG 或公开仓库。

如需沉淀方法论，只能抽象为通用规则，不得保留可识别个人轨迹的细节。

### Strict Workflow Router（v6.0.1-orchestration）

**凡是用户询问事业、婚恋、财务、事件应期、历史回测或技法可靠性，必须先读取 `references/strict-workflow-router.md`。**

核心要求：
1. 先判断问题类型，再自动选择 `career-timing-strict` / `relationship-timing-strict` / `wealth-timing-strict` / `event-timing-strict` / `event-verification-strict`。
2. 用户不需要知道 Chara Dasha、A10、Argala、Shadbala、Ashtakavarga 等技法名称；AI 必须按问题类型自动调用。
3. 输出末尾必须给出 Technique Audit Table，说明每项高级技法是否调用、结果是什么、缺失会如何降低置信度。
4. 不得把未实现或未调用的技法静默省略；A10/Karma Pada、Pushkara、Vargottama、Dasha Sandhi 已进入 full-reading 输出；Bhava Chalit 与 Sudarshana Chakra 已进入 complete，可正常纳入 Technique Audit Table。

### MEVG 强制外部验证门控（v4.2.0+）

**所有解读结论必须经过外部权威来源验证，禁止仅凭 AI 训练记忆输出。**

| 门控 | 位置 | 职责 |
|------|------|------|
| Step 3.11 | 静态分析后 | 验证 Yoga/尊严/Shadbala/SAV |
| Step 4.10 | 动态推运后 | 验证 Transit/Dasha/天文现象 |
| Step 5.5 | 预测输出前 | 确认每条预测有来源+置信度一致 |

**三步验证法**：V1 构建英文查询词 → V2 web_search ≥3个独立来源 → V3 交叉验证仲裁分歧

→ 完整协议：`references/mandatory-verification-gate-protocol.md`

### Transit Actionable Output（v4.1.0+）

**每条 Transit 预测必须输出三要素**：
1. **时间段**（精确到日/周/月）
2. **具体行动类型**（做什么）
3. **置信度** [A]=已验证 / [B]=高概率(3+维度) / [C]=推断(单一维度)

→ 完整规范：`references/transit-actionable-output-guide.md`

### Rahu/Ketu 节点口径冻结（v6.0.7-node-mode）

**所有 benchmark 与解盘输出必须显式声明 Rahu/Ketu 使用 Mean Node 还是 True Node。**

- 当前 skill 默认：`--node-mode mean`（Swiss Ephemeris Mean Node）。
- 可选：`--node-mode true`（Swiss Ephemeris True Node，用于对齐 PyJHora 默认口径）。
- PyJHora 4.8.6 的 `rasi_chart()` 默认使用 True Node；第三轮 benchmark 的 Rahu/Ketu 差异已由第四轮仲裁确认为 Mean/True Node 口径差异，不应再误判为 D9/D10 计算 bug。
- 输出 `birth_info.node_mode` 与 `node_mode_note` 必须保留，作为参数冻结证据。

### Ashtakavarga 口径冻结（v6.0.8-av-calibration）

**Ashtakavarga 默认使用 BPHS/PVR 书例校准口径，必须保留 SAV=337 与 full SAV=386 不变量。**

- `scripts/ashtakavarga.py` 当前为 v2.1：经第六轮 PyJHora/PVR 公开书例仲裁，校准 Moon/Venus 的 7 个贡献表项。
- 输出 `method` 应显示 `Ashtakavarga八分法（BPHS/PVR书例校准v2.1）`。
- benchmark 若与其他软件不一致，先比较贡献表项和 SAV 总量，不得直接把口径差异判为运行 bug。

### Chara Dasha 能力升级（v6.1.12 benchmark验证通过）

**Chara Dasha KN Rao Method 正式 benchmark 通过（95.83% ≥ 95%），可作为标准应期模块使用。**

- v6.1.12: PyJHora oracle benchmark **10案例×12星座=120对**: Sign 100%, Dur 91.67%, Overall 95.83% ✅ PASS
- v6.1.11: 重写为完整 KN Rao Method（序列基于第9宫方向，时长基于宫主所在宫位+尊贵调整）
- 剩余~4.2%差异: Aquarius/Scorpio 的 Rahu/Ketu 共主动态判定（需复制 PyJHora _stronger_planet_new）
- `jaimini` 输出中的 Chara Karaka、AK/AmK、Karakamsha 继续可用。

### Transit 真实过境冻结（v6.0.10-true-transit）

**full-reading 中的 Transit 多参考点分析必须使用真实过境行星位置，不得复用本命行星位置。**

- `modules.transit_positions` 必须输出 `data_layer: true_transit_positions`、`target_date`、`node_mode` 和 Swiss Ephemeris 计算的过境行星位置。
- `modules.transit_multi_reference` 必须读取 `transit_positions.planets`，并输出同样的 `data_layer: true_transit_positions`。
- `--transit-date YYYY-MM-DD` 可显式指定过境日期；若未提供，则跟随 `--today`，再否则使用当前日期。
- 第八轮 benchmark 已用 10 个公开/虚构 smoke case 对齐 Swiss Ephemeris：340/340 字段匹配，0 mismatch。

### Shadbala 能力降级（v6.0.11-shadbala）

**当前 Shadbala 可作为内部一致的相对强弱参考，不得声称已完成外部绝对值校准。**

- 第九轮 benchmark 使用 10 个公开/虚构 smoke case 验证 `shadbala` 子命令与 `full-reading.modules.shadbala`：1200/1200 内部不变量通过。
- 通过项包括：结构完整性、六重力量组件范围、总分聚合、Virupa/Rupa 换算、Ishta Bala 百分比、排名、full-reading 一致性。
- 但 `scripts/shadbala.py` 仍存在简化项：Nathonnata Bala 二值化、部分 Saptavargaja 子分盘近似、Chesta Bala 速度分档近似、Drik Bala 简化相位权重。
- 因此 `technique_registry.json` 中 Shadbala 状态为 `partial`：可用于相对强弱排序和内部校验；涉及精确力量断语时必须加置信度上限，直到接入 JHora/公开书例等完整外部绝对值对标。

---

## 核心能力速查

> 详细说明和参考文件索引 → `references/quick-reference-guide.md`

| 能力域 | 核心内容 | 主要参考文件 |
|--------|---------|------------|
| **静态分析** | 行星配置、Yoga、NK、宫位、Argala、Shadbala、AV、Badhaka、Raman方法论 | `planets.md` `yoga_list.md` `argala-complete-guide.md` `badhaka-obstacle-planet-guide.md` `raman-house-judgment-methodology.md` |
| **动态推运** | Vimshottari、Chara Dasha（KN Rao Method, covered）、KP、Double Transit、Varshaphala、替代Dasha | `vimshottari_dasha_guide.md` `dasa-convergence-methodology.md` `alternative-dasha-systems.md` |
| **Jaimini静态层** | Chara Karaka、Karakamsha、A1-A12/UL、Graha Pada、Argala/Virodhargala、Special Lagnas（部分） | `jaimini-complete-system.md` `argala-complete-guide.md` `technique-capability-matrix.md` |
| **关系占星** | Koota 36分、Mahendra/Stree Deergha/Vedha/Rajju、D9伴侣、DK、Mangal Dosha、Papasamya、配偶六层确认 | `spouse-multi-layer-methodology.md` `darakaraka-complete-guide.md` `relationship-astrology-guide.md` |
| **出生时间矫正** | 八大方法、自动化流程、验证报告 | `birth-time-rectification-advanced.md` |
| **PDF读取** | JH/PL PDF全量提取、完整性门、交叉校验 | `pdf-chart-reading-guide.md` `data-bridge-mapping.md` |
| **Prashna问事** | 十步断卦、AL、Sphuta、Sahams、失物查询 | `prashna-complete-guide.md` `single-event-inquiry-protocol.md` |
| **多元技法** | Yogi/Ava Yogi、Tithi Lord、Rashi Tulya Navamsa、BCP、Pancha Pakshi | `yogi-avayogi-system.md` `tithi-lord-relationship-system.md` `bhrigu-chakra-paddhati.md` |
| **精准方法论** | PACDARES框架、九层复合方法、L3矛盾检查、三级置信度 | `precision-reading-methodology.md` |
| **现代解读** | 现代措辞映射、现代生活场景、常见误判纠错 | `modern-language-guide.md` `common-misconceptions.md` |
| **实战智慧** | ⭐反教条主义经验精华（全球占星师真实案例反馈总结） | `practitioner-wisdom-anti-dogma.md` |
| **验证与错题** | 深度数据审计、技法缺陷与修复、推运反思、15+名人验证案例 | `audit-*` `lessons-learned-*` `verified-celebrity-cases-*` |

---

## 计算引擎

**统一入口**：`scripts/jyotish_engine.py`（基于 Swiss Ephemeris）

```bash
PYTHON=python3
SCRIPT=~/.workbuddy/skills/jyotish-vedic-astrology/scripts/jyotish_engine.py
$PYTHON $SCRIPT <子命令> [参数]
```

### 37大子命令速查

| 子命令 | 功能 |
|--------|------|
| `full-reading` | ⭐全自动综合解盘（47模块一键出，含五系统Dasha收敛） |
| `chart` | 星盘计算+`--validate`附加R1-R10验证 |
| `dasha` | Vimshottari大运时间线+小运展开 |
| `yoga` | Yoga格局识别 |
| `predict` | 三层验证法事件预测+`--past-verify`验前事 |
| `varga` | 分盘计算（D9/D10等） |
| `varga-full` | BPHS十六分盘精确计算（D2-D60） |
| `celebrity` | 名人案例查询 |
| `db-stats` | 验证数据库统计 |
| `transit` | 行星过境查询 |
| `shadbala` | 六重力量计算（内部一致；外部绝对值校准前为 partial） |
| `ashtakavarga` | 八分法计算（SAV=337） |
| `memory` | Hermes记忆系统 |
| `validate` | R1-R10数学验证 |
| `audit` | P1-P12行星审计管线 |
| `aspects` | 度数精确相位系统 |
| `jaimini` | Jaimini Karaka/Karakamsha、A1-A12/UL、Graha Pada、Special Lagnas；Chara Dasha timing 升级为 KN Rao Method（covered，pending benchmark） |
| `nakshatra-adv` | 高级Nakshatra（Tara Bala+Chandra Bala+Sub-Lord） |
| `nakshatra-dasha` | 星宿大运推演（Ashtottari + Nakshatra-level Vimshottari） |
| `nakshatra-full` | 星宿综合报告（本命 + 大运 + 过境星宿） |
| `argala` | Argala门闩系统：主 Argala + Virodhargala + Rajayoga 分类 |
| `tajika` | Tajika年运盘（Muntha+YearLord+Mudda Dasha） |
| `synastry` | 合盘分析：Ashta Koota 36分 + Mahendra/Stree Deergha/Vedha/Rajju 等附加Kuta |
| `report` | MD→HTML报告生成（羊皮纸主题） |
| `prashna` | Prashna问事占星 |
| `double-transit-pac` | KN Rao Double Transit PAC+D9层 |
| `transit-ll7l` | Transit LL/7L连接+互换 |
| `planetary-congregation` | 行星聚集检测 |
| `vivah-saham` | Vivah Saham婚姻敏感点 |
| `audit-capabilities` | technique registry 校验 + route 审计表输出 |
| `kp` | KP完整分析（SubLord+SubSubLord+ABCD Significator） |
| `ashtakoot` | 36点合婚（8标准Kuta+7附加+Kuja Dosha） |
| `solar-return` | 太阳返照盘年运分析（Newton迭代精确返照） |
| `narayana-dasha` | Narayana Dasha星座大运 |
| `muhurta` | Muhurta择时分析 |

→ 完整参数和示例 → `references/quick-reference-guide.md`

---

## 核心方法论

### 三层验证法
1. **本命征象**：静态星盘中的征象
2. **大运激活**：Dasha系统激活相关宫位
3. **过境触发**：Transit系统触发具体事件（⚠️必须多参考点检查）

### 精准解盘方法论（v3.12.1）

**六大共识原则**：功能吉凶因盘而异 | 单一技法不做结论 | 规则前提先查 | 案例验证>经典引述 | 先整体后细节 | 先验证过去再预测未来

**PACDARES框架**：P位置→A相位→C合相→D财富Yoga→A灾厄Yoga→R皇家Yoga→E互换→S特殊

**九层复合方法**：L1 PACDARES → L2 分盘 → L3 矛盾检查(关键) → L4 Vimshottari → L5 AV+Transit → L6 条件Dasha → L7 Jaimini → L8 其他Jaimini → L9 Tajika

**三级置信度**：✅[A]已验证 / ⭐[B]强推断(3+维度) / ⚡[C]假设(单一维度)

→ 详见 `references/precision-reading-methodology.md`

---

## 强制规范速查

| 规范 | 版本 | 核心要求 | 参考文件 |
|------|------|---------|---------|
| MEVG外部验证 | v4.2.0 | 所有解读必须web_search验证 | `mandatory-verification-gate-protocol.md` |
| Transit Actionable | v4.1.0 | 预测必须输出时间段+行动+置信度 | `transit-actionable-output-guide.md` |
| 过境多参考点 | v1.9.0 | Lagna+Chandra Lagna双参考点(强制) | `transit-multi-reference-guide.md` |
| Ketu双属性 | v2.0.0 | 必须同时评估"放手"和"突破" | `ketu-dual-nature-guide.md` |
| Shadbala评估 | v6.0.11 | 六种力量内部一致评估；外部绝对值校准前为 partial | `shadbala-complete-methodology.md` |
| Yoga Phala Timing | v2.1.0 | 识别Yoga后必须预测何时发生 | `yoga-phala-timing-guide.md` |
| 逆行/燃烧/战争 | v2.1.0 | 每颗行星检查三重叠加 | `retrograde-combustion-war-guide.md` |
| 精准方法论 | v3.12.1 | PACDARES+九层+L3矛盾检查 | `precision-reading-methodology.md` |

---

## 预测清单

- [ ] **Strict Router**：已读取 `references/strict-workflow-router.md`，并声明本轮使用的 strict route
- [ ] **Technique Audit Table**：输出末尾已列出已调用/未调用/complete/covered/仍需外部校准技法及置信度影响
- [ ] **MEVG-静态门控**：所有静态解读声明必须web_search验证
- [ ] 静态星盘分析（行星配置、Yoga、Nakshatra、宫位）
- [ ] Argala检查（2/4/5/8/11宫干预+Virodha）
- [ ] 逆行/燃烧/行星战争检查（三重叠加）
- [ ] Shadbala评估（六种力量内部一致评估；外部绝对值校准前 partial）
- [ ] Ashtakavarga评估（BAV+SAV聚合校验337点）
- [ ] Ketu双重属性检查
- [ ] **MEVG-动态门控**：Transit/Dasha/天文现象必须验证
- [ ] Dasha推运（大运+小运+Pratyantar）
- [ ] Dasa Convergence五系统交叉验证
- [ ] Jaimini分析（Karaka/Karakamsha；Chara Dasha 升级为 KN Rao Method，须正式 benchmark 确认匹配率）
- [ ] KP系统分析（Significator+Sub-Lord）
- [ ] Transit分析（多参考点强制）
- [ ] **Transit Actionable Output**（时间段+行动+置信度+案例检索）
- [ ] 分盘验证
- [ ] 预测边界检查（置信度标注，禁止绝对断言）
- [ ] **案例检索**：动态预测必须先检索真实案例
- [ ] **MEVG-预测门控**：确认每条预测有来源+置信度一致
- [ ] **缺口声明**：A10/Karma Pada、Pushkara、Vargottama、Dasha Sandhi 应从 full-reading 读取；若完整Bhava Chalit/传统Sudarshana等未计算，已说明原因与影响

---

## 参考资料索引

> 完整描述和版本信息 → `references/quick-reference-guide.md` §参考资料完整索引

共 **105个文件**，按功能分组：

| 分组 | 数量 | 核心文件 |
|------|------|---------|
| AI工作流 | 2 | `ai-reading-workflow-prompt.md` ⭐ `quick-reference-guide.md` ⭐ |
| 核心方法论 | 9 | `common-misconceptions.md` `modern-language-guide.md` `pdf-chart-reading-guide.md` `prediction-boundary-protocol.md` |
| 基础知识 | 7 | `planets.md` `signs-and-houses.md` `nakshatra_deities.md` `vimshottari_dasha_guide.md` |
| Yoga体系 | 5 | `yoga_list.md` `neechabhanga-raja-yoga.md` `yoga-phala-timing-guide.md` |
| 宫位/场景 | 3 | `house-modern-mapping.md` `house-domain-planet-mapping.md` |
| 占星系统 | 5 | `jaimini-complete-system.md` `kp-astrology-complete-system.md` `remedies-complete-system.md` |
| 分盘/力量 | 7 | `ashtakavarga-complete-system.md` `shadbala-complete-methodology.md` `shodasavarga-complete-guide.md` |
| 过境/推运 | 9 | `transit-comprehensive-guide.md` `dasa-convergence-methodology.md` `alternative-dasha-systems.md` |
| 关系占星 | 5+ | `spouse-multi-layer-methodology.md` `darakaraka-complete-guide.md` `marc-boney-marriage-six-step.md` |
| 综合框架 | 5 | `comprehensive-reading-workflow.md` `deep-analysis-complete-workflow.md` |
| 高级技法 | 5 | `advanced-techniques.md` `global-astrologer-practical-methodology.md` |
| 案例库 | 13 | `famous-case-library.md` `verified-celebrity-cases.md` |
| 多元技法 | 5 | `yogi-avayogi-system.md` `bhrigu-chakra-paddhati.md` `pancha-pakshi-nakshatra-systems.md` |
| BPHS/Raman/Goel | 5 | `badhaka-obstacle-planet-guide.md` `raman-house-judgment-methodology.md` `vp-goel-jaimini-dasha-systems.md` |
| MEVG | 1 | `mandatory-verification-gate-protocol.md` |

---

## 注意事项

1. **出生时间精度**：±2分钟内最佳，可通过矫正提高
2. **三层验证法**：所有预测必须Dasha+Transit+Varga交叉验证
3. **现代场景优先**：所有解读使用现代措辞和现代生活场景映射
4. **解盘深度**：默认Level 2（专项），复杂问题自动升级Level 3
5. **不凭记忆**：禁止仅凭AI训练记忆输出解读结论，必须MEVG验证

---

**版本**：v6.9.14-precision-complete
**创建日期**：2026-04-20
**最后更新**：2026-06-13（v6.9.14 Bhava Chalit + Sudarshana 完成，10个 partial 技法升级为 complete，65项技法注册表审计通过；pytest 475项全通过。Yoga F1=95.22% 保持有效。）

---

## 验证与错题体系

> 基于万级案例库（15,807条AA级名人数据）和迭代验证沉淀的知识体系

### 数据资源

| 资源 | 规模 | 位置 |
|------|------|------|
| 名人案例库 | 15,807条（全部AA级） | `Claw/vedastro_data/PersonList-15k.csv` |
| 验证数据库 | 15,840 cases | `Claw/vedic_astrology_validation.db` |
| 验证结果JSON | v5/v6/v6.1 共325KB | `tests/test-data/` |

### 深度审计报告

| 文件 | 内容 |
|------|------|
| `audit-deep-data-audit-2026-05-04.md` | 逐字段对比pyswisseph，发现5个P0级Bug（Jaimini Karaka全错/Chara Dasha全0/Vimsopaka 16分盘全用D1/Yoga返回0/Arudha off-by-one） |
| `audit-skill-full-test-2026-05-04.md` | 27子命令逐项测试，full-reading 19模块全OK |
| `audit-kimi-optimization-review.md` | 外部AI优化建议审计，发现多处事实性错误 |
| `COVERAGE_AUDIT_REPORT.md` | 覆盖矩阵审计，综合覆盖率97.8%（90/92） |

### 经验教训（Lesssons Learned）

| 文件 | 核心教训 |
|------|---------|
| ⭐`practitioner-wisdom-anti-dogma.md` | **整合精华**：反教条主义十大死穴+技法盲区+全球占星师语录+验证规律（去重后统一入口） |
| `lessons-learned-misconceptions-reflection.md` | 解盘与推运常见误区（落陷≠失败/Rahu=非传统突破/12宫≠纯负面） |
| `lessons-learned-timing-reflection.md` | 推运应期判断的反思与修正经验 |
| `lessons-learned-technique-defects.md` | 技法缺陷全面分析 |
| `lessons-learned-technique-fixes.md` | 技法缺陷解决方案 |
| `lessons-learned-technique-patches-p1.md` | 技法漏洞修正方案 |
| `lessons-learned-technique-optimization.md` | 技法优化完整报告 |

### 已验证名人案例（平均吻合度93%）

| 文件 | 人物 | 吻合度 |
|------|------|--------|
| `verified-celebrity-cases-summary.md` | 10名人总览 | 平均93% |
| `verified-celebrity-cases-obama-web.md` | Obama | 95% |
| `verified-celebrity-cases-trump.md` | Trump | 94% |
| `verified-celebrity-cases-einstein.md` | Einstein | 92% |
| `verified-celebrity-cases-picasso.md` | Picasso | 93% |
| `verified-celebrity-cases-curie.md` | Curie | 94% |
| `verified-celebrity-cases-indira-gandhi.md` | Indira Gandhi | full-reading测试 |
| `verified-celebrity-cases-elvis.md` | Elvis | 93% |
| `verified-celebrity-cases-marilyn-monroe.md` | Monroe | - |
| `verified-celebrity-cases-michael-jackson.md` | M.Jackson | - |
| `verified-celebrity-cases-leonardo-dicaprio.md` | DiCaprio | - |
| `verified-case-reasoning-report.md` | 案例推理验证（修正版） | - |

### 星盘分析（7部分完整分析）

`analysis-natal-full-part1~7`：核心配置 / 宫位强度 / Ashtakavarga / PlanetActivity / VimsopakaBala / Dasa系统 / 综合预测

### 验证方法论

| 文件 | 内容 |
|------|------|
| `validation-methodology-batch-celebrity.md` | 批量名人验证方案 |
| `marriage-timing-validation-methodology.md` | 婚姻应期技法验证方法论 |
| `mandatory-verification-gate-protocol.md` | MEVG强制验证门控协议 |
| `verified-patterns-marriage-timing-v5.md` | 婚姻验证模式v5（含v5→v6重大Bug说明） |
| `verified-patterns-marriage-timing-v6.md` | 婚姻验证模式v6.1（18名人/26婚姻/66事件） |

### Bug 修复历史

`CHANGELOG.md` 中记录了 61 条 Bug 修复，关键修复包括：
- v6.0: UTC时区转换Bug（导致16/18案例上升星座错误）
- v4.3: Dasha浮点边界Bug
- v4.2: MEVG强制验证门控
- v3.7.2: Antardasha（次级大运）只为当前大运计算→改为全部9个大运
- v3.7.2: Moon Chesta Bala溢出（>60分上限）、Exalted D1分数、Paksha Bala归一化

---
> Source: [732642856/yinduzhanxing](https://github.com/732642856/yinduzhanxing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
