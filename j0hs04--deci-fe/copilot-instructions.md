## convention

> Decision Radar F 프로젝트 코드 컨벤션. 새 파일 작성·기존 코드 수정 시 항상 적용.


# Decision Radar F — 코드 컨벤션

이 프로젝트의 실제 코드 패턴을 기준으로 작성했다. 새 파일을 만들기 전에 반드시 읽어라.

---

## 1. 디렉토리 구조

```
src/
├── config.ts                  # 환경 설정 (baseURL 등)
├── core/                      # 앱 전역 공통 인프라
│   ├── api/
│   │   ├── client.ts          # axios 인스턴스 + 인터셉터
│   │   ├── errors.ts          # ApiError 타입, ApiResponseError 클래스
│   │   ├── response.schema.ts # 공통 응답 Zod 스키마 + parseSchema()
│   │   └── tokenStore.ts      # 토큰 in-memory 저장소
│   ├── components/            # 기능 무관 공통 컴포넌트 (ErrorBanner 등)
│   └── hooks/                 # 기능 무관 공통 훅 (useApiError 등)
├── features/                  # 기능 단위 모듈
│   ├── auth/
│   │   ├── screens/           # 화면 컴포넌트
│   │   ├── auth.api.ts
│   │   ├── auth.schema.ts
│   │   ├── session.store.ts
│   │   └── useSession.ts
│   └── meeting/
│       ├── screens/
│       ├── meeting.api.ts
│       ├── cards.api.ts
│       └── meeting.schema.ts
├── i18n/
│   ├── i18n.ts                # t() 유틸
│   └── locales/ko.ts
├── navigation/
│   ├── RootNavigator.tsx
│   └── types.ts               # 모든 스택 파라미터 타입
└── theme/
    ├── tokens.ts              # 색상·타이포·스페이싱·radius 토큰
    ├── theme.ts               # lightTheme / darkTheme 조립
    └── useTheme.ts
```

**규칙**
- `core/` — feature에 속하지 않는 코드만. API 클라이언트, 공통 훅, 공통 컴포넌트.
- `features/{name}/` — 기능 단위로 자기 완결. api / schema / store / hooks / screens 포함.
- 화면 컴포넌트는 반드시 `features/{name}/screens/` 안에.

---

## 2. 파일 네이밍

| 종류 | 패턴 | 예시 |
|------|------|------|
| 화면 컴포넌트 | `PascalCaseScreen.tsx` | `LoginScreen.tsx` |
| 스택 네비게이터 | `PascalCaseStack.tsx` | `AuthStack.tsx` |
| API 모듈 | `camelCase.api.ts` | `auth.api.ts` |
| Zod 스키마 | `camelCase.schema.ts` | `auth.schema.ts` |
| Zustand 스토어 | `camelCase.store.ts` | `session.store.ts` |
| 커스텀 훅 | `useXxx.ts` | `useSession.ts` |
| 타입 정의 | `types.ts` | `navigation/types.ts` |

---

## 3. 컴포넌트 작성 패턴

```tsx
// ✅ named export 사용. default export 금지.
export function LoginScreen({ navigation }: AuthScreenProps<'Login'>) {
  const theme = useTheme();
  const { error, handle, clear } = useApiError();
  const c = theme.colors;
  const r = theme.radius;
  const sp = theme.spacing;

  return (
    <SafeAreaView style={[styles.safe, { backgroundColor: c.canvas }]}>
      ...
    </SafeAreaView>
  );
}

// StyleSheet는 파일 맨 아래에 한 곳에만
const styles = StyleSheet.create({
  safe: { flex: 1 },
  ...
});
```

**규칙**
- `SafeAreaView`는 `react-native-safe-area-context`에서 import.
- 색상·radius·spacing은 모두 `useTheme()`으로 참조. 하드코딩 금지.
- 스타일 상수(숫자)는 `StyleSheet.create()`에, 테마 연동 값은 인라인 `style={[styles.x, { color: c.ink }]}`.
- 폼이 있는 화면은 `KeyboardAvoidingView` + `Platform.OS === 'ios' ? 'padding' : undefined`.

---

## 4. API 모듈 패턴

```ts
// features/{name}/{name}.api.ts

export const authApi = {
  login: async (body: LoginInput) => {
    const res = await apiClient.post('/api/auth/login', body);
    return res.data as { message: string };
  },

  register: async (body: RegisterBody) => {
    const res = await apiClient.post('/api/auth/register', body);
    return parseSchema(sessionSchema, res.data);
  },
};
```

**규칙**
- `class` 금지. 함수 객체(`const xxxApi = { ... }`)로 통일.
- 응답 데이터는 `parseSchema(schema, res.data)`로 검증 후 반환.
- 에러 처리는 `client.ts` 인터셉터가 담당. 개별 API 함수에서 try/catch 금지.
- 화면에서 직접 `apiClient` 사용 금지. 반드시 feature api 모듈을 통해 호출.

---

## 5. Zod 스키마 패턴

```ts
// features/{name}/{name}.schema.ts — 입력 스키마 (폼/요청 body)
export const loginSchema = z.object({ ... });
export type LoginInput = z.infer<typeof loginSchema>;

// core/api/response.schema.ts — 응답 스키마 (API 응답 파싱)
export const meetingResponseSchema = z.object({ ... });
export type MeetingResponse = z.infer<typeof meetingResponseSchema>;
```

**규칙**
- 입력 스키마(폼 검증): `features/{name}/{name}.schema.ts`
- 응답 스키마(API 파싱): `core/api/response.schema.ts`
- `type`은 항상 `z.infer<>`로 도출. 별도 interface 정의 금지.

---

## 6. 상태 관리

```ts
// Zustand 스토어 — features/{name}/{name}.store.ts
export const useSessionStore = create<SessionState>()(set => ({
  userId: null,
  isAuthenticated: false,
  setSession: (userId, token) => { ... },
  clearSession: () => { ... },
}));

// 스토어 접근은 feature 훅으로 래핑 — features/{name}/useXxx.ts
export function useSession() {
  const setSession = useSessionStore(s => s.setSession);
  return { setSession, ... };
}
```

**규칙**
- 화면에서 `useXxxStore` 직접 사용 금지. 래핑 훅(`useSession` 등)을 통해 접근.
- 서버 상태(API 응답)는 각 화면의 `useState`로 관리. 전역 스토어에 캐싱 금지.
- 전역 상태는 세션처럼 진짜 전역인 것만. 화면 단위 상태는 로컬 `useState`.

---

## 7. 에러 처리

```ts
const { error, handle, clear } = useApiError();

const handleSubmit = async () => {
  clear();
  try {
    await authApi.login(input);
  } catch (e) {
    handle(e);
  }
};

{error && (
  <Text style={{ color: c.danger }}>
    {t(`error.${error.apiError.code}` as Parameters<typeof t>[0])}
  </Text>
)}
```

**규칙**
- 모든 API 에러는 `useApiError()` 훅으로 처리.
- 에러 메시지는 반드시 `t('error.ERROR_CODE')` — `ko.ts`에 등록된 메시지 사용.
- `UNAUTHORIZED` 에러는 인터셉터에서 전역 처리. 화면에서 별도 처리 금지.

---

## 8. 네비게이션

```ts
// navigation/types.ts — 파라미터 타입 한 곳에 정의
export type AuthStackParamList = {
  Login: undefined;
  OtpVerify: { email: string };
};

export type AuthScreenProps<T extends keyof AuthStackParamList> =
  NativeStackScreenProps<AuthStackParamList, T>;

// 화면에서 사용
export function OtpVerifyScreen({ route, navigation }: AuthScreenProps<'OtpVerify'>) {
  const { email } = route.params;
}
```

**규칙**
- 스택 파라미터 타입은 `navigation/types.ts`에만 정의.
- 화면 props는 `AuthScreenProps<'ScreenName'>` / `AppScreenProps<'ScreenName'>` 사용.
- 스택 네비게이터 파일(`AuthStack.tsx`)은 화면 등록만. 로직 금지.

---

## 9. i18n

```ts
import { t } from '../../../i18n/i18n';
<Text>{t('auth.loginTitle')}</Text>
```

**규칙**
- 화면에 한국어 문자열 하드코딩 금지. 전부 `t()` 사용.
- 키는 `namespace.camelCase`. namespace는 `auth` / `meeting` / `card` / `error` / `common`.
- 새 에러 코드 추가 시 `error.ERROR_CODE` 키도 반드시 함께 추가.

---

## 10. 테마

```ts
const theme = useTheme();
const c = theme.colors;   // c.ink, c.canvas, c.primary ...
const r = theme.radius;   // r.full, r.lg, r.sm
const sp = theme.spacing; // sp.lg, sp.xl ...

// 버튼 — radius.full 필수
<TouchableOpacity style={{ borderRadius: r.full, backgroundColor: c.primary }} />

// 카드 — radius.lg 필수
<View style={{ borderRadius: r.lg, borderWidth: 1, borderColor: c.hairline }} />
```

**규칙**
- `DESIGN.md` / `DESIGN-DARK.md`가 최종 기준. 토큰 값 임의 변경 금지.
- 버튼·인풋 → `radius.full`. 카드 → `radius.lg`. 그 외 radius 사용 금지.
- 그림자(`shadow`) 사용 금지. 구분은 `hairline` border(1px)로.
- 비활성 버튼: `backgroundColor: c.surfaceSoft`, `color: c.mute`.

---
> Source: [j0hs04/Deci_FE](https://github.com/j0hs04/Deci_FE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
