## comidaperuana

> Define la estructura de carpetas y organización de archivos del proyecto siguiendo Clean Architecture con separación por features.


# Rule: folder_architecture

## Objective

**OBLIGATORIO**: Seguir estrictamente la estructura de carpetas definida en `docs/flujo_de_arquitectura.md`. Esta arquitectura garantiza separación de responsabilidades, mantenibilidad y escalabilidad del proyecto.

---

## Estructura General

### 📁 Niveles Principales

```
lib/
├── core/          # Código compartido y base del proyecto
├── common/        # Widgets y componentes compartidos
└── features/      # Features organizadas por dominio
```

---

## 📦 Core (`lib/core/`)

**Propósito**: Código compartido, servicios base, utilidades y configuraciones globales.

### ✅ OBLIGATORIO: Organización de Core

#### `core/theme/`
- **Propósito**: Sistema de temas y estilos globales
- **Archivos requeridos**:
  - `app_colors.dart` - Paleta de colores centralizada
  - `app_typography.dart` - Tipografía y estilos de texto
  - `material_theme.dart` - ThemeData para Material Design
  - `cupertino_theme.dart` - CupertinoThemeData para iOS

```dart
// ✅ CORRECTO: Usar colores desde app_colors.dart
Container(
  color: AppColors.primary,
  child: Text(
    'Hello',
    style: AppTypography.headline1,
  ),
)
```

#### `core/ui/`
- **Propósito**: Componentes UI adaptativos y específicos de plataforma
- **Estructura**:
  - `adaptive/` - Widgets que se adaptan automáticamente (Material/Cupertino)
  - `material/` - Widgets específicos de Material Design
  - `cupertino/` - Widgets específicos de iOS

**Reglas**:
- **OBLIGATORIO**: Usar widgets `adaptive/` cuando sea posible para soporte multiplataforma
- **OBLIGATORIO**: Usar widgets `material/` o `cupertino/` solo cuando se requiera estilo específico

```dart
// ✅ CORRECTO: Widget adaptativo
AdaptiveScaffold(
  appBar: AdaptiveAppBar(title: Text('Home')),
  body: AdaptiveButton(
    onPressed: () {},
    child: Text('Click me'),
  ),
)

// ✅ CORRECTO: Widget específico cuando se requiere
MaterialButton(
  onPressed: () {},
  child: Text('Material only'),
)
```

#### `core/utils/`
- **Propósito**: Utilidades y helpers reutilizables
- **Archivos**:
  - `platform_utils.dart` - Detección de plataforma (`isIOS`, `isAndroid`, etc.)
  - `validators.dart` - Validaciones de formularios
  - `logger.dart` - Servicio de logging centralizado

**Reglas**:
- **OBLIGATORIO**: Usar `logger.dart` en lugar de `print()`
- **OBLIGATORIO**: Centralizar validaciones en `validators.dart`

#### `core/extensions/`
- **Propósito**: Extensiones de Dart/Flutter para mejorar API
- **Archivos**:
  - `context_extensions.dart` - Extensiones de `BuildContext`
  - `string_extensions.dart` - Extensiones de `String`
  - `iterable_extensions.dart` - Extensiones de `Iterable`

**Reglas**:
- **OBLIGATORIO**: Agrupar extensiones por tipo
- **OBLIGATORIO**: Documentar extensiones públicas

#### `core/generics/`
- **Propósito**: Tipos genéricos y estructuras de datos reutilizables
- **Archivos**:
  - `result.dart` - Tipo `Result<T>` o `Either<Failure, T>` genérico
  - `paginated_list.dart` - Estructura para listas paginadas

**Reglas**:
- **OBLIGATORIO**: Usar `Result` o `Either` de fpdart para manejo de errores
- **OBLIGATORIO**: Implementar `Equatable` en tipos genéricos cuando sea necesario

#### `core/services/`
- **Propósito**: Servicios globales y singletons
- **Archivos**:
  - `firebase_service.dart` - Inicialización y configuración de Firebase
  - `local_storage_service.dart` - Abstracción de almacenamiento local (SharedPreferences/Hive)
  - `network_info.dart` - Detección de conectividad
  - `api_client.dart` - Cliente HTTP (Dio, http, etc.)

**Reglas**:
- **OBLIGATORIO**: Registrar servicios como Lazy Singletons en `injection.dart`
- **OBLIGATORIO**: Implementar interfaces para servicios (para testing)
- **OBLIGATORIO**: Usar `Either<Failure, T>` para operaciones que pueden fallar

### 🔌 Inyección de dependencias (`lib/core/injection.dart`)
- **Propósito**: Punto único para declarar y registrar dependencias siguiendo el flujo Data → Domain → Presentation.
- **Reglas**:
  - **OBLIGATORIO**: Registrar servicios y clients compartidos como `registerLazySingleton`.
  - **OBLIGATORIO**: Registrar `UseCase` y repositorios de dominio como `registerLazySingleton`.
  - **OBLIGATORIO**: Registrar BLoC/Cubit en `registerFactory` para obtener instancias limpias por pantalla.
  - **OBLIGATORIO**: Mantener el orden de registro: data (sources/clients) → repositorios (impl) → casos de uso → presentadores (bloc/cubit).
  - **PROHIBIDO**: Registrar dependencias directamente en widgets/páginas; todo debe pasar por `injection.dart`.
- **Ejemplo**:
```dart
Future<void> configureDependencies() async {
  // Data
  getIt.registerLazySingleton<ApiClient>(() => ApiClientImpl());
  getIt.registerLazySingleton<AuthApiSource>(
    () => AuthApiSource(apiClient: getIt()),
  );

  // Domain
  getIt.registerLazySingleton<IAuthRepository>(
    () => AuthRepositoryImpl(remoteSource: getIt()),
  );
  getIt.registerLazySingleton<LoginUseCase>(
    () => LoginUseCase(repository: getIt()),
  );

  // Presentation
  getIt.registerFactory<AuthBloc>(
    () => AuthBloc(loginUseCase: getIt()),
  );
}
```

#### `core/errors/`
- **Propósito**: Manejo centralizado de errores y excepciones
- **Archivos**:
  - `exceptions.dart` - Excepciones específicas (ServerException, CacheException, etc.)
  - `failures.dart` - Failures para programación funcional (ServerFailure, AuthFailure, etc.)
  - `error_mapper.dart` - Mapeo de Exception → Failure

**Reglas**:
- **OBLIGATORIO**: Usar `Failure` en lugar de `Exception` en la capa de dominio
- **OBLIGATORIO**: Mapear `Exception` → `Failure` en la capa de datos
- **OBLIGATORIO**: Implementar `Equatable` en todas las clases `Failure`

#### `core/config/`
- **Propósito**: Configuraciones y variables de entorno
- **Archivos**:
  - `flavor_config.dart` - Configuración de flavors (dev, staging, prod)
  - `env.dart` - Variables de entorno y claves de API

**Reglas**:
- **OBLIGATORIO**: Usar `env.dart` para acceso a variables de entorno
- **OBLIGATORIO**: No hardcodear URLs, claves o configuraciones

---

## 🧩 Common (`lib/common/`)

**Propósito**: Widgets y componentes compartidos entre múltiples features.

### ✅ OBLIGATORIO: Organización de Common

#### `common/widgets/`
- **Propósito**: Widgets reutilizables genéricos
- **Archivos típicos**:
  - `loading_indicator.dart` - Indicador de carga
  - `empty_state.dart` - Estado vacío
  - `error_view.dart` - Vista de error

**Reglas**:
- **OBLIGATORIO**: Widgets deben ser genéricos y reutilizables
- **OBLIGATORIO**: No deben contener lógica de negocio específica
- **OBLIGATORIO**: Usar theme system para estilos

```dart
// ✅ CORRECTO: Widget genérico reutilizable
class LoadingIndicator extends StatelessWidget {
  final String? message;
  
  const LoadingIndicator({super.key, this.message});
  
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          CircularProgressIndicator(
            color: Theme.of(context).colorScheme.primary,
          ),
          if (message != null) ...[
            SizedBox(height: 16),
            Text(
              message!,
              style: Theme.of(context).textTheme.bodyMedium,
            ),
          ],
        ],
      ),
    );
  }
}
```

---

## 🎯 Features (`lib/features/`)

**Propósito**: Features organizadas por dominio de negocio, cada una auto-contenida.

### ✅ OBLIGATORIO: Estructura de Feature

Cada feature **DEBE** seguir esta estructura:

```
features/
└── [feature_name]/
    ├── data/
    │   ├── models/
    │   │   └── [entity]_model.dart                   // DTOs
    │   ├── sources/                                  // <── NUEVO nombre
    │   │   ├── [feature]_api_source.dart            // source REST/API
    │   │   ├── [feature]_local_source.dart          // source local (cache)
    │   │   └── [feature]_firestore_source.dart      // source Firestore
    │   └── repositories/
    │       └── [feature]_repository_impl.dart        // implementa interface
    ├── domain/
    │   ├── entities/
    │   │   └── [entity]_entity.dart                  // objetos de negocio
    │   ├── repositories/
    │   │   └── i_[feature]_repository.dart           // <── interfaz con prefijo i_
    │   └── usecases/
    │       └── [action]_usecase.dart                 // casos de uso
    └── presentation/
        ├── bloc/
        │   ├── [feature]_bloc.dart
        │   ├── [feature]_event.dart
        │   └── [feature]_state.dart
        ├── pages/
        │   └── [feature]_page.dart
        └── widgets/
            └── [feature]_widget.dart
```

### 📋 Reglas por Capa

#### Data Layer (`data/`)

**Propósito**: Implementación de fuentes de datos y modelos.

**Reglas**:
- **OBLIGATORIO**: Usar carpeta `sources/` (NO `datasources/`)
- **OBLIGATORIO**: Sources pueden ser: `api_source`, `local_source`, `firestore_source`
- **OBLIGATORIO**: Models deben extender/convertir a Entities del dominio
- **OBLIGATORIO**: Repository implementation debe implementar la interfaz del dominio
- **OBLIGATORIO**: Usar `Either<Failure, T>` para todas las operaciones

```dart
// ✅ CORRECTO: API Source
class AuthApiSource {
  final ApiClient apiClient;
  
  AuthApiSource({required this.apiClient});
  
  Future<Either<DataSourceFailure, UserModel>> login({
    required String email,
    required String password,
  }) async {
    try {
      final response = await apiClient.post('/auth/login', data: {
        'email': email,
        'password': password,
      });
      
      final user = UserModel.fromJson(response.data);
      return Right(user);
    } on ServerException catch (e) {
      logger.e('Login failed', error: e);
      return Left(DataSourceFailure.server(e.message));
    }
  }
}

// ✅ CORRECTO: Local Source
class AuthLocalSource {
  final LocalStorageService storage;
  
  AuthLocalSource({required this.storage});
  
  Future<Option<UserModel>> getCachedUser(String userId) async {
    // Implementación de cache local
  }
  
  Future<Either<DataSourceFailure, Unit>> cacheUser(UserModel user) async {
    // Implementación de guardado local
  }
}

// ✅ CORRECTO: Firestore Source
class AuthFirestoreSource {
  final FirebaseFirestore firestore;
  
  AuthFirestoreSource({required this.firestore});
  
  Future<Either<DataSourceFailure, UserModel>> getUserFromFirestore(String userId) async {
    // Implementación de Firestore
  }
}
```

#### Domain Layer (`domain/`)

**Propósito**: Lógica de negocio pura, sin dependencias externas.

**Reglas**:
- **OBLIGATORIO**: Entities deben ser objetos inmutables
- **OBLIGATORIO**: Repositories son solo interfaces (abstracciones)
- **OBLIGATORIO**: Use Cases deben ser clases simples con un solo método `call()`
- **OBLIGATORIO**: No dependencias de Flutter o paquetes externos (excepto fpdart)
- **OBLIGATORIO**: Usar `Either<Failure, T>` para todos los retornos

```dart
// ✅ CORRECTO: Entity inmutable
class UserEntity extends Equatable {
  final String id;
  final String email;
  final String name;
  
  const UserEntity({
    required this.id,
    required this.email,
    required this.name,
  });
  
  @override
  List<Object> get props => [id, email, name];
}

// ✅ CORRECTO: Repository Interface (con prefijo i_)
abstract class IAuthRepository {
  Future<Either<AuthFailure, UserEntity>> login({
    required String email,
    required String password,
  });
}

// ✅ CORRECTO: Use Case
class LoginUseCase {
  final IAuthRepository repository;
  
  LoginUseCase({required this.repository});
  
  Future<Either<AuthFailure, UserEntity>> call({
    required String email,
    required String password,
  }) async {
    return await repository.login(email: email, password: password);
  }
}
```

#### Presentation Layer (`presentation/`)

**Propósito**: UI, widgets y manejo de estado.

**Reglas**:
- **OBLIGATORIO**: Usar BLoC/Cubit para state management
- **OBLIGATORIO**: Separar BLoC, Events y States en archivos diferentes
- **OBLIGATORIO**: Pages deben ser StatelessWidget cuando sea posible
- **OBLIGATORIO**: Widgets específicos de la feature en `widgets/`
- **OBLIGATORIO**: Usar theme system para estilos

```dart
// ✅ CORRECTO: BLoC Structure
// auth_bloc.dart
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  final LoginUseCase loginUseCase;
  
  AuthBloc({required this.loginUseCase}) : super(AuthInitial()) {
    on<LoginRequested>(_onLoginRequested);
  }
  
  Future<void> _onLoginRequested(
    LoginRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(AuthLoading());
    
    final result = await loginUseCase(
      email: event.email,
      password: event.password,
    );
    
    result.fold(
      (failure) => emit(AuthError(failure)),
      (user) => emit(AuthAuthenticated(user)),
    );
  }
}

// ✅ CORRECTO: Page Structure
class LoginPage extends StatelessWidget {
  const LoginPage({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(
          'Login',
          style: Theme.of(context).textTheme.titleLarge,
        ),
      ),
      body: BlocProvider(
        create: (context) => getIt<AuthBloc>(),
        child: const LoginForm(),
      ),
    );
  }
}
```

---

## 🚫 Reglas de Prohibición

### ❌ PROHIBIDO

1. **Crear archivos fuera de la estructura definida**
   ```dart
   // ❌ INCORRECTO: Archivo en raíz de lib/
   lib/user_service.dart
   
   // ✅ CORRECTO: En la feature correspondiente
   lib/features/auth/data/sources/auth_api_source.dart
   ```

2. **Usar `datasources/` en lugar de `sources/`**
   ```dart
   // ❌ INCORRECTO: Nombre antiguo
   lib/features/auth/data/datasources/auth_remote_datasource.dart
   
   // ✅ CORRECTO: Usar sources/
   lib/features/auth/data/sources/auth_api_source.dart
   ```

3. **No usar prefijo `i_` en interfaces de repositorio**
   ```dart
   // ❌ INCORRECTO: Sin prefijo
   lib/features/auth/domain/repositories/auth_repository.dart
   
   // ✅ CORRECTO: Con prefijo i_
   lib/features/auth/domain/repositories/i_auth_repository.dart
   ```

4. **Mezclar capas de arquitectura**
   ```dart
   // ❌ INCORRECTO: Entity con dependencias de Flutter
   class UserEntity {
     Widget buildWidget() { ... } // ❌ NO
   }
   
   // ✅ CORRECTO: Entity pura
   class UserEntity {
     final String id;
     final String name;
   }
   ```

5. **Duplicar código que debería estar en core/**
   ```dart
   // ❌ INCORRECTO: Logger en cada feature
   class AuthService {
     void log(String message) {
       print(message); // ❌ NO
     }
   }
   
   // ✅ CORRECTO: Usar logger de core
   import 'package:app/core/utils/logger.dart';
   
   class AuthService {
     void doSomething() {
       logger.i('Action performed'); // ✅ SÍ
     }
   }
   ```

6. **Crear widgets genéricos en features/**
   ```dart
   // ❌ INCORRECTO: Widget genérico en feature
   lib/features/auth/presentation/widgets/loading_indicator.dart
   
   // ✅ CORRECTO: Widget genérico en common
   lib/common/widgets/loading_indicator.dart
   ```

---

## 📝 Convenciones de Nomenclatura

### Archivos
- **Snake_case** para todos los archivos: `user_repository.dart`
- **Sufijos descriptivos**:
  - `_impl.dart` para implementaciones
  - `_model.dart` para DTOs
  - `_entity.dart` para entidades de dominio
  - `_bloc.dart`, `_event.dart`, `_state.dart` para BLoC
  - `_usecase.dart` para casos de uso
- **Prefijos para interfaces**:
  - `i_` para interfaces de repositorio: `i_auth_repository.dart`
- **Nombres de sources**:
  - `[feature]_api_source.dart` para fuentes REST/API
  - `[feature]_local_source.dart` para fuentes locales (cache)
  - `[feature]_firestore_source.dart` para fuentes Firestore

### Carpetas
- **Snake_case** para carpetas: `sources/` (NO `datasources/`), `use_cases/`
- **Singular** para entidades: `entity/`, `model/`
- **Plural** para colecciones: `widgets/`, `pages/`

---

## ✅ Checklist de Implementación

Antes de crear un archivo, verificar:

- [ ] ¿El archivo está en la carpeta correcta según su propósito?
- [ ] ¿Sigue la convención de nomenclatura?
- [ ] ¿Está en la capa correcta (data/domain/presentation)?
- [ ] ¿Usa las interfaces/abstracciones correctas?
- [ ] ¿No duplica funcionalidad existente en `core/` o `common/`?
- [ ] ¿Implementa `Either<Failure, T>` para operaciones que pueden fallar?
- [ ] ¿Usa el theme system para estilos?
- [ ] ¿Está registrado en `injection.dart` si es necesario?

---

## 🔄 Flujo de Dependencias

**OBLIGATORIO**: Respetar el flujo de dependencias:

```
presentation → domain ← data
     ↓           ↑
   BLoC/Cubit  UseCase
     ↓           ↑
   Pages      Repository (interface)
     ↓           ↑
   Widgets   Repository (impl)
```

**Reglas**:
- **Presentation** puede depender de **Domain**
- **Data** puede depender de **Domain**
- **Domain** NO puede depender de **Presentation** ni **Data**
- **Presentation** NO puede depender directamente de **Data**

---

## 📚 Referencias

- **Documentación completa**: `docs/flujo_de_arquitectura.md`
- **Clean Architecture**: Seguir principios SOLID
- **BLoC Pattern**: Ver `docs/bloc_good_practices.md`
- **Error Handling**: Ver `docs/error_handling_flow.md`

---

## @todo

- [ ] Validar estructura con análisis estático
- [ ] Crear templates de archivos para nuevas features
- [ ] Documentar patrones de migración de código existente

---

**Última actualización**: Actualización según `docs/flujo_de_arquitectura.md` - Cambio de `datasources/` a `sources/`, prefijo `i_` para interfaces de repositorio, y estructura de sources (api, local, firestore)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/niltonDevelop)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/niltonDevelop)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
