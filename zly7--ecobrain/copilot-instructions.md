## ecobrain

> 在添加新功能的时候需要反复阅读当前的代码，谨慎思考新的功能和旧的功能之间能否有替代性，就是如何新的功能可以覆盖旧的功能，那就删掉旧的功能的接口，从而减少代码量和维护成本。如果不能替代，那么就需要考虑新功能的代码风格和旧功能是否一致，是否符合当前项目的整体设计思路。当一个旧的功能被弃用的时候，主动删掉，不要保持向后兼容，任何情况下不应该考虑向后兼容，因为这个不是一个当前已经有很多用户的项目。

# 书写新代码规范
在添加新功能的时候需要反复阅读当前的代码，谨慎思考新的功能和旧的功能之间能否有替代性，就是如何新的功能可以覆盖旧的功能，那就删掉旧的功能的接口，从而减少代码量和维护成本。如果不能替代，那么就需要考虑新功能的代码风格和旧功能是否一致，是否符合当前项目的整体设计思路。当一个旧的功能被弃用的时候，主动删掉，不要保持向后兼容，任何情况下不应该考虑向后兼容，因为这个不是一个当前已经有很多用户的项目。
# 书写新代码规范-2
尽量不要有备用逻辑，这个是一个科研项目，不是一个工程项目，备用逻辑是不被允许的，需要直接用最新的逻辑。存在备用逻辑直接删除。

# Python Environment Configuration

## Python Executable Path

Use the following Python environment for running all scripts:

```bash
/d/environment/miniconda/python.exe
```

## Running Scripts

Example:
```bash
/d/environment/miniconda/python.exe visual.py
```

## Environment Info

- Python Version: 3.12
- Location: /d/environment/miniconda
- Type: Anaconda Distribution

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zly7)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zly7)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
