## blog-automation-backend

> Next.js 프론트엔드 개발 가이드라인 및 모범 사례


- **파일 구조 및 네이밍**
  - App Router 사용: `frontend/src/app/` 디렉토리 구조 준수
  - 페이지 파일: `page.tsx`, 레이아웃: `layout.tsx`, 로딩: `loading.tsx`
  - 컴포넌트는 PascalCase: `UserProfile.tsx`, `ContentCard.tsx`
  - 유틸리티는 camelCase: `apiClient.ts`, `authUtils.ts`

- **컴포넌트 작성 규칙**
  ```tsx
  // ✅ DO: 함수형 컴포넌트와 명시적 타입 정의
  interface UserProfileProps {
    userId: string;
    showActions?: boolean;
  }
  
  export function UserProfile({ userId, showActions = true }: UserProfileProps) {
    return <div>...</div>;
  }
  
  // ❌ DON'T: 기본 내보내기나 암묵적 타입
  export default function UserProfile(props: any) {
    return <div>...</div>;
  }
  ```

- **shadcn/ui 컴포넌트 사용**
  - UI 컴포넌트는 `@/components/ui`에서 가져오기
  - 커스텀 스타일링은 Tailwind CSS 클래스 사용
  - 복합 컴포넌트는 `@/components`에 별도 생성

- **상태 관리**
  ```tsx
  // ✅ DO: useState와 useEffect를 적절히 사용
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        setIsLoading(true);
        // 데이터 페칭 로직
      } catch (err) {
        setError(err instanceof Error ? err.message : '알 수 없는 오류');
      } finally {
        setIsLoading(false);
      }
    };
    
    fetchData();
  }, []);
  ```

- **API 호출**
  - [apiClient.ts](mdc:frontend/src/lib/api.ts) 사용하여 일관된 API 호출
  - 에러 처리는 try-catch 블록 사용
  - 로딩 상태와 에러 상태 관리 필수

- **환경 변수**
  - 클라이언트 노출 변수는 `NEXT_PUBLIC_` 접두사 사용
  - [supabase.ts](mdc:frontend/src/lib/supabase.ts) 참조하여 설정

- **타입 안전성**
  ```tsx
  // ✅ DO: 명시적 타입 정의
  interface ApiResponse<T> {
    data: T;
    error?: string;
  }
  
  // ❌ DON'T: any 타입 사용 금지
  const response: any = await api.get('/users');
  ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ldhDeveloper)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ldhDeveloper)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
