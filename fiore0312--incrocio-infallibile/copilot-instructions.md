## incrocio-infallibile

> **Tipo Progetto**: Api

# incrocio-infallibile

**Tipo Progetto**: Api
**Framework**: Custom PHP
**Linguaggi**: php, markdown, sql, javascript, css
**Generato**: 2025-07-05

## Descrizione Progetto

Employee Analytics Dashboard sviluppato con architettura PHP custom per gestione dipendenti, KPI e analisi performance.

## Regole Context Engineering

### Best Practices Generali
- Usa nomi descrittivi
- Mantieni codice pulito
- Implementa testing
- Documenta changes
- Segui convenzioni del linguaggio

### Workflow
1. Analizza requisiti
2. Pianifica implementazione
3. Implementa soluzione
4. Testa funzionalità
5. Documenta changes

## Setup Ambiente di Sviluppo


### PHP Custom Setup
```bash
# Configura database
cp settings.php.example settings.php

# Configura credenziali database
edit settings.php

# Crea struttura database
mysql -u root -p < database_schema.sql

# Avvia server di sviluppo
php -S localhost:8000
```

### Testing
```bash
# Test componenti
php test_smart_parser.php

# Test database
php database_structure_analysis.php

# Verifica integrità dati
php verify_data_integrity.php
```




---
*Configurazione generata automaticamente da Context Engineering Agent*


## PHP Custom Specifiche

### Convenzioni Progetto
- Usa prepared statements per database security
- Implementa classi in /classes/ directory
- Usa autoloading con spl_autoload_register
- Implementa logging strutturato
- Gestione errori centralizzata

### Struttura File
```
/classes/        # Classi principali
/config/         # Configurazioni
/logs/           # File di log
/tasks/          # Documentazione e task
/uploads/        # File caricati
index.php        # Dashboard principale
settings.php     # Configurazione
```

### Componenti Principali
- **CsvParser**: Parsing e validazione CSV
- **Database**: Gestione connessioni DB
- **Configuration**: Gestione configurazioni
- **Master Data Console**: Dashboard analytics

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Fiore0312)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Fiore0312)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
