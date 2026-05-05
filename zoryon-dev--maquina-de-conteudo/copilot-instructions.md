## api-routes

> Padrões e convenções para API Routes do Next.js


# Padrões de API Routes

## Estrutura

### Localização
- `src/app/api/[route]/route.ts` - Rotas principais
- `src/app/api/[route]/[id]/route.ts` - Rotas com parâmetros

### Métodos HTTP Suportados
- `GET` - Buscar dados
- `POST` - Criar recursos ou ações
- `PUT` / `PATCH` - Atualizar recursos
- `DELETE` - Deletar recursos

## Autenticação

### Padrão com Clerk

```typescript
import { auth } from "@clerk/nextjs/server"
import { NextResponse } from "next/server"

export async function GET(request: Request) {
  const { userId } = await auth()
  
  if (!userId) {
    return NextResponse.json(
      { error: "Unauthorized" },
      { status: 401 }
    )
  }
  
  // Lógica da rota...
}
```

### Autenticação por Secret (Workers)

```typescript
const WORKER_SECRET = process.env.WORKER_SECRET

export async function POST(request: Request) {
  const authHeader = request.headers.get("authorization")
  const secret = authHeader?.replace("Bearer ", "")
  
  if (secret !== WORKER_SECRET) {
    return NextResponse.json(
      { error: "Unauthorized" },
      { status: 401 }
    )
  }
  
  // Lógica da rota...
}
```

## Respostas

### Padrão de Resposta

```typescript
import { NextResponse } from "next/server"

// Sucesso
return NextResponse.json({
  data: result,
  message: "Success"
})

// Erro
return NextResponse.json(
  { error: "Error message" },
  { status: 400 }
)
```

### Status Codes

- `200` - Sucesso (GET, PUT, PATCH)
- `201` - Criado com sucesso (POST)
- `400` - Bad Request (validação, dados inválidos)
- `401` - Unauthorized (não autenticado)
- `403` - Forbidden (sem permissão)
- `404` - Not Found (recurso não existe)
- `500` - Internal Server Error (erro do servidor)

## Validação de Input

### Padrão

```typescript
export async function POST(request: Request) {
  try {
    const body = await request.json()
    const { type, payload } = body as {
      type: JobType
      payload: JobPayload
    }
    
    // Validação
    if (!type || !payload) {
      return NextResponse.json(
        { error: "Missing required fields: type, payload" },
        { status: 400 }
      )
    }
    
    // Processar...
  } catch (error) {
    console.error("Error:", error)
    return NextResponse.json(
      { error: "Failed to process request" },
      { status: 500 }
    )
  }
}
```

### Query Parameters

```typescript
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const limit = parseInt(searchParams.get("limit") || "20", 10)
  const offset = parseInt(searchParams.get("offset") || "0", 10)
  
  // Validar
  if (limit < 1 || limit > 100) {
    return NextResponse.json(
      { error: "Limit must be between 1 and 100" },
      { status: 400 }
    )
  }
  
  // Processar...
}
```

## Rotas Existentes

### `/api/jobs`

#### POST - Criar Job
```typescript
// Body: { type, payload, priority?, scheduledFor? }
// Response: { jobId, status, message }
```

#### GET - Listar Jobs
```typescript
// Query: ?limit=20&offset=0
// Response: { jobs: [], pagination: { limit, offset } }
```

### `/api/jobs/[id]`

#### GET - Detalhes do Job
```typescript
// Response: { job: {...} }
```

#### DELETE - Cancelar Job
```typescript
// Response: { message: "Job cancelled" }
```

### `/api/workers`

#### POST - Processar Job
```typescript
// Auth: Bearer token (WORKER_SECRET) ou Clerk auth
// Response: { processed: true, jobId, result? }
```

### `/api/webhooks/clerk`

#### POST - Webhook do Clerk
```typescript
// Headers: svix-signature, svix-timestamp, svix-id
// Body: Event payload do Clerk
// Events: user.created, user.updated, user.deleted
```

## Error Handling

### Padrão de Try-Catch

```typescript
export async function POST(request: Request) {
  try {
    // Lógica principal
    const result = await processRequest()
    return NextResponse.json({ success: true, data: result })
  } catch (error) {
    console.error("Error in POST /api/route:", error)
    
    // Erro conhecido
    if (error instanceof ValidationError) {
      return NextResponse.json(
        { error: error.message },
        { status: 400 }
      )
    }
    
    // Erro genérico
    return NextResponse.json(
      { error: "Failed to process request" },
      { status: 500 }
    )
  }
}
```

### Logging

```typescript
// Log de sucesso (opcional)
console.log(`Job ${jobId} created successfully`)

// Log de erro (sempre)
console.error("Error creating job:", error)
```

## TypeScript

### Tipos de Request/Response

```typescript
// Request body type
interface CreateJobRequest {
  type: JobType
  payload: JobPayload
  priority?: number
  scheduledFor?: string
}

// Response type
interface CreateJobResponse {
  jobId: number
  status: "pending"
  message: string
}

export async function POST(request: Request) {
  const body = await request.json() as CreateJobRequest
  // ...
  return NextResponse.json({...} as CreateJobResponse)
}
```

## CORS (se necessário)

```typescript
export async function OPTIONS(request: Request) {
  return new NextResponse(null, {
    status: 200,
    headers: {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE",
      "Access-Control-Allow-Headers": "Content-Type, Authorization",
    },
  })
}
```

## Rate Limiting (futuro)

Implementar rate limiting quando necessário:
- Por usuário (Clerk userId)
- Por IP
- Por endpoint

## Webhooks

### Verificação de Assinatura (Clerk)

```typescript
import { Webhook } from "svix"

const webhook = new Webhook(process.env.CLERK_WEBHOOK_SECRET!)

export async function POST(request: Request) {
  const headers = Object.fromEntries(request.headers.entries())
  const payload = await request.text()
  
  try {
    const event = webhook.verify(payload, headers) as ClerkWebhookEvent
    // Processar evento...
  } catch (error) {
    return NextResponse.json(
      { error: "Invalid signature" },
      { status: 400 }
    )
  }
}
```

---
> Source: [zoryon-dev/maquina-de-conteudo](https://github.com/zoryon-dev/maquina-de-conteudo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
