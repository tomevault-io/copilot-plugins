## barpos

> 請扮演一位精通 **TypeScript + React 19** 開發的資深全端工程師，具備現代化 UI/UX 設計經驗，並熟悉現代前端架構模式。

# AGENTS.md - 調酒酒吧管理系統 🍸

## 0. 角色扮演

請扮演一位精通 **TypeScript + React 19** 開發的資深全端工程師，具備現代化 UI/UX 設計經驗，並熟悉現代前端架構模式。

> **開發工具**: 本專案使用 [opencode](https://opencode.ai) + GitHub Copilot 進行 AI 輔助開發  
> **技術理念**: TypeScript First, Modern React Patterns, Performance Optimized

## 1. 專案總覽

本專案是一個專為調酒酒吧設計的**現代化 POS 系統**，採用 **React 19 + TypeScript + Vite** 技術棧，配合 Supabase 雲端後端。系統全面採用 TypeScript，具備完整的類型安全和現代化架構。

**核心功能:**
- 🏠 **儀表板**: 即時數據監控、收益分析、關鍵指標展示
- 🪑 **座位管理**: 拖拽式佈局編輯器、即時狀態同步、視覺化管理
- 🍹 **點餐系統**: 專業調酒師UI、分類篩選、即時庫存管理  
- 📋 **訂單管理**: 完整生命週期、狀態追蹤、自動化流程
- ☁️ **雲端同步**: Supabase 即時資料庫、離線支援、資料備份
- 📱 **跨平台**: Web + Desktop (Electron) + Mobile (Capacitor)
- 📊 **分析報表**: 營收統計、銷售分析、趨勢預測

**技術特色:**
- ⚡ **效能優化**: 虛擬化列表、memo化組件、智能快取
- 🔒 **類型安全**: 全面 TypeScript、Zod 驗證、嚴格模式
- 🎨 **現代 UI**: Tailwind CSS、響應式設計、深色模式
- 🧪 **測試覆蓋**: Vitest + Testing Library、E2E 測試
- 🚀 **開發體驗**: 熱重載、DevTools、錯誤邊界

**相關文件參考:**
- `@README.md`: 專案總覽、快速開始指南和開發部署說明
- `@API_GUIDE.md`: **API 呼叫統一指南** - 狀態管理、服務層、錯誤處理完整規範
- `@SUPABASE_GUIDE.md`: Supabase 雲端資料庫設定詳細指南
- `@TESTING_GUIDE.md`: 測試策略與規範
- `@supabase-setup.sql`: 資料庫建立腳本（餐廳版）
- `@cocktail-bar-supabase-setup.sql`: 調酒酒吧資料庫建立腳本（推薦）

## 2. 現代化技術架構 🏗️

### 核心技術棧
- **前端框架**: React 19 + Vite 7 (ESM, HMR)
- **語言**: TypeScript 5.8+ (strict mode, path mapping)
- **狀態管理**: Context API → **遷移至 Zustand + Immer**
- **資料獲取**: **TanStack Query v5** (智能快取、背景同步)
- **表單處理**: **React Hook Form + Zod** (類型安全驗證)
- **UI 組件**: **Headless UI + Tailwind CSS** (無樣式組件 + 實用類別)
- **路由**: SPA (單頁應用) + 動態載入

### 後端 & 資料庫
- **雲端 BaaS**: Supabase (PostgreSQL + 即時訂閱 + 檔案儲存)
- **離線支援**: Service Worker + IndexedDB 快取
- **API 層**: 自定義 TypeScript API 抽象層

### 跨平台部署
- **Web**: Vite static build + CDN
- **Desktop**: Electron 37 (Windows/macOS/Linux)  
- **Mobile**: Capacitor 7 (iOS/Android native wrapper)

### 開發工具鏈
- **建置工具**: Vite (Rollup) + TypeScript compiler
- **程式碼品質**: ESLint + Prettier + TypeScript strict
- **測試**: Vitest + @testing-library/react + MSW
- **AI 輔助**: opencode + GitHub Copilot
- **版本控制**: Git + 語意化版本控制

## 3. 專案架構 📁

```
/
├── 📱 android/              # Android 原生專案 (Capacitor)
├── 🍎 ios/                  # iOS 原生專案 (Capacitor)  
├── 🖥️ electron/             # Electron 主進程檔案
│   └── main.cjs            # Electron 應用程式入口點
├── 📦 src/
│   ├── 🧩 components/       # React 組件 (全 TypeScript)
│   │   ├── Charts/         # 圖表組件庫
│   │   │   ├── BarChart.tsx
│   │   │   ├── LineChart.tsx
│   │   │   ├── PieChart.tsx
│   │   │   └── index.ts    # 統一導出
│   │   ├── ui/             # 基礎 UI 組件
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Modal.tsx
│   │   │   └── index.ts
│   │   ├── ErrorBoundary/  # 錯誤邊界系統
│   │   ├── Dashboard.tsx   # 儀表板組件
│   │   ├── Tables.tsx      # 座位管理
│   │   ├── Menu.tsx        # 菜單管理
│   │   ├── Settings.tsx    # 系統設定
│   │   └── ...             # 其他業務組件
│   ├── 🎣 hooks/           # 自定義 React Hooks
│   │   ├── useDebounce.ts  # 防抖 hook
│   │   ├── useLocalStorage.ts # 本地儲存
│   │   ├── useModal.ts     # 模態框管理
│   │   ├── useOnline.ts    # 網路狀態
│   │   └── ...
│   ├── 🗂️ contexts/         # React Context (Legacy → Zustand)
│   │   ├── AppContext.tsx  # 主要應用狀態
│   │   ├── ErrorContext.tsx
│   │   └── ...
│   ├── 🔧 services/        # API & 業務邏輯層
│   │   ├── supabaseService.ts    # Supabase API 封裝
│   │   ├── analyticsService.ts   # 分析服務
│   │   ├── storageService.ts     # 跨平台儲存

│   ├── 🏪 stores/          # Zustand 狀態管理 (新架構)
│   │   ├── appStore.ts     # 主要應用狀態
│   │   ├── userStore.ts    # 用戶狀態
│   │   └── ...
│   ├── 📝 schemas/         # Zod 驗證 schemas
│   │   ├── orderSchema.ts
│   │   ├── menuSchema.ts
│   │   └── ...
│   ├── 🔍 types/           # TypeScript 類型定義
│   │   ├── global.d.ts
│   │   ├── api.ts
│   │   └── index.ts
│   ├── 🧪 test/            # 測試檔案
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── setup.ts       # 測試環境設置
│   ├── 🛠️ utils/           # 工具函數
│   │   ├── chartHelpers.ts
│   │   ├── dataAnalysis.ts
│   │   └── performance.ts
│   ├── App.tsx            # 🚀 主應用組件
│   ├── main.tsx           # ⚡ React 入口點  
│   └── index.css          # 🎨 Tailwind 樣式
├── 📄 dist/               # 建置輸出目錄
├── 🗃️ public/             # 靜態資源
├── ⚙️ 設定檔案
│   ├── vite.config.js     # Vite 建置配置
│   ├── tsconfig.json      # TypeScript 配置
│   ├── tailwind.config.js # Tailwind 樣式配置
│   ├── vitest.config.js   # 測試配置
│   └── capacitor.config.ts# 移動端配置
└── 📚 文檔
    ├── README.md
    ├── SUPABASE_GUIDE.md
    └── AGENTS.md          # (本檔案)
```

### 📋 關鍵檔案說明

- **`App.tsx`**: 主應用組件，路由管理、Provider 層級設定
- **`main.tsx`**: React 18 入口點，啟動應用程式
- **`src/stores/`**: Zustand 狀態管理 (現代化替代 Context)
- **`src/hooks/`**: 自定義 React Hooks，可重用邏輯封裝
- **`src/services/`**: API 服務層，與外部服務交互的抽象
- **`src/schemas/`**: Zod 驗證 schemas，確保資料類型安全
- **`src/types/`**: TypeScript 類型定義，全專案類型契約
- **`src/components/ui/`**: 基礎 UI 組件庫，可重用設計系統

## 4. 快速開始 🚀

### 開發環境啟動
```bash
# 安裝依賴
npm install

# 啟動開發服務器 (熱重載)
npm run dev
# → http://localhost:5173

# TypeScript 類型檢查
npm run type-check

# ESLint 程式碼檢查
npm run lint
```

### 建置 & 部署
```bash
# 生產環境建置
npm run build

# 本地預覽生產版本
npm run preview

# 分析 bundle 大小
npm run build:analyze
```

## 5. 現代化開發準則 💎

### TypeScript First 原則
- **嚴格模式**: 啟用 `strict: true`，`noImplicitAny: true`
- **路徑對應**: 使用 `@/` 路徑別名，清晰的模組引用
- **類型安全**: 避免 `any`，優先使用聯合類型和泛型
- **介面優先**: 使用 `interface` 定義物件結構，`type` 定義聯合類型

### 組件開發模式
```typescript
// ✅ 現代化組件模式
interface ComponentProps {
  title: string;
  onSubmit: (data: FormData) => Promise<void>;
  isLoading?: boolean;
}

export const ModernComponent: React.FC<ComponentProps> = memo(({ 
  title, 
  onSubmit, 
  isLoading = false 
}) => {
  // Hooks 在最上方
  const [state, setState] = useState<ComponentState>();
  const { data, isError } = useQuery({...});
  const mutation = useMutation({...});
  
  // 事件處理函數使用 useCallback
  const handleSubmit = useCallback(async (formData: FormData) => {
    await onSubmit(formData);
  }, [onSubmit]);
  
  // 早期返回模式
  if (isError) return <ErrorFallback />;
  if (isLoading) return <LoadingSpinner />;
  
  return (
    <div className="modern-component">
      {/* JSX 內容 */}
    </div>
  );
});
```

### 狀態管理策略
- **全域狀態**: Zustand + Immer (替代 Context API)
- **服務端狀態**: TanStack Query (快取、背景同步、樂觀更新)
- **表單狀態**: React Hook Form + Zod 驗證
- **本地狀態**: useState, useReducer (組件內部)

### 效能優化準則
- **memo 化**: 使用 `React.memo` 包裝昂貴組件
- **虛擬化**: 大型列表使用 `@tanstack/react-virtual`
- **程式碼分割**: 動態 import() + React.lazy()
- **Bundle 分析**: 定期檢查 bundle 大小

### API 設計模式
```typescript
// ✅ 統一的 API 回應類型
interface ApiResponse<T> {
  data: T;
  error?: string;
  status: 'success' | 'error';
}

// ✅ 自定義 Hook 封裝 API 邏輯
export const useOrders = () => {
  return useQuery({
    queryKey: ['orders'],
    queryFn: async (): Promise<Order[]> => {
      const response = await api.orders.getAll();
      return response.data;
    },
    staleTime: 30_000, // 30秒
  });
};
```

## 6. 核心業務流程 🔄

### 現代化訂單管理流程

```typescript
// 1. 使用者互動 → UI 組件
const OrderCreationForm: React.FC = () => {
  // 2. 表單狀態管理 (React Hook Form + Zod)
  const form = useForm<OrderSchema>({
    resolver: zodResolver(orderSchema),
    defaultValues: getDefaultOrderValues(),
  });
  
  // 3. API 操作 (TanStack Query)
  const createOrderMutation = useMutation({
    mutationFn: createOrder,
    onSuccess: (newOrder) => {
      // 4. 狀態更新 (Zustand)
      useOrderStore.getState().addOrder(newOrder);
      // 5. 快取無效化
      queryClient.invalidateQueries(['orders']);
    },
  });
  
  const handleSubmit = async (data: OrderSchema) => {
    // 6. 資料驗證 → API 呼叫 → 狀態同步
    await createOrderMutation.mutateAsync(data);
  };
};
```

### 即時資料同步策略

```typescript
// Supabase 即時訂閱 + TanStack Query 整合
export const useRealtimeOrders = () => {
  const queryClient = useQueryClient();
  
  useEffect(() => {
    const subscription = supabase
      .channel('orders')
      .on('postgres_changes', {
        event: '*',
        schema: 'public',
        table: 'orders'
      }, (payload) => {
        // 即時更新本地快取
        queryClient.setQueryData(['orders'], (oldData: Order[]) => {
          return updateOrdersWithPayload(oldData, payload);
        });
      })
      .subscribe();
      
    return () => subscription.unsubscribe();
  }, [queryClient]);
};
```

### 錯誤處理 & 使用者體驗

```typescript
// 全域錯誤邊界 + 用戶友好的錯誤處理
export const GlobalErrorBoundary: React.FC = ({ children }) => {
  return (
    <ErrorBoundary
      fallback={<ErrorFallback />}
      onError={(error, errorInfo) => {
        // 錯誤記錄 & 分析
        analytics.captureException(error, errorInfo);
      }}
    >
      {children}
    </ErrorBoundary>
  );
};

// API 錯誤處理模式
const handleApiError = (error: ApiError) => {
  // 根據錯誤類型顯示適當的用戶訊息
  switch (error.status) {
    case 401: return showLoginPrompt();
    case 403: return showPermissionError();
    case 500: return showServerError();
    default: return showGenericError(error.message);
  }
};
```

## 7. 開發工作流程 ⚡

### Git 工作流程 & 版本管理
```bash
# 功能開發分支
git checkout -b feature/order-management-v2
git checkout -b fix/table-layout-bug
git checkout -b refactor/typescript-migration

# 提交訊息規範 (Conventional Commits)
git commit -m "feat(orders): add real-time order synchronization"
git commit -m "fix(ui): resolve table layout overflow issue"
git commit -m "refactor(types): migrate OrderContext to TypeScript"
git commit -m "perf(charts): implement virtualization for large datasets"
git commit -m "test(api): add comprehensive API error handling tests"

# 合併前檢查
npm run type-check  # TypeScript 編譯檢查
npm run lint        # ESLint 規則檢查  
npm run test        # 單元測試執行
npm run build       # 生產建置測試
```

### 程式碼審查檢查清單
- ✅ **類型安全**: 無 `any` 使用，完整型別標註
- ✅ **效能考量**: memo化昂貴組件，避免不必要重新渲染
- ✅ **錯誤處理**: 完整的 try-catch，用戶友好的錯誤訊息
- ✅ **可訪問性**: ARIA 標籤
- ✅ **測試覆蓋**: 關鍵業務邏輯單元測試
- ✅ **文檔**: JSDoc 註解，README 更新

## 8. 指令集 & 工具鏈 🛠️

### 核心開發指令
```bash
# 🚀 開發環境
npm run dev              # Vite 開發服務器 + HMR
npm run dev:host         # 外部訪問 (0.0.0.0:5173)

# 🔍 程式碼品質  
npm run type-check       # TypeScript 類型檢查
npm run lint             # ESLint 程式碼檢查
npm run lint:fix         # 自動修復 ESLint 問題

# 🧪 測試系統
npm run test             # Vitest 單元測試
npm run test:ui          # 測試 UI 介面
npm run test:watch       # 監視模式測試
npm run test:coverage    # 測試覆蓋率報告

# 📦 建置部署
npm run build            # 生產環境建置
npm run build:analyze    # Bundle 分析報告
npm run preview          # 本地預覽生產版本
```

### 跨平台開發
```bash
# 🖥️ Electron 桌面應用
npm run electron-dev     # 開發模式 (Vite + Electron)
npm run electron-build   # 建置 + 啟動 Electron
npm run dist             # 打包發佈版本
npm run dist-mac         # macOS 版本打包
npm run dist-win         # Windows 版本打包

# 📱 Mobile (Capacitor)
npm run mobile:setup     # 初始化移動端平台
npm run cap:sync         # 同步 Web → Native
npm run cap:run:ios      # iOS 模擬器執行
npm run cap:run:android  # Android 模擬器執行
npm run cap:open:ios     # 開啟 Xcode
npm run cap:open:android # 開啟 Android Studio
```

### 開發工具整合
```bash
# 🔧 開發輔助工具
npm run serve           # 本地靜態檔案服務器
npm run clean           # 清理 build artifacts
npm run deps:update     # 更新專案依賴
npm run deps:audit      # 安全漏洞掃描
```

## 9. 程式碼風格指南 📝

### TypeScript 撰寫準則

```typescript
// ✅ 介面定義 - 明確且可擴展
interface OrderItem {
  readonly id: string;
  name: string;
  price: number;
  quantity: number;
  category?: MenuCategory;
  customizations?: Record<string, unknown>;
}

// ✅ 泛型約束 - 類型安全
interface ApiResponse<T = unknown> {
  data: T;
  status: 'success' | 'error';
  message?: string;
}

// ✅ 聯合類型 - 精確建模
type OrderStatus = 
  | 'pending' 
  | 'confirmed' 
  | 'preparing' 
  | 'ready' 
  | 'delivered' 
  | 'cancelled';

// ✅ 工具類型 - 類型轉換
type CreateOrderRequest = Omit<Order, 'id' | 'createdAt' | 'updatedAt'>;
type PartialUpdate<T> = Partial<Pick<T, keyof T>>;
```

### 命名規範系統

| 類型 | 規範 | 範例 |
|------|------|------|
| 🧩 **Components** | `PascalCase` | `OrderDetailsModal`, `TableLayoutEditor` |
| 🎣 **Hooks** | `camelCase` + `use` prefix | `useOrderManagement`, `useRealtimeSync` |
| 📁 **Files** | `camelCase` / `PascalCase` | `apiClient.ts`, `OrderService.ts` |
| 🔧 **Functions** | `camelCase` | `calculateTotal`, `validateOrderData` |
| 📊 **Constants** | `UPPER_SNAKE_CASE` | `API_ENDPOINTS`, `ORDER_STATUS_MAP` |
| 🏪 **Types/Interfaces** | `PascalCase` | `Order`, `MenuItem`, `ApiResponse<T>` |

### 匯入順序規範
```typescript
// 1. React 相關
import React, { useState, useEffect, memo } from 'react';

// 2. 第三方庫
import { useMutation, useQuery } from '@tanstack/react-query';
import { z } from 'zod';

// 3. 內部模組 (@/ 路徑)
import { Button, Modal, Input } from '@/components/ui';
import { useOrderStore } from '@/stores/orderStore';
import { OrderService } from '@/services/OrderService';

// 4. 相對路徑
import './OrderModal.styles.css';

// 5. 類型導入 (最後，使用 type import)
import type { Order, OrderStatus } from '@/types';
```

### Tailwind CSS 組織
```typescript
// ✅ 組件內樣式 - 按功能分組
const buttonClasses = cn(
  // 基礎樣式
  'inline-flex items-center justify-center',
  'font-medium rounded-lg transition-all duration-200',
  
  // 狀態樣式
  'hover:scale-105 active:scale-95',
  'focus:outline-none focus:ring-2 focus:ring-offset-2',
  
  // 變體樣式
  variant === 'primary' && 'bg-blue-600 text-white hover:bg-blue-700',
  variant === 'secondary' && 'bg-gray-200 text-gray-900 hover:bg-gray-300',
  
  // 響應式
  'px-3 py-2 text-sm sm:px-4 sm:py-2 sm:text-base',
  
  // 條件樣式
  disabled && 'opacity-50 cursor-not-allowed hover:scale-100',
  className
);
```

## 10. 測試策略 🧪

### 測試金字塔結構
```typescript
// 🔬 單元測試 (Vitest + Testing Library)
describe('OrderService', () => {
  it('should calculate order total correctly', () => {
    const order = createMockOrder();
    const total = OrderService.calculateTotal(order);
    expect(total).toBe(150.50);
  });
  
  it('should handle empty orders gracefully', () => {
    const emptyOrder = { items: [] };
    const total = OrderService.calculateTotal(emptyOrder);
    expect(total).toBe(0);
  });
});

// 🧩 組件測試
describe('OrderModal', () => {
  it('renders order details correctly', async () => {
    render(<OrderModal order={mockOrder} onClose={jest.fn()} />);
    
    expect(screen.getByText('Order #123')).toBeInTheDocument();
    expect(screen.getByText('Table 5')).toBeInTheDocument();
    
    await waitFor(() => {
      expect(screen.getByText('NT$150.50')).toBeInTheDocument();
    });
  });
});

// 🔌 Integration 測試 (MSW Mock)
const server = setupServer(
  rest.get('/api/orders', (req, res, ctx) => {
    return res(ctx.json(mockOrders));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

### 測試覆蓋率目標
- **業務邏輯**: 90%+ (services, utils, hooks)
- **UI 組件**: 80%+ (critical user paths)
- **API 層**: 95%+ (error handling, data transformation)

### E2E 測試策略 (Playwright)
```typescript
// 🎭 端到端測試流程
test('complete order workflow', async ({ page }) => {
  // 1. 登入系統
  await page.goto('/');
  await page.fill('[data-testid=username]', 'admin');
  await page.fill('[data-testid=password]', 'password');
  await page.click('[data-testid=login-button]');
  
  // 2. 創建新訂單
  await page.click('[data-testid=new-order-button]');
  await page.selectOption('[data-testid=table-select]', '5');
  await page.fill('[data-testid=customer-count]', '2');
  
  // 3. 添加餐點
  await page.click('[data-testid=menu-item-cocktail-1]');
  await page.click('[data-testid=add-to-order]');
  
  // 4. 提交訂單
  await page.click('[data-testid=submit-order]');
  
  // 5. 驗證結果
  await expect(page.locator('[data-testid=success-message]')).toBeVisible();
});
```

## 11. 效能優化策略 ⚡

### 前端效能最佳化
```typescript
// 🎯 組件 memo 化
export const OrderItem = memo<OrderItemProps>(({ 
  order, 
  onUpdate 
}) => {
  // 只在 order 或 onUpdate 變化時重新渲染
  return <div>{/* 組件內容 */}</div>;
}, (prevProps, nextProps) => {
  // 自定義比較函數
  return prevProps.order.id === nextProps.order.id &&
         prevProps.order.status === nextProps.order.status;
});

// 🚀 虛擬化大型列表
const VirtualizedOrderHistory = () => {
  const parentRef = useRef<HTMLDivElement>(null);
  
  const virtualizer = useVirtualizer({
    count: orders.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 80, // 每個項目 80px 高度
  });
  
  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      {virtualizer.getVirtualItems().map((virtualItem) => (
        <OrderItemVirtualized 
          key={virtualItem.index}
          order={orders[virtualItem.index]}
          style={{
            height: `${virtualItem.size}px`,
            transform: `translateY(${virtualItem.start}px)`,
          }}
        />
      ))}
    </div>
  );
};

// ⚡ 程式碼分割
const LazyAnalytics = lazy(() => import('@/components/Analytics'));
const LazyTableEditor = lazy(() => import('@/components/TableLayoutEditor'));

// 📦 Bundle 分析優化
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          ui: ['@headlessui/react', '@heroicons/react'],
          data: ['@tanstack/react-query', 'zustand'],
        },
      },
    },
  },
});
```

### 資料載入優化
```typescript
// 🔄 智能快取策略
const useOrdersOptimized = () => {
  return useQuery({
    queryKey: ['orders'],
    queryFn: fetchOrders,
    staleTime: 30_000,        // 30秒內視為新鮮
    cacheTime: 300_000,       // 5分鐘快取
    refetchOnWindowFocus: false,
    refetchInterval: 60_000,  // 每分鐘背景更新
  });
};

// 🎣 預載入策略
const useOrderPrefetch = () => {
  const queryClient = useQueryClient();
  
  const prefetchOrder = useCallback((orderId: string) => {
    queryClient.prefetchQuery({
      queryKey: ['order', orderId],
      queryFn: () => fetchOrder(orderId),
      staleTime: 10_000,
    });
  }, [queryClient]);
  
  return { prefetchOrder };
};

// ⏱️ 防抖搜尋
const useSearchWithDebounce = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearch = useDebounce(searchTerm, 300);
  
  const { data: searchResults } = useQuery({
    queryKey: ['search', debouncedSearch],
    queryFn: () => searchOrders(debouncedSearch),
    enabled: debouncedSearch.length > 2,
  });
  
  return { searchTerm, setSearchTerm, searchResults };
};
```

## 12. API 開發規範 🚀

### 🔗 統一 API 指南
> **完整規範請參考**: `@API_GUIDE.md` - API 呼叫統一指南

本專案採用現代化三層架構：**UI Components → Zustand Stores → Service Layer**

#### 核心開發原則
```typescript
// ✅ 推薦：使用 Store Actions 進行 API 調用
const { orders, isLoading, error } = useOrders();
const { addOrder, updateOrder } = useOrderActions();

// ✅ 推薦：統一的錯誤處理模式  
const handleSubmit = async (data: OrderData) => {
  try {
    await addOrder(data);
    showSuccess('訂單創建成功！');
  } catch (error) {
    showError(error, '創建訂單');
  }
};

// ❌ 避免：直接在組件中調用 API
useEffect(() => {
  fetch('/api/orders'); // ❌ 應該透過 store action
}, []);
```

#### API 回應標準格式
```typescript
interface ApiResponse<T = unknown> {
  success: boolean;
  data?: T;
  error?: string;
  message?: string;
}
```

#### 開發檢查清單
- [ ] 定義清晰的 TypeScript 介面
- [ ] 實作統一的錯誤處理
- [ ] 添加適當的載入狀態
- [ ] 實作樂觀更新（如適用）
- [ ] 撰寫 API 單元測試
- [ ] 更新相關文檔

---

## 13. 開發規範 & 最佳實踐 📋

### 🔄 Git 協作規範
```bash
# ❌ 禁止操作
git commit -m "fix stuff"     # 不使用模糊提交訊息
git add .                     # 不盲目添加所有檔案

# ✅ 建議操作
git checkout -b feature/order-realtime-sync
git add src/components/OrderSync.tsx src/hooks/useRealtimeOrders.ts
git commit -m "feat(orders): implement real-time order synchronization

- Add WebSocket connection for live order updates
- Integrate with Supabase realtime subscriptions  
- Update order status automatically across all clients
- Add error handling for connection failures

Closes #123"

# 🔍 提交前檢查清單
npm run type-check && npm run lint && npm run test && npm run build
```

### 📁 專案組織原則
```typescript
// ✅ 單一職責原則
// 每個檔案/函數只負責一個明確的功能

// ✅ 明確的依賴關係 
// services → hooks → components
// types → schemas → validation

// ✅ 循環依賴檢測
// 避免 A → B → A 的依賴循環

// ✅ 深度限制
// 避免超過 3 層的巢狀迴圈或條件判斷
const processOrders = (orders: Order[]) => {
  return orders
    .filter(order => order.status === 'pending')    // 第1層
    .map(order => ({                                // 第2層
      ...order,
      items: order.items.map(item => ({            // 第3層 (建議重構)
        ...item,
        total: item.price * item.quantity
      }))
    }));
};

// ✅ 更好的重構方式
const calculateItemTotal = (item: OrderItem) => ({
  ...item,
  total: item.price * item.quantity
});

const processOrders = (orders: Order[]) => {
  return orders
    .filter(order => order.status === 'pending')
    .map(order => ({
      ...order,
      items: order.items.map(calculateItemTotal)
    }));
};
```

### 🎨 UI/UX 開發準則
```typescript
// ✅ 響應式設計原則
const ResponsiveComponent = () => (
  <div className="
    // Mobile first
    w-full p-4 text-sm
    // Tablet 
    sm:w-1/2 sm:p-6 sm:text-base
    // Desktop
    lg:w-1/3 lg:p-8 lg:text-lg
    // Large screens
    xl:w-1/4 xl:p-10
  ">
    Content
  </div>
);

// ✅ 可訪問性 (a11y) 要求
<button
  className="btn-primary"
  aria-label="創建新訂單"           // 螢幕閱讀器支援
  aria-describedby="order-help"    // 輔助說明
  data-testid="create-order-btn"   // 測試識別
  onClick={handleCreateOrder}
>
  新增訂單
</button>

// ✅ 載入狀態處理
const OrderComponent = () => {
  const { data, isLoading, error } = useOrders();
  
  if (isLoading) return <OrderSkeleton />;        // 骨架載入
  if (error) return <ErrorFallback error={error} />; // 錯誤處理
  if (!data?.length) return <EmptyState />;       // 空狀態
  
  return <OrderList orders={data} />;
};
```

### 🔒 安全性考量
```typescript
// ✅ 輸入驗證與清理
const orderSchema = z.object({
  tableNumber: z.number().int().min(1).max(100),
  customerCount: z.number().int().min(1).max(20),
  items: z.array(orderItemSchema).min(1),
  notes: z.string().max(500).optional(),
});

// ✅ 敏感資料處理
const sanitizeOrderForClient = (order: Order): PublicOrder => {
  const { internalNotes, staffComments, ...publicOrder } = order;
  return publicOrder;
};

// ✅ API 金鑰管理
// 環境變數使用，避免硬編碼
const supabaseClient = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
);
```

## 13. 專案路線圖 & 發展方向 🗺️

### 📅 短期目標 (當前版本 v3.0)
- ✅ **TypeScript 完全遷移**: 所有 `.js/.jsx` → `.ts/.tsx`
- ✅ **現代化狀態管理**: Context API → Zustand + TanStack Query
- ✅ **效能優化**: 虛擬化列表、memo化組件、程式碼分割
- 🔄 **測試覆蓋**: 核心業務邏輯單元測試 80%+
- 🔄 **API 層重構**: 統一錯誤處理、類型安全、快取策略

### 🚀 中期規劃 (v3.1 - v3.5)
- **PWA 功能**: 離線支援、推播通知、背景同步
- **進階分析**: 機器學習預測、趨勢分析、自動化建議
- **會員系統**

### 🌟 長期願景 (v4.0+)
- **AI 智能助手**: 自動化庫存管理、智能推薦系統
- **IoT 整合**: 智慧設備連接、環境感測、自動化控制
- **區塊鏈整合**: 供應鏈追蹤、數位貨幣支付
- **AR/VR 體驗**: 虛擬菜單展示、沉浸式管理介面

### 🛠️ 技術債務管理
```typescript
// TODO: 優先處理項目
interface TechnicalDebt {
  priority: 'high' | 'medium' | 'low';
  estimate: string; // 工時估算
  impact: string;   // 影響範圍
}

const technicalDebts: TechnicalDebt[] = [
  {
    priority: 'high',
    estimate: '2-3 days', 
    impact: 'Legacy Context API migration to Zustand'
  },
  {
    priority: 'medium',
    estimate: '1-2 days',
    impact: 'Consolidate duplicate utility functions'
  },
  {
    priority: 'low',
    estimate: '1 day',
    impact: 'Update outdated dependencies'
  }
];
```

---

## 14. 開發者指引 👨‍💻

### 🎯 角色扮演準則
> 當執行專案任務時，請扮演一位 **TypeScript + React 19 專家**，具備以下特質：
> - 🔍 **類型安全優先**: 重視 TypeScript 嚴格模式，避免 `any` 使用
> - ⚡ **效能導向**: 關注組件優化、memo化、虛擬化等效能技術  
> - 🧪 **測試驅動**: 重視程式碼品質，推崇測試覆蓋和文檔
> - 🎨 **用戶體驗**: 關注 UI/UX 設計，響應式布局，可訪問性
> - 🔄 **現代化架構**: 熟悉最新 React 模式、狀態管理、建置工具

### 📋 任務執行流程
1. **📖 需求分析**: 仔細理解用戶需求，識別技術關鍵點
2. **🏗️ 架構設計**: 制定實現方案，考慮擴展性和維護性
3. **👨‍💻 編碼實現**: 遵循專案規範，撰寫高品質 TypeScript 程式碼
4. **🧪 測試驗證**: 撰寫單元測試，確保功能正確性
5. **📝 文檔更新**: 更新相關文檔，包括 README、API 文檔等
6. **🔍 程式碼審查**: 自我檢查程式碼品質，確保符合專案標準

### ⚠️ 重要提醒
- **🚫 避免破壞性變更**: 保持向後相容性，漸進式重構
- **📁 檔案參考**: 遇到 `@filename` 時使用 Read 工具載入內容
- **🔄 懶惰載入**: 按需載入參考資料，避免預先載入所有檔案
- **💬 回應方式**: 優先提供計畫和建議，除非用戶明確要求立即實作

### 🚨 常見啟動問題與解決方案
在專案啟動或載入資料階段，可能會遇到應用程式卡住或行為異常的問題。以下是常見原因及排查方法：

1.  **應用程式卡在載入畫面 (無限載入)**
    *   **原因一：服務層初始化死鎖 (Service Layer Initialization Deadlock)**：`storageService.ts` 等服務層模組的初始化邏輯可能存在循環依賴或非同步阻塞，導致 `Promise` 無法解析。
        *   **排查與解決**：檢查 `StorageService.getInstance()` 等單例模式的初始化邏輯，確保其非阻塞且不會形成循環依賴。將非同步初始化移至背景，不阻塞 `getInstance` 的同步回傳。
    *   **原因二：模組依賴混亂 (Module Dependency Confusion)**：服務層不應依賴 UI 層或 Hook 層的模組（例如 `storageService.ts` 導入 `useDebounce.ts`）。這種反向依賴可能導致模組加載順序問題，尤其在熱重載環境下。
        *   **排查與解決**：將通用工具函數（如 `debounce`）從 `hooks` 目錄移至 `utils` 目錄，並確保服務層只導入底層工具或服務，避免跨層次的反向依賴。

2.  **控制台出現 `Uncaught InternalError: too much recursion`**
    *   **原因：日誌攔截器無限遞迴 (Logger Interceptor Infinite Recursion)**：`consoleInterceptorService.ts` 等日誌攔截服務在攔截 `console` 方法後，其內部又呼叫了被攔截後的 `console` 方法，形成無限循環。
        *   **排查與解決**：確保日誌攔截器在內部輸出日誌時，使用備份的原始 `console` 方法（例如 `this.originalConsole.log`），而非被替換後的 `console` 方法。

3.  **應用程式行為異常或資料顯示錯誤 (清除 `localStorage` 後正常)**
    *   **原因：本地儲存資料損壞 (Corrupted Local Storage Data)**：`localStorage` 中儲存的資料可能已損壞、格式不正確或與應用程式當前版本不兼容，導致解析失敗或資料結構不符預期。
        *   **排查與解決**：在瀏覽器開發者工具的控制台中執行 `localStorage.clear()`，或運行專案根目錄下的 `clear_storage.ts` 腳本。如果清除後應用程式正常，則需考慮在應用程式中加入更健壯的資料遷移、版本控制或錯誤處理機制，以應對未來資料格式變更。

---

## 🌟 核心價值
- **類型安全** > 運行時錯誤預防
- **開發體驗** > 提升生產力和程式碼品質  
- **效能優化** > 用戶體驗至上
- **測試驅動** > 長期可維護性
- **現代化架構** > 技術領先優勢

---

## ⚠️ 重要開發規則

### 🔥 API 呼叫統一規範
> **在創建或修改功能時，請先閱讀 `@API_GUIDE.md` 後尋找所需功能呼叫，以利於統一專案架構以及可維護性。**

這個規則確保：
1. **統一架構模式**: 所有功能都遵循 UI → Zustand → Service 三層架構
2. **類型安全**: 使用標準的 `ApiResponse<T>` 格式進行 API 回應
3. **錯誤處理一致性**: 採用分層錯誤處理策略
4. **開發效率**: 遵循明確的檢查清單和最佳實踐

---

*📝 最後更新: 2025年 | 🛠️ 開發工具: [opencode](https://opencode.ai) + GitHub Copilot | 💻 技術棧: React 19 + TypeScript + Vite*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/latteine1217)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/latteine1217)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
