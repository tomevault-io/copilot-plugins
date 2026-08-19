## fathom

> VR/AR hand gesture interaction system for a Chinese ancient architecture surveying game. Built with Vite + React + TypeScript + Three.js + MediaPipe Hands.

# AGENTS.md - Fathom

## Project Overview

VR/AR hand gesture interaction system for a Chinese ancient architecture surveying game. Built with Vite + React + TypeScript + Three.js + MediaPipe Hands.

## Build/Test/Lint Commands

```bash
# Development
npm run dev           # Start dev server
npm run build         # Production build
npm run preview       # Preview production build

# Testing (Vitest)
npm run test          # Run all tests
npm run test src/hand/gestures/pinch.test.ts  # Single test
npm run test -- --watch                       # Watch mode

# Linting
npx tsc --noEmit      # Type check
npm run lint          # Run ESLint
npm run lint -- --fix # Fix ESLint issues
```

## Code Style

### Imports

```typescript
// Group: React, Third-party, Internal
import React, { useEffect } from 'react';
import * as THREE from 'three';
import { HandLandmarker } from '@mediapipe/tasks-vision';

import { HandState } from '@/hand/types';
```

### Formatting

- 2-space indentation, single quotes, semicolons required
- Max line length: 100, trailing commas in multi-line

### Naming Conventions

```typescript
// PascalCase: Classes, Interfaces, Enums
class HandStateMachine {}
interface GestureConfig {}
enum ToolType { RULER = 'RULER' }

// camelCase: variables, functions, hooks
const handPosition = new THREE.Vector3();
function detectPinch(): boolean {}

// SCREAMING_SNAKE_CASE: constants
const PINCH_THRESHOLD = 0.05;
const CONFIDENCE_MIN = 0.7;
```

### TypeScript Guidelines

- Prefer `interface` for object shapes, `enum` for finite states
- Explicit function return types required
- Avoid `any`, use `unknown` for unknown types
- Never use `@ts-ignore` or `@ts-expect-error`

## Quality Standards (Mandatory)

### Comments (Chinese Required)

```typescript
// 中文注释：解释为什么这样做
// 捏合阈值基于人手自然捏合时的平均距离
const PINCH_THRESHOLD = 0.05;
```
- Comments must be in **Chinese (中文)**
- Explain **why**, not what
- Document complex logic, hacks, contracts

### Prohibited

- Giant functions (>50 lines) - split into smaller functions
- Implicit state - use explicit state management
- Magic numbers - use named constants
- Deep nesting (>3 levels) - use early returns
- Pinyin naming, ambiguous abbreviations
- @ts-ignore / @ts-expect-error

### Must Satisfy

- Single responsibility per function/module
- Naming expresses business semantics and intent
- Error handling must be traceable; never swallow exceptions
- Testability and replaceability

### Error Handling

```typescript
// Always handle errors explicitly
try {
  await handLandmarker.detect(video);
} catch (error) {
  console.error('手势检测失败:', error);
  setHandState(HandState.IDLE); // 降级到安全状态
}

// Guard clauses
function update(landmarks?: HandLandmark[]) {
  if (!landmarks?.length) return;
  // ... proceed with valid data
}
```

## Three.js Patterns

```typescript
// Dispose to prevent memory leaks
const geometry = new THREE.SphereGeometry(0.01);
const material = new THREE.MeshBasicMaterial({ color: 0xf5f5dc });

useEffect(() => {
  return () => {
    geometry.dispose();
    material.dispose();
  };
}, []);

// Group related objects
const handGroup = new THREE.Group();
handGroup.add(wristMesh);
```

## React Hooks Patterns

```typescript
// Custom hooks for reusable logic
export function useHandTracking() {
  const [landmarks, setLandmarks] = useState<HandLandmark[]>([]);
  const rendererRef = useRef<THREE.WebGLRenderer | null>(null);
  
  // Memoize expensive computations
  const smoothedPosition = useMemo(() => {
    return applyEMA(landmarks, alpha);
  }, [landmarks, alpha]);
  
  return { landmarks, rendererRef, smoothedPosition };
}
```

## Project Structure

```
src/
├── hand/           # 手部追踪与渲染
│   ├── tracking.ts
│   ├── types.ts
│   └── gestures/   # 手势检测
├── tools/          # 测量工具
│   ├── ChiselRuler.ts
│   ├── Camera.ts
│   └── Compass.ts
├── ui/             # React 组件
└── utils/          # 共享工具
    └── coords.ts
```

## Key Constants

```typescript
// 视觉设计
const HAND_MATERIAL_COLOR = 0xf5f5dc;  // 米白色硅胶材质
const GLOW_COLORS = {
  normal: 0x00d4ff,   // 青色边缘光
  hover: 0xff9f43,    // 橙色高亮
  pinch: 0x4caf50,    // 绿色捏合
  summon: 0xffd700,   // 金色召唤
};

// 手势检测参数
const PINCH_THRESHOLD = 0.05;     // 米
const PINCH_RELEASE = 0.08;
const FIST_DURATION = 300;        // 毫秒
const CONFIDENCE_MIN = 0.7;
```

## Testing

```typescript
// 测试文件: *.test.ts 与源文件同目录
import { describe, it, expect } from 'vitest';
import { detectPinch } from './pinch';

describe('detectPinch', () => {
  it('should return true when fingers are close', () => {
    const landmarks = createMockLandmarks({ thumbTip: 0, indexTip: 0.04 });
    expect(detectPinch(landmarks)).toBe(true);
  });
});
```

## Performance Guidelines

- Use `useRef` for Three.js objects to avoid re-renders
- Dispose geometries/materials on unmount
- Throttle high-frequency updates (gesture detection)
- Use `requestAnimationFrame` for render loop

## Self-Check Before Submitting

- [ ] 是否有更简单的实现方式？
- [ ] 是否存在隐式耦合？
- [ ] 是否易于测试？
- [ ] 新团队成员能否理解？
- [ ] 圈复杂度 < 10
- [ ] 核心逻辑覆盖率 > 80%

---
> Source: [xiangkehan/Fathom](https://github.com/xiangkehan/Fathom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
