## youtube-automation

> 인포그래픽/애니메이션 기반 페이스리스(Faceless) 유튜브 채널 자동화 시스템

# YouTube Automation Project

## 개요
인포그래픽/애니메이션 기반 페이스리스(Faceless) 유튜브 채널 자동화 시스템

## 기술 스택
- Python 3.13+ (파이프라인 코어)
- n8n / Make.com (워크플로우 오케스트레이션)
- FFmpeg (영상 렌더링)
- Canva API / Remotion (시각 에셋 생성)

## 디렉토리 구조
```
youtube-automation/
├── CLAUDE.md
├── docs/                  # 기획 문서
│   ├── research.md        # 벤치마크 리서치
│   ├── plan.md            # 개발 기획서
│   └── architecture.md    # 기술 아키텍처
├── pipeline/              # 자동화 파이프라인 코드
│   ├── ideation/          # 1단계: 기획/대본
│   ├── audio/             # 2단계: 음성 생성
│   ├── visual/            # 3단계: 시각 에셋
│   ├── assembly/          # 4단계: 영상 조립
│   └── publish/           # 5단계: 자동 배포
├── templates/             # 영상 템플릿 (패턴별)
├── assets/                # 재사용 에셋 라이브러리
└── config/                # 설정 파일
```

## 규칙
- 영상 소스는 실사 촬영 없이 인포그래픽/디자인 기반만 사용
- 자막과 프롬프트는 자동 생성 필수
- 패턴별 버전 분리하여 독립 개발

---
> Source: [weonguyeon/youtube-automation](https://github.com/weonguyeon/youtube-automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
