## jsonparsing

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

C++20 header-only JSON 파싱 & 파일 저장 라이브러리 PoC. Visual Studio 2022 (v145 toolset), Windows x64/Win32.

## Build

Visual Studio에서 `Project1.sln` 또는 `JsonParsing.slnx`를 열고 빌드.  
CLI 빌드:
```
msbuild Project1\Project1.vcxproj /p:Configuration=Debug /p:Platform=x64
```

실행 파일 출력: `Project1\x64\Debug\Project1.exe`  
실행 시 `output.json`이 현재 디렉토리에 생성된다.

## Architecture

모든 구현은 `Project1/` 아래 헤더 온리:

| 파일 | 역할 |
|------|------|
| `JsonValue.h` | 핵심 타입. `std::variant<Null, Bool, Number, String, Array, Object>` 기반. `operator[]`는 Null에서 Object/Array로 자동 초기화. |
| `JsonParser.h` | Recursive-descent 파서. `parse(string_view)` / `parseFile(path)` 정적 메서드. |
| `JsonWriter.h` | 직렬화기. `toString()` / `toFile()`. pretty(들여쓰기 2칸) / compact 옵션. |
| `JsonLib.h` | 세 헤더를 한 번에 포함하는 진입점. |
| `main.cpp` | 5가지 시나리오 데모 및 간단한 assert 검증. |

## Key Conventions

- `JsonValue::Number`는 `double` 단일 타입 (정수도 double로 저장, 직렬화 시 정수면 소수점 없이 출력).
- `JsonValue::Object`는 `std::map` → 키 알파벳 정렬 보장.
- `JsonWriter`는 `\u` 이스케이프를 UTF-8로 디코딩해 저장; surrogate pair는 미지원(BMP 한정).
- 에러는 모두 `std::runtime_error`로 throw.

---
> Source: [kjong37-max/JsonParsing](https://github.com/kjong37-max/JsonParsing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
