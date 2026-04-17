## hhgroup

> Bills vs Expenses 业务边界


# Bills vs Expenses — Do Not Double-Count

## Bills = Accounts Payable workflow

- **Bills** (`ap_bills`, `/bills`) are used for: vendor invoices, supplier bills, temporary labor payments, utility bills, permit/fee bills, equipment rental bills.
- Bills have a **payment lifecycle**: Draft → Pending → Partially Paid → Paid → Void.
- `paid_amount` = sum of `ap_bill_payments`; `balance_amount` = amount − paid_amount.
- **Bills track payments.** They are AP (accounts payable) and do **not** feed into project cost in the canonical profit model.

## Expenses = Direct cost records

- **Expenses** (`expenses` + `expense_lines`, `/financial/expenses`) are used for: small purchases, already-paid costs, company overhead, quick expense entries without a payment workflow.
- **Expenses track cost records** and **do** feed into project cost (expense_lines.amount per project).

## Rule: no double-counting

- **Project cost** in the canonical profit engine = approved labor cost + **expense_lines** + approved **subcontract_bills** only.
- **ap_bills** are never added to project cost. If a cost is recorded as a Bill (AP), do not also enter it as an expense_line for the same project.
- Subcontract work → use **subcontract_bills** (and optionally pay via AP if you track vendor payments there). Direct project spend → use **expense_lines**.

When adding new features, do not combine ap_bills amounts with expense_lines or subcontract_bills for the same cost.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hewenlian123) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
