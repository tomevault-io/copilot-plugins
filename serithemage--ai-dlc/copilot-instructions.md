## ai-dlc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

이 저장소는 AWS Raja SP의 **AI-DLC (AI-Driven Development Lifecycle)** 방법론을 한국어로 번역한 문서입니다. 이것은 소프트웨어 개발 프로젝트가 아니라 **문서 전용 저장소** 입니다.

- **원저자:** Raja SP (Amazon Web Services)
- **목적:** AI 네이티브 소프트웨어 개발 방법론에 대한 한국어 교육 자료 제공
- **호스팅:** GitHub Pages (https://serithemage.github.io/AI-DLC/)
- **저작권:** 모든 권리는 원저자에게 있음

## Documentation Structure

```
AI-DLC/
├── AI-Driven Development Liftcycle.md    # 주 문서 (2,837+ 줄)
├── index.md                               # GitHub Pages 한국어 버전
├── README.md                              # 저장소 개요
├── CLAUDE.md                              # Claude Code 작업 가이드
├── _config.yml                            # Jekyll 설정 (theme: cayman)
├── ja/                                    # 일본어 버전
│   ├── index.md                           # 일본어 메인 문서
│   ├── README.md                          # 일본어 개요
│   └── CLAUDE.md                          # 일본어 가이드
└── images/                                # 다이어그램
    ├── ai-dlc-framework.png
    ├── ai-dlc-workflow.png
    └── ai-dlc-detailed-workflow.png
```

**중요:** `AI-Driven Development Liftcycle.md`를 수정할 때는 `index.md`도 동기화해야 합니다.

## Translation Standards

### 영문 유지 용어 (Technical Terms)
다음 용어는 영어로 유지:
- **Core Concepts:** Intent, Unit, Bolt, Domain Design, Logical Design, Deployment Unit
- **Design Techniques:** DDD (Domain-Driven Design), BDD (Behavior-Driven Design), TDD (Test-Driven Development)
- **Ceremonies:** Mob Elaboration, Mob Construction
- **Technologies:** CI/CD, DevOps, GenAI, LLM

### 한국어 번역 용어
| English | Korean |
|---------|--------|
| Intent | 의도 |
| Unit | 유닛 |
| Bolt | 볼트 |
| Inception | 인셉션 |
| Construction | 구성 |
| Operations | 운영 |
| Greenfield | 그린필드 |
| Brownfield | 브라운필드 |

### 번역 원칙
- 기술 용어는 영문 유지, 설명은 한국어로 번역
- 원문의 마크다운 구조 유지 (헤딩, 테이블, 코드 블록)
- 다이어그램 참조 경로 정확히 유지

## Key AI-DLC Concepts

AI-DLC 방법론의 핵심 개념 이해를 위한 참고:

### 3단계 프레임워크
1. **Inception (인셉션):** Intent 수집 → Unit 분해 (Mob Elaboration 통해)
2. **Construction (구성):** Domain Design → Logical Design → 코드 생성 (Mob Construction 통해)
3. **Operations (운영):** 배포, 모니터링, 유지보수

### 핵심 산출물
- **Intent:** 고수준 비즈니스 목표
- **Unit:** 응집력 있는 작업 요소
- **Bolt:** 신속한 반복 주기 (시간/일 단위)
- **Domain Design:** DDD를 활용한 비즈니스 로직 모델링
- **Logical Design:** 아키텍처 패턴을 통한 NFR 적용

### 10가지 핵심 원칙
문서에서 자세히 설명되는 AI-DLC의 기초 철학 (재상상, 대화 방향 전환, 핵심 설계 기법 통합 등)

## GitHub Pages Workflow

### 자동 배포
- `main` 브랜치에 푸시하면 GitHub이 자동으로 Pages 빌드/배포
- Jekyll이 `_config.yml` 설정에 따라 정적 사이트 생성
- **수동 빌드 명령 불필요** (GitHub Actions가 자동 처리)

### 로컬 미리보기 (선택사항)
```bash
bundle install
bundle exec jekyll serve
# http://localhost:4000 에서 확인
```

## Content Management Guidelines

### 문서 수정 시
1. `AI-Driven Development Liftcycle.md` 수정
2. 동일한 내용을 `index.md`에도 반영
3. 다이어그램 추가/변경 시 `images/` 디렉토리에 저장
4. 마크다운 이미지 참조 경로 확인: `![설명](images/파일명.png)`

### 커밋 메시지
- 한국어 또는 영어 사용 가능
- Co-authored-by 태그 포함 권장

### 저작권 표기
원저자 귀속 표기 유지:
- README.md 하단의 저작권 문구 보존
- 원문 링크 유지

## Important Notes

- **빌드/테스트/린트 명령 없음:** 이 저장소는 문서 전용이므로 `package.json`, 테스트 프레임워크, 린터 설정이 없습니다
- **의존성 설치 불필요:** 순수 Markdown + Jekyll (GitHub에서 자동 처리)
- **코드 실행 없음:** 실행 가능한 코드가 포함되어 있지 않습니다
- **번역 정확성 우선:** AI-DLC 방법론의 의도를 정확히 전달하는 것이 핵심 목표

---
> Source: [serithemage/AI-DLC](https://github.com/serithemage/AI-DLC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
