## viridisca-frontend

> Frontend UI/UX руководство ViridiscaUi LMS - дизайн-система, компоненты, адаптивность, CRUD

# FRONTEND UI/UX GUIDE - VIRIDISCA LMS

## 🎨 ДИЗАЙН-СИСТЕМА И ПРИНЦИПЫ

### Основные принципы UI/UX:
1. **Минимализм** - чистый, незагроможденный интерфейс
2. **Консистентность** - единообразие во всех компонентах
3. **Адаптивность** - отзывчивый дизайн для всех устройств
4. **Доступность** - соответствие стандартам WCAG 2.1
5. **Производительность** - быстрая загрузка и отзывчивость
6. **Интуитивность** - понятная навигация без обучения

### Цветовая палитра:
```scss
// Основные цвета
$primary: #2563EB;      // Синий (основной)
$secondary: #10B981;    // Зеленый (успех)
$accent: #F59E0B;       // Оранжевый (акцент)
$danger: #EF4444;       // Красный (ошибки)
$warning: #F59E0B;      // Желтый (предупреждения)
$info: #3B82F6;         // Голубой (информация)

// Нейтральные цвета
$gray-50: #F9FAFB;
$gray-100: #F3F4F6;
$gray-200: #E5E7EB;
$gray-300: #D1D5DB;
$gray-400: #9CA3AF;
$gray-500: #6B7280;
$gray-600: #4B5563;
$gray-700: #374151;
$gray-800: #1F2937;
$gray-900: #111827;

// Темная тема
$dark-bg: #0F172A;
$dark-surface: #1E293B;
$dark-border: #334155;
$dark-text: #F1F5F9;
```

### Типографика:
```scss
// Шрифты
$font-primary: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
$font-mono: 'JetBrains Mono', 'Fira Code', monospace;

// Размеры
$text-xs: 0.75rem;    // 12px
$text-sm: 0.875rem;   // 14px
$text-base: 1rem;     // 16px
$text-lg: 1.125rem;   // 18px
$text-xl: 1.25rem;    // 20px
$text-2xl: 1.5rem;    // 24px
$text-3xl: 1.875rem;  // 30px
$text-4xl: 2.25rem;   // 36px

// Веса
$font-light: 300;
$font-normal: 400;
$font-medium: 500;
$font-semibold: 600;
$font-bold: 700;
```

### Отступы и размеры:
```scss
// Spacing scale (8px base)
$space-1: 0.25rem;   // 4px
$space-2: 0.5rem;    // 8px
$space-3: 0.75rem;   // 12px
$space-4: 1rem;      // 16px
$space-5: 1.25rem;   // 20px
$space-6: 1.5rem;    // 24px
$space-8: 2rem;      // 32px
$space-10: 2.5rem;   // 40px
$space-12: 3rem;     // 48px
$space-16: 4rem;     // 64px

// Border radius
$radius-sm: 0.25rem;  // 4px
$radius-md: 0.375rem; // 6px
$radius-lg: 0.5rem;   // 8px
$radius-xl: 0.75rem;  // 12px
$radius-2xl: 1rem;    // 16px
```

## 🏗️ АРХИТЕКТУРА СТИЛЕЙ

### Структура папок стилей:
```
Styles/
├── Base/
│   ├── Reset.axaml           # CSS Reset
│   ├── Typography.axaml      # Типографика
│   ├── Colors.axaml          # Цветовая палитра
│   └── Spacing.axaml         # Отступы и размеры
├── Components/
│   ├── Buttons.axaml         # Кнопки всех типов
│   ├── Forms.axaml           # Формы и поля ввода
│   ├── Cards.axaml           # Карточки
│   ├── Tables.axaml          # Таблицы
│   ├── Navigation.axaml      # Навигация
│   ├── Modals.axaml          # Модальные окна
│   ├── Alerts.axaml          # Уведомления
│   └── Charts.axaml          # Графики и диаграммы
├── Layout/
│   ├── Grid.axaml            # Сетка
│   ├── Containers.axaml      # Контейнеры
│   ├── Sidebar.axaml         # Боковая панель
│   └── Header.axaml          # Шапка
├── Pages/
│   ├── Dashboard.axaml       # Стили дашборда
│   ├── Education.axaml       # Образовательные страницы
│   ├── Auth.axaml            # Аутентификация
│   └── System.axaml          # Системные страницы
├── Themes/
│   ├── Light.axaml           # Светлая тема
│   ├── Dark.axaml            # Темная тема
│   └── HighContrast.axaml    # Высокий контраст
└── Utils/
    ├── Animations.axaml      # Анимации
    ├── Shadows.axaml         # Тени
    └── Utilities.axaml       # Утилитарные классы
```

### Принципы переиспользования:
1. **Атомарные компоненты** - мелкие переиспользуемые элементы
2. **Композиция** - сборка сложных компонентов из простых
3. **Модификаторы** - классы для изменения поведения
4. **Состояния** - стили для разных состояний элементов
5. **Responsive mixins** - адаптивные стили

## 📱 АДАПТИВНЫЙ ДИЗАЙН

### Breakpoints:
```scss
$breakpoints: (
  xs: 0px,      // Мобильные (портрет)
  sm: 640px,    // Мобильные (ландшафт)
  md: 768px,    // Планшеты
  lg: 1024px,   // Десктоп малый
  xl: 1280px,   // Десктоп средний
  2xl: 1536px   // Десктоп большой
);
```

### Адаптивная сетка:
```xml
<!-- 12-колоночная сетка -->
<Grid Name="ResponsiveGrid">
  <Grid.ColumnDefinitions>
    <ColumnDefinition Width="*" />
    <ColumnDefinition Width="*" />
    <ColumnDefinition Width="*" />
    <ColumnDefinition Width="*" />
    <ColumnDefinition Width="*" />
    <ColumnDefinition Width="*" />
    <ColumnDefinition Width="*" />
    <ColumnDefinition Width="*" />
    <ColumnDefinition Width="*" />
    <ColumnDefinition Width="*" />
    <ColumnDefinition Width="*" />
    <ColumnDefinition Width="*" />
  </Grid.ColumnDefinitions>
</Grid>
```

## 🎯 ОСНОВНЫЕ КОМПОНЕНТЫ UI

### 1. Кнопки (Buttons.axaml):
```xml
<!-- Основная кнопка -->
<Style Selector="Button.btn-primary">
  <Setter Property="Background" Value="{StaticResource PrimaryBrush}" />
  <Setter Property="Foreground" Value="White" />
  <Setter Property="Padding" Value="16,12" />
  <Setter Property="CornerRadius" Value="8" />
  <Setter Property="FontWeight" Value="Medium" />
  <Setter Property="BorderThickness" Value="0" />
  <Setter Property="Cursor" Value="Hand" />
</Style>

<!-- Вторичная кнопка -->
<Style Selector="Button.btn-secondary">
  <Setter Property="Background" Value="Transparent" />
  <Setter Property="Foreground" Value="{StaticResource PrimaryBrush}" />
  <Setter Property="BorderBrush" Value="{StaticResource PrimaryBrush}" />
  <Setter Property="BorderThickness" Value="1" />
  <Setter Property="Padding" Value="16,12" />
  <Setter Property="CornerRadius" Value="8" />
</Style>

<!-- Кнопка опасности -->
<Style Selector="Button.btn-danger">
  <Setter Property="Background" Value="{StaticResource DangerBrush}" />
  <Setter Property="Foreground" Value="White" />
</Style>

<!-- Размеры кнопок -->
<Style Selector="Button.btn-sm">
  <Setter Property="Padding" Value="12,8" />
  <Setter Property="FontSize" Value="14" />
</Style>

<Style Selector="Button.btn-lg">
  <Setter Property="Padding" Value="20,16" />
  <Setter Property="FontSize" Value="18" />
</Style>
```

### 2. Формы (Forms.axaml):
```xml
<!-- Поле ввода -->
<Style Selector="TextBox.form-input">
  <Setter Property="Padding" Value="12,10" />
  <Setter Property="BorderBrush" Value="{StaticResource Gray300Brush}" />
  <Setter Property="BorderThickness" Value="1" />
  <Setter Property="CornerRadius" Value="6" />
  <Setter Property="FontSize" Value="16" />
  <Setter Property="Background" Value="White" />
</Style>

<!-- Фокус -->
<Style Selector="TextBox.form-input:focus">
  <Setter Property="BorderBrush" Value="{StaticResource PrimaryBrush}" />
  <Setter Property="BorderThickness" Value="2" />
</Style>

<!-- Ошибка -->
<Style Selector="TextBox.form-input.error">
  <Setter Property="BorderBrush" Value="{StaticResource DangerBrush}" />
</Style>

<!-- Лейбл -->
<Style Selector="TextBlock.form-label">
  <Setter Property="FontWeight" Value="Medium" />
  <Setter Property="FontSize" Value="14" />
  <Setter Property="Foreground" Value="{StaticResource Gray700Brush}" />
  <Setter Property="Margin" Value="0,0,0,6" />
</Style>
```

### 3. Карточки (Cards.axaml):
```xml
<!-- Базовая карточка -->
<Style Selector="Border.card">
  <Setter Property="Background" Value="White" />
  <Setter Property="BorderBrush" Value="{StaticResource Gray200Brush}" />
  <Setter Property="BorderThickness" Value="1" />
  <Setter Property="CornerRadius" Value="12" />
  <Setter Property="Padding" Value="24" />
  <Setter Property="BoxShadow" Value="0 1 3 0 rgba(0,0,0,0.1)" />
</Style>

<!-- Карточка с ховером -->
<Style Selector="Border.card-hover:pointerover">
  <Setter Property="BoxShadow" Value="0 4 6 -1 rgba(0,0,0,0.1)" />
  <Setter Property="BorderBrush" Value="{StaticResource Gray300Brush}" />
</Style>

<!-- Статистическая карточка -->
<Style Selector="Border.stat-card">
  <Setter Property="Background" Value="White" />
  <Setter Property="CornerRadius" Value="16" />
  <Setter Property="Padding" Value="20" />
  <Setter Property="BoxShadow" Value="0 2 4 0 rgba(0,0,0,0.05)" />
</Style>
```

### 4. Таблицы (Tables.axaml):
```xml
<!-- Контейнер таблицы -->
<Style Selector="Border.table-container">
  <Setter Property="Background" Value="White" />
  <Setter Property="CornerRadius" Value="12" />
  <Setter Property="BorderBrush" Value="{StaticResource Gray200Brush}" />
  <Setter Property="BorderThickness" Value="1" />
  <Setter Property="ClipToBounds" Value="True" />
</Style>

<!-- Заголовок таблицы -->
<Style Selector="Border.table-header">
  <Setter Property="Background" Value="{StaticResource Gray50Brush}" />
  <Setter Property="Padding" Value="16,12" />
  <Setter Property="BorderBrush" Value="{StaticResource Gray200Brush}" />
  <Setter Property="BorderThickness" Value="0,0,0,1" />
</Style>

<!-- Строка таблицы -->
<Style Selector="Border.table-row">
  <Setter Property="Padding" Value="16,12" />
  <Setter Property="BorderBrush" Value="{StaticResource Gray100Brush}" />
  <Setter Property="BorderThickness" Value="0,0,0,1" />
</Style>

<Style Selector="Border.table-row:pointerover">
  <Setter Property="Background" Value="{StaticResource Gray50Brush}" />
</Style>
```

## 📄 СТРУКТУРА СТРАНИЦ

### 1. Главная страница (Dashboard):
```
┌─────────────────────────────────────────────────────────────┐
│ Header (Навигация, Профиль, Уведомления)                   │
├─────────────────────────────────────────────────────────────┤
│ Sidebar │ Main Content Area                                 │
│         │ ┌─────────────────────────────────────────────┐   │
│ - Home  │ │ Welcome Section                             │   │
│ - Edu   │ │ ┌─────────┐ ┌─────────┐ ┌─────────┐       │   │
│ - Sys   │ │ │ Stat 1  │ │ Stat 2  │ │ Stat 3  │       │   │
│ - Lib   │ │ └─────────┘ └─────────┘ └─────────┘       │   │
│         │ └─────────────────────────────────────────────┘   │
│         │ ┌─────────────────────────────────────────────┐   │
│         │ │ Quick Actions                               │   │
│         │ │ [New Course] [Add Student] [Schedule]       │   │
│         │ └─────────────────────────────────────────────┘   │
│         │ ┌─────────────────────────────────────────────┐   │
│         │ │ Recent Activity Feed                        │   │
│         │ └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 2. Список сущностей (Students, Teachers, etc.):
```
┌─────────────────────────────────────────────────────────────┐
│ Header с хлебными крошками                                  │
├─────────────────────────────────────────────────────────────┤
│ Sidebar │ ┌─────────────────────────────────────────────┐   │
│         │ │ Page Title + [Add New] Button               │   │
│         │ ├─────────────────────────────────────────────┤   │
│         │ │ Filters & Search                            │   │
│         │ │ [Search] [Filter] [Sort] [Export]           │   │
│         │ ├─────────────────────────────────────────────┤   │
│         │ │ Data Table                                  │   │
│         │ │ ┌─────┬─────────┬─────────┬─────────┬─────┐ │   │
│         │ │ │ ☐   │ Name    │ Email   │ Status  │ ... │ │   │
│         │ │ ├─────┼─────────┼─────────┼─────────┼─────┤ │   │
│         │ │ │ ☐   │ John    │ j@...   │ Active  │ ... │ │   │
│         │ │ └─────┴─────────┴─────────┴─────────┴─────┘ │   │
│         │ ├─────────────────────────────────────────────┤   │
│         │ │ Pagination                                  │   │
│         │ │ [<] 1 2 3 ... 10 [>] (50 of 500)          │   │
│         │ └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 3. Форма создания/редактирования:
```
┌─────────────────────────────────────────────────────────────┐
│ Header с хлебными крошками                                  │
├─────────────────────────────────────────────────────────────┤
│ Sidebar │ ┌─────────────────────────────────────────────┐   │
│         │ │ Form Title                                  │   │
│         │ ├─────────────────────────────────────────────┤   │
│         │ │ ┌─────────────────┐ ┌─────────────────┐     │   │
│         │ │ │ Personal Info   │ │ Academic Info   │     │   │
│         │ │ │ - First Name    │ │ - Student Code  │     │   │
│         │ │ │ - Last Name     │ │ - Group         │     │   │
│         │ │ │ - Email         │ │ - Curriculum    │     │   │
│         │ │ │ - Phone         │ │ - Status        │     │   │
│         │ │ └─────────────────┘ └─────────────────┘     │   │
│         │ ├─────────────────────────────────────────────┤   │
│         │ │ Form Actions                                │   │
│         │ │ [Cancel] [Save Draft] [Save & Continue]     │   │
│         │ └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## 🎭 МОДАЛЬНЫЕ ОКНА И ПОПАПЫ

### 1. Подтверждение удаления:
```xml
<Border Classes="modal-overlay">
  <Border Classes="modal-content modal-sm">
    <StackPanel Spacing="16">
      <!-- Иконка предупреждения -->
      <Border Classes="icon-warning">
        <PathIcon Data="{StaticResource WarningIcon}" />
      </Border>
      
      <!-- Заголовок -->
      <TextBlock Classes="modal-title">
        Подтвердите удаление
      </TextBlock>
      
      <!-- Описание -->
      <TextBlock Classes="modal-description">
        Вы уверены, что хотите удалить студента "Иван Иванов"? 
        Это действие нельзя отменить.
      </TextBlock>
      
      <!-- Кнопки -->
      <StackPanel Orientation="Horizontal" Spacing="12">
        <Button Classes="btn-secondary">Отмена</Button>
        <Button Classes="btn-danger">Удалить</Button>
      </StackPanel>
    </StackPanel>
  </Border>
</Border>
```

### 2. Быстрое создание:
```xml
<Border Classes="modal-overlay">
  <Border Classes="modal-content modal-lg">
    <Grid RowDefinitions="Auto,*,Auto">
      <!-- Заголовок -->
      <Border Grid.Row="0" Classes="modal-header">
        <Grid ColumnDefinitions="*,Auto">
          <TextBlock Grid.Column="0" Classes="modal-title">
            Быстрое добавление студента
          </TextBlock>
          <Button Grid.Column="1" Classes="btn-icon">
            <PathIcon Data="{StaticResource CloseIcon}" />
          </Button>
        </Grid>
      </Border>
      
      <!-- Содержимое -->
      <ScrollViewer Grid.Row="1" Classes="modal-body">
        <!-- Форма -->
      </ScrollViewer>
      
      <!-- Футер -->
      <Border Grid.Row="2" Classes="modal-footer">
        <StackPanel Orientation="Horizontal" Spacing="12">
          <Button Classes="btn-secondary">Отмена</Button>
          <Button Classes="btn-primary">Сохранить</Button>
        </StackPanel>
      </Border>
    </Grid>
  </Border>
</Border>
```

## 🔔 СИСТЕМА УВЕДОМЛЕНИЙ

### 1. Toast уведомления:
```xml
<!-- Успех -->
<Border Classes="toast toast-success">
  <Grid ColumnDefinitions="Auto,*,Auto">
    <PathIcon Grid.Column="0" Data="{StaticResource CheckIcon}" />
    <StackPanel Grid.Column="1" Spacing="4">
      <TextBlock Classes="toast-title">Успешно!</TextBlock>
      <TextBlock Classes="toast-message">Студент добавлен</TextBlock>
    </StackPanel>
    <Button Grid.Column="2" Classes="toast-close">×</Button>
  </Grid>
</Border>

<!-- Ошибка -->
<Border Classes="toast toast-error">
  <!-- Аналогично с красным цветом -->
</Border>

<!-- Предупреждение -->
<Border Classes="toast toast-warning">
  <!-- Аналогично с желтым цветом -->
</Border>
```

### 2. Inline уведомления:
```xml
<!-- Информационное -->
<Border Classes="alert alert-info">
  <Grid ColumnDefinitions="Auto,*">
    <PathIcon Grid.Column="0" Data="{StaticResource InfoIcon}" />
    <TextBlock Grid.Column="1">
      Для завершения регистрации проверьте email
    </TextBlock>
  </Grid>
</Border>

<!-- Ошибка валидации -->
<Border Classes="alert alert-error">
  <Grid ColumnDefinitions="Auto,*">
    <PathIcon Grid.Column="0" Data="{StaticResource ErrorIcon}" />
    <StackPanel Grid.Column="1">
      <TextBlock Classes="alert-title">Ошибки в форме:</TextBlock>
      <ItemsControl ItemsSource="{Binding ValidationErrors}">
        <ItemsControl.ItemTemplate>
          <DataTemplate>
            <TextBlock Text="{Binding}" Classes="alert-item" />
          </DataTemplate>
        </ItemsControl.ItemTemplate>
      </ItemsControl>
    </StackPanel>
  </Grid>
</Border>
```

## 📊 CRUD ФУНКЦИОНАЛ ПО РОЛЯМ

### 1. Студенты (Students):

#### SystemAdmin / AcademicAffairsHead:
- ✅ **Create**: Полная форма создания с всеми полями
- ✅ **Read**: Просмотр всех студентов с фильтрами
- ✅ **Update**: Редактирование всех полей
- ✅ **Delete**: Удаление с подтверждением

#### Teacher (контекст: свои курсы):
- ❌ **Create**: Нет доступа
- ✅ **Read**: Только студенты своих курсов
- ✅ **Update**: Только академические данные (оценки, посещаемость)
- ❌ **Delete**: Нет доступа

#### GroupCurator (контекст: своя группа):
- ❌ **Create**: Нет доступа  
- ✅ **Read**: Все студенты своей группы
- ✅ **Update**: Контактная информация, академический статус
- ❌ **Delete**: Нет доступа

#### Student:
- ❌ **Create**: Нет доступа
- ✅ **Read**: Только свои данные
- ✅ **Update**: Только контактная информация
- ❌ **Delete**: Нет доступа

#### Parent (контекст: свои дети):
- ❌ **Create**: Нет доступа
- ✅ **Read**: Только данные своих детей
- ❌ **Update**: Нет доступа
- ❌ **Delete**: Нет доступа

### 2. Курсы (CourseInstances):

#### SystemAdmin / AcademicAffairsHead:
- ✅ **Create**: Создание экземпляров курсов
- ✅ **Read**: Все курсы
- ✅ **Update**: Все параметры курса
- ✅ **Delete**: Удаление курсов

#### Teacher (контекст: назначенные курсы):
- ❌ **Create**: Нет доступа к созданию
- ✅ **Read**: Свои назначенные курсы
- ✅ **Update**: Содержание, задания, оценки
- ❌ **Delete**: Нет доступа

#### Student (контекст: записанные курсы):
- ❌ **Create**: Нет доступа
- ✅ **Read**: Курсы, на которые записан
- ❌ **Update**: Нет доступа
- ❌ **Delete**: Нет доступа

### 3. Библиотека (Library):

#### Librarian:
- ✅ **Create**: Добавление ресурсов
- ✅ **Read**: Все ресурсы и займы
- ✅ **Update**: Информация о ресурсах
- ✅ **Delete**: Удаление ресурсов

#### Teacher / Student:
- ❌ **Create**: Нет доступа
- ✅ **Read**: Каталог ресурсов
- ✅ **Update**: Запрос на займ
- ❌ **Delete**: Нет доступа

## 🎨 АНИМАЦИИ И ПЕРЕХОДЫ

### Базовые анимации:
```xml
<!-- Появление элементов -->
<Style Selector="Border.fade-in">
  <Style.Animations>
    <Animation Duration="0:0:0.3" FillMode="Forward">
      <KeyFrame Cue="0%">
        <Setter Property="Opacity" Value="0" />
        <Setter Property="TranslateTransform.Y" Value="10" />
      </KeyFrame>
      <KeyFrame Cue="100%">
        <Setter Property="Opacity" Value="1" />
        <Setter Property="TranslateTransform.Y" Value="0" />
      </KeyFrame>
    </Animation>
  </Style.Animations>
</Style>

<!-- Ховер эффекты -->
<Style Selector="Button:pointerover">
  <Style.Animations>
    <Animation Duration="0:0:0.2">
      <KeyFrame Cue="100%">
        <Setter Property="ScaleTransform.ScaleX" Value="1.02" />
        <Setter Property="ScaleTransform.ScaleY" Value="1.02" />
      </KeyFrame>
    </Animation>
  </Style.Animations>
</Style>
```

## 📱 МОБИЛЬНАЯ АДАПТАЦИЯ

### Принципы мобильного дизайна:
1. **Touch-friendly** - минимум 44px для кликабельных элементов
2. **Упрощенная навигация** - гамбургер меню
3. **Вертикальные стеки** - элементы друг под другом
4. **Увеличенные отступы** - больше пространства между элементами
5. **Скрытие второстепенного** - показ только важной информации

### Мобильная навигация:
```xml
<!-- Мобильное меню -->
<Border Classes="mobile-menu" IsVisible="{Binding IsMobileMenuOpen}">
  <StackPanel Spacing="0">
    <Button Classes="menu-item">
      <StackPanel Orientation="Horizontal" Spacing="12">
        <PathIcon Data="{StaticResource HomeIcon}" />
        <TextBlock>Главная</TextBlock>
      </StackPanel>
    </Button>
    <!-- Другие пункты меню -->
  </StackPanel>
</Border>
```

## 🔍 ПОИСК И ФИЛЬТРАЦИЯ

### Интерфейс поиска:
```xml
<Border Classes="search-container">
  <Grid ColumnDefinitions="*,Auto,Auto,Auto">
    <!-- Поиск -->
    <TextBox Grid.Column="0" 
             Classes="search-input"
             Watermark="Поиск студентов..." />
    
    <!-- Фильтры -->
    <Button Grid.Column="1" Classes="filter-btn">
      <StackPanel Orientation="Horizontal" Spacing="6">
        <PathIcon Data="{StaticResource FilterIcon}" />
        <TextBlock>Фильтры</TextBlock>
      </StackPanel>
    </Button>
    
    <!-- Сортировка -->
    <ComboBox Grid.Column="2" Classes="sort-select">
      <ComboBoxItem>По имени</ComboBoxItem>
      <ComboBoxItem>По дате</ComboBoxItem>
    </ComboBox>
    
    <!-- Экспорт -->
    <Button Grid.Column="3" Classes="export-btn">
      <PathIcon Data="{StaticResource ExportIcon}" />
    </Button>
  </Grid>
</Border>
```

---

**Этот UI/UX гайд обеспечивает создание современного, доступного и интуитивного интерфейса для всех пользователей системы.** 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Viridisca)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Viridisca)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
