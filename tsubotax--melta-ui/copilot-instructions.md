## components

> melta UI component Tailwind class reference


# melta UI Component Classes

## Button
```
CTA:     inline-flex items-center justify-center gap-2 h-10 px-4 text-[1rem] font-medium bg-primary-500 text-white rounded-lg hover:bg-primary-700 cursor-pointer
Sub:     inline-flex items-center justify-center gap-2 h-10 px-4 text-[1rem] font-medium bg-white text-slate-700 border border-slate-200 rounded-lg hover:bg-gray-50 cursor-pointer
Danger:  inline-flex items-center justify-center gap-2 h-10 px-4 text-[1rem] font-medium bg-red-500 text-white rounded-lg hover:bg-red-600 cursor-pointer
Sizes:   Large h-12 px-6 text-[1rem] | Medium h-10 px-4 text-[1rem] | Small h-8 px-3 text-[0.875rem]
Icon:    Large w-12 h-12 | Medium w-10 h-10 | Small w-8 h-8
Icon+Text: pl-3 pr-4 (M) | pl-2.5 pr-3 (S) | pl-5 pr-6 (L) — asymmetric padding
```

## Card
```
Basic:   bg-white rounded-xl border border-slate-200 p-6 shadow-sm
Grid:    grid grid-cols-2 md:grid-cols-3 gap-6
```

## Input / TextField
```
Default: w-full px-3 py-2 text-base border border-slate-300 rounded-lg focus:ring-2 focus:ring-primary-500/50 caret-primary-500
Wrapper: <div class="leading-normal"> + <label> + <input>
```

## Select (appearance-none REQUIRED)
```
Default: appearance-none w-full px-3 py-2 text-base border border-slate-300 rounded-lg pr-10
Chevron: <svg class="absolute right-3 top-1/2 -translate-y-1/2 w-4 h-4 text-slate-400 pointer-events-none">
```

## Table
```
Wrapper: bg-white rounded-xl border border-slate-200 overflow-hidden
Header:  border-b border-slate-200 bg-gray-50
TH:      <th scope="col"> text-left py-3 px-4 text-xs font-medium text-slate-500 uppercase tracking-wider
TD:      py-3 px-4 text-sm text-slate-900
Row:     hover:bg-gray-50 transition-colors
```

## Badge
```
Success: inline-flex items-center gap-1.5 px-2.5 py-0.5 rounded-full text-xs font-medium bg-emerald-50 text-emerald-700
Warning: ... bg-amber-50 text-amber-700
Danger:  ... bg-red-50 text-red-700
Neutral: ... bg-slate-100 text-slate-700
```

## Alert
```
Info:    bg-primary-50 border border-primary-200 text-primary-800 rounded-lg p-4
Error:   bg-red-50 border border-red-200 text-red-800 rounded-lg p-4
Warning: bg-amber-50 border border-amber-200 text-amber-800 rounded-lg p-4
Success: bg-emerald-50 border border-emerald-200 text-emerald-800 rounded-lg p-4
```

## Sidebar
```
Standard: w-64 bg-white border-r border-slate-200 flex-shrink-0 flex flex-col h-screen
Compact:  w-16 items-center
Nav Active:  flex items-center gap-3 px-4 py-2.5 text-sm font-medium text-primary-500 bg-primary-50 rounded-lg + aria-current="page"
Nav Default: flex items-center gap-3 px-4 py-2.5 text-sm font-medium text-body hover:bg-gray-50 rounded-lg transition-colors
<nav aria-label="メインナビゲーション"> REQUIRED
```

## Tabs
```
Active:   text-sm font-semibold text-primary-500 border-b-2 border-primary-500
Inactive: text-sm font-medium text-slate-500 border-b-2 border-transparent hover:text-slate-700
```

## Modal
```
Overlay: fixed inset-0 bg-black/50 z-40
Dialog:  bg-white rounded-xl p-6 shadow-xl z-50
role="dialog" aria-modal="true" aria-labelledby REQUIRED
```

## Pagination
```
Active:   w-10 h-10 inline-flex items-center justify-center bg-primary-500 text-white rounded-lg
Inactive: w-10 h-10 inline-flex items-center justify-center bg-white border border-slate-200 rounded-lg hover:bg-gray-50
```

## Avatar
```
Initials: w-10 h-10 rounded-full bg-primary-50 text-primary-500 font-medium inline-flex items-center justify-center text-sm
role="img" aria-label="名前" REQUIRED
```

## Progress
```
Track: bg-slate-200 rounded-full h-2
Fill:  bg-primary-500 rounded-full h-2
role="progressbar" aria-valuenow aria-valuemin="0" aria-valuemax="100" REQUIRED
```

## Divider
```
Horizontal: <hr class="border-t border-slate-200">
With text:  flex items-center gap-4 + flex-1 border-t border-slate-200 + text-sm text-slate-500
Vertical:   border-l border-slate-200 self-stretch + role="separator" aria-orientation="vertical"
```

## Stepper
```
Completed: w-8 h-8 rounded-full bg-primary-500 text-white
Active:    w-8 h-8 rounded-full border-2 border-primary-500 bg-white text-primary-500 + aria-current="step"
Upcoming:  w-8 h-8 rounded-full bg-slate-100 text-slate-500
Connector: flex-1 h-0.5 mx-3 (bg-primary-500 done / bg-slate-200 upcoming)
```

## Full reference
See `metadata/components.json` for complete component data with all variants and HTML samples.

---
> Source: [tsubotax/melta-ui](https://github.com/tsubotax/melta-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
