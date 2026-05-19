## base

> - 禁止修改 `node_modules` 文件夹中的任何文件


# Base Rules

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

## 函数声明

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

---
> Source: [1adybug/docker-management](https://github.com/1adybug/docker-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
