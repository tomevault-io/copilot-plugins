## flutter

> Reglas de diseño y desarrollo para Money Flow Flutter App


# Money Flow - Reglas de Desarrollo Flutter

## 🚫 REGLAS ESTRICTAS - PROHIBIDO

### ❌ COLORES HARDCODEADOS
```dart
// ❌ NUNCA HACER ESTO
color: Colors.red
color: Color(0xFF123456)
backgroundColor: Colors.grey[50]
Colors.blue[600]

// ✅ SIEMPRE HACER ESTO
color: Theme.of(context).colorScheme.error
color: Theme.of(context).colorScheme.primary
backgroundColor: Theme.of(context).colorScheme.surface
```

### ❌ WIDGETS SIN CONTEXTO DE TEMA
```dart
// ❌ NUNCA HACER ESTO
const Text('Título', style: TextStyle(color: Colors.black))
Container(color: Colors.white)

// ✅ SIEMPRE HACER ESTO
Text('Título', style: TextStyle(color: Theme.of(context).colorScheme.onSurface))
Container(color: Theme.of(context).colorScheme.surfaceContainerHighest)
```

### ❌ ESPACIADO INCONSISTENTE
```dart
// ❌ NUNCA HACER ESTO
const SizedBox(height: 15)
const EdgeInsets.all(13)
const EdgeInsets.only(top: 18, left: 22)

// ✅ SIEMPRE HACER ESTO
const SizedBox(height: 16)  // 8, 16, 24, 32
const EdgeInsets.all(16)    // 8, 16, 24
```

---

## ✅ OBLIGATORIO - SIEMPRE USAR

### 📱 Estructura de Pantalla Estándar
```dart
Scaffold(
  backgroundColor: Theme.of(context).colorScheme.surface,
  appBar: AppBar(
    title: const Text('Título'),
    backgroundColor: Colors.transparent,
    elevation: 0,
    actions: [
      TextButton(
        onPressed: onSave,
        child: const Text(
          'Guardar',
          style: TextStyle(
            fontSize: 16,
            fontWeight: FontWeight.w600,
          ),
        ),
      ),
    ],
  ),
  body: SingleChildScrollView(
    padding: const EdgeInsets.all(24),
    child: Form(
      key: _formKey,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          _buildHeader(),
          const SizedBox(height: 32),
          // Contenido...
        ],
      ),
    ),
  ),
)
```

### 🎨 Sistema de Colores
```dart
// Colores principales
Theme.of(context).colorScheme.primary          // AppColors.primary
Theme.of(context).colorScheme.onPrimary        // AppColors.white
Theme.of(context).colorScheme.primaryContainer // Para badges/chips

// Colores de superficie
Theme.of(context).colorScheme.surface                    // Fondo principal
Theme.of(context).colorScheme.surfaceContainerHighest   // Cards/inputs
Theme.of(context).colorScheme.surfaceContainerHigh      // Contenedores
Theme.of(context).colorScheme.surfaceContainerLow       // Secciones

// Colores de texto
Theme.of(context).colorScheme.onSurface                        // Texto principal
Theme.of(context).colorScheme.onSurface.withValues(alpha: 0.8) // Texto secundario
Theme.of(context).colorScheme.onSurface.withValues(alpha: 0.6) // Texto terciario
Theme.of(context).colorScheme.onSurface.withValues(alpha: 0.4) // Placeholders

// Colores semánticos
Theme.of(context).colorScheme.error           // Errores
Theme.of(context).colorScheme.errorContainer  // Fondo error
Theme.of(context).colorScheme.outline         // Bordes
```

### 📏 Espaciado Estándar
```dart
// Espaciado vertical
const SizedBox(height: 8)     // Pequeño
const SizedBox(height: 16)    // Mediano
const SizedBox(height: 24)    // Grande
const SizedBox(height: 32)    // Extra grande

// Padding de contenedores
const EdgeInsets.all(8)       // Pequeño
const EdgeInsets.all(16)      // Estándar
const EdgeInsets.all(24)      // Screen padding

// Border radius
BorderRadius.circular(8)      // Pequeño
BorderRadius.circular(12)     // Estándar
BorderRadius.circular(16)     // Grande
BorderRadius.circular(20)     // Modal tops
```

---

## 🧩 COMPONENTES OBLIGATORIOS

### 1. Header con Ícono
```dart
Widget _buildHeader() {
  return Row(
    children: [
      Container(
        width: 48,
        height: 48,
        decoration: BoxDecoration(
          color: Theme.of(context).colorScheme.primaryContainer, // Para ingresos
          // color: Theme.of(context).colorScheme.errorContainer, // Para gastos
          borderRadius: BorderRadius.circular(12),
        ),
        child: Icon(
          Icons.trending_up, // trending_up para ingresos, receipt para gastos
          color: Theme.of(context).colorScheme.onPrimaryContainer,
          size: 24,
        ),
      ),
      const SizedBox(width: 16),
      Expanded(
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Título Principal',
              style: TextStyle(
                fontSize: 20,
                fontWeight: FontWeight.bold,
                color: Theme.of(context).colorScheme.onSurface,
              ),
            ),
            Text(
              'Subtítulo descriptivo',
              style: TextStyle(
                fontSize: 14,
                color: Theme.of(context).colorScheme.onSurface.withValues(alpha: 0.6),
              ),
            ),
          ],
        ),
      ),
    ],
  );
}
```

### 2. Campo de Formulario Estándar
```dart
Widget _buildFormField(String label, String hint, TextEditingController controller) {
  return Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      Text(
        label,
        style: TextStyle(
          fontSize: 16,
          fontWeight: FontWeight.w600,
          color: Theme.of(context).colorScheme.onSurface,
        ),
      ),
      const SizedBox(height: 8),
      TextFormField(
        controller: controller,
        decoration: InputDecoration(
          hintText: hint,
          border: OutlineInputBorder(
            borderRadius: BorderRadius.circular(12),
            borderSide: BorderSide(color: Theme.of(context).colorScheme.outline),
          ),
          enabledBorder: OutlineInputBorder(
            borderRadius: BorderRadius.circular(12),
            borderSide: BorderSide(color: Theme.of(context).colorScheme.outline),
          ),
          focusedBorder: OutlineInputBorder(
            borderRadius: BorderRadius.circular(12),
            borderSide: BorderSide(color: Theme.of(context).colorScheme.primary, width: 2),
          ),
          fillColor: Theme.of(context).colorScheme.surfaceContainerHighest,
          filled: true,
        ),
      ),
    ],
  );
}
```

### 3. Campo de Monto
```dart
Widget _buildAmountField() {
  return Consumer<CurrencyProvider>(
    builder: (context, currencyProvider, child) {
      return Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            'Monto',
            style: TextStyle(
              fontSize: 16,
              fontWeight: FontWeight.w600,
              color: Theme.of(context).colorScheme.onSurface,
            ),
          ),
          const SizedBox(height: 8),
          Container(
            padding: const EdgeInsets.all(16),
            decoration: BoxDecoration(
              color: Theme.of(context).colorScheme.surfaceContainerHighest,
              borderRadius: BorderRadius.circular(12),
              border: Border.all(color: Theme.of(context).colorScheme.outline),
            ),
            child: Row(
              children: [
                Text(
                  currencyProvider.currencySymbol,
                  style: TextStyle(
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                    color: Theme.of(context).colorScheme.onSurface.withValues(alpha: 0.6),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: TextFormField(
                    controller: _amountController,
                    keyboardType: const TextInputType.numberWithOptions(decimal: true),
                    style: TextStyle(
                      fontSize: 24,
                      fontWeight: FontWeight.bold,
                      color: Theme.of(context).colorScheme.onSurface,
                    ),
                    decoration: InputDecoration(
                      hintText: '0.00',
                      border: InputBorder.none,
                      hintStyle: TextStyle(
                        fontSize: 24,
                        fontWeight: FontWeight.bold,
                        color: Theme.of(context).colorScheme.onSurface.withValues(alpha: 0.4),
                      ),
                    ),
                  ),
                ),
              ],
            ),
          ),
        ],
      );
    },
  );
}
```

### 4. Selector con Modal
```dart
Widget _buildSelector(String label, String? selectedValue, String hint, VoidCallback onTap) {
  return Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      Text(
        label,
        style: TextStyle(
          fontSize: 16,
          fontWeight: FontWeight.w600,
          color: Theme.of(context).colorScheme.onSurface,
        ),
      ),
      const SizedBox(height: 8),
      InkWell(
        onTap: onTap,
        borderRadius: BorderRadius.circular(12),
        child: Container(
          width: double.infinity,
          padding: const EdgeInsets.all(16),
          decoration: BoxDecoration(
            color: Theme.of(context).colorScheme.surfaceContainerHighest,
            borderRadius: BorderRadius.circular(12),
            border: Border.all(color: Theme.of(context).colorScheme.outline),
          ),
          child: Row(
            children: [
              if (selectedValue != null) ...[
                // Mostrar valor seleccionado
                Expanded(
                  child: Text(
                    selectedValue,
                    style: TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.w600,
                      color: Theme.of(context).colorScheme.onSurface,
                    ),
                  ),
                ),
              ] else ...[
                Icon(Icons.category, color: Theme.of(context).colorScheme.onSurface.withValues(alpha: 0.6)),
                const SizedBox(width: 12),
                Expanded(
                  child: Text(
                    hint,
                    style: TextStyle(
                      fontSize: 16,
                      color: Theme.of(context).colorScheme.onSurface.withValues(alpha: 0.6),
                    ),
                  ),
                ),
              ],
              Icon(Icons.keyboard_arrow_down, color: Theme.of(context).colorScheme.onSurface.withValues(alpha: 0.6)),
            ],
          ),
        ),
      ),
    ],
  );
}
```

### 5. Botón Principal
```dart
Widget _buildPrimaryButton(String text, VoidCallback? onPressed, {bool isLoading = false}) {
  return SizedBox(
    width: double.infinity,
    child: ElevatedButton(
      onPressed: isLoading ? null : onPressed,
      style: ElevatedButton.styleFrom(
        padding: const EdgeInsets.symmetric(vertical: 16),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(12),
        ),
      ),
      child: isLoading
          ? const SizedBox(
              height: 20,
              width: 20,
              child: CircularProgressIndicator(
                strokeWidth: 2,
                valueColor: AlwaysStoppedAnimation<Color>(Colors.white),
              ),
            )
          : Text(
              text,
              style: const TextStyle(
                fontSize: 16,
                fontWeight: FontWeight.w600,
              ),
            ),
    ),
  );
}
```

### 6. Modal Bottom Sheet
```dart
void _showBottomSheetModal(BuildContext context, String title, Widget content) {
  showModalBottomSheet(
    context: context,
    isScrollControlled: true,
    backgroundColor: Colors.transparent,
    builder: (context) => Container(
      height: MediaQuery.of(context).size.height * 0.6,
      decoration: BoxDecoration(
        color: Theme.of(context).colorScheme.surface,
        borderRadius: const BorderRadius.vertical(top: Radius.circular(20)),
      ),
      child: Column(
        children: [
          Container(
            width: 40,
            height: 4,
            margin: const EdgeInsets.symmetric(vertical: 12),
            decoration: BoxDecoration(
              color: Theme.of(context).colorScheme.onSurface.withValues(alpha: 0.3),
              borderRadius: BorderRadius.circular(2),
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(20),
            child: Text(
              title,
              style: TextStyle(
                fontSize: 20,
                fontWeight: FontWeight.bold,
                color: Theme.of(context).colorScheme.onSurface,
              ),
            ),
          ),
          Expanded(child: content),
        ],
      ),
    ),
  );
}
```

### 7. Campos Opcionales (ExpansionTile)
```dart
Widget _buildOptionalFields() {
  return ExpansionTile(
    title: Text(
      'Información adicional (opcional)',
      style: TextStyle(
        fontSize: 16,
        fontWeight: FontWeight.w600,
        color: Theme.of(context).colorScheme.onSurface,
      ),
    ),
    children: [
      const SizedBox(height: 16),
      // Campos opcionales aquí
      const SizedBox(height: 16),
    ],
  );
}
```

---

## 🎯 MANEJO DE ESTADOS

### Loading States
```dart
// En botones
child: isLoading
    ? const SizedBox(
        height: 20,
        width: 20,
        child: CircularProgressIndicator(strokeWidth: 2),
      )
    : Text('Texto del botón')

// En pantallas
if (provider.isLoading) {
  return const Center(child: CircularProgressIndicator());
}
```

### Error States
```dart
if (provider.error != null) {
  WidgetsBinding.instance.addPostFrameCallback((_) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text(provider.error!),
        backgroundColor: Theme.of(context).colorScheme.error,
      ),
    );
  });
}
```

### Consumer Pattern
```dart
// ✅ SIEMPRE usar Consumer para providers
Consumer<ExpenseProvider>(
  builder: (context, provider, child) {
    // UI que depende del provider
  },
)

// ❌ NUNCA hacer context.watch() directo en build
```

---

## 📁 ARQUITECTURA DE ARCHIVOS

### Estructura Obligatoria
```
lib/
├── core/
│   ├── app/              # Configuración de app
│   ├── providers/        # Providers globales
│   ├── services/         # Servicios
│   └── theme/           # Sistema de temas
├── features/
│   └── feature_name/
│       ├── data/        # Models, repositories
│       └── presentation/ # Screens, providers, widgets
├── shared/
│   ├── screens/         # Pantallas compartidas
│   └── widgets/         # Widgets reutilizables
└── main.dart
```

### Nombres de Archivos
```dart
// ✅ CORRECTO: snake_case
add_expense_screen.dart
expense_provider.dart
category_model.dart

// ❌ INCORRECTO: PascalCase o camelCase
AddExpenseScreen.dart
expenseProvider.dart
```

---

## 🔧 VALIDACIONES Y TESTING

### Validación de Formularios
```dart
// ✅ SIEMPRE validar antes de submit
if (!_formKey.currentState!.validate()) {
  return;
}

if (_selectedCategory == null) {
  ScaffoldMessenger.of(context).showSnackBar(
    const SnackBar(
      content: Text('Selecciona una categoría'),
      backgroundColor: Colors.orange,
    ),
  );
  return;
}
```

### Mounted Check
```dart
// ✅ SIEMPRE verificar mounted en async operations
if (success && mounted) {
  Navigator.of(context).pop();
}

// ✅ En setState después de async
if (!mounted) return;
setState(() {
  // update state
});
```

---

## 📋 CHECKLIST PRE-COMMIT

- [ ] No hay colores hardcodeados
- [ ] Se usa `Theme.of(context)` para todos los colores
- [ ] Los espaciados siguen el estándar (8, 16, 24, 32)
- [ ] Los border radius son consistentes (8, 12, 16, 20)
- [ ] Los componentes siguen los patrones establecidos
- [ ] Hay validación de estados de loading/error
- [ ] Se verifica `mounted` en operaciones async
- [ ] Los formularios tienen validación apropiada
- [ ] Se usan Consumer para providers
- [ ] Los nombres de archivos están en snake_case
- [ ] La estructura de directorios es correcta
- [ ] Se usan componentes Glassmorphism en lugar de BackdropFilter manual
- [ ] Las animaciones de entrada están habilitadas en componentes importantes
- [ ] Los efectos hover están configurados apropiadamente

## 🚀 COMANDOS DE VERIFICACIÓN

```bash
# Buscar colores hardcodeados
grep -r "Colors\." lib/ --exclude-dir=build
grep -r "Color(0x" lib/ --exclude-dir=build

# Verificar imports del tema
grep -r "app_theme.dart" lib/

# Verificar que no hay const en widgets que usan Theme.of(context)
grep -r "const.*Theme.of(context)" lib/

# Verificar uso correcto de componentes Glassmorphism
grep -r "BackdropFilter" lib/ --exclude-dir=shared/widgets
grep -r "ImageFilter.blur" lib/ --exclude-dir=shared/widgets

# Verificar imports de glassmorphism widgets
grep -r "glassmorphism_widgets.dart" lib/
```

---

## 🌟 SISTEMA GLASSMORPHISM - COMPONENTES AVANZADOS

### 📋 COMPONENTES DISPONIBLES

#### 1. **GlassmorphismCard** - Cards con efectos glass
```dart
import 'package:money_flow/shared/widgets/glassmorphism_widgets.dart';

// ✅ USAR PARA: Cards principales, contenedores importantes
GlassmorphismCard(
  style: GlassStyles.dynamic,        // light, medium, heavy, dynamic
  enableHoverEffect: true,           // Efectos hover sofisticados
  enableEntryAnimation: true,        // Animaciones de entrada
  animationDuration: Duration(milliseconds: 800),
  child: YourContent(),
)
```

#### 2. **GlassmorphismButton** - Botones con efectos avanzados
```dart
// ✅ USAR PARA: Botones principales, CTAs, acciones importantes
GlassmorphismButton(
  style: GlassButtonStyles.primary,  // primary, secondary, outline, floating
  enablePulseEffect: true,           // Pulsación continua
  enableRippleEffect: true,          // Ondas al tocar
  onPressed: () => doSomething(),
  child: Text('Action'),
)
```

#### 3. **GlassmorphismListItem** - Items de lista animados
```dart
// ✅ USAR PARA: Listas de transacciones, elementos importantes
GlassmorphismListItem(
  enableSlideAnimation: true,        // Entrada desde la derecha
  enableHoverEffect: true,           // Hover effects
  index: index,                      // Para animaciones secuenciales
  leading: Icon(Icons.payment),
  title: Text('Transaction'),
  subtitle: Text('Category'),
  trailing: Text('-\$50.00'),
  onTap: () => navigate(),
)
```

### 🎨 ESTILOS DISPONIBLES

#### **Card Styles:**
```dart
GlassStyles.light    // Sutil, para elementos secundarios
GlassStyles.medium   // Estándar, para la mayoría de cards
GlassStyles.heavy    // Intenso, para elementos destacados
GlassStyles.dynamic  // Blur variable, para cards principales
```

#### **Button Styles:**
```dart
GlassButtonStyles.primary   // Botón principal con color theme
GlassButtonStyles.secondary // Botón secundario translúcido
GlassButtonStyles.outline   // Botón con borde, fondo transparente
GlassButtonStyles.floating  // Botón flotante con sombra intensa
```

### ✨ CARACTERÍSTICAS AUTOMÁTICAS

#### **Animaciones de Entrada:**
- **Elastic scale**: Entrada suave con rebote
- **Fade in**: Aparición gradual con opacidad
- **Retrasos aleatorios**: Para efecto natural escalonado
- **Slide animations**: Para listas (derecha → izquierda)

#### **Efectos Hover:**
- **Scale sutil**: Crecimiento 1.01x - 1.02x
- **Blur dinámico**: Intensificación del efecto glass
- **Glow effect**: Resplandor con color theme
- **Border enhancement**: Bordes más brillantes

#### **Blur Dinámico:**
- **Variación continua**: Blur que cambia constantemente
- **Intensidad adaptativa**: Más intenso en hover
- **Performance optimizado**: GPU-accelerated effects

### 🚫 REGLAS GLASSMORPHISM

#### ❌ **NO HACER:**
```dart
// ❌ BackdropFilter manual (usar componentes)
ClipRRect(
  child: BackdropFilter(
    filter: ImageFilter.blur(sigmaX: 10, sigmaY: 10),
    child: Container(...),
  ),
)

// ❌ Hardcodear valores de blur
BackdropFilter(filter: ImageFilter.blur(sigmaX: 15, sigmaY: 15))

// ❌ Colores hardcodeados en glass effects
Colors.white.withOpacity(0.1)
```

#### ✅ **SIEMPRE HACER:**
```dart
// ✅ Usar componentes glassmorphism
GlassmorphismCard(
  style: GlassStyles.medium,
  child: YourContent(),
)

// ✅ Aprovechar animaciones automáticas
GlassmorphismButton(
  enablePulseEffect: true,
  enableRippleEffect: true,
  child: Text('Action'),
)

// ✅ Usar estilos predefinidos
GlassmorphismListItem(
  enableSlideAnimation: true,
  index: index, // Para animaciones secuenciales
)
```

### 🎯 CASOS DE USO RECOMENDADOS

#### **Dashboard Cards:**
```dart
GlassmorphismCard(
  style: GlassStyles.dynamic,     // Blur variable
  enableHoverEffect: true,
  child: _buildCardContent(),
)
```

#### **Botones de Acción:**
```dart
GlassmorphismButton(
  style: GlassButtonStyles.floating,
  enablePulseEffect: true,
  onPressed: () => addTransaction(),
)
```

#### **Listas de Transacciones:**
```dart
Column(
  children: transactions.map((transaction) {
    final index = transactions.indexOf(transaction);
    return GlassmorphismListItem(
      index: index,              // Animaciones escalonadas
      enableSlideAnimation: true,
      leading: CategoryIcon(),
      title: Text(transaction.name),
      onTap: () => viewDetails(transaction),
    );
  }).toList(),
)
```

### 📱 RESPONSIVE BEHAVIOR

- **Hover effects**: Solo en desktop/web (detección automática)
- **Touch feedback**: Ripple effects en móvil
- **Performance scaling**: Reduce blur en dispositivos de gama baja
- **Adaptive theming**: Colores automáticos según modo claro/oscuro

---

## 💡 EJEMPLOS RÁPIDOS

### ✅ Implementación Correcta
```dart
Container(
  padding: const EdgeInsets.all(16),
  decoration: BoxDecoration(
    color: Theme.of(context).colorScheme.surfaceContainerHighest,
    borderRadius: BorderRadius.circular(12),
    border: Border.all(color: Theme.of(context).colorScheme.outline),
  ),
  child: Text(
    'Contenido',
    style: TextStyle(
      fontSize: 16,
      fontWeight: FontWeight.w600,
      color: Theme.of(context).colorScheme.onSurface,
    ),
  ),
)
```

### ❌ Implementación Incorrecta
```dart
const Container(
  padding: EdgeInsets.all(20),
  decoration: BoxDecoration(
    color: Colors.white,
    borderRadius: BorderRadius.circular(15),
    border: Border.all(color: Colors.grey),
  ),
  child: Text(
    'Contenido',
    style: TextStyle(
      fontSize: 18,
      fontWeight: FontWeight.bold,
      color: Colors.black,
    ),
  ),
)
```

---

**RECORDATORIO**: Estas reglas son OBLIGATORIAS para mantener la consistencia visual y la calidad del código en Money Flow. El cumplimiento de estas reglas es esencial para una experiencia de usuario cohesiva y un código mantenible.

---
> Source: [nick130920/fintech-backend](https://github.com/nick130920/fintech-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
