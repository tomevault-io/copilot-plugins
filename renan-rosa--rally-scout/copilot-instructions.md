## rally-scout

> - **Framework:** Next.js 16 (App Router)

# Padrões de Código e Arquitetura

## Stack

- **Framework:** Next.js 16 (App Router)
- **Banco:** Neon DB (PostgreSQL serverless)
- **ORM:** Prisma 6+
- **UI:** shadcn/ui + Tailwind CSS
- **Validação:** Zod
- **Auth:** JWT (jose) + httpOnly cookies
- **Forms:** react-hook-form + @hookform/resolvers

---

## Estrutura de Pastas

```
src/
├── app/
│   ├── (auth)/                    # Grupo público (sign-in, sign-up)
│   │   ├── sign-in/page.tsx
│   │   └── sign-up/page.tsx
│   │
│   ├── (dashboard)/               # Grupo protegido (requer auth)
│   │   ├── layout.tsx             # Layout com sidebar
│   │   ├── page.tsx               # /dashboard
│   │   ├── teams/
│   │   │   ├── page.tsx           # Lista times
│   │   │   ├── new/page.tsx       # Criar time
│   │   │   └── [teamId]/
│   │   │       ├── page.tsx       # Detalhes time
│   │   │       └── players/
│   │   │           └── page.tsx   # Jogadores do time
│   │   ├── matches/
│   │   │   ├── page.tsx           # Lista partidas
│   │   │   ├── new/page.tsx       # Criar partida
│   │   │   └── [matchId]/
│   │   │       ├── page.tsx       # Detalhes/Stats
│   │   │       └── scout/page.tsx # Scout LIVE
│   │   └── stats/
│   │       └── page.tsx           # Relatórios
│   │
│   ├── layout.tsx                 # Root layout
│   └── page.tsx                   # Landing (redireciona)
│
├── actions/                       # Server Actions
│   ├── schemas/                   # Zod schemas (co-localizados)
│   │   ├── auth.ts
│   │   ├── teams.ts
│   │   ├── players.ts
│   │   ├── matches.ts
│   │   └── scout.ts
│   ├── auth.ts
│   ├── teams.ts
│   ├── players.ts
│   ├── matches.ts
│   └── scout.ts
│
├── components/
│   ├── ui/                        # shadcn (não editar)
│   ├── features/                  # Por domínio
│   │   ├── auth/
│   │   ├── teams/
│   │   ├── players/
│   │   ├── matches/
│   │   └── scout/
│   └── shared/                    # Reutilizáveis (Header, Sidebar, etc)
│
├── hooks/                         # Custom hooks
│   ├── use-auth.ts
│   └── use-scout.ts
│
├── lib/                           # Utilitários core
│   ├── prisma.ts                  # Singleton Prisma
│   ├── auth.ts                    # JWT helpers (server)
│   ├── auth-edge.ts               # JWT helpers (edge/proxy)
│   ├── utils.ts                   # cn() e helpers gerais
│   └── volleyball.ts              # Constantes do domínio
│
├── generated/                     # Prisma Client (auto-gerado)
│   └── prisma/
│
├── proxy.ts                       # Middleware (Next 16 = proxy)
└── env.ts                         # Validação de env vars
```

---

## Padrão de Server Actions

### 1. Schema (Zod)

**`src/actions/schemas/teams.ts`**
```typescript
import { z } from "zod"

// ══════════════════════════════════════════════════════════════
// CREATE
// ══════════════════════════════════════════════════════════════
export const createTeamSchema = z.object({
  name: z
    .string()
    .min(1, "Nome é obrigatório")
    .min(2, "Nome deve ter no mínimo 2 caracteres")
    .max(100, "Nome deve ter no máximo 100 caracteres"),
})

export type CreateTeamInput = z.infer<typeof createTeamSchema>

// ══════════════════════════════════════════════════════════════
// UPDATE
// ══════════════════════════════════════════════════════════════
export const updateTeamSchema = z.object({
  id: z.string().cuid(),
  name: z
    .string()
    .min(1, "Nome é obrigatório")
    .min(2, "Nome deve ter no mínimo 2 caracteres")
    .max(100, "Nome deve ter no máximo 100 caracteres"),
})

export type UpdateTeamInput = z.infer<typeof updateTeamSchema>

// ══════════════════════════════════════════════════════════════
// DELETE
// ══════════════════════════════════════════════════════════════
export const deleteTeamSchema = z.object({
  id: z.uuid(),
})

export type DeleteTeamInput = z.infer<typeof deleteTeamSchema>
```

---

### 2. Server Action

**`src/actions/teams.ts`**
```typescript
"use server"

import { revalidatePath } from "next/cache"
import { requireAuth } from "@/lib/auth"
import prisma from "@/lib/prisma"
import {
  createTeamSchema,
  updateTeamSchema,
  deleteTeamSchema,
  type CreateTeamInput,
  type UpdateTeamInput,
  type DeleteTeamInput,
} from "./schemas/teams"

// ══════════════════════════════════════════════════════════════
// TYPES
// ══════════════════════════════════════════════════════════════
export type ActionResponse<T = void> = {
  success: boolean
  data?: T
  error?: string
}

// ══════════════════════════════════════════════════════════════
// CREATE
// ══════════════════════════════════════════════════════════════
export async function createTeam(
  input: CreateTeamInput
): Promise<ActionResponse<{ id: string }>> {
  try {
    // 1. Auth
    const user = await requireAuth()

    // 2. Validação
    const parsed = createTeamSchema.safeParse(input)
    if (!parsed.success) {
      return { success: false, error: parsed.error.issues[0].message }
    }

    // 3. Execução
    const team = await prisma.team.create({
      data: {
        name: parsed.data.name,
        userId: user.id,
      },
    })

    // 4. Revalidação
    revalidatePath("/dashboard/teams")

    // 5. Retorno
    return { success: true, data: { id: team.id } }
  } catch (error) {
    console.error("[CREATE_TEAM]", error)
    return { success: false, error: "Erro ao criar time" }
  }
}

// ══════════════════════════════════════════════════════════════
// READ (Lista)
// ══════════════════════════════════════════════════════════════
export async function getTeams() {
  try {
    const user = await requireAuth()

    const teams = await prisma.team.findMany({
      where: { userId: user.id },
      include: {
        _count: {
          select: { players: true, matches: true },
        },
      },
      orderBy: { createdAt: "desc" },
    })

    return { success: true, data: teams }
  } catch (error) {
    console.error("[GET_TEAMS]", error)
    return { success: false, error: "Erro ao buscar times" }
  }
}

// ══════════════════════════════════════════════════════════════
// READ (Único)
// ══════════════════════════════════════════════════════════════
export async function getTeam(id: string) {
  try {
    const user = await requireAuth()

    const team = await prisma.team.findFirst({
      where: { id, userId: user.id },
      include: {
        players: {
          where: { isActive: true },
          orderBy: { number: "asc" },
        },
        _count: {
          select: { matches: true },
        },
      },
    })

    if (!team) {
      return { success: false, error: "Time não encontrado" }
    }

    return { success: true, data: team }
  } catch (error) {
    console.error("[GET_TEAM]", error)
    return { success: false, error: "Erro ao buscar time" }
  }
}

// ══════════════════════════════════════════════════════════════
// UPDATE
// ══════════════════════════════════════════════════════════════
export async function updateTeam(
  input: UpdateTeamInput
): Promise<ActionResponse> {
  try {
    const user = await requireAuth()

    const parsed = updateTeamSchema.safeParse(input)
    if (!parsed.success) {
      return { success: false, error: parsed.error.issues[0].message }
    }

    // Verifica ownership
    const existing = await prisma.team.findFirst({
      where: { id: parsed.data.id, userId: user.id },
    })

    if (!existing) {
      return { success: false, error: "Time não encontrado" }
    }

    await prisma.team.update({
      where: { id: parsed.data.id },
      data: { name: parsed.data.name },
    })

    revalidatePath("/dashboard/teams")
    revalidatePath(`/dashboard/teams/${parsed.data.id}`)

    return { success: true }
  } catch (error) {
    console.error("[UPDATE_TEAM]", error)
    return { success: false, error: "Erro ao atualizar time" }
  }
}

// ══════════════════════════════════════════════════════════════
// DELETE
// ══════════════════════════════════════════════════════════════
export async function deleteTeam(
  input: DeleteTeamInput
): Promise<ActionResponse> {
  try {
    const user = await requireAuth()

    const parsed = deleteTeamSchema.safeParse(input)
    if (!parsed.success) {
      return { success: false, error: parsed.error.issues[0].message }
    }

    // Verifica ownership
    const existing = await prisma.team.findFirst({
      where: { id: parsed.data.id, userId: user.id },
    })

    if (!existing) {
      return { success: false, error: "Time não encontrado" }
    }

    // Cascade delete (players, matches, actions)
    await prisma.team.delete({
      where: { id: parsed.data.id },
    })

    revalidatePath("/dashboard/teams")

    return { success: true }
  } catch (error) {
    console.error("[DELETE_TEAM]", error)
    return { success: false, error: "Erro ao deletar time" }
  }
}
```

---

### 3. Hook (Client)

**`src/hooks/use-teams.ts`**
```typescript
"use client"

import { useRouter } from "next/navigation"
import { useTransition } from "react"
import {
  createTeam,
  updateTeam,
  deleteTeam,
} from "@/actions/teams"
import type {
  CreateTeamInput,
  UpdateTeamInput,
  DeleteTeamInput,
} from "@/actions/schemas/teams"

export function useTeams() {
  const router = useRouter()
  const [isPending, startTransition] = useTransition()

  const handleCreate = async (data: CreateTeamInput) => {
    const result = await createTeam(data)
    if (result.success && result.data) {
      startTransition(() => {
        router.push(`/dashboard/teams/${result.data.id}`)
        router.refresh()
      })
    }
    return result
  }

  const handleUpdate = async (data: UpdateTeamInput) => {
    const result = await updateTeam(data)
    if (result.success) {
      startTransition(() => {
        router.refresh()
      })
    }
    return result
  }

  const handleDelete = async (data: DeleteTeamInput) => {
    const result = await deleteTeam(data)
    if (result.success) {
      startTransition(() => {
        router.push("/dashboard/teams")
        router.refresh()
      })
    }
    return result
  }

  return {
    createTeam: handleCreate,
    updateTeam: handleUpdate,
    deleteTeam: handleDelete,
    isPending,
  }
}
```

---

### 4. Form Component

**`src/components/features/teams/team-form.tsx`**
```typescript
"use client"

import { useState } from "react"
import { useForm } from "react-hook-form"
import { zodResolver } from "@hookform/resolvers/zod"
import { Loader2 } from "lucide-react"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import {
  Field,
  FieldLabel,
  FieldError,
  FieldGroup,
} from "@/components/ui/field"
import { useTeams } from "@/hooks/use-teams"
import {
  createTeamSchema,
  type CreateTeamInput,
} from "@/actions/schemas/teams"

export function TeamForm() {
  const { createTeam, isPending } = useTeams()
  const [error, setError] = useState<string | null>(null)

  const form = useForm<CreateTeamInput>({
    resolver: zodResolver(createTeamSchema),
    defaultValues: {
      name: "",
    },
  })

  const onSubmit = async (data: CreateTeamInput) => {
    setError(null)
    const result = await createTeam(data)
    if (!result.success) {
      setError(result.error ?? "Erro ao criar time")
    }
  }

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <FieldGroup>
        {error && (
          <div className="rounded-md bg-destructive/10 p-3 text-sm text-destructive">
            {error}
          </div>
        )}

        <Field>
          <FieldLabel htmlFor="name">Nome do Time</FieldLabel>
          <Input
            id="name"
            placeholder="Ex: Vôlei Clube"
            disabled={isPending}
            {...form.register("name")}
          />
          {form.formState.errors.name && (
            <FieldError>{form.formState.errors.name.message}</FieldError>
          )}
        </Field>

        <Button type="submit" disabled={isPending}>
          {isPending ? (
            <>
              <Loader2 className="mr-2 size-4 animate-spin" />
              Criando...
            </>
          ) : (
            "Criar Time"
          )}
        </Button>
      </FieldGroup>
    </form>
  )
}
```

---

### 5. Page (Server Component)

**`src/app/(dashboard)/teams/page.tsx`**
```typescript
import { getTeams } from "@/actions/teams"
import { TeamsList } from "@/components/features/teams/teams-list"

export default async function TeamsPage() {
  const result = await getTeams()

  if (!result.success) {
    return <div>Erro ao carregar times</div>
  }

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">Meus Times</h1>
        <a href="/dashboard/teams/new">
          <Button>Novo Time</Button>
        </a>
      </div>

      <TeamsList teams={result.data} />
    </div>
  )
}
```

---

## Regras Gerais

### Server Actions

1. **Sempre** começar com `"use server"`
2. **Sempre** validar com Zod antes de executar
3. **Sempre** verificar auth com `requireAuth()`
4. **Sempre** verificar ownership (userId) antes de update/delete
5. **Sempre** usar try/catch e retornar `ActionResponse`
6. **Sempre** chamar `revalidatePath()` após mutations

### Schemas

1. Co-localizados em `src/actions/schemas/`
2. Exportar schema + type inferido
3. Mensagens de erro em português

### Components

1. Forms são Client Components (`"use client"`)
2. Listas podem ser Server Components
3. Usar hooks para encapsular lógica de mutations

### Nomenclatura

| Tipo | Padrão | Exemplo |
|------|--------|---------|
| Schema | `camelCaseSchema` | `createTeamSchema` |
| Type | `PascalCaseInput` | `CreateTeamInput` |
| Action | `camelCase` | `createTeam` |
| Hook | `use-kebab-case.ts` | `use-teams.ts` |
| Component | `PascalCase` | `TeamForm` |

---

## Fluxo Completo

```
1. User preenche form
2. react-hook-form valida com Zod (client)
3. onSubmit chama hook
4. Hook chama Server Action
5. Server Action:
   a. requireAuth()
   b. Zod safeParse (server)
   c. Verifica ownership
   d. Prisma mutation
   e. revalidatePath()
   f. Retorna ActionResponse
6. Hook processa resultado
7. Redirect ou mostra erro
```

---

---
> Source: [Renan-Rosa/rally-scout](https://github.com/Renan-Rosa/rally-scout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
