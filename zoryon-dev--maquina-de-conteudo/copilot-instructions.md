## queue-system

> Padrões e convenções para o sistema de filas (background jobs)


# Sistema de Filas (Background Jobs)

## Arquitetura

### Stack
- **Queue**: Upstash Redis (HTTP REST API)
- **Persistência**: PostgreSQL (tabela `jobs`)
- **Workers**: API Routes (`/api/workers`) - Serverless-friendly

### Por que Serverless?
- Não requer worker process contínuo
- Escala automaticamente
- Free tier generoso (500K requests/dia)
- Zero infraestrutura adicional

## Estrutura de Arquivos

```
src/lib/queue/
├── types.ts      # JobType, JobStatus, JobPayload types
├── client.ts     # Redis client (enqueue, dequeue)
└── jobs.ts       # CRUD de jobs (createJob, updateJobStatus)
```

## Tipos de Jobs

### JobType Enum

```typescript
type JobType =
  | "ai_text_generation"      // Geração de texto com IA
  | "ai_image_generation"      // Geração de imagem com IA
  | "carousel_creation"        // Criação de carrossel
  | "scheduled_publish"        // Publicação agendada
  | "web_scraping"             // Web scraping
```

### JobStatus Enum

```typescript
type JobStatus =
  | "pending"      // Aguardando processamento
  | "processing"   // Em processamento
  | "completed"    // Concluído com sucesso
  | "failed"       // Falhou após tentativas
```

### JobPayload Types

```typescript
// Definidos em src/lib/queue/types.ts
interface AITextGenerationPayload {
  prompt: string
  model: string
  // ...
}

interface AIImageGenerationPayload {
  prompt: string
  model: string
  // ...
}
```

## Fluxo de Processamento

### 1. Criação de Job

```typescript
// Client → POST /api/jobs
import { createJob } from "@/lib/queue/jobs"

const jobId = await createJob(userId, "ai_text_generation", {
  prompt: "Write a blog post about...",
  model: "openai/gpt-5.2"
}, {
  priority: 10, // Opcional
  scheduledFor: new Date("2024-01-20") // Opcional
})
```

**O que acontece:**
1. Insere job no banco (status: `pending`)
2. Se não agendado, enfileira no Redis (`enqueueJob`)
3. Retorna `jobId`

### 2. Enfileiramento (Redis)

```typescript
// src/lib/queue/client.ts
export async function enqueueJob(jobId: number, priority = 0) {
  // Formato: priority:timestamp:jobId
  const score = `${String(999999 - priority).padStart(6, "0")}:${Date.now()}:${jobId}`
  await redis.lpush(JOB_QUEUE, score)
}
```

**Chaves Redis:**
- `jobs:pending` - Fila de jobs pendentes
- `jobs:processing` - Jobs em processamento

### 3. Processamento (Worker)

```typescript
// Agendador/Cron → POST /api/workers
import { dequeueJob } from "@/lib/queue/client"
import { getJob, updateJobStatus } from "@/lib/queue/jobs"

// 1. Desenfileirar
const jobId = await dequeueJob()

// 2. Buscar job no banco
const job = await getJob(jobId)

// 3. Marcar como processando
await markAsProcessing(jobId)
await updateJobStatus(jobId, "processing")

// 4. Processar (handlers por tipo)
const handler = jobHandlers[job.type]
const result = await handler(job.payload)

// 5. Atualizar status
await updateJobStatus(jobId, "completed", { result })
await removeFromProcessing(jobId)
```

### 4. Handlers de Jobs

```typescript
// src/app/api/workers/route.ts
const jobHandlers: Record<JobType, (payload: JobPayload) => Promise<unknown>> = {
  ai_text_generation: async (payload) => {
    // Chamar OpenRouter API
    // Retornar texto gerado
  },
  ai_image_generation: async (payload) => {
    // Chamar OpenRouter Image API
    // Retornar URL da imagem
  },
  // ...
}
```

## Funções Principais

### Client (Redis)

```typescript
// src/lib/queue/client.ts

// Enfileirar job
enqueueJob(jobId: number, priority?: number): Promise<void>

// Desenfileirar próximo job
dequeueJob(): Promise<number | null>

// Marcar como processando
markAsProcessing(jobId: number): Promise<void>

// Remover de processamento
removeFromProcessing(jobId: number): Promise<void>

// Tamanho da fila
getQueueSize(): Promise<number>

// Jobs em processamento
getProcessingCount(): Promise<number>
```

### Jobs (Database)

```typescript
// src/lib/queue/jobs.ts

// Criar job
createJob<T extends JobPayload>(
  userId: string,
  type: JobType,
  payload: T,
  options?: {
    priority?: number
    scheduledFor?: Date
    maxAttempts?: number
  }
): Promise<number>

// Buscar job
getJob(jobId: number): Promise<Job | null>

// Atualizar status
updateJobStatus(
  jobId: number,
  status: "processing" | "completed" | "failed",
  data?: { result?: unknown; error?: string }
): Promise<void>

// Listar jobs do usuário
listUserJobs(
  userId: string,
  options?: { limit?: number; offset?: number; status?: JobStatus }
): Promise<Job[]>
```

## Retry Logic

### Configuração

```typescript
// Ao criar job
createJob(userId, type, payload, {
  maxAttempts: 3 // Padrão: 3
})

// No worker
if (job.attempts < job.maxAttempts) {
  // Re-enfileirar com prioridade reduzida
  await enqueueJob(jobId, job.priority - 1)
  await updateJobStatus(jobId, "pending", {
    attempts: job.attempts + 1
  })
} else {
  // Marcar como failed
  await updateJobStatus(jobId, "failed", {
    error: "Max attempts reached"
  })
}
```

## Agendamento

### Jobs Agendados

```typescript
// Criar job agendado
const jobId = await createJob(userId, type, payload, {
  scheduledFor: new Date("2024-01-20T10:00:00Z")
})

// Não enfileira imediatamente
// Worker verifica jobs agendados antes de processar
```

### Verificação de Agendamento

```typescript
// No worker, antes de processar:
const scheduledJobs = await db
  .select()
  .from(jobs)
  .where(
    and(
      eq(jobs.status, "pending"),
      lte(jobs.scheduledFor, new Date())
    )
  )

// Enfileirar jobs agendados que já passaram
for (const job of scheduledJobs) {
  await enqueueJob(job.id, job.priority)
}
```

## Monitoramento

### Métricas Úteis

```typescript
// Tamanho da fila
const queueSize = await getQueueSize()

// Jobs em processamento
const processingCount = await getProcessingCount()

// Jobs por status
const pendingJobs = await db
  .select({ count: sql<number>`count(*)` })
  .from(jobs)
  .where(eq(jobs.status, "pending"))
```

## Variáveis de Ambiente

```env
UPSTASH_REDIS_REST_URL=https://...
UPSTASH_REDIS_REST_TOKEN=...
WORKER_SECRET=... # Para autenticação de workers
```

## Padrões de Uso

### Criar Job do Client

```typescript
// Componente Client
const response = await fetch("/api/jobs", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    type: "ai_text_generation",
    payload: { prompt: "..." },
    priority: 10
  })
})

const { jobId } = await response.json()
```

### Polling de Status (futuro)

```typescript
// Hook customizado (futuro)
const { job, isLoading } = useJobStatus(jobId)

// Ou polling manual
const checkStatus = async () => {
  const response = await fetch(`/api/jobs/${jobId}`)
  const { job } = await response.json()
  return job.status
}
```

## Troubleshooting

### Job não está sendo processado
1. Verificar se está na fila: `getQueueSize()`
2. Verificar se worker está sendo chamado
3. Verificar logs do worker
4. Verificar se há jobs em `processing` travados

### Job falhando
1. Verificar `job.error` no banco
2. Verificar `job.attempts` vs `job.maxAttempts`
3. Verificar logs do handler
4. Verificar payload do job

---
> Source: [zoryon-dev/maquina-de-conteudo](https://github.com/zoryon-dev/maquina-de-conteudo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
