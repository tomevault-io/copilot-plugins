## dialy-diet-api-nodejs

> Você é um desenvolvedor Node.js sênior especializado em arquitetura enterprise. Sua missão é criar uma **Daily Diet API** completa e production-ready que demonstre as melhores práticas do mercado internacional. Esta API será usada como projeto de portfolio para vagas de desenvolvedor Node.js sênior (3+ anos).


# Prompt para IA - Daily Diet API Enterprise

## **Contexto e Objetivo**

Você é um desenvolvedor Node.js sênior especializado em arquitetura enterprise. Sua missão é criar uma **Daily Diet API** completa e production-ready que demonstre as melhores práticas do mercado internacional. Esta API será usada como projeto de portfolio para vagas de desenvolvedor Node.js sênior (3+ anos).

## **Especificações Funcionais**

### **Requisitos de Negócio**
- **Gestão de Usuários**: Criação e autenticação de usuários
- **Controle de Refeições**: CRUD completo de refeições com relacionamento por usuário
- **Métricas Avançadas**: Cálculo de estatísticas e melhor sequência de dieta
- **Segurança**: Autenticação robusta e autorização por ownership
- **Performance**: API otimizada para uso em produção

### **Regras de Negócio Detalhadas**
1. **Usuários**:
   - Registro com nome, email único e senha
   - Autenticação via JWT com refresh token
   - Cada usuário só acessa seus próprios dados

2. **Refeições**:
   - Campos obrigatórios: nome, descrição, data/hora, status da dieta
   - Relacionamento N:1 com usuário
   - Operações CRUD completas com validação de ownership
   - Soft delete opcional

3. **Métricas do Usuário**:
   - Total de refeições registradas
   - Refeições dentro/fora da dieta
   - **Melhor sequência**: maior quantidade consecutiva de refeições dentro da dieta
   - Cálculo eficiente com cache quando possível

## **Stack Tecnológica Obrigatória**

```typescript
const techStack = {
  framework: 'NestJS',
  language: 'TypeScript (strict mode)',
  database: 'PostgreSQL 15+',
  orm: 'TypeORM',
  authentication: 'Passport.js + JWT Strategy',
  validation: 'Class-validator + Class-transformer',
  testing: 'Jest + Supertest',
  documentation: 'Swagger/OpenAPI automático',
  rateLimit: '@nestjs/throttler',
  caching: 'Redis (implementar para métricas)',
  containerization: 'Docker + Docker Compose',
  environment: 'dotenv + validation'
};
```

## **Arquitetura Obrigatória**

### **Clean Architecture Layers**
```
src/
├── modules/
│   ├── auth/
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── auth.module.ts
│   │   ├── strategies/jwt.strategy.ts
│   │   └── guards/jwt-auth.guard.ts
│   ├── users/
│   │   ├── entities/user.entity.ts
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   ├── users.repository.ts
│   │   └── users.module.ts
│   └── meals/
│       ├── entities/meal.entity.ts
│       ├── meals.controller.ts
│       ├── meals.service.ts
│       ├── meals.repository.ts
│       └── meals.module.ts
├── common/
│   ├── decorators/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   └── pipes/
├── config/
│   ├── database.config.ts
│   ├── jwt.config.ts
│   └── redis.config.ts
└── main.ts
```

## **Implementação Detalhada Requerida**

### **1. Entidades TypeORM**
```typescript
// Implementar com decorators completos
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  @OneToMany(() => Meal, meal => meal.user)
  meals: Meal[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

### **2. DTOs com Validação**
```typescript
// Usar class-validator para todas as validações
export class CreateMealDto {
  @IsString()
  @IsNotEmpty()
  @Length(1, 100)
  name: string;

  @IsString()
  @IsOptional()
  @Length(0, 500)
  description?: string;

  @IsDateString()
  datetime: string;

  @IsBoolean()
  isDietCompliant: boolean;
}
```

### **3. Authentication Completa**
- Implementar JWT Strategy com Passport
- Refresh token mechanism
- Password hashing com bcrypt
- Guards para proteção de rotas
- Decorators personalizados (@CurrentUser)

### **4. Swagger Documentation**
- Configuração automática do Swagger
- Documentação de todos os endpoints
- Schemas de request/response
- Exemplos de uso
- Autenticação Bearer Token

### **5. Rate Limiting Strategy**
```typescript
// Configurar throttling diferenciado por endpoint
const throttleConfig = {
  'POST /auth/login': { ttl: 60, limit: 5 },
  'POST /users': { ttl: 300, limit: 3 },
  'GET /meals': { ttl: 60, limit: 100 },
  'POST /meals': { ttl: 60, limit: 20 },
  'GET /users/metrics': { ttl: 60, limit: 30 }
};
```

### **6. Algoritmo de Melhor Sequência**
```typescript
// Implementar cálculo eficiente da melhor sequência
calculateBestDietSequence(meals: Meal[]): number {
  // Ordenar por datetime
  // Calcular sequências consecutivas
  // Retornar maior sequência
  // Considerar cache Redis para performance
}
```

### **7. Testing Strategy Completa**
- **Unit Tests**: Services, Utils (>90% coverage)
- **Integration Tests**: Controllers, Database
- **E2E Tests**: Fluxos completos da API
- **Test Database**: Configuração isolada
- **Mocking**: Externos dependencies

### **8. Error Handling Enterprise**
```typescript
// Global exception filter
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    // Logging estruturado
    // Response padronizado
    // Error tracking
  }
}
```

### **9. Docker Configuration**
```dockerfile
# Multi-stage build otimizado
FROM node:18-alpine AS builder
# Build steps

FROM node:18-alpine AS production
# Production configuration
```

```yaml
# docker-compose.yml com todos os serviços
version: '3.8'
services:
  app:
    # NestJS application
  db:
    # PostgreSQL
  redis:
    # Redis cache
  nginx:
    # Load balancer (opcional)
```

## **Endpoints Obrigatórios**

### **Authentication**
- `POST /auth/register` - Registro de usuário
- `POST /auth/login` - Login com JWT
- `POST /auth/refresh` - Refresh token
- `POST /auth/logout` - Logout

### **Meals Management**
- `GET /meals` - Listar refeições (paginado, filtros opcionais)
- `POST /meals` - Criar refeição
- `GET /meals/:id` - Buscar refeição específica
- `PUT /meals/:id` - Atualizar refeição
- `DELETE /meals/:id` - Deletar refeição

### **User Metrics**
- `GET /users/metrics` - Métricas completas do usuário
- `GET /users/profile` - Perfil do usuário atual

### **Health & Monitoring**
- `GET /health` - Health check
- `GET /metrics` - Métricas da aplicação (opcional)

## **Critérios de Qualidade Enterprise**

### **Performance**
- Response time < 200ms para operações simples
- Response time < 500ms para cálculo de métricas
- Connection pooling configurado
- Indexes otimizados no database

### **Security**
- Password hashing com salt
- JWT com expiração curta + refresh token
- Rate limiting por IP e usuário
- Validation de todos os inputs
- CORS configurado adequadamente

### **Monitoring & Logging**
- Structured logging (JSON)
- Request/Response logging
- Error tracking
- Performance metrics
- Health checks

### **Code Quality**
- ESLint + Prettier configurados
- Husky pre-commit hooks
- TypeScript strict mode
- 90%+ test coverage
- Documentação inline

### **Production Ready**
- Environment configuration
- Database migrations
- Graceful shutdown
- Process management (PM2 ready)
- Memory leak prevention

## **Deliverables Esperados**

1. **Código completo** com estrutura enterprise
2. **README.md detalhado** com setup e documentação
3. **Docker setup funcional** com todos os serviços
4. **Database migrations** com seeds de exemplo
5. **Postman collection** ou arquivo REST para testes
6. **Swagger documentation** acessível via `/api/docs`
7. **Scripts NPM** para desenvolvimento e produção
8. **Testes automatizados** com coverage report
9. **CI/CD pipeline** básico (GitHub Actions)
10. **Environment examples** (.env.example)

## **Configurações Específicas**

### **Package.json Scripts**
```json
{
  "scripts": {
    "build": "nest build",
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:prod": "node dist/main",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:e2e": "jest --config ./test/jest-e2e.json",
    "migration:generate": "typeorm migration:generate",
    "migration:run": "typeorm migration:run",
    "seed": "ts-node src/database/seeds/index.ts"
  }
}
```

### **Environment Variables**
```env
# Database
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=daily_diet
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=password

# JWT
JWT_SECRET=your-super-secret-key
JWT_EXPIRES_IN=15m
JWT_REFRESH_SECRET=your-refresh-secret
JWT_REFRESH_EXPIRES_IN=7d

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379

# Application
PORT=3000
NODE_ENV=development
```

## **Critérios de Sucesso**

A implementação será considerada enterprise-grade se:

- ✅ **Funcionalidade**: Todos os requisitos implementados
- ✅ **Arquitetura**: Clean Architecture aplicada corretamente
- ✅ **Segurança**: JWT + validation + rate limiting funcionais
- ✅ **Performance**: Métricas otimizadas com cache
- ✅ **Testes**: Coverage >90% com testes significativos
- ✅ **Documentação**: Swagger completo + README detalhado
- ✅ **Containerização**: Docker setup funcional
- ✅ **Code Quality**: TypeScript strict + ESLint + formatação
- ✅ **Production Ready**: Health checks + error handling + logging

## **Bonus Points**

- Implementar soft delete para refeições
- Cache Redis para métricas frequentes
- Middleware de request logging
- Database indexing otimizado
- API versioning strategy
- OpenAPI 3.0 com exemplos
- Background jobs para cálculos pesados
- Webhook notifications (simulado)
- Audit trail para alterações

**IMPORTANTE**: Esta deve ser uma implementação que impressione em uma entrevista técnica para vaga sênior internacional. Foque em demonstrar expertise enterprise-level em cada aspecto do código.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cdfortes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
