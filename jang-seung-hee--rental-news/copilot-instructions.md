## firebase-standards

> - **데이터 저장**: Firestore만 사용

# Firebase 사용 표준

## 데이터 저장 규칙
- **데이터 저장**: Firestore만 사용
- **파일 저장**: Firebase Storage만 사용
- **인증**: Firebase Authentication만 사용
- **절대 금지**: 로컬스토리지에 중복 저장

## Firestore 컬렉션 구조
```typescript
// 프로모션 컬렉션
interface Promotion {
  id: string;
  title: string;
  content: string;
  imageUrl?: string;
  createdAt: Timestamp;
  updatedAt: Timestamp;
  isActive: boolean;
}

// 고객 정보 컬렉션
interface Customer {
  id: string;
  name: string;
  email: string;
  phone?: string;
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

## Firebase 서비스 함수 예시
```typescript
// services/firebase.ts
import { initializeApp } from 'firebase/app';
import { getFirestore, collection, addDoc, getDocs } from 'firebase/firestore';
import { getStorage, ref, uploadBytes, getDownloadURL } from 'firebase/storage';

// 프로모션 추가
export const addPromotion = async (promotionData: Omit<Promotion, 'id' | 'createdAt' | 'updatedAt'>) => {
  const db = getFirestore();
  const docRef = await addDoc(collection(db, 'promotions'), {
    ...promotionData,
    createdAt: new Date(),
    updatedAt: new Date()
  });
  return docRef.id;
};

// 이미지 업로드
export const uploadImage = async (file: File): Promise<string> => {
  const storage = getStorage();
  const storageRef = ref(storage, `promotions/${Date.now()}_${file.name}`);
  await uploadBytes(storageRef, file);
  return getDownloadURL(storageRef);
};
```

## 보안 규칙
- **환경변수**: Firebase 설정은 환경변수로 관리
- **API 키**: 코드에 하드코딩 금지
- **권한 관리**: Firestore 보안 규칙 설정 필수
description:
globs:
alwaysApply: false
---

---
> Source: [jang-seung-hee/rental-news](https://github.com/jang-seung-hee/rental-news) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
