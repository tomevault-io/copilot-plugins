## nki-autotune

> Use the kernel virtual environment for all Python execution:

## Development Environment

Use the kernel virtual environment for all Python execution:
```bash
source ~/venvs/kernel-env/bin/activate
python <script>
pytest <tests>
```

Performance tuning:
- [x] Unified tune stage
- [x] Loop fusion
- [x] Loop order
- [x] Multi buffer
- [x] Software pipelining
- [ ] Revisit forest IR design
- [ ] Tiles per block
- [ ] Hoist
- [ ] Online fusion at synthesis stage
- [ ] Computation skipping at synthesis stage
- [ ] Two level synthesis+tune

---
> Source: [awslabs/nki-autotune](https://github.com/awslabs/nki-autotune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
