## west-resource

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Concourse CI resource for fetching Zephyr RTOS sources using the West meta-tool. The resource eliminates the need for jobs to run `west init` and `west update` every time by providing pre-fetched Zephyr workspace sources.

## Architecture

The resource follows the standard Concourse resource interface with three scripts:

- `assets/check`: Detects new commits in the Zephyr manifest repository using `git ls-remote`
- `assets/in`: Initializes west workspace and fetches all repositories using `west init` and `west update`
- `assets/out`: No-op script that returns input version (read-only resource)

The Docker image is based on `zephyrprojectrtos/ci:latest` which includes west and all necessary dependencies.

## Common Development Tasks

### Building the resource
```bash
docker build -t west-resource .
```

### Testing locally
```bash
# Test check script
echo '{"source": {"manifest_url": "https://github.com/zephyrproject-rtos/zephyr", "manifest_revision": "main"}}' | docker run -i west-resource /opt/resource/check

# Test in script
docker run -v /tmp/test:/tmp/test west-resource /opt/resource/in /tmp/test < input.json
```

### Configuration Parameters

- `manifest_url`: URL of west manifest repository (default: zephyr main repo)
- `manifest_revision`: Branch/tag to track (default: main)  
- `manifest_path`: Path to west.yml in manifest repo (default: west.yml)

## Integration Notes

This resource is designed for Concourse CI pipelines that need Zephyr sources without the overhead of running west commands in every job. Jobs can use the fetched workspace directly for building, testing, or other operations.

See `example-pipeline.yml` for usage examples.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anobli)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/anobli)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
