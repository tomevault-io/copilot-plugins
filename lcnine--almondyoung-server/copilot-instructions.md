## wms-transaction-rule

> WMS 모듈 전반에서 트랜잭션(tx) 전파/경계 규칙과 표준 시그니처 정의


## WMS 트랜잭션 전달 규칙 (tx propagation)

이 규칙은 WMS 모듈 전반에서 서로 다른 서비스/리포지토리 메소드 간에 동일 트랜잭션을 안정적으로 유지하기 위한 공통 컨벤션을 정의합니다.

### 목표
- 트랜잭션 경계의 단일화: 상위에서 시작한 트랜잭션을 하위 호출로 일관되게 전달
- 일관된 메소드 시그니처: DB 접근 메소드는 공통된 `tx` 파라미터 규약을 따름
- 중첩 호출의 안전성: 중첩된 서비스/스토어 호출에서도 하나의 트랜잭션으로 동작

### 공통 타입: DbTx
서비스/스토어 파일 상단에 다음 별칭을 선언해 트랜잭션 실행 컨텍스트 타입을 통일합니다.

```ts
type DbTx = Parameters<Parameters<TypedDatabase<typeof wmsTables>['transaction']>[0]>[0];
```

주의:
- 현재는 각 파일에서 위 별칭을 로컬 선언합니다. 공용 타입 모듈로 승격하는 경우, 동일 이름 `DbTx`로 export/import 하여 규칙을 유지합니다.

### 표준 트랜잭션 헬퍼: inTx
아래 헬퍼를 클래스 내부에 정의하여, 전달된 트랜잭션이 있으면 재사용하고 없으면 새 트랜잭션을 시작합니다.

```ts
private async inTx<T>(fn: (tx: DbTx) => Promise<T>, tx?: DbTx) {
  return tx ? fn(tx) : this.db.transaction(fn);
}
```

### 메소드 시그니처 규칙
- DB에 접근하는 퍼블릭 서비스/스토어 메소드: 마지막 파라미터로 `tx?: DbTx`를 둡니다.
  - 예: `async createFoo(dto: CreateFooDto, tx?: DbTx) { ... }`
- 트랜잭션 내부 전용(private) 헬퍼: `tx: DbTx`를 필수로 받습니다.
  - 예: `private async getOnHandQuantity(tx: DbTx, params: ...) { ... }`

### 호출 규칙
- 상위에서 `tx`를 전달받았다면, 반드시 그대로 하위 호출까지 전파합니다.
- 새로운 트랜잭션 경계를 열어야 할 경우에도 `this.db.transaction(...)`을 직접 사용하지 말고, 반드시 `this.inTx(exec, tx)` 패턴을 사용합니다.
  - 상위에서 `tx`가 넘어온 경우에는 동일 트랜잭션을 재사용하고, 없는 경우에만 새 트랜잭션을 시작합니다.

### 올바른/잘못된 예시

올바름: 상위에서 받은 `tx`를 `inTx`와 모든 하위 호출에 전달
```ts
async simpleInbound(dto: SimpleInboundDto, tx?: DbTx) {
  return this.inTx(async (trx) => {
    await this.eventStore.createEvent({ /* ... */ }, trx);
    await this.putawayFromOrigin({ /* ... */ }, trx);
  }, tx);
}
```

잘못됨: 상위에서 받은 `tx`를 `inTx`에 전달하지 않아 새 트랜잭션이 열림
```ts
async simpleInbound(dto: SimpleInboundDto, tx?: DbTx) {
  // BUG: 아래 호출은 항상 새로운 트랜잭션을 시작함
  return this.inTx(async (trx) => { /* ... */ });
}
```

올바른 내부 헬퍼: 내부 전용은 `tx: DbTx`를 필수로 요구
```ts
private async applyProjection(tx: DbTx, params: { skuId: string; /* ... */ }) { /* ... */ }
```

### 중첩 트랜잭션 정책
- `inTx`는 전달된 `tx`가 있으면 재사용하고, 없으면 새 트랜잭션을 시작합니다.
- 서비스 A → 서비스 B → 스토어 C 식의 중첩 호출에서도, A에 `tx`가 있으면 B와 C까지 동일 트랜잭션을 유지합니다.

### 리뷰 체크리스트
- DB 접근 메소드의 마지막 파라미터가 `tx?: DbTx`인가? (퍼블릭)
- 내부 전용 헬퍼는 `tx: DbTx`인가? (프라이빗)
- `this.inTx(exec, tx)`로 상위에서 받은 `tx`를 빠짐없이 전달했는가?
- 하위 서비스/스토어 호출 시 `tx`를 누락하지 않았는가?
- 새로운 트랜잭션이 정말 필요한 곳에서만 생성되는가?

### 네이밍/기타 컨벤션
- 트랜잭션 파라미터 이름은 항상 `tx`로 통일합니다.
- `inTx`의 첫 번째 인자는 `(tx: DbTx) => Promise<T>` 형태를 사용합니다.
- 컨트롤러 등 상위 계층에서 트랜잭션을 열어 요청 단위로 유지하려면, 시작 시점에서 만든 `tx`를 모든 서비스 호출에 전달합니다.

---
> Source: [LCNINE/almondyoung-server](https://github.com/LCNINE/almondyoung-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
