## i18n

> - Thư mục cấu hình: [i18n/index.ts](mdc:i18n/index.ts)

### Quy ước i18n cho dự án Credit Management

- Thư mục cấu hình: [i18n/index.ts](mdc:i18n/index.ts)
- Nguồn bản dịch chính: [i18n/resources.ts](mdc:i18n/resources.ts)
- File ngôn ngữ riêng:
  - Tiếng Việt: [i18n/locales/vi.ts](mdc:i18n/locales/vi.ts)
  - Tiếng Anh: [i18n/locales/en.ts](mdc:i18n/locales/en.ts)
  - Tiếng Nhật: [i18n/locales/ja.ts](mdc:i18n/locales/ja.ts)
- Provider: [components/i18n-provider.tsx](mdc:components/i18n-provider.tsx)

#### Sử dụng

- Import hook trong component client:

```ts
import { useTranslation } from "react-i18next";
const { t, i18n } = useTranslation();
```

- Lấy chuỗi dịch:

```tsx
<h1>{t("app.title")}</h1>
```

- Đổi ngôn ngữ runtime (vi | en | ja):

```ts
i18n.changeLanguage("en");
```

- Lưu ngôn ngữ: Provider tự lưu vào localStorage key `lng` và đồng bộ `document.documentElement.lang`.

#### Quy ước key

- Chung: `app.*`, `header.*`, `sidebar.*`
- Auth: `login.*`, `forgot.*`, `otp.*`
- Khu vực admin:
  - Từ vựng chung: `admin.common.*`
  - Tài khoản: `admin.accounts.*`
  - Khách hàng: `admin.*`
  - Khoản vay: `admin.credits.*`
  - Thu nợ: `admin.collections.*`
  - Báo cáo: `admin.reports.*`
  - Cài đặt: `admin.settings.*`

Ví dụ tham số hóa:

```tsx
{
  t("admin.pagination.rangeOf", { from: 1, to: 15, total: 124 });
}
```

#### Component hỗ trợ

- Chọn ngôn ngữ: [components/ui/language-selector.tsx](mdc:components/ui/language-selector.tsx) (tự gọi `i18n.changeLanguage` nếu không truyền `onChange`).

#### Quy tắc bắt buộc - NGHIÊM CẤM hardcode text

**🚫 NGHIÊM CẤM:**

- Hardcode text tiếng Việt, tiếng Anh, tiếng Nhật trong component/function
- Magic strings cho status, type, category (phải dùng enum)
- Text literals trong return statements
- Hardcode error messages, success messages
- Hardcode labels, placeholders, tooltips

**❌ CÁC VÍ DỤ SAI:**

```tsx
// SAI - hardcode text
<h1>Danh sách khoản vay</h1>
return "Đang thu nợ";
placeholder="Nhập từ khóa tìm kiếm"
console.log("Có lỗi xảy ra");

// SAI - hardcode status/enum values
if (status === "active") {}
case "overdue": return "Quá hạn";
```

**✅ ĐÚNG:**

```tsx
// ĐÚNG - dùng i18n
<h1>{t("admin.credits.title")}</h1>
return t("admin.credits.statusOptions.overdue");
placeholder={t("admin.credits.searchPlaceholder")}
console.log(t("admin.common.errorOccurred"));

// ĐÚNG - dùng enum + i18n
if (status === CreditStatus.ACTIVE) {}
case CreditStatus.OVERDUE: return t("admin.credits.statusOptions.overdue");
```

#### Xử lý text cần hiển thị

1. **Tất cả text** phải qua `t(key)` function
2. **Status/enum values** phải dùng enum từ [config/enum.ts](mdc:config/enum.ts)
3. **Thêm key mới** vào các file ngôn ngữ tương ứng trong [i18n/locales/](mdc:i18n/locales/)
4. **Helper functions** để map enum sang i18n key:

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
```

#### Cấu trúc file i18n

**File chính** [i18n/resources.ts](mdc:i18n/resources.ts):

```typescript
import vi from "./locales/vi";
import en from "./locales/en";
import ja from "./locales/ja";

const resources = {
  vi,
  en,
  ja,
} as const;

export default resources;
```

**File ngôn ngữ** [i18n/locales/{lang}.ts](mdc:i18n/locales/):

```typescript
const vi = {
  translation: {
    admin: {
      common: {
        loading: "Đang tải...",
        errorLoadingData: "Có lỗi xảy ra khi tải dữ liệu",
        retry: "Thử lại",
        minutes: "phút",
      },
      credits: {
        title: "Quản lý khoản vay",
        statusOptions: {
          unknown: "Không xác định",
        },
      },
    },
  },
} as const;

export default vi;
```

#### Thêm key mới

1. **Xác định ngôn ngữ**: Thêm key vào tất cả file ngôn ngữ (vi, en, ja)
2. **Cấu trúc key**: Theo quy ước `admin.[module].[section].[field]`
3. **Consistent naming**: Đảm bảo key giống nhau trong tất cả ngôn ngữ
4. **Export**: File tự động export, không cần thay đổi [i18n/resources.ts](mdc:i18n/resources.ts)

#### Lưu ý

- **Component server**: truyền text đã dịch từ client hoặc chuyển sang client component
- **Console logs**: dùng `t(key)` hoặc comment tiếng Anh cho debug
- **Type annotations**: có thể dùng tiếng Anh trong comments/types
- **Enum values**: phải match API format (snake_case)
- **File structure**: Mỗi ngôn ngữ có file riêng, import vào resources.ts

---
> Source: [HiuNT-Tech/backlog-clone-fe](https://github.com/HiuNT-Tech/backlog-clone-fe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
