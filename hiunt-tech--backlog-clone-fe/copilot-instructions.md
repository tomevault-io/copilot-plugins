## enum-usage

> **File**: [config/enum.ts](mdc:config/enum.ts)

# Quy ước sử dụng Enum

## 1. Định nghĩa enum

**File**: [config/enum.ts](mdc:config/enum.ts)

### Quy tắc đặt tên:

- Tên enum: PascalCase (VD: `CreditStatus`)
- Giá trị enum: UPPER_SNAKE_CASE (VD: `OVERDUE`)
- Value: snake_case giống API (VD: `"overdue"`)

### Format enum:

```typescript
export enum EntityStatus {
  ACTIVE = "active",
  INACTIVE = "inactive",
  MAINTENANCE = "maintenance",
}
```

## 2. Các enum được định nghĩa

### Core Status Enums:

- `CreditStatus`: active, overdue, paid, defaulted, cancelled
- `PaymentStatus`: pending, paid, overdue, failed, partial
- `CollectionStatus`: new, in_progress, resolved, written_off
- `CustomerStatus`: active, inactive, blacklisted
- `UserRole`: admin, collector, supervisor, auditor
- `Language`: vi, en, ja

### Type/Category Enums:

- `CreditType`: personal, business, mortgage, installment
- `PaymentMethod`: cash, bank_transfer, digital_wallet, installment
- `CollectionMethod`: phone_call, sms, email, personal_visit, legal_action
- `RiskLevel`: low, medium, high, critical
- `DocumentType`: id_card, passport, contract, proof_of_income

## 3. Sử dụng enum trong code

### ✅ ĐÚNG:

```typescript
import { CreditStatus } from "@/config/enum";

// So sánh status
if (credit.status === CreditStatus.OVERDUE) {
  // logic
}

// Array filter
const statusFilter = [
  CreditStatus.ACTIVE,
  CreditStatus.OVERDUE,
];

// Switch case
switch (credit.status) {
  case CreditStatus.ACTIVE:
    return "admin.credits.statusOptions.active";
  case CreditStatus.OVERDUE:
    return "admin.credits.statusOptions.overdue";
  default:
    return "admin.credits.statusOptions.unknown";
}
```

### ❌ SAI:

```typescript
// Hardcode string - KHÔNG BAO GIỜ LÀM ĐIỀU NÀY
if (credit.status === "overdue") {
}
if (credit.status === "active") {
}

// Magic string trong array
const statusFilter = ["active", "overdue"];
```

## 4. Enum trong API interfaces

### Interface definition:

```typescript
export interface Credit {
  id: string;
  status: CreditStatus; // Sử dụng enum type
  type: CreditType;
  // ...
}
```

### API params:

```typescript
export interface CreditListParams {
  status?: CreditStatus[]; // Array of enum values
  // ...
}
```

## 5. Enum với i18n

### Mapping enum sang i18n key:

```typescript
const getStatusI18nKey = (status: CreditStatus): string => {
  switch (status) {
    case CreditStatus.ACTIVE:
      return "admin.credits.statusOptions.active";
    case CreditStatus.OVERDUE:
      return "admin.credits.statusOptions.overdue";
    default:
      return "admin.credits.statusOptions.unknown";
  }
};

// Sử dụng trong component
const statusText = t(getStatusI18nKey(credit.status));
```

## 6. Enum trong select options

```typescript
const statusOptions = [
  { value: "all", label: t("admin.credits.statusOptions.label") },
  {
    value: CreditStatus.ACTIVE,
    label: t("admin.credits.statusOptions.active"),
  },
  {
    value: CreditStatus.OVERDUE,
    label: t("admin.credits.statusOptions.overdue"),
  },
  // ...
];
```

## 7. Thêm enum mới

### Khi thêm value mới:

1. Thêm vào enum trong `config/enum.ts`
2. Cập nhật mapping functions (nếu có)
3. Thêm i18n key tương ứng vào `i18n/resources.ts`
4. Cập nhật TypeScript interfaces nếu cần

### Khi thêm enum mới:

1. Đặt tên theo quy ước PascalCase
2. Group theo chức năng (Status, Type, Category, etc.)
3. Sử dụng value snake_case giống API
4. Export từ `config/enum.ts`

## 8. Validation với enum

```typescript
// Kiểm tra enum value hợp lệ
const isValidStatus = (status: string): status is CreditStatus => {
  return Object.values(CreditStatus).includes(
    status as CreditStatus
  );
};

// Type guard
if (isValidStatus(apiStatus)) {
  // status có type là CreditStatus
  credit.status = apiStatus;
}
```

## 9. Best practices

- **LUÔN** sử dụng enum thay vì string literals
- **KHÔNG BAO GIỜ** hardcode status/type values
- Import enum ở đầu file khi sử dụng
- Sử dụng enum làm type cho TypeScript interfaces
- Mapping enum sang i18n key thay vì hardcode text
- Group enum theo chức năng trong file
- Value của enum phải match với API response format

---
> Source: [HiuNT-Tech/backlog-clone-fe](https://github.com/HiuNT-Tech/backlog-clone-fe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
