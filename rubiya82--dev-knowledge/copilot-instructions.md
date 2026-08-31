## dev-knowledge

> 이 저장소는 개인 개발 Knowledge Base다. 개발 중 확인한 기술, 장애 원인, 해결 방법, 테스트 결과 및 설계 원칙을 장기간 축적한다.

# Repository Instructions

## Repository Purpose

이 저장소는 개인 개발 Knowledge Base다. 개발 중 확인한 기술, 장애 원인, 해결 방법, 테스트 결과 및 설계 원칙을 장기간 축적한다.

## Language

- 설명 문서는 기본적으로 한국어로 작성한다.
- 클래스명, API, 표준 기술 용어는 원래 영어 표기를 유지한다.
- 모든 기술 용어를 억지로 한글화하지 않는다.

## Documentation Rules

새 문서를 작성하기 전 다음을 수행한다.

1. 기존 관련 문서를 검색한다.
2. 중복 문서가 있으면 새 문서보다 기존 문서 확장을 우선 검토한다.
3. 알맞은 카테고리를 결정한다.
4. 기존 문서와 상호 링크할 수 있는지 검토한다.

문서 작성 후 다음을 수행한다.

1. 내부 링크를 확인한다.
2. 코드 블록 언어를 지정한다.
3. 민감정보 포함 여부를 확인한다.
4. `mkdocs build --strict`를 실행한다.
5. 빌드 오류가 없는지 확인한다.

모든 문서는 필요한 섹션만 사용한다. 일반적으로 개요, 문제/증상, 원인, 해결 방법, 코드/명령어, 주의사항, 테스트 환경, 결론, 참고 자료를 활용한다. 관련 문서는 Markdown 링크로 연결하고 같은 내용을 여러 파일에 중복하지 않는다.

## Security

회사명, 고객사명, 실제 프로젝트명, 사내 Repository URL·서버 주소·IP, 계정·Password·API Key·Access Token·라이선스 키, 내부 경로·설비 정보, 기밀 Source Code, 회사 고유 Class/Product 이름을 작성하지 않는다. 업무 사례는 일반적인 기술 사례로 변환하며, 의심되는 구체 정보는 삭제하고 일반화한다.

## Style

- 장황한 개론보다 실제 문제·원인·해결에 도움이 되는 내용을 우선한다.
- 확인되지 않은 내용은 사실처럼 단정하지 않는다.
- 명령어와 코드에는 필요한 설명을 덧붙인다.

---
> Source: [Rubiya82/dev-knowledge](https://github.com/Rubiya82/dev-knowledge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
