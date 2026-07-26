## blog

> TypeScript coding standards and type safety guidelines for enterprise applications


# 📝 TypeScript 编码标准 - TypeScript Coding Standards

## 🏗️ 类型定义标准 (Type Definition Standards)

### 🎯 接口设计原则

```typescript
// ✅ 良好的接口设计
interface WeatherData {
  readonly location: {
    readonly name: string
    readonly country: string
    readonly coordinates: {
      readonly lat: number
      readonly lon: number
    }
  }
  readonly current: CurrentWeather
  readonly snowDay: SnowDayPrediction
  readonly timestamp: string
}

interface CurrentWeather {
  readonly temperature: number
  readonly feelsLike: number
  readonly humidity: number
  readonly pressure: number
  readonly description: string
  readonly icon: string
  readonly windSpeed: number
  readonly windSpeedKmh: number
  readonly visibility: number
  readonly visibilityKm: number
  readonly snowfall: number
  readonly cloudCover: number
}

interface SnowDayPrediction {
  readonly probability: number
  readonly level: SnowDayLevel
  readonly color: string
  readonly recommendation: string
  readonly factors: SnowDayFactors
}

// 使用字面量类型确保类型安全
type SnowDayLevel = "Very Low" | "Low" | "Moderate" | "High" | "Very High"
type SearchType = "zip" | "city" | "coords"
type LoadingState = "idle" | "loading" | "success" | "error"
```

### 🔧 泛型与实用类型

```typescript
// 通用API响应类型
interface ApiResponse<T> {
  readonly data: T
  readonly success: boolean
  readonly message?: string
  readonly timestamp: string
}

interface ApiError {
  readonly error: string
  readonly code?: number
  readonly details?: Record<string, unknown>
}

// 条件类型用于API状态
type ApiResult<T> = ApiResponse<T> | ApiError

// 实用类型组合
type PartialUpdate<T> = Partial<Pick<T, keyof T>>
type RequiredFields<T, K extends keyof T> = T & Required<Pick<T, K>>

// 状态管理类型
interface AsyncState<T> {
  readonly data: T | null
  readonly loading: boolean
  readonly error: string | null
  readonly lastUpdated: Date | null
}

// Hook返回类型
interface UseAsyncResult<T> extends AsyncState<T> {
  readonly execute: (params?: unknown) => Promise<void>
  readonly reset: () => void
}
```

### 🎨 React组件类型

```typescript
// 组件Props类型定义
interface ButtonProps {
  readonly variant?: "primary" | "secondary" | "danger"
  readonly size?: "sm" | "md" | "lg"
  readonly loading?: boolean
  readonly disabled?: boolean
  readonly children: React.ReactNode
  readonly onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void
  readonly className?: string
  readonly type?: "button" | "submit" | "reset"
}

// 泛型组件类型
interface SelectOption<T = string> {
  readonly value: T
  readonly label: string
  readonly disabled?: boolean
}

interface SelectProps<T> {
  readonly options: SelectOption<T>[]
  readonly value: T | null
  readonly onChange: (value: T) => void
  readonly placeholder?: string
  readonly disabled?: boolean
  readonly multiple?: boolean
}

// 表单相关类型
interface FormFieldProps<T> {
  readonly name: keyof T
  readonly value: T[keyof T]
  readonly onChange: (name: keyof T, value: T[keyof T]) => void
  readonly error?: string
  readonly required?: boolean
  readonly disabled?: boolean
}

// 高阶组件类型
type WithLoadingProps<T> = T & {
  readonly isLoading: boolean
}

type HOC<TOriginalProps, TInjectedProps = {}> = (
  Component: React.ComponentType<TOriginalProps>
) => React.ComponentType<TOriginalProps & TInjectedProps>
```

## 🎯 函数与Hook类型安全

### 🪝 自定义Hook类型

```typescript
// 异步数据获取Hook
function useAsyncData<T>(
  fetcher: () => Promise<T>,
  dependencies: React.DependencyList = []
): UseAsyncResult<T> {
  const [state, setState] = useState<AsyncState<T>>({
    data: null,
    loading: false,
    error: null,
    lastUpdated: null,
  })

  const execute = useCallback(async () => {
    setState((prev) => ({ ...prev, loading: true, error: null }))

    try {
      const data = await fetcher()
      setState({
        data,
        loading: false,
        error: null,
        lastUpdated: new Date(),
      })
    } catch (error) {
      setState((prev) => ({
        ...prev,
        loading: false,
        error: error instanceof Error ? error.message : "Unknown error",
      }))
    }
  }, dependencies)

  const reset = useCallback(() => {
    setState({
      data: null,
      loading: false,
      error: null,
      lastUpdated: null,
    })
  }, [])

  useEffect(() => {
    execute()
  }, [execute])

  return { ...state, execute, reset }
}

// 本地存储Hook
function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T | ((prev: T) => T)) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === "undefined") return initialValue

    try {
      const item = window.localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch (error) {
      console.warn(`Error reading localStorage key "${key}":`, error)
      return initialValue
    }
  })

  const setValue = useCallback(
    (value: T | ((prev: T) => T)) => {
      try {
        const valueToStore = value instanceof Function ? value(storedValue) : value
        setStoredValue(valueToStore)

        if (typeof window !== "undefined") {
          window.localStorage.setItem(key, JSON.stringify(valueToStore))
        }
      } catch (error) {
        console.warn(`Error setting localStorage key "${key}":`, error)
      }
    },
    [key, storedValue]
  )

  return [storedValue, setValue]
}
```

### 🔄 事件处理类型

```typescript
// 表单事件处理
type FormSubmitHandler<T = HTMLFormElement> = (event: React.FormEvent<T>) => void | Promise<void>

type InputChangeHandler<T = HTMLInputElement> = (event: React.ChangeEvent<T>) => void

type ButtonClickHandler<T = HTMLButtonElement> = (
  event: React.MouseEvent<T>
) => void | Promise<void>

// 搜索相关事件
interface SearchEvents {
  readonly onSubmit: FormSubmitHandler
  readonly onInputChange: InputChangeHandler
  readonly onTypeChange: (type: SearchType) => void
  readonly onLocationSelect: (location: string) => void
  readonly onGeoLocation: ButtonClickHandler
}

// 键盘事件处理
type KeyboardEventHandler<T = HTMLElement> = (event: React.KeyboardEvent<T>) => void

const handleKeyDown: KeyboardEventHandler = (event) => {
  if (event.key === "Enter" && !event.shiftKey) {
    event.preventDefault()
    // 处理回车键
  }
}
```

## 🛡️ 类型守卫与验证

### ✅ 类型守卫函数

```typescript
// 基础类型守卫
function isString(value: unknown): value is string {
  return typeof value === "string"
}

function isNumber(value: unknown): value is number {
  return typeof value === "number" && !isNaN(value)
}

function isArray<T>(value: unknown): value is T[] {
  return Array.isArray(value)
}

// 对象类型守卫
function isWeatherData(value: unknown): value is WeatherData {
  return (
    typeof value === "object" &&
    value !== null &&
    "location" in value &&
    "current" in value &&
    "snowDay" in value &&
    "timestamp" in value
  )
}

function hasProperty<T extends PropertyKey>(obj: object, prop: T): obj is Record<T, unknown> {
  return prop in obj
}

// API响应验证
function isApiResponse<T>(
  value: unknown,
  dataValidator: (data: unknown) => data is T
): value is ApiResponse<T> {
  return (
    typeof value === "object" &&
    value !== null &&
    hasProperty(value, "data") &&
    hasProperty(value, "success") &&
    typeof value.success === "boolean" &&
    dataValidator(value.data)
  )
}

// 使用示例
async function fetchWeatherData(location: string): Promise<WeatherData> {
  const response = await fetch(`/api/weather?q=${encodeURIComponent(location)}`)
  const data = await response.json()

  if (!isWeatherData(data)) {
    throw new Error("Invalid weather data received")
  }

  return data
}
```

### 🔍 运行时验证

```typescript
// Zod schema验证
import { z } from "zod"

const WeatherRequestSchema = z
  .object({
    type: z.enum(["city", "coords", "zip"]),
    q: z.string().optional(),
    lat: z
      .string()
      .regex(/^-?\d+\.?\d*$/)
      .optional(),
    lon: z
      .string()
      .regex(/^-?\d+\.?\d*$/)
      .optional(),
    zip: z.string().min(3).max(10).optional(),
  })
  .refine(
    (data) => {
      if (data.type === "city") return data.q
      if (data.type === "coords") return data.lat && data.lon
      if (data.type === "zip") return data.zip
      return false
    },
    {
      message: "Missing required fields for search type",
    }
  )

type WeatherRequest = z.infer<typeof WeatherRequestSchema>

// 表单验证Schema
const SearchFormSchema = z.object({
  searchType: z.enum(["zip", "city", "coords"]),
  location: z.string().min(1, "Location is required"),
  useGeolocation: z.boolean().optional(),
})

type SearchFormData = z.infer<typeof SearchFormSchema>
```

## 🎯 错误处理类型

### 🚨 错误类型定义

```typescript
// 自定义错误类
class WeatherApiError extends Error {
  constructor(
    message: string,
    public readonly code?: number,
    public readonly details?: Record<string, unknown>
  ) {
    super(message)
    this.name = "WeatherApiError"
  }
}

class ValidationError extends Error {
  constructor(
    message: string,
    public readonly field: string,
    public readonly value: unknown
  ) {
    super(message)
    this.name = "ValidationError"
  }
}

// 错误联合类型
type AppError = WeatherApiError | ValidationError | Error

// Result类型模式
type Result<T, E = Error> = { success: true; data: T } | { success: false; error: E }

// 安全的异步函数包装
async function safeAsync<T>(fn: () => Promise<T>): Promise<Result<T, AppError>> {
  try {
    const data = await fn()
    return { success: true, data }
  } catch (error) {
    return {
      success: false,
      error: error instanceof Error ? error : new Error(String(error)),
    }
  }
}

// 使用示例
async function handleSearch(location: string): Promise<Result<WeatherData, AppError>> {
  return safeAsync(async () => {
    const validation = SearchFormSchema.safeParse({ location, searchType: "city" })

    if (!validation.success) {
      throw new ValidationError("Invalid input", "location", location)
    }

    const response = await fetch(`/api/weather?q=${encodeURIComponent(location)}`)

    if (!response.ok) {
      throw new WeatherApiError(`HTTP ${response.status}`, response.status)
    }

    const data = await response.json()

    if (!isWeatherData(data)) {
      throw new WeatherApiError("Invalid response format")
    }

    return data
  })
}
```

## 🔧 实用工具类型

### 🛠️ 通用工具类型

```typescript
// 深度只读
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P]
}

// 深度可选
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P]
}

// 提取函数参数类型
type ExtractParams<T> = T extends (...args: infer P) => unknown ? P : never

// 提取Promise类型
type Awaited<T> = T extends Promise<infer U> ? U : T

// 条件类型工具
type NonNullable<T> = T extends null | undefined ? never : T
type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>
type Required<T, K extends keyof T> = T & Required<Pick<T, K>>

// 字符串操作类型
type Capitalize<S extends string> = S extends `${infer T}${infer U}` ? `${Uppercase<T>}${U}` : S

type CamelCase<S extends string> = S extends `${infer P1}_${infer P2}${infer P3}`
  ? `${P1}${Uppercase<P2>}${CamelCase<P3>}`
  : S

// 对象路径类型
type Path<T> = T extends object
  ? {
      [K in keyof T]: K extends string
        ? T[K] extends object
          ? `${K}` | `${K}.${Path<T[K]>}`
          : `${K}`
        : never
    }[keyof T]
  : never

type PathValue<T, P extends Path<T>> = P extends `${infer K}.${infer Rest}`
  ? K extends keyof T
    ? Rest extends Path<T[K]>
      ? PathValue<T[K], Rest>
      : never
    : never
  : P extends keyof T
    ? T[P]
    : never
```

### 🎨 组件工具类型

```typescript
// 多态组件类型
type PolymorphicRef<C extends React.ElementType> =
  React.ComponentPropsWithRef<C>['ref']

type AsProp<C extends React.ElementType> = {
  as?: C
}

type PropsToOmit<C extends React.ElementType, P> =
  keyof (AsProp<C> & P)

type PolymorphicComponentProp<
  C extends React.ElementType,
  Props = {}
> = React.PropsWithChildren<Props & AsProp<C>> &
  Omit<React.ComponentPropsWithoutRef<C>, PropsToOmit<C, Props>>

type PolymorphicComponentPropWithRef<
  C extends React.ElementType,
  Props = {}
> = PolymorphicComponentProp<C, Props> & { ref?: PolymorphicRef<C> }

// 使用示例
interface BoxOwnProps {
  color?: string
  size?: number
}

type BoxProps<C extends React.ElementType> =
  PolymorphicComponentPropWithRef<C, BoxOwnProps>

const Box = <C extends React.ElementType = 'div'>({
  as,
  color,
  size,
  children,
  ...rest
}: BoxProps<C>) => {
  const Component = as || 'div'
  return (
    <Component
      style={{ color, fontSize: size }}
      {...rest}
    >
      {children}
    </Component>
  )
}

// 类型安全的使用
<Box as="button" onClick={() => {}}>Button</Box>
<Box as="a" href="/">Link</Box>
```

这套TypeScript标准确保代码具备强类型安全性、良好的开发体验和运行时稳定性。

---
> Source: [geekskai/blog](https://github.com/geekskai/blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
