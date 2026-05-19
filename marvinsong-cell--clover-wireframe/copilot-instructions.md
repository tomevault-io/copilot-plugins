## clover-wireframe

> 프리미엄 소개팅 서비스 클로버의 웹앱 프로젝트입니다.

# Clover - 소개팅 웹앱

프리미엄 소개팅 서비스 클로버의 웹앱 프로젝트입니다.

## Tech Stack

- **Frontend:** React 18 + TypeScript + Vite
- **Backend:** Supabase (Auth, Database, Storage, Realtime)
- **Styling:** Vanilla CSS with CSS Custom Properties
- **State:** Zustand (클라이언트) + React Query (서버)
- **Font:** Pretendard

## Project Structure

```
src/
├── app/                      # 앱 진입점
│   ├── App.tsx              # 루트 컴포넌트
│   ├── Router.tsx           # 라우팅 설정
│   └── providers/           # Context Providers
│       ├── AuthProvider.tsx
│       ├── QueryProvider.tsx
│       └── index.tsx
│
├── pages/                    # 페이지 컴포넌트 (라우트 단위)
│   ├── auth/                # 인증 관련
│   │   ├── LoginPage.tsx
│   │   ├── SignupPage.tsx
│   │   └── OnboardingPage.tsx
│   ├── main/                # 메인 앱
│   │   ├── FeedPage.tsx     # 매칭 피드
│   │   ├── ChatPage.tsx     # 채팅 목록
│   │   ├── ChatRoomPage.tsx # 채팅방
│   │   ├── MatchesPage.tsx  # 매칭 목록
│   │   └── ProfilePage.tsx  # 내 프로필
│   └── profile/             # 프로필 관련
│       ├── EditProfilePage.tsx
│       └── ViewProfilePage.tsx
│
├── components/               # 재사용 컴포넌트
│   ├── ui/                  # 기본 UI 컴포넌트 (디자인 시스템)
│   │   ├── Button/
│   │   ├── Input/
│   │   ├── Card/
│   │   └── index.ts
│   ├── layout/              # 레이아웃 컴포넌트
│   │   ├── Header.tsx
│   │   ├── BottomNav.tsx
│   │   ├── MobileFrame.tsx
│   │   └── Screen.tsx
│   └── features/            # 기능별 컴포넌트
│       ├── auth/
│       ├── matching/
│       ├── chat/
│       └── profile/
│
├── lib/                      # 라이브러리 & 유틸리티
│   ├── supabase/            # Supabase 클라이언트
│   │   ├── client.ts
│   │   ├── auth.ts
│   │   ├── database.ts
│   │   └── storage.ts
│   ├── hooks/               # 커스텀 훅
│   │   ├── useAuth.ts
│   │   ├── useProfile.ts
│   │   ├── useMatching.ts
│   │   └── useChat.ts
│   ├── api/                 # API 함수 (React Query용)
│   │   ├── auth.ts
│   │   ├── profiles.ts
│   │   ├── matching.ts
│   │   └── chat.ts
│   └── utils/               # 유틸리티 함수
│       ├── format.ts
│       ├── validation.ts
│       └── constants.ts
│
├── stores/                   # Zustand 스토어
│   ├── authStore.ts
│   ├── uiStore.ts
│   └── chatStore.ts
│
├── types/                    # TypeScript 타입 정의
│   ├── database.ts          # Supabase 테이블 타입
│   ├── auth.ts
│   ├── profile.ts
│   ├── matching.ts
│   └── chat.ts
│
├── styles/                   # 글로벌 스타일
│   ├── globals.css          # 전역 스타일 & CSS 변수
│   ├── reset.css            # CSS 리셋
│   └── animations.css       # 애니메이션
│
└── assets/                   # 정적 에셋
    ├── icons/
    └── images/

# 와이어프레임 (현재)
/wireframe
├── index.html
├── styles.css
├── App.jsx
└── components/
    ├── DesignSystem.jsx
    ├── Onboarding.jsx
    ├── PhoneAuth.jsx
    ├── BasicInfo.jsx
    ├── CloverInfo.jsx
    └── Complete.jsx
```

---

# Architecture & Design Patterns

## 1. Supabase Integration Pattern

### 클라이언트 초기화

```typescript
// lib/supabase/client.ts
import { createClient } from '@supabase/supabase-js';
import type { Database } from '@/types/database';

export const supabase = createClient<Database>(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
);
```

### Auth 패턴

```typescript
// lib/supabase/auth.ts
export const authApi = {
  // 휴대폰 인증
  signInWithPhone: async (phone: string) => {
    return supabase.auth.signInWithOtp({ phone });
  },

  // OTP 검증
  verifyOtp: async (phone: string, token: string) => {
    return supabase.auth.verifyOtp({ phone, token, type: 'sms' });
  },

  // 로그아웃
  signOut: async () => {
    return supabase.auth.signOut();
  },

  // 세션 가져오기
  getSession: async () => {
    return supabase.auth.getSession();
  },

  // 인증 상태 구독
  onAuthStateChange: (callback: (event, session) => void) => {
    return supabase.auth.onAuthStateChange(callback);
  }
};
```

### Database 패턴

```typescript
// lib/api/profiles.ts
export const profilesApi = {
  // 프로필 조회
  getProfile: async (userId: string) => {
    const { data, error } = await supabase
      .from('profiles')
      .select('*')
      .eq('user_id', userId)
      .single();

    if (error) throw error;
    return data;
  },

  // 프로필 업데이트
  updateProfile: async (userId: string, updates: ProfileUpdate) => {
    const { data, error } = await supabase
      .from('profiles')
      .update(updates)
      .eq('user_id', userId)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  // 매칭 피드 조회
  getMatchingFeed: async (userId: string, filters: MatchFilters) => {
    const { data, error } = await supabase
      .rpc('get_matching_feed', {
        current_user_id: userId,
        ...filters
      });

    if (error) throw error;
    return data;
  }
};
```

### Realtime 패턴 (채팅)

```typescript
// lib/hooks/useChat.ts
export const useRealtimeChat = (roomId: string) => {
  const [messages, setMessages] = useState<Message[]>([]);

  useEffect(() => {
    // 기존 메시지 로드
    loadMessages();

    // 실시간 구독
    const channel = supabase
      .channel(`room:${roomId}`)
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'messages',
          filter: `room_id=eq.${roomId}`
        },
        (payload) => {
          setMessages(prev => [...prev, payload.new as Message]);
        }
      )
      .subscribe();

    return () => {
      supabase.removeChannel(channel);
    };
  }, [roomId]);

  return { messages };
};
```

### Storage 패턴 (이미지 업로드)

```typescript
// lib/supabase/storage.ts
export const storageApi = {
  // 프로필 이미지 업로드
  uploadProfileImage: async (userId: string, file: File) => {
    const fileExt = file.name.split('.').pop();
    const fileName = `${userId}/${Date.now()}.${fileExt}`;

    const { data, error } = await supabase.storage
      .from('profiles')
      .upload(fileName, file, {
        cacheControl: '3600',
        upsert: false
      });

    if (error) throw error;

    // Public URL 반환
    const { data: { publicUrl } } = supabase.storage
      .from('profiles')
      .getPublicUrl(data.path);

    return publicUrl;
  },

  // 이미지 삭제
  deleteProfileImage: async (path: string) => {
    return supabase.storage.from('profiles').remove([path]);
  }
};
```

## 2. State Management Pattern

### Zustand Store (클라이언트 상태)

```typescript
// stores/authStore.ts
interface AuthState {
  user: User | null;
  session: Session | null;
  isLoading: boolean;
  setUser: (user: User | null) => void;
  setSession: (session: Session | null) => void;
  reset: () => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  session: null,
  isLoading: true,
  setUser: (user) => set({ user }),
  setSession: (session) => set({ session }),
  reset: () => set({ user: null, session: null })
}));
```

```typescript
// stores/uiStore.ts
interface UIState {
  isBottomSheetOpen: boolean;
  bottomSheetContent: React.ReactNode | null;
  openBottomSheet: (content: React.ReactNode) => void;
  closeBottomSheet: () => void;
  toast: ToastMessage | null;
  showToast: (message: ToastMessage) => void;
}

export const useUIStore = create<UIState>((set) => ({
  isBottomSheetOpen: false,
  bottomSheetContent: null,
  openBottomSheet: (content) => set({ isBottomSheetOpen: true, bottomSheetContent: content }),
  closeBottomSheet: () => set({ isBottomSheetOpen: false, bottomSheetContent: null }),
  toast: null,
  showToast: (message) => set({ toast: message })
}));
```

### React Query (서버 상태)

```typescript
// lib/hooks/useProfile.ts
export const useProfile = (userId: string) => {
  return useQuery({
    queryKey: ['profile', userId],
    queryFn: () => profilesApi.getProfile(userId),
    staleTime: 5 * 60 * 1000, // 5분
  });
};

export const useUpdateProfile = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ userId, data }: { userId: string; data: ProfileUpdate }) =>
      profilesApi.updateProfile(userId, data),
    onSuccess: (_, { userId }) => {
      queryClient.invalidateQueries({ queryKey: ['profile', userId] });
    }
  });
};
```

```typescript
// lib/hooks/useMatching.ts
export const useMatchingFeed = (filters: MatchFilters) => {
  const { user } = useAuthStore();

  return useInfiniteQuery({
    queryKey: ['matching-feed', filters],
    queryFn: ({ pageParam = 0 }) =>
      profilesApi.getMatchingFeed(user!.id, { ...filters, offset: pageParam }),
    getNextPageParam: (lastPage, pages) =>
      lastPage.length === 20 ? pages.length * 20 : undefined,
    enabled: !!user
  });
};
```

## 3. Component Patterns

### Container/Presenter 패턴

```typescript
// pages/main/FeedPage.tsx (Container)
const FeedPage = () => {
  const { data, fetchNextPage, hasNextPage } = useMatchingFeed(filters);
  const { mutate: sendLike } = useLikeAction();
  const { mutate: sendPass } = usePassAction();

  const handleLike = (targetId: string) => {
    sendLike({ targetId });
  };

  const handlePass = (targetId: string) => {
    sendPass({ targetId });
  };

  return (
    <FeedPresenter
      profiles={data?.pages.flat() ?? []}
      onLike={handleLike}
      onPass={handlePass}
      onLoadMore={fetchNextPage}
      hasMore={hasNextPage}
    />
  );
};

// components/features/matching/FeedPresenter.tsx (Presenter)
const FeedPresenter = ({ profiles, onLike, onPass, onLoadMore, hasMore }) => {
  return (
    <Screen>
      <Header title="매칭" />
      <div className="feed-container">
        {profiles.map(profile => (
          <ProfileCard
            key={profile.id}
            profile={profile}
            onLike={() => onLike(profile.id)}
            onPass={() => onPass(profile.id)}
          />
        ))}
      </div>
      <BottomNav />
    </Screen>
  );
};
```

### Compound Component 패턴

```typescript
// components/ui/Card/index.tsx
const CardContext = createContext<{ variant: string }>({ variant: 'default' });

const Card = ({ children, variant = 'default' }) => (
  <CardContext.Provider value={{ variant }}>
    <div className={`card card--${variant}`}>{children}</div>
  </CardContext.Provider>
);

Card.Header = ({ children }) => (
  <div className="card-header">{children}</div>
);

Card.Body = ({ children }) => (
  <div className="card-body">{children}</div>
);

Card.Footer = ({ children }) => (
  <div className="card-footer">{children}</div>
);

Card.Image = ({ src, alt }) => (
  <img className="card-image" src={src} alt={alt} loading="lazy" />
);

export { Card };

// 사용 예시
<Card variant="profile">
  <Card.Image src={profile.photo} alt={profile.nickname} />
  <Card.Body>
    <h3>{profile.nickname}, {profile.age}</h3>
    <p>{profile.region}</p>
  </Card.Body>
  <Card.Footer>
    <Button onClick={onPass}>패스</Button>
    <Button onClick={onLike} variant="primary">좋아요</Button>
  </Card.Footer>
</Card>
```

### Custom Hook 패턴

```typescript
// lib/hooks/useSwipeGesture.ts
export const useSwipeGesture = (onSwipeLeft: () => void, onSwipeRight: () => void) => {
  const [touchStart, setTouchStart] = useState<number | null>(null);
  const [touchEnd, setTouchEnd] = useState<number | null>(null);

  const minSwipeDistance = 50;

  const onTouchStart = (e: TouchEvent) => {
    setTouchEnd(null);
    setTouchStart(e.targetTouches[0].clientX);
  };

  const onTouchMove = (e: TouchEvent) => {
    setTouchEnd(e.targetTouches[0].clientX);
  };

  const onTouchEnd = () => {
    if (!touchStart || !touchEnd) return;

    const distance = touchStart - touchEnd;
    const isLeftSwipe = distance > minSwipeDistance;
    const isRightSwipe = distance < -minSwipeDistance;

    if (isLeftSwipe) onSwipeLeft();
    if (isRightSwipe) onSwipeRight();
  };

  return { onTouchStart, onTouchMove, onTouchEnd };
};
```

## 4. 모바일 웹앱 최적화 패턴

### Safe Area 대응

```css
/* styles/globals.css */
:root {
  --safe-area-top: env(safe-area-inset-top);
  --safe-area-bottom: env(safe-area-inset-bottom);
  --safe-area-left: env(safe-area-inset-left);
  --safe-area-right: env(safe-area-inset-right);
}

.screen {
  padding-top: var(--safe-area-top);
  padding-bottom: var(--safe-area-bottom);
}

.bottom-nav {
  padding-bottom: calc(var(--safe-area-bottom) + 8px);
}
```

### Pull-to-Refresh 패턴

```typescript
// lib/hooks/usePullToRefresh.ts
export const usePullToRefresh = (onRefresh: () => Promise<void>) => {
  const [isRefreshing, setIsRefreshing] = useState(false);
  const [pullDistance, setPullDistance] = useState(0);
  const startY = useRef(0);

  const handleTouchStart = (e: TouchEvent) => {
    if (window.scrollY === 0) {
      startY.current = e.touches[0].clientY;
    }
  };

  const handleTouchMove = (e: TouchEvent) => {
    if (startY.current === 0) return;
    const currentY = e.touches[0].clientY;
    const distance = Math.max(0, currentY - startY.current);
    setPullDistance(Math.min(distance, 100));
  };

  const handleTouchEnd = async () => {
    if (pullDistance > 60) {
      setIsRefreshing(true);
      await onRefresh();
      setIsRefreshing(false);
    }
    setPullDistance(0);
    startY.current = 0;
  };

  return { isRefreshing, pullDistance, handlers: { handleTouchStart, handleTouchMove, handleTouchEnd } };
};
```

### Skeleton Loading 패턴

```typescript
// components/ui/Skeleton/index.tsx
const Skeleton = ({ width, height, variant = 'rect' }) => (
  <div
    className={`skeleton skeleton--${variant}`}
    style={{ width, height }}
  />
);

// components/features/profile/ProfileCardSkeleton.tsx
const ProfileCardSkeleton = () => (
  <div className="profile-card">
    <Skeleton variant="rect" width="100%" height={400} />
    <div className="profile-card-body">
      <Skeleton width={120} height={24} />
      <Skeleton width={80} height={16} />
    </div>
  </div>
);

// 사용 예시
const FeedPage = () => {
  const { data, isLoading } = useMatchingFeed();

  if (isLoading) {
    return <ProfileCardSkeleton />;
  }

  return <ProfileCard profile={data} />;
};
```

### Optimistic Update 패턴

```typescript
// lib/hooks/useLikeAction.ts
export const useLikeAction = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (targetId: string) => matchingApi.sendLike(targetId),

    // 낙관적 업데이트
    onMutate: async (targetId) => {
      await queryClient.cancelQueries({ queryKey: ['matching-feed'] });

      const previousFeed = queryClient.getQueryData(['matching-feed']);

      // 즉시 UI에서 제거
      queryClient.setQueryData(['matching-feed'], (old: any) => ({
        ...old,
        pages: old.pages.map((page: any[]) =>
          page.filter(p => p.id !== targetId)
        )
      }));

      return { previousFeed };
    },

    // 에러 시 롤백
    onError: (err, targetId, context) => {
      queryClient.setQueryData(['matching-feed'], context?.previousFeed);
    }
  });
};
```

### 이미지 최적화 패턴

```typescript
// components/ui/OptimizedImage/index.tsx
const OptimizedImage = ({ src, alt, width, height, priority = false }) => {
  const [isLoaded, setIsLoaded] = useState(false);
  const [error, setError] = useState(false);

  // Supabase Storage transform으로 리사이징
  const optimizedSrc = src.includes('supabase')
    ? `${src}?width=${width}&height=${height}&quality=80`
    : src;

  return (
    <div className="image-container" style={{ width, height }}>
      {!isLoaded && <Skeleton width={width} height={height} />}
      <img
        src={error ? '/images/placeholder.png' : optimizedSrc}
        alt={alt}
        loading={priority ? 'eager' : 'lazy'}
        onLoad={() => setIsLoaded(true)}
        onError={() => setError(true)}
        style={{ opacity: isLoaded ? 1 : 0 }}
      />
    </div>
  );
};
```

## 5. 에러 처리 패턴

### Error Boundary

```typescript
// components/ErrorBoundary.tsx
class ErrorBoundary extends React.Component<Props, State> {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return (
        <Screen>
          <div className="error-container">
            <h2>문제가 발생했습니다</h2>
            <p>잠시 후 다시 시도해주세요</p>
            <Button onClick={() => window.location.reload()}>
              새로고침
            </Button>
          </div>
        </Screen>
      );
    }

    return this.props.children;
  }
}
```

### API 에러 처리

```typescript
// lib/utils/errorHandler.ts
export const handleApiError = (error: unknown): string => {
  if (error instanceof AuthError) {
    if (error.message.includes('Invalid login')) {
      return '인증 정보가 올바르지 않습니다';
    }
    return '인증에 실패했습니다. 다시 로그인해주세요';
  }

  if (error instanceof PostgrestError) {
    if (error.code === '23505') {
      return '이미 존재하는 데이터입니다';
    }
    return '데이터 처리 중 오류가 발생했습니다';
  }

  return '알 수 없는 오류가 발생했습니다';
};
```

## 6. 라우팅 & 네비게이션 패턴

### Protected Route

```typescript
// app/Router.tsx
const ProtectedRoute = ({ children }: { children: React.ReactNode }) => {
  const { session, isLoading } = useAuthStore();
  const location = useLocation();

  if (isLoading) {
    return <LoadingScreen />;
  }

  if (!session) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return <>{children}</>;
};

// 라우터 설정
const router = createBrowserRouter([
  {
    path: '/',
    element: <ProtectedRoute><MainLayout /></ProtectedRoute>,
    children: [
      { index: true, element: <Navigate to="/feed" replace /> },
      { path: 'feed', element: <FeedPage /> },
      { path: 'chat', element: <ChatPage /> },
      { path: 'chat/:roomId', element: <ChatRoomPage /> },
      { path: 'matches', element: <MatchesPage /> },
      { path: 'profile', element: <ProfilePage /> },
    ]
  },
  { path: '/login', element: <LoginPage /> },
  { path: '/signup/*', element: <SignupPage /> },
  { path: '/onboarding/*', element: <OnboardingPage /> },
]);
```

### Bottom Navigation 패턴

```typescript
// components/layout/BottomNav.tsx
const navItems = [
  { path: '/feed', icon: <HomeIcon />, label: '홈' },
  { path: '/chat', icon: <ChatIcon />, label: '채팅', badge: 'unreadCount' },
  { path: '/matches', icon: <HeartIcon />, label: '매칭' },
  { path: '/profile', icon: <UserIcon />, label: '프로필' },
];

const BottomNav = () => {
  const location = useLocation();
  const unreadCount = useChatStore(state => state.unreadCount);

  return (
    <nav className="bottom-nav">
      {navItems.map(item => (
        <NavLink
          key={item.path}
          to={item.path}
          className={({ isActive }) =>
            `bottom-nav-item ${isActive ? 'active' : ''}`
          }
        >
          {item.icon}
          {item.badge && unreadCount > 0 && (
            <span className="badge">{unreadCount}</span>
          )}
          <span className="label">{item.label}</span>
        </NavLink>
      ))}
    </nav>
  );
};
```

---

# Database Schema (Supabase)

```sql
-- 사용자 프로필
create table profiles (
  id uuid references auth.users primary key,
  nickname text not null,
  gender text not null check (gender in ('male', 'female')),
  birth_date date not null,
  region text not null,
  job_type text,
  company text,
  education text,
  height integer,
  drinking text,
  smoking text,
  religion text,
  interests text[],
  bio text,
  photos text[],
  clover_score integer default 0,
  is_verified boolean default false,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- 매칭 액션 (좋아요/패스)
create table match_actions (
  id uuid primary key default gen_random_uuid(),
  from_user_id uuid references profiles(id) not null,
  to_user_id uuid references profiles(id) not null,
  action text not null check (action in ('like', 'pass')),
  created_at timestamptz default now(),
  unique(from_user_id, to_user_id)
);

-- 매칭 (양쪽 좋아요)
create table matches (
  id uuid primary key default gen_random_uuid(),
  user1_id uuid references profiles(id) not null,
  user2_id uuid references profiles(id) not null,
  matched_at timestamptz default now(),
  unique(user1_id, user2_id)
);

-- 채팅방
create table chat_rooms (
  id uuid primary key default gen_random_uuid(),
  match_id uuid references matches(id) not null unique,
  created_at timestamptz default now()
);

-- 메시지
create table messages (
  id uuid primary key default gen_random_uuid(),
  room_id uuid references chat_rooms(id) not null,
  sender_id uuid references profiles(id) not null,
  content text not null,
  read_at timestamptz,
  created_at timestamptz default now()
);

-- RLS 정책 설정
alter table profiles enable row level security;
alter table match_actions enable row level security;
alter table matches enable row level security;
alter table chat_rooms enable row level security;
alter table messages enable row level security;

-- 프로필 정책: 본인만 수정, 모두 조회 가능
create policy "Profiles viewable by all" on profiles
  for select using (true);

create policy "Users can update own profile" on profiles
  for update using (auth.uid() = id);

-- 메시지 정책: 채팅방 참여자만 조회/작성
create policy "Messages viewable by room participants" on messages
  for select using (
    exists (
      select 1 from chat_rooms cr
      join matches m on cr.match_id = m.id
      where cr.id = messages.room_id
      and (m.user1_id = auth.uid() or m.user2_id = auth.uid())
    )
  );
```

---

---

# MCP Servers

## Figma MCP Server Rules

### Component Organization

- IMPORTANT: 모든 재사용 가능한 UI 컴포넌트는 `components/DesignSystem.jsx`에 정의
- 화면(Screen) 컴포넌트는 `components/` 디렉토리에 별도 파일로 생성
- 컴포넌트는 `window.*` 또는 `window.CloverUI.*`로 전역 노출

### Existing Component Library (CloverUI)

다음 컴포넌트들이 `CloverUI` 네임스페이스에 이미 존재합니다:

**Layout & Navigation:**
- `Header` - 헤더 (뒤로가기 버튼, 타이틀)
- `ProgressBar` - 진행률 표시
- `BottomButton` - 하단 고정 버튼

**Input Components:**
- `BorderInput` - 하단 보더 스타일 입력 (메인 입력)
- `BoxInput` - 박스 스타일 입력 (생년월일 등)
- `BorderTextarea` - 하단 보더 스타일 텍스트영역
- `SearchInput` - 검색 입력

**Selection Components:**
- `SelectionList` - 라디오 스타일 선택 리스트
- `ChipGroup` - 태그 그룹 (다중 선택)
- `InterestChip` - 관심사 칩
- `CardOptionGrid` - 카드형 옵션 그리드
- `SelectDropdown` - 드롭다운 셀렉트

**Data Display:**
- `SearchResultItem` - 검색 결과 항목
- `NoticeBox` - 안내 박스
- `HelperText` - 도움말 텍스트

**Form Components:**
- `PhotoUploadGrid` - 사진 업로드 그리드
- `AgreementList` - 약관 동의 리스트

**Icons:**
- `Icons.Back`, `Icons.Check`, `Icons.CheckSmall`
- `Icons.ChevronRight`, `Icons.Search`, `Icons.Plus`, `Icons.Clover`

### Design Tokens

IMPORTANT: 색상값을 절대 하드코딩하지 마세요. `styles.css`에 정의된 CSS 변수를 사용하세요.

**Colors:**
```css
--primary-1: #cef17b      /* 프라이머리 (라임 그린) */
--black: #000000
--white: #ffffff
--gray-1: #1a1a1a         /* 텍스트 진한 회색 */
--gray-2: #a09f9f         /* 보조 텍스트 */
--gray-3: #d8d7d7         /* 테두리 */
--gray-4: #f2f2f2         /* 배경 연한 회색 */
--gray-5: #f9f9f9         /* 배경 매우 연한 회색 */
--placeholder: #cccccc
--helper: #999999
--border-default: #e0e0e0
--border-light: #f5efea
--selected-bg: #e5dcd2
--error: #ff4d4d
--bg-gradient: linear-gradient(180deg, #fffdf8 0%, #ffffff 100%)
```

**Typography:**
```css
--font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, system-ui, sans-serif
```

**Transitions:**
```css
--transition-fast: 150ms ease
--transition-normal: 250ms ease
```

### Figma Implementation Flow

IMPORTANT: 다음 순서를 반드시 따르세요:

1. `get_design_context`를 먼저 실행하여 노드 구조 확인
2. `get_screenshot`으로 시각적 참조 확인
3. 필요한 에셋이 있다면 다운로드
4. 구현 시작 - 아래 규칙을 적용

### Styling Rules

- IMPORTANT: Tailwind 클래스를 사용하지 마세요. 이 프로젝트는 Vanilla CSS를 사용합니다.
- 모든 스타일은 `styles.css`에 정의된 클래스와 CSS 변수를 사용
- 인라인 스타일은 최소화하고, 필요시 CSS 변수 참조
- 새 컴포넌트 스타일은 `styles.css`에 추가

**Screen Layout Pattern:**
```jsx
<div className="screen">
  <CloverUI.Header title="화면 제목" onBack={handleBack} />
  <CloverUI.ProgressBar current={step} total={totalSteps} />
  <div className="screen-content">
    {/* 콘텐츠 */}
  </div>
  <CloverUI.BottomButton text="다음" isActive={isValid} onClick={handleNext} />
</div>
```

**Question Screen Pattern:**
```jsx
<div className="slide-up">
  <h2 className="question-title">질문 텍스트<br/>두 번째 줄</h2>
  {/* 입력 컴포넌트 */}
  <CloverUI.HelperText>도움말 텍스트</CloverUI.HelperText>
</div>
```

### Component Creation Rules

1. 새 UI 컴포넌트는 `components/DesignSystem.jsx`에 추가하고 `window.CloverUI`에 등록
2. 새 화면 컴포넌트는 `components/` 디렉토리에 별도 파일로 생성
3. 화면 컴포넌트는 `window.*Screen`으로 전역 등록
4. Props 패턴 준수:
   - `onComplete` - 화면 완료 콜백
   - `onBack` - 뒤로가기 콜백
   - `value`, `onChange` - 상태 제어
   - `isActive`, `onClick` - 버튼 활성화

### Asset Handling

- IMPORTANT: Figma MCP 서버가 localhost 소스를 반환하면 해당 소스를 직접 사용
- IMPORTANT: 새 아이콘 패키지를 설치하지 마세요. SVG 아이콘은 `Icons` 객체에 추가
- 이미지 에셋은 `screenshots/` 디렉토리에 저장

### Mobile Frame

- 모든 화면은 390x844px 모바일 프레임 기준
- `.mobile-frame` 클래스가 컨테이너 역할
- 스크롤 가능한 콘텐츠는 `.screen-content`에 배치

### Animation Classes

```css
.fade-in   /* 페이드 인 */
.slide-up  /* 슬라이드 업 */
```

### Code Style

- 한글 주석 사용
- 함수형 컴포넌트만 사용
- React Hooks (useState) 사용
- Arrow function 컴포넌트 형식

```jsx
const ComponentName = ({ prop1, prop2 }) => {
  const [state, setState] = React.useState(initialValue);

  return (
    <div className="component-class">
      {/* 내용 */}
    </div>
  );
};

window.ComponentName = ComponentName;
```

### Validation Rules

구현 완료 전 체크리스트:
1. 기존 `CloverUI` 컴포넌트를 최대한 재사용했는지 확인
2. 색상이 CSS 변수로 참조되는지 확인
3. 화면 레이아웃이 `.screen` 패턴을 따르는지 확인
4. Figma 스크린샷과 1:1 시각적 일치 확인
5. 반응형 동작 (모바일 프레임 내) 확인

---
> Source: [marvinsong-cell/clover-wireframe](https://github.com/marvinsong-cell/clover-wireframe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
