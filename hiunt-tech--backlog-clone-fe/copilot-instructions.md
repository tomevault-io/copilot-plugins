## static-method-confirm

> **File chính**: [components/modal/static-method-confirm.tsx](mdc:components/modal/static-method-confirm.tsx)

# Quy ước sử dụng Static Method Confirm

## 1. File và Location

**File chính**: [components/modal/static-method-confirm.tsx](mdc:components/modal/static-method-confirm.tsx)

Static method confirm được sử dụng để hiển thị modal xác nhận với API đơn giản, không cần quản lý state trong component.

## 2. Import và sử dụng

### Import:

```typescript
import { showConfirm } from '@/components/modal/static-method-confirm';
```

### Sử dụng cơ bản:

```typescript
showConfirm({
  title: 'Xác nhận xóa',
  content: 'Bạn có chắc chắn muốn xóa item này không?',
  onConfirm: () => {
    // Logic xử lý khi confirm
    console.log('Confirmed');
  },
  onCancel: () => {
    // Logic xử lý khi cancel (optional)
    console.log('Cancelled');
  },
});
```

## 3. API Interface

```typescript
interface ConfirmOptions {
  title: string; // Tiêu đề modal
  content: string; // Nội dung thông báo
  onConfirm: () => void; // Callback khi nhấn confirm
  onCancel?: () => void; // Callback khi nhấn cancel (optional)
  confirmText?: string; // Text cho button confirm (default: "Xác nhận")
  cancelText?: string; // Text cho button cancel (default: "Hủy bỏ")
  type?: 'warning' | 'danger' | 'info'; // Loại modal (default: 'warning')
}
```

## 4. Các trường hợp sử dụng

### Xóa item:

```typescript
const handleDelete = (itemId: string, itemName: string) => {
  showConfirm({
    title: t('admin.common.confirmDelete'),
    content: t('admin.common.confirmDeleteMessage', { name: itemName }),
    type: 'danger',
    onConfirm: () => {
      // Call delete API
      deleteItem(itemId);
    },
  });
};
```

### Bulk actions:

```typescript
const handleBulkUpdate = () => {
  showConfirm({
    title: t('admin.credits.bulkActions.collectionModal.title'),
    content: t('admin.credits.bulkActions.collectionModal.content', {
      count: selectedItems.length,
    }),
    onConfirm: () => {
      // Call bulk update API
      updateCreditsStatus(selectedItems, 'collection');
    },
  });
};
```

### Status change:

```typescript
const handleStatusChange = (itemId: string, newStatus: string) => {
  showConfirm({
    title: t('admin.common.confirmStatusChange'),
    content: t('admin.common.confirmStatusChangeMessage', {
      status: newStatus,
    }),
    onConfirm: () => {
      // Call status update API
      updateStatus(itemId, newStatus);
    },
  });
};
```

## 5. i18n Integration

Luôn sử dụng i18n cho title và content:

```typescript
// ✅ ĐÚNG
showConfirm({
  title: t('admin.machines.deleteModal.title'),
  content: t('admin.machines.deleteModal.content', { name: machineName }),
  onConfirm: () => deleteMachine(machineId),
});

// ❌ SAI - hardcode text
showConfirm({
  title: 'Xóa máy giặt',
  content: 'Bạn có chắc chắn muốn xóa máy giặt này?',
  onConfirm: () => deleteMachine(machineId),
});
```

## 6. Quy tắc đặt tên i18n key

### Cho modal confirm:

- `admin.[module].deleteModal.title`
- `admin.[module].deleteModal.content`
- `admin.[module].bulkActions.[action]Modal.title`
- `admin.[module].bulkActions.[action]Modal.content`

### Ví dụ:

```typescript
// File i18n
{
  admin: {
    credits: {
      deleteModal: {
        title: "Xác nhận xóa khoản vay",
        content: "Bạn có chắc chắn muốn xóa khoản vay {{name}} không?"
      },
      bulkActions: {
        collectionModal: {
          title: "Xác nhận chuyển thu nợ",
          content: "Bạn có chắc chắn muốn chuyển {{count}} khoản vay sang thu nợ không?"
        }
      }
    }
  }
}
```

## 7. Best Practices

### ✅ NÊN:

- Sử dụng cho các action quan trọng (delete, status change, bulk actions)
- Luôn dùng i18n cho text
- Truyền parameters vào i18n để dynamic content
- Sử dụng type phù hợp ('danger' cho delete, 'warning' cho update)
- Handle cả onConfirm và onCancel nếu cần

### ❌ KHÔNG NÊN:

- Hardcode text trong title/content
- Sử dụng cho action không quan trọng
- Quên handle loading state trong onConfirm
- Nest nhiều modal confirm

## 8. Ví dụ hoàn chỉnh

```typescript
// Component
import { showConfirm } from '@/components/modal/static-method-confirm';

const handleDeleteCredit = (creditId: string, creditCode: string) => {
  showConfirm({
    title: t('admin.credits.deleteModal.title'),
    content: t('admin.credits.deleteModal.content', { name: creditCode }),
    type: 'danger',
    confirmText: t('admin.common.delete'),
    cancelText: t('admin.common.cancel'),
    onConfirm: async () => {
      try {
        await deleteCreditApi(creditId);
        toastHelpers.success({ title: t('admin.credits.deleteSuccess') });
        refetch(); // Refresh data
      } catch (error) {
        toastHelpers.error({ title: t('admin.common.errorOccurred') });
      }
    },
  });
};
```

## 9. Lưu ý

- Static method không cần quản lý state trong component
- Modal sẽ tự động close sau khi confirm/cancel
- Có thể customize button text và icon theo type
- Support async operations trong onConfirm callback
- Tự động focus vào button confirm khi mở modal

---
> Source: [HiuNT-Tech/backlog-clone-fe](https://github.com/HiuNT-Tech/backlog-clone-fe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
