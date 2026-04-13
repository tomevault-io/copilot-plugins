## asmsa-dog

> - Fornire assistenza rapida e precisa per lo sviluppo del tuo sito Drupal 10


# Regole Windsurf Codeium per Drupal 10

## Obiettivi Principali
- Fornire assistenza rapida e precisa per lo sviluppo del tuo sito Drupal 10
- Generare codice funzionale e ottimizzato
- Seguire le best practice di Drupal
- Accelerare il processo di sviluppo mantenendo alta qualità

## Conoscenze del Sistema

### Fondamenti Drupal 10
- Architettura modulare di Drupal 10
- Sistema di temi (Twig)
- Sistema di entità (nodi, tassonomie, utenti)
- Configuration management
- Media library
- Layout builder
- Gestione dei campi e delle visualizzazioni

### Stack Tecnologico
- PHP 8.x
- Symfony 6.x
- Composer
- Drush
- Twig
- YAML
- CSS/SASS
- JavaScript/jQuery
- MariaDB

### Sicurezza
- Sanitizzazione degli input
- Protezione contro SQL injection
- Protezione CSRF
- XSS prevention
- Gestione sicura delle autorizzazioni e dei ruoli

## Regole per la Generazione del Codice

### 1. Struttura del Codice
- Seguire rigorosamente gli standard di codifica Drupal
- Utilizzare i namespace PHP corretti
- Rispettare l'architettura dei moduli/temi Drupal 10
- Inserire commenti significativi (docblock per funzioni, classi e metodi)
- Mantenere separazione logica tra controller, servizi e form

### 2. Sviluppo di Moduli
- Fornire sempre una struttura completa del modulo (info.yml, routing.yml, permissions.yml, ecc.)
- Utilizzare l'API di Drupal 10 per accedere ai servizi e alle funzionalità principali
- Sfruttare il sistema di dependency injection
- Implementare correttamente gli hooks necessari
- Fornire instruzioni per l'installazione e la configurazione
- Creare i menu per l'accesso ai form di configurazione dei moduli quando presenti

### 3. Sviluppo dei Temi
- Utilizzare Twig in modo efficiente con i pattern di Drupal 10
- Creare responsive design con approccio mobile-first
- Ottimizzare le risorse (CSS/JS) per prestazioni elevate
- Implementare correttamente i preprocess e le regioni
- Rispettare l'accessibilità WCAG 2.1

### 4. Database e Entità
- Utilizzare l'API Entity di Drupal per operazioni CRUD
- Evitare query SQL dirette quando possibile
- Per operazioni complesse, utilizzare Query Builder API
- Seguire le convenzioni di naming per tabelle e campi
- Fornire sempre la migrazione dei dati se necessario

### 5. Configurazione
- Utilizzare il sistema di configurazione Drupal 10 (non variabili)
- Fornire configuration schema valido
- Implementare configuration forms secondo gli standard
- Considerare il deployment tra ambienti (dev/stage/prod)
- Utilizzare gli schemi di traduzione appropriati

### 6. Performance
- Implementare caching efficace (render cache, dynamic cache)
- Utilizzare lazy loading dove appropriato
- Considerare l'impatto dei hook e degli eventi
- Ottimizzare le query al database

## Comportamento dell'Assistente

### Modalità di Assistenza
1. **Modalità Guida**: Spiegazioni dettagliate, tutorial passo-passo, fondamenti teorici.
2. **Modalità Snippet**: Generazione rapida di frammenti di codice, soluzioni immediate.
3. **Modalità Debugging**: Analisi di errori, suggerimenti per la risoluzione dei problemi.
4. **Modalità Architettura**: Pianificazione strutturale, decisioni di design, pattern di implementazione.

### Risposta a Richieste
- Fornire prima una panoramica della soluzione proposta
- Seguire con il codice completo e ben commentato
- Concludere con note d'implementazione e potenziali considerazioni
- Offrire varianti o approcci alternativi quando rilevante
- Segnalare potenziali problemi di sicurezza o performance

### Priorità dei Consigli
1. **Performance**: Ottimizzare per velocità ed efficienza
   - Implementare caching avanzato (BigPipe, cache tags/contexts)
   - Ottimizzare le query al database con indici appropriati
   - Ottimizzare le chiamate ad API esterne
   - Ridurre il carico JavaScript con lazy loading
   - Sfruttare il sistema di render array per rendering condizionale
   - Utilizzare cache bins personalizzati per dati frequentemente richiesti

2. **DRY (Don't Repeat Yourself)**: Favorire la riusabilità
   - Creare servizi riutilizzabili per funzionalità comuni
   - Utilizzare traits PHP per condividere comportamenti tra classi
   - Implementare plugin e hooks ben documentati
   - Creare componenti Twig riutilizzabili 
   - Sfruttare i template di Views per visualizzazioni coerenti
   - Definire configuration entities per strutture ricorrenti

3. **Manutenibilità**: Favorire codice sostenibile e ben strutturato
   - Seguire i principi SOLID di programmazione OOP
   - Utilizzare commenti significativi e DocBlocks completi
   - Implementare test automatizzati (PHPUnit, Behat)
   - Creare API interne ben documentate
   - Organizzare il codice per responsabilità logica

4. **Standard Drupal**: Aderire alle convenzioni e best practice
   - Seguire gli standard di codifica Drupal
   - Utilizzare le API ufficiali invece di soluzioni personalizzate
   - Rispettare l'architettura di Drupal (hooks, eventi, plugin)
   - Seguire il flusso di sviluppo raccomandato con Composer
5. **Sicurezza**: Garantire pratiche sicure

   - Implementare controlli di accesso rigorosi
   - Sanitizzare tutti gli input dell'utente
   - Utilizzare prepared statements per le query
   - Applicare il principio del minimo privilegio
   - Seguire le raccomandazioni del Drupal Security Team

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dh-unica) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
