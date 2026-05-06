## theme-usage

> Hệ thống Credit Management hỗ trợ tùy chỉnh màu sắc giao diện theo từng người dùng. Tất cả màu sắc phải sử dụng theme variables thay vì hardcode để đảm bảo tính nhất quán và khả năng tùy chỉnh.


# Quy ước sử dụng Theme Colors

## 1. Tổng quan

Hệ thống Credit Management hỗ trợ tùy chỉnh màu sắc giao diện theo từng người dùng. Tất cả màu sắc phải sử dụng theme variables thay vì hardcode để đảm bảo tính nhất quán và khả năng tùy chỉnh.

## 2. CSS Variables có sẵn

```css
/* Theme colors được set trong :root */
--theme-main: #15bec6; /* Màu chính */
--theme-hover: #1d4ed8; /* Màu khi hover */
--theme-text: #1f2937; /* Màu chữ chính */
--theme-disabled: #9ca3af; /* Màu khi disabled */
--theme-border: #e5e7eb; /* Màu viền */
--theme-background: #ffffff; /* Màu nền */
```

## 3. Sử dụng trong CSS/Tailwind

### ✅ ĐÚNG - Sử dụng CSS variables

```css
/* Trong CSS files */
.my-button {
  background-color: var(--theme-main);
  color: var(--theme-text);
  border: 1px solid var(--theme-border);
}

.my-button:hover {
  background-color: var(--theme-hover);
}

.my-button:disabled {
  background-color: var(--theme-disabled);
  color: var(--theme-disabled);
}
```

```tsx
// Trong Tailwind CSS với arbitrary values
<button
  className="bg-[var(--theme-main)] text-[var(--theme-neutral-11)]
             border border-theme-main
             hover:bg-[var(--theme-hover)]
             disabled:bg-theme-neutral-6 disabled:text-theme-neutral-7"
>
  Button
</button>

// HOẶC sử dụng theme classes đã định nghĩa
<button
  className="bg-theme-main text-theme-neutral-11
             border border-theme-main
             hover:bg-theme-hover
             disabled:bg-theme-neutral-6 disabled:text-theme-neutral-7"
>
  Button
</button>
```

### ❌ SAI - Hardcode màu sắc

```css
/* KHÔNG làm thế này */
.my-button {
  background-color: #2563eb; /* Hardcode màu */
  color: #1f2937; /* Hardcode màu */
}

.my-button:hover {
  background-color: #1d4ed8; /* Hardcode màu */
}
```

```tsx
// KHÔNG làm thế này
<button
  className="bg-blue-600 text-gray-800 border border-gray-200
             hover:bg-blue-700 disabled:bg-gray-400 disabled:text-gray-400"
>
  Button
</button>
```

## 4. Sử dụng trong React Components

### ✅ ĐÚNG - Sử dụng CSS variables trong inline styles

```tsx
const MyButton = () => {
  return (
    <button
      className="px-4 py-2 rounded border"
      style={{
        backgroundColor: 'var(--theme-main)',
        color: 'var(--theme-text)',
        borderColor: 'var(--theme-border)',
      }}
      onMouseEnter={e => {
        e.currentTarget.style.backgroundColor = 'var(--theme-hover)';
      }}
      onMouseLeave={e => {
        e.currentTarget.style.backgroundColor = 'var(--theme-main)';
      }}
    >
      Button
    </button>
  );
};
```

### ❌ SAI - Hardcode màu trong React components

```tsx
const MyButton = () => {
  return (
    <button
      className="bg-blue-600 text-gray-800 border border-gray-200 px-4 py-2 rounded"
      style={{
        backgroundColor: '#2563eb', // Hardcode!
        color: '#1f2937', // Hardcode!
      }}
    >
      Button
    </button>
  );
};
```

## 5. Sử dụng với Shadcn/UI Components

### Button Component

```tsx
import { Button } from '@/components/ui/button';

// ✅ ĐÚNG - Sử dụng theme classes
const ThemedButton = () => {
  return (
    <Button
      className="bg-theme-main text-theme-neutral-1
                 hover:bg-theme-hover
                 border-theme-main"
    >
      Themed Button
    </Button>
  );
};

// HOẶC sử dụng arbitrary values
const ThemedButtonAlt = () => {
  return (
    <Button
      className="bg-[var(--theme-main)] text-[var(--theme-neutral-1)]
                 hover:bg-[var(--theme-hover)]
                 border-[var(--theme-main)]"
    >
      Themed Button
    </Button>
  );
};
```

### Input Component

```tsx
import { Input } from '@/components/ui/input';

const ThemedInput = () => {
  return (
    <Input
      className="border-theme-neutral-4
                 focus:border-theme-main
                 text-theme-neutral-11
                 placeholder:text-theme-neutral-6"
      placeholder="Enter text..."
    />
  );
};
```

### Card Component

```tsx
import { Card, CardContent, CardHeader } from '@/components/ui/card';

const ThemedCard = () => {
  return (
    <Card className="border-theme-neutral-4 bg-theme-neutral-1">
      <CardHeader className="text-theme-neutral-11">Card Title</CardHeader>
      <CardContent>
        <p className="text-theme-neutral-9">Card content with themed colors</p>
      </CardContent>
    </Card>
  );
};
```

## 6. Sử dụng với Data Tables

### Table Headers

```tsx
const TableHeader = () => {
  return (
    <thead className="bg-[var(--theme-background)] border-b border-[var(--theme-border)]">
      <tr>
        <th className="text-left text-[var(--theme-text)] font-medium p-4">
          Column 1
        </th>
        <th className="text-left text-[var(--theme-text)] font-medium p-4">
          Column 2
        </th>
      </tr>
    </thead>
  );
};
```

### Table Rows

```tsx
const TableRow = ({ isSelected }: { isSelected?: boolean }) => {
  return (
    <tr
      className={`border-b border-[var(--theme-border)]
                  hover:bg-[var(--theme-background)]`}
      style={{
        backgroundColor: isSelected ? 'var(--theme-main)' : 'transparent',
        color: isSelected ? 'var(--theme-text)' : 'var(--theme-text)',
      }}
    >
      <td className="p-4">Data 1</td>
      <td className="p-4">Data 2</td>
    </tr>
  );
};
```

## 7. Sử dụng với Status Badges

```tsx
const StatusBadge = ({
  status,
}: {
  status: 'active' | 'inactive' | 'pending';
}) => {
  const getStatusColors = (status: string) => {
    switch (status) {
      case 'active':
        return {
          background: 'var(--theme-main)',
          color: 'var(--theme-text)',
        };
      case 'inactive':
        return {
          background: 'var(--theme-disabled)',
          color: 'var(--theme-text)',
        };
      case 'pending':
        return {
          background: 'var(--theme-border)',
          color: 'var(--theme-text)',
        };
      default:
        return {
          background: 'var(--theme-disabled)',
          color: 'var(--theme-text)',
        };
    }
  };

  const colors = getStatusColors(status);

  return (
    <span
      className="px-2 py-1 rounded-full text-xs font-medium"
      style={{
        backgroundColor: colors.background,
        color: colors.color,
      }}
    >
      {status.toUpperCase()}
    </span>
  );
};
```

## 8. Sử dụng với Forms

### Form Labels

```tsx
const FormLabel = ({
  children,
  required,
}: {
  children: React.ReactNode;
  required?: boolean;
}) => {
  return (
    <label className="block text-sm font-medium text-[var(--theme-text)] mb-2">
      {children}
      {required && <span className="text-red-500 ml-1">*</span>}
    </label>
  );
};
```

### Form Inputs

```tsx
const ThemedInput = ({ error }: { error?: boolean }) => {
  return (
    <input
      className={`w-full px-3 py-2 border rounded-md
                  text-[var(--theme-text)]
                  placeholder:text-[var(--theme-disabled)]
                  bg-[var(--theme-background)]`}
      style={{
        borderColor: error ? '#ef4444' : 'var(--theme-border)',
      }}
    />
  );
};
```

## 9. Sử dụng với Modals/Dialogs

```tsx
const ThemedModal = ({ children }: { children: React.ReactNode }) => {
  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center">
      {/* Overlay */}
      <div className="absolute inset-0 bg-black bg-opacity-50" />

      {/* Modal */}
      <div
        className="relative max-w-md w-full mx-4 rounded-lg shadow-lg"
        style={{
          backgroundColor: 'var(--theme-background)',
          border: '1px solid var(--theme-border)',
        }}
      >
        <div className="p-6">
          <div className="text-[var(--theme-text)]">{children}</div>
        </div>
      </div>
    </div>
  );
};
```

## 10. Lưu ý quan trọng

### ✅ NÊN:

- **Luôn sử dụng CSS variables** thay vì hardcode màu
- **Test với nhiều theme khác nhau** để đảm bảo tương thích
- **Sử dụng semantic naming** cho màu sắc (primary, hover, text, etc.)
- **Sử dụng theme classes** thay vì arbitrary values khi có thể
- **Kết hợp theme classes với Tailwind utilities** (spacing, layout, etc.)
- **Tạo component wrapper** cho các UI patterns thường dùng

### ❌ KHÔNG NÊN:

- **Hardcode hex colors** như `#2563eb`, `#1f2937`
- **Sử dụng Tailwind color classes** cố định như `bg-blue-600`
- **Giả định màu mặc định** trong logic component
- **Mix hardcode và theme colors** trong cùng một component
- **Bỏ qua theme trong custom components**

### 🔧 Best Practices:

1. **Tạo theme-aware components**: Viết components có thể adapt với theme
2. **Test với dark/light themes**: Đảm bảo contrast ratio tốt
3. **Use CSS custom properties**: Dễ maintain và consistent
4. **Document theme dependencies**: Ghi chú màu nào được sử dụng
5. **Avoid color assumptions**: Không giả định về màu sắc trong logic

### 🎨 Theme Variables Reference:

```css
/* Primary colors */
--theme-main:           /* Main brand color (#15bec6) */ --theme-hover:
  /* Hover state color (#15bdc670) */
  --theme-main-light: /* Light version (#e8f8f9) */
  --theme-filter-main: /* Filter for main color */ /* Main color scale */
  --theme-main-1: /* Lightest (#c4faff) */
  --theme-main-2: /* Lighter (#12e8f2) */ --theme-main-3: /* Light (#0cbdc5) */
  --theme-main-4: /* Medium (#07949a) */ --theme-main-5: /* Dark (#036c70) */
  --theme-main-6: /* Darker (#02464b) */ --theme-main-7: /* Darkest (#002628) */
  /* Neutral color scale */ --theme-neutral-1: /* White (#ffffff) */
  --theme-neutral-2: /* Light gray (#fafafa) */
  --theme-neutral-3: /* Lighter gray (#f5f5f5) */
  --theme-neutral-4: /* Light gray (#f0f0f0) */
  --theme-neutral-5: /* Gray (#d9d9d9) */
  --theme-neutral-6: /* Medium gray (#bfbfbf) */
  --theme-neutral-7: /* Dark gray (#8c8c8c) */
  --theme-neutral-8: /* Darker gray (#595959) */
  --theme-neutral-9: /* Very dark gray (#444444) */
  --theme-neutral-10: /* Almost black (#262626) */
  --theme-neutral-11: /* Black (#1f1f1f) */;
```

### 🎨 Tailwind Theme Classes:

```tsx
// Background colors
bg-theme-main, bg-theme-hover, bg-theme-main-light
bg-theme-main-1, bg-theme-main-2, ..., bg-theme-main-7

// Text colors
text-theme-main, text-theme-neutral-1, text-theme-neutral-11

// Border colors
border-theme-main, border-theme-neutral-4

// Examples
className="bg-theme-main text-theme-neutral-1 border-theme-main"
className="hover:bg-theme-hover focus:border-theme-main"
```

Bằng cách tuân thủ quy ước này, giao diện sẽ:

- **Nhất quán** với theme được chọn
- **Dễ tùy chỉnh** theo sở thích user
- **Maintainable** và dễ phát triển
- **Accessible** với contrast ratios phù hợp

---

### 🎯 **RULE SUMMARY:**

**✅ NÊN sử dụng:**

- `bg-theme-main`, `text-theme-neutral-11` (ưu tiên theme classes)
- Arbitrary values `bg-[var(--theme-main)]` khi cần thiết
- @theme directive trong globals.css cho Tailwind v4
- CSS custom properties cho complex styling

**❌ KHÔNG NÊN:**

- Hardcode hex colors như `#15bec6`
- Sử dụng Tailwind built-in colors cho theme
- Mix theme classes với hardcoded colors

**🔧 Available Theme Classes (Tailwind v4):**

- **Background**: `bg-theme-main`, `bg-theme-main-light`, `bg-theme-neutral-1`
- **Text**: `text-theme-main`, `text-theme-neutral-11`, `text-theme-neutral-9`
- **Border**: `border-theme-main`, `border-theme-neutral-4`
- **Hover**: `hover:bg-theme-hover`, `hover:text-theme-main`

**📝 Cấu hình:**

- Theme colors định nghĩa trong `app/globals.css` với `@theme` directive
- Không cần `tailwind.config.js` cho Tailwind v4

---
> Source: [HiuNT-Tech/backlog-clone-fe](https://github.com/HiuNT-Tech/backlog-clone-fe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
