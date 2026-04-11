## pleno

> Plan de Implementación - Pleno Fitness App

Plan de Implementación - Pleno Fitness App



 Contexto



 App de fitness/nutrición personal en Kotlin con Jetpack Compose. El usuario quiere reemplazar Google Keep para

 tracking de entrenamientos, con sistema de "Ghost" para mostrar registros anteriores.



 Decisiones del Usuario:

 - Prioridad: Base arquitectónica completa primero

 - Unidades: Kilogramos (kg) por defecto

 - Single-user (sin autenticación)

 - Gráficos: Vico



 Stack Tecnológico



 - Lenguaje: Kotlin

 - UI: Jetpack Compose + Material 3 Expresivo

 - DB: Room (SQLite)

 - Arquitectura: MVVM + Clean Architecture

 - Gráficos: Vico

 - DI: Hilt

 - Navegación: Compose Navigation



 Estructura de Paquetes (Clean Architecture)



 com.example.pleno/

 ├── core/

 │   ├── database/          # PlenaDatabase, Converters, Migrations

 │   ├── di/                # AppModule, DatabaseModule, RepositoryModule

 │   ├── navigation/        # NavGraph, Screen, BottomNavItem

 │   ├── theme/             # Color, Type, Theme, Shape, Dimensions

 │   └── util/              # DateUtils, MacroCalculator, Constants

 │

 ├── data/

 │   ├── local/

 │   │   ├── dao/           # DashboardDao, MealDao, WorkoutDao, MetricsDao

 │   │   └── entity/        # 10 entidades (ver abajo)

 │   └── repository/        # Implementaciones de repositorios

 │

 ├── domain/

 │   ├── model/             # Modelos de dominio (DailyCheck, GhostData, etc.)

 │   ├── repository/        # Interfaces de repositorios

 │   └── usecase/           # Use cases por módulo (dashboard, meal, workout, metrics, export)

 │

 └── presentation/

     ├── components/        # PlenaCard, CalorieProgressRing, GhostDataBanner, etc.

     ├── dashboard/         # Screen, ViewModel, State

     ├── nutrition/         # Screen, ViewModel, State, components

     ├── workout/           # Screen, ViewModel, State, routine/, components

     ├── metrics/           # Screen, ViewModel, State, components

     └── export/            # Screen, ViewModel, State



 Esquema de Base de Datos



 Entidades Room (10 tablas)



 1. DailyCheckEntity - Checklist diario (creatina, sueño)

 2. WaterLogEntity - Registros de agua (+250ml)

 3. FrequentMealEntity - Catálogo de comidas frecuentes (presets)

 4. MealLogEntity - Log de comidas consumidas

 5. WorkoutRoutineEntity - Rutinas (ej. "Día A: Full Body")

 6. ExerciseEntity - Ejercicios por rutina

 7. ExerciseSetEntity - Sets individuales (peso, reps, RPE) - CRÍTICO para Ghost

 8. WeightLogEntity - Peso diario matutino

 9. JournalEntryEntity - Bitácora de sensaciones (texto libre)

 10. InBodyLogEntity - Datos InBody mensuales (SMM, grasa, agua)



 Relaciones Clave



 WorkoutRoutine (1) ──< (N) Exercise

 Exercise (1) ──< (N) ExerciseSet (ordenados por date, setNumber)



 FrequentMeal (1) ──? (N) MealLog (FK opcional - puede ser NULL para "Extras")



 Feature "Ghost" - Implementación Crítica



 Objetivo: Mostrar último registro de un ejercicio al abrirlo.



 Flujo:

 1. Usuario abre ejercicio "Leg Press"

 2. WorkoutDao.getLastSessionSets(exerciseId) obtiene último entrenamiento

 3. GetGhostDataUseCase calcula "días atrás" y formatea datos

 4. GhostDataBanner muestra: "Última vez: 2 días atrás | 110kg × 10, 110kg × 8, 105kg × 9"



 Query SQL Crítica:

 SELECT * FROM exercise_sets

 WHERE exerciseId = :exerciseId

   AND date = (

     SELECT date FROM exercise_sets

     WHERE exerciseId = :exerciseId

     ORDER BY date DESC

     LIMIT 1

   )

 ORDER BY setNumber ASC



 Componente UI: GhostDataBanner.kt con color tertiario, banner destacado arriba de inputs.



 Tema Material 3 Expresivo



 Configuración de Fuente



 - Archivo: app/src/main/res/font/google_sans_flex_variable.ttf

 - Descargar de: Google Fonts (variable font)

 - Uso: FontFamily con FontVariation.Settings para weights



 Sistema de Diseño



 Dimensions.kt:

 PlenaSpacing: xs=4dp, sm=8dp, md=12dp, lg=16dp, xl=20dp, xxl=24dp, xxxl=32dp

 PlenaCorners: standard=28dp, prominent=36dp, hero=44dp

 PlenaTextSize: displayHero=96sp, displayLarge=72sp, heroTitle=44sp, etc.

 PlenaAlpha: glass=0.95f, border=0.25f, disabled=0.38f



 Principios:

 - Expresividad: Números grandes (72-96sp) para dashboard

 - Organicidad: Corners hyper-rounded (28-44dp)

 - Breathing Room: Padding 20-24dp, spacing 12dp+

 - Profundidad: Glass effect, borders sutiles

 - Dark-First: Optimizado para modo oscuro



 Orden de Implementación



 Fase 1: Fundamentos (Día 1-2)



 1. ✅ Actualizar build.gradle.kts con todas las dependencias

 2. ✅ Crear libs.versions.toml con catálogo de versiones

 3. ✅ Descargar y agregar google_sans_flex_variable.ttf

 4. ✅ Crear sistema de tema: Theme.kt, Type.kt, Color.kt, Shape.kt, Dimensions.kt

 5. ✅ Configurar Hilt: PlenaApplication.kt (@HiltAndroidApp)

 6. ✅ Crear estructura completa de paquetes

 7. ✅ Componentes base: PlenaCard.kt, PlenaButton.kt, PlenaTextField.kt



 Fase 2: Base de Datos (Día 2-3)



 8. ✅ Crear 10 entidades con anotaciones Room

 9. ✅ Crear 4 DAOs con queries optimizadas

 10. ✅ PlenaDatabase.kt con lista de entidades y TypeConverters

 11. ✅ Converters.kt si se necesitan (para enums, listas, etc.)

 12. ✅ DatabaseModule.kt (Hilt) para inyección de DB y DAOs



 Fase 3: Domain Layer (Día 3)



 13. ✅ Modelos de dominio (10+ modelos incluyendo GhostData.kt)

 14. ✅ Interfaces de repositorios (4 repositorios)

 15. ✅ Use Cases básicos (1-2 por módulo para testing inicial):

   - GetDailyProgressUseCase

   - GetGhostDataUseCase ⭐ CRÍTICO

   - GetFrequentMealsUseCase

   - LogWeightUseCase

   - ExportDatabaseUseCase



 Fase 4: Data Layer (Día 4)



 16. ✅ Implementar 4 repositorios (mapeo Entity ↔ Domain)

 17. ✅ RepositoryModule.kt (Hilt) para bindings



 Fase 5: Dashboard MVP (Día 4-5)



 18. ✅ DashboardViewModel.kt + DashboardState.kt

 19. ✅ DashboardScreen.kt con UI completa

 20. ✅ CalorieProgressRing.kt - Anillo circular hero (240dp, números 72sp+)

 21. ✅ ChecklistItem.kt - Checkbox para creatina/sueño

 22. ✅ WaterCounter.kt - Botón +250ml con progreso

 23. ✅ Integrar con Room - Testing completo del flujo



 Fase 6: Navegación (Día 6)



 24. ✅ Screen.kt - sealed class con rutas

 25. ✅ NavGraph.kt - NavHost con todas las pantallas

 26. ✅ BottomNavItem.kt - Items de navegación

 27. ✅ MainActivity.kt - Scaffold con BottomNavigation



 Fase 7: Nutrición (Día 6-8)



 28. ✅ NutritionViewModel.kt + NutritionState.kt

 29. ✅ NutritionScreen.kt - Lista de comidas frecuentes + logs del día

 30. ✅ FrequentMealCard.kt - Card con tap rápido para log

 31. ✅ AddMealDialog.kt - CRUD de comidas frecuentes

 32. ✅ MealLogItem.kt - Mostrar comida registrada

 33. ✅ FAB "Extras" - Input rápido de comida ad-hoc (sin guardar en frecuentes)

 34. ✅ Use Cases completos: SaveFrequentMealUseCase, LogMealUseCase, GetDailyNutritionUseCase



 Fase 8: Entrenamiento (Día 8-11) ⭐ MÁS COMPLEJO



 35. ✅ WorkoutViewModel.kt + WorkoutState.kt

 36. ✅ WorkoutScreen.kt - Lista de rutinas activas

 37. ✅ RoutineCard.kt - Card para cada rutina

 38. ✅ RoutineDetailViewModel.kt + RoutineDetailState.kt

 39. ✅ RoutineDetailScreen.kt - Lista de ejercicios de la rutina

 40. ✅ ExerciseDetailScreen.kt ⭐ PANTALLA CRÍTICA:

   - GhostDataBanner.kt arriba (mostrar último registro)

   - SetInputRow.kt (inputs: peso, reps, RPE, notas)

   - Lista de sets del día actual

   - Botón "Agregar Set"

 41. ✅ RestTimer.kt - FAB flotante con countdown (2:00)

 42. ✅ Use Cases completos:

   - GetGhostDataUseCase ⭐ Lógica compleja de última sesión

   - SaveWorkoutSetUseCase

   - GetRoutinesUseCase

   - GetWorkoutHistoryUseCase



 Fase 9: Métricas (Día 12-13)



 43. ✅ MetricsViewModel.kt + MetricsState.kt

 44. ✅ MetricsScreen.kt - Tabs o secciones (Peso, Diario, InBody)

 45. ✅ WeightChart.kt - Gráfico Vico de peso (últimos 30 días)

 46. ✅ WeightLogCard.kt - Input de peso diario

 47. ✅ JournalCard.kt - TextField expandido para sensaciones

 48. ✅ InBodyImporter.kt - Formulario para datos InBody

 49. ✅ Integración Vico: LineChart con Material 3 theming



 Fase 10: Exportación (Día 14)



 50. ✅ ExportViewModel.kt + ExportState.kt

 51. ✅ ExportScreen.kt - Opciones de export

 52. ✅ ExportDatabaseUseCase - Generar JSON/CSV completo de DB

 53. ✅ GenerateDailySummaryUseCase - Formato copiable:

 📅 08/Enero | 💪 Gym: Sí (Prensa 110kg) | 🍔 2850 kcal | 💧 3.1L | 💊 Creatina: Sí

 54. ✅ Clipboard API - Copiar al portapapeles

 55. ✅ File export - Guardar JSON en Downloads



 Fase 11: Polish (Día 15-16)



 56. ✅ Animaciones de navegación

 57. ✅ Loading states en todos los ViewModels

 58. ✅ Error handling y mensajes de usuario

 59. ✅ Empty states (lista vacía de rutinas, comidas, etc.)

 60. ✅ Confirmation dialogs (eliminar comida, rutina, etc.)

 61. ✅ Testing manual de todos los flujos

 62. ✅ Data seeding opcional para demo



 

 Future Enhancements (Post-MVP)



 - Widget de home screen (calorías glanceables)

 - Notificaciones (recordatorios)

 - Backup a Google Drive

 - Progressive overload tracker (análisis)

 - Import/Export de rutinas

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Fraga9)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Fraga9)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
