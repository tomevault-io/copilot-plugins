## db-field-git

> 코드 변경 시 아래에 해당하면 **사용자가 별도로 요청하지 않아도** `NOTION_API_DOCUMENTATION.md`를 반드시 수정한다.

# 프로젝트 규칙 (New_Db-Field)

## API 문서 자동 갱신
코드 변경 시 아래에 해당하면 **사용자가 별도로 요청하지 않아도** `NOTION_API_DOCUMENTATION.md`를 반드시 수정한다.

- **기존 API 구조 변경**: 경로, 메서드, 요청/응답 스키마, 파라미터, 인증 방식 등이 바뀐 경우
- **새 API 추가**: 새로운 엔드포인트가 생성된 경우

문서 수정 시 다음을 반영한다.
- 엔드포인트 경로 및 HTTP 메서드
- Request Body / Query 파라미터 형식 및 필드 설명
- Response 성공/실패 예시 (JSON 구조)
- 인증 요구사항 (세션, X-Auth-Token 등)
- 해당 API를 처리하는 컨트롤러/메서드가 코드와 일치하는지 확인

---
> Source: [Lim-dbeng/Db-Field_git](https://github.com/Lim-dbeng/Db-Field_git) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
