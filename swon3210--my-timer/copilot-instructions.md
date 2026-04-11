## my-timer

> 적용 범위: 전역. 페이지/컴포넌트(`src/app/**`), 공용 UI(`src/components/**`).

## 공통 마크업/스타일링 규칙 (App Router + Tailwind)

적용 범위: 전역. 페이지/컴포넌트(`src/app/**`), 공용 UI(`src/components/**`).
참고: 예시 코드는 `@goals` 구현에서 발췌했으나, 규칙은 모든 도메인에 적용됩니다.

### 원칙
- Tailwind 유틸리티 우선. 불가피한 동적 값만 인라인 스타일 사용.
- 의미 기반 색/토큰 일관성 유지(배지/버튼/카드). 매핑 함수로 중앙집중화.
- 레이아웃은 `flex`/`grid` 중심, 절대 위치 지양.
- 상태 피드백은 `hover:`/`focus:`와 `transition-*` 조합 사용.
- 접근성: 의미 있는 텍스트/레이블 제공, 포커스 가능한 요소는 포커스 링 유지.

### 카드/리스트 스타일 패턴 (예시)
```64:76:src/app/account-book/goals/components/GoalCard.tsx
<div className="bg-white rounded-xl p-6 border border-gray-200 hover:border-blue-300 hover:bg-blue-50 transition-all">
  ...
  <div className="w-16 h-16 bg-gray-200 rounded-lg flex items-center justify-center overflow-hidden">
    {goal.imageUrl ? (
      <img src={goal.imageUrl} alt={goal.displayName} className="w-full h-full object-cover" />
    ) : (
      <svg className="w-8 h-8 text-gray-400" ... />
    )}
  </div>
```

### 진행률/상태 표시 패턴 (예시)
```169:173:src/app/account-book/goals/components/GoalCard.tsx
<div className="w-full bg-gray-200 rounded-full h-3">
  <div className="bg-blue-500 rounded-full h-3 transition-all duration-1000" style={{ width: `${progressPercentage}%` }} />
</div>
```

### 페이지 헤더/CTA 패턴 (예시)
```56:85:src/app/account-book/goals/page.tsx
<div className="grow flex flex-col gap-6 p-6 bg-gray-50 min-h-screen">
  <div className="flex justify-between items-center">
    <div>
      <h1 className="text-2xl font-bold text-gray-800">목표 관리</h1>
      <p className="text-gray-600 text-sm mt-1">저축 목표를 설정하고 달성 상황을 관리하세요</p>
    </div>
    <button className="px-4 py-2 bg-blue-600 text-white rounded-xl font-medium hover:bg-blue-700 transition-colors flex items-center space-x-2">...
```

### 허용되는 인라인 스타일 예외
- 진행률 너비: `style={{ width: `${progressPercentage}%` }}`
- 그 외 인라인 스타일은 토큰화가 어려운 동적 길이/각도/좌표 등 최소한으로만.

### 작업 순서
1. UI 스켈레톤 구성(`rounded-xl border p-6`, 기본 타이포/색 적용).
2. 상태 피드백 추가(hover/focus/disabled, `transition-*`).
3. 의미 기반 매핑 추출(예: 우선순위 → 배지 색 클래스).
4. 동적 스타일 최소 인라인 처리(진행률 등).
5. 접근성/반응형 점검(레이블/alt/포커스 링, 간격/정렬 확인).

### 마지막 단계(필수)
- `pnpm tsc --noEmit`와 `pnpm lint`를 실행해 타입/린트 오류가 0인지 확인합니다.
- /≥,mnbv bghnbn bhb 오류가 있으면 수정 후 다시 `pnpm tsc --noEmit`, `pnpm lint`를 반복 실행합니다. 오류가 모두 해결될 때까지 반복합니다.
=-0`1` 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/swon3210) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
