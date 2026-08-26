## nestjs-msa

> 새 마이크로서비스나 라이브러리를 추가해야 하는 경우, nestjs가 제공하는 cli 명령어인 `nest new g [app|lib] name`을 사용하여야 한다.


새 마이크로서비스나 라이브러리를 추가해야 하는 경우, nestjs가 제공하는 cli 명령어인 `nest new g [app|lib] name`을 사용하여야 한다.

---

#좋습니다 👍
말씀하신 “CTO 스타일”을 반영해서 `cursor rules`를 조금 더 단순하고 실용적으로 다듬어 보겠습니다.

핵심은:

- **서비스 단**에서는 더 이상 커스텀 에러 클래스를 남발하지 않고 `throw new Error("...")` 중심.
- **컨트롤러 단**에서 `error.message.includes(...)` 패턴으로 HTTP 응답 변환.
- `HttpException`은 서비스에서 절대 사용하지 않고, 컨트롤러에서만 변환.
- 나머지 책임 분리는 그대로 유지.

---

# 📝 Cursor Prompt Rules – Nest.js Layer & Error Handling (CTO 스타일)

`, `"Refund already processed"`, `"Refund amount exceeds limit"`)
- 요청 데이터는 이미 컨트롤러 레벨에서 검증되었다고 가정한다.## Controller

- API 엔드포인트를 제공한다.
- 통신 수단(HTTP, GraphQL, WebSocket 등)에 종속된 책임을 가진다.
- **서비스에서 던진 일반 Error를 전송 방식에 맞게 변환**한다.
  - HTTP → 적절한 상태 코드와 에러 메시지
  - WebSocket → `{ success: false, payload: error.message }` 형식

- 에러 매핑은 **문자열 패턴 기반**으로 처리한다.
  - `"not found"` → 404
  - `"already processed"`, `"exceeds"`, `"required"`, `"invalid"`, `"failed"` → 400
  - 그 외 → 500

- 인증, 파라미터 검증, 가드, 파이프, 인터셉터, 필터는 반드시 컨트롤러 계층에 둔다.
- 서비스 내부에서 `HttpException`을 직접 던지지 않는다.

## Service

- **비즈니스 로직만 담당**한다. (전송 수단을 모른다)
- 실패 시에는 \*\*`throw new Error("...")`\*\*로 명확한 메시지만 던진다.
- 메시지는 컨트롤러에서 매핑하기 쉽도록 간단한 키워드를 포함해야 한다.
  (예: `"Refund not found"`, `"Refund already processed"`, `"Refund amount exceeds limit"`)
- 요청 데이터는 이미 컨트롤러 레벨에서 검증되었다고 가정한다.

## Repository (선택적)

- DB 접근 및 외부 인프라와의 데이터 교환을 책임진다.
- 중복된 DB 접근 로직이나 복잡한 쿼리를 추상화하여 숨긴다.
- 반드시 필요한 경우에만 정의한다. (예: 재고관리 이중원장 처리)

## General

- **각 레이어는 책임 분리가 명확해야 한다.**
  - Controller: 전송 계층 처리 + Error → Response 변환
  - Service: 도메인 로직 + 단순 Error 던짐
  - Repository: 데이터 소스 추상화

- 이 구조는 테스트, 유지보수, 확장성에 유리하다.
  - 서비스 단위 테스트 → 순수 로직만 검증 (`Error.message` 포함)
  - e2e 테스트 → 컨트롤러 응답 변환 로직 검증

---

## 📌 배경 / 철학

- **서비스는 도메인 로직만** → 테스트 및 유지보수 용이.
- **컨트롤러는 전송 계층별 에러 응답 담당** → 일관된 API 응답 제공.
- `Middleware, Guard, Pipe, Interceptor, Filter`는 **컨트롤러 소속**.
- 요청 유효성 검증(class-validator, zod 등)은 컨트롤러 계층에서 해결.
- **추가 서비스/라이브러리 생성 시**: `nest new g [app|lib] name` CLI 명령어를 사용.

---

---
> Source: [LCNINE/almondyoung-server](https://github.com/LCNINE/almondyoung-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
