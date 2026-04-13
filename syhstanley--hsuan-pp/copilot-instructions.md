## hsuan-pp

> 這個 repo 是要教瑄學 NYCU CS 平行程式設計課程的作業內容。

# hsuan_pp — 瑄的平行程式設計教學區

這個 repo 是要教瑄學 NYCU CS 平行程式設計課程的作業內容。

## 學習重點

1. **Bottleneck 分析** — 找出程式效能瓶頸，了解哪裡拖慢了執行速度

   ### 火焰圖（Flame Graph）教學

   **標準火焰圖**（x 軸為字母排序，看 hot path）：
   ```bash
   # 錄製
   perf record -g ./your_program

   # 轉成火焰圖
   perf script > out.perf
   git clone https://github.com/brendangregg/FlameGraph
   ./FlameGraph/stackcollapse-perf.pl out.perf | ./FlameGraph/flamegraph.pl > flamegraph.svg
   ```

   **時序火焰圖**（x 軸為時間軸，看執行順序與熱區分布）：
   ```bash
   # 錄製（加上 --call-graph dwarf 可以抓到更完整 stack）
   perf record -g --call-graph dwarf ./your_program

   # 匯出
   perf script > out.perf
   ```
   然後把 `out.perf` 拖進 👉 **https://profiler.firefox.com**
   - 可以看 flame chart（時間軸）
   - 可以看 call tree、stack chart
   - 支援縮放、filter function name

   > final project 的 `perf.data` 可以直接用 `perf script > out.perf` 匯出來試試看
2. **Thread + Mutex** — 多執行緒基礎，race condition 處理與鎖的使用
3. **OpenMP** — 共享記憶體平行化，pragma 指令加速迴圈與區塊
4. **CUDA** — GPU 平行運算，kernel 撰寫與 memory hierarchy 管理
5. **MPI** — 分散式記憶體平行化，跨節點通訊與 collective operations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/syhstanley)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/syhstanley)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
