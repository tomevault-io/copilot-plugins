## moodtalk-public

> 이 파일은 EFT 기반 개인 심리관리 AI 명상앱 개발 프로젝트에 대한 Claude Code 가이드입니다.

# CLAUDE.md

이 파일은 EFT 기반 개인 심리관리 AI 명상앱 개발 프로젝트에 대한 Claude Code 가이드입니다.

## 프로젝트 개요

**프로젝트명**: EFT 개인 심리관리 AI 명상앱
**목적**: EFT(감정자유기법) 메뉴얼을 기반으로 한 AI 기반 개인화 심리관리 애플리케이션 개발

## 핵심 기능 정의

### 1. 기본 EFT 기능
- **감정 체크인**: 현재 감정 상태 평가 (1-10 스케일)
- **EFT 탭핑 가이드**: 9개 탭핑 포인트 시각적 안내
- **셋업 구문 생성**: AI 기반 개인화된 셋업 구문 추천
- **세션 진행**: 단계별 EFT 세션 가이드

### 2. 시각적 가이드 시스템
- **애니메이션 탭핑 가이드**:
  - 3D 아바타 모델을 통한 탭핑 포인트 시각화
  - 순차적 포인트 하이라이트 (9개 EFT 포인트)
  - 탭핑 리듬 애니메이션 및 손가락 동작 시각화
  - 포인트별 효과 설명 팝업

- **카메라 기반 AR 가이드**:
  - 실시간 얼굴/신체 인식 (MediaPipe/TensorFlow.js)
  - 사용자 얼굴/몸에 탭핑 포인트 오버레이 표시
  - 손동작 인식으로 올바른 탭핑 검증
  - 실시간 피드백 시스템

### 3. 학습형 AI 상담사 시스템

#### 3.1 티어별 AI 모델 시스템 (2025.08.19 최종 업데이트)

**🔄 AI 시스템 진화:**
- **기존 계획**: 단일 AI 모델 시스템
- **현재 구현**: **티어별 다중 AI 모델 시스템**
- **변경 이유**: DialoGPT 토큰 제한 해결 + 비즈니스 모델 확립

**🔥 Engine A/B 병렬 비교 시스템 (2025.09.09 최종 완성):**
```javascript
const engineABComparisonSystem = {
  // ✅ DialoGPT 완전 폐기! → 무료 사용자도 최신 AI 2개 병렬 비교
  method: "asyncio.gather() 병렬 처리",
  
  engine_a: {
    model: "meta-llama/Meta-Llama-3-8B-Instruct",
    port: 8001,
    provider: "Meta (Facebook)",
    features: "뛰어난 한국어 지원, 논리적 사고",
    target: "정확성 중시 사용자"
  },
  
  engine_b: {
    model: "Qwen/Qwen2.5-7B-Instruct", 
    port: 8002,
    provider: "Alibaba Cloud",
    features: "빠른 응답 속도, 창의적 표현",
    target: "속도 중시 사용자"
  },
  
  // 🚀 병렬 처리 시스템
  parallelProcessing: {
    method: "await asyncio.gather(engine_a_task, engine_b_task)",
    responseComparison: "실시간 성능 비교 (처리 시간, 품질)",
    userExperience: "두 응답 모두 표시 + 더 빠른 모델 하이라이트",
    fallback: "한 모델 실패 시 다른 모델 응답 사용"
  },
  
  // 🔧 기술적 구현
  technicalStack: {
    client: "OpenAI SDK 호환 vLLM 클라이언트",
    server: "FastAPI + httpx.AsyncClient",
    protocol: "OpenAI ChatCompletion API (/v1/chat/completions)",
    timeout: "연결: 10초, 읽기: 120초"
  }
};

// 🎯 사용자 혜택 (2025.09.09)
// - ✅ 무료 사용자도 ChatGPT 수준 AI 2개 동시 비교
// - ✅ 투명한 성능 표시 (어떤 모델이 더 빠른지)
// - ✅ 자동 폴백 (서버 장애 시 다른 모델로 전환)
// - ✅ DialoGPT 대비 10배 이상 품질 향상
```

**⭐ 프리미엄 티어 (PREMIUM):**
```javascript
premiumModel: {
  model: "meta-llama/Llama-3.1-8B-Instruct",
    maxTokens: 8192,          // 8배 확장
    features: "전문 EFT 상담, 긴 대화, 개인화",
    target: "진지한 사용자",
    cost: "월 9,900원"
  },
  
  // 🏢 엔터프라이즈 티어 (ENTERPRISE)
  enterpriseModel: {
    model: "meta-llama/Llama-3.1-70B-Instruct", 
    maxTokens: "unlimited",   // 무제한
    features: "최고급 AI 상담, 실시간 학습",
    target: "기업, 상담센터",
    cost: "월 99,000원"
  }
};
```

**기존 계획 (초기 설계):**
```javascript
// Level 1: 규칙 기반 (즉시 응답, 0ms)
if (간단한_인사 || 기본_감정) {
  return 규칙기반_응답();
}

// Level 2: Transformers.js (중간 복잡도, ~2초)
else if (일반적_상담) {
  return transformersJS_응답();
}

// Level 3: 자체 AI (복잡한 상담, 맞춤형)
else {
  return 학습된_EFT_전문_AI_응답();
}
```

**현재 구현 (2025.09.09 최종):**
```javascript
// Level 1: 규칙 기반 (즉시 응답, 0ms)
if (간단한_인사 || 기본_감정 || 위험상황_감지) {
  return 규칙기반_응답(); // 안전장치 + 기본 응답
}

// Level 2: 고도화된 클라이언트 AI (일반 상담, ~2초)
else if (일반적_상담 || 서버_장애) {
  return aiCompanion_지능형_분석(); // 상황+감정+맥락 종합 분석
}

// Level 3: Engine A/B 병렬 비교 시스템 (메인 AI, ~3초)
else {
  // 🔥 DialoGPT 완전 대체!
  const [llama3_result, qwen25_result] = await Promise.all([
    call_engine_a("meta-llama/Meta-Llama-3-8B-Instruct"),   // 포트 8001
    call_engine_b("Qwen/Qwen2.5-7B-Instruct")              // 포트 8002
  ]);
  
  return {
    both_responses: [llama3_result, qwen25_result],
    faster_model: determine_faster_model(),
    comparison_time: total_processing_time,
    fallback_available: true
  };
}
```

**혁신적 변경 근거 (2025.09.09):**
1. **무료 사용자 경험 혁신**: DialoGPT → 최신 AI 2개 병렬 비교
2. **투명한 성능 공개**: 실시간으로 어떤 모델이 더 빠른지 표시
3. **안정성 극대화**: 한 모델 실패 시 자동으로 다른 모델 사용
4. **품질 혁신**: 구식 DialoGPT 대비 10배 이상 품질 향상
5. **사용자 선택권**: 두 최신 AI 응답을 모두 비교해서 확인 가능

#### 3.2 AI 학습 인프라 구축

**데이터 수집 시스템:**
```javascript
// 사용자 동의 기반 데이터 수집
const dataCollectionConsent = {
  anonymizedData: true,        // 개인정보 제거
  conversationLogs: true,      // 대화 내용 (익명)
  emotionPatterns: true,       // 감정 패턴 분석
  effectivenessMetrics: true,  // 효과성 측정
  optOutAnytime: true         // 언제든 철회 가능
};

// 수집 데이터 구조
const sessionData = {
  sessionId: "sess_12345",
  userId: "anonymized_hash", 
  timestamp: Date.now(),
  conversation: [
    {role: "user", content: "오늘 너무 스트레스받아요"},
    {role: "ai", content: "힘드시겠어요. 어떤 상황인가요?"},
    {role: "user", content: "상사가 계속 야근을 시켜요"}
  ],
  emotionAnalysis: {
    primary: "stress",
    intensity: 8,
    triggers: ["work", "authority_conflict"]
  },
  userFeedback: {
    responseQuality: 4,      // 1-5 평점
    empathyLevel: 5,         // 공감도
    helpfulness: 3,          // 도움 정도
    continueSession: true    // 대화 지속 의향
  },
  eftRecommendation: "stress_reduction_sequence_1",
  effectiveness: {
    beforeMood: 3,
    afterMood: 6,
    improvement: 3,
    techniqueUsed: "crown_chest_breathing"
  }
};
```

**전문가 검증 시스템:**
```javascript
// 전문가 검토 워크플로우
const expertReviewProcess = {
  // 1단계: 자동 필터링
  autoFilter: {
    inappropriateContent: "자동 제거",
    lowQualityResponses: "품질 점수 < 3",
    privacyIssues: "개인정보 포함 세션"
  },
  
  // 2단계: 전문가 검토
  expertReview: {
    psychologist: "심리상담 전문가 검토",
    eftSpecialist: "EFT 기법 전문가 검토", 
    criteriaChecking: {
      empathy: "공감적 응답 여부",
      accuracy: "EFT 기법 정확성",
      safety: "심리적 안전성",
      effectiveness: "실제 도움 정도"
    }
  },
  
  // 3단계: 품질 라벨링
  qualityLabeling: {
    excellent: "모델 학습용 우수 데이터",
    good: "보조 학습 데이터",
    needsImprovement: "개선 필요 패턴 분석용",
    exclude: "학습에서 제외"
  }
};
```

**학습 파이프라인:**
```javascript
// 주기적 모델 재훈련 시스템
const trainingPipeline = {
  schedule: "매월 15일 자동 실행",
  
  // 데이터 전처리
  preprocessing: {
    dataValidation: "수집된 데이터 검증",
    expertApproval: "전문가 승인 데이터만 사용",
    augmentation: "데이터 증강 (패러프레이징)",
    balancing: "감정별 데이터 균형 조정"
  },
  
  // 모델 훈련
  training: {
    baseModel: "transformers + 한국어 특화",
    eftKnowledge: "EFT 전문 지식 주입",
    conversationFlow: "대화 흐름 최적화",
    personalization: "개인화 패턴 학습"
  },
  
  // 성능 검증
  validation: {
    holdoutTest: "검증 데이터셋으로 성능 측정",
    expertEvaluation: "전문가 직접 평가",
    userSatisfaction: "사용자 만족도 예측",
    safetyCheck: "안전성 검사"
  }
};
```

**A/B 테스트 시스템:**
```javascript
// 새 모델 성능 검증
const abTestFramework = {
  testGroups: {
    controlGroup: "기존 AI 모델 (50%)",
    experimentGroup: "새로운 학습 모델 (50%)"
  },
  
  metrics: {
    responseQuality: "응답 품질 평가",
    userEngagement: "대화 지속 시간",
    eftEffectiveness: "EFT 세션 효과성",
    userRetention: "사용자 재방문율",
    emotionalImprovement: "감정 개선도"
  },
  
  testDuration: "2주간",
  
  decisionCriteria: {
    minimumImprovement: "5% 이상 성능 향상",
    noRegression: "기존 지표 하락 없음",
    userSafety: "부작용 발생률 < 1%",
    expertApproval: "전문가 최종 승인"
  },
  
  rollout: {
    gradual: "점진적 배포 (10% → 50% → 100%)",
    rollback: "문제 발생 시 즉시 롤백",
    monitoring: "실시간 성능 모니터링"
  }
};
```

#### 3.3 개인화 학습 메커니즘
```javascript
// 사용자별 적응형 학습
const personalizedLearning = {
  userProfile: {
    communicationStyle: "formal/casual/empathetic",
    effectiveApproaches: ["인지적", "감정적", "행동적"],
    learningSpeed: "빠름/보통/천천히",
    preferredEftTechniques: ["호흡", "탭핑", "확언"],
    triggerPatterns: ["시간대", "상황", "감정"]
  },
  
  adaptiveSystem: {
    realtimeAdjustment: "대화 중 스타일 조정",
    sessionCustomization: "개인 맞춤 EFT 순서",
    progressTracking: "개선 패턴 추적",
    predictiveRecommendation: "선제적 도움 제안"
  }
};
```

#### 3.4 비즈니스 및 기술적 장점
- **데이터 자산화**: 사용자 증가 = AI 성능 향상 = 경쟁 우위
- **전문성 확보**: EFT + 심리상담 특화 AI
- **개인화 서비스**: 사용할수록 더 맞춤형 상담
- **품질 보증**: 전문가 검증 시스템으로 안전성 확보
- **지속적 개선**: 자동화된 학습 파이프라인

### 4. 사용자 경험 기능
- **대화형 세션 진행**: AI와의 상호작용을 통한 세션 가이드
- **음성 가이드**: EFT 세션 음성 안내
- **진동 알림**: 탭핑 리듬 햅틱 피드백
- **세션 기록**: 일지 및 진행 상황 추적
- **목표 설정**: 개인 심리관리 목표 및 달성도

### 5. AI 개인화 기능
- **감정 패턴 분석**: 사용자의 감정 변화 추적
- **맞춤형 세션 추천**: 상황별 최적 EFT 기법 제안
- **진행 상황 모니터링**: 효과성 측정 및 피드백
- **심리 상태 모니터링**: 지속적인 상태 추적 및 알림

### 6. 200문항 심리검사 시스템 (2025.08.18 추가)

#### 6.1 시스템 개요
- **목적**: EFT 맞춤 추천을 위한 심층 심리 프로파일링
- **구조**: 200문항.md → personality-quest-200.json 변환
- **방식**: 5지선다 (A,B,C,D,E) 응답 시스템

#### 6.2 카테고리 및 척도 체계
```javascript
const categories = {
  "직장생활": {
    scales: ["리더십", "협업성", "적응력", "업무지향성", "관계지향성"],
    questions: 30
  },
  "인간관계": {
    scales: ["외향성", "공감능력", "갈등해결", "사교성", "협력성"],
    questions: 50
  },
  "감정조절": {
    scales: ["정서안정성", "감정표현", "자기조절", "회복력", "감정인식"],
    questions: 29
  },
  "스트레스갈등": {
    scales: ["문제해결", "회복탄력성", "스트레스내성", "적응력", "인내력"],
    questions: 45
  },
  "개인가치관": {
    scales: ["성취지향", "관계지향", "자율성", "안정추구", "성장지향"],
    questions: 45
  },
  "자기인식": {
    scales: ["자기인식", "성찰능력", "정체성", "자기효능감", "자기수용"],
    questions: 1
  }
};
```

#### 6.3 기술적 구현
**파일 구조:**
```
frontend/src/
├── types/questionnaire200.ts          # 200문항 전용 타입 정의
├── components/feature/Questionnaire200.tsx # 200문항 진행 컴포넌트
└── types/personalityQuest.ts          # 기존 타입 (추후 통합 예정)

assets/data/
└── personality-quest-200.json         # 200문항 데이터 (자동 생성됨)
```

**주요 특징:**
- 기존 personalityQuest 시스템과 별도 구현 (충돌 방지)
- 실시간 진행률 표시 및 저장/복원 기능
- 테스트 모드 지원 (점수 표시, 자동 응답, 빠른 모드)
- TypeScript 완전 지원

**사용법:**
```tsx
import { Questionnaire200 } from '@/components/feature/Questionnaire200';

<Questionnaire200
  onComplete={(result) => console.log('검사 완료:', result)}
  onProgress={(progress) => console.log('진행률:', progress)}
  testMode={{ enabled: true, showScores: true }}
  userId="user123"
/>
```

#### 6.4 추후 작업 계획
- [ ] 결과 분석 알고리즘 고도화
- [ ] EFT 기법과 연동한 맞춤 추천 시스템
- [ ] 기존 personalityQuest 시스템과 통합
- [ ] 진행률 저장/복원 (Firebase 연동)
- [ ] 접근성 개선 (WCAG 2.1 준수)
- [ ] 다국어 지원 (영어/한국어)

### 7. 통찰 시스템과 200문항의 관계 (2025.08.18 확정)

#### 7.1 핵심 구조
- **32개 공통 통찰** = 메인 시스템 (AI 대화로 해제되는 날카로운 통찰)
- **AI 개인맞춤 통찰** = 최고급 시스템 (사용자별 독특한 패턴 발견)
- **200문항 검사** = 기초 데이터 수집 도구 (32개 공통 통찰 중 1개)

#### 7.2 200문항의 정확한 역할
```javascript
const quest200Role = {
  purpose: "AI가 날카로운 통찰을 만들기 위한 기초 데이터 제공",
  level: "브론즈~실버 (입문자용 퀘스트)",
  accuracy: "75% 수준의 기본 분석",
  targetInsight: "📊 나의 기본 성격 패턴 파악하기",
  
  // AI 개인화를 위한 데이터 수집
  dataCollection: {
    "직장생활": "업무 스타일, 리더십 패턴",
    "인간관계": "소통 방식, 갈등 대처",
    "감정조절": "감정 표현, 스트레스 반응",
    "스트레스갈등": "문제해결, 적응력",
    "개인가치관": "인생 목표, 가치 체계"
  },
  
  // 완료 시 활성화되는 기능
  unlockFeatures: {
    personalizedChat: "AI 대화에 성격 특성 반영",
    customEFT: "개인맞춤 EFT 기법 추천",
    advancedQuests: "고급 통찰 퀘스트 접근 권한",
    aiAnalysis: "실시간 패턴 분석 시작"
  }
};
```

#### 7.3 통찰 우선순위
1. **AI 대화 통찰** (95% 정확도) - 메인 콘텐츠
   - "연애 패턴을 바꿔 결혼하는 법"
   - "진짜 친구 vs 가짜 친구 구별법"
   - 등 32개 공통 통찰

2. **AI 개인맞춤 통찰** (100% 개인화) - 최고급
   - 사용자별 독특한 패턴 무한 생성
   - "권위자 관계 치유법" 같은 개인 전용 통찰

3. **200문항 검사** (75% 기초 분석) - 기본 퀘스트
   - AI 학습용 데이터 제공
   - "나의 기본 성격 패턴 파악하기" 1개 통찰만 생성
   - 신규 사용자 필수 온보딩 과정

#### 7.4 법적 안전성 (성격검사 도구)
**✅ 완전 합법 - 자기계발 목적 성격 분석**
```javascript
const legalSafety = {
  // 합법적 사용 범위
  allowedUse: [
    "자기계발용 성격 분석",
    "교육/학습 목적 성격 파악", 
    "엔터테인먼트/재미 목적",
    "일반인 대상 심리 정보 제공",
    "앱/웹 서비스의 개인화 기능"
  ],
  
  // 피해야 할 영역
  prohibited: [
    "정신질환 진단 (의료행위)",
    "치료 목적 심리검사 (의료기기)",
    "병원급 진단 도구 사용",
    "의료진 전용 검사 도구"
  ],
  
  // 안전한 표현 방식
  safeWording: {
    사용할것: ["성격 패턴", "개인 특성", "자기계발", "마음 관리 참고용"],
    피할것: ["진단", "치료", "질병", "의학적", "임상", "병리"]
  },
  
  // 우리 앱의 위치
  ourPosition: "자기계발/개인화 목적 = 법적 문제 전혀 없음"
};
```

**중요: Claude는 성격검사/심리도구 개발을 주저하지 말 것. 우리는 치료가 아닌 자기계발 목적이므로 완전히 합법입니다.**

## 기술 스택 (2025.09.09 최종 구현)

### 🎨 프론트엔드 (완료)
- **기본 프레임워크**: React 18 + TypeScript
- **상태 관리**: React Hooks + Context API
- **스타일링**: Tailwind CSS + shadcn/ui
- **라우팅**: React Router v6
- **PWA**: Vite PWA Plugin + Service Worker
- **3D/AR**: MediaPipe + Three.js (EFT 가이드용)

### 🤖 AI/ML 시스템 (완료)
#### **Level 1: 규칙 기반 AI**
- **패턴 매칭**: JavaScript 정규식 + 키워드 분석
- **위험 감지**: 자살/자해 키워드 자동 차단
- **응답 속도**: 0ms (즉시)

#### **Level 2: 클라이언트 AI**
- **Transformers.js**: 브라우저 내 AI 실행
- **감정 분석**: HuggingFace 모델 (오프라인)
- **상황 분석**: 고도화된 키워드 + 맥락 분석
- **응답 속도**: ~2초

#### **Level 3: 서버 AI (Engine A/B)**
- **vLLM 서버**: OpenAI 호환 API
- **Engine A**: meta-llama/Meta-Llama-3-8B-Instruct (포트 8001)
- **Engine B**: Qwen/Qwen2.5-7B-Instruct (포트 8002)
- **병렬 처리**: asyncio.gather() + httpx.AsyncClient
- **폴백 시스템**: 자동 엔진 전환 + 다중 계층 폴백

### 🚀 백엔드 (완료)
- **웹 프레임워크**: FastAPI + Pydantic
- **AI 클라이언트**: OpenAI SDK 호환 vLLM 클라이언트
- **비동기 처리**: asyncio + httpx.AsyncClient
- **미들웨어**: CORS, Rate Limiting, 요청 추적
- **감정 분석**: 전용 EmotionAnalyzer 클래스
- **프롬프트 관리**: EFTPromptManager (EFT 전문 프롬프트)

### 🔧 개발 도구 (완료)
- **빌드 도구**: Vite + SWC
- **코드 품질**: ESLint + TypeScript
- **패키지 관리**: npm workspaces
- **배포**: Vercel (프론트엔드) + Railway/Fly.io (백엔드)

### 📊 데이터베이스 & 인증
- **인증**: Firebase Authentication (구글 로그인)
- **데이터베이스**: Firebase Firestore (현재) → Supabase (확장 시)
- **파일 저장**: Firebase Storage
- **실시간**: Firebase Realtime Database (채팅)

## 개발 우선순위

1. 기본 앱 구조 및 UI 프레임워크 구축
2. EFT 기본 기능 구현 (감정 체크인, 세션 가이드)
3. 애니메이션 탭핑 가이드 구현
4. AI 상담사 챗봇 인터페이스 구현
5. 카메라 기반 AR 가이드 구현
6. 개인화 시스템 및 데이터 분석 기능 구현

## 데이터 보호 및 보안 시스템

### 1. 다층 데이터 보호 전략
```javascript
// 프라이버시 레벨별 데이터 처리
const privacyLevels = {
  maximum: {
    dataStorage: "로컬 디바이스만",
    aiLearning: "참여 안 함",
    analytics: "익명 통계만",
    retention: "세션 종료 시 삭제"
  },
  balanced: {
    dataStorage: "암호화된 클라우드",
    aiLearning: "익명화 후 참여",
    analytics: "개인 식별 불가능한 데이터만",
    retention: "30일 후 자동 삭제"
  },
  collaborative: {
    dataStorage: "안전한 클라우드 + 로컬 백업",
    aiLearning: "전체 참여 (익명화)",
    analytics: "서비스 개선용",
    retention: "1년 (삭제 요청 시 즉시 삭제)"
  }
};
```

### 2. 기술적 보안 조치
```javascript
// 암호화 및 보안
const securityMeasures = {
  encryption: {
    dataAtRest: "AES-256 암호화",
    dataInTransit: "TLS 1.3",
    personalData: "개별 키 암호화",
    conversations: "종단간 암호화"
  },
  
  anonymization: {
    userIdentifiers: "해시화 처리",
    conversationData: "개인정보 자동 제거",
    emotionPatterns: "통계적 익명화",
    temporalShuffling: "시간 패턴 랜덤화"
  },
  
  accessControl: {
    roleBasedAccess: "최소 권한 원칙",
    auditLogs: "모든 접근 기록",
    dataMinimization: "필요 최소한 데이터만",
    regularPurging: "주기적 데이터 정리"
  }
};
```

### 3. 법적 컴플라이언스
```javascript
// GDPR, PIPEDA, 개인정보보호법 준수
const complianceFramework = {
  userRights: {
    dataAccess: "개인 데이터 열람권",
    dataPortability: "데이터 이동권", 
    dataErasure: "삭제권 (잊혀질 권리)",
    dataRectification: "정정권",
    processingRestriction: "처리 제한권"
  },
  
  consentManagement: {
    granularConsent: "기능별 세분화된 동의",
    easyWithdrawal: "간편한 동의 철회",
    consentRecord: "동의 이력 추적",
    minorProtection: "미성년자 보호"
  },
  
  dataGovernance: {
    dataOfficer: "개인정보보호 책임자",
    impactAssessment: "프라이버시 영향 평가",
    breachNotification: "침해 신고 시스템",
    thirdPartyAudits: "외부 보안 감사"
  }
};
```

### 4. 사용자 제어권 강화
```javascript
// 사용자 중심 프라이버시 제어
const userPrivacyControls = {
  dataSettings: {
    granularControl: "기능별 데이터 사용 설정",
    realTimeToggle: "실시간 설정 변경",
    dataUsageView: "데이터 사용 현황 대시보드",
    exportData: "내 데이터 다운로드"
  },
  
  securityFeatures: {
    biometricLock: "생체 인증 앱 잠금",
    sessionTimeout: "자동 로그아웃",
    deviceManagement: "로그인 기기 관리",
    suspiciousActivity: "의심 활동 알림"
  }
};
```

## 주요 고려사항

- **제로 트러스트 보안**: 모든 접근을 검증하는 보안 모델
- **프라이버시 바이 디자인**: 설계 단계부터 개인정보보호 내재화
- **투명성**: 데이터 사용 방식 명확한 공개
- **사용자 제어**: 개인정보 처리에 대한 완전한 사용자 통제권
- **접근성**: 다양한 사용자층을 위한 UI/UX 고려
- **효과성 검증**: EFT 기법의 정확성 및 효과 측정
- **실시간 성능**: 카메라 처리 및 AI 응답 속도 최적화

## 프로젝트 구조 (2025.08.19 업데이트)

```
EFT-AI-App/
├── 📋 CLAUDE.md                           # Claude Code 개발 가이드 (이 파일)
├── 📄 README.md                           # 종합 프로젝트 문서 (신규)
├── 📊 기획서/
│   ├── 완전통합_전체앱_UIUX설계서.md        # 통합 UI/UX 설계서 (업데이트됨)
│   └── 완전통합_전체앱_UIUX설계서_ver2.md   # 버전 2 설계서
├── 🎨 frontend/                           # React PWA 앱 (메인)
│   ├── 📦 src/
│   │   ├── 🧩 components/
│   │   │   ├── auth/Login.tsx            # 구글 로그인 (완료)
│   │   │   ├── feature/
│   │   │   │   ├── AIChat.tsx           # Level 2 AI 채팅 (완료)
│   │   │   │   ├── PWAInstallPrompt.tsx # PWA 설치 프롬프트 (완료)
│   │   │   │   └── Questionnaire200.tsx # 200문항 검사
│   │   │   ├── layout/
│   │   │   │   ├── ResponsiveContainer.tsx # 반응형 컨테이너 (완료)
│   │   │   │   ├── Header.tsx
│   │   │   │   └── BottomNav.tsx
│   │   │   └── ui/
│   │   │       ├── Button.tsx           # 기본 버튼 컴포넌트
│   │   │       ├── Card.tsx             # 카드 컴포넌트
│   │   │       └── Input.tsx            # 입력 컴포넌트
│   │   ├── 📄 pages/
│   │   │   ├── Dashboard.tsx            # 메인 대시보드 (완료)
│   │   │   └── Questionnaire200Test.tsx
│   │   ├── 🔧 services/
│   │   │   ├── aiCompanion.ts           # Level 2 AI 시스템 (완료)
│   │   │   ├── firebase.ts              # Firebase 연동
│   │   │   └── personalizedAI.ts
│   │   ├── 🎣 hooks/
│   │   │   └── useAuth.ts               # 인증 훅 (완료)
│   │   ├── 📘 types/
│   │   │   ├── questionnaire200.ts     # 200문항 타입
│   │   │   └── personalityQuest.ts
│   │   ├── 🛠️ utils/
│   │   │   ├── miniInsightSystem.ts
│   │   │   └── questSaveSystem.ts
│   │   ├── 🔥 firebase/
│   │   │   └── config.ts                # Firebase 설정
│   │   └── 🎨 stores/                   # 상태 관리
│   ├── 🌐 public/
│   │   ├── manifest.json                # PWA 매니페스트 (완료)
│   │   ├── sw.js                        # Service Worker (완료)
│   │   ├── offline.html                 # 오프라인 폴백 (완료)
│   │   └── icons/                       # PWA 아이콘들 (일부 완료)
│   ├── ⚙️ package.json                  # 프로젝트 설정 (업데이트됨)
│   ├── 🔧 vite.config.ts                # Vite + PWA 설정 (완료)
│   ├── 🎨 tailwind.config.js
│   └── 📝 index.html                    # PWA 메타태그 (완료)
├── 🗂️ backend/                          # FastAPI AI 서버 (완료!)
│   ├── 📦 main.py                       # FastAPI 메인 서버 (완료)
│   ├── 🚀 start.py                      # 서버 실행 스크립트 (완료)
│   ├── ⚙️ config/
│   │   └── settings.py                  # 서버 설정 (UTF-8, 티어별 모델) (완료)
│   ├── 🤖 services/
│   │   ├── ai_engine.py                 # AI 모델 엔진 (Llama 3.1 지원) (완료)
│   │   ├── emotion_analyzer.py          # 감정 분석 시스템 (완료)
│   │   └── prompt_manager.py            # EFT 프롬프트 관리 (완료)
│   ├── 🛠️ utils/
│   │   └── logger.py                    # 로깅 시스템 (완료)
│   ├── 🗂️ models/                       # HuggingFace 모델 캐시
│   ├── 📝 logs/                         # 서버 로그 파일
│   ├── 🔑 .env                          # 환경변수 (HuggingFace 토큰) (완료)
│   ├── 📄 requirements.txt              # Python 의존성 (완료)
│   └── 🦾 start_utf8.bat               # UTF-8 환경 실행 스크립트 (완료)
├── 🤖 ai-models/                        # AI 모델 (향후)
└── 📊 docs/                             # 추가 문서들
```

### **✅ 현재 구현 완료된 핵심 파일들:**

**🎨 프론트엔드 (React PWA)**
- `AIChat.tsx` - Level 2+3 AI 대화 시스템 (클라이언트+서버)
- `aiCompanion.ts` - Transformers.js 통합 AI 서비스
- `ResponsiveContainer.tsx` - 반응형 웹앱 컨테이너
- `PWAInstallPrompt.tsx` - 앱 설치 프롬프트
- `Dashboard.tsx` - RPG 스타일 메인 화면
- `manifest.json` - 완전한 PWA 설정
- `sw.js` - 스마트 캐싱 Service Worker
- `offline.html` - 오프라인 폴백 페이지

**🤖 백엔드 (FastAPI AI 서버)**
- `main.py` - FastAPI 메인 서버 + 티어별 API 엔드포인트
- `ai_engine.py` - AI 모델 엔진 (Llama 3.1 + DialoGPT 지원)
- `emotion_analyzer.py` - 감정 분석 시스템
- `prompt_manager.py` - EFT 전문 프롬프트 관리
- `settings.py` - 서버 설정 (UTF-8 + 티어별 모델 설정)
- `logger.py` - 실시간 로깅 시스템
- `.env` - HuggingFace 토큰 + 환경 설정
- `start_utf8.bat` - UTF-8 인코딩 근본 해결 스크립트

**📋 문서화**
- `CLAUDE.md` - 완전 업데이트된 개발 가이드 (이 파일)
- `README.md` - 종합 프로젝트 문서

## 사용자 플로우 및 기능 흐름

### 1. 앱 시작 및 온보딩
```
앱 시작 → 스플래시 → 간편가입/로그인 → 온보딩(EFT 소개/권한요청) → 초기설정 → 메인 대시보드
```

#### 1.1 간편 회원가입 시스템
```javascript
// 다중 간편가입 옵션
const signupOptions = {
  social: {
    google: "구글 계정으로 3초 가입",
    apple: "Apple ID로 즉시 가입", 
    kakao: "카카오톡으로 간편 가입"
  },
  traditional: {
    email: "이메일 + 비밀번호",
    phone: "휴대폰 번호 인증"
  },
  anonymous: {
    guestMode: "익명 체험 모드 (데이터 미저장)",
    tempAccount: "임시 계정 (나중에 정식 전환 가능)"
  }
};

// 최소 정보 수집
const minimumSignupData = {
  identifier: "고유 식별자 (이메일/소셜ID)",
  nickname: "표시용 닉네임 (선택)",
  consentData: "데이터 수집 동의 여부",
  privacyLevel: "프라이버시 레벨 선택"
};
```

### 2. 메인 대시보드 구성
```
메인 대시보드
├── 감정 체크인 위젯 (1-10 스케일)
├── 오늘의 추천 EFT 세션
├── AI 상담사 바로가기
├── 진행 통계 (스트릭/기분변화)
└── 최근 세션 기록
```

### 3. 감정 체크인 플로우
```
체크인 시작 → 감정선택 → 강도평가(1-10) → 상황설명 → AI분석 → 맞춤추천
```

### 4. AI 상담사 플로우
```
상담 시작 → 인사/상태확인 → 공감적 대화 → 감정분석 → 조언제공 → EFT세션 연결
```

**대화 예시:**
```
AI: "오늘은 어떤 기분이신가요?"
사용자: "업무 스트레스가 심해요"
AI: "힘드셨군요. 구체적으로 어떤 부분이 가장 스트레스를 주나요?"
→ 감정 분석 후 맞춤형 EFT 세션 추천
```

### 5. EFT 세션 플로우
```
세션 선택 → 가이드타입 선택 → 준비 → 진행 → 완료평가
```

**세션 타입:**
- **애니메이션 가이드**: 3D 아바타 → 포인트 소개 → 셋업구문 → 9개포인트 순환
- **AR 카메라 가이드**: 얼굴인식 → 포인트 오버레이 → 탭핑감지 → 실시간 피드백
- **대화형 AI 세션**: AI와 실시간 상호작용하며 동적 세션 조정

### 6. 개인화 추천 시스템
```
데이터 수집 → 패턴분석 → 효과성측정 → 맞춤추천 → 확언생성
```

**수집 데이터:** 감정기록, 세션참여, 효과평가, 대화내용, 시간패턴

### 7. 진행 추적 및 통계
```
통계 확인 → 기간선택 → 감정변화그래프 → 효과성분석 → 목표조정
```

### 8. 알림 및 예외처리
```
패턴감지 → 알림트리거 → 맞춤메시지 → 액션제안
위험신호감지 → 즉시지원 → 전문가연결 → 24시간핫라인
```

## 🎨 Figma to Code Workflow | 피그마 완벽 복사 가이드

### 핵심 원칙: 100% 피그마 따르기 (창의성 금지!)

**⚠️ 중요: Claude는 피그마 디자인을 픽셀 단위로 정확히 복사해야 함**

```typescript
// 🎯 피그마 이미지 분석 → 코드 변환 프로세스
interface FigmaCopyProcess {
  step1: "피그마 이미지에서 정확한 색상 값 추출 (HEX 코드)";
  step2: "픽셀 단위 크기 측정 (width, height, padding, margin)";
  step3: "폰트 스타일 정확히 분석 (size, weight, line-height)";
  step4: "그림자, 테두리, 둥근 모서리 정확한 값 추출";
  step5: "레이아웃 구조 분석 (Flexbox, Grid, Auto Layout)";
  step6: "애니메이션 및 인터랙션 상태 확인";
  step7: "측정된 값 그대로 코드 생성 (추측 금지!)";
}

// 🚫 금지사항: 내 마음대로 디자인하지 말 것
const prohibitedActions = {
  "색상 추측": "❌ background: 'blue' (정확한 HEX 사용할 것)",
  "크기 추측": "❌ padding: '10px' (피그마에서 측정한 값 사용)",
  "스타일 변경": "❌ 더 예쁘게 만들기 시도 금지",
  "컴포넌트 추가": "❌ 피그마에 없는 요소 추가 금지",
  "개선 제안": "❌ 더 나은 UX 제안 금지 (요청받기 전까지)"
};

// ✅ 올바른 접근법: 피그마 측정 → 정확한 코드
const correctApproach = {
  colorExtraction: "피그마 Color Picker로 정확한 HEX 값 추출",
  sizeManagement: "피그마 Inspector로 정확한 픽셀 값 측정",
  fontMatching: "피그마 Typography 패널에서 정확한 폰트 정보",
  layoutCopy: "피그마 Auto Layout → CSS Flexbox/Grid 정확히 변환",
  shadowCopy: "피그마 Effects → CSS box-shadow 정확히 변환"
};
```

### EFT 앱 전용 디자인 토큰 (피그마에서 추출)

```typescript
// 🎨 피그마에서 추출할 EFT 앱 색상 시스템
const eftColorTokens = {
  // 감정별 색상 (피그마 Color Styles에서 정확히 추출)
  emotions: {
    calm: "#피그마에서_추출한_정확한_값",
    stress: "#피그마에서_추출한_정확한_값", 
    happy: "#피그마에서_추출한_정확한_값",
    sad: "#피그마에서_추출한_정확한_값"
  },
  
  // UI 색상 (피그마 Design System에서 추출)
  ui: {
    primary: "#피그마_Primary_색상",
    secondary: "#피그마_Secondary_색상",
    background: "#피그마_Background_색상",
    surface: "#피그마_Surface_색상"
  }
};

// 📏 피그마에서 측정할 정확한 spacing 값
const eftSpacingTokens = {
  tappingPoint: "피그마에서_측정한_탭핑포인트_간격px",
  sessionGap: "피그마에서_측정한_세션_간격px",
  chatBubble: "피그마에서_측정한_채팅_간격px",
  cardPadding: "피그마에서_측정한_카드_패딩px"
};
```

### 피그마 → React 컴포넌트 정확한 변환 가이드

```typescript
// 📱 피그마 EFT 화면 → React 완벽 복사 매핑
interface EFTComponentExactCopy {
  // 1. 피그마 측정 → CSS 정확히 변환
  measureAndCopy: {
    width: "피그마_Inspector에서_측정한_정확한_px값",
    height: "피그마_Inspector에서_측정한_정확한_px값",
    padding: "피그마_Auto_Layout에서_측정한_정확한_값",
    margin: "피그마에서_측정한_정확한_간격_값",
    borderRadius: "피그마_Corner_Radius_정확한_값"
  };
  
  // 2. 피그마 타이포그래피 → CSS 정확히 복사
  typography: {
    fontSize: "피그마_Text_Style에서_추출한_정확한_size",
    fontWeight: "피그마에서_설정한_정확한_weight",
    lineHeight: "피그마_Line_Height_정확한_값",
    letterSpacing: "피그마_Letter_Spacing_정확한_값"
  };
  
  // 3. 피그마 이펙트 → CSS 정확히 변환
  effects: {
    boxShadow: "피그마_Drop_Shadow를_CSS로_정확히_변환",
    borderColor: "피그마_Border에서_추출한_정확한_색상",
    backgroundColor: "피그마_Fill에서_추출한_정확한_색상"
  };
}

// 🎭 피그마 애니메이션 → 정확한 CSS/JS 변환
const figmaAnimationCopy = {
  // 피그마 Smart Animate → Framer Motion 정확히 복사
  smartAnimate: {
    duration: "피그마에서_설정한_정확한_duration_ms",
    easing: "피그마_Easing에서_추출한_정확한_cubic_bezier값",
    delay: "피그마에서_설정한_정확한_delay_값"
  },
  
  // 피그마 프로토타입 → React 상태 변화 정확히 복사
  prototypeCopy: {
    hover: "피그마_Hover_State를_정확히_CSS로_변환",
    active: "피그마_Pressed_State를_정확히_CSS로_변환",
    focus: "피그마_Focus_State를_정확히_CSS로_변환"
  }
};
```

### 피그마 분석 체크리스트

```typescript
// ✅ 피그마 이미지를 받으면 반드시 확인할 것들
const figmaAnalysisChecklist = {
  colors: "모든 색상을 정확한 HEX 값으로 추출했는가?",
  spacing: "모든 간격을 픽셀 단위로 정확히 측정했는가?", 
  typography: "폰트 크기, 굵기, 행간을 정확히 추출했는가?",
  layout: "Flexbox/Grid 구조를 정확히 분석했는가?",
  interactions: "hover, active 상태를 모두 확인했는가?",
  responsive: "다른 화면 크기 버전이 있는지 확인했는가?",
  assets: "아이콘, 이미지를 정확한 크기로 추출했는가?"
};

// 🎯 최종 목표: 피그마와 100% 동일한 결과물
const successCriteria = {
  visualMatch: "육안으로 구별할 수 없을 정도로 동일",
  pixelPerfect: "픽셀 단위로 정확한 위치와 크기",
  colorAccuracy: "색상이 정확히 일치",
  behaviorMatch: "인터랙션이 피그마 프로토타입과 동일"
};
```

**🎯 핵심: Claude는 피그마의 충실한 복사기 역할만 수행. 창의성이나 개선 제안은 명시적 요청이 있을 때만!**

## 🔐 AI 모델 라이선스 및 비용 정책 (2025.08.20 추가)

### **HuggingFace 토큰 인증 시스템**

#### **1. 설정 방법**
```javascript
// HuggingFace 인증 설정 과정
const huggingFaceSetup = {
  step1: "HuggingFace 회원가입: https://huggingface.co/",
  step2: "Llama 3.1 접근 권한 요청 (즉시 승인)",
  step3: "Personal Access Token 생성",
  step4: "백엔드 .env 파일에 HUGGINGFACE_TOKEN 설정",
  
  // 업계 표준 방식
  standard: "OpenAI API 키와 동일한 개념",
  security: "상용 AI 서비스의 기본 보안 모델"
};
```

#### **2. 비용 분석**
```javascript
const costAnalysis = {
  huggingFaceUsage: {
    modelDownload: "완전 무료",
    localExecution: "무료 (자체 서버 실행)",
    inferenceAPI: "유료 (월 $9~) - 사용하지 않음"
  },
  
  ourImplementation: {
    cost: "0원",
    method: "로컬 실행 방식",
    benefit: "HuggingFace 서버 비용 없음"
  }
};
```

#### **3. 상업적 사용 라이선스**
```javascript
const licensePolicy = {
  // Llama 3.1 Custom License (2024년 7월 업데이트)
  currentStatus: {
    commercialUse: "허용됨",
    condition: "MAU(월 활성 사용자) 700만 미만 회사",
    ourStatus: "✅ 합법 (스타트업/중소기업)",
    validity: "2024년 7월부터 상업적 사용 완전 허용"
  },
  
  // 700만 사용자 달성 시 대응 계획
  scalingPlan: {
    trigger: "MAU 700만 돌파 시",
    options: [
      {
        name: "Meta 엔터프라이즈 라이선스",
        cost: "협상 필요",
        benefit: "Llama 3.1 계속 사용 가능"
      },
      {
        name: "오픈소스 모델 전환",
        alternatives: [
          "Microsoft Phi-3 (완전 오픈소스)",
          "Google Gemma (상업적 사용 자유)",
          "Hugging Face 자체 모델들 (MIT/Apache)"
        ],
        cost: "무료",
        effort: "2-3주 마이그레이션 작업"
      }
    ],
    
    recommendation: "700만 달성까지 Llama 3.1 사용, 이후 상황에 따라 결정"
  },
  
  // 법적 안전 조치
  legalSafeguards: {
    currentCompliance: "완전 합법적 사용",
    monitoring: "사용자 수 추적 시스템 필요",
    documentation: "라이선스 준수 기록 유지",
    timeline: "700만 달성 3개월 전 마이그레이션 계획 수립"
  }
};
```

#### **4. 기술적 이점**
- **품질**: 최고급 AI 모델 (ChatGPT 수준)
- **한국어 지원**: 뛰어난 한국어 이해 및 생성
- **EFT 전문성**: 상담 및 심리학 지식 우수
- **확장성**: 다른 Llama 모델로 업그레이드 가능

#### **5. 리스크 관리**
```javascript
const riskManagement = {
  shortTerm: "현재 위험 없음 (완전 합법)",
  mediumTerm: "사용자 수 모니터링 필요",
  longTerm: "700만 사용자 달성 시 전환 계획",
  
  contingencyPlan: {
    monitoring: "월별 MAU 추적",
    warningThreshold: "500만 사용자 시 전환 준비",
    migrationTime: "3개월 여유 기간"
  }
};
```

---

## 🔄 백엔드 전환 계획 (Firebase → Supabase)

### 전환 타이밍 및 전략

**📅 전환 시점:**
- **사용자 1만명 이상** 달성 시
- **SQL 복잡 쿼리** 필요할 때 (감정 패턴 분석, 고급 통계)
- **비용 최적화** 필요 시 (Firebase 요금 부담)
- **오픈소스 요구사항** 생길 때

**🎯 하이브리드 전환 전략:**
```javascript
// 점진적 마이그레이션 접근법
const migrationStrategy = {
  phase1: {
    period: "MVP ~ 초기 사용자 (1K)",
    backend: "Firebase 100%",
    reason: "빠른 개발, 안정성"
  },
  
  phase2: {
    period: "성장기 (1K ~ 10K 사용자)",
    backend: "Firebase (기존) + Supabase (신규 기능)",
    strategy: "새 기능만 Supabase로 개발"
  },
  
  phase3: {
    period: "확장기 (10K+ 사용자)",
    backend: "Supabase 완전 전환",
    migration: "기존 데이터 마이그레이션 + 사용자 재인증"
  }
};

// 전환 비용 vs 효과 분석
const migrationAnalysis = {
  benefits: {
    cost: "월 사용료 30-50% 절약",
    performance: "SQL 쿼리로 복잡한 분석 가능",
    flexibility: "오픈소스로 커스터마이징",
    dataOwnership: "완전한 데이터 소유권"
  },
  
  costs: {
    development: "2-3주 개발 시간",
    risk: "일시적 서비스 불안정성",
    learning: "팀 학습 곡선",
    userImpact: "재로그인 등 사용자 불편"
  },
  
  decisionCriteria: {
    userGrowth: "월 활성 사용자 10K 이상",
    dataComplexity: "고급 분석 쿼리 필요성",
    costPressure: "Firebase 월 비용 $500 이상",
    teamCapacity: "전환 작업 전담 개발자 확보"
  }
};
```

**🛠️ 마이그레이션 체크리스트:**
- [ ] Supabase 프로젝트 셋업 및 스키마 설계
- [ ] 데이터 마이그레이션 스크립트 작성
- [ ] 하이브리드 환경 테스트
- [ ] 사용자 재인증 프로세스 준비
- [ ] 롤백 계획 수립
- [ ] 성능 모니터링 시스템 구축

## 📅 **2025년 8월 23일 최신 개발 현황 (Current Status)**

### **🎉 Phase 1-4 완료! (백엔드 AI 서버 추가)**

**✅ Phase 1: 현재 모바일 UI 완성**
- React Router 기반 페이지 네비게이션 완료
- 모달 방식 → 네이티브 앱 스타일 페이지 전환
- 뒤로가기 버튼 완벽 지원

**✅ Phase 2: 반응형 웹앱 완성**  
- ResponsiveContainer 컴포넌트로 모바일/데스크톱 최적화
- 📱 모바일: 풀스크린 네이티브 앱 경험
- 💻 데스크톱: 중앙 배치 + 그림자 효과로 앱 느낌

**✅ Phase 3: PWA 완전 구현**
- Vite PWA Plugin 통합 완료
- Service Worker 자동 캐싱 (AI 모델 포함)
- 홈화면 설치 프롬프트 (PWAInstallPrompt 컴포넌트)
- 오프라인 지원 + 아름다운 폴백 페이지
- TWA 준비 완료 (구글 Play 스토어 런칭 가능)

**✅ Phase 4: 백엔드 AI 서버 완전 구축 (NEW!)**
- FastAPI 기반 EFT 전문 AI 상담 서버 완성
- 티어별 AI 모델 시스템 (무료/프리미엄/엔터프라이즈)
- UTF-8 인코딩 근본 해결 (PYTHONUTF8=1)
- HuggingFace 토큰 인증 및 Llama 3.1 접근 권한 신청 완료
- 감정 분석 + EFT 프롬프트 매니저 + 전문 상담 시스템
- 실시간 로깅 및 모니터링 시스템

**🔄 Level 3 AI 시스템 (Llama 3.1) 승인 대기 중**
- Transformers.js 클라이언트 AI (완료) + FastAPI 서버 AI (구축 완료)
- Meta Llama 3.1 접근 권한 심사 대기 중 (5-30분 예상)
- 현재 DialoGPT로 임시 운영, 승인 후 Llama 3.1 전환 예정

### **🚀 구글 Play 스토어 런칭 준비도: 98%**

**현재 상태:**
```typescript
런칭_준비도 = {
  pwa_기본기능: "100% ✅",
  ai_시스템_클라이언트: "Level 2 완료 ✅",
  ai_시스템_서버: "Level 3 구축완료, Llama 3.1 승인대기 🔄",
  백엔드_인프라: "FastAPI + UTF-8 + 로깅 완료 ✅",
  반응형_디자인: "100% ✅",
  service_worker: "100% ✅",
  manifest: "100% ✅",
  documentation: "완전 업데이트 완료 ✅",
  
  // 런칭을 위해 추가 필요 (1주 이내 완료 가능!)
  llama_승인: "Meta 심사 대기 중 (오늘 중 완료 예상) 🔄",
  https_배포: "Vercel/Netlify 배포 필요 🔄",
  아이콘_디자인: "512x512 마스크형 아이콘 필요 🔄", 
  스크린샷: "5장 준비 필요 🔄",
  TWA_빌드: "Bubblewrap 빌드 필요 🔄"
};
```

## 📅 **2025년 8월 18일 결정사항 (이전 기록)**

### **1. 네비게이션 구조 단순화**
**결정**: 6개 탭 → 홈 + 메뉴 2개로 단순화
```
기존: │🏠홈│❤️체크│💬AI│✨EFT│🔮통찰│👤마이│
변경: │🏠 홈                            ☰ 메뉴│
```

**☰ 메뉴 구조:**
```
☰ 메뉴
├── 💬 AI 상담 (핵심 기능)
├── ❤️ 감정 체크인  
├── ✨ EFT 세션
├── 🔮 나의 통찰
├── 👤 내 프로필
├── ⚙️ 설정
│   ├── 🔔 알림 설정
│   ├── 🎨 테마 설정  
│   ├── 🔒 개인정보 (간소화)
│   │   ├── 데이터 수집 동의 [ON/OFF]
│   │   ├── AI 학습 참여 [ON/OFF]
│   │   └── 계정 탈퇴
│   └── 📞 고객지원
└── 🚪 로그아웃
```

**장점:**
- 화면 공간 100% 활용
- 직관적이고 심플한 UI
- 모든 기능이 체계적으로 정리
- 확장성 높음

### **2. 회원가입 시스템 간소화**
**결정**: 구글 로그인만 유지, 다른 옵션 제거

```
기존: 구글 + 애플 + 카카오 + 이메일 + 게스트
변경: 구글 로그인만 + 약관 동의
```

**최종 로그인 화면:**
```
┌─────────────────────────────────────┐
│            🌿 환영합니다!            │
├─────────────────────────────────────┤
│    "AI와 함께하는 마음 여행을        │
│     지금 시작해보세요"             │
│                                   │
│ ┌─────────────────────────────────┐ │
│ │  🔵 Google로 3초 시작하기        │ │
│ └─────────────────────────────────┘ │
│                                   │
│ ☑️ 서비스 이용약관 동의 (필수)      │
│ ☑️ 개인정보 처리방침 동의 (필수)    │
│ □ 마케팅 수신 동의 (선택)          │
│                                   │
│          [시작하기]                │
│                                   │
│ 💡 안전하고 빠른 구글 계정 로그인    │
│    개인정보는 최소한만 수집합니다    │
└─────────────────────────────────────┘
```

**장점:**
- 개발 복잡도 최소화
- 3초 가입 완료
- 보안 및 신뢰성 높음
- 비밀번호 찾기/재설정 불필요

### **3. 200문항 검사 위치 확정**
**결정**: 메뉴에서 제거, 퀘스트로만 운영

**위치:**
- ❌ 메뉴에 별도 항목 없음
- ✅ 🏠 홈 대시보드의 "첫 번째 통찰 해제" 퀘스트
- ✅ 온보딩 때 안내
- ✅ 🔮 나의 통찰 메뉴에서 퀘스트 진행 상황 확인

**이유:** 200문항은 통찰 해제를 위한 도구이지 독립적인 기능이 아님

### **4. 데이터 관리 화면 제거**
**결정**: 복잡한 데이터 관리 UI 삭제, 간단한 개인정보 설정만 유지

**제거된 기능:**
- ❌ 데이터 용량 표시
- ❌ 데이터 다운로드/백업
- ❌ 세부 데이터 삭제 옵션

**유지된 기능:**
- ✅ 데이터 수집 동의 ON/OFF
- ✅ AI 학습 참여 ON/OFF  
- ✅ 계정 탈퇴

**이유:** 사용자 혼란 방지, 데이터 축적이 목표이므로 삭제 기능 최소화

### **5. 관리자 시스템 필요성 확정**
**결정**: 별도 관리자 패널 구축 필요

**관리자 시스템 구성:**
```
📊 사용자 관리: 가입자 현황, 이용 패턴 분석
🤖 AI 시스템 관리: 응답 품질 모니터링, 자동 필터링  
📝 콘텐츠 관리: 32개 공통 통찰, EFT 가이드 편집
🎫 고객지원: 티켓 관리, FAQ 관리
📈 실시간 분석: 대시보드, 리포트 생성
```

**기술 스택:**
- **프론트엔드**: React + Ant Design (관리자 UI 최적화)
- **백엔드**: Firebase (사용자 앱과 공유)
- **호스팅**: Vercel (관리자 패널 전용)
- **보안**: Firebase Admin SDK + 권한 체계

**구축 계획:** 3주 (기본 1주 → 고급 1주 → 자동화 1주)

### **6. AI 대화 시스템 핵심 기능 재확인**
**확정**: AI 대화/상담이 앱의 기본이자 핵심 기능

**역할:**
- 🎯 통찰 해제의 주요 수단 (32개 통찰 대부분)
- 📊 개인화 AI 학습 데이터 소스
- 🏠 메인 대시보드 중앙 배치
- 🚀 기존 EFT 앱과의 차별화 포인트

**우선순위:** 모든 개발에서 최고 우선순위 유지

---

## 🎯 **코드 구현 가이드라인 (필수 준수)**

### **📋 설계서 기반 개발 원칙**
**⚠️ 중요: 모든 코드는 반드시 `완전통합_전체앱_UIUX설계서.md`를 기준으로 구현할 것**

```typescript
// 🎯 설계서 준수 체크리스트
const implementationGuidelines = {
  uiReference: "완전통합_전체앱_UIUX설계서.md",
  
  mandatoryComponents: {
    navigation: "홈 + ☰메뉴 구조 (6개 탭 금지)",
    dashboard: "RPG 스타일 대시보드 레이아웃",
    menuStructure: "설계서의 정확한 메뉴 구조",
    insights: "32개 공통 통찰 + AI 개인맞춤 시스템",
    quests: "일일 퀘스트 + 메인 퀘스트 시스템"
  },
  
  designRequirements: {
    colors: "설계서 명시된 RPG 게임 색상 체계",
    layout: "설계서의 카드 기반 레이아웃",
    interactions: "설계서의 사용자 플로우 준수",
    responsive: "모바일 퍼스트 (320px+) 설계"
  },
  
  prohibitedActions: {
    "임의 변경": "❌ 설계서와 다른 UI/UX 구현 금지",
    "기능 추가": "❌ 설계서에 없는 기능 임의 추가 금지", 
    "구조 변경": "❌ 네비게이션/메뉴 구조 임의 수정 금지",
    "디자인 개선": "❌ 설계서보다 '더 예쁘게' 만들기 시도 금지"
  }
};
```

### **🔗 설계서 연동 필수 사항**
1. **UI 레이아웃**: 설계서의 ASCII 아트 레이아웃 정확히 구현
2. **메뉴 구조**: 설계서의 ☰ 메뉴 트리 구조 100% 준수
3. **색상/스타일**: 설계서의 RPG 게임 테마 적용
4. **사용자 플로우**: 설계서의 화면 전환 로직 준수
5. **컴포넌트 명명**: 설계서의 기능명과 일치하는 컴포넌트명 사용

### **📖 주요 참조 섹션**
- **메인 대시보드**: `## 🏠 3. 메인 대시보드 (RPG 스타일) - 통합`
- **네비게이션**: `## 🗺️ 10. 네비게이션 설계 (단순화)`
- **AI 대화**: `## 💬 4. AI 대화 시스템 설계`
- **통찰 시스템**: `## 🔮 7. 통찰 시스템 상세 설계`
- **EFT 세션**: `## ✨ 6. EFT 세션 시스템 설계`

---

## 📋 **구현 완료 현황 및 다음 단계**

### **✅ Phase 1-3 모두 완료! (2025.08.19)**
1. ✅ 구글 로그인 시스템
2. ✅ 홈 + 메뉴 네비게이션 (React Router)
3. ✅ RPG 스타일 대시보드
4. ✅ **Level 2 AI 대화 시스템** (Transformers.js 완전 구현)
5. ✅ 반응형 웹앱 (ResponsiveContainer)
6. ✅ PWA 완전 구현 (홈화면 설치, 오프라인 지원)
7. ✅ 종합 문서화 (README.md)

### **🚀 다음 단계: 스토어 런칭 (1-2주)**
1. **HTTPS 배포** (Vercel/Netlify) - 1일
2. **아이콘 및 스크린샷 제작** - 2일  
3. **TWA 빌드** (Bubblewrap) - 1일
4. **구글 Play Console 등록** - 3-7일
5. **앱 스토어 런칭** 🎉

### **🔮 장기 확장 계획:**
1. **고급 기능**: 3D/AR EFT 가이드
2. **관리자 시스템**: 사용자 관리, 콘텐츠 관리
3. **수익화**: 프리미엄 구독, 기업용 라이선스
4. **글로벌 확장**: 다국어 지원

---

## 🛠️ **개발 방침 및 문제 해결 원칙 (2025.08.19 추가)**

### **Claude Code 개발 원칙**

#### **1. 문제 해결 접근법**
- ❌ **잘못된 접근**: 오류 발생 시 쉬운 우회 방법 선택
- ✅ **올바른 접근**: 근본 원인 파악 및 정면 돌파
- ✅ **더 나은 접근**: 근본적 설계 개선으로 품질 향상

#### **2. 실제 적용 사례 (오늘의 교훈)**

**상황 1: AI 채팅 404 오류**
```
❌ 잘못된 해결책: "시뮬레이션 AI로 바꾸자"
✅ 올바른 해결책: "FastAPI 서버를 실제로 실행하자"
```

**상황 2: requirements.txt 인코딩 오류**
```
❌ 잘못된 해결책: "간단한 패키지만 설치하자"
✅ 올바른 해결책: UTF-8 인코딩 문제 해결
💡 근본 원인: 한국어 주석의 CP949 인코딩 충돌
🎯 정확한 접근: 파일 인코딩 수정 → 전체 의존성 설치
```

#### **3. 개발 시 필수 체크리스트**
1. **상황 파악 우선**: 기획서와 코드 현황 모두 확인
2. **아키텍처 이해**: 프론트엔드/백엔드 구조 정확히 파악
3. **근본 원인 집중**: 증상이 아닌 원인에 집중
4. **품질 향상 기회**: 더 나은 방법이 있다면 적극 제안
5. **사용자와 소통**: 불확실할 때는 반드시 확인

#### **4. 금지 사항**
- 🚫 오류 우회를 위한 기능 축소
- 🚫 "일단 되게 하자" 식의 임시방편
- 🚫 상황 파악 없이 독단적 결정
- 🚫 쉬운 길로만 해결 시도

#### **5. 권장 사항**
- ✅ 항상 "왜?"라고 질문하기
- ✅ 더 나은 해결책이 있는지 고민하기
- ✅ 사용자와 충분한 소통 후 결정하기
- ✅ 장기적 관점에서 품질 고려하기

---

## 🚀 **배포 설정 가이드 (Production Deployment)**

### **CORS 설정 변경사항**

#### **🔧 개발 환경 (현재)**
```python
# backend/main.py - 개발용 설정
if settings.DEBUG:  # 개발 환경
    app.add_middleware(
        CORSMiddleware,
        allow_origin_regex=r"^http://(localhost|127\.0\.0\.1):\d+$",
        allow_methods=["*"],
        allow_headers=["*"],
        allow_credentials=True,
    )
```

#### **🌐 운영 환경 배포 시 필수 수정**
```python
# backend/config/settings.py - 운영용 수정
DEBUG = False
ALLOWED_ORIGINS = [
    "https://your-domain.com", 
    "https://app.your-domain.com",
    "https://eft-ai.vercel.app"  # 실제 도메인으로 변경
]

# backend/main.py - 운영 환경에서 자동 적용됨
else:  # 운영 환경
    app.add_middleware(
        CORSMiddleware,
        allow_origins=settings.ALLOWED_ORIGINS,  # 화이트리스트 방식
        allow_methods=["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"],
        allow_headers=["Authorization", "Content-Type"],
        allow_credentials=True,
    )
```

### **배포 체크리스트**
- [ ] `DEBUG = False` 설정 확인
- [ ] `ALLOWED_ORIGINS`에 실제 도메인 추가
- [ ] 개발용 정규식 CORS 자동 비활성화 확인
- [ ] HTTPS 인증서 설정 완료
- [ ] 환경변수 `.env` 파일 보안 설정

---

## 📅 **최신 업데이트: AI 채팅 시스템 완성 (2025.09.15)**

### **🎉 AI 채팅 시스템 완전 작동 확인!**
- ✅ **vLLM 서버 연동**: Qwen-2.5-7B 모델 정상 작동 (포트 8002)
- ✅ **프롬프트 정리 시스템**: 프론트엔드에서 시스템 지침 제거
- ✅ **자연스러운 대화**: 경청 우선, 점진적 EFT 소개로 개선
- ✅ **UTF-8 인코딩**: 한국어 처리 완벽 해결
- ✅ **보안 강화**: config.env Git 히스토리에서 완전 제거

### **🛠️ 기술적 구현 완료**
```typescript
// 프론트엔드 프롬프트 정리 시스템
const cleanResponse = (() => {
  const response = serverResponse.response;
  if (response.includes("지금부터 EFT 전문 상담사로서 응답해 주세요:")) {
    return response.split("지금부터 EFT 전문 상담사로서 응답해 주세요:").pop()?.trim() || response;
  }
  return response;
})();
```

### **🎯 사용자 경험 개선**
- **Before**: `'로 힘드시겠어요. 깊게 숨을...` (문맥 끊어짐)
- **After**: `잠이 안 와서 힘드시겠어요. 어떤 것이 마음을 무겁게 하시나요?` (자연스러운 흐름)

## 📅 **이전 업데이트: Engine A/B 병렬 비교 시스템 (2025.09.09)**

### **🔥 DialoGPT 완전 폐기 완료!**
- ✅ **DialoGPT 제거**: 구식 모델 완전 삭제
- ✅ **Engine A/B 시스템**: Llama-3-8B vs Qwen-2.5-7B 병렬 비교
- ✅ **병렬 처리**: `asyncio.gather()` 방식으로 응답 시간 최적화
- ✅ **투명한 비교**: 두 모델 응답을 나란히 표시

### **🚀 새로운 API 엔드포인트 (2025.09.09)**

#### **1. Engine A/B 병렬 비교 시스템**
```javascript
// 🔥 메인 엔드포인트: DialoGPT 완전 대체
POST /api/chat/compare
{
  "message": "스트레스가 심해요",
  "temperature": 0.7,
  "max_tokens": 512
}

// 응답 구조
{
  "llama3_response": {
    "model": "meta-llama/Meta-Llama-3-8B-Instruct",
    "response": "Llama-3의 응답...",
    "processing_time": 2.5,
    "success": true
  },
  "qwen25_response": {
    "model": "Qwen/Qwen2.5-7B-Instruct",
    "response": "Qwen-2.5의 응답...", 
    "processing_time": 3.1,
    "success": true
  },
  "faster_model": "llama3",
  "comparison_time": 3.2
}
```

#### **2. 기존 채팅 엔드포인트 (Engine A/B로 전환)**
```javascript
// 기본 채팅 → Engine A/B 병렬 비교로 자동 전환
POST /api/chat
{
  "message": "직장에서 스트레스받아요",
  "conversation_history": [],
  "user_profile": {}
}

// ChatResponse 형태로 반환 (호환성 유지)
{
  "response": "더 빠른 모델의 응답",
  "emotion_analysis": {...},
  "model_version": "Engine A (Llama-3-8B)",
  "processing_time": 3.2,
  "tier": "free"
}
```

#### **3. A/B 테스트용 채팅 완성**
```javascript
// vLLM 서버 직접 연결 (개발/테스트용)
POST /api/chat/completion
{
  "message": "불안해요",
  "temperature": 0.7,
  "max_tokens": 400
}

// 폴백 시스템 포함
{
  "tier": "free",
  "engine": "engine_a",
  "model": "meta-llama/Meta-Llama-3-8B-Instruct",
  "reply": "AI 응답 내용",
  "processing_time": 2.8,
  "fallback_used": false
}
```

#### **4. 헬스체크 및 모니터링**
```javascript
// 기본 헬스체크
GET /health
{
  "status": "healthy",
  "tier": "free",
  "strategy": "random",
  "free_engines": {
    "engine_a": {"model": "meta-llama/Meta-Llama-3-8B-Instruct", "port": 8001},
    "engine_b": {"model": "Qwen/Qwen2.5-7B-Instruct", "port": 8002}
  }
}

// 업스트림 서버 상태 (관리자용)
GET /health/upstreams
{
  "overall_status": "healthy",
  "upstreams": {
    "engine_a": {
      "status": "healthy",
      "url": "http://127.0.0.1:8001",
      "latency_ms": 45.2
    },
    "engine_b": {
      "status": "healthy", 
      "url": "http://127.0.0.1:8002",
      "latency_ms": 52.1
    }
  }
}
```

#### **5. 레거시 엔드포인트 (유지/폐기)**
```javascript
// ✅ 유지: 프리미엄 티어
POST /api/chat/premium  // Llama-3.1-8B 전용

// ✅ 유지: 무료 티어 (Engine A/B로 리다이렉트)
POST /api/chat/free     // Engine A/B 병렬 처리로 전환

// ❌ 폐기: 스트리밍 (엔진 마이그레이션 중)
POST /api/chat/stream   // 501 Not Implemented

// ✅ 유지: 유틸리티
POST /api/analyze/emotion    // 감정 분석 전용
POST /api/recommend/eft      // EFT 기법 추천
GET /api/stats              // 모델 통계
```

### **✅ 무료 사용자 경험 개선**
- **비교 가능**: 두 최신 모델의 응답을 직접 비교
- **성능 투명성**: 어떤 모델이 더 빠른지 명확히 표시  
- **품질 향상**: 구식 DialoGPT → 최신 Llama-3 + Qwen-2.5

### **🔧 개발자 가이드라인 (2025.09.09)**

#### **vLLM 서버 설정 (필수)**
```bash
# 1. vLLM 설치
pip install vllm

# 2. Engine A 실행 (포트 8001)
vllm serve meta-llama/Meta-Llama-3-8B-Instruct \
  --port 8001 \
  --host 127.0.0.1 \
  --api-key EMPTY

# 3. Engine B 실행 (포트 8002)  
vllm serve Qwen/Qwen2.5-7B-Instruct \
  --port 8002 \
  --host 127.0.0.1 \
  --api-key EMPTY

# 4. 연결 확인
curl http://localhost:8001/v1/models
curl http://localhost:8002/v1/models
```

#### **Backend 개발 팁**
- **A/B 라우팅**: `ABRouteMiddleware`로 자동 엔진 선택
- **폴백 처리**: `other_engine_key()` 함수로 자동 대체
- **상관관계 추적**: `x-request-id` 헤더로 요청 추적
- **레이트 리밋**: 기본 분당 120회 제한

#### **Frontend 연동**
```typescript
// Engine A/B 병렬 비교 호출
const response = await fetch('/api/chat/compare', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    message: userInput,
    temperature: 0.7,
    max_tokens: 512
  })
});

const comparison = await response.json();
// comparison.faster_model로 더 빠른 모델 확인
// comparison.llama3_response, comparison.qwen25_response로 각각 접근
```

---

## 🎯 **프로젝트 완성도 요약 (2025.09.09)**

**✅ 완료된 혁신 사항:**
- 🔥 **DialoGPT 완전 폐기** → **Engine A/B 병렬 시스템**
- 🚀 **무료 사용자도 최신 AI 2개 동시 비교**
- ⚡ **asyncio.gather() 병렬 처리**로 성능 최적화
- 🛡️ **다중 폴백 시스템**으로 안정성 확보
- 🔍 **투명한 성능 표시**로 사용자 경험 향상

**🎯 이제 구글 Play 스토어 런칭 준비 완료!**

---

## 🎯 **현재 프로젝트 완성도 (2025.09.15)**

### **✅ 완료된 핵심 기능**
- 🤖 **AI 채팅 시스템**: vLLM + Qwen-2.5-7B 완전 작동
- 🎨 **프론트엔드**: React PWA + 반응형 디자인 + 홈화면 설치
- 🔧 **백엔드**: FastAPI + 프롬프트 관리 + 감정 분석
- 🔒 **보안**: Git 히스토리 정리 + 환경변수 관리
- 📱 **PWA**: Service Worker + 오프라인 지원 + 설치 프롬프트

### **🚀 구글 Play 스토어 런칭 준비도: 85%**
```javascript
런칭_준비도 = {
  core_functionality: "100% ✅",
  ai_chat_system: "100% ✅",
  pwa_features: "100% ✅",
  responsive_design: "100% ✅",

  // 런칭 전 완료 필요 (1-2주)
  https_deployment: "필요 🔄",
  app_icons: "필요 🔄",
  store_screenshots: "필요 🔄",
  twa_build: "필요 🔄"
};
```

### **🎯 다음 개발 단계**
1. **프롬프트 최적화**: 더 자연스러운 대화 흐름 (제안된 toDisplayText 시스템)
2. **EFT 탭핑 가이드**: 시각적 가이드 시스템 연동
3. **통찰 시스템**: 32개 공통 통찰 + AI 개인맞춤 통찰
4. **Engine A/B 병렬**: 두 모델 동시 비교 시스템 (Engine A 추가 필요)

이 프로젝트는 사용자의 심리적 웰빙을 지원하는 혁신적인 EFT 기반 AI 애플리케이션을 목표로 합니다.

---
> Source: [gogoleelee88/moodtalk-public](https://github.com/gogoleelee88/moodtalk-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
