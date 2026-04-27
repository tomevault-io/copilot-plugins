## 2025-capstone

> [FontSizeContext.tsx](mdc:src/context/FontSizeContext.tsx)를 참조하여 Context 생성:

# 상태 관리 패턴 가이드

## 🎯 전역 상태 관리

### Context API 패턴
[FontSizeContext.tsx](mdc:src/context/FontSizeContext.tsx)를 참조하여 Context 생성:

1. **Context 생성**
```typescript
const YourContext = createContext<YourContextType | undefined>(undefined);
```

2. **Provider 컴포넌트**
```typescript
export const YourProvider = ({ children }) => {
  const [state, setState] = useState(initialState);
  
  useEffect(() => {
    // localStorage에서 상태 복원
  }, []);
  
  return (
    <YourContext.Provider value={{ state, setState }}>
      {children}
    </YourContext.Provider>
  );
};
```

3. **Hook 생성**
```typescript
export const useYourContext = () => {
  const context = useContext(YourContext);
  if (!context) {
    throw new Error('Provider 내에서 사용해야 합니다');
  }
  return context;
};
```

## 💾 localStorage 활용

### 사용자 설정 영속성
다음 데이터들을 localStorage에 저장:
- 폰트 사이즈 설정 (`fontSize`)
- 대중교통 선호도 (`userPreference`)
- 선호도 클릭 통계 (`preferenceClickCounts`)
- 인증 토큰 (`accessToken`, `refreshToken`)

### 저장 패턴
```typescript
// 저장
localStorage.setItem('key', JSON.stringify(data));

// 불러오기
const data = localStorage.getItem('key');
if (data) {
  const parsedData = JSON.parse(data);
}
```

## 🎨 CSS 변수와 연동

### 실시간 스타일 업데이트
[FontSizeContext.tsx](mdc:src/context/FontSizeContext.tsx)의 패턴 참조:
```typescript
document.documentElement.style.setProperty('--css-variable', value);
```

### 전역 CSS 변수
[_global.tsx](mdc:src/style/_global.tsx)에서 CSS 변수 정의:
- `--base-font-size`
- `--title-font-size`
- `--subtitle-font-size`

## 📊 사용자 행동 추적

### 클릭 통계 패턴
[Preference.tsx](mdc:src/page/search/Preference.tsx) 참조:
```typescript
const [clickCounts, setClickCounts] = useState<ClickCounts>({});

const handleClick = (option) => {
  const newCounts = {
    ...clickCounts,
    [option]: (clickCounts[option] || 0) + 1
  };
  setClickCounts(newCounts);
  localStorage.setItem('counts', JSON.stringify(newCounts));
};
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yongchane) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
