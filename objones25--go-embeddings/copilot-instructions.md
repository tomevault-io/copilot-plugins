## go-embeddings

> scripts/*.py = moderate

############ Project Rules ############
[rules]
# Go code rules
go.mod = moderate
go.sum = moderate
*.go = high

# Python scripts
scripts/*.py = moderate
scripts/*.sh = moderate

# Configuration files
*.yml = moderate
*.yaml = moderate
*.json = moderate
Makefile = moderate
.env* = moderate

# Documentation
*.md = low
LICENSE = low

# Test files
test/**/*_test.go = high
testdata/* = low

############ Directory Rules ############
[dirs]
cmd/ = high
pkg/ = high
internal/ = high
test/ = moderate
scripts/ = moderate
configs/ = moderate

############ Ignore Rules ############
[ignore]
# Binary and generated files
*.dylib
*.a
*.so
*.exe
*.test
*.prof

# Build and cache directories
build/
dist/
tmp/
.cache/

# Dependencies
vendor/
libs/
onnxruntime*/

# Model files
models/
*.onnx
*.bin
*.safetensors

# IDE and system files
.idea/
.vscode/
.DS_Store
*.swp

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/objones25) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
