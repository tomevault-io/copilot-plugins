## dialoguegenerator

> Règles spécifiques pour l'éditeur de graphe (GraphEditor/GraphCanvas/graphStore + API /graph).


- **Invariant éditeur de nœud** : au changement de sélection dans `NodeEditorPanel`, flush le formulaire avec **`mergeNodeFormIntoStoreData()`** / **`mergeDialogueNodeFormIntoStoreData()`** (`frontend/src/utils/mergeNodeEditorForm.ts`), pas un spread `{ ...nodeData, ...formValues }` — sinon `choices[i].targetNode` (rempli par `connectNodes` / `edgeSlice`) peut être écrasé. Détails du flux dans `AGENTS.md` (Learned Workspace Facts).

- **Architecture contrôlée React Flow (ADR-007)** :
  - `GraphCanvas` reste en **mode controlled** : `nodes` / `edges` proviennent exclusivement de `useGraphStore`, jamais de `useState` local.
  - Toute mutation de graphe passe par les slices du store (`nodeSlice`, `edgeSlice`, `undoSlice`, `layoutSlice`, `persistenceSlice`, `uiSlice`) et **jamais** par des setters React Flow.
  - Les handlers React Flow (`onNodesChange`, `onEdgesChange`, `onConnect`, drag, context menu…) sont centralisés dans `useReactFlowHandlers` ; ne pas réintroduire de logique événementielle directement dans `GraphCanvas`.

- **Contrôleur de vue typé (`graphViewStore`)** :
  - Toute commande UI inter-composants (focus nœud, fitView, menu contextuel, édition label d'arête, ouverture panneau IA/prompt, flush/save, notification suppression) passe par `useGraphViewStore` (`frontend/src/store/graphViewStore.ts`).
  - **Ne plus utiliser** `window.dispatchEvent(new CustomEvent(...))` ni `window.addEventListener(...)` pour la communication inter-composants du graphe. Ces patterns ont été intégralement remplacés.
  - Les producteurs appellent les actions du store (ex: `useGraphViewStore.getState().focusNode(nodeId)` qui **enqueue** dans `focusQueue`). `GraphCanvas` consomme la tête via `dequeueFocus()` après `fitView`. `clearFocus()` vide toute la file.
  - Le protocole flush/save fonctionne ainsi : `requestFlush()` → `NodeEditorPanel` exécute le flush → `confirmFlush()` → `useDialogueLoader` détecte `flushCompleted` et déclenche la sauvegarde.

- **Helper de transaction graphe (`runGraphTransaction`)** :
  - Toute mutation de store qui doit pousser un snapshot undo + mettre à jour nodes/edges + synchroniser document/layout + marquer dirty doit utiliser `runGraphTransaction` (`frontend/src/store/utils/runGraphTransaction.ts`).
  - Le helper accepte des options (`skipUndo`, `skipSyncDoc`, `skipMarkDirty`) pour les cas spéciaux (ex: undo/redo lui-même).
  - `edgeSlice` (`connectNodes`, `disconnectNodes`, `updateChoiceEdgeLabel`) et `nodeSlice` (`addNode`, `deleteNode`) l'utilisent déjà. `layoutSlice` conserve sa logique custom de sync.

- **Flux de données graphe** :
  - Chargement / sauvegarde s'effectuent uniquement via `graphStore` → `graphAPI` → `/api/v1/unity-dialogues/graph/*` (`load`, `save`, `save-and-write`, `validate`, `calculate-layout`, `generate-node`, `accept/reject`).
  - **Ne jamais créer** d'endpoints dédiés `/connections`, `/position`, `/batch-*` : toute persistance se fait via `save-and-write` (ADR-006).
  - La conversion Unity JSON ↔ React Flow reste confinée aux services backend (`GraphConversionService`) et aux schémas `api/schemas/graph.py`.

- **Sélection, multi-sélection et opérations batch** :
  - `selectedNodeId` et `selectedNodeIds` sont la **source de vérité** (slice UI). La multi-sélection se gère via `setSelectedNodes` / `clearSelection` + `multiSelectionKeyCode="Shift"` et `selectionOnDrag` sur `<ReactFlow>`.
  - Les opérations batch (delete/tag/validate) s'appuient sur des actions de store (`batchDeleteNodes`, `batchTagNodes`, etc.) et sur `BatchOperationsMenu` ; ne jamais dupliquer la logique côté composants.
  - Tout nouveau comportement lié à la sélection doit être couvert par des tests `graphStore.*.test.ts` **et** des tests d'intégration `GraphEditor.*.test.tsx`.

- **Recherche, jump, filtres** :
  - Recherche, jump et filtres sont **100 % client** :
    - `searchNodes`, `findNodesByQuery`, `setFilters` / `resetFilters`, `getUniqueSpeakers` vivent dans le slice UI ; aucun appel backend dédié.
    - Les composants `GraphSearchBar`, `JumpToNodeModal`, `GraphFiltersPanel` consomment `useGraphStore` et `useGraphViewStore.focusNode()` pour le centrage.
  - Réutiliser `highlightedNodeIds` pour tous les highlights (search, validation, cycles…), en les dérivant dans `GraphCanvas` plutôt que via des styles locaux ad hoc.

- **Undo/redo et auto-layout** :
  - L'historique undo/redo est géré via `undoSlice` (`undoStack` / `redoStack`, `_pushUndoSnapshot`, `undo`, `redo`) avec limite stricte à 50 snapshots.
  - On ne pousse un snapshot qu'aux **entrées de transactions** (add/delete/connect/disconnect/move stop…), jamais sur chaque frame de drag (utiliser le throttling existant).
  - **Layout** : `layoutSlice.applyAutoLayout` avec algorithme `dagre` utilise **dagre côté client** (`utils/dagreLayout`) pour un recalcul local rapide ; les **autres** algorithmes passent par `POST /graph/calculate-layout`. La **génération de nœuds** peut aussi appliquer dagre client pour lisibilité immédiate. Ne pas dupliquer une troisième variante sans aligner tests et epics.

- **Performances et UX (NFR-P1/P4, NFR-SC3, NFR-A1)** :
  - Garder `onlyRenderVisibleElements={true}`, `minZoom` / `maxZoom` et `autoPanOnNodeDrag` activés sur `<ReactFlow>` ; ne pas les désactiver sans raison forte + tests.
  - Tout nouveau raccourci clavier passe par `useKeyboardShortcuts`, avec vérification explicite du focus (pas d'actions globales quand le focus est dans un input/textarea/select).
  - Les nouveaux comportements du canvas doivent être couverts par des tests ciblés (`GraphCanvas.*.test.tsx`, `GraphEditor.keyboard.test.tsx`, tests de perf/logique sur le store).

- **Tests et non-régression** :
  - Chaque nouvelle story Epic 2+ pour le graphe doit ajouter au moins :
    - 1 test de store (`graphStore.*.test.ts`) pour la logique métier.
    - 1 test de composant (`Graph*.test.tsx`) pour l'intégration UI.
  - Ne jamais supprimer un test graph existant sans vérifier que la feature correspondante est réellement dépréciée et documentée dans les epics.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Radical-Horizons)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Radical-Horizons)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
