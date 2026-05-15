## component-standards

> // components/admin/PromotionEditor.tsx

# 컴포넌트 개발 표준

## 컴포넌트 구조
```tsx
// components/admin/PromotionEditor.tsx
import React from 'react';
import { Button } from '@/components/ui/button';
import { Card } from '@/components/ui/card';

interface PromotionEditorProps {
  onSave: (data: PromotionData) => void;
  initialData?: PromotionData;
}

export const PromotionEditor: React.FC<PromotionEditorProps> = ({
  onSave,
  initialData
}) => {
  // 상태 관리
  const [formData, setFormData] = useState<PromotionData>(initialData || {
    title: '',
    content: '',
    imageUrl: ''
  });

  // 이벤트 핸들러
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSave(formData);
  };

  return (
    <Card className="p-6">
      <form onSubmit={handleSubmit}>
        {/* 폼 내용 */}
      </form>
    </Card>
  );
};
```

## 컴포넌트 분류
- **ui/**: Shadcn UI 기본 컴포넌트
- **guide/**: 가이드 관련 컴포넌트
- **admin/**: 관리자 기능 컴포넌트
- **common/**: 공통 컴포넌트

## Props 인터페이스 규칙
- **필수 props**: 기본값 없이 정의
- **선택적 props**: `?` 표시로 정의
- **이벤트 핸들러**: `on` 접두사 사용
- **데이터 props**: 명확한 타입 정의

## 스타일링 규칙
- **Shadcn UI 컴포넌트** 우선 사용
- **TailwindCSS 클래스**로 커스터마이징
- **반응형 디자인** 필수 적용
- **접근성** 고려 (aria-label, role 등)
description:
globs:
alwaysApply: false
---

---
> Source: [jang-seung-hee/rental-news](https://github.com/jang-seung-hee/rental-news) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
