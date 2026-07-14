## skill2

> Skill2 = Superpowers-style Skill Library。六个 Skills 管理其他 Skill Library。Skills 是产品；Python CLI 是确定性脚手架。

# AGENTS.md

## Product

Skill2 = Superpowers-style Skill Library。六个 Skills 管理其他 Skill Library。Skills 是产品；Python CLI 是确定性脚手架。

## Invariants

- `skills/<name>/SKILL.md` 单一行为来源。
- `description` 写触发，不写流程摘要。
- Agent 判断；CLI 做 scan/test/package/usage/report。
- Usage 本地；不输出 prompt、transcript、绝对路径。
- Suggest/Visualize 只读。
- Package 无远端副作用。
- Publish 远端动作需 dry-run + 用户确认。
- Skill2 必须符合自己教的规则。

## Verify

```bash
PYTHONPATH=src uv run python -m unittest discover -s tests
uv run ruff check .
uv run skill2 lint skills
uv run skill2 package-check .
```

真实测试：

```bash
skill2 test skills/<name> --cases cases/<name>.yaml --trials 1 --baseline
skill2 test skills/skill2-create --cases cases/skill2-routing.yaml --pack --trials 1
```

长跑必须用 checkpoint/resume/early-stop。不要重跑已完成 trial。

## Fixtures

- 真实 dogfood：`~/workspace/my-agent-config/skills`
- Test fixtures：`tests/fixtures/`
- 运行证据：`.skill2/`；不提交。

## Release

先 `package-check`、`publish-check`、clean install。tag、push、GitHub Release、PyPI upload 前再次获取用户确认。

---
> Source: [MisterBrookT/skill2](https://github.com/MisterBrookT/skill2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
