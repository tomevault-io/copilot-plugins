## drizzle-orm

> 알겠어요. 지금 CTO가 작성해둔 `schema.ts` + `types.ts` 패턴을 그대로 재사용할 수 있도록 **Cursor Rules** 스타일 가이드를 만들어 드리겠습니다. 이 규칙만 보고도 Cursor가 자동으로 코드/타입을 생성할 수 있게 하는 것이 목적이죠.

알겠어요. 지금 CTO가 작성해둔 `schema.ts` + `types.ts` 패턴을 그대로 재사용할 수 있도록 **Cursor Rules** 스타일 가이드를 만들어 드리겠습니다. 이 규칙만 보고도 Cursor가 자동으로 코드/타입을 생성할 수 있게 하는 것이 목적이죠.

---

## 📝 Cursor Rules – Drizzle ORM + Typescript 타입 규칙
jsdoc
### 1. 스키마 정의 규칙 (`schema.ts`)

- **모든 테이블 정의는 하나의 `schema.ts`에서만 한다.**

  - `pgTable`과 `uuid`, `varchar`, `jsonb`, `timestamp` 등 Drizzle DSL을 사용.
  - `index`, `uniqueIndex`, `foreignKey`를 함께 정의해 구조를 명확히 한다.

- **관계(Relations)는 별도 export로 작성**

  ```ts
  export const productCategoriesRelations = relations(
    productCategories,
    ({ one, many }) => ({
      parent: one(productCategories, {
        fields: [productCategories.parentId],
        references: [productCategories.id],
      }),
      children: many(productCategories),
    }),
  );
  ```

- **전체 스키마 객체를 만들어 export**

  ```ts
  export const pimSchema = {
    productCategories,
    productMasters,
    // ...
  };
  export type PimSchema = typeof pimSchema;
  ```

- 테이블 이름, 컬럼 이름은 snake_case, TypeScript export const 이름은 camelCase 사용.

---

### 2. 타입 정의 규칙 (`types.ts`)

- **`InferSelectModel` / `InferInsertModel` 기반으로 Select/Insert 타입 자동 생성.**

  ```ts
  export type ProductCategory = InferSelectModel<typeof productCategories>;
  export type NewProductCategory = InferInsertModel<typeof productCategories>;
  ```

- **Update 타입은 공통 패턴 적용:**
  `Partial<Omit<NewEntity, 'id' | 'createdAt' | 'updatedAt'>>`
- **트랜잭션 타입은 DB Schema 전체를 제네릭으로:**

  ```ts
  export type DbTransaction = PostgresJsDatabase<PimSchema>;
  ```

- **DTO 계층은 서비스/컨트롤러 입장에서 바로 사용할 수 있도록 정의.**

  - 생성 DTO: Insert 타입 기반 + 추가 필드
  - 응답 DTO: Select 타입 + join된 값

---

### 3. Cursor Prompt / 사용 예시

- “이 테이블에 대한 Insert DTO를 만들어줘” → Cursor가 `InferInsertModel`을 가져와 자동으로 Insert 타입 생성.
- “결제 서버의 payment_intents 테이블 타입을 만들어줘” → Cursor가 `schema.ts`에서 `paymentIntents`를 찾아 `types.ts`에 `PaymentIntent`, `NewPaymentIntent`, `UpdatePaymentIntent`를 만들어줌.
- “Update 타입 만들어줘” → Cursor가 위 규칙대로 `Partial<Omit<NewEntity, …>>` 패턴을 적용.

---

### 4. Wallet 서버 적용 예시

`schema.ts` (결제 서버)

```ts
export const paymentIntents = pgTable('payment_intents', {
  id: varchar('id', { length: 30 }).primaryKey(),
  customerId: varchar('customer_id', { length: 64 }).notNull(),
  amount: bigint('amount', { mode: 'number' }).notNull(),
  status: varchar('status', { length: 20 }).default('PENDING'),
  createdAt: timestamp('created_at').defaultNow(),
  updatedAt: timestamp('updated_at').defaultNow(),
});
export const walletSchema = { paymentIntents /* ... */ };
export type WalletSchema = typeof walletSchema;
```

`types.ts` (결제 서버)

```ts
export type PaymentIntent = InferSelectModel<typeof paymentIntents>;
export type NewPaymentIntent = InferInsertModel<typeof paymentIntents>;
export type UpdatePaymentIntent = Partial<
  Omit<NewPaymentIntent, 'id' | 'createdAt' | 'updatedAt'>
>;

export type DbTransaction = PostgresJsDatabase<WalletSchema>;
```

이런 식으로 결제 서버도 CTO가 만든 PIM 서버와 동일한 규칙을 따라가면 됩니다.

---

이제 Cursor에 “위 규칙을 적용해 payment_attempts 테이블 타입 만들어줘”라고 지시하면 Cursor가 자동으로 이 규칙을 따라 타입을 생성해 줄 수 있습니다.
필요하면 제가 바로 Wallet 서버용 `schema.ts` + `types.ts` 스캐폴드를 만들어 드릴까요?

---
> Source: [LCNINE/almondyoung-server](https://github.com/LCNINE/almondyoung-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
