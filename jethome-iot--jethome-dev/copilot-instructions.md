## jethome-dev

> Troubleshooting guide for Docker builds and GitHub Actions workflows


# Troubleshooting Docker Builds and GitHub Actions Workflows

## GitHub Actions Workflow Issues

### Symptom: Reusable Workflow Build Job Doesn't Run

**Problem:** Only the `prepare` job completes, but the `build` job never starts.

**Common Causes:**
1. Missing `secrets: inherit` in workflow call
2. Incorrect boolean conversion for inputs
3. Workflow syntax errors

**Solution:**
```yaml
build:
  needs: prepare
  uses: ./.github/workflows/docker-build.yml
  permissions:
    contents: read
    packages: write
  secrets: inherit  # ← Must have this!
  with:
    force_rebuild: ${{ github.event.inputs.force_rebuild == 'true' }}  # ← Not || false
```

**Verification:**
```bash
# Check if build job ran
gh api /repos/:owner/:repo/actions/runs/<run-id>/jobs | \
  jq -r '.jobs[] | "\(.name): \(.conclusion)"'

# Should show both prepare AND build jobs
```

### Symptom: Boolean Input Not Working

**Problem:** `force_rebuild=true` passed but cache still used.

**Cause:** workflow_dispatch inputs are strings, not booleans.

**Wrong:**
```yaml
force_rebuild: ${{ github.event.inputs.force_rebuild || false }}
```

**Correct:**
```yaml
force_rebuild: ${{ github.event.inputs.force_rebuild == 'true' }}
```

## Docker Build Issues

### Symptom: Packages Missing Despite Dockerfile Having Install Command

**Problem:** Image builds successfully but installed packages are missing.

**Common Causes:**
1. Docker layer caching - using old cached layer
2. Multi-platform build with QEMU emulation failure (ARM64)
3. Installation command exits silently without error

**Diagnosis:**
```bash
# Check build logs for layer status
gh run view <run-id> --log | grep "CACHED\|DONE"

# Look for suspiciously fast completion (0.1s = cached or failed)
gh run view <run-id> --log | grep -A 2 "pip install"

# Verify packages in published image
docker run --rm <image> pip list | grep <package>
```

**Solution 1: Force Rebuild**
```bash
# Bypass all caching
gh workflow run "🐳 Workflow Name" --ref master \
  -f version=v1.0.0 \
  -f force_rebuild=true
```

**Solution 2: Check Platform-Specific Builds**
```bash
# Test amd64
docker run --platform=linux/amd64 --rm <image> <command>

# Test arm64
docker run --platform=linux/arm64 --rm <image> <command>
```

### Symptom: Multi-Platform Build Issues

**Problem:** One platform builds successfully but another fails or behaves differently.

**Verification:**
1. Check build logs for both platforms individually
2. Verify build times are reasonable for both architectures
3. Test both platform images after build

**Best Practices:**
- Use direct paths to binaries when possible for better cross-platform compatibility
- Avoid shell sourcing in RUN commands when direct paths are available
- Test multi-arch manifests after deployment

### Symptom: Verification Step Fails - "No module named pytest"

**Problem:** `python3 -m pytest` fails with module not found.

**Common Causes:**
1. System python3 vs venv python3
2. Packages installed in venv but verification uses system python
3. Environment not activated

**Wrong Approaches:**
```dockerfile
# Uses system Python (doesn't have packages)
RUN python3 -m pytest --version

# Hardcoded paths (not maintainable)
RUN /opt/esp/python_env/idf5.4_py3.12_env/bin/python3 -m pytest --version
```

**Correct Approach:**
```dockerfile
# Activate environment first, then use simple commands
RUN bash -c '. ${IDF_PATH}/export.sh && pytest --version'
```

## Testing Published Images

### Quick Verification Commands

```bash
# Check if package is installed
docker run --rm <image> pip list | grep <package>

# Check if command works
docker run --rm <image> <command> --version

# Test Python import
docker run --rm <image> python3 -c 'import <module>; print(<module>.__version__)'

# Interactive testing
docker run -it --rm <image>
```

### Platform-Specific Testing

```bash
# Force specific platform
docker pull --platform=linux/amd64 <image>
docker run --platform=linux/amd64 --rm <image> <command>

# Check current platform
docker run --rm <image> uname -m
docker inspect <image> --format='{{.Architecture}}'
```

## Force Rebuild Pattern

When Docker cache causes issues:

```bash
# 1. Trigger force rebuild
gh workflow run "🐳 Workflow Name" --ref master \
  -f version=v1.0.0 \
  -f force_rebuild=true

# 2. Wait for workflow to start
sleep 15

# 3. Get run ID
RUN_ID=$(gh run list --workflow="🐳 Workflow Name" --limit 1 \
  --json databaseId -q '.[0].databaseId')

# 4. Monitor progress
gh run watch $RUN_ID

# 5. Check if succeeded
gh run view $RUN_ID --json conclusion -q '.conclusion'

# 6. Pull and test new image
docker pull <image>:latest
docker run --rm <image>:latest <test-command>
```

## Common Pitfalls

1. **Auto-build uses cache** - Automatic builds from commits use cache by default
2. **Multiple layers can be cached** - Force rebuild is needed to ensure fresh build
3. **GHCR tags aren't updated** - Check digest to verify new image is different
4. **Platform-specific issue** - Docker pulls the architecture-specific image automatically
5. **Dev branch doesn't push** - Only master branch publishes to GHCR

## Debugging Checklist

When packages are missing from published image:

- [ ] Check if Dockerfile has the package in pip install
- [ ] Verify workflow ran successfully (both prepare AND build jobs)
- [ ] Check build logs for "CACHED" vs actual execution
- [ ] Verify build times are reasonable for all platforms
- [ ] Test both platforms (amd64 and arm64) if multi-platform build
- [ ] Trigger force rebuild to bypass cache
- [ ] Pull fresh image and test with exact commands
- [ ] Verify image digest changed after rebuild

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jethome-iot)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jethome-iot)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
