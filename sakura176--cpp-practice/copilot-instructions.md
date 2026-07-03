## cpp-practice

> C++ RAII、智能指针、并发编程练习项目。使用 CMake + Google Test 构建。

# C++ Practice Project

C++ RAII、智能指针、并发编程练习项目。使用 CMake + Google Test 构建。

## Build & Test

```bash
cd build && cmake .. -DCMAKE_BUILD_TYPE=Debug && make <target> && ./week2/<target>
```

常用 target: `day03_unique_ptr_test` `day04_shared_ptr_test` `day05_linked_list_test`

## Code Review (评审)

当我说"评审 某个任务"（如"评审 week02/day05"或"评审 day05"）时，请按以下流程执行：

1. **读取代码**: 阅读该 task 的所有源文件（.h / .cpp / _test.cpp）
2. **编译运行**: 确保构建通过、测试运行，记录测试结果
3. **深入评审**: 以资深 C++ 工程师和面试官的双重视角分析代码：
   - 正确性：有无 UB、内存错误、逻辑 bug
   - 架构设计：RAII、所有权、异常安全、接口设计
   - C++ 现代特性：移动语义、完美转发、noexcept、constexpr 等
   - 代码风格：可读性、一致性、工程细节
   - 测试质量：覆盖率、边界情况、编译期检查
4. **产出评审文档**: 写入 `docs/<task>_code_review.md`，包含：
   - 逐项问题列表（标注严重度：严重/中等/低）
   - 正确的代码示例/修复方案
   - 面试扩展话题
   - 总结表格
5. **添加代码注释**: 在源文件中 key 位置以 `【BUG】` / `【设计】` / `NOTE` 标注评审发现

## Project Structure

- `week2/day0*_<name>.h` — 模板类实现（unique_ptr, shared_ptr, linked_list）
- `week2/day0*_<name>_test.cpp` — GTest 测试
- `docs/` — 代码评审文档
- 编译启用 ThreadSanitizer (`-fsanitize=thread`)

---
> Source: [Sakura176/cpp_practice](https://github.com/Sakura176/cpp_practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
