## performance-standards

> - **압축**: 모든 이미지는 압축 후 업로드

# 성능 최적화 표준

## 이미지 최적화
- **압축**: 모든 이미지는 압축 후 업로드
- **포맷**: WebP, AVIF 우선 사용
- **지연 로딩**: `loading="lazy"` 속성 사용
- **반응형 이미지**: `srcset` 속성 활용

## 번들 최적화
- **코드 분할**: React.lazy() 사용
- **트리 쉐이킹**: 사용하지 않는 코드 제거
- **캐싱**: 적절한 캐시 헤더 설정

## React 최적화
```tsx
// 메모이제이션 사용
const MemoizedComponent = React.memo(({ data }) => {
  return <div>{data}</div>;
});

// useMemo로 계산 최적화
const expensiveValue = useMemo(() => {
  return heavyCalculation(data);
}, [data]);

// useCallback으로 함수 최적화
const handleClick = useCallback(() => {
  // 클릭 핸들러
}, [dependencies]);
```

## 로딩 상태 관리
- **스켈레톤 UI**: 로딩 중 사용자 경험 개선
- **에러 바운더리**: 에러 상황 처리
- **오프라인 지원**: Service Worker 활용

## 성능 목표
- **페이지 로딩**: 3초 이내
- **이미지 로딩**: 1초 이내
- **인터랙션**: 100ms 이내 응답
- **번들 크기**: 500KB 이하
description:
globs:
alwaysApply: false
---

---
> Source: [jang-seung-hee/rental-news](https://github.com/jang-seung-hee/rental-news) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
