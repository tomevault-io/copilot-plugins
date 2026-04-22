## sea-survival

> ppt-creation-guide

# Unity 게임 PPT 작성 가이드라인

## 프로젝트 개요
이 프로젝트는 바다 생존 테마의 뱀파이어 서바이벌 스타일 Unity 게임입니다.

## PPT 구성 요구사항 (10-15장)

### 1. 게임 소개 (2-3장)
- 장르 및 컨셉: [장르 및 컨셉.md](mdc:Assets/dev_doc/ppt/장르 및 컨셉.md)
- 게임 플레이 방법: [어떻게 플레이 하나요.md](mdc:Assets/dev_doc/ppt/어떻게 플레이 하나요.md)

### 2. 핵심 시스템 구현 (7-9장)

#### 2.1 이동 시스템 (1-2장)
- 핵심 파일: [Player.cs](mdc:Assets/sea-survival/Scripts/Players/Player.cs)
- 구현 이유: 물 위/아래 환경에서의 차별화된 이동 경험 제공
- 핵심 코드 설명:
  - WASD 기본 이동
  - 물 위에서는 숨쉬기 모드 (Space 키)
  - 숨쉬기 모드에서는 이동속도 감소 및 Y축 고정

#### 2.2 산소 시스템 (1-2장)
- 핵심 파일: [Player.cs](mdc:Assets/sea-survival/Scripts/Players/Player.cs) (lines 27-32, 81-106)
- 구현 이유: 바다 테마에 맞는 생존 요소 추가로 게임 긴장감 조성
- 핵심 코드 설명:
  - 물 아래에서 산소 감소
  - 산소 부족 시 데미지
  - 숨쉬기 모드에서 산소 회복

#### 2.3 성장/전투 시스템 (2-3장)
- 핵심 파일: [Card.cs](mdc:Assets/sea-survival/Scripts/CardSystem/Card.cs)
- 구현 이유: 뱀파이어 서바이벌의 핵심 재미 요소인 무작위 성장 시스템
- 핵심 코드 설명:
  - 카드 기반 스킬/스탯 강화
  - 무기 시스템과 연동
  - 동적 카드 설명 시스템

#### 2.4 스테이지 시스템 (1장)
- 핵심 파일: [StageManager.cs](mdc:Assets/sea-survival/Scripts/StageSystem) 폴더
- 구현 이유: 횡스크롤 진행형 스테이지로 목표 의식 제공

#### 2.5 UI/HUD 시스템 (1장)
- 핵심 파일: [PlayerExpUI.cs](mdc:Assets/sea-survival/Scripts/Players/PlayerExpUI.cs)
- 구현 이유: 플레이어가 게임 상태를 직관적으로 파악할 수 있도록

### 3. 플레이 영상 (1장)
- 실제 게임플레이 시연 영상

## 작성 지침

### 코드 표시 방법
- 핵심 코드 스니펫만 표시 (3-5줄 내외)
- 복잡한 구현보다는 **왜 이렇게 구현했는지** 설명에 집중

### 졸업작품 한계 언급
- "졸업작품이라는 시간적 한계로 인해 핵심 기능에 집중"
- "더 복잡한 시스템은 차후 확장 예정"

### 슬라이드 디자인
- 깔끔하고 현대적인 디자인
- 적절한 애니메이션과 전환 효과
- 코드는 syntax highlighting 적용

## 주요 구현 철학
1. **바다 테마와 게임성의 조화**: 산소 시스템으로 바다의 특징을 게임플레이에 녹임
2. **친숙함과 신선함의 균형**: 뱀파이어 서바이벌의 검증된 재미 + 바다 테마의 차별화
3. **직관적인 조작**: 복잡하지 않지만 전략적 사고가 필요한 시스템

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/santutu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
