## react

> - 生成 `React` 组件时，尽量使用函数式组件，而不是类组件


# React Rules

## 规则

- 生成 `React` 组件时，尽量使用函数式组件，而不是类组件

- 禁止使用 `<></>`，必须使用从 `React` 导入的 `Fragment` 组件

- 组件的 `props` 书写的优先级为：身份属性 (`ref`、`key`、`id`) > 样式属性 (`className`、`classNames`、`style`、`size` 等等) > 其他属性 (`value`、`defaultValue` 等等) > 回调事件 (`onClick`、`onChange` 等等)

- 请始终使用 `on` + 事件名作为事件处理函数的名称，比如 `onClick` 事件处理函数应该命名为 `onClick`，而不是 `handleClick`

- 你应该将根组件的 `props` 当做基础的 `props` 类型，将当前组件所需的原始数据当做 `data` 属性

    ```tsx
    import { ComponentProps, FC } from "react"

    import { clsx, StrictOmit } from "deepsea-tools"

    export interface Book {
        id: string
        name: string
        isbn: string
    }

    export interface BookProps extends StrictOmit<ComponentProps<"div">, "children"> {
        data?: Book
    }

    const Book: FC<BookProps> = ({ className, data, ...rest }) => (
        <div className={clsx("container", className)} {...rest}>
            <div>{data?.name}</div>
            <div>{data?.isbn}</div>
        </div>
    )

    export default Book
    ```

    因为 `Book` 组件的根元素是 `div`，所以 `BookProps` 类型应该继承自 `StrictOmit<ComponentProps<"div">, "children">`，如果 `Book` 组件的根组件不是 `html` 元素，例如 `Container` 组件，则应该继承自 `StrictOmit<ComponentProps<typeof Container>, "children">`，或者如果存在 `ContainerProps` 类型，则应该继承自 `StrictOmit<ContainerProps, "children">`

    `data` 属性是指整个项目中某种数据的原始类型，例如从 `queryBook` 接口等 api 函数中获取到的数据，这时 `data` 的类型就是 `Book` 类型

- 尽量直接在函数式组件的参数中解构 `props`，获取需要使用的属性，将剩余的属性作为 `rest` 属性

- 如果你需要根组件设置 `className`，请使用从 `deepsea-tools` 中导入的 `clsx` 函数来合并 `className`，例如上方的：

    ```tsx
    return (
        <div className={clsx("container", className)} {...rest}>
            ...
        </div>
    )
    ```

- 如果组件是一个受控组件，请使用 `value` 和 `onValueChange` 来实现受控组件，这两个属性都应该是可选，并且在组件内部，你应该使用从 `soda-hooks` 中导入的 `useInputState` 的钩子来实现内部状态与外部状态的同步，例如：

    ```tsx
    import { ComponentProps, FC } from "react"

    import { StrictOmit } from "deepsea-tools"

    export interface MyInputProps extends StrictOmit<ComponentProps<typeof OtherInput>, "value" | "onValueChange"> {
        value?: string
        onValueChange?: (value: string) => void
    }

    const MyInput: FC<MyInputProps> = ({ value: _value, onValueChange: _onValueChange, ...rest }) => {
        const [value, setValue] = useInputState(_value)

        function onValueChange(value: string) {
            setValue(value)
            _onValueChange?.(value)
        }

        return <OtherInput value={value} onValueChange={onValueChange} {...rest} />
    }

    export default MyInput
    ```

- 如果你需要使用 `React` 中的某个导入，请使用 `import { xxx } from "react"` 而不是 `React.xxx` 的形式，如果已经存在同名的变量或者类型，请使用 `import { xxx as reactXxx } from "react"`，变量使用小驼峰命名，类型使用大驼峰命名

- 如果你需要在组件内部添加一个事件处理函数，而组件的 `props` 中存在同名的事件处理函数，你应该这样处理：

    ```tsx
    // 因为 global 中存在 MouseEvent 类型，与 react 中的 MouseEvent 类型冲突，所以需要将 react 中的 MouseEvent 类型重命名为 ReactMouseEvent
    import { ComponentProps, FC, MouseEvent as ReactMouseEvent } from "react"

    import { StrictOmit } from "deepsea-tools"

    export interface AppProps extends StrictOmit<ComponentProps<"div">, "children"> {}

    // 将 props 中的同名事件处理函数加一个下划线前缀
    const App: FC<AppProps> = ({ onClick: _onClick, ...rest }) => {
        function onClick(event: ReactMouseEvent<HTMLDivElement, MouseEvent>) {
            // 优先处理内部逻辑
            console.log("onClick")

            // 然后调用外部的事件处理函数
            _onClick?.(event)
        }

        return (
            <div onClick={onClick} {...rest}>
                Hello World!
            </div>
        )
    }

    export default App
    ```

- 如果你的组件内部没有任何逻辑，只有 `return` 一个组件，请直接返回该组件，不要使用 `return` 关键字，例如：

    ```tsx
    const App: FC<AppProps> = ({ className, ...rest }) => (
        <div className={clsx("container", className)} {...rest}>
            Hello World!
        </div>
    )
    ```

- 当你在组件内部需要获取根组件的 `ref`，而 `props` 中也有 `ref` 属性时，你应该这样处理：

    ```tsx
    const App: FC<AppProps> = ({ ref, ...rest }) => {
        const container = useRef<HTMLDivElement>(null)

        useImperativeHandle(ref, () => container.current!)

        return (
            <div ref={container} {...rest}>
                Hello World!
            </div>
        )
    }
    ```

- 如果组件没有 `children`，请使用自闭合标签，例如 `<div />` 而不是 `<div></div>`

- 如果 jsx 中某个元素的属性（非 `children` 属性）的类型为回调函数，并且这个回调函数无法使用一行代码完成，请使用 `function` 关键字声明一个函数，然后传递给该属性，例如：

    ```tsx
    const App: FC<AppProps> = ({ className, ...rest }) => {
        function onClick(event: ReactMouseEvent<HTMLDivElement, MouseEvent>) {
            console.log("onClick")
            doSomething()
        }

        return (
            <div onClick={onClick} {...rest}>
                Hello World!
            </div>
        )
    }
    ```

- 如果你使用的是 `shadcn/ui` 的组件，禁止自动生成组件代码，必须使用命令行工具 `npx shadcn@latest add <component-name>` 来添加组件

- 禁止修改 `shadcn/ui` 添加的原始组件，一般路径为 `@/components/ui/**/*.tsx`

- 如果你使用的是 `ai-elements` 的组件，禁止修改原始组件，一般路径为 `@/components/ai-elements/**/*.tsx`

## 组件与页面

请遵循以下规则生成组件或页面，并在新增时考虑复用与抽取：

1. 先分析页面结构，识别重复的 UI 片段与逻辑，并判断是否值得抽取。不要为了抽取而抽取，优先考虑维护成本。
2. 抽取的组件应该放在公共的 `@/components` 目录下，工具函数应该放在公共的 `@/utils` 目录下，禁止放在其他目录下。
3. 新增组件或页面前，检查已有目录（尤其是 `@/components` 与 `@/utils`）是否已有可复用实现，优先复用而非重复创建。
4. 抽取时保持原有 UI 风格与交互一致，避免引入不必要的样式或行为变化。
5. 组件拆分要能提升可读性与可测试性；若拆分后跨文件沟通成本增加，则保留在原文件。
6. 对抽取出的组件与工具，提供清晰的 props 或函数签名与命名，便于后续维护与扩展。

---
> Source: [1adybug/docker-management](https://github.com/1adybug/docker-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
