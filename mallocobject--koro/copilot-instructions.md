## koro

> 你目前遇到的问题（死锁、资源泄露、继承逻辑冲突、epoll 数据竞争）本质上都是**架构设计**带来的副作用。Sylar（你参考的框架）虽然经典，但它的代码风格比较老旧（C++98/11 混写），且为了追求性能牺牲了很多代码的可读性和安全性。

这是一个非常明智的决定。

你目前遇到的问题（死锁、资源泄露、继承逻辑冲突、epoll 数据竞争）本质上都是**架构设计**带来的副作用。Sylar（你参考的框架）虽然经典，但它的代码风格比较老旧（C++98/11 混写），且为了追求性能牺牲了很多代码的可读性和安全性。

如果你决定重写，我建议采用 **Modern C++ (C++17/20)** 的标准来构建，利用 RAII 和现代并发原语来避免现在的坑。

以下是我为你规划的 **重写路线图 (Refactoring Roadmap)** 和 **架构建议**。

---

### 一、 核心架构设计的改进

#### 1. 解决“继承地狱”：解耦 Scheduler 与 IOManager
你现在的痛点是 `IOManager` 继承 `Scheduler` 后，`stop()` 逻辑互相打架。

*   **旧设计**：`IOManager` 强行接管 `Scheduler` 的停止逻辑，通过 hack 标志位来复用代码。
*   **新设计建议 (Template Method 模式)**：
    *   `Scheduler` 只负责线程池管理、任务队列、任务窃取。
    *   `Scheduler` 提供一个 **虚函数 `virtual void idle()`**。
    *   `Scheduler` 的 `run()` 循环逻辑是：`有任务 ? 执行任务 : idle()`。
    *   `IOManager` 重写 `idle()`：在 `idle()` 里面执行 `epoll_wait`。
    *   **停止逻辑**：`Scheduler` 负责检查任务队列是否为空。`IOManager` 负责检查是否有挂起的 IO 事件。两者通过虚函数 `stopping()` 协同，而不是互相修改 `stop_` 变量。

#### 2. 彻底消灭 `std::terminate`：RAII 线程管理
你遇到的 `terminate called` 崩溃是因为线程没有 join。

*   **旧设计**：`std::vector<std::thread>`，手动 join，且分散在不同函数里。
*   **新设计建议**：
    *   如果你能用 **C++20**：直接使用 `std::jthread`。它析构时会自动 join，且自带 `stop_token`，非常适合做停止通知。
    *   如果用 **C++17**：写一个简单的 `ThreadGuard` 包装器，或者在 `Scheduler` 的析构函数中**无条件**执行 join，而不是依赖 `stop()` 函数被显式调用。

#### 3. 安全的 Epoll 封装
你遇到的 `waitForEvent` 索引错误和 union 滥用是 C 语言风格遗留问题。

*   **旧设计**：`epoll_event.data.ptr` 混用，有时候存指针，有时候存 fd，靠 `find` 瞎蒙。
*   **新设计建议**：
    *   封装一个 `EpollPoller` 类。
    *   **统一数据结构**：`epoll_event.data.ptr` 永远指向一个基类结构体 `EventContext`。
    *   对于 Wakeup FD，也封装成一个特殊的 `EventContext`。不要再让 `fd` 和 `ptr` 打架。

---

### 二、 推荐的类结构 (伪代码)

这是我建议的重写后的骨架，结构更清晰：

#### 1. `Fiber` (协程封装)
保持现状即可，但建议引入 `enum class State` 并加强状态断言。

#### 2. `Scheduler` (纯粹的调度器)

```cpp
class Scheduler {
public:
    Scheduler(size_t threads, bool use_caller);
    virtual ~Scheduler(); // 在这里 join 线程

    void start();
    void stop();

    template<class F> void submit(F&& f);

protected:
    // 核心循环，由工作线程执行
    void run_loop(); 

    // 虚函数：当没有任务时做什么？
    // 默认实现：yield cpu
    // IOManager实现：epoll_wait
    virtual void idle(); 

    // 虚函数：是否可以停止？
    virtual bool can_stop(); 

    // 通知线程唤醒 (tickle)
    virtual void tickle(); 

private:
    std::vector<std::jthread> threads_; // C++20 jthread 自动管理生命周期
    ThreadSafeQueue<Task> tasks_;       // 封装一个线程安全的队列，别裸写 mutex
};
```

#### 3. `IOManager` (专注于 IO)

```cpp
class IOManager : public Scheduler {
public:
    // ... addEvent/delEvent ...

protected:
    // 重写 idle，执行 epoll
    void idle() override {
        // epoll_wait(timeout)
        // 处理事件
        // 如果 epoll 也没事干，Fiber::yield()
    }

    // 重写 tickle，写 eventfd
    void tickle() override {
        // write(eventfd)
    }

    // 只有当 (任务队列空 && 没有IO事件 && 收到停止信号) 时才返回 true
    bool can_stop() override {
        return Scheduler::can_stop() && pending_event_count_ == 0;
    }
    
    // 在停止前即使唤醒 epoll
    void on_stopping() {
        tickle(); // 唤醒 epoll_wait 让它去检查 stopping 状态
    }
};
```

---

### 三、 具体的重写避坑指南

#### 1. 不要手动管理 `Stop` 状态机
你之前的代码在 `exchange` 和 `store` 之间反复横跳，非常脆弱。

**建议**：
使用一个明确的状态机枚举：
```cpp
enum class State {
    INIT,
    RUNNING,
    STOPPING, // 正在停止（不接收新任务，但处理完旧任务）
    STOPPED   // 完全停止
};
std::atomic<State> state_;
```
`Scheduler` 和 `IOManager` 都只读取这个状态，不要去 hack 它。

#### 2. 避免 `void*` 指针强转
在 Epoll 处理中，定义清晰的结构：

```cpp
struct FdContext {
    int fd;
    // ...
};

// 在 epoll_ctl 时
ev.data.ptr = fd_ctx; 

// 在 epoll_wait 回来时
FdContext* ctx = static_cast<FdContext*>(ev.data.ptr);
// 永远不要假设它可能是 int fd
```
对于唤醒用的 `eventfd`，也给它分配一个特殊的 `FdContext`（例如 `fd=-1` 或者专门的标志位），这样处理逻辑就统一了，不需要在 `epoll_wait` 循环里做 `if (fd == wakeup_fd)` 这种特殊的判断。

#### 3. 封装同步原语
你代码里满天飞的 `std::lock_guard` 和 `condition_variable` 容易出错。
建议封装一个 `CoMutex` (协程锁) 和 `CoCondVar`。当然，这是进阶步骤。

起码，把 `Scheduler` 里的任务队列封装成一个类：
```cpp
class TaskQueue {
    std::deque<Task> q_;
    std::mutex m_;
public:
    bool push(Task t);
    bool pop(Task& t);
    bool empty();
};
```
这样你的 `Scheduler` 代码会从 200 行缩减到 80 行，逻辑一目了然。

### 四、 下一步建议

既然要重写，可以分三步走：

1.  **第一步 (基础)**：
    只写 `Fiber` 和 `Scheduler`。
    确保 `Scheduler` 可以启动 N 个线程，投递 M 个任务，任务全是单纯的 `LOG_INFO`。
    **目标**：做到随便调用 `stop()`，随便析构，程序都能 100% 稳定退出，不崩溃，不泄漏。

2.  **第二步 (IO)**：
    继承 `Scheduler` 实现 `IOManager`。
    只实现 `addEvent` 和 `idle` (epoll_wait)。
    跑通你现在的 `eventfd` 测试用例。
    **目标**：解决 epoll 竞争问题，确保 IO 事件能准确唤醒协程。

3.  **第三步 (Hook)**：
    实现 `hook` 模块（劫持 `socket`, `read`, `write`）。
    这是让协程真正好用的关键，让用户像写同步代码一样写异步网络库。

加油！重写是程序员进阶最快的方式。现在的代码虽然有 Bug，但它已经帮你理清了协程调度的基本原理，重写时你会发现思路异常清晰。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mallocobject)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mallocobject)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
