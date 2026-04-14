## basal-fit

> Este é um projeto **Flutter** de calculadora de Taxa Metabólica Basal (TMB) desenvolvido com **Clean Architecture** e **Firebase**. Foi criado utilizando IA (Claude Sonnet) + Cursor como exemplo de desenvolvimento assistido por IA.

# Cursor Rules - Basal Fit Flutter

## 🎯 Contexto do Projeto

Este é um projeto **Flutter** de calculadora de Taxa Metabólica Basal (TMB) desenvolvido com **Clean Architecture** e **Firebase**. Foi criado utilizando IA (Claude Sonnet) + Cursor como exemplo de desenvolvimento assistido por IA.

## 🏗️ Arquitetura e Padrões

### **Clean Architecture Implementada:**

- **Entities**: `lib/models/` - Modelos de dados (UserProfile, Exercise, etc.)
- **Use Cases**: `lib/use_cases/` - Regras de negócio
- **Repositories**: `lib/repositories/` - Abstração de dados
- **Services**: `lib/services/` - Implementação externa (Firebase)
- **Providers**: `lib/providers/` - State management (ChangeNotifier)
- **UI**: `lib/modules/` - Interface dividida por módulos

### **Padrões Obrigatórios:**

- **MVVM Pattern** para UI components
- **Repository Pattern** para acesso a dados
- **Result Pattern** (`lib/core/utils/result.dart`) para tratamento de erros
- **Dependency Injection** com Modular
- **Single Responsibility Principle**

## 📁 Estrutura do Projeto

```
lib/
├── core/                   # Núcleo da aplicação
│   ├── constants/         # Constantes globais
│   ├── di/               # Dependency Injection
│   ├── exceptions/       # Exceções customizadas
│   ├── interfaces/       # Contratos/Interfaces
│   ├── theme/           # Design System (AppTheme)
│   ├── utils/           # Utilitários (Result pattern)
│   └── widgets/         # Componentes reutilizáveis
├── models/              # Entidades/Modelos
├── modules/             # Features por módulo
│   ├── auth/           # Autenticação
│   └── home/           # Funcionalidades principais
├── providers/          # State management
├── repositories/       # Camada de dados
├── services/          # Serviços externos
├── use_cases/         # Regras de negócio
└── main.dart
```

## 🎨 Design System

### **Tema e Cores:**

- Use **AppTheme** de `lib/core/theme/app_theme.dart`
- Cores principais: `primaryColor`, `secondaryColor`, `backgroundColor`
- Gradientes: `primaryLinearGradient`, `cardGradient`
- Espaçamentos: `spacingXS`, `spacingS`, `spacingM`, `spacingL`, `spacingXL`

### **Componentes Reutilizáveis:**

```dart
// Use componentes de lib/core/widgets/ui_components.dart
GradientCard()
MenuCard()
StatCard()
PrimaryButton()
SecondaryButton()
CustomTextField()
UserAvatar()
LoadingOverlay()
SectionHeader()
```

## 🔥 Firebase

### **Configuração:**

- **Firestore**: Banco principal com regras de segurança em `firebase/firestore.rules`
- **Auth**: Email/senha, Google, Facebook
- **Storage**: NÃO usar - imagens em base64 no Firestore
- **Rules**: Sempre validar permissões e tipos de dados

### **Modelos Firebase:**

```dart
// Sempre implementar toMap() e fromMap()
class UserProfile {
  Map<String, dynamic> toMap() { ... }
  static UserProfile fromMap(Map<String, dynamic> map) { ... }
}
```

## 📝 Convenções de Código

### **Nomenclatura:**

- **Classes**: PascalCase (`UserProfile`, `TmbCalculatorScreen`)
- **Variáveis/Métodos**: camelCase (`calculateTmb`, `userProfile`)
- **Arquivos**: snake_case (`user_profile.dart`, `tmb_calculator_screen.dart`)
- **Constantes**: SCREAMING_SNAKE_CASE (`API_TIMEOUT`)

### **Estrutura de Widgets:**

```dart
class ExampleScreen extends StatefulWidget {
  const ExampleScreen({super.key});

  @override
  State<ExampleScreen> createState() => _ExampleScreenState();
}

class _ExampleScreenState extends State<ExampleScreen> {
  // Estado local

  @override
  void initState() {
    super.initState();
    // Inicialização
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // UI
    );
  }

  // Métodos privados por último
  void _privateMethod() { ... }
}
```

### **Tratamento de Erros:**

```dart
// Sempre usar Result pattern
Future<Result<UserProfile>> getUserProfile() async {
  try {
    final profile = await _firebaseService.getUserProfile();
    return Result.success(profile);
  } catch (e) {
    return Result.failure(DatabaseException('Erro ao buscar perfil: $e'));
  }
}

// Na UI
final result = await _getUserProfile();
result.when(
  success: (profile) => _updateUI(profile),
  failure: (error) => _showError(error.message),
);
```

## 🚀 Guidelines de Desenvolvimento

### **State Management:**

- Use **ChangeNotifier** para providers
- Use **StreamBuilder** para dados em tempo real do Firebase
- Use **StatefulWidget** apenas quando necessário

### **Performance:**

- Use `const` constructors sempre que possível
- Implemente `ListView.builder` para listas grandes
- Use `FutureBuilder`/`StreamBuilder` adequadamente

### **Validação:**

```dart
// Use validators consistentes
String? validateEmail(String? value) {
  if (value == null || value.isEmpty) return 'Email é obrigatório';
  if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
    return 'Email inválido';
  }
  return null;
}
```

### **Imports:**

```dart
// Ordem dos imports:
import 'package:flutter/material.dart';           // Flutter
import 'package:firebase_auth/firebase_auth.dart'; // Packages
import '../models/user_profile.dart';              // Relativos
```

## 📚 Documentação

### **Comentários:**

- Documente **métodos públicos** e **classes importantes**
- Use **português** para comentários e documentação
- Siga o padrão existente em `docs/`

### **README e Docs:**

- Mantenha `docs/` atualizado com mudanças
- Use **markdown** com emojis e formatação consistente
- Documente **breaking changes** e **novas features**

## 🧪 Testes

### **Estrutura de Testes:**

```dart
void main() {
  group('UserProfile', () {
    test('deve calcular TMB corretamente', () {
      // Arrange
      final profile = UserProfile(/* ... */);

      // Act
      final tmb = profile.calculateTmb();

      // Assert
      expect(tmb, equals(expectedValue));
    });
  });
}
```

## 🔧 Firebase Rules

### **Sempre validar:**

- Autenticação: `request.auth != null`
- Propriedade: `request.auth.uid == resource.data.id`
- Tipos de dados e campos obrigatórios
- Tamanhos e formatos (email, nome, etc.)

## 🎭 UI/UX Guidelines

### **Responsividade:**

- Use `MediaQuery.of(context).size` para adaptação
- Implemente breakpoints para tablet/desktop
- Use `Flexible` e `Expanded` adequadamente

### **Animações:**

```dart
// Use AnimationController com dispose
class _ScreenState extends State<Screen> with TickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

## 🚫 Não Faça

- **NÃO** use Firebase Storage (apenas base64 no Firestore)
- **NÃO** faça commits sem documentação
- **NÃO** ignore tratamento de erros
- **NÃO** use hardcoded strings (use constantes)
- **NÃO** misture lógica de negócio na UI

## ✅ Sempre Faça

- **USE** o Result pattern para tratamento de erros
- **IMPLEMENTE** loading states e feedback visual
- **VALIDE** dados de entrada
- **TESTE** em diferentes dispositivos
- **DOCUMENTE** mudanças importantes
- **SIGA** a Clean Architecture estabelecida

## 🤖 IA Development Notes

Este projeto foi criado com **Claude Sonnet + Cursor** como demonstração de desenvolvimento assistido por IA. Mantenha:

- **Consistência** arquitetural
- **Qualidade** de código
- **Documentação** completa
- **Padrões** estabelecidos

---

**Objetivo:** Manter o projeto como referência de qualidade para desenvolvimento Flutter com IA.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/camargoguilherme) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
