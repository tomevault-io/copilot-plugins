## projeto-website

> Cole este arquivo como `.cursorrules` na raiz do projeto OU cole diretamente no chat do Cursor antes de começar.

# 🧠 Contexto do Projeto — Projeto-WebSite

Cole este arquivo como `.cursorrules` na raiz do projeto OU cole diretamente no chat do Cursor antes de começar.

---

## 🎯 O que é este projeto

E-commerce full-stack criado como portfólio pessoal e aprendizado.
O frontend original era estático (HTML/CSS/JS). Estamos reformando para uma arquitetura moderna com backend, banco de dados, busca com IA e automações.

---

## 🏗️ Arquitetura

```
frontend/       → Next.js (TypeScript) — deploy na Vercel
backend/        → NestJS (TypeScript) — deploy no Railway
ai-search/      → FastAPI (Python) — deploy no Railway
docker-compose  → sobe tudo localmente para dev
```

### Serviços e portas locais

| Serviço       | Porta | Tecnologia              |
|---------------|-------|-------------------------|
| Frontend      | 3001  | Next.js                 |
| Backend API   | 3000  | NestJS                  |
| AI Search     | 8000  | FastAPI                 |
| PostgreSQL    | 5432  | pgvector/pgvector:pg16  |
| Redis         | 6379  | redis:7-alpine          |
| Qdrant        | 6333  | qdrant/qdrant           |
| N8N           | 5678  | n8nio/n8n               |

---

## 🛒 Funcionalidades a construir

1. **Catálogo de produtos** — listagem, detalhe, CRUD admin
2. **Autenticação** — registro, login, JWT + sessão no Redis
3. **Carrinho** — temporário no Redis, persistido ao finalizar
4. **Busca semântica** — texto livre → FastAPI gera embedding → busca no pgvector/Qdrant → retorna produtos similares
5. **Finalizar pedido** — salva no Postgres, dispara webhook no N8N
6. **Automações N8N** — email de boas-vindas no cadastro + confirmação de pedido

---

## 🗄️ Banco de dados (PostgreSQL)

### Tabelas principais

```sql
-- Usuários
users (id, name, email, password_hash, created_at)

-- Produtos
products (id, name, description, price, stock, image_url, embedding vector(1536), created_at)

-- Pedidos
orders (id, user_id, total, status, created_at)
order_items (id, order_id, product_id, quantity, unit_price)
```

### Extensão vetorial
```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE INDEX ON products USING ivfflat (embedding vector_cosine_ops);
```

---

## 🔧 Padrões de código

### NestJS
- Usar módulos por domínio: `ProductsModule`, `AuthModule`, `OrdersModule`, `CartModule`
- DTOs com `class-validator` para validação
- Repositórios com TypeORM
- Guards JWT para rotas protegidas
- `.env` para todas as variáveis de ambiente

### FastAPI
- Endpoint principal: `POST /search` recebe `{ query: string }` e retorna lista de produtos
- Usar `sentence-transformers` (modelo: `all-MiniLM-L6-v2`) para gerar embeddings
- Conectar ao Qdrant para busca vetorial
- Sincronizar produtos do Postgres via endpoint `POST /sync`

### Next.js
- App Router (pasta `app/`)
- Tailwind CSS para estilização
- Componentes em `components/`
- Chamadas à API em `services/api.ts`
- Variável de ambiente: `NEXT_PUBLIC_API_URL`

---

## 🐳 docker-compose.yml

```yaml
version: "3.9"
services:
  postgres:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_DB: ecommerce
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    ports: ["5432:5432"]
    volumes: ["postgres_data:/var/lib/postgresql/data"]

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

  qdrant:
    image: qdrant/qdrant
    ports: ["6333:6333"]
    volumes: ["qdrant_data:/qdrant/storage"]

  n8n:
    image: n8nio/n8n
    ports: ["5678:5678"]
    environment:
      N8N_BASIC_AUTH_ACTIVE: "true"
      N8N_BASIC_AUTH_USER: admin
      N8N_BASIC_AUTH_PASSWORD: secret
    volumes: ["n8n_data:/home/node/.n8n"]

  backend:
    build: ./backend
    ports: ["3000:3000"]
    depends_on: [postgres, redis]
    environment:
      DATABASE_URL: postgresql://admin:secret@postgres:5432/ecommerce
      REDIS_URL: redis://redis:6379
      JWT_SECRET: troque_isso_em_producao

  ai-search:
    build: ./ai-search
    ports: ["8000:8000"]
    depends_on: [qdrant, postgres]
    environment:
      QDRANT_URL: http://qdrant:6333
      DATABASE_URL: postgresql://admin:secret@postgres:5432/ecommerce

volumes:
  postgres_data:
  qdrant_data:
  n8n_data:
```

---

## 📦 Variáveis de ambiente

### backend/.env
```
DATABASE_URL=postgresql://admin:secret@localhost:5432/ecommerce
REDIS_URL=redis://localhost:6379
JWT_SECRET=troque_isso_em_producao
JWT_EXPIRES_IN=7d
AI_SEARCH_URL=http://localhost:8000
N8N_WEBHOOK_URL=http://localhost:5678/webhook
```

### ai-search/.env
```
QDRANT_URL=http://localhost:6333
DATABASE_URL=postgresql://admin:secret@localhost:5432/ecommerce
COLLECTION_NAME=products
```

### frontend/.env.local
```
NEXT_PUBLIC_API_URL=http://localhost:3000
```

---

## 🚀 Deploy

| Serviço    | Plataforma | Como                                      |
|------------|------------|-------------------------------------------|
| Frontend   | Vercel     | Conectar repo GitHub, apontar para /frontend |
| Backend    | Railway    | Novo serviço → Deploy from GitHub → /backend |
| AI Search  | Railway    | Novo serviço → Deploy from GitHub → /ai-search |
| PostgreSQL | Railway    | Add Plugin → PostgreSQL                   |
| Redis      | Railway    | Add Plugin → Redis                        |
| Qdrant     | Railway    | Novo serviço → Docker Image → qdrant/qdrant |
| N8N        | Railway    | Novo serviço → Docker Image → n8nio/n8n   |

---

## 📋 Ordem de desenvolvimento recomendada

1. `docker-compose up` — verificar que todos os serviços sobem
2. NestJS: módulo de produtos (CRUD básico + TypeORM + Postgres)
3. NestJS: autenticação JWT + Redis para sessão
4. Next.js: migrar o frontend estático para componentes React consumindo a API
5. FastAPI: endpoint de busca semântica com pgvector/Qdrant
6. NestJS: carrinho (Redis) + pedidos (Postgres)
7. N8N: webhook de pedido + email de boas-vindas
8. Deploy: Railway (backend) + Vercel (frontend)

---

## ⚠️ Regras importantes

- NUNCA commitar arquivos `.env` — usar `.env.example` com valores fictícios
- Sempre tipar tudo em TypeScript (no `any`)
- Migrations do TypeORM para toda mudança de schema
- Endpoints REST seguindo padrão: `GET /products`, `POST /products`, `GET /products/:id`, etc.
- Respostas de erro padronizadas: `{ statusCode, message, error }`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neuxxkk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
