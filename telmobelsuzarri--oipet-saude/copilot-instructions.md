## oipet-saude

> O **OiPet Saúde** é um aplicativo multiplataforma de gestão de saúde e alimentação para pets, integrado ao ecossistema da empresa OiPet. O projeto consiste em aplicações mobile (iOS/Android), web e um backend unificado que suporta todas as plataformas.

# CLAUDE.md - Projeto OiPet Saúde

## Visão Geral do Projeto

O **OiPet Saúde** é um aplicativo multiplataforma de gestão de saúde e alimentação para pets, integrado ao ecossistema da empresa OiPet. O projeto consiste em aplicações mobile (iOS/Android), web e um backend unificado que suporta todas as plataformas.

## Arquitetura do Sistema

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Mobile App    │    │    Web App      │    │   Admin Panel   │
│  (iOS/Android)  │    │   (React/Vue)   │    │   (React/Vue)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   API Gateway   │
                    │  (Express/NestJS)│
                    └─────────────────┘
                                 │
                    ┌─────────────────┐
                    │    Backend      │
                    │ (Node.js/Python)│
                    └─────────────────┘
                                 │
                    ┌─────────────────┐
                    │   Database      │
                    │ (PostgreSQL/MongoDB)│
                    └─────────────────┘
```

## Estrutura de Pastas

```
oipet-saude/
├── backend/
│   ├── src/
│   │   ├── controllers/
│   │   ├── models/
│   │   ├── routes/
│   │   ├── middleware/
│   │   ├── services/
│   │   └── utils/
│   ├── tests/
│   ├── docs/
│   └── config/
├── mobile/
│   ├── src/
│   │   ├── components/
│   │   ├── screens/
│   │   ├── services/
│   │   ├── hooks/
│   │   ├── utils/
│   │   └── assets/
│   ├── __tests__/
│   └── docs/
├── web/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/
│   │   ├── hooks/
│   │   ├── utils/
│   │   └── assets/
│   ├── tests/
│   └── docs/
├── admin/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/
│   │   ├── hooks/
│   │   ├── utils/
│   │   └── assets/
│   ├── tests/
│   └── docs/
├── shared/
│   ├── types/
│   ├── constants/
│   ├── utils/
│   └── schemas/
└── docs/
    ├── api/
    ├── deployment/
    └── development/
```

## Stack Tecnológica

### Backend
- **Framework**: Node.js com Express/NestJS ou Python com Django/FastAPI
- **Database**: MongoDB (principal) + Redis (cache)
- **Authentication**: JWT + bcrypt
- **File Upload**: Railway Volume Storage ou AWS S3
- **Email**: SendGrid ou AWS SES
- **Push Notifications**: Firebase Cloud Messaging
- **Image Recognition**: TensorFlow Lite ou AWS Rekognition
- **Deployment**: Railway (GitHub integration)

### Mobile
- **Framework**: React Native ou Flutter
- **State Management**: Redux Toolkit ou Zustand
- **Navigation**: React Navigation com liquid glass tab bars
- **HTTP Client**: Axios
- **Camera**: react-native-camera ou expo-camera
- **Charts**: Victory Native ou React Native Chart Kit com glass containers
- **Glass Effects**: Custom glass components com blur effects

### Web/Admin
- **Framework**: React.js ou Vue.js
- **State Management**: Redux Toolkit ou Pinia
- **UI Library**: Material-UI ou Ant Design + Liquid Glass custom components
- **Charts**: Chart.js ou Recharts com glass overlays
- **HTTP Client**: Axios
- **Build Tool**: Vite ou Webpack
- **Glass Effects**: CSS backdrop-filter + custom glass components

## Design System: Liquid Glass (Apple-Inspired)

O OiPet Saúde segue o padrão **Liquid Glass** inspirado no ecosistema Apple, criando uma interface moderna e fluida com elementos translúcidos que se adaptam perfeitamente ao conteúdo de fundo.

### Filosofia de Design Apple-Inspired
- **Adaptabilidade**: Glass effects que se ajustam ao conteúdo de fundo
- **Consistência**: Mesmo padrão visual em todas as plataformas
- **Profundidade**: Camadas visuais que criam hierarquia
- **Sutileza**: Transparência que não compromete a legibilidade

### Configurações Glass Específicas (baseadas no iOS/macOS)
```css
:root {
  /* Configurações glass precisas baseadas nas imagens */
  --glass-blur: 21.8%;
  --glass-translucency: 50%;
  --glass-dark-overlay: 42%;
  --glass-shadow-opacity: 50%;
  --glass-border-radius: 16px;
  --glass-border: 1px solid rgba(255, 255, 255, 0.18);
  
  /* Cores OiPet adaptadas para glass */
  --glass-coral-primary: rgba(232, 90, 90, 0.8);
  --glass-coral-secondary: rgba(232, 90, 90, 0.5);
  --glass-teal-primary: rgba(90, 163, 163, 0.8);
  --glass-teal-secondary: rgba(90, 163, 163, 0.5);
  
  /* Variações de contexto */
  --glass-widget: rgba(255, 255, 255, 0.1);
  --glass-sidebar: rgba(255, 255, 255, 0.05);
  --glass-dock: rgba(255, 255, 255, 0.08);
  --glass-notification: rgba(255, 255, 255, 0.12);
}
```

### Componentes Glass Específicos OiPet

#### 1. Pet Health Widget (inspirado nos widgets de clima)
```jsx
const PetHealthWidget = ({ pet }) => (
  <GlassContainer
    style={{
      background: 'var(--glass-widget)',
      backdropFilter: 'blur(20px)',
      borderRadius: 16,
      padding: 16,
      border: 'var(--glass-border)',
    }}
  >
    <Text style={styles.petName}>{pet.name}</Text>
    <Text style={styles.lastWeight}>{pet.weight}kg</Text>
    <Text style={styles.healthStatus}>Saudável</Text>
  </GlassContainer>
);
```

#### 2. Sidebar Glass (inspirado na sidebar do design tool)
```jsx
const SidebarGlass = ({ navigation }) => (
  <View style={{
    background: 'var(--glass-sidebar)',
    backdropFilter: 'blur(25px)',
    borderRight: 'var(--glass-border)',
    width: 280,
    height: '100vh',
  }}>
    <NavigationItems />
  </View>
);
```

#### 3. Dock Navigation (inspirado no dock do macOS)
```jsx
const DockNavigation = ({ activeTab, onTabChange }) => (
  <View style={{
    position: 'absolute',
    bottom: 20,
    left: 20,
    right: 20,
    background: 'var(--glass-dock)',
    backdropFilter: 'blur(30px)',
    borderRadius: 24,
    padding: 12,
    flexDirection: 'row',
    justifyContent: 'space-around',
    border: 'var(--glass-border)',
  }}>
    <DockItem icon="paw" label="Pets" active={activeTab === 'pets'} />
    <DockItem icon="heart" label="Saúde" active={activeTab === 'health'} />
    <DockItem icon="camera" label="Scanner" active={activeTab === 'scanner'} />
    <DockItem icon="shopping-cart" label="Loja" active={activeTab === 'store'} />
  </View>
);
```

#### 4. Notification Glass (inspirado nos lembretes)
```jsx
const NotificationGlass = ({ notification }) => (
  <View style={{
    background: 'var(--glass-notification)',
    backdropFilter: 'blur(18px)',
    borderRadius: 12,
    padding: 12,
    marginBottom: 8,
    border: 'var(--glass-border)',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.1,
    shadowRadius: 12,
  }}>
    <Text style={styles.notificationTitle}>{notification.title}</Text>
    <Text style={styles.notificationMessage}>{notification.message}</Text>
  </View>
);
```

### Elementos com Glass Effect no OiPet

#### Elementos QUE devem usar Glass (baseado nas imagens):
- **Dock de navegação**: Como o dock do macOS
- **Widgets de saúde**: Como os widgets de clima/calendário
- **Sidebar de navegação**: Como painéis laterais
- **Modals e overlays**: Janelas flutuantes
- **Status indicators**: Indicadores de progresso (100%)
- **Floating action buttons**: Botões de ação rápida
- **Notifications**: Notificações push
- **Toolbars**: Barras de ferramentas
- **Cards destacados**: Informações importantes sobre pets

#### Elementos que NÃO devem usar Glass:
- **Conteúdo principal**: Listas de dados, formulários
- **Texto corrido**: Parágrafos longos
- **Inputs de texto**: Campos de formulário
- **Backgrounds principais**: Papel de parede da aplicação
- **Tabelas de dados**: Listagens extensas

### Adaptação Multiplataforma

#### Mobile (React Native)
```jsx
const GlassEffectMobile = {
  backgroundColor: 'rgba(255, 255, 255, 0.1)',
  borderRadius: 16,
  borderWidth: 1,
  borderColor: 'rgba(255, 255, 255, 0.18)',
  shadowColor: '#000',
  shadowOffset: { width: 0, height: 8 },
  shadowOpacity: 0.12,
  shadowRadius: 24,
  // No iOS, usar backdrop-filter quando disponível
  backdropFilter: 'blur(20px)',
};
```

#### Web (CSS)
```css
.glass-effect-web {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.18);
  border-radius: 16px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.12);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}
```

### Diretrizes de Implementação Apple-Style

1. **Logo OiPet**: OBRIGATÓRIO em todas as telas principais
   - Header de dashboards, páginas de login, splash screens
   - Manter cores originais: coral (#E85A5A) e teal (#5AA3A3)
   - Incluir elemento pata como ícone secundário
   - Usar em glass containers quando apropriado
2. **Consistência visual**: Manter mesmo blur e opacidade em contextos similares
3. **Adaptabilidade**: Glass effects que funcionam em backgrounds claros e escuros
4. **Performance**: Otimizar backdrop-filter para não impactar FPS
5. **Hierarquia**: Diferentes intensidades de glass para diferentes níveis de importância
6. **Contextualidade**: Glass effects que fazem sentido com o conteúdo subjacente
7. **Responsividade**: Adaptar intensidade baseada no tamanho da tela
8. **Acessibilidade**: Garantir contraste mínimo mesmo com transparência
9. **Branding**: Cores OiPet integradas sutilmente nos glass effects

## Tipos de Usuário

### 1. Usuário Normal (Cliente)
**Funcionalidades:**
- Cadastro/Login com confirmação por email
- Gerenciamento de perfil de pets
- Monitoramento de saúde (peso, altura, atividade, calorias)
- Escaneamento de alimentos por foto
- Visualização de gráficos de saúde
- Acesso ao catálogo de produtos OiPet
- Recebimento de notificações e notícias

### 2. Usuário Administrativo
**Funcionalidades:**
- Dashboard com métricas gerais
- Gestão de usuários e pets
- Monitoramento de acessos e analytics
- Envio de notícias e push notifications
- Análise financeira (receita, custos, margem)
- Geração e exportação de relatórios (Excel/PDF)

## Fluxos Principais

### Fluxo do Usuário Normal
1. **Onboarding**: Cadastro → Confirmação email → Login → Tutorial
2. **Configuração**: Cadastro do pet → Dados iniciais → Foto
3. **Uso Diário**: Check-in saúde → Registro alimentação → Escaneamento → Gráficos
4. **E-commerce**: Navegação catálogo → Redirecionamento site OiPet

### Fluxo do Administrador
1. **Acesso**: Login admin → Dashboard principal
2. **Gestão**: Usuários → Pets → Analytics → Comunicações
3. **Relatórios**: Filtros → Geração → Exportação

## Modelos de Dados Principais

### Modelos de Dados Principais (MongoDB)

### User
```javascript
interface User {
  _id: ObjectId;
  email: string;
  password: string;
  name: string;
  phone?: string;
  avatar?: string;
  isAdmin: boolean;
  isEmailVerified: boolean;
  createdAt: Date;
  updatedAt: Date;
}
```

### Pet
```javascript
interface Pet {
  _id: ObjectId;
  userId: ObjectId;
  name: string;
  species: 'dog' | 'cat' | 'other';
  breed: string;
  birthDate: Date;
  weight: number;
  height: number;
  gender: 'male' | 'female';
  isNeutered: boolean;
  avatar?: string;
  createdAt: Date;
  updatedAt: Date;
}
```

### HealthRecord
```javascript
interface HealthRecord {
  _id: ObjectId;
  petId: ObjectId;
  date: Date;
  weight?: number;
  height?: number;
  activity?: {
    type: string;
    duration: number; // minutes
    intensity: 'low' | 'medium' | 'high';
  };
  calories?: number;
  notes?: string;
  createdAt: Date;
}
```

### FoodScan
```javascript
interface FoodScan {
  _id: ObjectId;
  petId: ObjectId;
  userId: ObjectId;
  imageUrl: string;
  recognizedFood?: string;
  nutritionalInfo?: {
    calories: number;
    protein: number;
    carbs: number;
    fat: number;
  };
  confidence: number;
  createdAt: Date;
}
```

## APIs Principais

### Authentication
- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/verify-email`
- `POST /api/auth/forgot-password`
- `POST /api/auth/reset-password`

### Pets
- `GET /api/pets` - Lista pets do usuário
- `POST /api/pets` - Cadastra novo pet
- `PUT /api/pets/:id` - Atualiza pet
- `DELETE /api/pets/:id` - Remove pet

### Health
- `GET /api/pets/:id/health` - Histórico de saúde
- `POST /api/pets/:id/health` - Adiciona registro
- `PUT /api/health/:id` - Atualiza registro

### Food Scanning
- `POST /api/food/scan` - Escaneia alimento
- `GET /api/food/history` - Histórico de escaneamentos

### Admin
- `GET /api/admin/users` - Lista usuários
- `GET /api/admin/analytics` - Métricas gerais
- `POST /api/admin/notifications` - Envia notificação
- `GET /api/admin/reports` - Gera relatórios

## Configurações de Desenvolvimento

### Variáveis de Ambiente
```bash
# Backend
PORT=3000
NODE_ENV=development
DATABASE_URL=mongodb://localhost:27017/oipet
JWT_SECRET=your-secret-key
REDIS_URL=redis://localhost:6379

# Railway
RAILWAY_TOKEN=your-railway-token
MONGODB_URI=mongodb://[railway-mongo-url]
REDIS_URL=redis://[railway-redis-url]

# AWS/Cloud (opcional)
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_BUCKET_NAME=oipet-files

# Email
SENDGRID_API_KEY=your-sendgrid-key
FROM_EMAIL=noreply@oipet.com

# Push Notifications
FCM_SERVER_KEY=your-fcm-key
```

### Scripts de Desenvolvimento
```bash
# Backend
npm run dev          # Inicia servidor de desenvolvimento
npm run build        # Build para produção
npm run test         # Executa testes
npm run seed         # Popula banco com dados de teste

# Mobile
npm run android      # Inicia app Android
npm run ios          # Inicia app iOS
npm run build        # Build para produção

# Web
npm run dev          # Inicia servidor de desenvolvimento
npm run build        # Build para produção
npm run preview      # Preview do build

# Railway
railway login        # Login no Railway
railway deploy       # Deploy manual
railway logs         # Visualizar logs
railway shell        # Shell no container
```

## Convenções de Código

### Nomenclatura
- **Arquivos**: kebab-case (user-profile.component.tsx)
- **Componentes**: PascalCase (UserProfile)
- **Funções**: camelCase (getUserProfile)
- **Constantes**: UPPER_SNAKE_CASE (API_BASE_URL)

### Commits
Usar Conventional Commits:
```
feat: adiciona funcionalidade de escaneamento de alimentos
fix: corrige bug no login de usuários
docs: atualiza documentação da API
style: ajusta cores do tema
test: adiciona testes para módulo de pets
```

### Branches
- `main`: Produção
- `develop`: Desenvolvimento
- `feature/nome-da-feature`: Novas funcionalidades
- `hotfix/nome-do-fix`: Correções urgentes

## Testes

### Estrutura de Testes
```
tests/
├── unit/          # Testes unitários
├── integration/   # Testes de integração
├── e2e/          # Testes end-to-end
└── fixtures/     # Dados de teste
```

### Cobertura Mínima
- **Backend**: 80%
- **Frontend**: 70%
- **Mobile**: 60%

## Deployment

### Ambientes
- **Development**: Branch develop
- **Staging**: Branch release/*
- **Production**: Branch main

### CI/CD Pipeline
1. **Testes**: Executa testes automatizados
2. **Build**: Gera artefatos de produção
3. **Deploy**: Deployment automático por ambiente
4. **Monitoring**: Logs e métricas de saúde

## Integrações Externas

### E-commerce OiPet
- **Base URL**: https://oipetcomidadeverdade.com.br
- **Endpoints**:
  - Produtos para cachorros: `/para-cachorros`
  - Petiscos: `/petiscos.html`
- **Redirecionamento**: Deep linking para carrinho

### Reconhecimento de Imagem
- **Serviço**: TensorFlow Lite ou AWS Rekognition
- **Modelos**: Alimentos naturais para pets
- **Confidence**: Mínimo 70% para auto-classificação

## Segurança

### Autenticação
- JWT com refresh tokens
- Expiração: 1h (access) / 30d (refresh)
- Rate limiting: 100 requests/min

### Dados Sensíveis
- Criptografia de senhas com bcrypt
- Validação de entrada em todas as APIs
- Sanitização de dados para XSS

### LGPD Compliance
- Consentimento explícito para coleta de dados
- Direito de exclusão e portabilidade
- Logs de auditoria para acesso a dados

## Monitoramento

### Métricas de Negócio
- Usuários ativos (DAU/MAU)
- Retenção de usuários
- Conversão para e-commerce
- Engajamento com notificações

### Métricas Técnicas
- Tempo de resposta das APIs
- Uptime do sistema
- Taxa de erro
- Uso de recursos (CPU/Memory)

## Considerações Especiais

### Performance
- Lazy loading de imagens
- Paginação em listas extensas
- Cache de dados frequentemente acessados
- Otimização de queries do banco

### Offline
- Cache local de dados essenciais
- Sincronização quando voltar online
- Indicadores visuais de status de conexão

### Acessibilidade
- Suporte a leitores de tela
- Contraste adequado nas cores
- Navegação por teclado
- Textos alternativos em imagens

## Roadmap de Funcionalidades

### Fase 1 (MVP - 3 meses)
- [ ] Autenticação e cadastro
- [ ] CRUD de pets
- [ ] Monitoramento básico de saúde
- [ ] Painel administrativo essencial
- [ ] Integração com e-commerce

### Fase 2 (2 meses)
- [ ] Escaneamento de alimentos
- [ ] Gráficos e relatórios
- [ ] Notificações push
- [ ] Analytics avançados

### Fase 3 (2 meses)
- [ ] IA para recomendações
- [ ] Gamificação
- [ ] Integração com veterinários
- [ ] Marketplace de serviços

## Protocolo de Trabalho para Claude Code

### 🔄 Fluxo Obrigatório para Cada Sessão

**SEMPRE execute estas etapas no INÍCIO de toda nova conversa:**

1. **📋 LEIA O PLANNING.md**
   - Localize e leia completamente o arquivo `PLANNING.md` na raiz do projeto
   - Entenda o contexto atual do desenvolvimento
   - Identifique a fase atual do projeto e próximos marcos

2. **✅ VERIFIQUE O TASKS.md**
   - Abra e analise o arquivo `TASKS.md` antes de iniciar qualquer trabalho
   - Identifique tasks pendentes, em progresso e concluídas
   - Priorize tasks baseadas no planejamento atual

3. **🎯 EXECUTE O TRABALHO**
   - Foque nas tasks de maior prioridade
   - Siga as convenções e padrões definidos neste documento
   - Mantenha qualidade e consistência do código

4. **✅ MARQUE TASKS CONCLUÍDAS**
   - **IMEDIATAMENTE** após finalizar uma task corretamente, marque como concluída no `TASKS.md`
   - Use o formato: `- [x] Task concluída - [Data] - [Breve descrição do que foi feito]`
   - Seja específico sobre o que foi realizado

5. **📝 ADICIONE NOVOS TASKS**
   - **SEMPRE** que descobrir novas necessidades durante o desenvolvimento, adicione ao `TASKS.md`
   - Categorize adequadamente (Bug, Feature, Refactor, etc.)
   - Inclua prioridade e descrição clara

### 📂 Estrutura dos Arquivos de Controle

```
projeto/
├── CLAUDE.md       # Este arquivo - Guia principal
├── PLANNING.md     # Planejamento atual e roadmap
├── TASKS.md        # Lista de tasks e status
└── ... (resto do projeto)
```

### 🏷️ Formato para TASKS.md

```markdown
# Tasks do Projeto OiPet Saúde

## 🔥 Alta Prioridade
- [ ] Task importante 1
- [ ] Task importante 2

## 📋 Média Prioridade  
- [ ] Task média 1
- [x] Task concluída - [15/07/2025] - Implementado sistema de auth

## 🔧 Baixa Prioridade
- [ ] Task baixa 1

## 🐛 Bugs
- [ ] Bug crítico 1

## 🆕 Novas Descobertas
- [ ] Task descoberta durante desenvolvimento
```

### ⚠️ Regras Importantes

1. **NUNCA** comece a trabalhar sem ler `PLANNING.md` e `TASKS.md`
2. **SEMPRE** marque tasks como concluídas imediatamente quando terminadas
3. **NUNCA** deixe tasks descobertas sem documentar
4. **SEMPRE** siga as convenções de código definidas neste documento
5. **MANTENHA** os arquivos `PLANNING.md` e `TASKS.md` atualizados

### 🎯 Objetivos do Protocolo

- **Continuidade**: Manter contexto entre sessões
- **Produtividade**: Evitar retrabalho e confusão
- **Qualidade**: Garantir padrões consistentes
- **Rastreabilidade**: Documentar progresso e decisões
- **Eficiência**: Maximizar tempo de desenvolvimento

---

**Última atualização**: 15/07/2025  
**Versão**: 1.1  
**Mantenedor**: Equipe OiPet

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Telmobelsuzarri)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Telmobelsuzarri)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
