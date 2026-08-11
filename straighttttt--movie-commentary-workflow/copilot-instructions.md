## movie-commentary-workflow

> 1. 所有新电影、现有电影接管和跨窗口续做都先使用 `movie-master-director`。

# Movie Commentary Workflow

## 工作方式

1. 所有新电影、现有电影接管和跨窗口续做都先使用 `movie-master-director`。
2. 当前窗口是唯一总导演主脑；阶段技能只改变工作模式，不自动创建固定岗位任务。
3. 只有用户明确要求并行候选时，才使用 `movie-batch-director`；候选只能写入隔离目录。
4. `production/state.json` 是正式采用状态的事实源。文件存在不等于采用，只有锁定记录有效。
5. `DIRECTOR_MEMORY.md` 只保存跨窗口仍需要的剧情理解、用户审美、认可样片和失败教训。
6. 开始任何阶段前先核对当前状态、输入路径和指纹，只继续未完成部分。

## 创作原则

- 先理解完整剧情和结局，再使用索引和模型召回素材。
- 关键动作、对白、道具交接、身份变化和反转必须查看带声音的连续原片。
- 按约 8～15 秒的叙事节拍组织画面，不按标点机械换镜。
- A级关键事实必须准确；B级心理与C级过渡按自然理解和整体观感判断。
- 所有观众可见的画面变化都算切；后期新建拼接不得制造小于 1 秒的最终可见画面。
- 样片基础技术通过并由总导演完整观看后，最多集中修正一次，随后交给用户。
- 没有用户样片认可，不扩展或锁定正式全片。

## 公开仓库边界

- 不得把真实原片、字幕、剧照、证据帧、成片、旁白、声音参考或模型权重写入仓库。
- 不得提交 `.env`、密钥、音色私有编号、个人绝对路径或真实项目状态。
- 示例必须使用合成、公共领域或明确获授权的素材。
- 修改后运行 `python scripts/check_repository.py` 和 `python -m pytest`。

---
> Source: [straighttttt/movie-commentary-workflow](https://github.com/straighttttt/movie-commentary-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
