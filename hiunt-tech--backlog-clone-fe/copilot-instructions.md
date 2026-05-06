## api-integration

> **File**: [config/interface.ts](mdc:config/interface.ts)

# Quy trình tích hợp API

## 1. Định nghĩa interface

**File**: [config/interface.ts](mdc:config/interface.ts)

- Tạo interface cho request parameters (VD: `CreditListParams`)
- Tạo interface cho response data (VD: `Credit`, `CreditListResponse`)
- Sử dụng nested interfaces cho object phức tạp (VD: `CustomerInfo`, `PaymentSchedule`)
- Interface response luôn có format:
  ```typescript
  export interface ApiResponse<T = any> {
    success?: boolean;
    data?: T;
    message?: string;
    code?: string;
  }
  ```

## 2. Tạo API function

**File**: [lib/api/[module-name].ts](mdc:lib/api/)

- Import axios helper từ [lib/axios-instance.ts](mdc:lib/axios-instance.ts)
- Sử dụng `sendGet`, `sendPost`, `sendPut`, `sendDelete`
- Export object chứa các API methods:
  ```typescript
  export const moduleApi = {
    getList: async (
      params: ParamsInterface
    ): Promise<ApiResponse<ResponseInterface>> => {
      return await sendGet('/endpoint', params);
    },
  };
  ```

## 3. Tích hợp với React Query

**Provider**: Đã được setup trong [components/query-provider.tsx](mdc:components/query-provider.tsx)

### Trong component:

```typescript
import { useQuery } from '@tanstack/react-query';
import { moduleApi } from '@/lib/api/module-name';

// Sử dụng trong component
const {
  data: responseData,
  isLoading,
  error,
  refetch,
} = useQuery({
  queryKey: ['module-name', params],
  queryFn: () => moduleApi.getList(params),
  staleTime: 30 * 1000, // 30 seconds
});
```

### Query key naming convention:

- Format: `['resource-name', ...params]`
- VD: `['credits', { status: 'active', page: 1 }]`

## 4. Xử lý states trong component

### Loading state:

```typescript
if (isLoading) {
  return <LoadingSpinner />;
}
```

### Error state:

```typescript
if (error) {
  return (
    <ErrorMessage
      message="Có lỗi xảy ra khi tải dữ liệu"
      onRetry={() => refetch()}
    />
  );
}
```

### Data handling:

```typescript
const items = responseData?.data?.items || [];
const total = responseData?.data?.total || 0;
```

## 5. Pagination & Filtering

### API params:

```typescript
const apiParams = {
  skip: (currentPage - 1) * itemsPerPage,
  limit: itemsPerPage,
  ...(searchTerm && { keyword: searchTerm }),
  ...(filter && { status: [filter] }),
};
```

### Pagination component:

```typescript
<Pagination
  currentPage={currentPage}
  totalPages={Math.ceil(total / itemsPerPage)}
  onPageChange={setCurrentPage}
  totalItems={total}
  itemsPerPage={itemsPerPage}
/>
```

## 6. Data transformation

Tạo helper functions để transform data từ API:

```typescript
const getDisplayStatus = (apiStatus: string) => {
  switch (apiStatus) {
    case 'active':
      return 'Đang hoạt động';
    case 'overdue':
      return 'Quá hạn';
    case 'paid':
      return 'Đã thanh toán';
    default:
      return 'Không xác định';
  }
};
```

## 7. i18n Integration

- Sử dụng `useTranslation()` hook
- Tất cả text đều phải dùng `t(key)` function
- Thêm key mới vào [i18n/resources.ts](mdc:i18n/resources.ts) nếu thiếu
- Quy ước key: `admin.[module].[section].[field]`

## 8. Checklist tích hợp API

- [ ] Interface được định nghĩa trong `config/interface.ts`
- [ ] API function trong `lib/api/[module].ts`
- [ ] Import `useQuery` từ `@tanstack/react-query`
- [ ] Xử lý loading và error states
- [ ] Data transformation nếu cần
- [ ] Pagination setup đúng
- [ ] i18n cho tất cả text
- [ ] Query key có format đúng
- [ ] Type safety cho TypeScript

---
> Source: [HiuNT-Tech/backlog-clone-fe](https://github.com/HiuNT-Tech/backlog-clone-fe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
