## faker-cxx

> git submodule update --init --recursive

# faker-cxx

## Local dev environment setup

```bash
git submodule update --init --recursive
cmake -B build -DFAKER_BUILD_TESTING=ON
cmake --build build
ctest --test-dir build
```

---
> Source: [cieslarmichal/faker-cxx](https://github.com/cieslarmichal/faker-cxx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
