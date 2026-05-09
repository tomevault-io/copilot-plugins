## jumin-s-project

> 이 프로젝트에는 완성된 디자인 시스템이 있습니다. **항상 기존 컴포넌트를 사용하세요.**

# TDS Mobile Design System — AI Usage Guide

## Critical Rule: Never Create UI Components from Scratch

이 프로젝트에는 완성된 디자인 시스템이 있습니다. **항상 기존 컴포넌트를 사용하세요.**
`<button>`, `<input>`, `<div>` 래퍼 등 raw HTML을 직접 작성하지 마세요.

```ts
import { Button, TextField, Modal, ... } from './design-system';
// 또는 tsconfig paths에 따라 '@/design-system'
```

---

## Detailed Reference

> 전체 API / 토큰 / 훅은 아래 파일을 읽어서 사용하세요:
> - **컴포넌트 API**: `.claude/docs/component-apis.md`
> - **디자인 토큰** (색상/간격/타입/반경/모션/그림자): `.claude/docs/design-tokens.md`
> - **훅** (useDialog, useToast, useBottomSheet): `.claude/docs/hooks.md`

---

## Available Components

### Basic
| 컴포넌트 | Import | 한 줄 설명 |
|---|---|---|
| `Badge` | `Badge` | 색상 태그/라벨. fill/weak variant |
| `Border` | `Border` | 구분선 |
| `Button` | `Button` | 주요 액션 버튼. color/variant/display/size |
| `Checkbox` | `Checkbox` | 체크박스. `Checkbox.Circle` / `Checkbox.Line` |
| `IconButton` | `IconButton` | 아이콘만 있는 버튼 |
| `TextButton` | `TextButton` | 텍스트만 있는 경량 버튼 |
| `Switch` | `Switch` | 온/오프 토글. 완전 제어 |

### Input
| 컴포넌트 | Import | 한 줄 설명 |
|---|---|---|
| `TextField` | `TextField` | 텍스트 입력. 4가지 variant, 에러/도움말 지원 |
| `TextArea` | `TextArea` | 멀티라인 텍스트 입력 |
| `SplitTextField` | `SplitTextField` | 분할 입력 (인증번호 등) |
| `SearchField` | `SearchField` | 검색 전용 입력 |
| `Slider` | `Slider` | 범위 선택 슬라이더 |
| `NumberStepper` | `NumberStepper` | +/− 버튼으로 숫자 증감. 완전 제어 |

### Navigation & Selection
| 컴포넌트 | Import | 한 줄 설명 |
|---|---|---|
| `Tab` | `Tab` | 탭 네비게이션. compound component |
| `SegmentedControl` | `SegmentedControl` | 세그먼트 필터. options 배열 |
| `Menu` | `Menu` | 드롭다운 메뉴. trigger prop으로 토글 |

### Data Display
| 컴포넌트 | Import | 한 줄 설명 |
|---|---|---|
| `List` | `List` | 목록 컨테이너 |
| `ListRow` | `ListRow` | 목록 행. contents/subTitle (title/description 아님) |
| `ListHeader`, `ListFooter` | `ListHeader`, `ListFooter` | 목록 헤더/푸터 |
| `ProgressBar` | `ProgressBar` | 진행 막대 |
| `ProgressStepper` | `ProgressStepper` | 단계 표시기 |
| `Rating` | `Rating` | 별점 |
| `Skeleton` | `Skeleton` | 로딩 플레이스홀더 |

### Feedback & Overlay
| 컴포넌트 | Import | 한 줄 설명 |
|---|---|---|
| `Toast`, `ToastProvider` | `Toast` | 짧은 알림 메시지 |
| `Tooltip` | `Tooltip` | hover 시 설명 말풍선. content는 string만 |
| `AlertDialog` | `AlertDialog` | 단순 확인 다이얼로그 (버튼 1개) |
| `ConfirmDialog` | `ConfirmDialog` | 확인/취소 다이얼로그 (버튼 2개) |
| `Modal` | `Modal` | 저수준 오버레이 primitive |
| `BottomSheet` | `BottomSheet` | 자유 레이아웃 바텀 패널 |
| `BottomInfo` | `BottomInfo` | 제목+설명 고정 바텀 패널 |
| `BottomCTA` | `BottomCTA` | 하단 고정 CTA 버튼. Single/Double |
| `Loader` | `Loader` | 스피너 로더 |

### Navigation
| 컴포넌트 | Import | 한 줄 설명 |
|---|---|---|
| `Top` | `Top` | 상단 앱 바. 뒤로가기/타이틀/서브타이틀/하단 영역 지원 |

### Hooks
| 훅 | Import | 용도 |
|---|---|---|
| `useDialog` | `useDialog` | async/await 방식으로 다이얼로그 제어 |
| `useToast` | `useToast` | Toast 메시지 표시 |
| `useBottomSheet` | `useBottomSheet` | BottomSheet 열기/닫기 상태 관리 |

---

## Overlay 선택 가이드

| 상황 | 컴포넌트 |
|---|---|
| 단순 확인 메시지 (버튼 1개) | `AlertDialog` |
| 확인/취소 선택 (버튼 2개) | `ConfirmDialog` |
| 제목+설명 고정 바텀 패널 | `BottomInfo` |
| 스크롤/폼/목록이 있는 바텀 패널 | `BottomSheet` |
| 완전 커스텀 오버레이 (위 케이스 불가 시) | `Modal` |
| 화면 하단 고정 버튼 | `BottomCTA` |

---

## Common Mistakes to Avoid

1. **`Button display="block"` 없음** → `display="full"` 사용
2. **`AlertDialog`는 `onClose`** — `onOpenChange` 아님
3. **`ConfirmDialog` 버튼은 자동 닫힘** — `onClick` 안에서 `onClose()` 호출 금지
4. **`actionsLayout`은 `'split'`/`'end'`** — `'horizontal'`/`'right'` 없음
5. **항상 `'./design-system'` 단일 경로로 import** — deep path 금지
6. **커스텀 modal/dialog 마크업 금지** — AlertDialog/ConfirmDialog/BottomSheet가 모든 케이스 커버
7. **`ListRow`는 `contents`/`subTitle`** — `title`/`description` 아님
8. **`ListHeader`는 `title`** — `label` 아님. `ListFooter`는 `children` 사용
9. **`ListRow` + Switch** — `nonInteractive` 추가해서 행 자체가 클릭 타겟 되지 않도록
10. **`Menu.Trigger`/`Menu.List` 없음** — trigger는 prop, 항목은 children으로 직접 전달
11. **lucide-react `Menu` 충돌** → `import { Menu as MenuIcon } from 'lucide-react'`
12. **`BottomSheet`=`open`, `BottomInfo`=`isOpen`** — prop 이름 혼동 주의
13. **`Switch`/`Checkbox` `onChange` 인자는 `boolean`** — `ChangeEvent` 아님
14. **`Checkbox`는 `Checkbox.Circle` / `Checkbox.Line`** — 단독 `<Checkbox />` 불가
15. **`BottomCTA`는 `BottomCTA.Single` / `BottomCTA.Double`** — 단독 불가
16. **`Top` `onBack`+`onClose` 동시 제공 시 `onBack`(←)만 표시** — X 버튼은 `right` prop으로
17. **`Tooltip content`는 string만** — ReactNode 불가
18. **`useDialog`는 UI 렌더 안 함** — `dialogState`를 AlertDialog에 수동 연결 필요

---
> Source: [joomin130-cmyk/jumin-s-project](https://github.com/joomin130-cmyk/jumin-s-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
