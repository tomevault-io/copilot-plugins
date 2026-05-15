## security-secrets

> 시크릿/보안 관련 금지 및 가이드

## 보안/시크릿 가이드

- 민감정보는 `.env` 등 시크릿 매니저/CI 변수로만 관리. 저장소 커밋 금지
- 금지 파일 예시: `google-services.json`, `serviceAccountKey.json`, `firebase-adminsdk-*.json`, `client_secret*.json`, `*.p12`, `*.pem`, `*.pfx`, `*.keystore`, `AppleAuthKey_*.p8`, `aws_credentials`, `*.ovpn`, `postman_environment*.json`, 압축 아카이브(`*.zip`, `*.rar`, `*.7z`, `*.tar`, `*.gz`)
- `.gitignore`에 상기 패턴을 추가하고, 과거 커밋 노출 시 즉시 폐기/회전
- 필요 시 pre-commit 훅으로 차단 규칙 적용 권장

참고: [템플릿-AI 코딩,바이브 코딩 표준 정책.md](mdc:템플릿-AI 코딩,바이브 코딩 표준 정책.md)

---
> Source: [jang-seung-hee/rental-news](https://github.com/jang-seung-hee/rental-news) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
