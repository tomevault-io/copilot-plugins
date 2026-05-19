## docker-management

> - 页面的 CSS 样式你应该尽量通过以下两种方式实现：

# Agent Rules

## Style Rules

- 页面的 CSS 样式你应该尽量通过以下两种方式实现：
    1. 对于 `Ant Design` 或者 `@heroui/react` 等组件库提供的组件，请在组件库提供的 `ConfigProvider` 等类似的全局配置组件进行修改，如果你需要修改某个组件的全局样式，你可以在 `@/components/Registry.tsx` 中进行修改，它包裹了整个应用，如果你只需要单独修改某个位置的某个组件，请使用 `ConfigProvider` 包裹你需要修改的组件
    2. 对于一般样式，优先使用组件的 `className` 或者 `classNames` 或其他类名属性 + `tailwindcss` 实现
    3. 有且仅有以上两种方式无法实现时，请你使用 `style` 属性或者在 css 文件中定义样式

- 当你使用 `flex` 布局时，对于宽度或者高度需要保持固定的子元素设置 `flex-none`
- 对于 `React` 组件（也就是非 `div` 等 `html` 元素）的样式，请谨慎使用 `!important` 修改样式，请优先使用 `ConfigProvider` 或者组件暴露的属性（比如 `radius` / `shape`等）修改样式，最后再考虑使用 `!important`
- 请不要使用模板字符串的形式来实现动态样式，例如 ``className={`w-${width}px`}``，如果你想要实现条件类名，请使用 `deepsea-tools` 中导出的 `clsx` 函数，比如 `clsx("text-base", isPrimary ? "text-primary" : "text-secondary")`

## Base Rules

- 永远使用中文回复
- 禁止修改 `node_modules` 文件夹中的任何文件
- 当我让你修复一个问题，而你尝试一次或多次修复失败后，我会在每次失败修复后的问题现象再次反馈给你，在进行下一次修复之前，你必须思考之前所有的修复是否还有必要，是否需要先撤回之前的修复，然后再进行修复
- 尽量使用 `interface` 而不是 `type`，函数类型除外
- 所有的类型定义都使用 `export` 导出
- 如果某一个属性的类型是 `[key]: someType | null`，请将它改写为 `[key]?: someType`，尽量不要使用 `null` 类型
- 禁止使用字面量类型，必须使用独立的类型定义，比如:

    ```typescript
    export interface Student {
        father: {
            name: string
            age: number
        }
    }
    ```

    你应该将 `Father` 类型独立出来，而不是使用字面量类型:

    ```typescript
    export interface Father {
        name: string
        age: number
    }

    export interface Student {
        father: Father
    }
    ```

- 尽量为代码添加注释，尽量使用 `//` 而不是 `/** */`
- 但是对于变量名、函数名、类型名、属性等具有明确意义的名称，使用 `/** 名称的作用 */` 进行注释
- 尽量使用 `const` 而不是 `let`，除非需要使用 `let` 的特性
- 尽量使用 `"` 而不是 `'`，除非是 `"` 中包含 `'`
- 尽量不要使用 `;` 进行结尾
- 尽量使用模板字符串而不是 `+` 进行字符串拼接
- 中文和英文之间加一个空格
- 不需要为类型文件单独生成一个 `types/index.ts` 文件，而是直接在需要使用的地方进行类型声明并且导出
- 当你使用 `@heroui/react` 组件库中的 `Button` 组件时，点击事件请使用 `onPress` 而不是 `onClick`
- 当你使用 `@tanstack/react-query` 的 `useQuery` 时，请使用函数名的烤肉串命名法和参数组成 `key`，例如 `queryKey: ["query-book", queryParams]`
- 函数的参数数量尽量控制在 2 个以内，如果超过 2 个，请使用对象形式的参数，参数类型名称使用函数名的大驼峰 + `Params` 后缀，例如 `QueryBookParams`
- 尽量直接从模块中导入方法，而不是使用 `默认导出.方法` 的形式

    ```typescript
    // 而不是使用 默认导出.方法 的形式
    import fs from "fs/promises"

    fs.readFile
    ```

- 如果某个方法存在同步和异步两种形式，你应该尽量使用异步形式，而不是同步形式，比如读取文件，你应该尽量使用 `fs/promises` 提供的 `readFile` 方法，而不是 `fs` 提供的 `readFileSync` 方法
- 在 `Node.js` 中，你应该尽量使用模块的 `Promise` 版本，而不是回调版本，比如读取文件，你应该尽量使用 `fs/promises` 提供的 `readFile` 方法，而不是 `fs` 提供的 `readFile` 方法
- 涉及到文件读写操作时，尽量使用 `fs` 提供的 `createReadStream` 或者 `createWriteStream` 的方式来实现，而不是一次性读取所有内容
- `Web API` 中的 `ReadableStream` 可以使用以下方法转换为 `Node.js` 中的 `Readable`:

    ```typescript
    import { Readable } from "stream"
    import { ReadableStream } from "stream/web"

    // 这里的 webStream 是 Web API 中的 ReadableStream
    const webStream = someWebApi()

    // 将 Web API 中的 ReadableStream 转换为 Node.js 中的 Readable
    const nodeStream = Readable.fromWeb(webStream as ReadableStream)
    ```

    `Web API` 中的 `WritableStream` 转换为 `Node.js` 中的 `Writable` 的方法同理

- `zod/v4` 就是 `zod` 在 `3.25` 及之后的 `v3` 版本中所提供的 `v4` 版本的 `zod`，如果当前项目的版本是 `3.25` 及之后的 `v3` 版本，你应该使用 `zod/v4` 而不是 `zod`，如果当前项目使用的就是 `zod/v4`，那你不需要检查 `zod` 的版本，保持一致，使用 `zod/v4` 即可
- 严格区分中英文标点符号，不要混用，如果用户混用了，你应该提示用户使用正确的标点符号
- 当使用网络请求时，请使用 `@/utils/request` 中的 `request` 方法，而不是使用 `fetch` 方法，`request` 方法与 `fetch` 的调用方法基本一致
- 当你需要进行包管理相关的操作时，比如安装、更新、卸载、执行 `package.json` 中的脚本或者 `npx` 执行某个命令时，请检查当前项目中的 `lock` 文件，如果是 `bun.lock`，你应该使用 `bun` 进行包管理，如果是 `package-lock.json`，你应该使用 `npm` 进行包管理，如果是 `yarn.lock`，你应该使用 `yarn` 进行包管理，如果是 `pnpm-lock.yaml`，你应该使用 `pnpm` 进行包管理，如果同时存在多个 `lock` 文件，优先级为 `bun` > `pnpm` > `yarn` > `npm`
- 请不要使用 `enum` 来声明枚举，而是使用以下方式声明：

    ```typescript
    export const Gender = {
        男: "male",
        女: "female",
    } as const

    export type Gender = (typeof Gender)[keyof typeof Gender]
    ```

- 在创建 `git` 提交记录，必须使用 `[emoji] [type]: 具体内容`的格式进行提交，具体内容使用中文，以下是预设的 `emoji` 和 `type`：

    ```text
    ✨ feature: Select when creating new things
    🐞 fix: Select when fixing a bug
    📝 docs: Select when editing documentation
    💻 wip: Select when work is not finished
    🚄 perfs: Select when working on performances
    ⏪ rollback:Select when undoing something
    🔵 other: Select when fixing a bug
    ```

    你必须使用预设的 `emoji` 和 `type`。如果你的提交记录包含了多种内容，你可以使用多行比如：

    ```text
    ✨ feature: some feature u did
    🐞 fix: some bug u fixed
    ```

- 除了 `React` 组件和页面以外所有的导出必须使用 `export` 关键字导出，不要使用 `export default` 关键字导出

- 当一个文件中需要导出多个 `React` 组件时，主组件必须使用 `export default` 关键字导出，其他组件必须使用 `export` 关键字导出

- 在你每次进行比较大的修改后，你必须使用 `tsc --noEmit` 和 `eslint` 检查代码，确保代码没有错误

- 当你执行了一个 `mutation` 类型的操作后，你必须参考项目中更新数据的逻辑，更新所有需要更新的数据，比如如果你使用的使用 `@tanstack/react-query` 中 `useMutation` 时，你应该在 `onSuccess` 回调中更新所有需要更新的数据：`context.client.invalidateQueries({ queryKey: ["query-book"] })`

- 如果你发现组件中使用 `messgae` 或者 `toast` 之类的提示方法并没有被导入，请不要自动导入，因为在我的大多数项目中，我都已经它们挂载在了全局对象上，可以直接使用，通常是在 `@/components/Registry.tsx` 中进行挂载，有且仅当 `tsc --noEmit` 检查出错误时，你才需要手动导入

### 函数声明

- **具名实体用 `function`**：凡是需要被导出（`export`）、在别处引用、或作为返回值（`return`）的具名函数，必须使用 `function` 关键字声明，并显式指定函数名。
- **匿名逻辑用箭头函数**：凡是不需要变量名的临时逻辑、回调函数、内联函数，优先使用箭头函数。
- **禁止**使用 `const add = (a, b) => ...` 这种形式定义顶层或具名工具函数。
- **禁止**在 `map`、`filter`、`setTimeout` 等回调中使用 `function` 匿名函数。

| 场景分类                 | 推荐声明形式                     | 示例                              |
| :----------------------- | :------------------------------- | :-------------------------------- |
| **顶层导出/工具函数**    | **必须**使用 `function`          | `export function formatData() {}` |
| **React 函数式组件**     | **优先**使用箭头函数 + `FC` 类型 | `const MyComp: FC<P> = () => {}`  |
| **自定义 Hooks**         | **必须**使用 `function`          | `export function useCustom() {}`  |
| **数组/异步回调**        | **必须**使用箭头函数             | `.map(item => item.id)`           |
| **高阶函数返回的函数**   | **必须**使用 `function` 并具名   | `return function resolver() {}`   |
| **组件内独立的 Handler** | **必须**使用 `function`          | `function onSubmit() {}`          |

## React Rules

### 规则

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

### 组件与页面

请遵循以下规则生成组件或页面，并在新增时考虑复用与抽取：

1. 先分析页面结构，识别重复的 UI 片段与逻辑，并判断是否值得抽取。不要为了抽取而抽取，优先考虑维护成本。
2. 抽取的组件应该放在公共的 `@/components` 目录下，工具函数应该放在公共的 `@/utils` 目录下，禁止放在其他目录下。
3. 新增组件或页面前，检查已有目录（尤其是 `@/components` 与 `@/utils`）是否已有可复用实现，优先复用而非重复创建。
4. 抽取时保持原有 UI 风格与交互一致，避免引入不必要的样式或行为变化。
5. 组件拆分要能提升可读性与可测试性；若拆分后跨文件沟通成本增加，则保留在原文件。
6. 对抽取出的组件与工具，提供清晰的 props 或函数签名与命名，便于后续维护与扩展。

## API Rules

当我将 api 文档发送给你时，请按照以下规则生成代码：

1. 使用 4 个空格进行缩进，尾部保留一个空行
2. 函数名与文件名保持一致
3. 如果 api 函数需要传递参数，请使用 `params` 作为参数名，请使用函数名 + Params 作为参数类型，比如 `async function queryUser(params: QueryUserParams)`
4. api 函数的请求方法始终使用 `@/utils/request` 中的 `request` 函数，它的使用方法与 `fetch` 大致一致
5. 如果 api 函数需要在 `body` 中传递参数，请直接将 `params` 传递给 `body`，不需要进行 `JSON.stringify`，不需要设置 `Content-Type` 为 `application/json`，不需要设置 `method` 为 `POST`，`request` 函数内部会自动处理
6. 如果 api 函数需要在 `query` 中传递参数，请直接将 `params` 传递给 `search` 属性，不需要进行其他处理
7. 如果 api 函数的参数是一个对象，并且这个对象的所有属性都是可选的，请给 `params` 后面添加 `= {}` 的默认值，比如 `async function queryUser(params: QueryUserParams = {})`
8. 如果 api 函数的返回值是一个分页数据，使用 `deepsea-tools` 中的 `Page` 泛型，它的泛型参数为每一项的类型，比如 `Page<User>`
9. 请将 api 函数的返回值的类型传递给 `request` 函数的泛型参数，比如 `const response = await request<Page<User>>("/user/query", { search: params })`
10. 如果 api 函数的返回值是一个对象，对于每一个属性，如果没有文档明确说明是可选的，请不要使用 `?` 将它标记为可选
11. 请不要直接返回 `request` 函数的返回值，而是先传递给 `response` 变量，然后返回 `response` 变量，比如：

    ```typescript
    const response = await request<Page<User>>("/user/query", { search: params })
    return response
    ```

12. 常规的 api 函数为 5 种类型，以 `User` 为例，它们分别是：
    - 查询用户列表，请求参数为 `QueryUserParams`，返回值为 `Page<User>`
    - 新增用户，请求参数为 `AddUserParams`，返回值为 `User`
    - 更新用户，请求参数为 `UpdateUserParams`，返回值为 `User`
    - 删除用户，请求参数为 `DeleteUserParams`，返回值为 `User`，请求方法为 `DELETE`
    - 获取用户详情，请求参数为 `User` 类型中的唯一标识符字段的类型，字段一般为 `id`，类型一般为 `string` | `number`，返回值为 `User`
13. 新增参数和更新参数一般与原类型高度一致，请尽可能复用原类型，以 `User` 为例，新增参数为 `AddUserParams`，更新参数为 `UpdateUserParams`，尽可能 `extends` 原类型中可以复用的属性，灵活使用 `Omit` 和 `Pick` 等工具类型：

    ```typescript
    // 你可以移除不需要的属性，比如 `id`
    interface AddUserParams extends Omit<User, "id" | "createdAt" | "updatedAt"> {}

    // 更新参数可能与原类型高度一致，也可能不完全一致，请灵活处理
    interface UpdateUserParams extends User {}
    ```

14. 如果某个类型的说明中体现了它是一个枚举类型，请使用枚举类型代替原先的 `string` 或者 `number` 类型，枚举类型的 `key` 使用它的中文说明，`value` 使用它的取值，`key` 的长度尽量保持一致，比如 api 文档中 `userStatus` 的属性是 `类型：int，说明：用户状态，取值：1-正常，0-禁用`，则可以这样定义：

    ```typescript
    export const UserStatus = {
        正常: 1,
        禁用: 0,
    } as const

    export type UserStatus = (typeof UserStatus)[keyof typeof UserStatus]

    export interface User {
        // ...
        // 使用 UserStatus 类型代替原先的 `string` 或者 `number` 类型
        status: UserStatus
        // ...
    }
    ```

15. 请尽可能地复用类型，比如你可能在 `addUser` 中需要使用 `User` 类型，在 `queryUser` 中已经定义了 `User` 类型，请直接使用 `User` 类型，而不是重新定义一个 `User` 类型，当然这只是比较简单的场景，有时间两个文件名关联都不一定很大，请灵活处理，枚举类型也是如此。
16. 当你完成 api 函数的代码后，请在 `@/hooks` 目录下生成一个相应的 hook 函数，它的文件名为 use + 函数名(首字母大写)，api 函数为查询和操作两种类型，生成的 hook 函数也有两种类型：
    - `query` 函数

        ```typescript
        import { createUseQuery } from "soda-tanstack-query"
        import { queryUser } from "@/apis/queryUser"

        export const useQueryUser = createUseQuery({
            queryFn: queryUser,
            // 这里的 key 为 api 函数的烤肉串命名
            queryKey: "query-user",
        })
        ```

    - `get` 函数

        ```typescript
        import { isNonNullable } from "deepsea-tools"
        import { createUseQuery } from "soda-tanstack-query"
        import { getUser } from "@/apis/getUser"

        export function getUserOptional(id?: string | undefined) {
            return isNonNullable(id) ? getUser(id) : null
        }

        export const useGetUser = createUseQuery({
            queryFn: getUserOptional,
            queryKey: "get-user",
        })
        ```

    - `add`、`update`、`delete` 等等操作函数

        ```typescript
        import { useMutation, UseMutationOptions } from "@tanstack/react-query"
        import { addUser } from "@/apis/addUser"

        // UseMutationOptions 的泛型参数为 api 函数的返回值类型、错误类型（默认 `Error`）、请求参数类型、上下文类型
        export interface UseAddUserParams<TOnMutateResult = unknown> extends Omit<
            UseMutationOptions<Awaited<ReturnType<typeof addUser>>, Error, Parameters<typeof addUser>[0], TOnMutateResult>,
            "mutationFn"
        > {}

        export function useAddUser<TOnMutateResult = unknown>({ onMutate, onSuccess, onError, onSettled, ...rest }: UseAddUserParams<TOnMutateResult> = {}) {
            const key = useId()

            return useMutation({
                mutationFn: addUser,
                onMutate(variables, context) {
                    message.open({
                        key,
                        type: "loading",
                        content: "新增用户中...",
                        duration: 0,
                    })

                    return onMutate?.(variables, context) as TOnMutateResult | Promise<TOnMutateResult>
                },
                onSuccess(data, variables, onMutateResult, context) {
                    // 成功后刷新 user 相关的 query
                    context.client.invalidateQueries({ queryKey: ["query-user"] })

                    context.client.invalidateQueries({ queryKey: ["get-user", data.id] })

                    message.open({
                        key,
                        type: "success",
                        content: "新增用户成功",
                    })

                    return onSuccess?.(data, variables, onMutateResult, context)
                },
                onError(error, variables, onMutateResult, context) {
                    // 失败后关闭 loading
                    message.destroy(key)

                    return onError?.(error, variables, onMutateResult, context)
                },
                onSettled(data, error, variables, onMutateResult, context) {
                    return onSettled?.(data, error, variables, onMutateResult, context)
                },
                ...rest,
            })
        }
        ```

## Next Rules

针对 `Next.js` 16 项目的规则

### server action

如果你需要创建一个 `server action`，例如 `addUser`，你应该按照以下规则创建：

1. 在 `@/schemas` 目录下创建一个名为 `addUser.ts` 的文件，它的内容应该如下：

    ```typescript
    import { getParser } from "."
    import { z } from "zod"

    import { phoneSchema } from "./phone"

    import { roleSchema } from "./role"

    import { usernameSchema } from "./username"

    export const addUserSchema = z.object(
        {
            username: usernameSchema,
            phone: phoneSchema,
            role: roleSchema,
        },
        { message: "无效的用户参数" },
    )

    export type AddUserParams = z.infer<typeof addUserSchema>

    export const addUserParser = getParser(addUserSchema)
    ```

2. 在 `@/shared` 目录下创建一个名为 `addUser.ts` 的文件，它的内容应该如下：

    ```typescript
    import { prisma } from "@/prisma"
    import { User } from "@/prisma/generated/client"
    import { AddUserParams } from "@/schemas/addUser"
    import { ClientError } from "@/utils/clientError"

    export async function addUser({ username, phone }: AddUserParams) {
        const count = await prisma.user.count({ where: { username } })

        // 如果函数内部需要抛出错误，请使用 `ClientError` 类，它的使用方法与 `Error` 类一致，同时支持更多用法，详细请参考 `@/utils/clientError` 文件
        if (count > 0) throw new ClientError("用户名已存在")

        const count2 = await prisma.user.count({ where: { phone } })
        if (count2 > 0) throw new ClientError("手机号已存在")
        const user = await prisma.user.create({ data: { username, phone } })
        return user
    }

    // 如果这个 server action 只允许特定的用户指定，可以为 @/shared 目录下的响应函数添加一个 `filter` 属性
    // `filter` 属性接受两种类型，一种是 `boolean` 类型，一种是 `(user: User) => boolean` 函数类型
    // 当你传入 `boolean` 类型时， `true` 代表只有登录且未被禁用的用户才能访问，`false` 代表不做任何限制，包括未登录用户
    // 当你传入 `(user: User) => boolean` 函数类型时，函数返回 `true` 代表用户可以访问，返回 `false` 代表用户不能访问
    // filter 属性默认值为 `true`，代表只有登录且未被禁用的用户才能访问
    addUser.filter = function filter(user: User) {
        return user.role === "ADMIN"
    }
    ```

3. 在 `@/actions` 目录下创建一个名为 `addUser.ts` 的文件，它的内容应该如下：

    ```typescript
    "use server"

    import { addUserSchema } from "@/schemas/addUser"
    import { createResponseFn } from "@/server/createResponseFn"
    import { addUser } from "@/shared/addUser"

    export const addUserAction = createResponseFn({
        fn: addUser,
        schema: addUserSchema,
        name: "addUser",
    })
    ```

4. 在 `@/presets` 目录下创建一个名为 `createUseAddUser.ts` 的文件，它的内容应该如下：

    ```typescript
    import { useId } from "react"

    import { withUseMutationDefaults } from "soda-tanstack-query"
    import { addUser } from "@/shared/addUser"

    export const createUseAddUser = withUseMutationDefaults<typeof addUser>(() => {
        const key = useId()
        return {
            onMutate(variables, context) {
                message.open({
                    key,
                    type: "loading",
                    content: "新增用户中...",
                    duration: 0,
                })
            },
            onSuccess(data, variables, onMutateResult, context) {
                // 请在此刷新其他需要刷新的关联的 query
                context.client.invalidateQueries({ queryKey: ["query-user"] })

                context.client.invalidateQueries({ queryKey: ["get-user", data.id] })

                message.open({
                    key,
                    type: "success",
                    content: "新增用户成功",
                })
            },
            onError(error, variables, onMutateResult, context) {
                message.destroy(key)
            },
            onSettled(data, error, variables, onMutateResult, context) {},
        }
    })
    ```

5. 在 `@/hooks` 目录下创建一个名为 `useAddUser.ts` 的文件，它的内容应该如下：

    ```typescript
    import { createRequestFn } from "deepsea-tools"
    import { addUserAction } from "@/actions/addUser"
    import { createUseAddUser } from "@/presets/createUseAddUser"

    export const addUserClient = createRequestFn(addUserAction)

    export const useAddUser = createUseAddUser(addUserClient)
    ```

### Schema

当你需要创建一个 `schema` 时，如果是一个对象或者数组，你应该将它们独立出来作为一个文件，而不是直接在 `schema` 中定义，例如：

```typescript
import { getParser } from "."
import { z } from "zod"

export const addUserSchema = z.object(
    {
        username: z
            .string({ message: "无效的用户名" })
            .min(4, { message: "用户名长度不能低于 4 位" })
            .max(16, { message: "用户名长度不能超过 16 位" })
            .regex(/^[a-zA-Z0-9_]+$/, { message: "用户名只能包含字母、数字和下划线" })
            .regex(/^[a-zA-Z]/, { message: "用户名必须以字母开头" }),
        phone: z.string({ message: "无效的手机号" }).regex(phoneRegex, { message: "无效的手机号" }),
    },
    { message: "无效的用户参数" },
)

export type AddUserParams = z.infer<typeof addUserSchema>

export const addUserParser = getParser(addUserSchema)
```

你应该将 `usernameSchema` 和 `phoneSchema` 独立出来成为两个独立的文件，便于复用，而不是直接在 `schema` 中定义，例如：

```typescript
import { getParser } from "."
import { z } from "zod"

export const usernameSchema = z
    .string({ message: "无效的用户名" })
    .min(4, { message: "用户名长度不能低于 4 位" })
    .max(16, { message: "用户名长度不能超过 16 位" })
    .regex(/^[a-zA-Z0-9_]+$/, { message: "用户名只能包含字母、数字和下划线" })
    .regex(/^[a-zA-Z]/, { message: "用户名必须以字母开头" })

export type UsernameParams = z.infer<typeof usernameSchema>

export const usernameParser = getParser(usernameSchema)
```

### Utils

`@/utils` 目录下的文件必须只能是客户端可以访问的工具函数，或者客户端和服务器都可以访问的工具函数，如果一个工具函数只能在服务器访问，或者不能暴露给客户端，请将它放在 `@/server` 目录下

### Middleware

在 `Next.js` 16 中，`middleware.ts` 已经被弃用，请使用 `proxy.ts` 文件来实现中间件功能，并且导出的函数名必须为 `proxy`，例如：

```typescript
import { NextRequest, NextResponse } from "next/server"

export async function proxy(request: NextRequest) {
    const requestHeaders = new Headers(request.headers)
    requestHeaders.set("current-url", request.url)

    const response = NextResponse.next({
        request: {
            headers: requestHeaders,
        },
    })

    return response
}
```

### Api Route

- 只有当你的某个功能必须通过 `HTTP` 接口的方式才能实现时，比如需要允许第三方调用的接口或者 `server action` 无法满足需求时，你才需要创建一个 `api route`，否则你应该创建一个 `server action`
- 你依然需要遵守 `server action` 的规则，核心的实现逻辑与 `server action` 一致，比如 `schema` （负责校验数据）的创建、`shared` 函数（负责核心逻辑）的创建等，这些规则同样适用于 `api route`
- 所以 `api route` 的创建规则与 `server action` 的创建规则一致，你只需要按照 `server action` 的创建规则创建即可，唯一不同的地方是，你需要将 `api route` 按照 Next.js 的 `api route` 规则创建，而不是 `@/actions` 目录下
- 当然如果这个功能可以通过 `server action` 实现并且是给内部使用的，你也可以同时按照 `server action` 的创建规则创建一个 `server action`，这样 `server action` 和 `api route` 都可以使用，但是 `server action` 是给内部使用的，而 `api route` 是给外部使用的

---
> Source: [1adybug/docker-management](https://github.com/1adybug/docker-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
