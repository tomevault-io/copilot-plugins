## qwen3-tts-on-aws

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AWS CDK (TypeScript) project that deploys Qwen3 TTS (Text-to-Speech) on an EC2 GPU instance. The stack provisions a VPC, security group, IAM role, and EC2 instance with a UserData script that installs dependencies and runs the TTS web interface as a systemd service.

## Commands

```bash
npm run build          # TypeScript compile
npm run test           # Run Jest tests
npx cdk synth          # Generate CloudFormation template (validates stack)
npx cdk diff           # Show pending infrastructure changes
npx cdk deploy         # Deploy stack (requires: export $(cat .env | xargs))
npx cdk destroy        # Tear down stack
```

Before `cdk deploy`/`synth`, load env vars: `export $(cat .env | xargs)`

## Architecture

```
bin/qwen3-tts-cdk-init.ts       → CDK app entry point, loads config from env vars
lib/config.ts                    → Qwen3TtsConfig interface + loadConfigFromEnv()
lib/qwen3-tts-stack.ts           → Main stack: VPC, SecurityGroup, IAM Role, EC2 Instance, S3 Assets
scripts/server.py                → FastAPI + Gradio Web UI サーバー (S3 Asset として転送)
scripts/generate-voices.py       → VoiceDesign ボイス生成スクリプト
scripts/voice-design-clone.py    → VoiceDesign→VoiceClone フルワークフロー
scripts/config.json              → デフォルトキャラクター定義
```

- **Config flow**: Environment variables → `loadConfigFromEnv()` → `Qwen3TtsConfig` → `Qwen3TtsStackProps` → stack resources
- **UserData**: CDK が `addCommands()` + `addS3DownloadCommand()` でプログラム的に構築。Python ファイルは S3 Asset として転送し、16KB 制限を回避
- **設定値の受け渡し**: Python スクリプトは環境変数 (`QWEN3_TTS_MODEL`, `QWEN3_TTS_PORT`) から設定を取得。systemd サービスで環境変数を設定
- **AMI**: Uses Deep Learning Base AMI (Ubuntu 22.04) with pre-installed NVIDIA drivers/CUDA, hardcoded AMI ID per region in `qwen3-tts-stack.ts`
- **Network**: Public subnet with direct internet access for model downloads from Hugging Face

## Key Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `INSTANCE_TYPE` | `g4dn.xlarge` | EC2 GPU instance type |
| `WEB_PORT` | `8000` | Gradio web UI port |
| `MODEL_NAME` | `Qwen/Qwen3-TTS-12Hz-1.7B-Base` | HuggingFace model |
| `KEY_PAIR_NAME` | (none) | SSH key pair name |
| `VPC_ID` | (none) | Use existing VPC |

## Language & Conventions

- TypeScript with strict mode, target ES2022, module NodeNext
- Japanese comments throughout the codebase
- CDK v2 (`aws-cdk-lib` single package)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nunocky)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Nunocky)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
