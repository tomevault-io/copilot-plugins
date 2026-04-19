## youcra-v03-new

> - **이름**: YouCra v03 (유크라)

# YouCra v03 (유크라) 프로젝트 컨텍스트

## 프로젝트 개요
- **이름**: YouCra v03 (유크라)
- **유형**: React 기반 실시간 채팅 + YouTube 통합 소셜 플랫폼
- **목적**: YouTube 영상 공유와 실시간 채팅이 결합된 크리에이터 팬 커뮤니티 앱
- **도메인**: ucrachat.com (Vercel 배포)

## 기술 스택
- **Frontend**: React 18.2.0, Tailwind CSS, Framer Motion
- **Backend**: Firebase (Firestore, Auth, Storage, Analytics)
- **API**: YouTube Data API v3, Google Identity Services
- **UI**: React Icons, React Modal, React YouTube
- **Testing**: Jest, React Testing Library
- **Deployment**: Vercel

## 핵심 기능
1. **사용자 인증**: Firebase Auth + Google OAuth 2.0 이중 인증
2. **실시간 채팅**: Firestore 실시간 리스너, 이미지 업로드, 이모지
3. **YouTube 통합**: 영상 공유/재생/좋아요/구독, 팬 인증 시스템
4. **소셜 기능**: 프로필 관리, 마이채널, 1:1 DM, 인기 리포트

## 주요 컴포넌트
- `Home.js`: YouTube 피드, 실시간 검색어, 팬 인증
- `ChatList.js`: 채팅방 목록, "내 채팅방"/"전체" 탭
- `ChatRoom.js`: 실시간 채팅, YouTube 영상 공유, 팝업 플레이어
- `MyChannel.js`: 프로필 편집, 내 영상 방 목록, 채널 링크
- `GoogleAuth.js`: Google Identity Services 구현, OAuth 처리
- `AuthContext.js`: 전역 인증 상태 관리

## 환경변수
```
REACT_APP_GOOGLE_CLIENT_ID=234134940808-jtnudlrb28o2hheq2ppmfr4rocgn09pj.apps.googleusercontent.com
REACT_APP_YOUTUBE_API_KEY=[YouTube API 키]
```

## 라우팅
- `/`: 홈 (YouTube 피드)
- `/chat`: 채팅방 목록
- `/chat/:roomId`: 채팅방
- `/my`: 마이채널
- `/report`: 인기 리포트
- `/login`: 로그인
- `/profile/:roomId/:uid`: 사용자 프로필
- `/dm/:uid`: 1:1 DM
- `/videos`: 영상 목록
- `/add-video`: 영상 추가

## 서비스 레이어
- `userService.js`: 사용자 프로필 CRUD, 포인트 시스템
- `chatService.js`: 채팅방 관리, 실시간 메시지 구독
- `videoService.js`: YouTube API 통합

## Firebase 구조
```
Firestore Collections:
├── chatRooms/
│   ├── {roomId}/
│   │   ├── messages/
│   │   ├── participants/
│   │   └── videos/
├── users/
└── reports/
```

## 알려진 이슈
- Google OAuth 로그인에서 가끔 "unregistered_origin" 오류 발생
- 해결책: Google Cloud Console에서 OAuth 클라이언트 설정 확인

## 개발 명령어
- `npm start`: 개발 서버 실행 (localhost:3000)
- `npm run build`: 프로덕션 빌드
- `npm test`: 테스트 실행

## 중요한 파일들
- `src/firebase.js`: Firebase 설정
- `src/contexts/AuthContext.js`: 인증 상태 관리
- `src/components/GoogleAuth.js`: Google 로그인 구현
- `package.json`: 의존성 및 스크립트 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ericsone1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
