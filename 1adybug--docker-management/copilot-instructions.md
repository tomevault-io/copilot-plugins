## next

> 针对 `Next.js` 16 项目的规则


# Next Rules

针对 `Next.js` 16 项目的规则

## server action

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

## Schema

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

## Utils

`@/utils` 目录下的文件必须只能是客户端可以访问的工具函数，或者客户端和服务器都可以访问的工具函数，如果一个工具函数只能在服务器访问，或者不能暴露给客户端，请将它放在 `@/server` 目录下

## Middleware

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

## Api Route

- 只有当你的某个功能必须通过 `HTTP` 接口的方式才能实现时，比如需要允许第三方调用的接口或者 `server action` 无法满足需求时，你才需要创建一个 `api route`，否则你应该创建一个 `server action`
- 你依然需要遵守 `server action` 的规则，核心的实现逻辑与 `server action` 一致，比如 `schema` （负责校验数据）的创建、`shared` 函数（负责核心逻辑）的创建等，这些规则同样适用于 `api route`
- 所以 `api route` 的创建规则与 `server action` 的创建规则一致，你只需要按照 `server action` 的创建规则创建即可，唯一不同的地方是，你需要将 `api route` 按照 Next.js 的 `api route` 规则创建，而不是 `@/actions` 目录下
- 当然如果这个功能可以通过 `server action` 实现并且是给内部使用的，你也可以同时按照 `server action` 的创建规则创建一个 `server action`，这样 `server action` 和 `api route` 都可以使用，但是 `server action` 是给内部使用的，而 `api route` 是给外部使用的

---
> Source: [1adybug/docker-management](https://github.com/1adybug/docker-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
