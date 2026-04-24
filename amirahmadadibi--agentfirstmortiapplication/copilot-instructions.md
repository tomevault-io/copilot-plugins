## agentfirstmortiapplication

> - Always Respect Clean Code, Correct Convention and SOLID Principle and Architectuer Best Practises

# Flutter Project Rules

- Always Respect Clean Code, Correct Convention and SOLID Principle and Architectuer Best Practises
- You are helping me vibe-code a Flutter application and i'm attending a very important two day hakathon.
- Always write clean, production-quality Flutter code.
- Use Material 3.
- Follow best practices for folder structure (view, domain, data) and featuer based folder struction.
- Domain holds usecases and abstract repostiories and entities.
- Data holds datasource and repositories.
- View holds screens and view comoponents and bloc.
- Always When Implementin a screen adhere to best practies keep statfull widgets small for faster rebuilds and also 
break down big widget to sperate file and keep it in relevent featuer view folder.
- When I give you UI from Figma, generate widgets with responsive layout.
- Never modify files without showing a diff preview first.
- Always use conventional and comperhensive naming.
- Be Consistant in ui, alwasy create Custom Widget for Ui like EditText,Button,AppBar,BottomSheet and reuse components.
- Alwasy Respect Project Feauter Based Structure.
- Prepare a good commit command with comperhensive message after each task get's done.
- Add ComperHensive Comments for all code that you right.
- If You Want to Emmit state from bloc alwasy handle Loadin,Success,Error and Try Again in state and ui actions.
- Always keep in Mind to provide Depencies throth constructor injection and prove them using service locator, so you have to setup dependecies in service locator accrodingly.
- Handle Navigaiton Using Go_Router Package.
- Create a custon style folder in lib/core/widgets create sperate files for style of custom widget's that you create and source all styles from these styles in this folder.
- Create Theme dark and light at first glance and get all your colors from Theme that you careted lib/core/themes, these theme fiels are single soruce of themes in applicaiton and anything in ui.
- App default is in persian language but, implement I10n and provide english languge too.Implement I10n in lib/ and manage all the text content inside it.  
- Add persian localizations value as comment  only and only for screen titles and componnet text's like button and edit text.
- Handling Erros and return respective error message to ui layer and implmeent error hanling ui and logic in all layaers is a must.

### Architecture Components:
- **Network Layer** (`lib/core/network/`): 
  - Dio client with general and authenticated instances
  - Retrofit API services (GeneralApiService, AuthenticatedApiService)
  - Error handling (ApiException, ApiErrorResponse)
  - Interceptors (logging, error handling)
  
- **Router** (`lib/core/router/`):
  - GoRouter configuration
  - AppRoutes class for route path constants
  - Error handling for unknown routes
  
- **Service Locator** (`lib/core/serviceLocator/`):
  - GetIt-based dependency injection
  - `setupServiceLocator()` function for initialization
  - Helper getters for accessing registered services
  
- **Themes** (`lib/core/themes/`):
  - Material 3 theme configuration
  - Light/dark theme support
  - Locale-aware typography (YekanBakh for Persian, default for English)
  - Single source of truth for all colors and styling
  
- **Custom Widgets** (`lib/core/widgets/`):
  - CustomButton, CustomEditText, CustomAppBar, CustomBottomSheet
  - Styles folder with separate style files for each widget
  - Language toggle icon
  
- **Services** (`lib/core/services/`):
  - LocaleService for language management

### Important Conventions:
- All colors MUST come from Theme (lib/core/themes/app_theme.dart)
- All custom widget styles MUST be in lib/core/widgets/styles/
- Service locator setup is required before app initialization
- Router uses AppRoutes class for all route paths
- Network layer supports both authenticated and general API calls
- Error handling is implemented at network, domain, and view layers

### When Adding New Features:
- Register dependencies in service_locator.dart
- Add routes to AppRouter if needed
- Use existing custom widgets from core/widgets
- Follow the feature-based structure (view/domain/data)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amirahmadadibi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
