## api

> 当我将 api 文档发送给你时，请按照以下规则生成代码：


# API Rules

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

---
> Source: [1adybug/docker-management](https://github.com/1adybug/docker-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
