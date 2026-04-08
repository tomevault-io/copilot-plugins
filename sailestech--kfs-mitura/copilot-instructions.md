## kfs-mitura

> cd "/Users/charlie/PycharmProjects/formularz/formularz-vite"

# Formularz Vite - Claude Development Guide

## 🚀 Szybki Start dla Claude

### Uruchomienie aplikacji
```bash
cd "/Users/charlie/PycharmProjects/formularz/formularz-vite"
npm run dev
```
Aplikacja dostępna na: http://localhost:5173/

**⚠️ WAŻNE: ZAWSZE używaj narzędzia Bash do uruchamiania serwera!**
- Server automatycznie wyłącza się gdy Claude kończy pracę z innymi narzędziami
- Tylko Bash tool utrzymuje server w działaniu w tle
- Jeśli server padnie, zawsze uruchom ponownie przez `Bash` tool

**🔧 Rozwiązywanie problemów z portem:**
```bash
# Jeśli port 5173 jest zablokowany, zabij proces:
kill $(lsof -ti:5173)

# Lub uruchom na innym porcie:
npx vite --port 3000
```

**🚀 STABILNY SERVER - zawsze działający:**
```bash
# Uruchom server jako daemon (nie kończy się po zakończeniu zadania):
cd "/Users/charlie/PycharmProjects/formularz/formularz-vite"
nohup npm run dev > /dev/null 2>&1 & 

# Sprawdź czy działa:
curl -s -o /dev/null -w "Status: %{http_code}" http://localhost:5173

# Zabij daemon jeśli potrzeba:
pkill -f "npm.*dev"
```

### Sprawdzenie jakości kodu
```bash
# TypeScript i budowanie
npm run build

# ESLint
npm run lint
```

## 📁 Mapa Kluczowych Plików

### 🎯 Główne komponenty logiczne
- `src/App.tsx` - Root aplikacji
- `src/components/FormWizard.tsx` - Główna logika kroków formularza
- `src/components/steps/EmployeesStep.tsx` - Zarządzanie listą pracowników  
- `src/components/ModernEmployeeCard.tsx` - Edycja/widok pojedynczego pracownika

### 📋 Definicje i konfiguracja
- `src/types/index.ts` - Wszystkie typy TypeScript
- `src/constants/texts.ts` - Teksty aplikacji
- `src/services/airtableServiceSecure.ts` - Integracja z Airtable (używa field IDs zamiast nazw)

## 🗄️ Airtable Field ID Mapping

**WAŻNE: Airtable integracja używa field IDs zamiast nazw dla odporności na zmiany**

### Company Fields (Dane podmiotu)
- `submission_id` → `fldb2lUUPqVyg3qHJ`
- `company_name` → `fldWKTMxAQILBkDKr`
- `company_nip` → `fldOrZL39rXQFy41x`
- `representative_person` → `fldJBWA0L39GHhbzN`
- `representative_email` → `fld2L1bM5FxT4p2Vs`
- `link_do_formularza` → `fldGAYlZU8vRDh7lG`
- (i pozostałe...)

### Employee Fields (Pracownicy)
- `employee_name` → `fld42KA9aezSe7K7k`
- `gender` → `fldl8rKWB7NTlJzKa`
- `education` → `fldQjfALgtcEAjg1m`
- `position` → `fldQGMKAJcVwU2lAQ`
- `application_id` → `fldX8Bp2PpYuVFpjy`
- (i pozostałe...)

**Dlaczego field IDs?**
- Odporność na zmiany nazw pól w Airtable
- Konsystentność API nawet gdy użytkownik zmienia nazwy
- Mniejsze ryzyko błędów przy aktualizacjach schematu

### 🎨 Style i UI
- `src/styles/design-system.css` - Zmienne CSS, kolory, spacing
- `src/styles/components.css` - Style komponentów
- `src/components/ui/` - Komponenty UI (Button, Input, FormField)

## 🛠️ Standardy Debugowania

### 1. Błędy TypeScript
**Zawsze sprawdź typy przed edycją**
```typescript
// ❌ Unikaj
const employee: any = data;

// ✅ Używaj
const employee: Employee = data;
```

### 2. Walidacja formularzy
**Pattern walidacji:**
```typescript
const newErrors: Record<string, string> = {};
if (!formData.name?.trim()) newErrors.name = 'Pole wymagane';
setErrors(newErrors);

// Czyszczenie błędów
if (errors[field]) {
  setErrors(prev => ({ ...prev, [field]: '' }));
}
```

### 3. Stan komponentów
**Stan pracowników:**
```typescript
// employees: EmployeeCollection = Record<string, Employee>
// Klucz = ID pracownika, Wartość = obiekt Employee
```

### 4. Najczęstsze błędy
- **text-white na białym tle** - użyj `text-gray-900`
- **brak typów** - zawsze importuj z `types/index.ts`
- **case w switch bez bloków** - owijaj w `{}`

## 📊 Przepływ Danych

### FormWizard State
```typescript
const [currentStep, setCurrentStep] = useState(1);
const [companyData, setCompanyData] = useState<CompanyData>({...});
const [employees, setEmployees] = useState<EmployeeCollection>({});
```

### Employee Management
```typescript
// Dodawanie
const newEmployee: Employee = {
  id: newId,
  name: '',
  isEditing: true,
  isNew: true
};

// Aktualizacja
onChange({
  ...employees,
  [id]: { ...employee, ...updates }
});

// Usuwanie
const newEmployees = { ...employees };
delete newEmployees[id];
onChange(newEmployees);
```

## 🎨 Konwencje UI/UX

### Responsywne siatki
```css
.form-grid {
  display: grid;
  gap: var(--space-8);
  grid-template-columns: 1fr;
}

@media (min-width: 768px) {
  .form-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
```

### Kolory semantyczne
- `text-gray-900` - główny tekst
- `text-gray-600` - dodatkowy tekst  
- `text-primary-600` - akcenty
- `text-success-600` - powodzenie
- `text-error-600` - błędy

### Ikony
```typescript
// ✅ Używaj emoji dla szybkiego rozwoju
<span className="mr-2">🏢</span>
<span className="mr-2">👥</span>

// Lub małe SVG
<svg className="w-4 h-4" viewBox="0 0 20 20" fill="currentColor">
```

## 🔧 Najczęstsze Zadania

### Dodanie nowego pola do Employee
1. **Typ** - zaktualizuj `Employee` interface w `types/index.ts`
2. **Formularz** - dodaj do `ModernEmployeeCard.tsx`
3. **Walidacja** - dodaj sprawdzenie w `handleSave()`
4. **Widok** - zaktualizuj `ReviewStep.tsx`

### Debugowanie problemów z pracownikami
1. Sprawdź stan `employees` w React DevTools
2. Zweryfikuj czy `employee.id` jest unikalny
3. Sprawdź `isEditing` i `isNew` flags
4. Przetestuj `updateEmployee` i `removeEmployee`

### Stylowanie komponentów
1. Używaj klas z `components.css`
2. Sprawdź zmienne w `design-system.css`
3. Testuj responsywność na różnych ekranach
4. Upewnij się o kontraście kolorów

## ⚠️ Znane Problemy i Rozwiązania

### ESLint Errors
```typescript
// Problem: case declarations
switch (step) {
  case 1:
    const component = <Component />; // ❌
    
  case 1: {
    const component = <Component />; // ✅
  }
}
```

### TypeScript Strict Mode
```typescript
// Problem: any types
employee: any // ❌

// Rozwiązanie: explicit types
employee: Employee // ✅
```

### CSS text-white Issues
```css
/* Problem: biały tekst na białym tle */
.text-white { color: white; } /* Na białym bg */

/* Rozwiązanie */
.text-gray-900 { color: #111827; }
```

## 🚦 Status Sprawdzania

Przed commitem zawsze uruchom:
```bash
npm run lint  # Sprawdź błędy kodu
npm run build # Sprawdź błędy TypeScript
```

### Krytyczne pliki do sprawdzenia
- `FormWizard.tsx` - główna logika
- `EmployeesStep.tsx` - zarządzanie pracownikami  
- `ModernEmployeeCard.tsx` - edycja pracowników
- `ReviewStep.tsx` - widok przeglądu

## 💡 Tips dla Efektywnej Pracy

1. **🚨 ZAWSZE uruchamiaj server przez Bash tool: `npm run dev`**
2. **Server pada przy przełączaniu narzędzi - restartuj przez Bash**
3. **Sprawdź typy TypeScript przed edycją**
4. **Testuj na różnych ilościach pracowników (1, 5, 10+)**
5. **Używaj React DevTools do sprawdzania stanu**
6. **Sprawdzaj responsywność na mobile**

## 🔍 Debugowanie Specyficznych Problemów

### Problem: Pracownik się nie zapisuje
```typescript
// Sprawdź walidację
console.log('Validation errors:', newErrors);
// Sprawdź stan
console.log('Form data:', formData);
// Sprawdź callback
console.log('onUpdate called with:', updates);
```

### Problem: Lista pracowników nie aktualizuje się
```typescript
// Sprawdź klucze w employees object
console.log('Employees keys:', Object.keys(employees));
// Sprawdź czy ID jest string
console.log('Employee ID type:', typeof employee.id);
```

### Problem: Style nie działają
```css
/* Sprawdź czy importy CSS są w main.tsx */
import './index.css'
import './styles/design-system.css'
import './styles/components.css'
```

## 🧹 TODO: Czyszczenie kodu

### Usunięcie niepotrzebnych bibliotek
Po przywróceniu natywnych inputów dat, trzeba usunąć:
```bash
npm uninstall react-datepicker @types/react-datepicker date-fns
```

### Usuniecie zakomentowanych stylów
W `src/styles/components.css` są zakomentowane style dla react-datepicker - można je usunąć całkowicie.

Ten plik pomoże w szybkim rozpoczęciu pracy i debugowaniu najczęstszych problemów.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SailesTech)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SailesTech)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
