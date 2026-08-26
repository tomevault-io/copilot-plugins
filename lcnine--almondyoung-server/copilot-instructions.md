## wms-core-query-and-tx

> 서비스 클래스 리팩토링 규칙


## WMS 코어 쿼리/트랜잭션 규칙 (wms-core-query-and-tx)

### 목적
- 트랜잭션 맥락을 일관되게 전파하여 데이터 정합성 보장
- `core query` 패턴으로 쿼리 작성 통일(`select/from/join/where/orderBy`)
- enum/nullable 필드 타입 안전성 강화
- any/as 사용 금지 정책으로 타입 안정성 유지

### 적용 범위
- WMS 모듈의 모든 서비스/스토어 클래스
- 퍼블릭 DB 접근 메소드, 내부 전용 DB 헬퍼 메소드

### DbTx 타입 및 트랜잭션 헬퍼
- DbTx는 반드시 `apps/wms/database/schemas/wms-schema.ts`에서 export된 것을 import하여 사용한다. 로컬에서 타입 별칭으로 재선언 금지.
- 클래스 내부에 다음 헬퍼를 정의한다.

```ts
private async inTx<T>(fn: (tx: DbTx) => Promise<T>, tx?: DbTx) {
  return tx ? fn(tx) : this.db.transaction(fn);
}
```

### 메소드 시그니처 규칙
- 퍼블릭(외부 호출) DB 접근 메소드: 마지막 파라미터로 `tx?: DbTx`를 둔다.
  - 예: `async createX(dto: CreateXDto, tx?: DbTx) { ... }`
- 내부 전용(private) DB 헬퍼: `tx: DbTx`를 필수로 받는다.
  - 예: `private async loadX(tx: DbTx, params: ...) { ... }`

### 트랜잭션 전파 규칙
- 상위에서 받은 `tx`는 반드시 `this.inTx(exec, tx)`로 감싸고, 내부 DB 접근에는 `trx`만 사용한다.
- 트랜잭션 블록 내부에서 `this.db`로 쿼리하지 않는다. 항상 `trx`를 사용한다.
- 중첩 호출에서도 동일 `tx`를 끝까지 전파한다.

### 코어 쿼리 패턴
- 금지: `db.query.*`, `with` 기반의 리레이션 자동 조회
- 권장: 명시적 코어 쿼리
  - `select(...).from(...).innerJoin(...).where(...).orderBy(...)`
  - 비교/조건: Drizzle 연산자 사용(`eq`, `and`, `lt`, ...)

### 타입/정합성 규칙
- nullable → DTO/응답 타입으로 안전하게 정규화
  - 문자열: `value ?? ''`
  - 숫자: `value ?? 0`
  - 날짜: `value ?? undefined`
- enum 값은 스키마 정의만 사용한다.
  - 예: `fulfillmentMode` 비교 시 `'drop_ship'` 사용. 존재하지 않는 `'direct_ship'` 사용 금지.

### any/as 사용 금지 정책
- any와 `as` 캐스팅은 원칙적으로 금지한다.
- 반드시 사용해야 하는 예외 상황이 생기면, 다음을 선행한다.
  1) 사유를 설명(타입 한계, 외부 라이브러리 제약 등)
  2) 팀 구성원(리뷰어) 승인 획득
  3) 최소 범위로 국소화하고 TODO가 아닌 즉시 타입 보완 방안을 문서화

### 리팩토링 절차(요청형 가이드)
1) DbTx 처리
   - 서비스 상단에서 `DbTx`를 `wms-schema.ts`에서 import
   - 클래스에 `inTx` 헬퍼 추가
2) 시그니처 정리
   - 퍼블릭 DB 메소드에 `tx?: DbTx` 추가
   - 내부 전용 메소드는 `tx: DbTx`
3) 트랜잭션 래핑/전파
   - 본문을 `return this.inTx(async (trx) => { ... }, tx)`로 래핑
   - 내부 DB 접근은 전부 `trx`로 교체
4) 코어 쿼리 전환
   - `db.query.*`/`with` 제거 → `select/from/innerJoin/where/orderBy`로 재작성
5) 비즈니스 검증/업데이트
   - 상태 검증은 코어 쿼리 조회 결과로 수행
   - 수치 비교/집계는 Drizzle 연산자 및 JS 안전 연산(`?? 0`) 활용
6) DTO 매핑
   - nullable 정규화, enum 값 엄수, 캐스팅 금지
7) 로깅/예외
   - 변경 로그 남기기, `NotFound/BadRequest/Conflict` 등 표준 예외 사용

### 리뷰 체크리스트
- DbTx import 출처가 `wms-schema.ts`인가?
- 퍼블릭 메소드 `tx?: DbTx`, 내부 메소드 `tx: DbTx` 적용 여부
- `this.inTx(exec, tx)` 사용 및 하위 호출까지 `tx` 전파 여부
- 트랜잭션 블록 내부에서 `trx`만 사용했는가? (`this.db` 금지)
- 코어 쿼리만 사용하고 `db.query/with` 잔존 없음
- nullable/enum 정규화 및 응답 타입 일치
- any/as 사용이 없는가? (예외 시 사유/승인/국소화 확인)

### 예시 스니펫
```ts
// DbTx import (서비스 파일)
import { DbTx, wmsTables } from '.../wms-schema';

private async inTx<T>(fn: (tx: DbTx) => Promise<T>, tx?: DbTx) {
  return tx ? fn(tx) : this.db.transaction(fn);
}

async startPicking(batchId: string, tx?: DbTx) {
  return this.inTx(async (trx) => {
    const [batch] = await trx
      .select({ id: wmsTables.outboundBatches.id, status: wmsTables.outboundBatches.status })
      .from(wmsTables.outboundBatches)
      .where(eq(wmsTables.outboundBatches.id, batchId))
      .limit(1);

    if (!batch) throw new NotFoundException(...);
    if (batch.status !== 'created') throw new ConflictException(...);

    await trx.update(wmsTables.outboundBatches)
      .set({ status: 'picking', startedAt: new Date() })
      .where(eq(wmsTables.outboundBatches.id, batchId));
  }, tx);
}
```

---
> Source: [LCNINE/almondyoung-server](https://github.com/LCNINE/almondyoung-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
