## database-bonus-pj

> 本项目是复旦大学数据库相关课程的 Bonus Lab：事务系统的性能优化。

# AGENTS.md

## 项目身份

本项目是复旦大学数据库相关课程的 Bonus Lab：事务系统的性能优化。

组号：34

实验仓库：

```text
https://github.com/DASLab-IDA/Bonus_Lab_About_TP_Performance_Optimization
```

最终报告文件名：

```text
34-BonusLab.pdf
```

最终压缩包文件名：

```text
34-BonusLab.zip
```

提交内容：

```text
代码文件夹 + 34-BonusLab.pdf
```

然后整体打包为：

```text
34-BonusLab.zip
```

截止时间：

```text
6 月 28 日 23:59
```

实验报告需要简单介绍：

```text
优化方法
优化思路
实验结果
性能提升
参考资料
```

课程要求中明确说明：完成实验内容与要求中的索引部分实验并得到性能提升，可获得该实验 100% 分数；在此基础上继续优化，会根据性能提升获得额外加分。

## 当前状态

当前实验仓库已经 clone 完成。

Agent 接手时，应默认自己已经在仓库目录内，或者需要先进入已经 clone 好的仓库目录。

不要重复 clone 仓库，除非当前目录明显不是实验仓库，或者用户明确要求重新 clone。

Agent 接手后的任务是：

```text
确认仓库状态
阅读 README.md
阅读 AGENTS.md
安装依赖
编译项目
跑通 baseline
分析瓶颈
实现索引优化
重新 benchmark
可选实现额外优化
生成报告
打包提交
```

## 初始化检查

首先确认当前目录是否是实验仓库根目录：

```bash
pwd
ls
```

应能看到类似内容：

```text
README.md
CMakeLists.txt
src/
scripts/
```

如果当前不在仓库根目录，先进入仓库目录，例如：

```bash
cd Bonus_Lab_About_TP_Performance_Optimization
```

然后阅读任务信息：

```bash
cat README.md
cat AGENTS.md
```

## 环境与编译

安装依赖：

```bash
sudo apt update
sudo apt install -y build-essential cmake flex bison git
```

编译项目：

```bash
cmake -B build
cmake --build build --target rmdb tpcc_loader tpcc_driver -j$(nproc)
```

运行实验脚本：

```bash
bash scripts/01_setup.sh
bash scripts/02_run_s10_t4_d60.sh
```

如果时间允许，再运行长时间 benchmark：

```bash
bash scripts/03_run_s1_t4_d300.sh
```

注意：

```text
停止数据库 server 时使用 kill -INT。
不要使用 kill -9。
```

因为 `kill -9` 可能导致脏页和元数据没有正常落盘，从而让数据库状态损坏。

## 实验目标

本实验基于一个 RucBase 派生的教学事务系统。该系统已经可以正确运行 TPC-C benchmark，但故意保留了多个性能瓶颈。

本项目的目标不是从零重写数据库，而是在保留原框架的基础上完成性能优化：

```text
定位瓶颈
提出优化
实现优化
量化收益
写清楚实验报告
```

评分重点是索引优化。因此，必须优先完成：

```text
让可以通过索引完成的点查不再走 SeqScan，而是走 IndexScan。
```

在索引优化完成并确认性能提升后，再考虑其他附加优化。

## 当前系统的主要性能问题

当前系统的主要瓶颈包括：

```text
B1：点查强制走 SeqScan
B2：表级 S/X 锁粒度过粗
B3：wait-die 导致年轻事务大量 abort
B4：读后写路径容易发生锁升级冲突
B5：缺少更高级的 WAL / undo / MVCC 等机制
```

其中，最优先解决的是：

```text
点查强制走 SeqScan
```

## 本项目的核心思路

本项目采用如下优化主线：

```text
索引引导的锁冲突缩短优化
Index-guided Lock Conflict Reduction
```

核心观点：

```text
索引优化不仅能减少全表扫描成本，还能缩短事务持有表级锁的时间。
```

在当前系统中，读操作会通过 `SeqScanExecutor` 获取表级 S 锁，写操作会获取表级 X 锁，并且锁会一直持有到 commit 或 abort。由于系统使用严格 2PL 和 wait-die，长时间的 SeqScan 会扩大锁冲突窗口，使年轻事务更容易 abort。

因此，索引优化的收益有两层：

```text
第一层：减少 tuple 扫描和 page 访问，提高单次查询速度。
第二层：缩短表锁持有时间，降低事务冲突概率，改善 abort rate 和尾延迟。
```

报告中需要强调：

```text
SeqScan -> 长时间持有表级 S 锁 -> 与表级 X 锁冲突 -> wait-die 下年轻事务 abort -> TPS / tpmC 下降，p95 / p99 延迟恶化
```

索引优化之后：

```text
IndexScan -> 点查更快 -> 持锁窗口更短 -> 冲突概率下降 -> 吞吐和尾延迟改善
```

## 必须完成的任务

### 任务 1：确认仓库并跑通 baseline

先确认当前在仓库根目录：

```bash
pwd
ls
```

然后安装依赖并编译：

```bash
sudo apt update
sudo apt install -y build-essential cmake flex bison git

cmake -B build
cmake --build build --target rmdb tpcc_loader tpcc_driver -j$(nproc)
```

接着运行：

```bash
bash scripts/01_setup.sh
bash scripts/02_run_s10_t4_d60.sh
```

如果时间允许，再运行：

```bash
bash scripts/03_run_s1_t4_d300.sh
```

记录 baseline 指标：

```text
committed transactions
aborted transactions
abort rate
TPS
tpmC
latency p50
latency p95
latency p99
```

报告中必须写明实验环境：

```text
操作系统
CPU
内存
编译器版本
测试参数
```

### 任务 2：分析 TPC-C 查询模式

重点查看：

```text
src/test/tpcc/tpcc_driver.cpp
src/test/tpcc/tpcc_loader.cpp
src/test/tpcc/tpcc_common.h
```

需要找出 TPC-C 五类事务中的高频点查模式：

```text
NewOrder
Payment
OrderStatus
Delivery
StockLevel
```

重点关注这些表：

```text
warehouse
district
customer
stock
orders
new_orders
order_line
item
```

高价值索引候选：

```text
warehouse:    w_id
district:     d_w_id, d_id
customer:     c_w_id, c_d_id, c_id
stock:        s_w_id, s_i_id
orders:       o_w_id, o_d_id, o_id
new_orders:   no_w_id, no_d_id, no_o_id
order_line:   ol_w_id, ol_d_id, ol_o_id
item:         i_id
```

最优先关注：

```text
stock
customer
district
warehouse
item
```

因为 NewOrder 和 Payment 高频访问这些表。

### 任务 3：实现索引优化

目标：

```text
让可以通过索引完成的点查不再走 SeqScan，而是走 IndexScan。
```

重点查看代码目录：

```text
src/index/
src/execution/
src/optimizer/
src/system/
src/test/tpcc/
```

推荐实现方式：

```text
1. 确认或补充 TPC-C 关键表上的索引创建逻辑。
2. 确认 loader 是否已经在建表时创建索引。
3. 若未创建索引，则在 tpcc_loader.cpp 中补充 CREATE INDEX 或等价逻辑。
4. 修改 optimizer 或 planner，使等值谓词命中索引时生成 IndexScan plan。
5. 修改或补全 IndexScanExecutor，使其可以根据索引 key 定位 record。
6. 保留 fallback：没有合适索引时继续使用 SeqScan。
```

不要写成只匹配某一个 SQL 字符串的 hard-code。

可以接受的最小通用规则：

```text
对于单表 selection，如果 WHERE 条件中的等值谓词覆盖某个索引 key，
则使用该索引生成 IndexScan；
否则使用原有 SeqScan。
```

实现时优先保证：

```text
正确性
可解释性
稳定可复现
性能确实提升
```

不要为了跑分删除事务、锁、abort、rollback 等逻辑。

### 任务 4：重新 benchmark

索引优化完成后，重新运行与 baseline 相同的 benchmark。

至少比较：

```text
baseline
index-only
```

记录并报告：

```text
TPS
tpmC
abort rate
p50 latency
p95 latency
p99 latency
```

报告中需要说明：

```text
索引优化前后访问路径的变化
哪些表建立了索引
哪些事务受益最大
性能提升幅度
是否影响 abort rate
是否改善尾延迟
```

## 可选加分任务

在索引优化完成并稳定后，可以继续做一个小型并发优化。

推荐方向：

```text
写意图锁优化
Write-intent locking for read-before-write paths
```

问题：

```text
当前系统中，某些事务会先 SELECT 再 UPDATE。
这会导致先申请 S 锁，再升级为 X 锁。
在 wait-die 机制下，锁升级冲突可能导致事务直接 abort。
```

典型场景：

```text
Payment 会更新 warehouse、district、customer。
NewOrder 会更新 district、stock，并插入 orders、new_orders、order_line。
Delivery 会更新 new_orders、orders、order_line、customer。
```

优化思路：

```text
如果某个事务路径已经确定后续会写某张表，
则可以提前申请 X 锁，而不是先申请 S 锁再升级。
```

注意：

```text
不要对所有 SELECT 都无脑申请 X 锁。
只对 TPC-C 中明确存在 read-before-write 的路径做保守优化。
```

预期收益：

```text
减少 S -> X 锁升级冲突
降低 wait-die abort
改善 p95 / p99 延迟
提升 tpmC
```

如果实现该优化，报告中的对比应为：

```text
baseline
index-only
index + write-intent locking
```

## 不建议做的方向

除非时间非常充足，否则不要做：

```text
完整 MVCC
完整 OCC
完整 WAL
重写事务管理器
重写存储引擎
重写 SQL parser
重写整个 executor
```

这些方向工程量大、风险高、很容易影响正确性。

本项目应优先追求：

```text
小改动
可复现
可解释
有明确性能提升
```

## 正确性要求

每次修改后都必须确认：

```text
项目可以成功编译
tpcc_loader 可以成功灌库
tpcc_driver 可以正常运行
事务不会崩溃
事务不会静默跳过必要操作
无索引时仍然可以回退到 SeqScan
不要移除锁、事务、commit、abort、rollback 逻辑
```

如果某个优化提升性能但破坏正确性，应当撤回。

## 推荐报告结构

报告文件名：

```text
34-BonusLab.pdf
```

推荐结构：

```text
1. 实验背景与目标
2. 实验环境
3. Baseline 复现
4. 瓶颈分析
5. 优化一：索引引导的点查优化
6. 优化二：写意图锁优化，若实现
7. 实验结果与性能对比
8. 总结与局限
9. 参考资料
```

瓶颈分析部分重点写：

```text
当前系统强制使用 SeqScan。
SeqScan 不仅导致扫描开销大，也会延长表级 S 锁持有时间。
在严格 2PL + wait-die 下，长时间持锁会放大事务冲突，使年轻事务更容易 abort。
因此，索引优化不仅是访问路径优化，也是并发冲突窗口优化。
```

实验结果表格建议：

```text
方案 | TPS | tpmC | abort rate | p50 | p95 | p99
baseline | ... | ... | ... | ... | ... | ...
index-only | ... | ... | ... | ... | ... | ...
index + write-intent | ... | ... | ... | ... | ... | ...
```

总结部分可以写：

```text
本项目没有试图将系统改造成完整工业级事务数据库，而是在保留原有表级锁与 wait-die 框架的前提下，通过索引化点查缩短扫描路径和锁持有窗口，从而提升 TPC-C 吞吐并改善尾延迟。进一步地，针对读后写事务的写意图锁优化可以减少锁升级带来的无谓 abort。
```

## 代码风格要求

遵循仓库原有代码风格。

修改尽量局部化。

变量名应表达数据库语义，例如：

```text
matched_index
index_key_values
try_index_scan
write_intent_tables
lock_upgrade_avoidance
```

避免使用含糊或临时性的名字，例如：

```text
fast_mode
hack
magic
special_case_1
tmp_opt
```

注释应解释非显然的优化逻辑，不要给显然代码写过多注释。

## Agent 执行顺序

Agent 在接手该项目时，应按如下顺序工作：

```text
1. 确认当前目录是否是实验仓库根目录。
2. 阅读 README.md，确认实验目标、运行方式和参考瓶颈。
3. 阅读本 AGENTS.md，确认组号、提交要求和优化路线。
4. 安装依赖并编译项目。
5. 运行 scripts/01_setup.sh。
6. 运行 scripts/02_run_s10_t4_d60.sh，记录 baseline。
7. 阅读 tpcc_driver.cpp，整理 TPC-C 查询模式。
8. 阅读 tpcc_loader.cpp，确认索引创建情况。
9. 阅读 optimizer / executor / index 相关代码，找出 SeqScan 替换为 IndexScan 的最小修改点。
10. 实现索引优化。
11. 编译并运行 benchmark。
12. 对比 baseline 与 index-only。
13. 若时间允许，实现 write-intent locking。
14. 再次运行 benchmark。
15. 整理实验结果。
16. 生成 34-BonusLab.pdf。
17. 打包代码文件夹和报告为 34-BonusLab.zip。
```

## 最终交付检查清单

提交前检查：

```text
[ ] 当前目录是正确实验仓库
[ ] 代码可以编译
[ ] tpcc_loader 可以运行
[ ] tpcc_driver 可以运行
[ ] baseline 数据已记录
[ ] index-only 数据已记录
[ ] 若有额外优化，额外优化数据已记录
[ ] 报告文件命名为 34-BonusLab.pdf
[ ] ZIP 文件命名为 34-BonusLab.zip
[ ] ZIP 中包含代码文件夹和报告 PDF
[ ] 报告写明优化方法、优化思路、实验结果和参考资料
```

---
> Source: [VagRant2333/DataBase-Bonus-PJ](https://github.com/VagRant2333/DataBase-Bonus-PJ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
