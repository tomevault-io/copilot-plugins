## westfamilytree

> WestFamilyTree - Cursor Rules

WestFamilyTree - Cursor Rules

Projektöversikt

WestFamilyTree är ett släktforskningsverktyg byggt med:

Frontend: React + Vite + Tailwind CSS

Backend: Electron (Node.js)

Databas: SQLite

Språk: JavaScript/JSX, kommentarer på svenska

Viktiga Arkitekturprinciper

1. Media-hantering (KRITISKT!)

ALLA bildsökvägar ska vara relativa från media/ mappen

Använd INTE absoluta sökvägar (t.ex. C:\Users\...)

Format: media://persons/filename.png eller persons/filename.png

Rekursiv sökning: Om en bild inte hittas på exakt sökväg, sök automatiskt i alla undermappar

Använd MediaImage komponenten för att visa bilder (hanterar media:// URLs automatiskt)

Använd useMediaUrl hook för att konvertera media:// URLs till blob URLs

read-file IPC handler och registerFileProtocol hanterar rekursiv sökning automatiskt

2. Auto-save Mekanism

ALLA ändringar sparas automatiskt i realtid - INGA undantag!

Använd state wrappers (setDbData, setPerson, setFamilyTreeFocusPersonId, etc.) som automatiskt sätter isDirty(true)

Auto-save triggas via useAppContext.js med 300ms debounce

Använd setDbDataRaw endast vid initial load för att undvika onödiga saves

handleEditFormChange använder dbDataRef.current för att undvika stale closures

3. Databasstruktur (SQLite)

Tabeller: people, sources, places, media, meta, relations

Mappning: Frontend använder sex, databas använder gender - mappa vid save/load

Relations: Lagras som JSON i people.relations, men dbData.relations array byggs via buildRelationsFromPeople

Media: Lagras som JSON array i people.media kolumn

Migrations: Använd ALTER TABLE för att lägga till nya kolumner, kör FÖRE CREATE TABLE IF NOT EXISTS

4. State Management

Global state: useAppContext.js via React Context

Viktiga state wrappers:

setDbData - automatiskt isDirty(true)

setPerson - automatiskt onChange callback

setFamilyTreeFocusPersonId - automatiskt isDirty(true)

Relations sync: Använd syncRelations.js för tvåvägssynkronisering

Deep copying: Använd JSON.parse(JSON.stringify(obj)) för att undvika mutationer

5. Komponentstruktur

Huvudkomponenter:

App.jsx - Routing och global state

EditPersonModal.jsx - Redigera personer

MediaManager.jsx - Hantera media

SourceCatalog.jsx - Hantera källor

PlaceCatalogNew.jsx - Hantera platser

MediaSelector.jsx - Välj/länka media

Hooks:

useMediaUrl - Konvertera media:// URLs till blob URLs

useApp - Global app context

Komponenter:

MediaImage - Visa bilder med automatisk media:// hantering

6. Electron IPC

Viktiga handlers:

read-file - Läsa filer med rekursiv sökning i undermappar

save-database - Spara till SQLite med transaktioner

open-database - Öppna SQLite databas

registerFileProtocol('media') - Hantera media:// URLs för <img> taggar

Sökvägar: Använd ALLTID relativa sökvägar från media/ mappen

Rekursiv sökning: Implementerad i read-file och registerFileProtocol

7. Kodningskonventioner

Kommentarer: På svenska

Variabelnamn: camelCase

Komponentnamn: PascalCase

Filer: PascalCase för komponenter (t.ex. EditPersonModal.jsx), camelCase för utilities

Debug logging: Använd console.log('[ComponentName] ...') med komponentnamn i prefix

8. Design System

Färger: Se DESIGN_SYSTEM.md

Tailwind CSS: Använd Tailwind-klasser, undvik inline styles

Dark theme: Slate-900/800/700 för bakgrunder, Slate-200/300 för text

Accent: Blue-500/600 för primära knappar, Red-500 för destruktiva åtgärder

9. Relations-hantering

Struktur: person.relations = { parents: [], children: [], spouseId: null, partners: [], siblings: [] }

Synkronisering: Använd syncRelations.js för tvåvägssynkronisering

Byggning: Använd buildRelationsFromPeople för att bygga dbData.relations array från people array

Spara: Relations sparas som JSON i people.relations, inte i separat relations tabell

10. Media Connections

Struktur: media.connections = { people: [], sources: [], places: [], events: [] }

Länkning: Använd MediaSelector för att länka media till personer/källor/platser

Visning: Visa kopplingar i MediaManager, EditPersonModal, SourceCatalog, PlaceCatalogNew

11. Tag-hantering

Centraliserad: Använd useApp().getAllTags() för att hämta alla taggar från people/sources/media/places

Autocomplete: Implementera autocomplete med getAllTags() i alla tag-inputs

Format: Array av strings, t.ex. ['family', 'test']

12. Viktiga Filer

electron-test/main.js - Electron main process, IPC handlers, databasoperationer

src/useAppContext.js - Global state, auto-save, state wrappers

src/App.jsx - Routing, global handlers

src/hooks/useMediaUrl.js - Hook för media:// URL-hantering

src/components/MediaImage.jsx - Komponent för att visa bilder

src/syncRelations.js - Relations-synkronisering

src/database.js - Frontend database utilities

13. Felhantering

Media-filer: Om en bild inte hittas, visa felmeddelande men krascha inte appen

Databas: Använd transaktioner (BEGIN TRANSACTION / COMMIT / ROLLBACK)

IPC: Returnera { error: '...' } objekt vid fel, inte throw exceptions

14. Prestanda

Debouncing: Auto-save använder 300ms debounce

Deep comparison: Använd JSON.stringify för att jämföra objekt i useEffect

Lazy loading: Använd loading="lazy" för bilder

15. När du ändrar kod

Testa auto-save: Verifiera att ändringar sparas automatiskt

Testa media: Verifiera att bilder visas korrekt med relativa sökvägar

Testa relations: Verifiera att relations sparas korrekt

Kolla konsolen: Läs Electron-konsolen (inte webbläsarens) för debug-loggar

Viktiga Regler

ALLA sökvägar ska vara relativa från media/ - INGA absoluta sökvägar!

ALLA ändringar sparas automatiskt - INGA undantag!

Använd MediaImage komponenten för att visa bilder

Använd state wrappers för att säkerställa auto-save

Rekursiv sökning ska alltid användas för media-filer

Använd dbDataRef.current i callbacks för att undvika stale closures

Deep copy objekt innan mutationer

Mappa sex ↔ gender vid save/load från databas

Family Tree Visualization Rules (Visualisering)

When working with VerticalFamilyTree.jsx or any family tree visualization, follow these specific rules:

1. LAYOUT STRATEGY: Vertical Step Layout

NEVER use standard tree libraries (D3, Dagre) unless requested.

Use the custom "Vertical Step" logic where:

The Focus Person is centered.

Siblings are on the same Y-level (Older left, Younger right).

Parents (Gen 1) are centered above.

Partners (Gen 2) are listed VERTICALLY under the focus person.

Children (Gen 3) are listed under their specific parent, indented to the right.

2. LINE ROUTING: Orthogonal ONLY

STRICT RULE: Never draw diagonal straight lines (like L x2 y2).

All lines must use "Manhattan routing" (only Vertical and Horizontal segments).

Use SVG Path commands: M (Move), V (Vertical), H (Horizontal).

3. SPECIFIC PATH LOGIC

Siblings -> Parents: Elbow-Up (M start V midY H targetX V targetY).

Partner -> Child: Step-Indent (M startX startY V childY H childX).

Focus -> Partner: Step-Left (M startX startY V partnerY H partnerX).

Step-Parent -> Bio-Parent: Straight Horizontal (dashed).

4. COMPLEX RELATIONSHIPS

Always account for "Step-Parents" (partners to parents who are not the user's biological parent).

Draw siblings connecting to the correct parent-pair midpoint.

5. STYLING

Use the internal STYLES constant for CSS (no external CSS files or Tailwind dependencies unless requested).

Cards must have fixed dimensions defined in CONFIG.

When modifying VerticalFamilyTree.jsx, PRESERVE the calculateVerticalLayout logic structure. Do not simplify it back to a standard tree.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RogerDodger8989) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
