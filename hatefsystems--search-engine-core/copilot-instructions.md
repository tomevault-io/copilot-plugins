## search-engine-core

> Pre-submit checklist and common issues


# Code Review Checklist

Before submitting:

- [ ] Every `onData` paired with `onAborted`
- [ ] Controllers use lazy init (no service init in constructor)
- [ ] Use `LOG_DEBUG()` not `std::cout`; respect LOG_LEVEL
- [ ] MongoDB: basic builder + `.extract()` for complex updates
- [ ] MongoDB instance initialized where needed
- [ ] Error handling and logging in place
- [ ] CMakeLists.txt updated if adding libs
- [ ] No memory leaks; thread-safe where required

## Common Issues

| Symptom | Cause | Fix |
|--------|--------|-----|
| Empty reply / POST crash | Missing onAborted | Pair with onData |
| MongoDB crash | Instance not initialized | MongoDBInstance::getInstance() first |
| Static init crash | Services in controller ctor | Lazy init with getters |
| Document loses fields | Stream builder / no extract | Basic builder + .extract() |
| Undefined reference | Missing lib | Add to target_link_libraries |
| Collection not found | Inconsistent collection names | Use same name in code and imports |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hatefsystems)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hatefsystems)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
