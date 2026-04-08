## flutter-dashboard-ui

> Sistema de Gestão Fiscal desenvolvido em Flutter para a **Domani Fiscal**. Interface profissional com design moderno, tema claro/escuro, sidebar responsiva com densidade otimizada e sistema completo de formulários e dialogs.

# Contexto do Projeto - Domani Fiscal

## Visão Geral

Sistema de Gestão Fiscal desenvolvido em Flutter para a **Domani Fiscal**. Interface profissional com design moderno, tema claro/escuro, sidebar responsiva com densidade otimizada e sistema completo de formulários e dialogs.

## Diretrizes Essenciais

- **Idioma:** Português do Brasil (pt-BR) em todo código, comentários e interface
- **Nomenclatura:** Funções, variáveis e componentes em português
- **Branding:** Sempre usar "Domani Fiscal" (nunca "Dashboard UI")
- **Paleta de Cores Atualizada (Outubro 2025):**
  - **Modo Light:**
    - **Destaque:** `#007BFF` (Azul primário)
    - **Fundo Principal:** `#EDF1F5` (Cinza suave para conforto visual)
    - **Fundo Sidebar:** `#F5F7FA` (Tom intermediário para separação)
    - **Cards:** `#FFFFFF` (Branco para áreas de conteúdo)
  - **Modo Dark:**
    - **Destaque:** `#5B9CF8`
    - **Fundo Principal:** `#121212`
    - **Cards:** `#1E1E1E`

## Arquitetura Atual (Refatorada - Outubro 2025)

### 🏗️ Separação de Responsabilidades

```
lib/
├── services/                           # 🔧 ESPECÍFICOS DO PROJETO
│   ├── README.md                       # Documentação da arquitetura
│   └── dialog/
│       ├── form_dialog_service.dart    # Serviços específicos do projeto
│       └── domani_dialog_callbacks.dart # Callbacks específicos do projeto
├── widgets/                            # 🎨 GENÉRICOS E REUTILIZÁVEIS PARA QUALQUER PROJETO NOVO
│   ├── README.md (documentação completa)
│   ├── app_initializer.dart, app_layout.dart, dialog.dart
│   └── src/
│       ├── app_layout/ (app_shell.dart, responsive_drawer.dart, drawer_list_item.dart)
│       ├── dialog/ (form_dialog.dart, dialog_service.dart, dialog_callbacks.dart)
│       └── forms/ (inputs/, layout/, validators/, formatters/)
├── config/sidebar_config.dart          # Usa DomaniDialogCallbacks
├── screens/
│   ├── splash_screen.dart, dashboard_screen.dart
│   └── em_construcao_screen.dart, teste_screen.dart
├── theme/
│   ├── theme.dart (paleta completa), theme_manager.dart
│   ├── animations.dart, dark.dart, light.dart
├── assets/ (logo_light.png, logo_dark.png)
└── DESIGN_IMPROVEMENTS.md (histórico completo de melhorias)
```

### 🎯 Princípios da Nova Arquitetura

- **lib/services/** - Lógica de negócio e serviços específicos do projeto
- **lib/widgets/** - Componentes visuais 100% reutilizáveis (copy-paste ready)
- **lib/screens/** - Páginas da aplicação que combinam widgets e serviços

## Recursos e Padrões de UI

### 💬 Paleta de Comandos (Busca Global)

A aplicação implementa uma busca global no estilo "Paleta de Comandos", acessível pela barra de navegação superior ou pelo atalho `Ctrl+K`.

- **Ativação:** Clique no campo de busca na `AppBar` ou use o atalho de teclado.
- **Widget Principal:** `lib/widgets/src/app_layout/command_palette.dart`
- **Funcionamento:** Um diálogo modal (`CommandPalette`) é exibido, permitindo ao usuário buscar e filtrar uma lista de ações e itens de navegação.
- **Padrão de Atalho:** A lógica é centralizada usando um `SearchIntent` e `SearchAction`, definidos em `lib/utils/keyboard_shortcuts.dart` e implementados em `lib/widgets/src/app_layout/app_shell.dart`.

### 🔔 Serviço de Feedback (Notificações)

Para exibir mensagens de feedback (sucesso, erro, aviso), o sistema utiliza um serviço customizado que renderiza uma notificação no topo da tela.

- **Serviço:** `SnackBarService` (localizado em `lib/widgets/feedback.dart`)
- **Implementação:** Este serviço foi refatorado para usar uma `Overlay` do Flutter em vez do `SnackBar` padrão. Isso garante controle total sobre a posição (topo da tela), aparência e animação, desacoplando o feedback do `Scaffold`.
- **Uso:** `SnackBarService.showSuccess(context, message: 'Operação concluída!');`

## Padrões de Código

### Design System

- **Responsividade:** Use `AppTheme.getResponsive*()` para tamanhos adaptativos
- **Cores:** Sempre usar `AppTheme.primaryLight/primaryDark` para consistência
- **Animações:** Durações padronizadas (100ms, 200ms, 300ms)
- **Espaçamento:** Grid 8px - use `AppTheme.spacing*` (XS=4, S=8, M=16, L=24, XL=32)

### Formulários (OBRIGATÓRIO - Migração Completa)

#### 🎯 **Widgets Padronizados (ÚNICOS):**

- **Campos de Texto:** SEMPRE usar `TextInputField` (lib/widgets/src/forms/inputs/text_input_field.dart)
- **Dropdowns:** SEMPRE usar `SelectInputField` (lib/widgets/src/forms/inputs/select_input_field.dart)

#### 📐 **Layout e Organização:**

- **Alinhamento:** SEMPRE usar `FormRow.top()` ao invés de `Row` manual
- **Proporções:** Use `FormRowFlex` para controle de proporções (ex: 2:1, 1:2:1)
- **Espaçamento:** 16px entre campos, 20px entre seções
- **Validação:** Implementar em todos os campos obrigatórios

### Dialogs (OBRIGATÓRIO - Arquitetura Refatorada)

#### 🔧 Para Funcionalidades Específicas do Projeto:

- **Serviços:** Use `FormDialogService.mostrar*()` (lib/services/dialog/)
- **Widgets:** Use `TextInputField` e `SelectInputField` (importados via `lib/widgets/form.dart`)
- **Import:** `import '../../services/dialog/form_dialog_service.dart';`

#### 🎨 Para Widgets Genéricos (Reutilizáveis):

- **Serviços:** Use `DialogService.show*()` (lib/widgets/src/dialog/)
- **Callbacks:** Use `DialogCallbacks.show*()` (lib/widgets/src/dialog/)
- **Widgets:** Use `TextInputField` e `SelectInputField` (lib/widgets/form.dart)
- **Import:** `import 'widgets/src/dialog/dialog_service.dart';`

- **Tamanhos:** Pequeno=400px, Médio=600px, Grande=700px, XL=800px+
- **Scroll:** Automático para formulários longos
- **Ícones:** Sempre incluir ícone apropriado no header

## Estado Atual

### ✅ Implementado e Funcional

- **Interface:** Splash screen, dashboard com métricas, navegação completa
- **Temas:** Claro/escuro com paleta profissional refinada
- **Sidebar:** Responsiva, densidade otimizada, indicador de item ativo elegante
- **Formulários:** Sistema FormRow com alinhamento perfeito
- **Dialogs:** Sistema completo com scroll automático e validação
- **Documentação:** Completa e profissional (README.md em cada módulo)

### 🔧 Características Técnicas

- **Performance:** Otimizada, sem travamentos
- **Responsividade:** Breakpoints inteligentes para diferentes telas
- **Acessibilidade:** Cursor apropriado, feedback visual, navegação por teclado
- **UX:** Animações suaves (200ms), estados de hover, indicadores visuais

## Widgets Essenciais do Sistema

### Widgets de Formulário (SEMPRE usar os padronizados)

#### ✅ **TextInputField - Campo de Texto Padrão**

```dart
TextInputField(
  labelText: 'Nome Completo',
  hintText: 'Digite o nome completo',
  keyboardType: TextInputType.text,
  validator: FormValidators.required('Nome é obrigatório'),
  onChanged: (value) => setState(() => _nome = value),
)
```

#### ✅ **SelectInputField - Dropdown Padrão**

```dart
SelectInputField<String>(
  labelText: 'Tipo de Pessoa',
  hintText: 'Selecione o tipo',
  value: _tipoSelecionado,
  onChanged: (value) => setState(() => _tipoSelecionado = value),
  options: [
    SelectOption(value: 'PF', label: 'Pessoa Física'),
    SelectOption(value: 'PJ', label: 'Pessoa Jurídica'),
  ],
  validator: FormValidators.required('Tipo é obrigatório'),
)
```

#### 📐 **FormRow - Sistema de Alinhamento**

```dart
// Distribuição igual
FormRow.top(
  children: [
    TextInputField(labelText: 'Email', hintText: 'email@exemplo.com'),
    TextInputField(labelText: 'Telefone', hintText: '(11) 99999-9999'),
  ],
)

// Proporções customizadas
FormRowFlex.top(
  items: [
    FormRowItem(
      flex: 2,
      child: TextInputField(labelText: 'Endereço', hintText: 'Rua, número')
    ),
    FormRowItem(
      flex: 1,
      child: TextInputField(labelText: 'Número', hintText: '123')
    ),
  ],
)
```

### Cards - Exemplos de Uso

#### ✅ **InfoCard - Card de Informações Padrão**

Use este widget para exibir um card de destaque com gradiente, ícone e métricas.

```dart
InfoCard(
  title: 'Receita Mensal',
  value: 'R\$ 125.430,00',
  details: '+12%',
  detailsPositive: true,
  icon: Icons.trending_up_rounded,
  color: isDarkMode ? AppTheme.successDark : AppTheme.successLight,
)
```

#### ✅ **InfoCardGrid - Container Responsivo para Cards**

Este widget organiza uma lista de `InfoCard` em um layout responsivo. Em telas grandes, exibe um grid. Em telas menores, exibe um layout com quebra de linha (`Wrap`).

```dart
InfoCardGrid(
  children: [
    InfoCard(...),
    InfoCard(...),
  ],
)
```

### Dialogs - Exemplos de Uso

#### 🔧 Específicos do Projeto (FormDialogService)

```dart
// Import correto
import '../../services/dialog/form_dialog_service.dart';

// Cadastros pré-configurados
FormDialogService.mostrarCadastroCliente(
  context: context,
  onConfirmar: () => _salvarCliente(),
);

// Dialog customizado
FormDialogService.mostrarFormulario(
  context: context,
  titulo: 'Meu Dialog',
  icone: Icons.edit,
  formulario: _buildFormulario(),
  onConfirmar: () => _processar(),
);
```

#### 🎨 Genéricos Reutilizáveis (DialogService)

```dart
// Import correto
import 'widgets/src/dialog/dialog_service.dart';

// Dialog genérico
DialogService.showFormDialog(
  context: context,
  title: 'Generic Dialog',
  form: MyFormWidget(),
  onConfirm: () => _save(),
);

// Callback genérico para menus
DialogCallbacks.showFormCallback(
  context: context,
  title: 'Add Item',
  form: ItemFormWidget(),
  onConfirm: () => _saveItem(),
  successMessage: 'Item saved!',
);
```

#### 🔗 Callbacks para Sidebar (DomaniDialogCallbacks)

```dart
// Import correto
import '../services/dialog/domani_dialog_callbacks.dart';

// Uso na sidebar
DrawerItem(
  title: 'Clientes',
  onDialogTap: DomaniDialogCallbacks.cadastroCliente(context),
);
```

## Melhorias Implementadas (Dezembro 2024)

### 🎨 Design Profissional

- **Nova paleta de cores** seguindo especificações de design moderno
- **Cards com profundidade adequada** - sombras sutis no light, bordas no dark
- **Tipografia hierárquica** - títulos bold, texto normal, detalhes smaller
- **Espaçamento ultra-compacto** na sidebar para máximo aproveitamento

### 🔧 Sistema de Formulários

- **FormRow** - Alinhamento automático perfeito
- **FormRowFlex** - Controle de proporções (2:1, 1:2:1, etc.)
- **Validação integrada** - Campos obrigatórios e formatos
- **Responsividade** - Adapta-se a diferentes tamanhos

### 💬 Sistema de Dialogs

- **FormDialog** - Widget base reutilizável
- **FormDialogService** - Serviços pré-configurados
- **Scroll automático** - Formulários longos funcionam perfeitamente
- **Animações suaves** - Entrada/saída com scale + fade

### 🎯 UX Aprimorada

- **Cursor inteligente** - Pointer em áreas clicáveis
- **Indicador de item ativo** - Linha vertical de 4px elegante
- **Estados de hover** - Feedback visual profissional
- **Divisores entre grupos** - Organização visual na sidebar

## Documentação e Manutenibilidade

### 📚 Sistema de Documentação

- **README.md principal** - Visão geral completa
- **README.md por módulo** - Documentação específica
- **Código autodocumentado** - Comentários detalhados
- **Exemplos práticos** - Copy-paste ready

### 🏗️ Arquitetura Limpa

- **Separação de responsabilidades** - Cada widget tem propósito claro
- **Reutilização máxima** - Widgets genéricos e específicos
- **Padrões consistentes** - Estrutura uniforme
- **Escalabilidade** - Base sólida para crescimento

## Próximas Implementações

1. **Tabelas de dados** - Sistema de DataTable responsivo
2. **Gráficos fiscais** - Charts para métricas e relatórios
3. **Sistema de notificações** - Alerts e toasts padronizados
4. **Upload de arquivos** - Drag & drop para documentos
5. **Calendário fiscal** - Componente de agenda integrado
6. **APIs fiscais** - Integrações com serviços externos

## Troubleshooting Comum

### API Desatualizada

❌ **Não faça:** `label`, `hint`, `items`, `DropdownMenuItem`
✅ **Faça:** `labelText`, `hintText`, `options`, `SelectOption`

### Formulários Desalinhados

❌ **Não faça:** `Row(children: [Expanded(...), Expanded(...)])`
✅ **Faça:** `FormRow.top(children: [...])`

### Dialogs Quebrados

❌ **Não faça:** `showDialog` manual
✅ **Faça:** `FormDialogService.mostrarFormulario()`

### Cores Inconsistentes

❌ **Não faça:** `Color(0xFF...)`
✅ **Faça:** `Theme.of(context).colorScheme.primary`

### Performance Lenta

❌ **Não faça:** Widgets sem `const`
✅ **Faça:** `const TextInputField(...)` quando possível

## Arquitetura Reorganizada (Dezembro 2024)

### 🏗️ Separação de Responsabilidades

- **lib/services/** - SERVIÇOS (lógica de negócio, APIs, utilitários)
- **lib/widgets/** - WIDGETS (componentes visuais reutilizáveis)
- **lib/screens/** - TELAS (páginas da aplicação)

### 📁 Estrutura de Serviços

```
lib/services/
└── dialog/
    └── form_dialog_service.dart  # Serviços de dialog (não é widget!)
```

### 🎯 Import Correto para FormDialogService

```dart
// CORRETO - serviço está em lib/services/
import '../../services/dialog/form_dialog_service.dart';

// INCORRETO - não está mais em widgets
import '../widgets/src/dialog/form_dialog_service.dart';
```

### ✅ Benefícios da Nova Arquitetura

- **Widgets desacoplados** - Não dependem de serviços específicos
- **Serviços organizados** - Lógica de negócio centralizada
- **Manutenibilidade** - Mudanças em serviços não afetam widgets
- **Escalabilidade** - Fácil adicionar novos serviços
- **Testabilidade** - Serviços podem ser testados independentemente

#### ✅ **Widgets Padronizados (USAR SEMPRE):**

- **`TextInputField`** - Para todos os campos de texto
- **`SelectInputField`** - Para todos os dropdowns
- **Localização:** `lib/widgets/form.dart` (barril principal)

#### 🔄 **Mudanças na API:**

```dart
// ❌ REMOVIDO - Não usar mais
DomaniTextField(label: 'Nome', hint: 'Digite')
DomaniDropdown(label: 'Tipo', items: [...])

// ✅ USAR SEMPRE
TextInputField(labelText: 'Nome', hintText: 'Digite')
SelectInputField(labelText: 'Tipo', options: [...])
```

### ✅ **Widgets 100% Reutilizáveis Implementados**

A maior refatoração do projeto foi concluída com sucesso, separando completamente:

#### 🎨 **lib/widgets/** - Genéricos e Reutilizáveis

- **Objetivo:** Widgets que podem ser copiados para qualquer projeto Flutter
- **Características:** Zero dependências específicas do projeto
- **Status:** Copy-paste ready - funcionam imediatamente em novos projetos
- **Documentação:** README.md + EXAMPLE_USAGE.md com exemplos completos

### 📋 **Checklist de Uso**

#### Para Desenvolvedores do Domani Fiscal:

- ✅ Use `FormDialogService` para dialogs específicos (cliente, fornecedor, etc.)
- ✅ Use `DomaniDialogCallbacks` na sidebar e menus
- ✅ Imports corretos: `lib/services/dialog/`

#### Para Novos Projetos:

- ✅ Copie apenas `lib/widgets/` para o novo projeto
- ✅ Use `DialogService` e `DialogCallbacks` genéricos
- ✅ Consulte `EXAMPLE_USAGE.md` para exemplos práticos
- ✅ Customize conforme necessário sem quebrar funcionalidade

---

**Refatoração concluída:** Dezembro 2024  
**Status:** ✅ Implementado e testado  
**Próximo passo:** Aplicar mesmo padrão para forms, layouts e outros widgets

#### 📋 **Checklist de Desenvolvimento:**

**Para Novos Formulários:**

- [ ] ✅ Usar `TextInputField`, `SelectInputField`, etc., importando de `widgets/form.dart`.
- [ ] ✅ Usar `FormButton` para ações.
- [ ] ✅ Usar `FormRow.top()` para alinhamento.
- [ ] ✅ Usar `FormValidators` para validação.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pablogeokar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pablogeokar)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
