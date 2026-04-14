## navo

> - 터미널 명령: CMD/PowerShell 우선(Windows 환경 고려).

# Navo Workspace Rules

- 응답/로그 언어: 한국어 우선.
- 터미널 명령: CMD/PowerShell 우선(Windows 환경 고려).
- 로깅 가이드: 제품 로그는 JSONL(파일), 콘솔은 간결 안내. 과도한 디버깅 로그 지양.
- 표준/일관성: 프로젝트 문서 및 설정 우선 준수. 변경이 표준을 훼손하면 유지 방안 먼저 제안.

## Proactive Challenge (능동적 제안 원칙)
- 사용자가 요청한 변경이 표준/베스트 프랙티스와 충돌하거나 부작용(성능/보안/운영) 우려가 있으면,
  1) 예상 영향과 트레이드오프를 1~2줄로 요약,
  2) 더 안전한 대안(현상 유지/하이브리드/스코프 축소)을 제안,
  3) 짧은 확인 후 실행.
- 예: JSON 로그의 level 고정폭 패딩 요청 → JSONL에서는 불필요하므로 유지 권고, 콘솔만 가독성 보정 제안.

## 문서/정책
- 프리뷰/편집은 VFS, 물리 파일 생성은 IR 이후 배포 단계(문서 기준 유지).
- 변경 후에는 관련 문서(ADR/tech docs)를 가능한 한 즉시 갱신.

## 커밋/작업 단위
- 가능한 기능 단위로 묶어서 커밋. 사소한 다건 커밋 지양. 커밋 메시지는 한글로 작성. 커밋 방법은 파일로 한다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vogomkkim) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
