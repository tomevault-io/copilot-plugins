## ponto-pj

> Este arquivo contém todas as boas práticas e padrões arquiteturais implementados neste projeto. Use estas regras como guia para manter a qualidade e consistência do código.

# 🚀 Ponto PJ - Cursor Rules & Best Practices

Este arquivo contém todas as boas práticas e padrões arquiteturais implementados neste projeto. Use estas regras como guia para manter a qualidade e consistência do código.

## 🏗️ Arquitetura & Padrões

### Clean Architecture
- **Separação de responsabilidades**: Cada camada tem uma função específica
- **Presentation Layer**: React components, hooks, stores
- **Business Logic Layer**: Services, validation, error handling
- **Data Access Layer**: Repositories, cache, external APIs

### Repository Pattern
- **BaseRepository**: Classe abstrata com funcionalidades comuns
- **CachedRepository**: Extensão com sistema de cache inteligente
- **Especialização**: Cada entidade tem seu próprio repository
- **Validação**: Integrada nos métodos create, update, upsert

### SOLID Principles
- **S** - Single Responsibility: Cada classe tem uma única responsabilidade
- **O** - Open/Closed: Extensível sem modificação
- **L** - Liskov Substitution: Repositories intercambiáveis
- **I** - Interface Segregation: Interfaces específicas
- **D** - Dependency Inversion: Dependências injetadas

## 📁 Estrutura de Arquivos

### Organização por Responsabilidade
```
src/
├── components/     # UI Components (apenas apresentação)
├── hooks/         # Custom hooks (lógica de UI)
├── services/      # Business logic (orquestração)
├── repositories/  # Data access (comunicação com APIs)
├── stores/        # State management (apenas estado)
├── lib/           # Utilities (funções puras)
├── types/         # TypeScript definitions
└── test/          # Testes organizados por tipo
```

### Convenções de Nomenclatura
- **Components**: PascalCase (ex: `WorkSessionCard.tsx`)
- **Hooks**: camelCase com prefixo `use` (ex: `useWorkSession.ts`)
- **Services**: camelCase com sufixo `Service` (ex: `workSessionService.ts`)
- **Repositories**: camelCase com sufixo `Repository` (ex: `workSessionRepository.ts`)
- **Stores**: camelCase com sufixo `Store` (ex: `workSessionStore.ts`)
- **Types**: PascalCase (ex: `WorkSession.ts`)
- **Constants**: UPPER_SNAKE_CASE (ex: `API_ENDPOINTS.ts`)

## 🎯 Padrões de Código

### TypeScript
- **Tipagem forte**: Sempre use tipos explícitos
- **Interfaces**: Para estruturas de dados
- **Types**: Para unions, intersections, mapped types
- **Generics**: Para componentes e funções reutilizáveis
- **Strict mode**: Sempre habilitado

### React Components
```typescript
// ✅ Padrão correto
interface ComponentProps {
  data: WorkSession
  onAction: (id: string) => void
  loading?: boolean
}

export function Component({ data, onAction, loading = false }: ComponentProps) {
  // Lógica do componente
  return <div>...</div>
}

// ❌ Evitar
export function Component(props: any) {
  // Sem tipagem
}
```

### Custom Hooks
```typescript
// ✅ Padrão correto
export function useWorkSession(userId: string) {
  const [data, setData] = useState<WorkSession | null>(null)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  // Lógica do hook
  return { data, loading, error, refetch }
}

// ❌ Evitar
export function useWorkSession() {
  // Sem parâmetros tipados
  // Sem retorno tipado
}
```

### Services
```typescript
// ✅ Padrão correto
export class WorkSessionService {
  async getSessionByDate(date: string): Promise<WorkSession | null> {
    try {
      const userId = await workSessionRepository.getCurrentUserId()
      return await workSessionRepository.findByUserAndDate(userId, date)
    } catch (error) {
      console.error('Erro ao buscar sessão por data:', error)
      throw error
    }
  }
}

// ❌ Evitar
export class WorkSessionService {
  async getSessionByDate(date: any) {
    // Sem tipagem
    // Sem tratamento de erro
    return await supabase.from('work_sessions').select('*')
  }
}
```

### Repositories
```typescript
// ✅ Padrão correto
export class WorkSessionRepository extends CachedRepository {
  protected tableName = 'work_sessions'

  async findByUserAndDate(userId: string, date: string): Promise<WorkSession | null> {
    const cacheKey = this.generateCacheKey('session', userId, date)
    
    return this.getCached(cacheKey, async () => {
      const { data, error } = await this.getSupabase()
        .from(this.tableName)
        .select('*')
        .eq('user_id', userId)
        .eq('date', date)
        .single()

      if (error && error.code !== 'PGRST116') {
        this.handleError(error, 'findByUserAndDate')
      }

      return data
    }, { ttl: 2 * 60 * 1000 })
  }
}

// ❌ Evitar
export class WorkSessionRepository {
  async findByUserAndDate(userId: any, date: any) {
    // Sem cache
    // Sem tratamento de erro padronizado
    // Sem tipagem
  }
}
```

## 🔒 Validação & Tratamento de Erros

### Sistema de Validação
```typescript
// ✅ Usar validadores específicos
const validation = WorkSessionValidator.validateCreateData(data)
if (!validation.isValid) {
  throw new Error(`validation.error: ${validation.errors.join(', ')}`)
}

// ❌ Evitar validação manual
if (!data.user_id) {
  throw new Error('User ID is required')
}
```

### Tratamento de Erros
```typescript
// ✅ Usar ErrorHandler centralizado
import { ErrorHandler } from '@/lib/errorHandler'

try {
  // Operação
} catch (error) {
  const userMessage = ErrorHandler.mapError(error)
  // Mostrar mensagem amigável
}

// ❌ Evitar tratamento genérico
try {
  // Operação
} catch (error) {
  console.error(error)
  // Sem mapeamento de erro
}
```

## ⚡ Performance & Cache

### Cache Inteligente
```typescript
// ✅ Usar cache com TTL apropriado
return this.getCached(cacheKey, async () => {
  // Operação custosa
}, { ttl: 2 * 60 * 1000 }) // 2 minutos para dados dinâmicos

// ❌ Evitar cache sem TTL
return this.getCached(cacheKey, async () => {
  // Operação
}) // Sem TTL definido
```

### Invalidação de Cache
```typescript
// ✅ Invalidar cache relacionado
this.invalidateCacheByPattern(`session:${userId}`)
this.invalidateCacheByPattern(`sessions:${userId}`)

// ❌ Evitar invalidação desnecessária
this.clearCache() // Limpa todo o cache
```

## 🧪 Testes

### Estratégia de Testes
- **Unit Tests**: Business logic isolada
- **Integration Tests**: Fluxos principais
- **Component Tests**: Comportamento de UI
- **Repository Tests**: Mock de dependências

### Padrão de Testes
```typescript
// ✅ Teste de business logic
describe('WorkSessionBusinessService', () => {
  it('should calculate statistics correctly', () => {
    const sessions = [/* mock data */]
    const stats = WorkSessionBusinessService.calculateStatistics(sessions)
    expect(stats.totalHours).toBe(40)
  })
})

// ✅ Teste de repository
describe('WorkSessionRepository', () => {
  it('should cache results', async () => {
    const result1 = await repository.findByUserAndDate(userId, date)
    const result2 = await repository.findByUserAndDate(userId, date)
    expect(result1).toBe(result2) // Cache hit
  })
})
```

## 🌐 Internacionalização

### Uso de Traduções
```typescript
// ✅ Usar hook de tradução
import { useTranslation } from '@/i18n/useTranslation'

export function Component() {
  const { t } = useTranslation()
  return <div>{t('app.title')}</div>
}

// ❌ Evitar strings hardcoded
export function Component() {
  return <div>Ponto PJ</div> // Sem tradução
}
```

### Estrutura de Traduções
```json
{
  "app": {
    "title": "Ponto PJ",
    "subtitle": "Sistema de Ponto Eletrônico"
  },
  "errors": {
    "validation": {
      "required": "Campo obrigatório"
    }
  }
}
```

## 📱 PWA & Mobile-First

### Componentes Mobile-Friendly
```typescript
// ✅ Componentes touch-friendly
<button className="min-h-[44px] min-w-[44px]">
  {/* Conteúdo */}
</button>

// ❌ Evitar elementos muito pequenos
<button className="h-6 w-6">
  {/* Difícil de tocar */}
</button>
```

### Responsividade
```css
/* ✅ Mobile-first approach */
.container {
  padding: 1rem;
}

@media (min-width: 768px) {
  .container {
    padding: 2rem;
  }
}

/* ❌ Desktop-first */
.container {
  padding: 2rem;
}

@media (max-width: 767px) {
  .container {
    padding: 1rem;
  }
}
```

## 🔐 Segurança

### Validação de Entrada
```typescript
// ✅ Validar sempre dados de entrada
const validation = UserValidator.validateCreateData(data)
if (!validation.isValid) {
  throw new Error(`validation.error: ${validation.errors.join(', ')}`)
}

// ❌ Confiar apenas em validação client-side
// Sem validação server-side
```

### Autenticação
```typescript
// ✅ Usar repository para autenticação
const userId = await workSessionRepository.getCurrentUserId()

// ❌ Acessar Supabase diretamente
const { data: { user } } = await supabase.auth.getUser()
```

## 📊 Estado & Gerenciamento

### Stores (Zustand)
```typescript
// ✅ Store focado apenas em estado
export const useSessionStore = create<SessionState>((set) => ({
  session: null,
  loading: false,
  
  setSession: (session) => set({ session }),
  setLoading: (loading) => set({ loading }),
  
  // Apenas orquestração, sem lógica de negócio
  loadSession: async () => {
    set({ loading: true })
    try {
      const session = await workSessionService.getSessionByDate(today)
      set({ session })
    } finally {
      set({ loading: false })
    }
  }
}))

// ❌ Store com lógica de negócio
export const useSessionStore = create<SessionState>((set) => ({
  // Lógica de cálculo de tempo trabalhado
  // Comunicação direta com Supabase
  // Validação de dados
}))
```

## 🎨 Design System

### Componentes Reutilizáveis
```typescript
// ✅ Componente com props bem definidas
interface SquareCTAProps {
  title: string
  subtitle?: string
  icon: React.ReactNode
  onClick: () => void
  variant?: 'primary' | 'secondary' | 'danger'
  disabled?: boolean
}

export function SquareCTA({ 
  title, 
  subtitle, 
  icon, 
  onClick, 
  variant = 'primary',
  disabled = false 
}: SquareCTAProps) {
  // Implementação
}

// ❌ Componente com props genéricas
export function SquareCTA(props: any) {
  // Sem tipagem
}
```

### Cores e Gradientes
```css
/* ✅ Usar variáveis CSS */
:root {
  --primary-gradient: linear-gradient(135deg, #3b82f6, #1d4ed8);
  --success-gradient: linear-gradient(135deg, #10b981, #059669);
  --danger-gradient: linear-gradient(135deg, #ef4444, #dc2626);
}

/* ❌ Cores hardcoded */
.button {
  background: linear-gradient(135deg, #3b82f6, #1d4ed8);
}
```

## 🚀 Performance

### Code Splitting
```typescript
// ✅ Lazy loading de componentes
const HistoryPage = lazy(() => import('@/pages/History'))

// ✅ Lazy loading de rotas
const routes = [
  {
    path: '/history',
    element: <Suspense fallback={<Loading />}><HistoryPage /></Suspense>
  }
]
```

### Bundle Optimization
```typescript
// ✅ Manual chunks para otimização
manualChunks: {
  'react-vendor': ['react', 'react-dom'],
  'mantine': ['@mantine/core', '@mantine/hooks'],
  'supabase': ['@supabase/supabase-js'],
  'i18n': ['i18next', 'react-i18next'],
  'utils': ['dayjs', 'jspdf']
}
```

## 📝 Commits & Git

### Conventional Commits
```bash
# ✅ Padrão correto
feat: add intelligent caching to repositories
fix: resolve validation error in work session creation
docs: update README with PWA features
refactor: extract business logic to separate service
test: add unit tests for WorkSessionBusinessService

# ❌ Commits genéricos
update code
fix bug
add feature
```

### Git Hooks (Husky)
O projeto usa Husky para automatizar verificações de qualidade:

#### Pre-commit Hook
- **Lint-staged**: Executa ESLint e Prettier apenas nos arquivos staged
- **Testes**: Roda testes unitários automaticamente
- **Formatação**: Aplica formatação automática com Prettier

#### Pre-push Hook
- **Type checking**: Verifica tipos TypeScript
- **Testes com coverage**: Executa testes com relatório de cobertura
- **Validação rigorosa**: Garante qualidade antes do push

#### Commit-msg Hook
- **Conventional Commits**: Valida formato das mensagens de commit
- **Padrão obrigatório**: Força uso de conventional commits

```bash
# Hooks configurados
.husky/
├── pre-commit      # Lint + Testes
├── pre-push        # Type check + Coverage
└── commit-msg      # Validação de mensagem
```

### Branch Naming
```bash
# ✅ Padrão correto
feature/intelligent-caching
fix/validation-errors
refactor/repository-pattern
docs/update-readme

# ❌ Nomes genéricos
feature
fix
update
```

## 🎯 Checklist de Qualidade

### Antes de Commitar
- [ ] Código segue padrões TypeScript
- [ ] Componentes são reutilizáveis
- [ ] Business logic está em services
- [ ] Validação implementada
- [ ] Tratamento de erro adequado
- [ ] Cache implementado quando necessário
- [ ] Testes escritos para nova funcionalidade
- [ ] Internacionalização adicionada
- [ ] Performance não degradada
- [ ] Acessibilidade mantida
- [ ] **Husky hooks executados automaticamente** ✅

### Antes de Fazer PR
- [ ] Todos os testes passando
- [ ] ESLint sem warnings
- [ ] TypeScript sem errors
- [ ] Documentação atualizada
- [ ] Performance testada
- [ ] Mobile responsivo
- [ ] PWA funcional
- [ ] Cache funcionando
- [ ] Validação robusta
- [ ] **Pre-push hook executado** ✅
- [ ] **Coverage de testes adequado** ✅

## 🚨 Anti-Patterns a Evitar

### ❌ Violações de SRP
```typescript
// ❌ Classe com múltiplas responsabilidades
class WorkSessionService {
  async startSession() {
    // Comunicação com API
    // Cálculo de tempo trabalhado
    // Validação de dados
    // Atualização de estado
    // Logging
  }
}

// ✅ Separação de responsabilidades
class WorkSessionService {
  async startSession() {
    const userId = await workSessionRepository.getCurrentUserId()
    const createData = WorkSessionBusinessService.formatCreateData(...)
    return await workSessionRepository.upsert(createData)
  }
}
```

### ❌ Comunicação Direta com APIs
```typescript
// ❌ Acessar Supabase diretamente em componentes
export function Component() {
  const { data } = await supabase.from('work_sessions').select('*')
  return <div>{data}</div>
}

// ✅ Usar repositories
export function Component() {
  const sessions = await workSessionRepository.findByUser(userId)
  return <div>{sessions}</div>
}
```

### ❌ Lógica de Negócio em Stores
```typescript
// ❌ Store com lógica de negócio
const store = create((set) => ({
  startJourney: async () => {
    const workedHours = calculateWorkedHours(start, end)
    // Lógica de negócio no store
  }
}))

// ✅ Store apenas com estado
const store = create((set) => ({
  startJourney: async () => {
    const session = await workSessionService.startSession()
    set({ session })
  }
}))
```

## 🎉 Conclusão

Seguindo estas regras, você manterá a qualidade e consistência do código, garantindo que o projeto continue sendo um boilerplate de referência para aplicações React escaláveis.

**Lembre-se**: Qualidade não é um ato, é um hábito. Aplique estas regras consistentemente em todo o projeto! 🚀 

---
> Source: [tiagovilasboas/ponto-pj](https://github.com/tiagovilasboas/ponto-pj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
