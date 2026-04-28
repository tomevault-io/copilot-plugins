## habit-system-discord

> 🧠 Erweiterte Schritte – Klar & Effektiv für Cursor

🧠 Erweiterte Schritte – Klar & Effektiv für Cursor
🎯 Ziel (Goal) definieren

Formuliere in einem Satz: Warum baust du das Projekt, für wen, und welchen konkreten Nutzen es bringt.

Schreibe es als Cursor-Notiz oder Rule, z. B.:

vbnet
Kopieren
Bearbeiten
Goal: Build a file‑upload tool that allows non‑technical users to easily share documents securely, with automatic virus scanning.
👤 User Stories formulieren

Pro Nutzerhilfe: „Als Nutzer möchte ich …, damit ….“

Halte sie simpel und anwendungsorientiert, z. B.:

css
Kopieren
Bearbeiten
- As a user, I can register an account so I can track my uploads.
- As a user, I can upload a document so I can share it.
📦 Datenmodell skizzieren

Fasse Entitäten plus Schlüsselattribute und Beziehungen in wenigen Sätzen zusammen:

pgsql
Kopieren
Bearbeiten
Entities: User(id, email, password), Document(id, userId, filename, virusScanStatus)
Relation: A User can have many Documents.
Nutze Cursor Rules, um dieses Schema allen Teammitgliedern verfügbar zu machen.

🔪 MVP definieren

Frage: „Was muss das Projekt absolut können?“ – eliminiere alles Nicht‑Essenzielle.

Beispiel:

javascript
Kopieren
Bearbeiten
MVP: User registration, document upload, document listing, basic virus scan status.
📝 Grober Wireframe / Flow skizzieren

Zeichne—auch handschriftlich—Screenabläufe wie Login → Upload → Liste.

Notiere im Rule-File als ASCII-Sketch oder kurze Bullet-Navigation.

🔭 Zukunftsperspektive klären

Lege fest: Prototyp, MVP‑Produkt oder skalierbare Plattform?

Beispielregel:

pgsql
Kopieren
Bearbeiten
Mode: Prototype (not production‑ready). Use SQLite & local file storage.
🧱 Architekturentscheidung treffen

Bestimme Komponenten: Backend‑API, Frontend‑Web, Datenbank, Cronjobs etc.

Dokumentiere sie im Rule-File, z. B.:

diff
Kopieren
Bearbeiten
Components:
- REST API (Node.js/Express)
- Frontend (React)
- Database (SQLite)
⚙️ Tech‑Stack wählen

Nutze ein bestehendes Skillset – kein Experiment.

Notiere stichpunktartig:

kotlin
Kopieren
Bearbeiten
Stack: React (frontend), Node.js+Express (API), SQLite (data), Docker (deployment)
🚀 Entwicklungsprozess strukturieren

Dokumentiere Sequenz & Workflow im Rule-File:

markdown
Kopieren
Bearbeiten
Process:
1. Initialize repo + version control.
2. Implement data models & migrations.
3. Build & test backend endpoints.
4. Build frontend screens & integrate API.
5. Iteratively add features, test, and deploy.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MarcBaumholz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
