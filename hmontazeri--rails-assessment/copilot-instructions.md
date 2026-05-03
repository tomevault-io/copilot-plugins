## rails-assessment

> Baue und pflege eine **modulare Ruby on Rails 8 Engine** (Gem) namens **`rails-assessment`**,

# 🧠 agents.md — Rails Assessment Project

## 🎯 Mission

Baue und pflege eine **modulare Ruby on Rails 8 Engine** (Gem) namens **`rails-assessment`**,  
die interaktive, dynamisch konfigurierbare Fragebögen („Assessments“) mit bedingter Logik und anpassbarem Look & Feel bereitstellt.

Der Code soll **Headless**, **konfigurationsbasiert** und **einbettbar** sein – ideal für SaaS-, Marketing- oder Lead-Generation-Systeme.

---

## ⚙️ Zielarchitektur

### 1️⃣ Core Engine (Mountable Rails Engine)

- Namespace: `Rails::Assessment`
- Mountable unter `/assessments`
- Keine Admin-UI, kein CRUD
- DSL / YAML-basierte Definition von Assessments
- Responses werden persistent gespeichert (JSONB)

### 2️⃣ Core Components

| Komponente           | Zweck                                                      |
| -------------------- | ---------------------------------------------------------- |
| `Assessment`         | Definition eines Fragebogens (aus YAML oder Ruby DSL)      |
| `Question`           | Text + Optionen, nur in Config definiert                   |
| `ResultRule`         | Bedingte Logik („wenn diese Tags, dann dieser Text“)       |
| `Response`           | Benutzerantworten (persistiert)                            |
| `LogicEngine`        | Evaluierung der Antworten gegen die Regeln                 |
| `ThemeResolver`      | Rendert CSS-Variablen basierend auf konfigurierbarem Theme |
| `StimulusController` | Step-by-Step Flow (Next / Back / Finish) mit Turbo Frames  |

---

## 🧩 Features (MVP Scope)

### ✅ Assessments definieren

- Via Ruby DSL (`Rails::Assessment.define`) oder YAML in `config/assessments/*.yml`.
- DSL / Parser konvertiert in interne Ruby-Struktur (`AssessmentDefinition`).
- Optional pro Assessment Theme-Overrides.

### ✅ Logic Engine

- Evaluierung über Tags:
  ```ruby
  LogicEngine.evaluate(selected_tags, rules)
  ```
- Findet erstes passendes Regel-Set; fallback-Text wenn keine Regel zutrifft.

### ✅ Persistence

```ruby
create_table :assessment_responses do |t|
  t.string :assessment_slug
  t.jsonb  :answers
  t.string :result
  t.timestamps
end
```

### ✅ Rendering

- Turbo Frames + Stimulus Controller (Hotwire)
- Ein Frage-Screen pro Frame
- Dynamisches Ergebnis nach letzter Antwort

### ✅ Theming

- Konfigurierbar per Initializer (`config/initializers/rails_assessment.rb`)
- Tokens: `colors`, `typography`, `radius`, `shadow`, `dark_mode`
- CSS-Variablen-Injection im Layout
- Tailwind-Preset (`tailwind.preset.cjs`) optional
- Mehrere Themes & Resolver-Strategien:
  - `:initializer` (fix)
  - `:param` (z. B. `?theme=dark`)
  - `:proc` (per Request-Lambda)

---

## 🧠 Architekturüberblick

rails-assessment/
├── lib/
│ ├── rails-assessment.rb
│ └── rails/assessment/
│ ├── engine.rb
│ ├── version.rb
│ ├── logic_engine.rb
│ ├── configuration.rb
│ ├── dsl/
│ │ ├── builder.rb
│ │ └── loader.rb
│ └── theme/
│ ├── resolver.rb
│ └── helpers.rb
├── app/
│ ├── controllers/rails/assessment/
│ │ ├── assessments_controller.rb
│ │ └── responses_controller.rb
│ ├── views/rails/assessment/
│ │ ├── layouts/application.html.erb
│ │ ├── assessments/
│ │ │ ├── show.html.erb
│ │ │ └── result.html.erb
│ │ └── shared/
│ ├── javascript/controllers/
│ │ └── assessment_controller.js
│ └── assets/stylesheets/rails/assessment/theme.css
├── config/
│ └── routes.rb
├── db/
│ └── migrations/
└── docs/
└── agents.md

---

## 🧰 Aufgaben des Coding-Agents

1. **DSL & Loader entwickeln**

   - Ruby-Parser für `Rails::Assessment.define`
   - YAML-Importer (`config/assessments/*.yml`)
   - Caching & Reloading im Development

2. **Logic Engine implementieren**

   - Tag-Matching-System
   - Support für Kombinationen, Scoring, Fallbacks

3. **Theming-System integrieren**

   - Initializer laden (`Rails::Assessment.configure`)
   - CSS-Var-Injection & Tailwind-Preset
   - ThemeResolver API + Helper (`tkn`, `current_assessment_theme`)

4. **Frontend-Flow**

   - Stimulus Controller: `next`, `back`, `submit`
   - Turbo Frames Navigation
   - Graceful Fallback ohne JS

5. **Persistence & Tracking**

   - Responses speichern
   - (optional) Webhook oder MailerHook nach Abschluss

6. **Testing & Quality**

   - Minitest / RSpec / Systemtests
   - CI Checks: Rubocop, StandardRB
   - Sicherstellen, dass Engine mountable & isoliert ist

7. **Docs**
   - `README.md` (Usage)
   - `agents.md` (diese Datei)
   - Beispiel-Assessment (`smart_home_check.yml`)
   - Beispiel-Initializer (`config/initializers/rails_assessment.rb`)

---

## 🧩 Erweiterungs-Roadmap (nach v0.1)

| Version | Feature                                  |
| ------- | ---------------------------------------- |
| v0.2    | YAML-Export / Import CLI                 |
| v0.3    | Webhooks / Lead Integrationen            |
| v0.4    | Embedded Mode (iframe für Landing Pages) |
| v0.5    | AI-gestützte Result-Text Vorschläge      |
| v0.6    | Mehrsprachigkeit (i18n pro Assessment)   |

---

## 🧠 Coding-Style & Konventionen

- Rails 8 / Ruby 3.3
- Solid Queue optional (keine Abhängigkeit)
- Naming: `rails-assessment` (snake_case) für gemspec
- Namespace: `Rails::Assessment`
- Keine ActiveRecord-Modelle außer `Response`
- DSL/Config > DB
- Verwende Hotwire, kein React
- Theme-Variablen > Hardcoded CSS
- Keine Inline-Styles im Ruby-Code außer Token-Injection

---

## 🚀 Lokaler Testlauf

```bash
# im Host-Projekt
rails plugin new rails-assessment --mountable
cd rails-assessment
# Dateien aus docs/agents.md implementieren
bundle exec rspec
rails s
# open http://localhost:3000/assessments/demo
```

---

## 🧭 Erfolgskriterien

✅ Läuft isoliert als Gem  
✅ Konfiguration ausschließlich über YAML/Ruby DSL + Initializer  
✅ Frontend minimal, aber ästhetisch & Hotwire-ready  
✅ Theming variabel via Config  
✅ Responses persistieren  
✅ Kein Admin UI, kein CRUD  
✅ Doku + Beispiel-Assessment vollständig

---
> Source: [hmontazeri/rails-assessment](https://github.com/hmontazeri/rails-assessment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
