## caravanshare

> 카라반(캠핑카)을 소유한 사람과 이를 이용하고 싶은 사람을 연결하는 공유 플랫폼

# 🚐 Project: CaravanShare (카라반 쉐어)

## 1. 프로젝트 개요
카라반(캠핑카)을 소유한 사람과 이를 이용하고 싶은 사람을 연결하는 공유 플랫폼

### 👥 핵심 역할
* **공급자 (Host):** 카라반을 소유하고 있으며, 유휴 시간에 이를 임대하여 수익을 창출하려는 사용자.
* **수요자 (Guest):** 카라반을 소유하지 않았지만, 저렴한 비용으로 카라반 여행을 경험하고 싶은 사용자.
* **핵심 가치:** 유휴 자산의 활용, 저렴한 여행 경험, 커뮤니티


? 핵심 요구사항
Phase 1: MVP(최소 기능 제품)
1.1 사용자 관리
- [x] 회원가입 (호스트/게스트 구분)
- [x] 프로필 관리 (이름, 연락처, 평가)
- [x] 인증/인가 (로그인, 권한 관리)
- [x] 사용자 신뢰도 시스템
- [ ] 신원 확인 
1.2 카라반 정보 관리
- [x] 카라반 등록 (호스트)
- [x] 카라반 정보: 수용 인원, 편의시설, 사진, 위치
- [x] 카라반 검색/조회 (게스트)
- [x] 카라반 상태 관리 (사용가능, 예약됨, 정비중)
1.3 예약 시스템
- [x] 예약 신청 (게스트)
- [x] 예약 승인/거절 (호스트)
- [x] 예약 날짜 관리 (캘린더)
- [x] 중복 예약 방지
1.4 결제 및 가격
- [x] 일일 요금 설정 (호스트)
- [x] 가격 계산 (렌탈 기간 기반)
- [x] 선결제 시스템
- [x] 결제 이력 조회
1.5 리뷰/평가
- [x] 거래 후 리뷰 작성
- [x] 평점 시스템 (1-5점)
- [x] 호스트/게스트 신뢰도 반영




> ?? 데이터 모델 (설계 과제)

현재 문제가 있는 코드 (개선 필요)

# ? 나쁜 설계 예시nclass CaravanApp:n def __init__(self):n self.users = []n self.caravans = []n self.reservations = []n [self.reviews](<http://self.reviews>) = []n self.payments = []n n def create_reservation(self, user_id, caravan_id, start_date, end_date, price):n # 복잡한 로직이 한 메서드에 집중n user = Nonen for u in self.users:n if u['id'] == user_id:n user = un breakn n caravan = Nonen for c in self.caravans:n if c['id'] == caravan_id:n caravan = cn breakn n if user is None or caravan is None:n return Falsen n # 중복 예약 검사n for r in self.reservations:n if (r['caravan_id'] == caravan_id and n ((start_date >= r['start_date'] and start_date <= r['end_date']) orn (end_date >= r['start_date'] and end_date <= r['end_date']))):n return Falsen n # 결제 처리n if user['balance'] < price:n return Falsen n user['balance'] -= pricen n # 예약 생성n reservation = {n 'id': len(self.reservations) + 1,n 'user_id': user_id,n 'caravan_id': caravan_id,n 'start_date': start_date,n 'end_date': end_date,n 'status': 'pending',n 'price': pricen }n n self.reservations.append(reservation)n return Truen n def get_caravan_info(self, caravan_id):n # 검색이 비효율적n for c in self.caravans:n if c['id'] == caravan_id:n return cn return Nonen

문제점 분석 (바이브코딩 관점)

1. 단일 책임 원칙 위반

CaravanApp 클래스가 모든 기능을 담당
사용자 관리, 예약, 결제 등이 섞여있음

2. 낮은 응집도

관련 데이터가 분리되어 있음
사용자의 인증 정보와 프로필이 혼재

3. 비효율적인 검색

리스트 순회로 O(n) 성능
대규모 데이터에서 성능 저하

4. 중복 코드

사용자/카라반 검색 로직 반복
중복 예약 검사 로직이 복잡함

5. 강한 결합도

결제와 예약이 직접 결합
도메인 로직이 흩어져 있음

6. 테스트 불가능

의존성이 하드코딩됨
모킹(Mocking)이 어려움


> ? 바이브코딩 실전 과제

과제 1: 깨끗한 도메인 모델 설계

목표: 책임을 분리하고 응집도 높은 클래스 구조 설계

요구사항:
User 클래스: 사용자 관련 책임만
Caravan 클래스: 카라반 정보 관리
Reservation 클래스: 예약 정보 관리
Payment 클래스: 결제 로직
Review 클래스: 리뷰/평가

설계 원칙:
단일 책임 원칙 (SRP)
개방/폐쇄 원칙 (OCP)
의존성 역전 원칙 (DIP)


과제 2: 복잡한 비즈니스 로직 분리

목표: 예약 검증 로직을 명확하게 분리

요구사항:
ReservationValidator 클래스 설계
각 검증을 별도의 메서드로 분리
각 검증이 독립적으로 테스트 가능해야 함


과제 3: 효율적인 데이터 구조와 검색 알고리즘

목표: 성능 최적화와 가독성 있는 코드

요구사항:
ReservationRepository 클래스 설계
인덱싱을 통한 O(1) 검색 구현
날짜별 충돌 검사 최적화


과제 4: 변수명과 함수명의 명확성

목표: 자기설명적 코드 작성

네이밍 가이드:

변수명: user_id (not uid)

함수명: 동사로 시작 create_, validate_

Boolean 함수: is_, has_, can_ 접두사

상수: 대문자 MIN_RESERVATION_DAYS = 1


과제 5: 에러 처리와 예외 관리

목표: 견고한 에러 처리 전략

요구사항:
커스텀 예외 클래스 정의
각 에러 상황에 맞는 예외 사용
명확한 에러 메시지 제공


과제 6: 테스트 가능한 코드 작성

목표: 단위 테스트 작성 가능하도록 설계

요구사항:
의존성 주입 (Dependency Injection) 패턴 사용
Mock 객체 활용 가능한 구조
최소 70% 테스트 커버리지


과제 7: 디자인 패턴 적용

목표: 적절한 디자인 패턴 선택과 적용

패턴 활용:

팩토리 패턴: 예약 객체 생성

전략 패턴: 할인 계산

옵저버 패턴: 알림

리포지토리 패턴: 데이터 접근


> ? 구현 평가 기준

1단계 평가 (기본)

[ ] 클래스가 단일 책임 원칙을 따르는가?
[ ] 변수명과 함수명이 명확한가?
[ ] 중복 코드가 없는가?

2단계 평가 (중급)

[ ] 에러 처리가 명확하고 적절한가?
[ ] 의존성이 주입되는가?
[ ] 기본 단위 테스트가 작성되었는가?

3단계 평가 (고급)

[ ] 디자인 패턴이 적절하게 적용되었는가?
[ ] 코드 복잡도가 낮은가? (Cyclomatic Complexity)
[ ] 성능 최적화가 고려되었는가?

코드 품질 지표

순환 복잡도 (Cyclomatic Complexity): 10 이하
함수 길이: 30줄 이하
클래스 크기: 200줄 이하
테스트 커버리지: 70% 이상

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Na0927s)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Na0927s)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
