## promptready-nextjs

> >


## วิธีสร้าง Page ใหม่

อ่าน reference หลักก่อนเสมอ:

```
references/CREATE_PAGE_PATTERN.md
```

## ขั้นตอนสรุป

1. **Repository Interface** — กำหนด contract ใน `src/application/repositories/I[PageItem]Repository.ts`
2. **Mock Repository** — implement ใน `src/infrastructure/repositories/mock/Mock[PageItem]Repository.ts`
3. **Server Page** — สร้าง `app/[page-path]/page.tsx` (Server Component)
4. **Presenter** — business logic ใน `src/presentation/presenters/[page-name]/[PageName]Presenter.ts`
5. **Presenter Factories** — แยก Server/Client factory
6. **usePresenter Hook** — state management สำหรับ client
7. **View Component** — UI ใน `src/presentation/components/[page-name]/[PageName]View.tsx`

## Placeholder ที่ต้องแทนที่

| Placeholder | รูปแบบ | ตัวอย่าง |
|---|---|---|
| `[PageName]` | PascalCase | `Customers` |
| `[page-name]` | kebab-case | `customers` |
| `[PageItem]` | PascalCase | `Customer` |
| `[PAGE_ITEMS]` | SCREAMING_SNAKE | `CUSTOMERS` |
| `[PageThaiName]` | ภาษาไทย | `ลูกค้า` |

## Mock-First Workflow (แนะนำ)

ให้พัฒนา UI กับ Mock ก่อนเสมอ จนกว่า UI จะ stable
แล้วค่อย switch ไป Supabase Repository

ดูรายละเอียดทั้งหมดใน `references/CREATE_PAGE_PATTERN.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danya0365) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
