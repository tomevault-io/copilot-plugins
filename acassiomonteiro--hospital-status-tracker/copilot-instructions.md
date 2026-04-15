## hospital-status-tracker

> Sistema web MVP para rastreamento em tempo real do status de pacientes no pronto-socorro do Hospital Santa Helena Norte. Projeto acadêmico (PBL) focado em resolver fragmentação de informações clínicas.

# Hospital Status Tracker - Sistema de Rastreamento de Pacientes

## Visão Geral
Sistema web MVP para rastreamento em tempo real do status de pacientes no pronto-socorro do Hospital Santa Helena Norte. Projeto acadêmico (PBL) focado em resolver fragmentação de informações clínicas.

---

## Stack Tecnológica
- **Backend**: Django 5.2.7 + Python 3.11
- **Frontend**: Django Templates + Tailwind CSS 3.x (via CDN)
- **Banco de Dados**: PostgreSQL (com Docker Compose)
- **Autenticação**: Django Auth nativo (User + Profissional)
- **Deploy**: Docker + Docker Compose

---

## Estrutura de Arquivos
```
hospital-status-tracker/
├── manage.py
├── CLAUDE.md                    # Este arquivo
├── requirements.txt             # Dependências Python
├── Dockerfile                   # Container da aplicação
├── docker-compose.yml           # Orquestração de serviços
├── entrypoint.sh                # Script de inicialização (migrations automáticas)
├── core/                        # Configuração Django
│   ├── settings.py             # Configurações principais (PostgreSQL)
│   ├── urls.py                 # URL raiz (inclui urls dos 4 apps)
│   ├── wsgi.py
│   └── asgi.py
├── pacientes/                   # App de Pacientes
│   ├── models.py               # Model Paciente (prontuário eletrônico)
│   ├── forms.py                # PacienteForm
│   ├── admin.py                # PacienteAdmin
│   ├── apps.py
│   └── migrations/
├── usuarios/                    # App de Usuários e Autenticação
│   ├── models.py               # Model Profissional
│   ├── views.py                # CustomLoginView, CustomLogoutView
│   ├── urls.py                 # Rotas: /login/, /logout/
│   ├── admin.py                # ProfissionalAdmin
│   ├── apps.py
│   ├── migrations/
│   └── templates/
│       └── registration/
│           └── login.html      # Template de login
├── atendimentos/                # App de Atendimentos
│   ├── models.py               # Model Atendimento
│   ├── views.py                # DashboardView, NovoAtendimentoView, AtualizarStatusView
│   ├── urls.py                 # Rotas: /, /novo/, /atualizar/<id>/
│   ├── forms.py                # AtendimentoForm
│   ├── admin.py                # AtendimentoAdmin
│   ├── apps.py
│   ├── migrations/
│   └── templates/
│       └── atendimento/
│           ├── base.html       # Template base com Tailwind + Navbar
│           ├── dashboard.html  # Lista de atendimentos
│           ├── novo_atendimento.html
│           └── atualizar_status.html
├── prontuario/                  # App de Prontuário (Evoluções, Sinais Vitais, Prescrições e Exames)
│   ├── models.py               # Models: Evolucao, SinalVital, Prescricao, ItemPrescricao, SolicitacaoExame, ResultadoExame
│   ├── views.py                # Views para evoluções, sinais vitais, prescrições e exames
│   ├── urls.py                 # Rotas: evolucoes, sinais-vitais, prescricoes, exames
│   ├── forms.py                # Forms: EvolucaoForm, SinalVitalForm, PrescricaoForm, ItemPrescricaoFormSet, SolicitacaoExameForm, ResultadoExameForm
│   ├── admin.py                # Admin: EvolucaoAdmin, SinalVitalAdmin, PrescricaoAdmin, SolicitacaoExameAdmin, ResultadoExameAdmin
│   ├── apps.py
│   ├── migrations/
│   └── templates/
│       ├── atendimento/
│       │   ├── base.html       # Template base (compartilhado)
│       │   ├── nova_evolucao.html
│       │   └── evolucoes_atendimento.html
│       └── prontuario/
│           ├── novo_sinal_vital.html
│           ├── sinais_vitais_atendimento.html
│           ├── nova_prescricao.html
│           ├── prescricoes_atendimento.html
│           ├── nova_solicitacao_exame.html
│           ├── solicitacoes_exame_atendimento.html
│           └── adicionar_resultado_exame.html
├── media/                      # Uploads de arquivos (laudos de exames)
└── staticfiles/                # Arquivos estáticos coletados
```

**Arquitetura Modular:**
O projeto foi refatorado de um app monolítico (`atendimento`) para **4 apps especializados** por domínio:
- **`pacientes`**: Gerencia dados dos pacientes (prontuário eletrônico)
- **`usuarios`**: Gerencia profissionais e autenticação
- **`atendimentos`**: Gerencia fluxo de atendimentos no pronto-socorro
- **`prontuario`**: Gerencia evoluções clínicas e registros médicos


---

## Arquitetura e Relacionamentos

### Diagrama de Dependências
```
usuarios (Profissional)
    ↓
pacientes (Paciente)    →    atendimentos (Atendimento)    →    prontuario (Evolucao)
```

**Relacionamentos entre Models:**
- `Atendimento` → `Paciente` (ForeignKey)
- `Atendimento` → `Profissional` (ForeignKey, opcional)
- `Evolucao` → `Atendimento` (ForeignKey)
- `Evolucao` → `Profissional` (ForeignKey)
- `SinalVital` → `Atendimento` (ForeignKey)
- `SinalVital` → `Profissional` (ForeignKey)
- `Prescricao` → `Atendimento` (ForeignKey)
- `Prescricao` → `Profissional` (ForeignKey)
- `ItemPrescricao` → `Prescricao` (ForeignKey, CASCADE)
- `SolicitacaoExame` → `Atendimento` (ForeignKey)
- `SolicitacaoExame` → `Profissional` (ForeignKey)
- `ResultadoExame` → `SolicitacaoExame` (OneToOneField)

**Fluxo de Dados:**
1. **Cadastro**: Profissional → Paciente → Atendimento
2. **Acompanhamento**: Atendimento → Múltiplas Evoluções + Múltiplos Sinais Vitais + Múltiplas Prescrições + Múltiplas Solicitações de Exames
3. **Auditoria**: Todas as ações rastreáveis pelo Profissional
4. **Segregação**: Prescrições e Solicitações de Exames somente por perfil MEDICO
5. **Upload**: ResultadoExame permite upload de laudos (PDF, imagens)

**Configuração em settings.py:**
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'pacientes',      # Base de dados de pacientes
    'usuarios',       # Autenticação e perfis profissionais
    'atendimentos',   # Fluxo de atendimentos
    'prontuario',     # Registros clínicos
]
```

---

## Models Atuais

### Paciente ✅ Expandido (Prontuário Completo)

**Dados Básicos (obrigatórios):**
- `nome`: CharField (200 chars)
- `cpf`: CharField (11 chars, unique, validado com regex)
- `data_nascimento`: DateField
- `criado_em`: DateTimeField (auto)
- `atualizado_em`: DateTimeField (auto)

**Dados Pessoais (opcionais):**
- `sexo`: CharField (choices: M, F, O)
- `nome_mae`: CharField (200 chars)
- `telefone`: CharField (10-11 dígitos, validado)
- `email`: EmailField

**Documentos (opcionais):**
- `cartao_sus`: CharField (15 dígitos, validado)
- `rg`: CharField (20 chars)

**Endereço (opcionais):**
- `cep`: CharField (8 dígitos, validado)
- `rua`: CharField (200 chars)
- `numero`: CharField (10 chars)
- `bairro`: CharField (100 chars)
- `cidade`: CharField (100 chars)
- `uf`: CharField (2 chars, validado para maiúsculas)

**Dados Clínicos (opcionais):**
- `tipo_sanguineo`: CharField (choices: A+, A-, B+, B-, AB+, AB-, O+, O-)
- `alergias`: TextField
- `observacoes_clinicas`: TextField

**Métodos:**
- `get_endereco_completo()`: Retorna endereço formatado

### Profissional ✅ Implementado
- `user`: OneToOneField → User (CASCADE)
- `perfil`: CharField (choices: MEDICO, ENFERMEIRO, ADMINISTRATIVO)
- `registro_profissional`: CharField (50 chars, opcional - CRM, COREN, etc.)
- `criado_em`: DateTimeField (auto)

**Perfis disponíveis:**
1. MEDICO
2. ENFERMEIRO
3. ADMINISTRATIVO

### Atendimento
- `paciente`: ForeignKey → Paciente (PROTECT)
- `profissional_responsavel`: ForeignKey → Profissional (PROTECT, opcional)
- `data_hora_entrada`: DateTimeField (auto)
- `queixa`: TextField
- `status`: CharField com 7 choices (default: TRIAGEM)
- `atualizado_em`: DateTimeField (auto)

**Status disponíveis:**
1. TRIAGEM
2. EM_ATENDIMENTO
3. AGUARDANDO_EXAME
4. EM_EXAME
5. AGUARDANDO_RESULTADO
6. ALTA
7. INTERNACAO

**Métodos:**
- `get_status_badge_class()`: Retorna classe Tailwind por status

### Evolucao ✅ Implementado
- `atendimento`: ForeignKey → Atendimento (PROTECT)
- `profissional`: ForeignKey → Profissional (PROTECT)
- `tipo`: CharField (choices: ANAMNESE, EVOLUCAO_MEDICA, EVOLUCAO_ENFERMAGEM, EXAME_FISICO)
- `descricao`: TextField
- `data_hora`: DateTimeField (auto_now_add)

**Tipos de Evolução disponíveis:**
1. ANAMNESE
2. EVOLUCAO_MEDICA
3. EVOLUCAO_ENFERMAGEM
4. EXAME_FISICO

**Métodos:**
- `get_tipo_badge_class()`: Retorna classe Tailwind por tipo de evolução

### SinalVital ✅ Implementado
- `atendimento`: ForeignKey → Atendimento (PROTECT)
- `profissional`: ForeignKey → Profissional (PROTECT)
- `pressao_arterial_sistolica`: PositiveSmallIntegerField (50-300 mmHg, opcional)
- `pressao_arterial_diastolica`: PositiveSmallIntegerField (30-200 mmHg, opcional)
- `frequencia_cardiaca`: PositiveSmallIntegerField (30-250 bpm, opcional)
- `frequencia_respiratoria`: PositiveSmallIntegerField (8-60 irpm, opcional)
- `temperatura`: DecimalField (32.0-45.0 °C, opcional)
- `saturacao_o2`: PositiveSmallIntegerField (50-100%, opcional)
- `glicemia`: PositiveSmallIntegerField (20-600 mg/dL, opcional)
- `observacoes`: TextField (opcional)
- `data_hora`: DateTimeField (auto_now_add)

**Validações:**
- Ao menos um sinal vital deve ser preenchido
- Pressão sistólica > diastólica (validação de coerência)
- Ranges clínicos realistas com MinValueValidator/MaxValueValidator

**Métodos:**
- `get_pressao_arterial()`: Retorna pressão formatada (ex: "120/80")
- `tem_sinais_alterados()`: Lista alertas automáticos baseados em parâmetros normais
  - Pressão elevada (>140/90)
  - Bradicardia (<60 bpm) / Taquicardia (>100 bpm)
  - Hipotermia (<36°C) / Febre (>37.5°C)
  - Saturação baixa (<95%)

### Prescricao ✅ Implementado
- `atendimento`: ForeignKey → Atendimento (PROTECT)
- `profissional`: ForeignKey → Profissional (PROTECT)
- `data_prescricao`: DateTimeField (auto_now_add)
- `validade`: DateField
- `status`: CharField (choices: ATIVA, SUSPENSA, CONCLUIDA, default: ATIVA)
- `observacoes`: TextField (opcional)

**Status disponíveis:**
1. ATIVA
2. SUSPENSA
3. CONCLUIDA

**Métodos:**
- `get_status_badge_class()`: Retorna classe Tailwind por status
- `total_itens()`: Retorna total de medicamentos prescritos

**Restrições:**
- **Apenas perfil MEDICO pode criar prescrições**
- Validação no método `dispatch()` da view `NovaPrescricaoView`
- Botão de nova prescrição visível apenas para médicos

### ItemPrescricao ✅ Implementado
- `prescricao`: ForeignKey → Prescricao (CASCADE)
- `medicamento`: CharField (200 chars)
- `dose`: CharField (100 chars)
- `via`: CharField (choices com 10 opções)
- `frequencia`: CharField (100 chars)
- `duracao_dias`: PositiveSmallIntegerField (1-365)
- `observacoes_item`: TextField (opcional)

**Vias de Administração disponíveis:**
1. ORAL
2. INTRAVENOSA (IV)
3. INTRAMUSCULAR (IM)
4. SUBCUTANEA (SC)
5. TOPICA
6. INALATORIA
7. SUBLINGUAL
8. RETAL
9. OCULAR
10. NASAL

**Validações:**
- Mínimo de 1 medicamento por prescrição (via FormSet)
- Duração entre 1 e 365 dias
- Validade não pode ser anterior à data atual

### SolicitacaoExame ✅ Implementado
- `atendimento`: ForeignKey → Atendimento (PROTECT)
- `profissional`: ForeignKey → Profissional (PROTECT)
- `tipo`: CharField (choices: LABORATORIO, IMAGEM, CARDIOLOGIA, ANATOMIA_PATOLOGICA, OUTRO)
- `nome_exame`: CharField (200 chars)
- `justificativa`: TextField
- `status`: CharField (choices: SOLICITADO, COLETADO, RESULTADO_DISPONIVEL, CANCELADO, default: SOLICITADO)
- `data_solicitacao`: DateTimeField (auto_now_add)
- `data_atualizacao`: DateTimeField (auto_now)

**Tipos de Exame disponíveis:**
1. LABORATORIO
2. IMAGEM
3. CARDIOLOGIA
4. ANATOMIA_PATOLOGICA
5. OUTRO

**Status disponíveis:**
1. SOLICITADO
2. COLETADO
3. RESULTADO_DISPONIVEL
4. CANCELADO

**Métodos:**
- `get_status_badge_class()`: Retorna classe Tailwind por status
- `get_tipo_badge_class()`: Retorna classe Tailwind por tipo
- `tem_resultado()`: Verifica se a solicitação possui resultado registrado

**Restrições:**
- **Apenas perfil MEDICO pode solicitar exames**
- **Apenas perfil MEDICO pode cancelar solicitações**
- Não é possível cancelar solicitação que já possui resultado
- Status atualizado automaticamente para RESULTADO_DISPONIVEL ao adicionar resultado

### ResultadoExame ✅ Implementado
- `solicitacao`: OneToOneField → SolicitacaoExame (PROTECT)
- `resultado_texto`: TextField
- `arquivo_laudo`: FileField (upload_to='laudos/%Y/%m/', opcional)
- `data_resultado`: DateTimeField (auto_now_add)
- `observacoes`: TextField (opcional)

**Upload de Arquivos:**
- Tipos aceitos: PDF, JPG, JPEG, PNG
- Tamanho máximo: 10MB
- Path de upload: `media/laudos/YYYY/MM/`
- Validação em dupla camada (form + model)

**Configuração de Media:**
```python
# settings.py
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# urls.py (desenvolvimento)
urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

---

## Padrões de Código

### Geral
- **Idioma**: Nomes de models, campos e variáveis em português
- **Style Guide**: PEP 8 (linhas até 100 chars)
- **Imports**: Ordem padrão Django (stdlib → django → terceiros → local)
- **Imports entre Apps**: Usar caminhos absolutos para importar de outros apps
  ```python
  # Correto
  from pacientes.models import Paciente
  from usuarios.models import Profissional

  # Incorreto
  from ..pacientes.models import Paciente
  ```

### Models
- Sempre usar `verbose_name` em português
- ForeignKeys com `on_delete=models.PROTECT` (segurança de dados)
- `related_name` descritivo quando apropriado
- Método `__str__()` obrigatório

### Views
- **Preferir CBVs** (Class-Based Views): ListView, FormView, DetailView
- ✅ **Todas as views protegidas com `LoginRequiredMixin`**
- `select_related()` e `prefetch_related()` para otimizar queries
- Messages framework para feedback ao usuário
- **Views de autenticação**: `CustomLoginView`, `CustomLogoutView`
- **Lógica de negócio**: `get_or_create` para evitar pacientes duplicados

### Forms
- Validação no Form E no Model (dupla camada)
- `clean_<campo>()` para validações customizadas
- Labels em português
- **PacienteForm expandido**: Todos os campos do prontuário
- **Validações automáticas**: Remove caracteres não numéricos (CPF, telefone, CEP, SUS)
- **Widgets customizados**: Classes Tailwind CSS aplicadas

### Templates
- **Herança**: Todos herdam de `base.html`
- **Estilo**: Tailwind CSS inline (classes diretas no HTML)
- **Mobile-first**: Design responsivo prioritário
- **Acessibilidade**: Usar tags semânticas (header, nav, main, footer)

---

## Convenções Específicas

### Validações Implementadas
```python
# CPF: RegexValidator(regex=r'^\d{11}$') + clean_cpf() remove não numéricos
# Telefone: RegexValidator(regex=r'^\d{10,11}$') + clean_telefone()
# CEP: RegexValidator(regex=r'^\d{8}$') + clean_cep()
# Cartão SUS: RegexValidator(regex=r'^\d{15}$') + clean_cartao_sus()
# UF: RegexValidator(regex=r'^[A-Z]{2}$') + clean_uf() converte para maiúsculas
```

### Feedback ao Usuário
```python
from django.contrib import messages
messages.success(request, 'Atendimento criado com sucesso!')
```

### Queries Otimizadas
```python
# SEMPRE use select_related para ForeignKeys
atendimentos = Atendimento.objects.select_related(
    'paciente', 
    'profissional_responsavel__user'
).all()
```

### Evitar Duplicação de Pacientes
```python
# Usar get_or_create para evitar pacientes duplicados pelo CPF
paciente, created = Paciente.objects.get_or_create(
    cpf=cpf,
    defaults={'nome': nome, 'data_nascimento': data_nascimento}
)
```

### Rotas Implementadas

**Autenticação (app: usuarios):**
- `/login/` - Login de profissionais
- `/logout/` - Logout

**Atendimentos (app: atendimentos):**
- `/` - Dashboard de atendimentos
- `/novo/` - Novo atendimento
- `/atualizar/<id>/` - Atualizar status do atendimento

**Prontuário (app: prontuario):**
- `/atendimento/<id>/evolucoes/` - Timeline de evoluções clínicas
- `/atendimento/<id>/evolucao/nova/` - Registrar nova evolução
- `/atendimento/<id>/sinais-vitais/` - Timeline de sinais vitais
- `/atendimento/<id>/sinais-vitais/novo/` - Registrar novos sinais vitais
- `/atendimento/<id>/prescricoes/` - Timeline de prescrições médicas
- `/atendimento/<id>/prescricao/nova/` - Criar nova prescrição (MÉDICOS APENAS)
- `/atendimento/<id>/exames/` - Timeline de solicitações de exames
- `/atendimento/<id>/exame/solicitar/` - Solicitar novo exame (MÉDICOS APENAS)
- `/exame/<id>/resultado/` - Adicionar resultado a uma solicitação
- `/exame/<id>/cancelar/` - Cancelar solicitação de exame (MÉDICOS APENAS)

**Admin:**
- `/admin/` - Interface administrativa Django

---

## Ambiente Docker

O projeto é **100% dockerizado** com migrations automáticas via `entrypoint.sh`.

### Comandos Docker
```bash
# Iniciar serviços (web + postgres)
docker-compose up

# Rebuild após mudanças
docker-compose down
docker-compose up --build

# Ver logs em tempo real
docker-compose logs -f web

# Acessar shell Django no container
docker-compose exec web python manage.py shell

# Criar superusuário (se necessário)
docker-compose exec web python manage.py createsuperuser

# Acessar banco PostgreSQL
docker-compose exec db psql -U hospital_admin -d hospital_db
```

**IMPORTANTE:** 
- Migrations são aplicadas **AUTOMATICAMENTE** pelo `entrypoint.sh` ao subir o container
- Após modificar models, basta fazer: `docker-compose down && docker-compose up --build`
- **NÃO** é necessário rodar `makemigrations` ou `migrate` manualmente

---

## Regras para Novas Funcionalidades

### ✅ Sempre Fazer
- [ ] Criar migration após alterar models
- [ ] Testar no navegador antes de commitar
- [ ] Adicionar validações no model E no form
- [ ] Usar messages para feedback
- [ ] Manter consistência visual (Tailwind)
- [ ] Documentar mudanças complexas com comentários
- [ ] Proteger views com LoginRequiredMixin
- [ ] Usar select_related/prefetch_related para otimizar queries
- [ ] Testar com Docker antes de fazer commit

### ❌ Nunca Fazer
- ❌ Quebrar funcionalidades existentes
- ❌ Remover migrations já aplicadas
- ❌ Usar JavaScript externo (manter simples)
- ❌ Adicionar dependências sem necessidade
- ❌ Hardcode de valores (usar constantes)
- ❌ Expor dados sensíveis no template
- ❌ Commitar variáveis de ambiente (.env) no Git
- ❌ Usar SQLite em produção (usar PostgreSQL)

---

## Roadmap de Desenvolvimento

### ✅ FASE 1: AUTENTICAÇÃO (COMPLETA)
- [x] Model Profissional (OneToOne com User)
- [x] 3 perfis de acesso: Médico, Enfermeiro, Administrativo
- [x] LoginView e LogoutView customizados
- [x] Todas as views protegidas com LoginRequiredMixin
- [x] Template de login responsivo
- [x] Navbar com nome do usuário e perfil
- [x] Vinculação de profissional aos atendimentos

**Problema da PBL resolvido:**
> "Credenciais genéricas compartilhadas por turno, inviabilizando rastreabilidade"

✅ Cada profissional tem login único e ações são rastreáveis.

---

### ✅ FASE 2: DADOS DO PACIENTE (COMPLETA)
- [x] Expandir Paciente com dados pessoais (telefone, email, sexo, nome_mae)
- [x] Adicionar documentação (Cartão SUS, RG)
- [x] Adicionar endereço completo (CEP, rua, número, bairro, cidade, UF)
- [x] Adicionar dados clínicos (tipo sanguíneo, alergias, observações)
- [x] Todos os novos campos opcionais (backward compatibility)
- [x] Validações em todos os campos
- [x] Formulário completo com widgets customizados
- [x] Docker e PostgreSQL configurados

**Problema da PBL resolvido:**
> "Cadastros com grafias distintas, variações de CPF, reaproveitamento"

✅ Identificação mais precisa com múltiplos dados + validações rigorosas.

---

### ✅ FASE 3: EVOLUÇÃO CLÍNICA (COMPLETA)

**Por que é crítica:** É o **coração do prontuário eletrônico**. Sem evoluções clínicas, o sistema é apenas "cadastro + status", não um prontuário real.

- [x] Model Evolucao vinculado a Atendimento e Profissional
- [x] 4 tipos de evolução: ANAMNESE, EVOLUCAO_MEDICA, EVOLUCAO_ENFERMAGEM, EXAME_FISICO
- [x] Campo descricao (TextField) para texto da evolução
- [x] Data/hora automática (auto_now_add)
- [x] Form de registro rápido (tipo + descricao)
- [x] View para adicionar evolução
- [x] View para listar evoluções (timeline cronológica)
- [x] Template com cards coloridos por tipo
- [x] Botão "Ver Evoluções" no dashboard

**Problema da PBL que resolve:**
> "Prontuário existe como combinação de papel, telas do legado e arquivos externos"

✅ Tudo num único lugar digital, com timeline completa do atendimento.

**🔄 Refatoração Arquitetural (Novembro 2025):**
- Projeto refatorado de app monolítico (`atendimento`) para **4 apps especializados** por domínio
- Melhora **organização, escalabilidade e manutenibilidade** do código
- Separação clara de responsabilidades: `pacientes`, `usuarios`, `atendimentos`, `prontuario`
- ✅ **100% dos dados preservados** - apenas reorganização de código
- Tabelas do banco renomeadas para refletir a nova estrutura modular

---

### ✅ FASE 4: SINAIS VITAIS (COMPLETA)

**Por que é crítica:** Digitaliza parâmetros vitais que hoje ficam fragmentados em quadros brancos e planilhas, permitindo acompanhamento temporal estruturado da evolução do paciente.

- [x] Model SinalVital vinculado a Atendimento e Profissional
- [x] Campos: pressao_arterial (sistólica/diastólica), frequencia_cardiaca, frequencia_respiratoria, temperatura, saturacao_o2, glicemia
- [x] Form rápido para profissionais (especialmente enfermagem)
- [x] Listagem cronológica por atendimento (timeline visual)
- [x] Validações de ranges clínicos realistas (dupla camada)
- [x] Alertas automáticos para valores fora dos parâmetros normais
- [x] Integração com dashboard (botão + badge de contagem)
- [x] Template responsivo com ícones coloridos (verde normal, amarelo alerta)

**Problema da PBL que resolve:**
> "Quadros brancos físicos e planilhas produzem instantâneos que se desatualizam"

✅ Sinais vitais digitalizados, timestamped, persistentes e com alertas automáticos.

---

### ✅ FASE 5: PRESCRIÇÕES MÉDICAS (COMPLETA)

**Por que é crítica:** Implementa segregação de acesso por perfil profissional, garantindo que apenas médicos possam prescrever medicamentos, com rastreabilidade completa e verificação automática de alergias.

- [x] Model Prescricao (atendimento, profissional, data_prescricao, validade, status)
- [x] Model ItemPrescricao (medicamento, dose, via, frequência, duração_dias)
- [x] FormSet dinâmico para múltiplos medicamentos
- [x] Restrição de acesso: apenas perfil MEDICO pode criar prescrições
- [x] Timeline de prescrições por atendimento
- [x] Alerta visual destacado quando paciente possui alergias
- [x] Status de prescrição (ATIVA, SUSPENSA, CONCLUIDA)
- [x] Integração com dashboard (botão + badge visível conforme perfil)
- [x] 10 vias de administração disponíveis
- [x] Validações: validade futura, mínimo 1 medicamento

**Problema da PBL que resolve:**
> "Políticas de acesso não refletem necessidade de segregação por perfil"

✅ Apenas médicos podem prescrever, registro auditável com profissional vinculado, alergias destacadas em vermelho durante prescrição.

---

### ✅ FASE 6: EXAMES (COMPLETA)

**Por que é crítica:** Centraliza solicitações e resultados de exames que hoje ficam fragmentados em portais de terceiros, criando nexo causal claro entre solicitação, atendimento e resultado.

- [x] Model SolicitacaoExame (tipo, nome_exame, justificativa, status)
- [x] Model ResultadoExame (resultado_texto, arquivo_laudo, observacoes)
- [x] Upload de laudos em PDF ou imagens (max 10MB)
- [x] Configuração de MEDIA_ROOT e MEDIA_URL para uploads
- [x] Rastreamento completo de solicitações por atendimento
- [x] 5 tipos de exame: Laboratório, Imagem, Cardiologia, Anatomia Patológica, Outro
- [x] 4 status: Solicitado, Coletado, Resultado Disponível, Cancelado
- [x] Restrição de acesso: apenas perfil MEDICO pode solicitar e cancelar
- [x] Qualquer profissional pode adicionar resultado
- [x] Timeline visual de solicitações com badges coloridos por status
- [x] Visualização de laudos anexados
- [x] Validações: tipo de arquivo (PDF/imagens), tamanho (10MB max)
- [x] Atualização automática de status ao adicionar resultado
- [x] Integração com dashboard (botão + badge de contagem)

**Problema da PBL que resolve:**
> "Resultados disponibilizados em portais de terceiros, notificações por email genérico"

✅ Centraliza solicitações e resultados, mesmo que laudos venham de fora há vínculo claro com atendimento, profissional solicitante e histórico completo.

---

### ✅ FASE 7: PRONTUÁRIO COMPLETO (COMPLETA - PROJETO FINALIZADO)

**Por que é crítica:** Elimina a fragmentação informacional consolidando todas as informações clínicas em uma única timeline cronológica, eliminando a necessidade de navegar entre múltiplas telas para visualizar o histórico completo do paciente.

- [x] View consolidada ProntuarioCompletoView mostrando TODOS os registros
- [x] Timeline cronológica completa ordenada por data/hora
- [x] Agregação de Evoluções, Sinais Vitais, Prescrições e Exames
- [x] Cards visuais diferenciados por tipo de registro
- [x] Botão destacado "📋 Prontuário Completo" no dashboard
- [x] Queries otimizadas com select_related e prefetch_related
- [x] Estatísticas de totais por categoria no topo
- [x] Alerta visual de alergias no topo
- [x] Informações do atendimento e paciente no cabeçalho
- [x] Design responsivo mobile-first
- [x] Partials reutilizáveis para cada tipo de card

**Problema da PBL que resolve:**
> "Ecossistema informacional fragmentado - prontuário como combinação de papel, telas do legado e arquivos externos"

✅ **ECOSSISTEMA COMPLETAMENTE UNIFICADO** - Todas as informações clínicas em uma única view consolidada, acessível com um clique, eliminando 100% da fragmentação informacional!

---

## Estado Atual do Projeto

### ✅ Funcionalidades Implementadas

**FASE 1 - Autenticação:** ✅ COMPLETA
- Sistema de login/logout funcional
- 3 perfis de usuário implementados
- Todas as views protegidas
- Navbar com informações do usuário

**FASE 2 - Dados do Paciente:** ✅ COMPLETA
- Prontuário completo com 20+ campos
- Validações robustas em todos os campos
- Formulário responsivo e organizado
- Infraestrutura Docker + PostgreSQL

**FASE 3 - Evolução Clínica:** ✅ COMPLETA
- Registro de evoluções clínicas por profissionais
- Timeline cronológica completa por atendimento
- Distinção visual para tipos de evolução (Médica, Enfermagem, etc.)

**FASE 4 - Sinais Vitais:** ✅ COMPLETA
- Registro de sinais vitais (PA, FC, FR, Temp, SpO₂, Glicemia)
- Validações clínicas rigorosas com ranges realistas
- Alertas automáticos para valores alterados
- Timeline visual com ícones coloridos
- Integração completa com dashboard

**FASE 5 - Prescrições Médicas:** ✅ COMPLETA
- Prescrição de múltiplos medicamentos por atendimento
- **Restrição de acesso: apenas perfil MEDICO**
- Alerta visual de alergias destacado durante prescrição
- FormSet dinâmico (adicionar/remover medicamentos)
- Timeline de prescrições com status (ativa/suspensa/concluída)
- 10 vias de administração disponíveis
- Integração com dashboard (botão visível apenas para médicos)

**FASE 6 - Exames:** ✅ COMPLETA
- Solicitação de exames laboratoriais e de imagem
- **Restrição de acesso: apenas perfil MEDICO pode solicitar**
- Upload de laudos (PDF, imagens até 10MB)
- Timeline de solicitações com status
- 5 tipos de exame (Laboratório, Imagem, Cardiologia, etc.)
- 4 status (Solicitado, Coletado, Resultado Disponível, Cancelado)
- Qualquer profissional pode adicionar resultado
- Integração com dashboard (botão + badge laranja)
- Configuração de MEDIA para uploads

**FASE 7 - Prontuário Completo:** ✅ COMPLETA
- Timeline cronológica unificada com todos os registros clínicos
- Agregação de Evoluções, Sinais Vitais, Prescrições e Exames
- Cards visuais diferenciados por tipo (cores: azul, roxo, índigo, laranja)
- Queries otimizadas para evitar N+1 (select_related/prefetch_related)
- Estatísticas de totais por categoria no topo
- Alerta destacado de alergias do paciente
- Botão "📋 Prontuário Completo" em destaque no dashboard
- Eliminação total da fragmentação informacional

### 🎉 PROJETO CONCLUÍDO

**Todas as 7 fases do roadmap foram implementadas com sucesso!**

O sistema agora oferece um **Prontuário Eletrônico Integrado completo**, resolvendo todos os problemas identificados na PBL:
- ✅ Autenticação individual com rastreabilidade
- ✅ Cadastro completo de pacientes com validações
- ✅ Evoluções clínicas digitalizadas
- ✅ Sinais vitais com alertas automáticos
- ✅ Prescrições médicas com controle de acesso
- ✅ Solicitações e resultados de exames centralizados
- ✅ **Timeline unificada eliminando fragmentação informacional**

---

## Contexto Importante para IA

### Quando Modificar Código
1. **Leia o arquivo completo** antes de modificar
2. **Preserve funcionalidades existentes** - não remova código funcionando
3. **Siga os padrões estabelecidos** - mantenha consistência
4. **Teste mentalmente** - verifique se não vai quebrar nada
5. **Mantenha validações** - não remova validações de campos

### Quando Criar Novas Features
1. **Comece pelos Models** - estrutura de dados primeiro
2. **Depois Forms** - validação e interface com usuário
3. **Depois Views** - lógica de negócio (com LoginRequiredMixin)
4. **Por último Templates** - apresentação

### Segurança e Boas Práticas
- **Nunca** expor senhas ou tokens no código
- **Sempre** validar entrada do usuário (model + form)
- **Sempre** usar CSRF protection (Django faz automaticamente)
- **Sempre** escapar output no template (Django faz automaticamente)
- **Sempre** proteger views com LoginRequiredMixin
- **Sempre** usar variáveis de ambiente para configurações sensíveis

### Docker e Ambiente
- **Migrations automáticas:** `entrypoint.sh` cuida disso
- **Não rodar comandos manuais:** Docker gerencia tudo
- **Rebuild após mudanças:** `docker-compose down && docker-compose up --build`
- **PostgreSQL em produção:** Nunca usar SQLite

---

## Links Úteis
- Django Docs: https://docs.djangoproject.com/en/5.2/
- Tailwind CSS: https://tailwindcss.com/docs
- PEP 8: https://pep8.org/
- Docker Docs: https://docs.docker.com/
- PostgreSQL Docs: https://www.postgresql.org/docs/

---

## Notas do Projeto
- **Objetivo acadêmico**: PBL do 5º período - Programação para Web
- **Professor**: Esp. Guilherme Ferreira dos Reis
- **Prazo final**: 27 de novembro de 2025
- **Repositório**: https://github.com/acassiomonteiro/hospital-status-tracker
- **Aluno**: Acássio Monteiro

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/acassiomonteiro)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/acassiomonteiro)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
