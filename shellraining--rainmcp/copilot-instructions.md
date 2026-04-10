## rainmcp

> 1. 你可以使用一切命令行工具来完成任务，并且我还很鼓励这样做。但是你需要记住一点，对于不幂等的操作或者无法撤销的命令，你需要提前备份，或者是经过我的允许才可以执行。

## 工具使用规则

1. 你可以使用一切命令行工具来完成任务，并且我还很鼓励这样做。但是你需要记住一点，对于不幂等的操作或者无法撤销的命令，你需要提前备份，或者是经过我的允许才可以执行。
2. 你被鼓励使用各种工具来从网络获取更多信息
   - 优先考虑官方网页，其次是知名博主写的经验文章，不要从 CSDN 或者各种内容农场获取信息
   - 禁止使用内置的 fetch 和 search 工具，应该使用我们准备的 crawl 工具，使用时可以参考 fetch-and-search skill
   - jina（备用）：仅用于学术搜索（arxiv、ssrn、bibtex）和高级功能（classify、deduplicate、expand_query 等），如果额度受限可以考虑 fetch MCP

## 文风规范

1. 你的文风必须准确精炼，避免使用无关字符，如 emoji。
2. 尽量避免编写额外的文档，如果你认为有必要编写，你**必须**先征得我的同意。
3. 你不可以使用加粗、斜体 markdown 语法规则。
4. 无论用户使用什么语言输入，你都要用中文回复用户

## 编码规范

原则：你的代码可能运行在下面这些复杂的平台和环境：

- Node.js 16-22
- Windows 7-11、Mac、Linux(CentOS、Ubuntu)
- x86、ARM 平台
- 同时打开实例
- 工作区内可能有无限多文件，文件大小也可能非常大
- 用户连续操作一周

因此你可能要注意下面的问题：

1. 不同操作系统的文件路径以及不同的规则（比如 macOS 大小写不敏感，Linux 大小写敏感），路径长度可能有限制
2. IO 环境非常复杂，你必须注意下面的情况：
   1. 假设要读写的文件可能不存在、可能无比巨大、可能是个软链接、可能编码不是 UTF-8
   2. 假设要监听、读取的目录下文件无限多、别的软件可能也会修改。尤其是 `.git`、`node_modules` 等
   3. 合并写操作，合并多个小文件操作，减少磁盘 I/O。
   4. 用 stream 记得关闭流
   5. 写文件一定要考虑写竞争问题
   6. 下载的文件默认没有执行权限
   7. 磁盘大小有限，不要无限写
   8. 用户当前的版本不一定是上一个版本，可能跨版本升级。持久存储要做好数据兼容

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shellRaining)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shellRaining)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
