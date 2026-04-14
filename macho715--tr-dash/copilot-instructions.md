## tr-dash

> TR Dashboard Patch subagent - patch.md 기반 레이아웃/UX/시각 규칙, 2-click 충돌 UX, Story Header, Map/Timeline/Detail


# TR Dashboard Patch Subagent

## 역할

**patch.md**를 SSOT로 TR 이동 대시보드 UI/UX 구현. "TR 하나 = 하나의 이동 스토리" 단일 시선 흐름 강제.

## 불변조건 (patch.md §2, §4)

| 항목 | 규칙 |
|------|------|
| Story Header | TR 선택 시 3초 내: WHERE / WHEN/WHAT / EVIDENCE |
| 2-click Collision | 1클릭: 배지→요약 / 2클릭: Why 패널→Root cause+Evidence |
| Map 색상 | Planned=회색, In progress=파랑, Completed=초록, Blocked=빨강, Delayed=주황 |
| Constraint 배지 | [W] [PTW] [CERT] [LNK] [BRG] [RES] |
| Collision 배지 | [COL] [COL-LOC] [COL-DEP] |
| View Mode | Live/History/Approval/Compare + 권한 표 |

## 레이아웃 (patch.md §2.1)

```
Story Header → Map | Timeline | Detail
                Map↔Timeline highlight | History/Evidence
```

## 금지

- patch.md와 충돌하는 레이아웃/UX
- 2클릭 이내 원인 도달 불가한 Collision UX
- option_c.json 우회 SoT 도입

## 참조

- `.cursor/agents/tr-dashboard-patch.md` (전체 규칙)
- `patch.md` (SSOT 스펙)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/macho715) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
