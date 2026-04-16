## forwaiq

> - React 18 + TypeScript 5

# ForwaIQ 專案開發規範

## 技術棧
- React 18 + TypeScript 5
- Vite 6 (使用 SWC)
- Tailwind CSS + Radix UI
- Supabase (後端 API)
- React Hook Form (表單管理)

---

## 一、TypeScript 規範

### 基本原則
- 嚴格模式啟用：使用 `strict: true`
- 所有函數參數和返回值必須明確定義型別
- 使用 `interface` 定義物件結構，使用 `type` 定義聯合型別或複雜型別
- 避免使用 `any`，必要時使用 `unknown` 並進行型別守衛
- 使用 `Record<string, T>` 而非索引簽名 `{ [key: string]: T }`

### 型別定義規範
```typescript
// ✅ Props 介面命名：ComponentNameProps
interface QuoteListProps {
  quotes: Quote[];
  onEdit: (quote: Quote) => void;
  onDelete: (id: string) => void;
}

// ✅ 使用 interface 定義物件
interface Quote {
  id: string;
  vendorName: string;
  price: number;
}

// ✅ 使用 type 定義聯合型別
type VendorType = 'shipping' | 'trucking' | 'customs';

// ✅ 使用 Record 而非索引簽名
type CustomFieldValues = Record<string, any>;

// ❌ 避免使用 any
const processData = (data: any) => { };

// ✅ 使用具體型別
const processData = (data: Quote[]) => { };
```

---

## 二、React 組件規範

### 組件結構
- 使用函數式組件和 Hooks
- 組件檔案使用 PascalCase：`QuoteList.tsx`
- 一個檔案一個主要組件
- 使用具名導出 `export function ComponentName()` 而非預設導出（除了 App.tsx）
- Props 解構應在函數參數中完成

### 組件內部結構順序
```typescript
export function MyComponent({ prop1, prop2 }: MyComponentProps) {
  // 1. Hooks (useState, useRef, useContext)
  const [state, setState] = useState<string>('');
  const ref = useRef<HTMLDivElement>(null);
  
  // 2. useEffect
  useEffect(() => {
    loadData();
  }, []);
  
  // 3. 數據載入函數
  const loadData = async () => {
    try {
      // ...
    } catch (error) {
      console.error('載入數據失敗:', error);
    }
  };
  
  // 4. 事件處理函數
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // ...
  };
  
  const handleDelete = (id: string) => {
    // ...
  };
  
  // 5. 工具/輔助函數
  const formatData = (data: any) => {
    // ...
  };
  
  // 6. 渲染輔助函數
  const renderItem = (item: Item) => {
    // ...
  };
  
  // 7. 條件提早返回
  if (loading) {
    return <div>載入中...</div>;
  }
  
  if (error) {
    return <div>發生錯誤</div>;
  }
  
  // 8. 主要渲染
  return (
    <div>
      {/* JSX 內容 */}
    </div>
  );
}
```

### 組件範例
```typescript
// ✅ 好的組件結構
import { useState, useEffect } from 'react';
import type { Quote } from '../App';
import { Button } from './ui/button';

interface QuoteListProps {
  quotes: Quote[];
  onEdit: (quote: Quote) => void;
  onDelete: (id: string) => void;
}

export function QuoteList({ quotes, onEdit, onDelete }: QuoteListProps) {
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    loadData();
  }, []);

  const loadData = async () => {
    try {
      setLoading(true);
      // API call
    } catch (error) {
      console.error('載入資料時發生錯誤:', error);
    } finally {
      setLoading(false);
    }
  };

  const handleDelete = (id: string) => {
    if (confirm('確定要刪除嗎？')) {
      onDelete(id);
    }
  };

  if (loading) {
    return <div>載入中...</div>;
  }

  return (
    <div className="space-y-4">
      {quotes.map((quote) => (
        <div key={quote.id}>
          {/* Content */}
        </div>
      ))}
    </div>
  );
}
```

---

## 三、Hooks 使用規範

### 基本規則
- 自定義 Hook 必須以 `use` 開頭
- `useEffect` 依賴陣列必須完整且正確
- 避免在 `useEffect` 中直接定義 async 函數，應在外部定義後調用
- 使用 `useState` 時提供明確的初始值型別
- 複雜狀態邏輯考慮使用 `useReducer`

### 範例
```typescript
// ✅ useState 提供型別
const [quotes, setQuotes] = useState<Quote[]>([]);
const [loading, setLoading] = useState<boolean>(false);
const [formData, setFormData] = useState<FormData>({
  name: '',
  email: '',
});

// ✅ useEffect 正確使用
useEffect(() => {
  loadData();
}, []); // 依賴陣列完整

const loadData = async () => {
  // async 函數在外部定義
};

// ✅ 自定義 Hook
function useQuotes() {
  const [quotes, setQuotes] = useState<Quote[]>([]);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    // ...
  }, []);
  
  return { quotes, loading };
}

// ❌ 不要在 useEffect 中直接定義 async
useEffect(async () => {
  // 錯誤！
  const data = await fetchData();
}, []);
```

---

## 四、函數撰寫與命名規範

### 4.1 命名規則

#### 基本規則
- **camelCase**：所有函數使用駝峰式命名
- **動詞開頭**：函數名應以動詞開頭，清楚描述其行為
- **具體明確**：避免模糊命名

#### 常見動詞前綴

**事件處理函數**
```typescript
// 組件內部定義：使用 handle*
const handleSubmit = (e: React.FormEvent) => { };
const handleDelete = (id: string) => { };
const handleClickOutside = (event: MouseEvent) => { };

// 作為 props 傳遞：使用 on*
interface Props {
  onSubmit: (data: any) => void;
  onDelete: (id: string) => void;
  onOpenChange: (open: boolean) => void;
}
```

**數據操作函數**
```typescript
// load* - 從 API 或存儲載入數據
const loadQuotes = async () => { };
const loadCustomFields = async (vendorType: string) => { };

// save* - 保存數據
const saveVendor = async (vendor: Vendor) => { };

// update* - 更新數據
const updateContact = (id: string, field: string, value: any) => { };

// delete* / remove* - 刪除數據
const removeContact = (id: string) => { };

// create* / add* - 創建/新增數據
const addContact = () => { };
```

**狀態判斷函數（返回布林值）**
```typescript
// is* - 判斷狀態
const isExpired = (validUntil: string): boolean => {
  return new Date(validUntil) < new Date();
};

// has* - 判斷是否擁有
const hasValidContacts = (contacts: VendorContact[]): boolean => {
  return contacts.some(c => c.name?.trim());
};

// should* - 判斷是否應該
const shouldShowCustomField = (field: CustomField): boolean => {
  return !field.isRequired && activeOptionalFields.has(field.id);
};

// can* - 判斷是否能夠
const canSubmitForm = (): boolean => {
  return formData.name && formData.email;
};
```

**UI 相關函數**
```typescript
// show* / hide* - 顯示/隱藏
const showCustomCarrier = () => { };
const hideModal = () => { };

// toggle* - 切換狀態
const toggleDropdown = () => { };

// open* / close* - 打開/關閉
const openDialog = () => { };
const closeDialog = () => { };

// render* - 渲染內容
const renderCustomField = (field: CustomField) => { };
```

**工具函數**
```typescript
// get* - 獲取計算值或轉換數據
const getTypeIcon = (type: string) => { };
const getTypeLabel = (type: string) => { };
const getDaysUntilExpiry = (validUntil: string): number => { };

// calculate* - 計算
const calculateAveragePrice = (quotes: Quote[]): number => { };

// format* - 格式化
const formatPrice = (price: number): string => { };
const formatDate = (date: string): string => { };

// validate* - 驗證
const validateEmail = (email: string): boolean => { };

// setup* - 設置/初始化
const setupFormValidation = (form: HTMLFormElement) => { };

// reset* / clear* - 重置/清空
const resetForm = () => { };
const clearFilters = () => { };
```

### 4.2 函數聲明方式

```typescript
// ✅ 組件內函數：使用箭頭函數
export function MyComponent() {
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
  };

  const loadData = async () => {
    // ...
  };

  return <div>...</div>;
}

// ✅ 工具函數：使用 function 聲明或箭頭函數
export function setupFormValidation(formElement: HTMLFormElement | null) {
  if (!formElement) return;
  // ...
}

export const formatCurrency = (amount: number, currency: string): string => {
  return `${currency} $${amount.toLocaleString()}`;
};

// ✅ 組件：使用 function 聲明
export function QuoteList({ quotes, onEdit, onDelete }: QuoteListProps) {
  // ...
}

// ❌ 避免：預設導出（除了 App.tsx）
export default function QuoteList() { }
```

### 4.3 參數設計

```typescript
// ✅ 0-2 個參數：直接傳遞
const updateContact = (id: string, field: keyof VendorContact, value: any) => { };

// ✅ 3+ 個參數：使用物件參數
interface CreateQuoteParams {
  vendorName: string;
  vendorType: string;
  price: number;
  currency: string;
  validUntil: string;
  customFields?: Record<string, any>;
}

const createQuote = (params: CreateQuoteParams) => { };

// ✅ 可選參數放最後
const formatDate = (date: string, format?: string): string => { };

// ✅ 使用預設值
const loadData = async (page: number = 1, limit: number = 10) => { };

// ✅ 物件參數使用預設值
interface LoadOptions {
  page?: number;
  limit?: number;
  sortBy?: string;
}

const loadData = async ({ 
  page = 1, 
  limit = 10, 
  sortBy = 'createdAt' 
}: LoadOptions = {}) => { };
```

### 4.4 返回值

```typescript
// ✅ 明確標註返回型別
const getTypeLabel = (type: string): string => {
  switch (type) {
    case 'shipping': return '海運';
    case 'trucking': return '拖車';
    case 'customs': return '報關';
    default: return type;
  }
};

// ✅ 返回 JSX
const getTypeIcon = (type: string): JSX.Element | null => {
  switch (type) {
    case 'shipping': return <Ship className="w-4 h-4" />;
    case 'trucking': return <Truck className="w-4 h-4" />;
    default: return null;
  }
};

// ✅ Async 函數標註 Promise
const loadQuotes = async (): Promise<Quote[]> => {
  const response = await fetch(API_URL);
  return await response.json();
};

// ✅ 提早返回（Early Return）- 減少嵌套
const processQuote = (quote: Quote | null): string => {
  if (!quote) return '無報價';
  if (isExpired(quote.validUntil)) return '已過期';
  if (quote.price === 0) return '價格待定';
  
  return `${quote.currency} $${quote.price}`;
};
```

### 4.5 函數長度與複雜度

```typescript
// ✅ 單一職責原則 - 每個函數只做一件事
const validateEmail = (email: string): boolean => {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
};

const validatePhone = (phone: string): boolean => {
  return /^[\d-+()]+$/.test(phone);
};

const validateContact = (contact: VendorContact): boolean => {
  if (contact.email && !validateEmail(contact.email)) return false;
  if (contact.phone && !validatePhone(contact.phone)) return false;
  return true;
};

// ✅ 提取重複邏輯
const filterActiveFields = (fields: CustomField[]) => {
  return fields.filter(f => !f.isRequired && !activeOptionalFields.has(f.id));
};
```

### 4.6 Async/Await 函數

```typescript
// ✅ 完整的錯誤處理
const loadCustomFields = async (vendorType: string): Promise<void> => {
  try {
    setLoading(true);
    
    const response = await fetch(
      `${API_URL}/custom-fields/vendor/${vendorType}`,
      {
        headers: {
          'Authorization': `Bearer ${token}`,
        },
      }
    );

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const fields = await response.json();
    setCustomFields(fields);
  } catch (error) {
    console.error('載入自定義欄位時發生錯誤:', error);
    toast.error('無法載入自定義欄位');
  } finally {
    setLoading(false);
  }
};

// ✅ 使用 Promise.all 並行處理
const loadData = async () => {
  try {
    const [quotesRes, vendorsRes] = await Promise.all([
      fetch(`${API_URL}/quotes`),
      fetch(`${API_URL}/vendors`),
    ]);

    const quotesData = await quotesRes.json();
    const vendorsData = await vendorsRes.json();

    setQuotes(quotesData);
    setVendors(vendorsData);
  } catch (error) {
    console.error('Error loading data:', error);
  }
};
```

### 4.7 特殊場景函數

**表單處理**
```typescript
const handleSubmit = (e: React.FormEvent) => {
  e.preventDefault();
  
  if (!validateForm()) {
    toast.error('請檢查表單內容');
    return;
  }
  
  const submitData = {
    ...formData,
    customFields: customFieldValues,
  };
  
  if (editingQuote) {
    onSubmit({ ...editingQuote, ...submitData });
  } else {
    onSubmit(submitData);
  }
};

const resetForm = () => {
  setFormData(initialFormState);
  setCustomFieldValues({});
};
```

**列表操作**
```typescript
const addContact = () => {
  setContacts([
    ...contacts,
    { id: crypto.randomUUID(), name: '', isPrimary: false }
  ]);
};

const removeContact = (id: string) => {
  if (contacts.length > 1) {
    setContacts(contacts.filter(c => c.id !== id));
  }
};

const updateContact = (id: string, field: keyof VendorContact, value: any) => {
  setContacts(contacts.map(c => 
    c.id === id ? { ...c, [field]: value } : c
  ));
};
```

**過濾與轉換**
```typescript
const filterValidContacts = (contacts: VendorContact[]): VendorContact[] => {
  return contacts.filter(c => 
    c.name?.trim() || c.email?.trim() || c.phone?.trim()
  );
};

const filterExpiredQuotes = (quotes: Quote[]): Quote[] => {
  const now = new Date();
  return quotes.filter(q => new Date(q.validUntil) <= now);
};
```

**計算與統計**
```typescript
const calculateAveragePrice = (quotes: Quote[]): number => {
  if (quotes.length === 0) return 0;
  const total = quotes.reduce((sum, q) => sum + q.price, 0);
  return total / quotes.length;
};

const getDaysUntilExpiry = (validUntil: string): number => {
  const now = new Date();
  const expiryDate = new Date(validUntil);
  const diffTime = expiryDate.getTime() - now.getTime();
  return Math.ceil(diffTime / (1000 * 60 * 60 * 24));
};
```

---

## 五、命名規範

### 變數命名
```typescript
// ✅ camelCase
const userName = 'John';
const isLoading = false;
const hasError = false;

// ✅ 布林值以 is, has, should, can 開頭
const isExpired = true;
const hasValidData = false;
const shouldShowModal = true;
const canSubmit = false;

// ✅ 常數使用 UPPER_SNAKE_CASE
const API_BASE_URL = 'https://api.example.com';
const MAX_RETRY_COUNT = 3;
const DEFAULT_PAGE_SIZE = 10;

// ✅ 陣列使用複數形式
const quotes = [];
const vendors = [];
const customFields = [];
```

### 組件和型別命名
```typescript
// ✅ PascalCase
interface QuoteListProps { }
type VendorType = 'shipping' | 'trucking' | 'customs';
export function QuoteList() { }
```

### 檔案命名
```typescript
// ✅ 組件檔案：PascalCase
QuoteList.tsx
VendorDialog.tsx
AddQuoteDialog.tsx

// ✅ 工具檔案：camelCase
formValidation.ts
dateUtils.ts

// ✅ 頁面檔案：PascalCase + Page 後綴
DashboardPage.tsx
QuotesPage.tsx
VendorsPage.tsx
```

---

## 六、狀態管理

### 基本原則
- 優先使用 React 內建狀態管理（useState, useContext）
- 避免 prop drilling，超過 3 層考慮使用 Context
- 表單狀態使用 React Hook Form
- API 請求狀態模式：loading, error, data

### 範例
```typescript
// ✅ 基本狀態管理
const [quotes, setQuotes] = useState<Quote[]>([]);
const [loading, setLoading] = useState(false);
const [error, setError] = useState<string | null>(null);

// ✅ 複雜狀態使用物件
const [formData, setFormData] = useState({
  name: '',
  email: '',
  phone: '',
});

// ✅ 更新物件狀態 - 使用展開運算符
setFormData({ ...formData, name: 'New Name' });

// ✅ 更新陣列狀態 - immutable 方式
setQuotes([...quotes, newQuote]); // 新增
setQuotes(quotes.filter(q => q.id !== id)); // 刪除
setQuotes(quotes.map(q => q.id === id ? updatedQuote : q)); // 更新
```

---

## 七、API 與資料獲取

### 基本規則
- 使用 async/await 而非 Promise chains
- 所有 API 請求必須包含錯誤處理
- 使用 try-catch 捕獲錯誤並提供用戶友好的錯誤訊息
- API 端點集中管理
- 使用 `Promise.all` 並行處理獨立請求

### 範例
```typescript
// ✅ 標準 API 請求模式
const loadQuotes = async () => {
  try {
    setLoading(true);
    setError(null);
    
    const response = await fetch(`${API_URL}/quotes`, {
      headers: {
        'Authorization': `Bearer ${token}`,
      },
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    setQuotes(data);
  } catch (error) {
    console.error('載入報價時發生錯誤:', error);
    setError('無法載入報價資料');
    toast.error('無法載入報價資料，請稍後再試');
  } finally {
    setLoading(false);
  }
};

// ✅ 並行請求
const loadData = async () => {
  try {
    const [quotesRes, vendorsRes] = await Promise.all([
      fetch(`${API_URL}/quotes`),
      fetch(`${API_URL}/vendors`),
    ]);

    const quotesData = await quotesRes.json();
    const vendorsData = await vendorsRes.json();

    setQuotes(quotesData);
    setVendors(vendorsData);
  } catch (error) {
    console.error('Error loading data:', error);
  }
};
```

---

## 八、樣式與 UI

### Tailwind CSS 規範
- 使用 Tailwind CSS utility classes
- 遵循 mobile-first 響應式設計
- 顏色使用語義化命名
- 使用 Radix UI 組件作為基礎
- 條件樣式使用模板字串和三元運算符

### 範例
```typescript
// ✅ 基本樣式
<div className="flex items-center justify-between p-4 bg-white rounded-lg shadow">

// ✅ 響應式設計 (mobile-first)
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">

// ✅ 條件樣式
<div className={`border rounded-lg ${
  isExpired ? 'border-red-200 bg-red-50' : 'border-gray-200 bg-white'
}`}>

// ✅ 使用 cn 工具函數處理複雜條件
import { cn } from '@/lib/utils';

<div className={cn(
  "base-class",
  isActive && "active-class",
  isDisabled && "disabled-class"
)}>

// ✅ 顏色語義化
text-gray-900    // 主要文字
text-gray-500    // 次要文字
bg-blue-600      // 主要按鈕
bg-red-50        // 錯誤背景
border-gray-200  // 邊框
```

### 可訪問性 (a11y)
```typescript
// ✅ 語義化 HTML
<button type="button">提交</button>
<nav>...</nav>
<main>...</main>

// ✅ 圖標按鈕提供 aria-label
<button aria-label="刪除報價">
  <Trash2 className="w-4 h-4" />
</button>

// ✅ 表單輸入關聯 label
<label htmlFor="email">Email</label>
<input id="email" type="email" />

// ✅ 鍵盤訪問
<div 
  role="button" 
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => e.key === 'Enter' && handleClick()}
>
```

---

## 九、錯誤處理

### 基本原則
- 所有 async 操作必須有 try-catch
- 使用 `console.error` 記錄錯誤，包含上下文訊息
- 向用戶顯示友好的中文錯誤訊息
- 使用 Sonner toast 顯示操作結果通知

### 範例
```typescript
// ✅ 完整錯誤處理
const saveQuote = async (quote: Quote) => {
  try {
    const response = await fetch(`${API_URL}/quotes`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`,
      },
      body: JSON.stringify(quote),
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    toast.success('報價已成功保存');
    return data;
  } catch (error) {
    console.error('保存報價時發生錯誤:', error);
    toast.error('無法保存報價，請稍後再試');
    throw error; // 如需在上層處理，重新拋出
  }
};

// ✅ 使用者操作確認
const handleDelete = (id: string) => {
  if (confirm('確定要刪除這個報價嗎？此操作無法復原。')) {
    deleteQuote(id);
  }
};
```

---

## 十、性能優化

### 基本原則
- 使用 `React.memo` 包裝純組件（需要時）
- 使用 `useMemo` 快取昂貴的計算
- 使用 `useCallback` 穩定回調函數引用（傳遞給子組件時）
- 列表渲染必須提供穩定的 `key` 屬性
- 避免在 render 中創建新的物件或陣列

### 範例
```typescript
// ✅ 使用 useMemo 快取計算
const averagePrice = useMemo(() => {
  if (quotes.length === 0) return 0;
  return quotes.reduce((sum, q) => sum + q.price, 0) / quotes.length;
}, [quotes]);

// ✅ 使用 useCallback 穩定函數引用
const handleDelete = useCallback((id: string) => {
  setQuotes(prev => prev.filter(q => q.id !== id));
}, []);

// ✅ 列表渲染使用穩定的 key
{quotes.map((quote) => (
  <QuoteItem key={quote.id} quote={quote} />
))}

// ❌ 避免在 render 中創建新物件
<Component style={{ margin: 10 }} /> // 每次 render 都創建新物件

// ✅ 提取到外部或使用 useMemo
const componentStyle = { margin: 10 };
<Component style={componentStyle} />
```

---

## 十一、程式碼組織

### 目錄結構
```
src/
├── components/        # 可重用組件
│   ├── ui/           # UI 基礎組件 (Radix UI)
│   ├── QuoteList.tsx
│   └── VendorDialog.tsx
├── pages/            # 頁面組件
│   ├── DashboardPage.tsx
│   └── QuotesPage.tsx
├── utils/            # 工具函數
│   ├── formValidation.ts
│   └── supabase/
├── styles/           # 全域樣式
└── App.tsx           # 主應用組件
```

### 匯入順序
```typescript
// 1. React 相關
import { useState, useEffect } from 'react';

// 2. 第三方套件
import { toast } from 'sonner';

// 3. 型別定義
import type { Quote, Vendor } from '../App';

// 4. 組件
import { Button } from './ui/button';
import { Input } from './ui/input';

// 5. 工具函數
import { formatDate } from '../utils/dateUtils';

// 6. 樣式
import '../styles/custom.css';
```

---

## 十二、註解與文檔

### 註解原則
- 複雜邏輯必須添加註解說明
- 使用 JSDoc 註解公共 API 和工具函數
- 避免無意義的註解（如 `// set state`）
- 中文註解用於業務邏輯說明，英文註解用於技術實現

### JSDoc 範例
```typescript
/**
 * 設置表單元素的中文驗證訊息
 * @param formElement - 要設置驗證的表單元素
 * @returns void
 */
export function setupFormValidation(formElement: HTMLFormElement | null): void {
  if (!formElement) return;
  // ...
}

/**
 * 計算報價到期前的剩餘天數
 * @param validUntil - ISO 8601 格式的日期字串
 * @returns 剩餘天數，如已過期則返回負數
 */
const getDaysUntilExpiry = (validUntil: string): number => {
  const now = new Date();
  const expiryDate = new Date(validUntil);
  const diffTime = expiryDate.getTime() - now.getTime();
  return Math.ceil(diffTime / (1000 * 60 * 60 * 24));
};
```

---

## 十三、專案特定規則

### 報價系統
- 報價型別：`shipping` (海運), `trucking` (拖車), `customs` (報關)
- 所有日期使用 ISO 8601 格式
- 價格顯示使用 `toLocaleString()` 格式化
- 過期報價需要視覺提示（紅色邊框）

### 供應商管理
- 供應商可以有多個聯絡人
- 至少一個聯絡人標記為 `isPrimary`

### 自定義欄位
- 支援動態自定義欄位
- 欄位定義從 API 載入
- 使用 `customFields` 物件存儲值

### 國際化
- UI 文字使用繁體中文（zh-TW）
- 日期格式遵循台灣習慣
- 保留英文用於技術術語

---

## 十四、Git 提交規範

使用語義化提交訊息：

```
feat: 新增報價比較功能
fix: 修復供應商列表排序錯誤
refactor: 重構表單驗證邏輯
style: 調整報價卡片樣式
docs: 更新 API 文檔
perf: 優化報價列表渲染性能
test: 新增供應商驗證測試
chore: 更新依賴套件
```

---

## 十五、禁止事項

```typescript
// ❌ 不使用 var
var name = 'John';

// ✅ 使用 const 或 let
const name = 'John';
let count = 0;

// ❌ 不使用內聯樣式（除非動態計算）
<div style={{ color: 'red' }}>

// ✅ 使用 Tailwind classes
<div className="text-red-600">

// ❌ 不直接修改 state
state.push(newItem);

// ✅ 使用 immutable 更新
setState([...state, newItem]);

// ❌ 不在循環中使用 Hooks
for (let i = 0; i < 10; i++) {
  const [state, setState] = useState(); // 錯誤！
}

// ❌ 不忽略 ESLint 警告
// eslint-disable-next-line

// ❌ 不使用 @ts-ignore
// @ts-ignore

// ✅ 使用 @ts-expect-error 並說明原因
// @ts-expect-error - 第三方套件型別定義不完整
const data = externalLib.getData();

// ❌ 不在組件內定義組件
function Parent() {
  function Child() { } // 錯誤！
  return <Child />;
}

// ✅ 在外部定義
function Child() { }
function Parent() {
  return <Child />;
}

// ❌ 不在循環中定義函數
quotes.map(quote => {
  const formatPrice = () => { }; // 錯誤！
  return formatPrice();
});

// ✅ 在外部定義
const formatPrice = (price: number) => { };
quotes.map(quote => formatPrice(quote.price));
```

---

## 十六、最佳實踐總結

### 程式碼品質
1. **型別安全**：所有函數參數和返回值都要明確標註型別
2. **單一職責**：每個函數/組件只做一件事
3. **命名清晰**：變數、函數、組件名稱應該清楚表達其用途
4. **提早返回**：減少嵌套，提高可讀性
5. **錯誤處理**：所有 async 操作必須有 try-catch

### 性能
6. **避免重複渲染**：使用 React.memo、useMemo、useCallback
7. **並行請求**：使用 Promise.all 處理獨立的 API 請求
8. **列表優化**：提供穩定的 key，避免不必要的重新渲染

### 可維護性
9. **程式碼組織**：相關功能放在一起，保持檔案結構清晰
10. **註解適當**：複雜邏輯添加註解，但不過度註解
11. **保持一致**：整個專案使用統一的命名和結構風格
12. **可測試性**：函數應該易於測試和模擬

### 用戶體驗
13. **錯誤提示**：提供友好的中文錯誤訊息
14. **載入狀態**：顯示載入指示器
15. **操作確認**：重要操作（如刪除）需要確認
16. **可訪問性**：確保鍵盤訪問和螢幕閱讀器支援

---

## 持續改進

- 定期重構以提高程式碼品質
- 保持依賴更新
- 遵循 React 和 TypeScript 最新最佳實踐
- 優先考慮可維護性和可讀性
- 團隊成員定期 code review
- 記錄並分享最佳實踐

---

**記住：好的程式碼不僅能運行，更要易讀、易維護、易擴展！**
**記住：用中文回覆！**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kurt-liao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
