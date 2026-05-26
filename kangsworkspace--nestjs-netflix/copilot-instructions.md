## main

> NestJS 기반 Netflix 클론 프로젝트로, 웹소켓, 큐 시스템, 파일 업로드 등의 기능을 포함합니다.


# Cursor Rules for NestJS-Netflix Project

## 프로젝트 개요
NestJS 기반 Netflix 클론 프로젝트로, 웹소켓, 큐 시스템, 파일 업로드 등의 기능을 포함합니다.

## 주요 기술 스택
- **Backend**: NestJS, TypeScript
- **Database**: PostgreSQL, Redis
- **Queue**: BullMQ (Redis 기반)
- **WebSocket**: Socket.io
- **File Processing**: FFmpeg
- **Testing**: Jest

## 코드 스타일 가이드
- TypeScript strict 모드 사용
- NestJS 컨벤션 준수
- Entity, DTO, Service, Controller 구조 유지
- 적절한 데코레이터 사용
- 에러 핸들링 필수

## 특별 규칙
- 노션 MCP 작업 시 `.cursor/rules/notion.md` 파일 참조
- 파일 업로드 시 적절한 validation 적용
- WebSocket 연결 시 인증 처리 필수
- Queue 작업 시 적절한 에러 핸들링 및 재시도 로직 구현
# Cursor Rules for NestJS-Netflix Project

## 프로젝트 개요
NestJS 기반 Netflix 클론 프로젝트로, 웹소켓, 큐 시스템, 파일 업로드 등의 기능을 포함합니다.

## 주요 기술 스택
- **Backend**: NestJS, TypeScript
- **Database**: PostgreSQL, Redis
- **Queue**: BullMQ (Redis 기반)
- **WebSocket**: Socket.io
- **File Processing**: FFmpeg
- **Testing**: Jest

## 코드 스타일 가이드
- TypeScript strict 모드 사용
- NestJS 컨벤션 준수
- Entity, DTO, Service, Controller 구조 유지
- 적절한 데코레이터 사용
- 에러 핸들링 필수

## 특별 규칙
- 노션 MCP 작업 시 `.cursor/rules/notion.md` 파일 참조
- 파일 업로드 시 적절한 validation 적용
- WebSocket 연결 시 인증 처리 필수
- Queue 작업 시 적절한 에러 핸들링 및 재시도 로직 구현

---
> Source: [kangsworkspace/NestJS-Netflix](https://github.com/kangsworkspace/NestJS-Netflix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
