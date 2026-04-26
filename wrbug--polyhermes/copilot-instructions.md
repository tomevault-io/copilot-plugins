## polyhermes

> - **禁止**在代码中添加 TODO/FIXME/XXX 注释


# 前端开发规范

## 核心原则
- **禁止**在代码中添加 TODO/FIXME/XXX 注释
- **禁止**使用 `any` 类型
- **禁止**硬编码中文或英文文本
- **必须**支持移动端和桌面端

## 技术栈
- **框架**: React + TypeScript
- **UI 库**: Ant Design（移动端使用 Ant Design Mobile）
- **HTTP 客户端**: axios
- **状态管理**: Zustand
- **国际化**: react-i18next

## 移动端适配

### 响应式断点
- **移动端**: < 768px
- **平板**: 768px - 1024px
- **桌面端**: > 1024px

### 适配要求
- 使用响应式布局（Mobile First）
- 导航栏：移动端抽屉菜单，桌面端顶部导航
- 列表：移动端卡片布局，桌面端表格布局
- 按钮：移动端不小于 44x44px
- 支持触摸操作和手势

### 性能优化
- 图片懒加载
- 代码分割（Code Splitting）
- 移动端减少动画效果

## 代码规范

### 组件
- 使用函数式组件
- 使用 TypeScript 类型定义
- 组件文件使用 PascalCase 命名

```typescript
// ✅ 正确
interface MarketProps {
  marketId: string;
  platform: string;
}

export const MarketCard: React.FC<MarketProps> = ({ marketId, platform }) => {
  // ...
};

// ❌ 错误
export const marketCard = (props: any) => { }
```

### API 调用
- 使用 axios
- 统一错误处理
- 使用 TypeScript 定义响应类型

```typescript
interface Market {
  id: string;
  marketId: string;
  title: string;
}

export const marketService = {
  getMarkets: async (): Promise<Market[]> => {
    const response = await axios.get<Market[]>('/api/markets');
    return response.data;
  }
};
```

### 状态管理
- 全局状态使用 Zustand
- 本地状态使用 `useState`
- 复杂状态使用 `useReducer`

```typescript
interface MarketStore {
  markets: Market[];
  setMarkets: (markets: Market[]) => void;
}

export const useMarketStore = create<MarketStore>((set) => ({
  markets: [],
  setMarkets: (markets) => set({ markets }),
}));
```

### USDC 金额格式化
**必须**使用 `formatUSDC` 函数（从 `../utils` 导入）

```typescript
import { formatUSDC } from '../utils'

// ✅ 正确
<Statistic
  title="总盈亏"
  value={formatUSDC(stats?.totalPnl || '0')}
  suffix="USDC"
/>

// ❌ 错误
const balance = parseFloat(value).toFixed(4)
```

**格式化规则**:
- 最多显示 4 位小数（截断，不四舍五入）
- 自动去除尾随零
- 空值返回 `'-'`

## 多语言规范

### 使用方式
**必须**使用 `useTranslation` Hook

```typescript
// ✅ 正确
import { useTranslation } from 'react-i18next'

const MyComponent: React.FC = () => {
  const { t } = useTranslation()
  
  return (
    <div>
      <h1>{t('page.title')}</h1>
      <Button>{t('common.save')}</Button>
    </div>
  )
}

// ❌ 错误：硬编码文本
<h1>页面标题</h1>
<Button>保存</Button>
```

### 语言文件
- 位置: `src/locales/{locale}/common.json`
- 支持: `zh-CN`、`zh-TW`、`en`
- 键名: 点号分隔的层级结构（如 `notificationSettings.title`）

### 添加新翻译
1. 在 `src/locales/zh-CN/common.json` 添加中文
2. 在 `src/locales/zh-TW/common.json` 添加繁体
3. 在 `src/locales/en/common.json` 添加英文
4. 在组件中使用 `t('key')`

## 移动端适配示例

### 响应式布局
```typescript
import { useMediaQuery } from 'react-responsive';

const MyComponent: React.FC = () => {
  const isMobile = useMediaQuery({ maxWidth: 768 });
  
  return (
    <div className={isMobile ? 'mobile-layout' : 'desktop-layout'}>
      {isMobile ? <MobileView /> : <DesktopView />}
    </div>
  );
};
```

### 响应式表格
```typescript
import { Table } from 'antd';

const ResponsiveTable: React.FC = () => {
  const isMobile = useMediaQuery({ maxWidth: 768 });
  
  return (
    <Table
      dataSource={data}
      columns={columns}
      scroll={{ x: isMobile ? 600 : 'auto' }}
      pagination={{
        pageSize: isMobile ? 10 : 20,
        showSizeChanger: !isMobile
      }}
    />
  );
};
```

## 禁止事项

### 代码质量
- ❌ 禁止使用 `any` 类型
- ❌ 禁止忽略错误处理
- ❌ 禁止硬编码 API 地址
- ❌ 禁止在组件中直接使用 `fetch`
- ❌ 禁止忽略移动端适配

### 类型安全
- ❌ 禁止使用 `@ts-ignore` 除非有明确原因

### 移动端适配
- ❌ 禁止固定宽度布局
- ❌ 禁止使用过小的触摸目标（< 44x44px）
- ❌ 禁止使用桌面端专用交互（如 hover）

### 多语言
- ❌ 禁止硬编码中文或英文文本
- ❌ 禁止绕过 `useTranslation` 直接显示文本

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WrBug) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
