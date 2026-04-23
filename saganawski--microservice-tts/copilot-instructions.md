## microservice-tts

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a microservice-based text-to-speech (TTS) system built with AWS CDK, Java 21, and Maven. The system processes document uploads through a series of Lambda functions orchestrated by S3 events, transforming text files into audio output via TTS services.

## Architecture

The system consists of two main CDK stacks:

### FileFlowStack
- **OriginalFileBucket**: Stores uploaded files (PDF/TXT)
- **ChunkFileBucket**: Stores text chunks split from original files  
- **ProcessedFileBucket**: Stores final audio files
- **ValidationLambda**: Validates file uploads and stores in OriginalFileBucket
- **TransformLambda**: Downloads files from OriginalFileBucket, splits into 4096-char chunks for TTS API limits
- **TTSLambda**: Converts text chunks to audio (placeholder implementation)

### ApiStack  
- **REST API**: Provides `/file-upload` POST endpoint
- **CloudWatch Integration**: Comprehensive logging with custom access log format
- **IAM Roles**: Proper permissions for API Gateway CloudWatch logging

## Lambda Flow
1. File uploaded via API Gateway → ValidationLambda
2. File stored in OriginalFileBucket → triggers TransformLambda  
3. Chunks stored in ChunkFileBucket → triggers TTSLambda
4. Audio files stored in ProcessedFileBucket

## Development Commands

### Build and Package
```bash
# Build entire project
mvn package

# Build specific lambda
mvn -f lambdas/file-validation-lambda/pom.xml package
```

### CDK Operations
```bash
# List all stacks
cdk ls

# Synthesize CloudFormation templates
cdk synth

# Deploy stacks
cdk deploy --all

# Show differences from deployed stacks
cdk diff
```

### Testing
```bash
# Run tests
mvn test

# Run tests for specific module
mvn -f cdk/pom.xml test
```

## Project Structure

- **Root pom.xml**: Multi-module Maven project with Java 21
- **cdk/**: CDK infrastructure code (TtsApp main class)
- **lambdas/**: Individual Lambda function modules
  - file-validation-lambda: Handles file uploads, validates PDF/TXT files
  - file-transform-lambda: Downloads and chunks files (incomplete implementation)
  - file-tts-lambda: TTS conversion (placeholder)
  - notification-lambda: (exists but not wired in stacks)

## Key Implementation Details

- **Java 21** runtime across all components
- **AWS SDK v2** for S3 operations
- **Apache Commons FileUpload2** for multipart file handling in ValidationLambda
- **Account Number**: Hardcoded as `272765753210` in bucket names (FileFlowStack:23)
- **TTS Chunk Limit**: 4096 characters per OpenAI TTS API requirements (TransformLambda:57)
- **Lambda Timeout**: 5 minutes for all functions
- **S3 Event Triggers**: Automatic processing pipeline via S3 notifications

## Current Implementation Status

- ✅ ValidationLambda: Complete file upload and validation
- 🚧 TransformLambda: File download implemented, chunking/upload incomplete  
- 🚧 TTSLambda: Placeholder implementation only
- ❓ NotificationLambda: Exists but not integrated

## Notes

- TransformLambda comment indicates potential switch to Python for TTS functionality
- No external TTS service integration implemented yet
- Bucket versioning disabled across all S3 buckets

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saganawski) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
