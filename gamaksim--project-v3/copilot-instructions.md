## project-v3

> **Claude Code, прежде чем писать ЛЮБОЙ код:**

# Zzz — Premium Sleep App (Senior Flutter Architecture)

---

## ⚠️ ПЕРЕД НАЧАЛОМ РАБОТЫ — ОБЯЗАТЕЛЬНО!

**Claude Code, прежде чем писать ЛЮБОЙ код:**

1. 📚 Открой и прочитай **LEARNING.md** полностью
2. 🔍 Изучи документацию всех пакетов из LEARNING.md
3. 🏗️ Пойми Clean Architecture и Feature-First структуру
4. ✅ Только после этого начинай кодить!

**Без этого ты не сможешь писать код на уровне супер-сеньора!**

---

## 🎯 Суть проекта

Мобильное приложение для улучшения сна через **вечерние ритуалы**. Умный коуч готовит пользователя ко сну через серию push-уведомлений от ужина до сна.

**Уникальность:** Sleep Score (0-100) при выборе времени с объяснением пользы/вреда. Полностью бесплатное, без рекламы.

---

## 👨‍💻 Уровень разработки

**Супер-сеньор Flutter разработчик**

Используем:
- ✅ Flutter 3.24+ (последняя стабильная)
- ✅ Dart 3.5+ с Records, Patterns, Sealed classes
- ✅ Feature-first Clean Architecture
- ✅ Строгая типизация + Null Safety
- ✅ Code generation (Freezed, Riverpod Generator)
- ✅ Все современные best practices

---

## 🏗️ Архитектура: Feature-First Clean Architecture

### Принципы:
1. **Feature-first** — код организован по фичам, не по типам
2. **Clean Architecture** — чёткое разделение слоёв
3. **SOLID** принципы
4. **Dependency Injection** через Riverpod
5. **Immutability** через Freezed
6. **Type Safety** везде

### Слои (от внешнего к внутреннему):
```
Presentation → Application → Domain → Infrastructure
```

**Presentation (UI):**
- Screens, Widgets
- Riverpod Consumers
- Только отображение

**Application (Business Logic):**
- State Notifiers / AsyncNotifiers
- Use Cases
- State классы (Freezed)

**Domain (Core Business):**
- Entities (Freezed)
- Repository Interfaces
- Value Objects
- Domain Errors

**Infrastructure (Data):**
- Repository Implementations
- Data Sources (Hive, FCM)
- DTOs (Data Transfer Objects)
- Mappers

---

## 📦 Современный Tech Stack

### pubspec.yaml:
```yaml
name: zzz
description: Premium Sleep Ritual App
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.5.0
  flutter: ^3.24.0

dependencies:
  flutter:
    sdk: flutter

  # State Management (современный Riverpod с codegen)
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5

  # Navigation (type-safe routes)
  go_router: ^14.2.7

  # Immutable Models + Union Types
  freezed_annotation: ^2.4.4
  json_annotation: ^4.9.0

  # Локальная БД (быстрая NoSQL)
  hive: ^2.2.3
  hive_flutter: ^1.1.0

  # Push Notifications
  flutter_local_notifications: ^17.2.2
  timezone: ^0.9.4

  # Audio (профессиональное решение)
  just_audio: ^0.9.40
  audio_service: ^0.18.15

  # UI & Animations
  google_fonts: ^6.2.1
  flutter_animate: ^4.5.0
  shimmer: ^3.0.0

  # Utilities
  intl: ^0.19.0
  collection: ^1.18.0
  path_provider: ^2.1.4

dev_dependencies:
  flutter_test:
    sdk: flutter

  # Code Generation
  build_runner: ^2.4.12
  freezed: ^2.5.7
  json_serializable: ^6.8.0
  riverpod_generator: ^2.4.3
  hive_generator: ^2.0.1

  # Linting (очень строгие правила)
  flutter_lints: ^4.0.0
  custom_lint: ^0.6.7
  riverpod_lint: ^2.3.13

  # Testing
  mocktail: ^1.0.4

flutter:
  uses-material-design: true
  assets:
    - assets/sounds/
```

### Почему эти пакеты:

**riverpod_annotation + riverpod_generator:**
- Современный Riverpod с code generation
- Автоматическая генерация providers
- Type-safe dependency injection
- Меньше boilerplate кода

**freezed:**
- Immutable модели
- Union types для состояний
- Автоматические `copyWith`, `==`, `toString`
- Pattern matching

**go_router:**
- Декларативная навигация
- Type-safe routes через code generation
- Deep linking из коробки
- Imperative + Declarative navigation

**just_audio + audio_service:**
- Лучшее решение для аудио в Flutter
- Фоновое воспроизведение
- Поддержка iOS background audio
- Media controls в notification

---

## 📁 Feature-First структура проекта

```
lib/
├── main.dart
├── app.dart                          # MaterialApp с routing
│
├── core/
│   ├── theme/
│   │   ├── app_theme.dart            # ThemeData
│   │   └── app_colors.dart           # Color constants
│   ├── constants/
│   │   ├── durations.dart
│   │   └── notification_content.dart
│   ├── router/
│   │   ├── app_router.dart           # GoRouter config
│   │   └── app_router.g.dart         # Generated routes
│   ├── utils/
│   │   ├── extensions/
│   │   │   ├── context_extensions.dart
│   │   │   ├── datetime_extensions.dart
│   │   │   └── string_extensions.dart
│   │   └── logger.dart
│   └── widgets/
│       ├── app_error_widget.dart
│       ├── app_loading_widget.dart
│       └── primary_button.dart
│
├── features/
│   ├── onboarding/
│   │   ├── domain/
│   │   │   └── onboarding_repository.dart    # Interface
│   │   ├── application/
│   │   │   ├── onboarding_notifier.dart
│   │   │   └── onboarding_notifier.g.dart    # Generated
│   │   ├── infrastructure/
│   │   │   └── onboarding_repository_impl.dart
│   │   └── presentation/
│   │       ├── onboarding_screen.dart
│   │       └── widgets/
│   │           ├── onboarding_page.dart
│   │           └── page_indicator.dart
│   │
│   ├── schedule/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   ├── sleep_schedule.dart       # Freezed
│   │   │   │   ├── sleep_schedule.freezed.dart
│   │   │   │   ├── sleep_schedule.g.dart
│   │   │   │   ├── sleep_score.dart          # Freezed
│   │   │   │   └── day_of_week.dart          # Enum
│   │   │   └── repositories/
│   │   │       └── schedule_repository.dart  # Interface
│   │   ├── application/
│   │   │   ├── schedule_notifier.dart
│   │   │   ├── schedule_notifier.g.dart
│   │   │   └── sleep_score_calculator.dart
│   │   ├── infrastructure/
│   │   │   ├── dtos/
│   │   │   │   ├── sleep_schedule_dto.dart   # Hive
│   │   │   │   └── sleep_schedule_dto.g.dart
│   │   │   └── schedule_repository_impl.dart
│   │   └── presentation/
│   │       ├── schedule_setup_screen.dart
│   │       └── widgets/
│   │           ├── day_selector.dart
│   │           ├── sleep_score_badge.dart
│   │           └── recommendation_card.dart
│   │
│   ├── home/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── upcoming_notification.dart
│   │   │   └── repositories/
│   │   │       └── home_repository.dart
│   │   ├── application/
│   │   │   ├── home_notifier.dart
│   │   │   └── timer_notifier.dart           # Для обновления таймера
│   │   ├── infrastructure/
│   │   │   └── home_repository_impl.dart
│   │   └── presentation/
│   │       ├── home_screen.dart
│   │       └── widgets/
│   │           ├── timer_circle.dart
│   │           └── notification_card.dart
│   │
│   ├── soundscapes/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   ├── sound_preset.dart
│   │   │   │   └── mood.dart
│   │   │   └── repositories/
│   │   │       └── audio_repository.dart
│   │   ├── application/
│   │   │   ├── audio_player_notifier.dart
│   │   │   └── soundscapes_notifier.dart
│   │   ├── infrastructure/
│   │   │   ├── audio_handler.dart            # AudioService
│   │   │   └── audio_repository_impl.dart
│   │   └── presentation/
│   │       ├── soundscapes_screen.dart
│   │       └── widgets/
│   │           └── music_preset_tile.dart
│   │
│   ├── statistics/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   ├── sleep_record.dart
│   │   │   │   ├── statistics.dart
│   │   │   │   └── achievement.dart
│   │   │   └── repositories/
│   │   │       └── statistics_repository.dart
│   │   ├── application/
│   │   │   ├── statistics_notifier.dart
│   │   │   └── streak_calculator.dart
│   │   ├── infrastructure/
│   │   │   ├── dtos/
│   │   │   │   └── sleep_record_dto.dart
│   │   │   └── statistics_repository_impl.dart
│   │   └── presentation/
│   │       ├── statistics_screen.dart
│   │       └── widgets/
│   │           ├── stat_card.dart
│   │           └── calendar_grid.dart
│   │
│   ├── notifications/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── notification_config.dart
│   │   │   └── repositories/
│   │   │       └── notification_repository.dart
│   │   ├── application/
│   │   │   └── notification_scheduler.dart
│   │   └── infrastructure/
│   │       └── notification_repository_impl.dart
│   │
│   └── settings/
│       ├── domain/
│       │   ├── entities/
│       │   │   └── user_settings.dart
│       │   └── repositories/
│       │       └── settings_repository.dart
│       ├── application/
│       │   └── settings_notifier.dart
│       ├── infrastructure/
│       │   └── settings_repository_impl.dart
│       └── presentation/
│           └── settings_screen.dart
│
└── shared/
    ├── data/
    │   └── hive_database.dart                # Hive init
    └── providers/
        └── providers.dart                    # Global providers
```

---

## 🎨 Современная тема (Material 3)

### lib/core/theme/app_theme.dart:
```dart
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';
import 'app_colors.dart';

class AppTheme {
  static ThemeData get light => ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: AppColors.deepIndigo,
      brightness: Brightness.light,
      primary: AppColors.deepIndigo,
      secondary: AppColors.softMauve,
      tertiary: AppColors.warmSand,
      surface: Colors.white,
      background: AppColors.cream,
    ),
    
    textTheme: GoogleFonts.interTextTheme().copyWith(
      displayLarge: GoogleFonts.cormorantGaramond(
        fontSize: 42,
        fontWeight: FontWeight.w500,
        letterSpacing: -0.5,
      ),
      displayMedium: GoogleFonts.cormorantGaramond(
        fontSize: 28,
        fontWeight: FontWeight.w500,
        letterSpacing: -0.3,
      ),
      bodyLarge: GoogleFonts.inter(
        fontSize: 15,
        fontWeight: FontWeight.w400,
        height: 1.6,
      ),
    ),
    
    scaffoldBackgroundColor: AppColors.cream,
    
    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        backgroundColor: AppColors.deepIndigo,
        foregroundColor: Colors.white,
        elevation: 0,
        padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 18),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(14),
        ),
      ),
    ),
    
    cardTheme: CardTheme(
      elevation: 0,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(16),
        side: BorderSide(color: AppColors.whisperGray),
      ),
      color: Colors.white,
    ),
  );
}
```

---

## 🔧 Freezed модели (примеры)

### Immutable Entity:
```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'sleep_schedule.freezed.dart';
part 'sleep_schedule.g.dart';

@freezed
class SleepSchedule with _$SleepSchedule {
  const factory SleepSchedule({
    required String id,
    required DayOfWeek dayOfWeek,
    required TimeOfDay plannedBedtime,
    @Default(true) bool isEnabled,
  }) = _SleepSchedule;

  const SleepSchedule._();

  factory SleepSchedule.fromJson(Map<String, dynamic> json) =>
      _$SleepScheduleFromJson(json);

  // Domain logic methods
  bool isTimeAfterMidnight() {
    return plannedBedtime.hour < 6;
  }
}
```

### Union Types для состояний:
```dart
@freezed
class ScheduleState with _$ScheduleState {
  const factory ScheduleState.initial() = _Initial;
  const factory ScheduleState.loading() = _Loading;
  const factory ScheduleState.loaded(List<SleepSchedule> schedules) = _Loaded;
  const factory ScheduleState.error(String message) = _Error;
}
```

---

## 🎯 Riverpod Providers (с code generation)

### Notifier с riverpod_generator:
```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../domain/entities/sleep_schedule.dart';
import '../domain/repositories/schedule_repository.dart';

part 'schedule_notifier.g.dart';

@riverpod
class ScheduleNotifier extends _$ScheduleNotifier {
  @override
  Future<List<SleepSchedule>> build() async {
    final repository = ref.watch(scheduleRepositoryProvider);
    return repository.getAllSchedules();
  }

  Future<void> updateSchedule(SleepSchedule schedule) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      final repository = ref.read(scheduleRepositoryProvider);
      await repository.updateSchedule(schedule);
      return repository.getAllSchedules();
    });
  }
}
```

### Provider для Repository:
```dart
@riverpod
ScheduleRepository scheduleRepository(ScheduleRepositoryRef ref) {
  final database = ref.watch(hiveDatabaseProvider);
  return ScheduleRepositoryImpl(database);
}
```

---

## 🧪 Тестирование (обязательно!)

### Unit Test (domain логика):
```dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('SleepScoreCalculator', () {
    late SleepScoreCalculator calculator;

    setUp(() {
      calculator = SleepScoreCalculator();
    });

    test('should return excellent score for 22:00', () {
      final score = calculator.calculate(const TimeOfDay(hour: 22, minute: 0));
      expect(score.rating, SleepScoreRating.excellent);
      expect(score.value, greaterThanOrEqualTo(85));
    });

    test('should return poor score for 02:00', () {
      final score = calculator.calculate(const TimeOfDay(hour: 2, minute: 0));
      expect(score.rating, SleepScoreRating.poor);
      expect(score.value, lessThan(50));
    });
  });
}
```

### Widget Test:
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  testWidgets('DaySelector shows correct time', (tester) async {
    await tester.pumpWidget(
      ProviderScope(
        child: MaterialApp(
          home: DaySelector(
            schedule: SleepSchedule(
              id: '1',
              dayOfWeek: DayOfWeek.monday,
              plannedBedtime: const TimeOfDay(hour: 22, minute: 0),
            ),
          ),
        ),
      ),
    );

    expect(find.text('22:00'), findsOneWidget);
  });
}
```

---

## ⚙️ Hive с Type Adapters

### DTO для Hive:
```dart
import 'package:hive/hive.dart';

part 'sleep_schedule_dto.g.dart';

@HiveType(typeId: 1)
class SleepScheduleDto {
  @HiveField(0)
  final String id;

  @HiveField(1)
  final int dayOfWeek;

  @HiveField(2)
  final int hour;

  @HiveField(3)
  final int minute;

  @HiveField(4)
  final bool isEnabled;

  const SleepScheduleDto({
    required this.id,
    required this.dayOfWeek,
    required this.hour,
    required this.minute,
    required this.isEnabled,
  });
}
```

### Mapper:
```dart
extension SleepScheduleMapper on SleepScheduleDto {
  SleepSchedule toDomain() => SleepSchedule(
    id: id,
    dayOfWeek: DayOfWeek.values[dayOfWeek],
    plannedBedtime: TimeOfDay(hour: hour, minute: minute),
    isEnabled: isEnabled,
  );
}

extension SleepScheduleDtoMapper on SleepSchedule {
  SleepScheduleDto toDto() => SleepScheduleDto(
    id: id,
    dayOfWeek: dayOfWeek.index,
    hour: plannedBedtime.hour,
    minute: plannedBedtime.minute,
    isEnabled: isEnabled,
  );
}
```

---

## 🔔 Notifications (современный подход)

```dart
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:timezone/timezone.dart' as tz;

class NotificationService {
  final FlutterLocalNotificationsPlugin _plugin;

  Future<void> scheduleWeeklyNotification({
    required int id,
    required String title,
    required String body,
    required DayOfWeek dayOfWeek,
    required TimeOfDay time,
  }) async {
    await _plugin.zonedSchedule(
      id,
      title,
      body,
      _nextInstanceOfWeekday(dayOfWeek, time),
      const NotificationDetails(
        android: AndroidNotificationDetails(
          'sleep_reminders',
          'Sleep Reminders',
          channelDescription: 'Notifications for sleep routine',
          importance: Importance.high,
          priority: Priority.high,
        ),
        iOS: DarwinNotificationDetails(
          presentAlert: true,
          presentBadge: true,
          presentSound: true,
        ),
      ),
      androidScheduleMode: AndroidScheduleMode.exactAllowWhileIdle,
      uiLocalNotificationDateInterpretation:
          UILocalNotificationDateInterpretation.absoluteTime,
      matchDateTimeComponents: DateTimeComponents.dayOfWeekAndTime,
    );
  }

  tz.TzDateTime _nextInstanceOfWeekday(DayOfWeek day, TimeOfDay time) {
    final now = tz.TzDateTime.now(tz.local);
    tz.TzDateTime scheduledDate = tz.TzDateTime(
      tz.local,
      now.year,
      now.month,
      now.day,
      time.hour,
      time.minute,
    );

    while (scheduledDate.weekday != day.index + 1) {
      scheduledDate = scheduledDate.add(const Duration(days: 1));
    }

    if (scheduledDate.isBefore(now)) {
      scheduledDate = scheduledDate.add(const Duration(days: 7));
    }

    return scheduledDate;
  }
}
```

---

## 🎵 Audio Service (профессиональный подход)

```dart
import 'package:audio_service/audio_service.dart';
import 'package:just_audio/just_audio.dart';

class ZzzAudioHandler extends BaseAudioHandler {
  final _player = AudioPlayer();

  ZzzAudioHandler() {
    _notifyAudioHandlerAboutPlaybackEvents();
  }

  Future<void> playSound(String assetPath) async {
    final mediaItem = MediaItem(
      id: assetPath,
      title: 'Sleep Sound',
      artist: 'Zzz',
    );

    mediaItem.value = mediaItem;

    try {
      await _player.setAsset(assetPath);
      await _player.setLoopMode(LoopMode.one);
      await _player.play();
    } catch (e) {
      throw Exception('Failed to play sound: $e');
    }
  }

  @override
  Future<void> play() => _player.play();

  @override
  Future<void> pause() => _player.pause();

  @override
  Future<void> stop() async {
    await _player.stop();
    await super.stop();
  }

  void _notifyAudioHandlerAboutPlaybackEvents() {
    _player.playbackEventStream.listen((event) {
      playbackState.add(playbackState.value.copyWith(
        controls: [
          MediaControl.pause,
          MediaControl.stop,
        ],
        systemActions: const {
          MediaAction.seek,
        },
        processingState: const {
          ProcessingState.idle: AudioProcessingState.idle,
          ProcessingState.loading: AudioProcessingState.loading,
          ProcessingState.buffering: AudioProcessingState.buffering,
          ProcessingState.ready: AudioProcessingState.ready,
          ProcessingState.completed: AudioProcessingState.completed,
        }[_player.processingState]!,
        playing: _player.playing,
        updatePosition: _player.position,
        bufferedPosition: _player.bufferedPosition,
        speed: _player.speed,
      ));
    });
  }
}
```

---

## 🎨 Анимации (flutter_animate)

```dart
import 'package:flutter_animate/flutter_animate.dart';

class OnboardingPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const Text('🌙')
            .animate()
            .fadeIn(duration: 600.ms, curve: Curves.easeOut)
            .slideY(begin: -0.2, end: 0),
        
        Text('Zzz', style: Theme.of(context).textTheme.displayLarge)
            .animate(delay: 200.ms)
            .fadeIn(duration: 600.ms)
            .slideY(begin: 0.2, end: 0),
      ],
    );
  }
}
```

---

## ✅ Правила супер-сеньора

1. **ВСЕГДА используй code generation** — freezed, riverpod_generator, json_serializable
2. **Feature-first структура** — не models/, services/, providers/ в корне
3. **Immutability везде** — все модели через Freezed
4. **Dependency Injection** — все зависимости через Riverpod providers
5. **Тестируй domain логику** — минимум unit тесты для business logic
6. **Type-safe navigation** — go_router с typed routes
7. **Material 3** — useMaterial3: true
8. **Null safety строго** — никогда не используй `!` без проверки
9. **Extensions вместо utils** — context.theme вместо Theme.of(context)
10. **Const everywhere** — все виджеты const где возможно

---

## 🚀 Commands для работы

```bash
# Code generation (запускать после изменений в @freezed/@riverpod)
flutter pub run build_runner build --delete-conflicting-outputs

# Watch mode (автоматическая генерация)
flutter pub run build_runner watch --delete-conflicting-outputs

# Clean + generate
flutter pub run build_runner clean
flutter pub run build_runner build --delete-conflicting-outputs

# Analyze (lint checking)
flutter analyze

# Format code
dart format lib/ -l 100

# Run tests
flutter test

# Coverage
flutter test --coverage
```

---

## 📍 Текущий статус

Смотри **TASKS.md** для прогресса разработки.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GAMaksim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GAMaksim)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
