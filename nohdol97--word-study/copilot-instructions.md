## word-study

> 실전 영어 회화에 특화된 Flutter 기반 단어장 애플리케이션 개발

# CLAUDE.md - 영어 회화 단어장 앱 개발 가이드

## 프로젝트 개요
실전 영어 회화에 특화된 Flutter 기반 단어장 애플리케이션 개발

## 핵심 개발 원칙

### 1. 실용성 최우선
- 실제 회화에서 사용 빈도가 높은 단어만 선별
- 교과서적 표현보다 구어체 표현 우선
- Native speaker가 일상에서 사용하는 자연스러운 예문

### 2. 사용자 중심 설계
- 직관적이고 심플한 UI
- 최소 클릭으로 학습 시작
- 빠른 반응 속도 (< 100ms)

### 3. 오프라인 우선
- 모든 데이터 로컬 저장
- 네트워크 없이 완전한 기능 제공
- SQLite 활용한 효율적 데이터 관리

## 기술 스택

### Frontend
- **Framework**: Flutter 3.16+
- **State Management**: Provider or Riverpod
- **Local Storage**: SQLite (sqflite package)
- **Preferences**: shared_preferences

### 주요 패키지
```yaml
dependencies:
  flutter:
    sdk: flutter
  sqflite: ^2.3.0
  path: ^1.8.3
  provider: ^6.1.1
  shared_preferences: ^2.2.2
  flutter_slidable: ^3.0.1  # 스와이프 액션
  percent_indicator: ^4.2.3  # 진도 표시
  fl_chart: ^0.65.0  # 통계 차트
```

## 프로젝트 구조
```
lib/
├── main.dart
├── models/
│   ├── word.dart           # 단어 모델
│   └── category.dart       # 카테고리 모델
├── screens/
│   ├── home_screen.dart    # 홈 화면
│   ├── word_list_screen.dart
│   ├── study_screen.dart
│   ├── test_screen.dart
│   └── settings_screen.dart
├── widgets/
│   ├── word_card.dart      # 단어 카드 위젯
│   └── progress_bar.dart
├── services/
│   ├── database_service.dart  # DB 관리
│   └── word_service.dart      # 단어 비즈니스 로직
└── utils/
    └── constants.dart
```

## 데이터베이스 스키마

### Words 테이블
```sql
CREATE TABLE words (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  english TEXT NOT NULL,
  korean TEXT NOT NULL,
  pronunciation TEXT,
  category TEXT NOT NULL,
  difficulty INTEGER DEFAULT 3,
  is_memorized INTEGER DEFAULT 0,
  review_count INTEGER DEFAULT 0,
  last_review_date TEXT,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP
);
```

### Examples 테이블
```sql
CREATE TABLE examples (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  word_id INTEGER,
  example_text TEXT NOT NULL,
  translation TEXT,
  FOREIGN KEY (word_id) REFERENCES words (id)
);
```

## 필수 회화 단어 데이터

### 카테고리별 핵심 단어 (상위 100개)

#### Daily Conversation (일상 대화)
1. **actually** - 사실은, 실제로
   - "Actually, I changed my mind." (사실, 마음이 바뀌었어)
   - "It's actually pretty good!" (실제로 꽤 좋아!)

2. **basically** - 기본적으로, 간단히 말해서
   - "Basically, it's not working." (간단히 말해서, 작동 안 해)
   - "I'm basically done." (거의 다 끝났어)

3. **definitely** - 확실히, 당연히
   - "I'll definitely come!" (꼭 갈게!)
   - "Definitely worth it." (확실히 그럴 만한 가치가 있어)

4. **honestly** - 솔직히
   - "Honestly, I don't know." (솔직히, 모르겠어)
   - "I'm honestly surprised." (정말 놀랐어)

5. **literally** - 말 그대로, 정말로
   - "I literally just got here." (진짜 방금 도착했어)
   - "It's literally amazing!" (정말 놀라워!)

#### Business (비즈니스)
1. **deadline** - 마감일
   - "When's the deadline?" (마감일이 언제야?)
   - "We need to meet the deadline." (마감일을 지켜야 해)

2. **budget** - 예산
   - "It's over budget." (예산 초과야)
   - "What's our budget?" (우리 예산이 얼마야?)

3. **schedule** - 일정
   - "Let me check my schedule." (일정 확인해볼게)
   - "Can we reschedule?" (일정 변경할 수 있을까?)

#### Emotions (감정 표현)
1. **frustrated** - 좌절한, 답답한
   - "I'm so frustrated!" (너무 답답해!)
   - "Don't get frustrated." (좌절하지 마)

2. **excited** - 신난, 흥분한
   - "I'm so excited!" (너무 신나!)
   - "Are you excited?" (기대돼?)

3. **overwhelmed** - 압도된, 벅찬
   - "I feel overwhelmed." (벅차게 느껴져)
   - "Don't be overwhelmed." (너무 부담스러워하지 마)

## 개발 우선순위

### Phase 1 (MVP) - 2주
1. 기본 데이터베이스 구조 설정
2. 500개 핵심 단어 데이터 입력
3. 단어 목록 및 상세 화면
4. 기본 학습 기능 (카드 넘기기)
5. 암기 체크 기능

### Phase 2 - 2주
1. 단어 추가/삭제 기능
2. 테스트 기능 구현
3. 학습 통계 화면
4. 카테고리별 필터링

### Phase 3 - 2주
1. 발음 기능
2. 학습 알림
3. 데이터 백업/복원
4. UI 개선 및 애니메이션

## 성능 최적화 가이드

### 데이터베이스 최적화
- 인덱스 활용: category, is_memorized 필드
- 페이징 처리: 한 번에 50개씩 로드
- 캐싱: 자주 사용하는 단어 메모리 캐싱

### UI 최적화
- Lazy loading for 리스트
- 이미지 최적화 (필요시)
- 불필요한 rebuild 방지

## 테스트 전략

### 단위 테스트
- 데이터베이스 CRUD 작업
- 단어 서비스 로직
- 학습 알고리즘

### 위젯 테스트
- 단어 카드 상호작용
- 네비게이션 플로우
- 폼 입력 검증

### 통합 테스트
- 전체 학습 플로우
- 데이터 저장 및 복원
- 성능 벤치마크

## Git 커밋 컨벤션

```
feat: 새로운 기능 추가
fix: 버그 수정
docs: 문서 수정
style: 코드 포맷팅
refactor: 코드 리팩토링
test: 테스트 코드
chore: 빌드 업무 수정
```

예시:
- `feat: Add word card swipe animation`
- `fix: Resolve database connection issue`
- `docs: Update README with setup instructions`

## 주의사항

1. **저작권**: 예문 생성 시 저작권 문제 없는 자체 제작 문장 사용
2. **데이터 보안**: 사용자 학습 데이터 암호화 저장
3. **접근성**: 시각 장애인을 위한 TalkBack 지원
4. **국제화**: 추후 다국어 지원 고려한 구조

## 디버깅 팁

### 데이터베이스 디버깅
```dart
// DB 경로 확인
final dbPath = await getDatabasesPath();
print('Database path: $dbPath');

// 쿼리 로깅
await db.rawQuery('SELECT * FROM words').then((result) {
  print('Query result: $result');
});
```

### 성능 모니터링
```dart
// 함수 실행 시간 측정
final stopwatch = Stopwatch()..start();
// ... 코드 실행
print('Execution time: ${stopwatch.elapsed}');
```

## 리소스 및 참고자료

- [Flutter 공식 문서](https://flutter.dev/docs)
- [SQLite Flutter Tutorial](https://docs.flutter.dev/cookbook/persistence/sqlite)
- [Material Design Guidelines](https://material.io/design)
- [실용 영어 회화 빈도 데이터](https://www.wordfrequency.info/)

## 연락처
프로젝트 관련 문의: https://github.com/nohdol97/word_study/issues

---
> Source: [nohdol97/word_study](https://github.com/nohdol97/word_study) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
